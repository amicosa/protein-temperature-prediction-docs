# Changelog

Use the tabs below to switch between English and Spanish.

=== "English"

    ## [Unreleased]

    *No changes have been made yet for the next version.*

    ---

    ## [v1.0.0] - 2026-07-19

    ### 🎉 Added
    - **Project Foundation:** Initial repository setup, documentation structure (Overview, Motivation, Methodology, Datasets, Models, Experiments, Results, Roadmap).

    - **Data Pipeline:** Implementation of `src/utils/download_data.py` for UniProt API queries (RuBisCO + archaea focus) and `src/utils/data_enricher.py` for TEMPURA cross-referencing and hierarchical fallback rules (Plant heuristics, Genus-level averaging).

    - **Embedding Extraction:** `src/models/esm_extractor.py` (Mean Pooling for Baseline) and `src/models/esm_positional_extractor.py` (Per-residue positional embeddings with MSA projection and HDF5 storage).

    - **Baseline Models:** Training pipeline for classical ML models (`src/models/train_predict.py`): Random Forest (optimized with GridSearchCV), Gradient Boosting, SVR, and a Voting Ensemble.

    - **Positional CNN:** Implementation of the 1D-CNN architecture (`PositionalCNN` class) and training script with early stopping (patience=15) in `5_Positional_training.ipynb`.

    - **Interpretability:** Grad-CAM implementation to visualize positional importance across different thermal groups.

    - **Auxiliary Tools:** Sequence alignment utilities (`src/utils/alignment_utils.py` using BioPython).

    ### 📈 Changed
    - *Initial release – no prior changes to compare.*

    ### 🔧 Fixed
    - *Initial release – no prior bugs to fix.*

    ### 🗑️ Removed
    - *Initial release – no features were removed.*


=== "Español"

    ## [Unreleased]

    *No se han realizado cambios aún para la próxima versión.*

    ---

    ## [v1.0.0] - 2026-07-19

    ### 🎉 Added (Añadido)
    - **Fundación del proyecto:** Configuración inicial del repositorio, estructura de documentación (Overview, Motivation, Methodology, Datasets, Models, Experiments, Results, Roadmap).

    - **Pipeline de datos:** Implementación de `src/utils/download_data.py` para consultas a la API de UniProt (RuBisCO + arqueas) y `src/utils/data_enricher.py` para el cruce con TEMPURA y reglas de fallback jerárquicas (heurística de plantas, promedio por género).

    - **Extracción de embeddings:** `src/models/esm_extractor.py` (Mean Pooling para Baseline) y `src/models/esm_positional_extractor.py` (Embeddings posicionales por residuo con proyección MSA y almacenamiento HDF5).

    - **Modelos Baseline:** Pipeline de entrenamiento para modelos clásicos de ML (`src/models/train_predict.py`): Random Forest (optimizado con GridSearchCV), Gradient Boosting, SVR y Ensemble (Voting).

    - **CNN Posicional:** Implementación de la arquitectura 1D-CNN (clase `PositionalCNN`) y script de entrenamiento con early stopping (paciencia=15) en `5_Positional_training.ipynb`.

    - **Interpretabilidad:** Implementación de Grad-CAM para visualizar la importancia posicional en diferentes grupos térmicos.
    
    - **Herramientas auxiliares:** Utilidades de alineamiento de secuencias (`src/utils/alignment_utils.py` usando BioPython).

    ### 📈 Changed (Cambiado)
    - *Lanzamiento inicial – no hay cambios previos para comparar.*

    ### 🔧 Fixed (Corregido)
    - *Lanzamiento inicial – no hay errores previos que corregir.*

    ### 🗑️ Removed (Eliminado)
    - *Lanzamiento inicial – no se eliminaron características.*

