# Methodology

Use the tabs below to switch between English and Spanish.

=== "English"

    The complete implementation of this pipeline is available in the companion **[code repository](https://github.com/amicosa/Ruby-project)**.
    This section provides a conceptual and algorithmic overview of the entire workflow, from raw sequence retrieval to the final predictive model.

    ---

    ## 📊 Pipeline Overview

    The methodology is structured as a sequential pipeline with six main phases:

    ```mermaid
    flowchart TD
        A["1. Data Acquisition\nUniProt REST API"] --> B["2. Data Enrichment\nTEMPURA + Hierarchical Fallback"]
        B --> C["3. Sequence Alignment\nPairwise / MSA"]
        C --> D["4. Positional Embedding Extraction\nESM-2 + Projection"]
        D --> E["5. Feature Engineering & Split\nFilters + Stratified Split"]
        E --> F["6. Model Training & Evaluation\nEnsemble of Regressors"]
    ```

    ---

    ### 1. Data Acquisition (UniProt)

    **Objective:** Retrieve a diverse and curated set of RuBisCO sequences, with special emphasis on thermophilic archaea.

    **Implementation details** (`download_data.py`):

    * Query design: The script builds a complex Boolean query targeting:

        * The gene name (`rbcL`) and protein name ("Ribulose bisphosphate carboxylase").

        * Explicit terms for archaea (e.g., *Archaea*, *Pyrococcus*, *Sulfolobus*) to ensure thermal diversity.

        * Type III RuBisCO variants, which are characteristic of archaea.

    * **Pagination:** Uses UniProt's cursor-based pagination to handle large result sets (up to 2,000 records per run) without overwhelming the API.

    * **Output:** Raw TSV containing metadata (organism, lineage, sequence) and a tokenized version of the sequence (space-separated amino acids) for downstream ESM-2 compatibility.

    **Rationale:** RuBisCO is highly conserved but spans all domains of life. Including hyperthermophilic archaea ensures our dataset covers a wide T_opt range (5 °C to >100 °C), which is critical for training a robust regression model.

    ---

    ### 2. Data Enrichment (Thermal Parameters)

    **Objective:** Assign experimentally-derived or biologically-plausible thermal parameters (`T_min`, `T_opt`, `T_max`) to each sequence.

    **Implementation details** (`data_enricher.py`):

    We employ a **hierarchical fallback strategy** to maximize the number of sequences with thermal labels:

    1. Exact species match (TEMPURA database):
    The primary source is TEMPURA, a curated database of prokaryotic growth temperatures. We merge on `genus_and_species` (standardized to lower case).

    2. Plant lineage fallback:
    If a sequence lacks TEMPURA data but belongs to Viridiplantae or Streptophyta (green plants), we assign a default photosynthetic optimum:
    `T_min = 5.0`, `T_opt = 25.0`, `T_max = 35.0`.

    3. Genus-level fallback:
    If the exact species is not found, we aggregate TEMPURA data at the genus level (mean `T_opt` per genus) and use that average as a proxy.

    4. Unassigned:
    Sequences that fail all three rules are discarded from the modeling dataset (though they remain in the raw archive).

    **Rationale:** This hierarchical approach is a pragmatic balance between data quantity and quality. While exact TEMPURA matches are ideal, the genus-level and plant fallbacks allow us to retain ecologically relevant sequences that would otherwise be lost.

    ---

    ### 3. Sequence Alignment

    **Objective:** Homogenize sequence lengths to enable positional embedding extraction. This step ensures that homologous residues across different species align to the same index in the final tensor.

    **Implementation details** (`alignment_utils.py`):

    * **Method selection:** The `SequenceAligner` class supports two modes:

        * **Pairwise (default):** Aligns each sequence against a chosen reference (e.g., the first sequence or a provided canonical sequence) using BioPython's `PairwiseAligner` with BLOSUM62 matrix.

        * **External MSA:** Optionally uses **Clustal Omega** or **MAFFT** for multiple sequence alignment (MSA). If the external tool is missing, it gracefully falls back to pairwise alignment.

    * **Gap handling:** All alignments use `-` as the gap character. After alignment, sequences are padded to a uniform length (the maximum alignment length).

    * **Output:** A list of strings of equal length, where each position corresponds to a specific column in the multiple alignment.

    **Rationale:** Alignments are crucial for our positional embedding strategy. Without alignment, the same amino acid at index 50 in two different sequences would have no biological equivalence. By fixing the alignment, we create a "structural grid" that the model can learn from.

    ---

    ### 4. Positional Embedding Extraction (ESM-2)

    **Objective:** Convert aligned sequences into dense vector representations while preserving the alignment grid. This is the core innovation of our pipeline.

    **Implementation details** (`esm_positional_extractor.py`):

    This step is handled by the `ESMPositionalExtractor` class and follows a precise three-stage process:

    1. **Strip gaps:** For each aligned sequence, we remove all `-` characters to obtain the "raw" amino acid sequence (e.g., `M-V-L` -> `MVL`).

    2. **Extract raw embeddings:** The raw sequence is passed through the **ESM-2** model (we use `esm2_t6_8M_UR50D` for initial prototyping, but the architecture supports larger variants). ESM-2 returns a tensor of shape `(L_raw, 768)` where each token (amino acid) is mapped to a 768-dimensional vector.

    3. **Project back to alignment:** We create an empty tensor of shape `(L_aligned, 768)` filled with zeros. We then iterate over the aligned sequence:

        * If the character is an amino acid (not a gap), we place the next raw embedding vector into that position.

        * If the character is a gap (`-`), we leave the zero vector.

    **Result:** A tensor `X_pos` of shape `(N_samples, L_aligned, 768)`.
    Gap positions are represented as zero vectors, effectively masking them.

    **Storage:**

    * Embeddings are saved in **HDF5** format (`embeddings.h5`) with chunked compression (GZIP). This allows efficient random access and streaming without loading the entire tensor into memory.

    * Metadata (`fixed_length`, `embed_dim`, `model_name`) are stored as HDF5 attributes.

    **Rationale:**

    * Preserves evolutionary and structural context.

    * Allows the model to learn which alignment positions are critical for thermal stability.

    * Zero-masking handles gaps naturally, avoiding the need for complex padding mechanisms in the downstream regressor.

    ---

    ### 5. Feature Engineering and Data Split

    **Objective:** Clean the dataset, apply quality filters, and split the data to ensure unbiased evaluation.

    **Implementation details** (`train_predict.py`):

    **Quality filters** (applied via `prepare_data_with_filters`):

    * Remove sequences with `T_opt` outside the range [0, 100] °C.

    * Enforce a minimum sequence length of 350 residues unless the organism is an *archaeon* (to preserve thermophilic diversity).

    * Classify taxa into *Archaea*, *Plants*, *Bacteria*, or *Other* based on the taxonomic lineage.

    **Stratified split** (via `stratified_split_simple`):
    A simple random split would be biased if most mesophilic sequences end up in training and thermophilic ones in test. To prevent this:

    * We bin `T_opt` into 5 categories: [0-20), [20-40), [40-60), [60-80), [80-100].

    * We perform a **stratified split** (80/20), ensuring each temperature bin is proportionally represented in both training and test sets.

    * The function gracefully handles edge cases (e.g., bins with fewer than 2 samples) by falling back to random split with a warning.

    **Output:**

    `X_train`, `X_test`: Flattened embeddings of shape `(N, L_aligned * 768)` (the positional dimension is flattened into the feature vector).

    `y_train`, `y_test`: Corresponding `T_opt` labels.

    ---

    ### 6. Modeling and Prediction

    **Objective:** Train and compare multiple regression models to predict `T_opt` from the ESM-2 embeddings.

    **Implementation details** (`train_predict.py`):

    We adopt a **multi-model strategy** to identify the best performer:

    | Model | Strengths | Hyperparameter Optimization |
    |:---|:---|:---|
    | Random Forest | Handles high-dimensional sparse data well. Robust to outliers. | GridSearchCV over `n_estimators`, `max_depth`, `min_samples_split`. |
    | Gradient Boosting | Often yields higher accuracy on tabular data. Sequential learning. | Fixed parameters (`n_estimators=200`, `lr=0.1`). |
    | SVR (RBF) | Captures non-linear relationships. | Requires feature scaling (`StandardScaler`). Fixed `C=10`, `gamma='scale'`. |
    | Voting Ensemble | Combines the three above via averaging. Reduces overfitting. | Weighted voting (equal weights). |

    **Training pipeline** (orchestrated by `train_thermal_predictor_v2`):

    1. *Scale features* (only for SVR, but the ensemble automatically handles scaling via the scaler passed to SVR).

    2. Train each model independently.

    3. Evaluate on the held-out test set using:

        * **RMSE** (Root Mean Squared Error) as the primary metric.

        * **R²** (Coefficient of Determination).

        * **MAE** (Mean Absolute Error).

    4. Perform **5-fold cross-validation** on the training set for the ensemble to estimate generalization performance.

    **Saving:**

    The best ensemble and its scaler are serialized using `joblib` and stored in the `../models/` directory for later use in a web app or inference pipeline.

    ## 🛠️ Tools and Technologies

    | Tool/Library | Purpose |
    |:-------------|:--------|
    | Python 3.9+ | Core language. |
    | UniProt REST API | Sequence and metadata retrieval. |
    | Pandas / NumPy | Data manipulation and numerical operations. |
    | BioPython | Pairwise alignment, substitution matrices. |
    | Hugging Face Transformers | ESM-2 model loading and inference. |
    | PyTorch | Backend for ESM-2 (GPU acceleration). |
    | Scikit-learn | Model training, cross-validation, metrics. |
    | Joblib | Model serialization. |
    | H5Py | Efficient storage of large embedding tensors. |
    | MkDocs (Material) | Documentation framework. |

    ## 🔄 Reproducibility

    To ensure full reproducibility:

    All scripts accept explicit paths for inputs and outputs.

    Random seeds are fixed (`random_state=42`) across all splitting and training steps.

    The exact versions of dependencies will be frozen in a `requirements.txt` (to be added in the code repository).

    ## 📌 Future Methodological Extensions

    While this methodology focuses on supervised regression for `T_opt`, the framework is designed to be extensible:

    * **Generative Design:** Using the positional embeddings as conditioning for diffusion models (e.g., EvoDiff) to generate novel RuBisCO sequences.

    * **Multi-task Learning:** Predicting additional properties (optimal pH, catalytic rate `kcat`) simultaneously.

    * **Attention Visualization:** Extracting attention maps from ESM-2 to identify residues critical for thermal adaptation (e.g., via Integrated Gradients or attention rollout).

=== "Español"

    La implementación completa de este pipeline está disponible en el **[repositorio de código](https://github.com/amicosa/Ruby-project)**.
    Esta sección ofrece una visión conceptual y algorítmica de todo el flujo de trabajo, desde la recuperación de secuencias crudas hasta el modelo predictivo final.

    ---

    ## 📊 Resumen del pipeline

    La metodología está estructurada como un pipeline secuencial con seis fases principales:

    ```mermaid
    flowchart TD
        A["1. Adquisición de datos\nUniProt REST API"] --> B["2. Enriquecimiento de datos\nTEMPURA + fallback jerárquico"]
        B --> C["3. Alineamiento de secuencias\nPairwise / MSA"]
        C --> D["4. Extracción de embeddings posicionales\nESM-2 + proyección"]
        D --> E["5. Ingeniería de características y split\nFiltros + split estratificado"]
        E --> F["6. Entrenamiento y evaluación\nEnsamble de regresores"]
    ```

    ---

    ### 1. Adquisición de datos (UniProt)

    **Objetivo:** Recuperar un conjunto diverso y curado de secuencias de RuBisCO, con especial énfasis en arqueas termófilas.

    **Detalles de implementación** (`download_data.py`):

    * Diseño de consulta: El script construye una consulta booleana compleja orientada a:

        * El nombre del gen (`rbcL`) y el nombre de la proteína ("Ribulose bisphosphate carboxylase").

        * Términos explícitos de arqueas (por ejemplo, *Archaea*, *Pyrococcus*, *Sulfolobus*) para asegurar diversidad térmica.

        * Variantes de RuBisCO tipo III, características de arqueas.

    * **Paginación:** Utiliza paginación por cursor de UniProt para manejar conjuntos grandes (hasta 2,000 registros por corrida) sin sobrecargar la API.

    * **Salida:** TSV crudo con metadatos (organismo, linaje, secuencia) y una versión tokenizada de la secuencia (aminoácidos separados por espacios) compatible con ESM-2.

    **Justificación:** RuBisCO está altamente conservada, pero abarca todos los dominios de la vida. Incluir arqueas hipertermófilas garantiza que el dataset cubra un rango amplio de T_opt (5 °C a >100 °C), algo crítico para entrenar un modelo de regresión robusto.

    ---

    ### 2. Enriquecimiento de datos (parámetros térmicos)

    **Objetivo:** Asignar parámetros térmicos derivados experimentalmente o biológicamente plausibles (`T_min`, `T_opt`, `T_max`) a cada secuencia.

    **Detalles de implementación** (`data_enricher.py`):

    Empleamos una **estrategia de fallback jerárquico** para maximizar el número de secuencias con etiquetas térmicas:

    1. Coincidencia exacta de especie (base TEMPURA):
    La fuente primaria es TEMPURA, una base curada de temperaturas de crecimiento de procariotas. Hacemos merge por `genus_and_species` (normalizado en minúsculas).

    2. Fallback por linaje de plantas:
    Si una secuencia no tiene datos en TEMPURA pero pertenece a Viridiplantae o Streptophyta (plantas verdes), asignamos un óptimo fotosintético por defecto:
    `T_min = 5.0`, `T_opt = 25.0`, `T_max = 35.0`.

    3. Fallback a nivel de género:
    Si no se encuentra la especie exacta, agregamos los datos de TEMPURA a nivel de género (media de `T_opt` por género) y usamos ese valor como proxy.

    4. Sin asignación:
    Las secuencias que no cumplen ninguna de las tres reglas se descartan del dataset de modelado (aunque permanecen en el archivo crudo).

    **Justificación:** Este enfoque jerárquico es un equilibrio pragmático entre cantidad y calidad de datos. Aunque las coincidencias exactas con TEMPURA son ideales, los fallbacks por género y por plantas permiten conservar secuencias ecológicamente relevantes que de otro modo se perderían.

    ---

    ### 3. Alineamiento de secuencias

    **Objetivo:** Homogeneizar longitudes de secuencia para permitir la extracción de embeddings posicionales. Este paso asegura que residuos homólogos entre especies distintas queden alineados al mismo índice del tensor final.

    **Detalles de implementación** (`alignment_utils.py`):

    * **Selección de método:** La clase `SequenceAligner` soporta dos modos:

        * **Pairwise (por defecto):** Alinea cada secuencia contra una referencia elegida (por ejemplo, la primera secuencia o una secuencia canónica) usando `PairwiseAligner` de BioPython con matriz BLOSUM62.

        * **MSA externo:** Opcionalmente usa **Clustal Omega** o **MAFFT** para alineamiento múltiple (MSA). Si la herramienta externa no está disponible, hace fallback de forma segura al alineamiento pairwise.

    * **Manejo de gaps:** Todos los alineamientos usan `-` como carácter de gap. Tras alinear, las secuencias se rellenan hasta una longitud uniforme (la longitud máxima del alineamiento).

    * **Salida:** Una lista de cadenas de igual longitud, donde cada posición corresponde a una columna específica del alineamiento múltiple.

    **Justificación:** El alineamiento es clave para nuestra estrategia de embeddings posicionales. Sin alineamiento, el mismo aminoácido en el índice 50 de dos secuencias distintas no tendría equivalencia biológica. Al fijar el alineamiento, creamos una "rejilla estructural" que el modelo puede aprender.

    ---

    ### 4. Extracción de embeddings posicionales (ESM-2)

    **Objetivo:** Convertir secuencias alineadas en representaciones vectoriales densas preservando la rejilla del alineamiento. Esta es la innovación central del pipeline.

    **Detalles de implementación** (`esm_positional_extractor.py`):

    Este paso es gestionado por la clase `ESMPositionalExtractor` y sigue un proceso preciso de tres etapas:

    1. **Eliminar gaps:** Para cada secuencia alineada, removemos todos los caracteres `-` para obtener la secuencia aminoacídica "cruda" (por ejemplo, `M-V-L` -> `MVL`).

    2. **Extraer embeddings crudos:** La secuencia cruda se pasa por el modelo **ESM-2** (usamos `esm2_t6_8M_UR50D` para prototipado inicial, pero la arquitectura soporta variantes mayores). ESM-2 devuelve un tensor de forma `(L_raw, 768)` donde cada token (aminoácido) se mapea a un vector de 768 dimensiones.

    3. **Proyectar de vuelta al alineamiento:** Creamos un tensor vacío de forma `(L_aligned, 768)` relleno con ceros. Luego iteramos sobre la secuencia alineada:

        * Si el carácter es un aminoácido (no gap), colocamos el siguiente vector de embedding crudo en esa posición.

        * Si el carácter es un gap (`-`), dejamos el vector cero.

    **Resultado:** Un tensor `X_pos` de forma `(N_samples, L_aligned, 768)`.
    Las posiciones con gap se representan como vectores cero, enmascarándolas de forma efectiva.

    **Almacenamiento:**

    * Los embeddings se guardan en formato **HDF5** (`embeddings.h5`) con compresión por bloques (GZIP). Esto permite acceso aleatorio y streaming eficiente sin cargar todo el tensor en memoria.

    * Los metadatos (`fixed_length`, `embed_dim`, `model_name`) se guardan como atributos de HDF5.

    **Justificación:**

    * Preserva el contexto evolutivo y estructural.

    * Permite que el modelo aprenda qué posiciones del alineamiento son críticas para la estabilidad térmica.

    * El enmascaramiento con ceros maneja gaps de forma natural, evitando mecanismos complejos de padding en el regresor downstream.

    ---

    ### 5. Ingeniería de características y división de datos

    **Objetivo:** Limpiar el dataset, aplicar filtros de calidad y dividir los datos para asegurar una evaluación no sesgada.

    **Detalles de implementación** (`train_predict.py`):

    **Filtros de calidad** (aplicados con `prepare_data_with_filters`):

    * Eliminar secuencias con `T_opt` fuera del rango [0, 100] °C.

    * Exigir una longitud mínima de 350 residuos, salvo que el organismo sea una *arquea* (para preservar diversidad termófila).

    * Clasificar taxones en *Archaea*, *Plants*, *Bacteria* u *Other* según el linaje taxonómico.

    **Split estratificado** (vía `stratified_split_simple`):
    Un split aleatorio simple podría sesgar el conjunto si la mayoría de secuencias mesófilas quedan en entrenamiento y las termófilas en test. Para evitarlo:

    * Agrupamos `T_opt` en 5 categorías: [0-20), [20-40), [40-60), [60-80), [80-100].

    * Realizamos un **split estratificado** (80/20), asegurando que cada bin de temperatura esté representado proporcionalmente en entrenamiento y test.

    * La función maneja casos borde (por ejemplo, bins con menos de 2 muestras) haciendo fallback a split aleatorio con advertencia.

    **Salida:**

    `X_train`, `X_test`: Embeddings aplanados de forma `(N, L_aligned * 768)` (la dimensión posicional se aplana dentro del vector de características).

    `y_train`, `y_test`: Etiquetas `T_opt` correspondientes.

    ---

    ### 6. Modelado y predicción

    **Objetivo:** Entrenar y comparar múltiples modelos de regresión para predecir `T_opt` a partir de embeddings de ESM-2.

    **Detalles de implementación** (`train_predict.py`):

    Adoptamos una **estrategia multimodelo** para identificar el mejor desempeño:

    | Modelo | Fortalezas | Optimización de hiperparámetros |
    |:---|:---|:---|
    | Random Forest | Maneja bien datos dispersos y de alta dimensionalidad. Robusto a outliers. | GridSearchCV sobre `n_estimators`, `max_depth`, `min_samples_split`. |
    | Gradient Boosting | Suele ofrecer mayor precisión en datos tabulares. Aprendizaje secuencial. | Parámetros fijos (`n_estimators=200`, `lr=0.1`). |
    | SVR (RBF) | Captura relaciones no lineales. | Requiere escalado de características (`StandardScaler`). `C=10`, `gamma='scale'` fijos. |
    | Voting Ensemble | Combina los tres modelos por promedio. Reduce sobreajuste. | Votación ponderada (pesos iguales). |

    **Pipeline de entrenamiento** (orquestado por `train_thermal_predictor_v2`):

    1. *Escalar características* (solo para SVR, aunque el ensamble gestiona el escalado mediante el scaler asociado a SVR).

    2. Entrenar cada modelo de forma independiente.

    3. Evaluar en el conjunto de prueba con:

        * **RMSE** (Root Mean Squared Error) como métrica principal.

        * **R²** (coeficiente de determinación).

        * **MAE** (Mean Absolute Error).

    4. Realizar **validación cruzada de 5 folds** sobre entrenamiento para estimar capacidad de generalización del ensamble.

    **Guardado:**

    El mejor ensamble y su scaler se serializan con `joblib` y se almacenan en el directorio `../models/` para su uso posterior en una app web o pipeline de inferencia.

    ## 🛠️ Herramientas y tecnologías

    | Herramienta/Librería | Propósito |
    |:-------------|:--------|
    | Python 3.9+ | Lenguaje principal. |
    | UniProt REST API | Recuperación de secuencias y metadatos. |
    | Pandas / NumPy | Manipulación de datos y operaciones numéricas. |
    | BioPython | Alineamiento pairwise, matrices de sustitución. |
    | Hugging Face Transformers | Carga e inferencia del modelo ESM-2. |
    | PyTorch | Backend para ESM-2 (aceleración por GPU). |
    | Scikit-learn | Entrenamiento de modelos, validación cruzada y métricas. |
    | Joblib | Serialización de modelos. |
    | H5Py | Almacenamiento eficiente de tensores de embeddings grandes. |
    | MkDocs (Material) | Framework de documentación. |

    ## 🔄 Reproducibilidad

    Para asegurar reproducibilidad completa:

    Todos los scripts aceptan rutas explícitas para entradas y salidas.

    Las semillas aleatorias se fijan (`random_state=42`) en todos los pasos de split y entrenamiento.

    Las versiones exactas de dependencias quedarán congeladas en un `requirements.txt` (por añadir en el repositorio de código).

    ## 📌 Extensiones metodológicas futuras

    Aunque esta metodología se centra en regresión supervisada para `T_opt`, el marco de trabajo está diseñado para extenderse:

    * **Diseño generativo:** Usar embeddings posicionales como condicionamiento para modelos de difusión (por ejemplo, EvoDiff) y generar nuevas secuencias de RuBisCO.

    * **Aprendizaje multitarea:** Predecir simultáneamente propiedades adicionales (pH óptimo, tasa catalítica `kcat`).

    * **Visualización de atención:** Extraer mapas de atención de ESM-2 para identificar residuos críticos en la adaptación térmica (por ejemplo, con Integrated Gradients o attention rollout).
