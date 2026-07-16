# Modelo CNN Posicional (1D-CNN)

## 📖 Descripción general del enfoque

El modelo CNN posicional representa el segundo escalón en la complejidad del proyecto. Mientras que el Baseline utilizaba **embeddings promediados** (perdiendo toda la información de posición), este enfoque explota la **naturaleza secuencial de las proteínas** mediante el uso de embeddings **por residuo** (token-level).

Para ello, las secuencias se alinean previamente contra una referencia común, y los embeddings de ESM-2 se proyectan sobre dicho alineamiento. Esto permite que una **Red Neuronal Convolucional 1D (1D-CNN)** aprenda motivos locales y patrones de corto alcance directamente relacionados con la estabilidad térmica de la enzima.

## ⚙️ Preparación de los datos para el modelo

* **Tipo de representación:** Embeddings posicionales (por residuo). Matriz tridimensional de forma `(N, L_fija, 768)`, donde `N` es el número de muestras, `L_fija` es la longitud del alineamiento y `768` es la dimensión del embedding de ESM-2 (`t6_8M`).

* **Preprocesamiento específico:** 

  * **Alineamiento por pares:** Las secuencias crudas se alinean contra una referencia (la primera secuencia del dataset) utilizando el algoritmo de BioPython con matriz `BLOSUM62`.

  * **Recorte de longitud (`MAX_LENGTH = 1000`):** Para controlar el uso de memoria y estandarizar la entrada, las secuencias alineadas se recortan a una longitud fija de 1000 posiciones.

  * **Proyección de embeddings:** Los embeddings crudos de ESM-2 se extraen y se proyectan sobre el alineamiento. Las posiciones que contienen un gap (`-`) se rellenan con **vectores de ceros**, mientras que las posiciones con aminoácidos conservan su embedding original.

  * **Almacenamiento en HDF5:** Debido al tamaño de la matriz resultante (aprox. 2-3 GB en RAM), los embeddings posicionales se guardan en un archivo `.h5` comprimido para su reutilización en múltiples experimentos sin necesidad de re-extraerlos.

* **División de datos:** Para el entrenamiento de la CNN se utilizó una **división aleatoria simple 80/20** (Train/Validation) utilizando `train_test_split` de scikit-learn. A diferencia del Baseline, no se aplicó estratificación, ya que el objetivo principal de esta partición era monitorizar la curva de pérdida para aplicar *early stopping* y evitar el sobreajuste.

## 🧠 Arquitectura y configuración del modelo

* **Arquitectura (1D-CNN):**

  * **Capas convolucionales:** Tres bloques secuenciales de `Conv1d` con `padding=same`, seguidos de `BatchNorm1d`, activación `ReLU` y `Dropout`.

  * **Canales y kernels:** `conv_channels = [128, 256, 128]`, `kernel_sizes = [5, 5, 3]`.

  * **Pooling:** `AdaptiveAvgPool1d(1)` para condensar la información posicional en un vector global.

  * **Capas fully-connected (MLP):** `Linear(128 -> 64)` con activación `ReLU` y `Dropout`, seguido de una capa final `Linear(64 -> 1)` para la regresión de la temperatura.

* **Hiperparámetros de entrenamiento:**

  * **Optimizador:** `Adam` con `lr = 1e-3` y `weight_decay = 1e-4`.

  * **Función de pérdida:** `MSELoss` (Error cuadrático medio).

  * **Batch size:** `32`.

  * **Épocas máximas:** `100`.

  * **Early Stopping:** `patience = 15` (detiene el entrenamiento si la pérdida de validación no mejora durante 15 épocas consecutivas).

## 🎯 Justificación del enfoque

La elección de un modelo 1D-CNN basado en embeddings posicionales se justifica por varias razones biológicas y técnicas:

1. **Preservación del contexto posicional:** La estabilidad térmica no depende solo de la composición global de aminoácidos, sino de **motivos locales** y de cómo interactúan residuos adyacentes. La CNN está diseñada para detectar estos patrones de corto alcance (por ejemplo, "si en la posición 45 hay Glicina y en la 47 Prolina, la proteína es más flexible").

2. **Manejo de alineamientos (gaps):** Al proyectar los embeddings sobre un alineamiento fijo (rellenando con ceros las posiciones con gaps), la CNN aprende a ignorar esas regiones no conservadas y a centrarse en las posiciones informativas, que son las que realmente determinan la estructura y función de la proteína.

3. **Balance entre capacidad y eficiencia:** Una CNN 1D es computacionalmente mucho más ligera que un Transformer (como el propio ESM-2). Esto permite entrenar modelos personalizados sobre los embeddings de manera rápida y efectiva en una GPU T4, sin renunciar a la capacidad de aprender patrones posicionales complejos.

## 📊 Evaluación de resultados

*(Estos valores se rellenarán tras la ejecución del notebook `5_Positional_training.ipynb`)*

* **Métricas principales (Validation set - Mejor época `40`):** 

  * **RMSE (Validación):** `7.54 °C`

  * **R² (Validación):** `0.939`

  * **Pérdida (Loss):** `56.89`

### **Gráficos relevantes:**

#### **Curva de pérdida (Loss curve):** 

![Curva de pérdida del CNN](../assets/figures/models/loss_curve.png)

El gráfico muestra una **caída abrupta y espectacular** de la pérdida (MSE) en las primeras 5 épocas, pasando de valores cercanos a 3400 a valores por debajo de 200. A partir de la época 10, la pérdida se estabiliza cerca de cero. La pérdida de entrenamiento y validación se mantienen extremadamente cercanas a lo largo de todo el proceso, lo que indica que **no existe sobreajuste significativo**. El entrenamiento se detuvo en la época 55 debido al criterio de *early stopping* (`patience=15`), habiendo alcanzado su mejor rendimiento en la época 40 (que es el estado del modelo que se conserva).

#### **Gráfico de dispersión Predicción vs Real:** 

![Gráfico de dispersión real vs predicción](../assets/figures/models/real_vs_predicted_scatter.png)

El gráfico de dispersión confirma una **buena correlación general** entre los valores predichos y los reales, con la mayoría de los puntos alineados a lo largo de la línea diagonal Y=X (línea negra discontinua). Sin embargo, se observan dos fenómenos claros:

1. **Agrupación en los extremos:** Existe una alta densidad de puntos en torno a las temperaturas de ~25 °C y ~85-100 °C, coincidiendo con los nichos ecológicos mejor representados en el dataset.

2. **Mayor dispersión en el rango medio:** En el rango de 40-80 °C, los puntos son escasos y presentan una **mayor desviación** respecto a la línea de predicción perfecta (línea roja de regresión). Algunos valores reales de ~30-40 °C son sobrestimados hacia los 50 °C, mientras que otros en el rango de 70 °C son subestimados.

## 🔍 Análisis y limitaciones específicas

### **Fortalezas** 

* El enfoque posicional supera al Baseline al permitir que el modelo aprenda **dónde** están ocurriendo los cambios en la secuencia, no solo *qué* aminoácidos están presentes. Esto lo hace teóricamente más sensible a mutaciones puntuales y motivos de estabilidad térmica. La curva de pérdida confirma que el modelo converge de forma muy rápida y estable, sin signos de sobreajuste.

### **Debilidades / Limitaciones:**

* **El "missing middle" se confirma visualmente:** El gráfico de dispersión es la prueba gráfica definitiva del sesgo térmico documentado en el análisis del dataset. La escasez de muestras en el rango de 40-80 °C provoca que el modelo tenga una **menor capacidad de generalización en esa región**, resultando en predicciones menos precisas para organismos termófilos moderados.

* **Dependencia del alineamiento:** El rendimiento del modelo depende completamente de la calidad del alineamiento por pares. Si la referencia es muy divergente, la proyección de los embeddings puede perder información relevante.

* **Recorte a 1000 posiciones:** Aunque la mayoría de las secuencias de RuBisCO están por debajo de este umbral, existen algunas excepciones (ej. los outliers de 5902 aa). El recorte descarta la información de la cola de estas secuencias, lo que podría ser una limitación para estudiar proteínas de fusión con dominios extra.

* **Coste computacional:** La extracción de embeddings posicionales requiere un paso de alineamiento y ocupa ~2-3 GB en RAM, lo que hace que este pipeline sea más pesado que el de embeddings promediados.

## 💻 Código asociado

* Notebook principal: `5_Positional_training.ipynb`

* Scripts de soporte: 

  * `src/models/esm_positional_extractor.py` (Contiene la clase `ESMPositionalExtractor` para extraer y proyectar los embeddings posicionales, así como el guardado en HDF5).

  * `src/utils/alignment_utils.py` (Contiene la clase `SequenceAligner` para el alineamiento por pares contra la referencia).
