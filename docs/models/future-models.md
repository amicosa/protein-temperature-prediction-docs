# Future Directions and Proposed Improvements

Use the tabs below to switch between English and Spanish.

=== "English"

    Although the developed models (Baseline and positional CNN) have proven to be a solid proof of concept, there are many possible improvements. These proposals are not arbitrary, but directly arise from limitations observed in bias analyses and model performance.

    ## 🔬 Model and architecture improvements

    1. **Incorporating attention mechanisms (Transformer instead of CNN):**
    The current 1D-CNN is excellent at detecting local motifs and short-range patterns (a few adjacent amino acids). However, protein thermal stability often depends on **long-range interactions** (for example, how a residue in the N-terminus stabilizes a fold in the C-terminus). Replacing the CNN with a **Transformer Encoder** (such as ESM-2 itself) would allow learning these global dependencies, potentially improving accuracy in intermediate ranges (45-80 °C).

    2. **Multi-task learning:**
    The current model is trained exclusively to predict `T_opt`. A major improvement would be to train it to predict `T_min`, `T_opt`, and `T_max` simultaneously. This would not only leverage all available TEMPURA information, but also force the model to learn more complex relationships between sequence and the organism's **full thermal spectrum**, improving generalization.

    3. **Larger foundation models:**
    The current ESM-2 version used (`t6_8M`, 8M parameters) is the smallest in the family for computational efficiency. Future iterations should evaluate higher-capacity models such as `esm2_t36_3B_UR50D` (3 billion parameters). Although more expensive, these models capture much deeper and richer semantic representations of protein sequences.

    ## 🌿 Improvements in dataset preparation and embeddings

    4. **Integration of structural information (AlphaFold DB):**
    Since thermal stability is intrinsically determined by final three-dimensional structure (not only primary sequence), the dataset could be enriched with **structural embeddings** from AlphaFold DB. Adding folding and residue-contact information may significantly increase R² in thermophilic ranges, where sequence alone is more ambiguous.

    5. **Dataset expansion to cover the "missing middle":**
    The main limitation of current models is the scarcity of samples in the 45-80 °C range. A proposed strategy is to broaden UniProt queries to include greater bacterial phylum diversity and **explore complementary databases** to TEMPURA, such as **BacDive** or **NCBI BioSample**, which provide growth-temperature information for a wider spectrum of microorganisms.

    6. **Alignment and padding optimization:**
    Sequences are currently truncated to 1000 positions to control memory usage. If more resources become available, one option is to test **full-length alignment without truncation** or, alternatively, implement **dynamic padding with masking** (similar to Transformers), so the model does not need to "learn" to ignore zeros in truncated positions.

    ## 🧪 Experimental validation (Benchmarking)
    Every in silico prediction gains substantial credibility when contrasted with real experiments. In the medium term, an ideal step would be to establish collaboration with a biochemistry lab to:

    * **Select a set of proteins** whose optimal temperatures were predicted with high confidence (and others with low confidence).
    * **Validate them experimentally** using enzymatic activity assays across temperature ranges.

=== "Español"

    A pesar de que los modelos desarrollados (Baseline y CNN posicional) han demostrado ser una prueba de concepto sólida, existen numerosas vías de mejora. Estas propuestas no son arbitrarias, sino que surgen directamente de las limitaciones observadas en los análisis de sesgos y en el rendimiento de los modelos actuales.

    ## 🔬 Mejoras en el modelo y la arquitectura

    1. **Incorporación de mecanismos de atención (Transformer en lugar de CNN):**
    El modelo 1D-CNN actual es excelente detectando motivos locales y patrones de corto alcance (unos pocos aminoácidos adyacentes). Sin embargo, la estabilidad térmica de una proteína a menudo depende de **interacciones de largo alcance** (por ejemplo, cómo un residuo en el extremo N-terminal estabiliza un pliegue en el extremo C-terminal). Sustituir la CNN por un **Encoder Transformer** (como el propio ESM-2) permitiría al modelo aprender estas dependencias globales, mejorando potencialmente la precisión en los rangos intermedios (45-80 °C).

    2. **Aprendizaje multitarea (Multi-task Learning):**
    Actualmente, el modelo se entrena exclusivamente para predecir `T_opt`. Una mejora significativa sería entrenarlo para predecir simultáneamente `T_min`, `T_opt` y `T_max`. Esto no solo aprovecharía toda la información disponible en TEMPURA, sino que forzaría al modelo a aprender relaciones más complejas entre la secuencia y el **espectro térmico completo** del organismo, mejorando su generalización.

    3. **Modelos fundacionales más grandes:**
    La versión actual de ESM-2 utilizada (`t6_8M`, 8M parámetros) es la más pequeña de la familia por motivos de eficiencia computacional. Futuras iteraciones deberían evaluar modelos de mayor capacidad, como `esm2_t36_3B_UR50D` (3.000 millones de parámetros). Aunque más costosos, estos modelos capturan representaciones semánticas mucho más profundas y complejas de las secuencias proteicas.

    ## 🌿 Mejoras en la preparación del dataset y los embeddings

    4. **Integración de información estructural (AlphaFold DB):**
    Dado que la estabilidad térmica está intrínsecamente determinada por la estructura tridimensional final (no solo por la secuencia primaria), se propone enriquecer el dataset con **embeddings estructurales** procedentes de AlphaFold DB. La incorporación de información sobre plegamiento y contactos de residuos podría aumentar drásticamente el R² en los rangos termófilos, donde la secuencia es más ambigua.

    5. **Expansión del dataset para cubrir el "missing middle":**
    La principal limitación de los modelos actuales es la escasez de muestras en el rango de 45-80 °C. Se propone ampliar la búsqueda en UniProt incluyendo una mayor diversidad de filos bacterianos y **explorar bases de datos complementarias** a TEMPURA, como **BacDive** o **NCBI BioSample**, que ofrecen información térmica de crecimiento para un espectro más amplio de microorganismos.

    6. **Optimización del alineamiento y el padding:**
    Actualmente, las secuencias se recortan a 1000 posiciones para controlar el uso de memoria. Si en el futuro se dispone de recursos suficientes, se podría probar un **alineamiento sin recorte** o, alternativamente, implementar un esquema de **padding dinámico con masking** (similar al usado por los Transformers) para que el modelo no tenga que "aprender" a ignorar ceros en las posiciones recortadas.

    ## 🧪 Validación experimental (Benchmarking)
    Toda predicción in silico gana una credibilidad inmensurable cuando se contrasta con la realidad. A medio plazo, lo ideal es establecer una colaboración con un laboratorio de bioquímica para:

    * **Seleccionar un conjunto de proteínas** cuyas temperaturas óptimas hayan sido predichas por el modelo con alta confianza (y otras con baja confianza).
    * **Validarlas experimentalmente** mediante ensayos de actividad enzimática a diferentes temperaturas.