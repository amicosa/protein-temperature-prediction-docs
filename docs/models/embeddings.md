# Estrategias de Vectorización y Modelos de Embeddings

Use the tabs below to switch between English and Spanish.

=== "English"

    This section describes the **foundation model** chosen to extract the vector representations of the protein sequences, as well as the different pooling and vectorization strategies (embeddings) employed in this project.

    ## 🧠 1. Choice of the Foundation Model: ESM-2

    To convert amino acid sequences into numerical vectors suitable for machine learning, we selected **ESM-2 (Evolutionary Scale Modeling 2)** developed by Meta AI.

    **Why ESM-2 over other models?**

    1. **State-of-the-art performance:** ESM-2 is one of the most powerful protein language models (pLMs) available, achieving top results in downstream tasks such as structure prediction and property prediction.

    2. **Evolutionary and structural encoding:** Trained on over 250 million protein sequences, ESM-2 captures deep evolutionary relationships and implicitly learns contact maps and secondary structure motifs, which are critical for predicting thermostability.

    3. **Ease of deployment:** Available via the Hugging Face `transformers` library, it allows for seamless integration into Python pipelines.

    4. **Resource efficiency:** We specifically selected the **`esm2_t6_8M_UR50D`** variant (8 million parameters) for this proof-of-concept. It offers the best trade-off between representation quality and computational efficiency, making it possible to run large-scale extraction on consumer-grade GPUs (like a T4 in Google Colab) in a matter of minutes.

    *Note: The embedding dimension depends on the chosen model variant (e.g., 320 for the 8M model, 480 for 35M, 640 for 150M).*

    ---

    ## 🧬 2. Vectorization Strategies: Mean Pooling vs. Positional

    Once the base sequence is tokenized and passed through the ESM-2 encoder, we are left with a matrix of embeddings—one vector per amino acid. Depending on the experiment, we have applied two different "pooling" strategies to convert this matrix into a usable input for our models.

    ### 📊 2.1. Mean Pooled Embeddings (Global Representation)
    **How it works:** We take the average of all per-token embeddings across the entire sequence. This condenses the whole protein into a **single, fixed-length vector** (e.g., 768 dimensions).

    **When it was used:** Phase 1 (Baseline / Classical ML).

    **Pros:**
    - **Simplicity:** Very easy to compute. No alignment is needed.
    - **Global properties:** Captures the overall composition and "average flavor" of the protein's physicochemical properties.

    **Cons:**
    - **Complete loss of spatial context:** It discards the order and relative position of residues, effectively treating the protein as a "bag of amino acids".

    *Note: This strategy is ideal for establishing a quick baseline to verify if the embedding space contains any relevant thermal information.*

    ---

    ### 🧩 2.2. Positional Embeddings (Per-Residue Representation)
    **How it works:** We keep **one vector per amino acid**, preserving the original order of the sequence. To make sequences comparable, we project them onto a **Multiple Sequence Alignment (MSA)**. Gaps in the alignment are represented as **zero-vectors**, effectively telling the downstream model to ignore those positions.

    **When it was used:** Phase 2 (1D-CNN Positional Model).

    **Pros:**

    - **Preserves structural context:** By using an MSA, homologous residues are aligned to the same column. The model learns which specific positions contribute to thermal stability.

    - **Spatial patterns:** Allows the model to learn local motifs (e.g., residue X at position 45 predicts flexibility).

    **Cons:**

    - **Computational overhead:** Requires a prior alignment step (using BioPython, Clustal Omega, or MAFFT).

    - **Memory usage:** The resulting 3D matrix (N samples × M positions × Embedding dimension) is significantly larger and requires HDF5 compression for storage.

    ---

    ## 🔮 3. Possible Future Embedding Strategies

    As the project evolves, we plan to explore additional embedding methods to break through the current performance ceiling:

    - **ESM-3 (Generative & Structural):** A newer generation model from Meta that integrates sequence, structure, and function simultaneously. It could provide richer embeddings without the need for explicit external alignments.

    - **AlphaFold Embeddings (Structure-aware):** Instead of using ESM-2, we could use embeddings derived from the structure predictions of AlphaFold. Since thermostability is fundamentally a structural property, these 3D-aware embeddings could boost accuracy in the problematic "missing middle" range.

    - **Hybrid Representations (SaProt):** SaProt uses a vocabulary that combines amino acids with their structural context (secondary structure), offering a middle ground between sequence-only and structure-only methods.

    - **Classical Physicochemical Features:** For comparison, we could also test traditional features like amino acid composition, hydrophobicity scales, and charge profiles to evaluate how well the deep learning embeddings outperform simple feature engineering.


=== "Español"

    Esta sección describe el **modelo fundacional** elegido para extraer las representaciones numéricas de las secuencias proteicas, así como las diferentes estrategias de vectorización (embeddings) empleadas a lo largo del proyecto.

    ## 🧠 1. Elección del modelo fundacional: ESM-2

    Para convertir las secuencias de aminoácidos en vectores numéricos adecuados para el aprendizaje automático, se seleccionó **ESM-2 (Evolutionary Scale Modeling 2)**, desarrollado por Meta AI.

    **¿Por qué ESM-2 y no otros modelos?**
    
    1. **Rendimiento de vanguardia:** ESM-2 es uno de los modelos de lenguaje de proteínas (pLMs) más potentes disponibles, logrando resultados líderes en tareas de predicción de estructura y propiedades.

    2. **Codificación evolutiva y estructural:** Entrenado con más de 250 millones de secuencias proteicas, ESM-2 captura relaciones evolutivas profundas y aprende implícitamente mapas de contacto y motivos de estructura secundaria, fundamentales para predecir la termoestabilidad.

    3. **Facilidad de uso:** Disponible a través de la librería `transformers` de Hugging Face, lo que permite una integración perfecta en pipelines de Python.

    4. **Eficiencia de recursos:** Se seleccionó específicamente la variante **`esm2_t6_8M_UR50D`** (8 millones de parámetros) para esta prueba de concepto. Ofrece el mejor equilibrio entre calidad de representación y eficiencia computacional, permitiendo ejecutar extracciones a gran escala en GPU de consumo (como una T4 en Google Colab) en cuestión de minutos.

    *Nota: La dimensión del embedding depende de la variante del modelo elegido (por ejemplo, 320 para el modelo 8M, 480 para 35M, 640 para 150M).*

    ---

    ## 🧬 2. Estrategias de vectorización: Mean Pooling vs. Posicional

    Una vez que la secuencia base es tokenizada y pasa por el codificador de ESM-2, obtenemos una matriz de embeddings (un vector por cada aminoácido). Dependiendo del experimento, hemos aplicado dos estrategias diferentes para convertir esta matriz en una entrada utilizable para nuestros modelos.

    ### 📊 2.1. Embeddings promediados (Mean Pooling / Representación global)

    **Cómo funciona:** Se toma el promedio de todos los embeddings por token a lo largo de toda la secuencia. Esto condensa la proteína completa en un **único vector de longitud fija** (por ejemplo, 320 o 768 dimensiones).

    **Cuándo se utilizó:** Fase 1 (Baseline / ML Clásico).

    **Ventajas:**

    - **Simplicidad:** Muy fácil de calcular. No se necesita alineamiento.

    - **Propiedades globales:** Captura la composición general y el "sabor" promedio de las propiedades fisicoquímicas de la proteína.

    **Desventajas:**

    - **Pérdida total del contexto espacial:** Descarta el orden y la posición relativa de los residuos, tratando la proteína como una "bolsa de aminoácidos".

    *Nota: Esta estrategia es ideal para establecer un baseline rápido y verificar si el espacio de embedding contiene información térmica relevante.*

    ---

    ### 🧩 2.2. Embeddings posicionales (Representación por residuo)

    **Cómo funciona:** Se conserva **un vector por cada aminoácido**, preservando el orden original de la secuencia. Para hacer comparables las secuencias, se proyectan sobre un **Alineamiento Múltiple de Secuencias (MSA)**. Los gaps en el alineamiento se representan como **vectores de ceros**, indicando efectivamente al modelo downstream que ignore esas posiciones.

    **Cuándo se utilizó:** Fase 2 (Modelo Posicional 1D-CNN).

    **Ventajas:**

    - **Preserva el contexto estructural:** Al usar un MSA, los residuos homólogos se alinean en la misma columna. El modelo aprende qué posiciones específicas contribuyen a la estabilidad térmica.

    - **Patrones espaciales:** Permite al modelo aprender motivos locales (ej. "el residuo X en la posición 45 predice flexibilidad").

    **Desventajas:**

    - **Costo computacional:** Requiere un paso de alineamiento previo (usando BioPython, Clustal Omega o MAFFT).

    - **Uso de memoria:** La matriz 3D resultante (N muestras × M posiciones × Dimensión del embedding) es significativamente mayor y requiere compresión HDF5 para su almacenamiento.

    ---

    ## 🔮 3. Posibles estrategias de embeddings futuros

    A medida que el proyecto evolucione, se planea explorar métodos adicionales para superar el techo de rendimiento actual:

    - **ESM-3 (Generativo y estructural):** Un modelo de nueva generación de Meta que integra secuencia, estructura y función simultáneamente. Podría proporcionar embeddings más ricos sin necesidad de alineamientos externos explícitos.

    - **Embeddings de AlphaFold (Con información de estructura):** En lugar de usar ESM-2, se podrían utilizar embeddings derivados de las predicciones estructurales de AlphaFold. Dado que la termoestabilidad es fundamentalmente una propiedad estructural, estos embeddings conscientes de la estructura 3D podrían aumentar la precisión en el problemático rango del "missing middle".

    - **Representaciones híbridas (SaProt):** SaProt utiliza un vocabulario que combina aminoácidos con su contexto estructural (estructura secundaria), ofreciendo un punto intermedio entre los métodos basados solo en secuencia y los basados en estructura.

    - **Características fisicoquímicas clásicas:** A modo de comparación, también se podrían probar características tradicionales como la composición de aminoácidos, escalas de hidrofobicidad y perfiles de carga para evaluar cómo los embeddings de deep learning superan a la ingeniería de características simple.