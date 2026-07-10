# Project Overview

Use the tabs below to switch between English and Spanish.

=== "English"

    ## 🌿 What is RuBisCO?

    **RuBisCO** (Ribulose-1,5-bisphosphate carboxylase/oxygenase) is a key enzyme in carbon fixation during photosynthesis. It catalyzes the reaction between CO2 and ribulose-1,5-bisphosphate (RuBP), producing two molecules of 3-phosphoglycerate.

    It is considered **the most abundant enzyme on Earth** and a high-priority target in crop engineering to improve photosynthetic efficiency.

    ### 🔬 RuBisCO challenges

    - **Low catalytic efficiency**: RuBisCO is relatively slow (kcat ~1-5 s^-1) compared with many other enzymes.
    - **Oxygenation reaction**: It competes with CO2, producing phosphoglycolate (a toxic metabolite) in a process called **photorespiration**.
    - **Variable thermostability**: RuBisCO T_opt varies widely across species:
      - **Plants**: 25-35 °C
      - **Mesophilic bacteria**: 30-40 °C
      - **Thermophilic archaea**: 60-100 °C

    ### 🧬 RuBisCO as a model system

    RuBisCO is ideal for this study because:

    1. **Phylogenetic diversity**: It spans all three domains of life (Bacteria, Archaea, Eukarya).
    2. **Data availability**: Curated sequences and experimental T_opt data (TEMPURA, BRENDA).
    3. **Biotechnological relevance**: Improving plant RuBisCO thermostability is a protein-engineering goal with strong agronomic impact.

    ---

    ## 🧠 Why protein embeddings?

    **Embeddings** are dense vector representations that capture semantic information from biological sequences. In the protein context:

    - **Protein language models (pLMs)** such as **ESM-2**, **ProtTrans**, or **SaProt** are trained on hundreds of millions of sequences.
    - They learn **hierarchical representations** encoding:
      - **Evolutionary information** (residue conservation).
      - **Physicochemical properties** (hydrophobicity, charge, size).
      - **Secondary and tertiary structure** (residue-residue contacts).

    ### Advantage of positional embeddings

    Instead of using pooled embeddings (the average over the whole sequence), we use **per-residue embeddings** projected onto a multiple sequence alignment. This enables:

    - **Preserving structural context**: homologous residues occupy the same alignment position.
    - **Training models with spatial information**: each position contributes independently to prediction.
    - **Interpretability**: we can identify which alignment positions are most relevant for T_opt.

    ---

    ## 🎯 Scientific objective

    **Predict the optimal temperature (T_opt) of RuBisCO from its amino acid sequence**, using ESM-2 embeddings as an intermediate representation.

    ### Research questions

    1. Do ESM-2 embeddings capture enough information to predict T_opt accurately?
    2. Which alignment positions are most relevant for thermostability?
    3. How do positional embeddings compare with pooled embeddings?
    4. Can this approach be generalized to other enzymes or properties (optimal pH, catalytic activity)?

    ---

    ## 🛠️ Methodological approach (summary)

    1. **Sequence download** from UniProt using a RuBisCO/archaea-focused query.
    2. **Enrichment** with thermal data from TEMPURA (plus hierarchical fallback rules).
    3. **Multiple sequence alignment** (Clustal Omega or MAFFT) to standardize positional length.
    4. **Positional embedding extraction** with ESM-2 projected onto the alignment.
    5. **Supervised regression model training** (Random Forest, Gradient Boosting, SVR, Ensemble).
    6. **Rigorous evaluation** with temperature-stratified cross-validation.

    ---

    ## 🧪 Expected results

    - A **robust T_opt predictor** for RuBisCO with error < 10 °C.
    - **Attention/importance maps** identifying key residues for thermostability.
    - A **reproducible pipeline** that can be extended to other enzymes.

    ---

    ## 📚 Key references

    - **ESM-2**: [Lin et al., 2023. "Evolutionary-scale prediction of atomic-level protein structure with a language model". Science.](https://www.science.org/doi/10.1126/science.ade2574)
    - **TEMPURA**: [Pinney et al., 2021. "TEMPURA: a database of growth temperatures for prokaryotes".](https://www.jstage.jst.go.jp/article/jsme2/35/3/35_ME20074/_article/-char/en)
    - **RuBisCO**: [Erb TJ, Zarzycki J., 2017. "A short history of RubisCO: the rise and fall (?) of Nature's predominant CO2 fixing enzyme".](https://pmc.ncbi.nlm.nih.gov/articles/PMC7610757/)

=== "Español"

    ## 🌿 ¿Qué es RuBisCO?

    La **RuBisCO** (Ribulosa-1,5-bisfosfato carboxilasa/oxigenasa) es una enzima clave en la fijación de carbono durante la fotosíntesis. Cataliza la reacción entre el CO2 y la ribulosa-1,5-bisfosfato (RuBP), produciendo dos moléculas de 3-fosfoglicerato.

    Es considerada **la enzima más abundante en la Tierra** y un objetivo prioritario en la ingeniería de cultivos para mejorar la eficiencia fotosintética.

    ### 🔬 Desafíos de RuBisCO

    - **Baja eficiencia catalítica**: RuBisCO es relativamente lenta (kcat ~1-5 s^-1) en comparación con otras enzimas.
    - **Reacción de oxigenación**: Compite con el CO2, produciendo fosfoglicolato (un metabolito tóxico) en un proceso llamado **fotorrespiración**.
    - **Termoestabilidad variable**: La T_opt de RuBisCO varía ampliamente entre especies:
      - **Plantas**: 25-35 °C
      - **Bacterias mesófilas**: 30-40 °C
      - **Arqueas termófilas**: 60-100 °C

    ### 🧬 RuBisCO como sistema modelo

    RuBisCO es ideal para este estudio porque:

    1. **Diversidad filogenética**: Abarca los tres dominios de la vida (Bacteria, Archaea, Eukarya).
    2. **Datos disponibles**: Secuencias curadas y datos experimentales de T_opt (TEMPURA, BRENDA).
    3. **Relevancia biotecnológica**: Mejorar la termoestabilidad de RuBisCO de plantas es un objetivo de ingeniería de proteínas con impacto agronómico.

    ---

    ## 🧠 ¿Por qué embeddings de proteínas?

    Los **embeddings** son representaciones vectoriales densas que capturan información semántica de las secuencias biológicas. En el contexto de proteínas:

    - **Modelos de lenguaje de proteínas (pLMs)** como **ESM-2**, **ProtTrans** o **SaProt** se entrenan con cientos de millones de secuencias.
    - Aprenden **representaciones jerárquicas** que codifican:
      - **Información evolutiva** (conservación de residuos).
      - **Propiedades físico-químicas** (hidrofobicidad, carga, tamaño).
      - **Estructura secundaria y terciaria** (contactos entre residuos).

    ### Ventaja de los embeddings posicionales

    En lugar de usar el embedding pooled (promedio de toda la secuencia), utilizamos **embeddings por residuo** proyectados sobre un alineamiento múltiple. Esto permite:

    - **Preservar el contexto estructural**: residuos homólogos ocupan la misma posición en el alineamiento.
    - **Entrenar modelos con información espacial**: cada posición contribuye independientemente a la predicción.
    - **Interpretabilidad**: podemos identificar qué posiciones del alineamiento son más relevantes para la T_opt.

    ---

    ## 🎯 Objetivo científico del proyecto

    **Predecir la temperatura óptima (T_opt) de RuBisCO a partir de su secuencia aminoacídica**, utilizando embeddings de ESM-2 como representación intermedia.

    ### Preguntas de investigación

    1. ¿Los embeddings de ESM-2 capturan información suficiente para predecir T_opt con precisión?
    2. ¿Qué posiciones del alineamiento son más relevantes para la termoestabilidad?
    3. ¿Cómo se comparan los embeddings posicionales vs. los embeddings pooled?
    4. ¿Es posible generalizar este enfoque a otras enzimas o propiedades (pH óptimo, actividad catalítica)?

    ---

    ## 🛠️ Enfoque metodológico (resumen)

    1. **Descarga de secuencias** desde UniProt con query especializada para RuBisCO y arqueas.
    2. **Enriquecimiento** con datos térmicos de la base de datos TEMPURA (y reglas de fallback jerárquicas).
    3. **Alineamiento múltiple** de secuencias (usando Clustal Omega o MAFFT) para homogeneizar longitudes.
    4. **Extracción de embeddings posicionales** con ESM-2, proyectados sobre el alineamiento.
    5. **Entrenamiento de modelos de regresión** supervisados (Random Forest, Gradient Boosting, SVR, Ensemble).
    6. **Evaluación rigurosa** con validación cruzada estratificada por temperatura.

    ---

    ## 🧪 Resultados esperados

    - Un **predictor robusto** de T_opt para RuBisCO con error < 10 °C.
    - **Mapas de atención/importancia** que identifiquen residuos clave para la termoestabilidad.
    - Un **pipeline reproducible** que pueda extenderse a otras enzimas.

    ---

    ## 📚 Referencias clave

    - **ESM-2**: [Lin et al., 2023. "Evolutionary-scale prediction of atomic-level protein structure with a language model". Science.](https://www.science.org/doi/10.1126/science.ade2574)
    - **TEMPURA**: [Pinney et al., 2021. "TEMPURA: a database of growth temperatures for prokaryotes".](https://www.jstage.jst.go.jp/article/jsme2/35/3/35_ME20074/_article/-char/en)
    - **RuBisCO**: [Erb TJ, Zarzycki J., 2017. "A short history of RubisCO: the rise and fall (?) of Nature's predominant CO2 fixing enzyme".](https://pmc.ncbi.nlm.nih.gov/articles/PMC7610757/)
