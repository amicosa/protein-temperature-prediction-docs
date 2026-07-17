# Phase 001: Feasibility Study (Classical ML)

Use the tabs below to switch between English and Spanish.

=== "English"

    ## 🔬 Phase 1: Feasibility Study (Classical ML with Mean Pooling)

    **Objective:**

    The first step was to establish a **performance baseline**. We asked: *"Is the averaged semantic representation learned by ESM-2 (`t6_8M`) sufficient to capture the biophysical properties that determine thermal stability?"* 

    **Setup & Methodology:**

    * **Representation:** We extracted per-sequence embeddings using `ESMFeatureExtractor` and applied **mean pooling** (averaging all token embeddings per sequence), resulting in a single 768-dimensional vector per protein.

    * **Models:** We trained a suite of classical algorithms: **Random Forest** (optimized via GridSearchCV), **Gradient Boosting**, **SVR**, and a **Voting Ensemble** combining all three.

    * **Validation:** A strict temperature-stratified 80/20 split was used to prevent data leakage and ensure a fair evaluation.

    **Key Findings & Lessons Learned:**

    * **The embedding works:** The Ensemble achieved a solid baseline performance (RMSE ~ `6.80` °C, R² ~ `0.953`). This confirmed that **ESM-2 embeddings inherently encode physicochemical and evolutionary information relevant to thermostability**.

    * **The limitation of pooling:** While the global average captures the overall "flavor" of the protein, it completely discards positional information. This limitation naturally suggested that preserving the sequence context could lead to a significant performance improvement.

    **Conclusion of Phase 1:** The classical models proved the viability of the approach but hit a "performance ceiling" dictated by the loss of spatial information. This justified moving to the more complex, position-aware Phase 2.


=== "Español"

    ## 🔬 Fase 1: Estudio de viabilidad (ML Clásico con Mean Pooling)

    **Objetivo:**
    El primer paso fue establecer un **punto de referencia (baseline)**. Nos preguntamos: *"¿Es la representación semántica promediada aprendida por ESM-2 (`t6_8M`) suficiente para capturar las propiedades biofísicas que determinan la estabilidad térmica?"*

    **Configuración y metodología:**

    * **Representación:** Se extrajeron embeddings por secuencia utilizando `ESMFeatureExtractor` y se aplicó **mean pooling** (promediando los embeddings de todos los tokens por secuencia), obteniendo un único vector de 768 dimensiones por proteína.

    * **Modelos:** Se entrenó un conjunto de algoritmos clásicos: **Random Forest** (optimizado mediante GridSearchCV), **Gradient Boosting**, **SVR** y un **Ensemble (Voting)** combinando los tres.

    * **Validación:** Se utilizó una estricta partición 80/20 estratificada por temperatura para evitar la fuga de datos y garantizar una evaluación justa.

    **Hallazgos clave y lecciones aprendidas:**

    * **El embedding funciona:** El Ensemble logró un rendimiento base sólido (RMSE ~ `6.80` °C, R² ~ `0.953`). Esto confirmó que **los embeddings de ESM-2 codifican inherentemente información fisicoquímica y evolutiva relevante para la termoestabilidad**.

    * **La limitación del pooling:** Si bien el promedio global captura el "sabor" general de la proteína, descarta por completo la información posicional. Esta limitación sugirió de forma natural que preservar el contexto de la secuencia podría conducir a una mejora significativa del rendimiento.

    **Conclusión de la Fase 1:** Los modelos clásicos demostraron la viabilidad del enfoque, pero alcanzaron un "techo de rendimiento" dictado por la pérdida de información espacial. Esto justificó el paso a la Fase 2, más compleja y consciente de la posición.
