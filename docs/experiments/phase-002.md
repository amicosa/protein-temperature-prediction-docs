# Phase 002: Hypothesis Testing (Positional 1D-CNN)

Use the tabs below to switch between English and Spanish.

=== "English"

    ## 🧬 Phase 2: Hypothesis Testing (Positional 1D-CNN)

    **Objective:**

    After validating the baseline, we formulated the central hypothesis of the project: *"Preserving the position of residues via multiple sequence alignment allows a deep learning model to learn local structural motifs, resulting in a more accurate T_opt prediction than mean pooling."*

    **Setup & Methodology:**

    * **Representation:** We used `ESMPositionalExtractor` to extract **per-residue embeddings** (768 dimensions per token). Crucially, these were projected onto a **fixed Multiple Sequence Alignment (MSA)**, where gap positions were filled with zero-vectors. (The alignment was truncated to 1000 positions to manage memory).

    * **Model:** A **1D Convolutional Neural Network (CNN)** was designed to learn local patterns within this positional matrix.

    * **Validation:** The model was trained with early stopping (`patience=15`) on an 80/20 train/validation split, using the `MSELoss` and an Adam optimizer.

    **Key Findings & Lessons Learned:**

    * **Positional context is key:** The CNN significantly improved the correlation between predictions and real values, especially in well-sampled regions (Mesophiles and Hyperthermophiles).

    * **The "Missing Middle" is confirmed:** The scatter plot revealed a clear dispersion in the 45-80 °C range. This demonstrates that **the primary bottleneck is not the model's architecture, but the severe data scarcity in the thermophilic range** (the "missing middle" discussed in the `Datasets` section).

    * **No overfitting:** The loss curve showed a rapid and stable convergence, with train and validation losses remaining close, indicating the model generalized well to unseen sequences.

    **Conclusion of Phase 2:** The CNN confirmed the hypothesis that positional information is crucial. However, it also highlighted that no amount of architectural sophistication can compensate for a lack of data in a specific thermal niche.


=== "Español"

    ## 🧬 Fase 2: Testeo de hipótesis (CNN Posicional 1D)

    **Objetivo:**

    Tras validar el baseline, formulamos la hipótesis central del proyecto: *"Preservar la posición de los residuos mediante alineamiento múltiple permite a un modelo de deep learning aprender motivos estructurales locales, lo que resulta en una predicción más precisa de la T_opt que el mean pooling."*

    **Configuración y metodología:**

    * **Representación:** Se utilizó `ESMPositionalExtractor` para extraer **embeddings por residuo** (768 dimensiones por token). De forma crucial, estos se proyectaron sobre un **Alineamiento Múltiple (MSA)** fijo, donde las posiciones con gaps se rellenaron con vectores de ceros. (El alineamiento se truncó a 1000 posiciones para gestionar la memoria).

    * **Modelo:** Se diseñó una **Red Neuronal Convolucional 1D (CNN)** para aprender patrones locales dentro de esta matriz posicional.

    * **Validación:** El modelo se entrenó con parada temprana (*early stopping*, `patience=15`) en una partición 80/20 de entrenamiento/validación, utilizando `MSELoss` y un optimizador Adam.

    **Hallazgos clave y lecciones aprendidas:**

    * **El contexto posicional es clave:** La CNN mejoró significativamente la correlación entre predicciones y valores reales, especialmente en las regiones bien muestreadas (mesófilos e hipertermófilos).

    * **El "missing middle" se confirma:** El gráfico de dispersión reveló una dispersión clara en el rango de 45-80 °C. Esto demuestra que **el principal cuello de botella no es la arquitectura del modelo, sino la escasez de datos en el rango termófilo** (el "missing middle" analizado en la sección `Datasets`).

    * **Sin sobreajuste:** La curva de pérdida mostró una convergencia rápida y estable, permaneciendo las pérdidas de entrenamiento y validación cercanas, lo que indica que el modelo generalizó bien a secuencias no vistas.

    **Conclusión de la Fase 2:** La CNN confirmó la hipótesis de que la información posicional es crucial. Sin embargo, también puso de manifiesto que ninguna sofisticación arquitectónica puede compensar la falta de datos en un nicho térmico específico.
