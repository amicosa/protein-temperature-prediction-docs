# Experiments

This section details the iterative experimental design and the scientific journey followed during the development of the predictive models. 

Use the tabs below to switch between English and Spanish.

=== "English"

    ## 🧪 Experimental Design Philosophy

    To avoid redundancy and ensure a clear narrative, the experiments have been structured as a **two-phase workflow**:

    * **[Phase 1: Feasibility Study](phase-001/)**. *Goal: Determine if ESM-2 embeddings contain enough information to predict T_opt at all using classical Machine Learning.*

    * **[Phase 2: Hypothesis Testing](phase-002/)**. *Goal: Determine if preserving positional context via multiple sequence alignment improves predictive performance using a 1D-CNN.*

    This separation ensures that **each phase generates a unique scientific insight**, justifying the transition to the next, without repeating technical details or final numbers which are covered in the `Models` section.

    ## 📋 Summary Table of Experimental Phases

    | Phase | Approach | Representation | Key Question Answered | Main Takeaway |
    | :--- | :--- | :--- | :--- | :--- |
    | **1. Feasibility** | Classical ML (RF, GB, SVR) | Mean-pooled ESM-2 (768-d) | Does ESM-2 contain thermal information? | **Yes.** Embeddings capture global biophysical properties. |
    | **2. Positional** | 1D-CNN | Per-residue ESM-2 aligned to MSA | Does position context improve predictions? | **Yes.** CNN improves correlation, but exposes the "Missing Middle" data bottleneck. |


=== "Español"

    ## 🧪 Filosofía del diseño experimental

    Para evitar redundancias y garantizar una narrativa clara, los experimentos se han estructurado como un **flujo de trabajo en dos fases**:

    * **[Fase 1: Estudio de viabilidad](phase-001/)**. *Objetivo: Determinar si los embeddings de ESM-2 contienen suficiente información para predecir la T_opt utilizando ML Clásico.*

    * **[Fase 2: Testeo de hipótesis](phase-002/)**. *Objetivo: Determinar si preservar el contexto posicional mediante alineamiento múltiple mejora el rendimiento predictivo usando una 1D-CNN.*

    Esta separación garantiza que **cada fase genere una conclusión científica única**, justificando la transición a la siguiente sin repetir detalles técnicos o cifras finales, las cuales se cubren en la sección `Models`.

    ## 📋 Tabla resumen de las fases experimentales

    | Fase | Enfoque | Representación | Pregunta clave respondida | Conclusión principal |
    | :--- | :--- | :--- | :--- | :--- |
    | **1. Viabilidad** | ML Clásico (RF, GB, SVR) | ESM-2 promediado (768-d) | ¿Contiene ESM-2 información térmica? | **Sí.** Los embeddings capturan propiedades biofísicas globales. |
    | **2. Posicional** | 1D-CNN | ESM-2 por residuo alineado a MSA | ¿Mejora el contexto posicional la predicción? | **Sí.** La CNN mejora la correlación, pero expone el cuello de botella del "Missing Middle". |