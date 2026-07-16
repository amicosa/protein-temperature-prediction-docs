# Futuras direcciones y mejoras propuestas

A pesar de que los modelos desarrollados (Baseline y CNN posicional) han demostrado ser una prueba de concepto sólida, existen numerosas vías de mejora. Estas propuestas no son arbitrarias, sino que surgen directamente de las limitaciones observadas en los análisis de sesgos y en el rendimiento de los modelos actuales.

## 🔬 Mejoras en el modelo y la arquitectura

1. **Incorporación de mecanismos de atención (Transformer en lugar de CNN):**
El modelo 1D-CNN actual es excelente detectando motivos locales y patrones de corto alcance (unos pocos aminoácidos adyacentes). Sin embargo, la estabilidad térmica de una proteína a menudo depende de **interacciones de largo alcance** (por ejemplo, cómo un residuo en el extremo N-terminal estabiliza un pliegue en el extremo C-terminal). Sustituir la CNN por un **Encoder Transformer** (como el propio ESM-2) permitiría al modelo aprender estas dependencias globales, mejorando potencialmente la precisión en los rangos intermedios (45-80 °C).

2. **Aprendizaje multitarea (Multi-task Learning):**
Actualmente, el modelo se entrena exclusivamente para predecir `T_opt`. Una mejora significativa sería entrenarlo para predecir simultáneamente `T_min`, `T_opt` y `T_max`. Esto no solo aprovecharía toda la información disponible en TEMPURA, sino que forzaría al modelo a aprender relaciones más complejas entre la secuencia y el **espectro térmico completo** del organismo, mejorando su generalización.

3. **Modelos fundacionales más grandes:**
La versión actual de ESM-2 utilizada (`t6_8M`, 8M parámetros) es la más pequeña de la familia por motivos de eficiencia computacional. Futuras iteraciones deberían evaluar modelos de mayor capacidad, como `esm2_t36_3B_UR50D` (3.000 millones de parámetros). Aunque más costosos, estos modelos capturan representaciones semánticas mucho más profundas y complejas de las secuencias proteicas.

## 🌿 Mejoras en la preparación del dataset y los embeddings

4. **Integración de información estructural (AlphaFold DB):**
Dado que la estabilidad térmica está intrínsecamente determinada por la estructura tridimensional final (no solo por la secuencia primaria), se propone enriquecer el dataset con **embeddings estructurales** procedentes de AlphaFold DB. La incorporación de información sobre plegamiento y contactos de residuos podría aumentar drásticamente el R² en los rangos termófilos, donde la secuencia es más ambigua.

5. **Expansión del dataset para cubrir el "missing middle":**
La principal limitación de los modelos actuales es la escasez de muestras en el rango de 45-80 °C. Se propone ampliar la búsqueda en UniProt incluyendo una mayor diversidad de filos bacterianos y **explorar bases de datos complementarias** a TEMPURA, como **BacDive** o **NCBI BioSample**, que ofrecen información térmica de crecimiento para un espectro más amplio de microorganismos.

6. **Optimización del alineamiento y el padding:**
Actualmente, las secuencias se recortan a 1000 posiciones para controlar el uso de memoria. Si en el futuro se dispone de recursos suficientes, se podría probar un **alineamiento sin recorte** o, alternativamente, implementar un esquema de **padding dinámico con masking** (similar al usado por los Transformers) para que el modelo no tenga que "aprender" a ignorar ceros en las posiciones recortadas.

## 🧪 Validación experimental (Benchmarking)
Toda predicción in silico gana una credibilidad inmensurable cuando se contrasta con la realidad. A medio plazo, lo ideal establecer una colaboración con un laboratorio de bioquímica para:

* **Seleccionar un conjunto de proteínas** cuyas temperaturas óptimas hayan sido predichas por el modelo con alta confianza (y otras con baja confianza).
* **Validarlas experimentalmente** mediante ensayos de actividad enzimática a diferentes temperaturas.