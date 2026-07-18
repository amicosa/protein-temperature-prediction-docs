# Metrics (Comparative Analysis)

Use the tabs below to switch between English and Spanish.

=== "English"

    ## 🎯 Performance Comparison

    To provide a truly fair and definitive assessment, both the **Baseline Ensemble** and the **Positional 1D-CNN** were evaluated on the exact same **stratified test set** (20% of the total data, held out from the very beginning of the pipeline).

    The table below summarizes the key regression metrics:

    | Model | RMSE (°C) | R² | MAE (°C) |
    | :--- | :---: | :---: | :---: |
    | **Baseline Ensemble** <br> *(RF + GB + SVR)* | **6.80** | **0.953** | **4.16** |
    | **Positional 1D-CNN** <br> *(Evaluated on the same Test set)* | 7.36 | 0.942 | 4.54 |

    *(Note: The Baseline Ensemble was validated using 5-fold CV on the training set, while the Positional CNN used early stopping with a patience of 15 on a held-out validation set. Both models were finally scored on the same 338-sample unseen test set).*

    ---

    ## 📈 Analysis of the Quantitative Results

    At first glance, the classical Ensemble appears to slightly outperform the CNN on global metrics. However, this outcome provides **critical scientific insight** into the nature of the problem:

    - **Why does the Ensemble win globally?** The Baseline Ensemble relies on **mean-pooled embeddings**, which average out the entire sequence. This makes it inherently robust to data sparsity, as it only looks at the global "flavor" of the protein. 

    - **Why does the CNN struggle globally?** The Positional CNN learns **local motifs and specific residue patterns**. When it encounters sequences in the sparsely populated thermophilic range (the "Missing Middle"), it lacks sufficient examples to learn accurate local patterns, resulting in higher global errors compared to the averaged baseline.

    **The real value of the CNN is not in lowering the global RMSE, but in its capability for interpretability.** Because the CNN preserves positional context, we can apply techniques like **Grad-CAM** to visualize exactly which alignment positions are driving the model's predictions. This allows us to:

    - Identify specific residues or motifs that correlate with thermal stability.

    - Compare how attention maps differ between mesophilic and hyperthermophilic RuBisCO variants.

    *This positional insight will be explored in detail in the following `Error Analysis` and `Visualizations` sections.*

    ---

    ## ✅ Validation Robustness

    It is important to note that the metrics shown here are the result of a rigorous, reproducible validation protocol:
    
    - **For the Baseline:** The 5-fold cross-validation showed high stability, confirming that the 6.80 °C RMSE is not a lucky split.

    - **For the CNN:** The loss curve remained tightly coupled between training and validation. The early stopping criterion (patience=15) ensured the model was saved at its actual generalization peak (epoch 40), preventing any overfitting to the validation set.

    This gives us high confidence that the reported metrics are a true reflection of how each model would perform on new, unseen RuBisCO sequences.


=== "Español"

    ## 🎯 Comparativa final de rendimiento sobre el conjunto de test

    Para ofrecer una evaluación verdaderamente justa y definitiva, tanto el **Ensemble del Baseline** como la **CNN Posicional 1D** se evaluaron exactamente sobre el mismo **conjunto de test estratificado** (20 % de los datos totales, reservado desde el inicio del pipeline).

    La siguiente tabla resume las métricas de regresión clave:

    | Modelo | RMSE (°C) | R² | MAE (°C) |
    | :--- | :---: | :---: | :---: |
    | **Ensemble Baseline** <br> *(RF + GB + SVR)* | **6.80** | **0.953** | **4.16** |
    | **CNN Posicional 1D** <br> *(Evaluada sobre el mismo Test)* | 7.36 | 0.942 | 4.54 |

    *(Nota: El Ensemble del Baseline fue validado mediante 5-fold CV sobre el conjunto de entrenamiento, mientras que la CNN Posicional utilizó early stopping con paciencia 15 sobre un conjunto de validación. Ambos modelos fueron evaluados finalmente sobre el mismo conjunto de test de 338 muestras no vistas).*

    ---

    ## 📈 Análisis de los resultados cuantitativos

    A simple vista, el Ensemble clásico parece superar ligeramente a la CNN en métricas globales. Sin embargo, este resultado proporciona una **información científica crítica** sobre la naturaleza del problema:

    - **¿Por qué gana el Ensemble a nivel global?** El Ensemble del Baseline se basa en **embeddings promediados (mean-pooling)**, que promedian toda la secuencia. Esto lo hace inherentemente robusto a la escasez de datos, ya que solo mira el "sabor" global de la proteína.

    - **¿Por qué lucha la CNN a nivel global?** La CNN Posicional aprende **motivos locales y patrones de residuos específicos**. Cuando encuentra secuencias en el rango termófilo escasamente poblado (el "Missing Middle"), carece de ejemplos suficientes para aprender patrones locales precisos, lo que resulta en errores globales más altos en comparación con el baseline promediado.

    **El verdadero valor de la CNN no está en reducir el RMSE global, sino en su capacidad de interpretabilidad.** Debido a que la CNN preserva el contexto posicional, podemos aplicar técnicas como **Grad-CAM** para visualizar exactamente qué posiciones del alineamiento están impulsando las predicciones del modelo. Esto nos permite:

    - Identificar residuos o motivos específicos que correlacionan con la estabilidad térmica.

    - Comparar cómo difieren los mapas de atención entre variantes mesófilas e hipertermófilas de RuBisCO.

    *Esta visión posicional se explorará en detalle en las siguientes secciones de `Error Analysis` y `Visualizations`.*

    ---

    ## ✅ Robustez de la validación

    Es importante señalar que las métricas mostradas aquí son el resultado de un protocolo de validación riguroso y reproducible:
    
    - **Para el Baseline:** La validación cruzada de 5 folds mostró una alta estabilidad, confirmando que el RMSE de 6.80 °C no es fruto de una partición afortunada.
    
    - **Para la CNN:** La curva de pérdida se mantuvo estrechamente acoplada entre entrenamiento y validación. El criterio de early stopping (paciencia=15) aseguró que el modelo se guardara en su pico real de generalización (época 40), evitando cualquier sobreajuste al conjunto de validación.

    Esto nos otorga una alta confianza en que las métricas reportadas son un fiel reflejo de cómo se comportaría cada modelo ante nuevas secuencias de RuBisCO no vistas.