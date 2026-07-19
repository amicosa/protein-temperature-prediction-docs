# Roadmap

Use the tabs below to switch between English and Spanish.

=== "English"

    ## 🎯 Vision Statement

    The ultimate goal of this project is to build a **robust, scalable, and interpretable pipeline** for predicting biophysical properties of proteins from their sequence embeddings. 
    
    While RuBisCO is the current model system, the methodologies developed here are designed to be enzyme-agnostic. The roadmap below details the concrete steps we will take to achieve this vision.

    ---

    ## 🚀 Phase 1: Data Expansion (Short-term)

    *Goal: Address the "Missing Middle" by enriching the dataset with more thermophilic examples.*

    - **Integrate additional thermal databases:** Expand beyond TEMPURA by incorporating growth temperature data from **BacDive** and **NCBI BioSample**. This will specifically target bacterial and archaeal strains in the 45–80 °C range.

    - **Expand taxonomic diversity:** Modify the UniProt query to include a wider variety of bacterial phyla, reducing the current bias towards plants and hyperthermophilic archaea.

    - **Re-train and re-evaluate:** Once the new data is cleaned and aligned, re-train the Positional CNN and the Baseline Ensemble on the expanded dataset to measure the quantitative improvement in the thermophilic range.

    ---

    ## 🧠 Phase 2: Model and Representation Upgrades (Medium-term)

    *Goal: Improve predictive accuracy and extract richer biological insights through advanced architectures.*

    - **Scale up ESM-2:** Transition from the `esm2_t6_8M` (8M parameters) to larger variants like `esm2_t12_35M_UR50D` (35M) or `esm2_t36_3B_UR50D` (3B). This will provide higher-quality, deeper embeddings that capture more complex evolutionary relationships.

    - **Implement Transformer architectures:** Replace the 1D-CNN with a **Transformer Encoder**. Transformers can learn long-range dependencies between residues (e.g., N-terminus to C-terminus interactions), which are critical for thermostability in hyperthermophiles.

    - **Incorporate structural embeddings:** Test embeddings from **AlphaFold DB** or **ESM-3** (which integrates sequence, structure, and function). Since thermostability is fundamentally a structural property, adding 3D-aware embeddings could significantly improve accuracy, especially in the hyperthermophilic range where sequence variability is higher.

    - **Multi-task Learning:** Train models to predict `T_min`, `T_opt`, and `T_max` simultaneously. This will force the model to learn the full thermal spectrum of each organism, potentially improving generalization.

    ---

    ## 🧪 Phase 3: Experimental Validation (Long-term)

    *Goal: Move from in-silico predictions to real-world biological validation.*

    - **Establish collaborations:** Partner with a biochemistry or molecular biology laboratory to validate the model's predictions experimentally.

    - **Targeted assay design:** Select a set of RuBisCO sequences where the model's predictions are:

        - **High confidence:** To confirm the model works correctly.

        - **Low confidence or unexpected:** To identify novel biological insights or edge cases where the model fails.

    - **Experimental benchmarking:** Perform enzyme activity assays across a range of temperatures to determine the true optimal growth temperature (T_opt) of the selected variants.

    - **Iterative feedback loop:** Use the experimental results to refine the model's training data and feature engineering, creating a true "dry-lab" to "wet-lab" feedback cycle.

    ---

    ## 🌍 Phase 4: Generalization and Scaling (Vision)

    *Goal: Expand the pipeline beyond RuBisCO to other enzymes and properties.*

    - **Extend to new enzymes:** Apply the same pipeline to other families of enzymes with diverse thermal profiles (e.g., **DNA polymerases**, **amylases**, or **proteases**). This will test the model's ability to generalize its learned representations across completely different protein folds.

    - **Predict other biophysical properties:** Move beyond temperature to predict properties like **optimal pH**, **catalytic efficiency (kcat)**, or **substrate specificity** using the same embedding-based approach.

    - **Open-source contributions:** Upon successful generalization, release the methodology and benchmark datasets as a public resource, contributing to the broader protein engineering and synthetic biology community.


=== "Español"

    ## 🎯 Declaración de visión

    El objetivo final de este proyecto es construir un **pipeline robusto, escalable e interpretable** para predecir propiedades biofísicas de proteínas a partir de sus embeddings de secuencia.
    
    Aunque RuBisCO es el sistema modelo actual, las metodologías desarrolladas aquí están diseñadas para ser independientes de la enzima. El siguiente roadmap detalla los pasos concretos que daremos para lograr esta visión.

    ---

    ## 🚀 Fase 1: Expansión de datos (Corto plazo)

    *Objetivo: Abordar el "Missing Middle" enriqueciendo el conjunto de datos con más ejemplos termófilos.*

    - **Integrar bases de datos térmicas adicionales:** Ampliar más allá de TEMPURA incorporando datos de temperatura de crecimiento de **BacDive** y **NCBI BioSample**. Esto se dirigirá específicamente a cepas bacterianas y de arqueas en el rango de 45–80 °C.

    - **Expandir la diversidad taxonómica:** Modificar la consulta de UniProt para incluir una variedad más amplia de filos bacterianos, reduciendo el sesgo actual hacia plantas y arqueas hipertermófilas.

    - **Re-entrenar y re-evaluar:** Una vez que los nuevos datos estén limpios y alineados, re-entrenar la CNN Posicional y el Ensemble del Baseline en el conjunto de datos ampliado para medir la mejora cuantitativa en el rango termófilo.

    ---

    ## 🧠 Fase 2: Mejoras en el modelo y la representación (Medio plazo)

    *Objetivo: Mejorar la precisión predictiva y extraer conocimientos biológicos más ricos a través de arquitecturas avanzadas.*

    - **Escalar ESM-2:** Realizar la transición de `esm2_t6_8M` (8M parámetros) a variantes más grandes como `esm2_t12_35M_UR50D` (35M) o `esm2_t36_3B_UR50D` (3B). Esto proporcionará embeddings de mayor calidad y más profundos que capturen relaciones evolutivas más complejas.

    - **Implementar arquitecturas Transformer:** Reemplazar la 1D-CNN por un **Encoder Transformer**. Los Transformers pueden aprender dependencias de largo alcance entre residuos (por ejemplo, interacciones N-terminal a C-terminal), que son críticas para la termoestabilidad en los hipertermófilos.

    - **Incorporar embeddings estructurales:** Probar embeddings de **AlphaFold DB** o **ESM-3** (que integra secuencia, estructura y función). Dado que la termoestabilidad es fundamentalmente una propiedad estructural, añadir embeddings conscientes de la estructura 3D podría mejorar significativamente la precisión, especialmente en el rango hipertermófilo, donde la variabilidad de la secuencia es mayor.

    - **Aprendizaje multitarea:** Entrenar modelos para predecir `T_min`, `T_opt` y `T_max` simultáneamente. Esto forzará al modelo a aprender el espectro térmico completo de cada organismo, mejorando potencialmente la generalización.

    ---

    ## 🧪 Fase 3: Validación experimental (Largo plazo)

    *Objetivo: Pasar de las predicciones in-silico a la validación biológica en el mundo real.*

    - **Establecer colaboraciones:** Asociarse con un laboratorio de bioquímica o biología molecular para validar las predicciones del modelo experimentalmente.

    - **Diseño de ensayos dirigidos:** Seleccionar un conjunto de secuencias de RuBisCO donde las predicciones del modelo sean:

        - **De alta confianza:** Para confirmar que el modelo funciona correctamente.

        - **De baja confianza o inesperadas:** Para identificar nuevos conocimientos biológicos o casos límite en los que el modelo falla.

    - **Benchmarking experimental:** Realizar ensayos de actividad enzimática en un rango de temperaturas para determinar la temperatura óptima de crecimiento (T_opt) real de las variantes seleccionadas.

    - **Bucle de retroalimentación iterativo:** Utilizar los resultados experimentales para refinar los datos de entrenamiento y la ingeniería de características del modelo, creando un verdadero ciclo de retroalimentación entre el "laboratorio seco" y el "laboratorio húmedo".

    ---

    ## 🌍 Fase 4: Generalización y escalado (Visión)

    *Objetivo: Ampliar el pipeline más allá de RuBisCO a otras enzimas y propiedades.*

    - **Extender a nuevas enzimas:** Aplicar el mismo pipeline a otras familias de enzimas con perfiles térmicos diversos (por ejemplo, **ADN polimerasas**, **amilasas** o **proteasas**). Esto pondrá a prueba la capacidad del modelo para generalizar sus representaciones aprendidas a través de pliegues proteicos completamente diferentes.

    - **Predecir otras propiedades biofísicas:** Ir más allá de la temperatura para predecir propiedades como el **pH óptimo**, la **eficiencia catalítica (kcat)** o la **especificidad de sustrato** utilizando el mismo enfoque basado en embeddings.
    
    - **Contribuciones de código abierto:** Tras una generalización exitosa, liberar la metodología y los conjuntos de datos de referencia como un recurso público, contribuyendo a la comunidad más amplia de ingeniería de proteínas y biología sintética.