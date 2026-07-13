# 🧬 Protein Thermal Stability Prediction using ESM-2 Embeddings

Use the tabs below to switch between English and Spanish.

=== "English"

    Welcome to the documentation of the **Protein Embedding Design (PED)** project.

    This project explores the use of **protein language models (pLMs)** to predict key biophysical properties, with an initial focus on the **optimal growth temperature (T_opt)** of the **RuBisCO** enzyme (Ribulose-1,5-bisphosphate carboxylase/oxygenase).

    ---

    ## 🎯 Main objective

    Develop a reproducible pipeline that:

    1. **Extracts positional embeddings** from RuBisCO sequences using **ESM-2** (a state-of-the-art protein language model).
    2. **Projects** these embeddings onto multiple sequence alignments to preserve evolutionary and structural context.
    3. **Trains regression models** (Random Forest, Gradient Boosting, SVR, Ensemble) to predict T_opt from embeddings.
    4. **Provides a robust documentation base** for future extensions (diffusion, inverse design, etc.).

    ---

    ## 📊 Pipeline summary

    ```mermaid
    graph LR
        A[Download UniProt<br>RuBisCO + archaea] --> B[Cleaning and tokenization]
        B --> C[Enrichment with TEMPURA]
        
        C --> D[Branch 1: Vectorization<br>with ESMFeatureExtractor]
        C --> E[Branch 2: Multiple<br>Sequence Alignment]
        
        D --> F[Train Ensemble Models<br>(RF, GB, SVM)]
        E --> G[Train CNN on<br>Positional Embeddings]
        
        F --> H[Evaluation &<br>Comparison]
        G --> H
    ```


=== "Español"

    Bienvenido a la documentación del proyecto **Protein Embedding Design (PED)**.

    Este proyecto explora el uso de **modelos de lenguaje de proteínas (pLMs)** para predecir propiedades biofísicas clave, con un enfoque inicial en la **temperatura óptima de crecimiento (T_opt)** de la enzima **RuBisCO** (Ribulosa-1,5-bisfosfato carboxilasa/oxigenasa).

    ---

    ## 🎯 Objetivo principal

    Desarrollar un pipeline reproducible que:

    1. **Extraiga embeddings posicionales** de secuencias de RuBisCO utilizando **ESM-2** (un modelo de lenguaje de proteínas de vanguardia).
    2. **Proyecte** estos embeddings sobre alineamientos múltiples para preservar el contexto evolutivo y estructural.
    3. **Entrene modelos de regresión** (Random Forest, Gradient Boosting, SVR, Ensemble) para predecir la T_opt a partir de los embeddings.
    4. **Proporcione una base documental** robusta para futuras extensiones (difusión, diseño inverso, etc.).

    ---

    ## 📊 Pipeline resumido

    ```mermaid
    graph LR
        A[Descarga UniProt<br>RuBisCO + arqueas] --> B[Limpieza y tokenización]
        B --> C[Enriquecimiento con TEMPURA]
        
        C --> D[Branch 1: Vectorizaciónn<br>con ESMFeatureExtractor]
        C --> E[Branch 2: Alineamiento<br>Múltiple de Secuencias]
        
        D --> F[Entrenamiento y ensamblaje de modelos<br>(RF, GB, SVM)]
        E --> G[Entrenaiento de CNN con<br>Embeddings Posicionales]
        
        F --> H[Evaluación y<br>Comparación]
        G --> H
    ```
