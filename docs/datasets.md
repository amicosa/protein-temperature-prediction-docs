# Datasets

Use the tabs below to switch between English and Spanish.

=== "English"

    ## Overview

    Open access to high-quality scientific data has transformed how research is developed. Thanks to initiatives such as UniProt and other public databases, independent researchers, students, and small teams can now build high-impact projects without relying exclusively on the resources of large institutions or companies. Open science not only improves reproducibility and transparency, but also democratizes access to knowledge and accelerates innovation by allowing anyone to build on prior community work.

    This project is built on that philosophy. Its goal is to explore how far we can predict physicochemical properties of **RuBisCO** using only information contained in its primary sequence and associated biological metadata. While the first use case focuses on predicting optimal activity temperature, the project architecture is designed to be extended to other biotechnologically relevant properties.

    To do this, we use two main information sources. On one hand, RuBisCO protein sequences provide the primary signal about potential protein structure and function. On the other hand, biological metadata, especially taxonomic information and experimental measurements, allow each sequence to be linked to the target variable, in this case **optimal activity temperature**.

    At present, the dataset is built by integrating information from **UniProt** and **TEMPURA**. However, the project is designed as an evolving platform, so future versions may incorporate additional sources (for example, functional annotations, structural data, or specialized databases) to expand predictive capabilities and improve model performance.

    ## Data sources

    ### UniProt

    **UniProt** is one of the most complete and widely used protein databases in the scientific community. It provides curated protein sequences and high-quality biological annotations, making it a reference source for bioinformatics, molecular biology, and machine learning studies on proteins.

    In this project, UniProt is the main data source. From it, we retrieve **RuBisCO** sequences together with metadata needed to uniquely identify each protein and link it to its source organism.

    The currently used fields are:

    | Field | Description |
    |:------|:------------|
    | Entry | Unique UniProt identifier for each protein. |
    | Protein names | Annotated protein name. |
    | Organism | Source organism of the sequence. |
    | Taxonomic lineage | Full taxonomic classification of the organism. |
    | Temperature dependence | Experimental annotations related to temperature dependence, when available. |
    | Sequence | Primary amino acid sequence used as model input. |

    Although all these fields are stored, the most relevant information in this first stage is the **protein sequence**, the **organism**, and its **taxonomic lineage**, because these fields enable downstream linkage with experimental information from other data sources.

    ### TEMPURA

    **TEMPURA** (Database of Growth TEMPERatures of Usual and Rare Prokaryotes) is a database that compiles experimentally measured growth temperatures for microorganisms, mainly bacteria and archaea.

    While UniProt provides protein-level information, TEMPURA contributes complementary data that is essential for this project: the thermal conditions under which source organisms live.

    Using shared taxonomic information between both databases, RuBisCO sequences from UniProt can be enriched with experimentally reported organism growth temperatures. In the current phase, this temperature is used as a proxy for the protein optimal operating temperature and serves as the target variable for prediction models.

    In addition to mean temperature, the integration process also retains minimum and maximum temperatures for each organism. Although these values are not yet used for model training, they are stored to support future research and new predictive strategies.

    ### Future integrations

    A long-term objective is to progressively enrich the dataset by incorporating new biological information sources.

    One potential integration is **AlphaFold DB**, which would complement primary sequences with AI-predicted 3D structural information. Structural features could improve predictive performance for certain models and enable new research lines centered on sequence-structure-function relationships.

    However, incorporating this type of information involves major computational and methodological challenges. For now, the project focuses exclusively on features derived from primary sequence and associated biological metadata.

    As the project evolves, we will also evaluate other specialized databases to add functional, structural, or evolutionary annotations and broaden the scope of developed models.

    ## Why these datasets?

    The choice of **UniProt** and **TEMPURA** is based on scientific criteria as well as reproducibility and accessibility principles.

    From a technical perspective, both databases provide highly complementary information. UniProt offers carefully curated protein sequences and broad biological annotation, while TEMPURA adds experimental thermal-growth data for bacteria and archaea. Integrating both allows building a consistent dataset to study relationships between RuBisCO primary sequence and functional properties.

    At the same time, these databases are a clear example of the value of open science. Their public availability allows researchers, students, and developers worldwide to access high-quality data without depending on exclusive infrastructure or organizational resources. This accessibility improves reproducibility, enables independent validation, and lowers barriers for new research initiatives.

    This project is largely possible thanks to that collective effort to share scientific data openly. In the same spirit, the methodology developed here is designed to be transparent and reproducible so that any interested person can understand it, replicate it, and use it as a starting point for new work.

    ## Dataset construction

    The dataset used in this project does not come from a single source. It is built through an integration and enrichment process across multiple public databases. The objective is to obtain a consistent dataset that combines UniProt sequence information with TEMPURA experimental temperature data, followed by cleaning, normalization, and preprocessing steps suitable for machine-learning models.

    ```mermaid
    flowchart TD

    A[UniProt REST API] --> B[Protein sequence\ndownload]

    B --> C[Sequence cleaning]
    C --> D[Sequence tokenization]

    D --> E[Merge with TEMPURA]

    E --> F[Species matching]

    F --> G{Temperature available?}

    G -->|Exact match| H[Assign experimental temperature]

    G -->|No| I[Taxonomic fallback rules]

    I --> J[Plant heuristic]
    I --> K[Genus mean]

    H --> L[Enriched dataset]
    J --> L
    K --> L

    L --> M[Sequence alignment]

    M --> N[Final dataset]
    ```

    ## Preprocessing

    After downloading data from different sources, we apply a preprocessing pipeline to build a consistent, reproducible dataset suitable for machine-learning training. This process includes the following stages:

    ### 1. Sequence download

    The first phase consists of automatic retrieval of RuBisCO sequences from the UniProt REST API.

    - A specific RuBisCO search is executed using both gene name (`rbcL`) and protein annotations, expanding the query to include archaeal variants, especially thermophilic and hyperthermophilic species.
    - Download is performed with pagination to retrieve large volumes efficiently while respecting API constraints.
    - After download, duplicate records are removed by UniProt unique identifier (`Entry`), keeping one entry per protein.

    ### 2. Sequence cleaning and preparation

    Before feeding sequences into models, several transformations are applied to ensure quality and homogeneity.

    - Entries without a valid protein sequence are removed to avoid incomplete samples.
    - Sequences are normalized by removing spaces, line breaks, and formatting characters introduced during retrieval.
    - Each amino acid is then separated by spaces, generating a tokenized representation compatible with transformer-based models such as ESM-2.

    ### 3. Dataset enrichment

    UniProt sequences are complemented with TEMPURA experimental information to provide the target variable used by models.

    - Scientific organism names are normalized beforehand to facilitate cross-database matching and minimize formatting discrepancies.
    - When an exact species match exists, minimum, optimum, and maximum temperatures from TEMPURA are assigned.
    - Although only optimum temperature is currently used as target variable, minimum and maximum temperatures are also stored to support future extensions.

    ### 4. Hierarchical assignment strategy

    Not all UniProt species have experimental data in TEMPURA. To increase coverage without arbitrary assignments, a biologically informed hierarchical strategy is used.

    - Priority is always exact-species assignment when available.
    - For organisms in Viridiplantae or Streptophyta (where TEMPURA coverage is limited), a conservative thermal range based on common photosynthetic plant conditions is assigned to keep these sequences in the dataset until better experimental data are available.
    - When a species is missing in TEMPURA, genus-level mean temperatures are used as a fallback.
    - If none of these strategies can assign temperature with sufficient confidence, the sequence remains unlabeled and is excluded from supervised training.

    ### 5. Sequence alignment

    As a final preprocessing stage, sequences can be aligned against a common reference before being used in certain models.

    - Alignment uses pairwise algorithms with BLOSUM62 and configurable gap opening/extension penalties.
    - After alignment, sequences have uniform length, facilitating position-wise comparison and positional feature extraction.
    - The implementation also supports optional external MSA tools (Clustal Omega, MAFFT), allowing experimentation without changing the rest of the pipeline.

    ## Dataset statistics

    | Metric | Value |
    |:-----------------------------------|--------:|
    | Number of proteins | 2000 |
    | Number of organisms | 1184 |
    | Number of genera | 708 |
    | Number of taxonomic domains | 4 |
    | Mean sequence length (aa) | 448.8 |
    | Median sequence length (aa) | 461 |
    | Minimum length (aa) | 16 |
    | Maximum length (aa) | 5902 |
    | Mean optimal temperature (°C) | 52.9 |
    | Minimum optimal temperature (°C) | 20 |
    | Maximum optimal temperature (°C) | 105 |

    The table above summarizes the main characteristics of the dataset used in this first project stage. After integration and preprocessing, we obtained **2,000 RuBisCO sequences** from **1,184 organisms**, distributed across **708 genera** and **four taxonomic domains**, providing broad representation of evolutionary diversity in this enzyme family.

    Sequences show a **mean length of 448.8 amino acids**, very close to the median (461), suggesting a relatively homogeneous distribution for most of the dataset. Still, extreme values are present, with sequences substantially shorter or longer than typical RuBisCO length. Their influence is analyzed in later sections via distribution analysis and outlier detection.

    For the target variable, **optimal temperatures** span a broad experimental range, **from 20 °C to 105 °C with a mean of 52.9 °C**. This variability is especially relevant for machine learning because it enables training on proteins adapted to very different environmental conditions, including mesophilic, thermophilic, and hyperthermophilic organisms.

    ### Sequence statistics

    | Statistic | Value |
    |:---------------------|--------:|
    | Number of sequences | 2000 |
    | Mean | 448.8 |
    | Median | 461 |
    | Standard deviation | 280.02 |
    | Minimum | 16 |
    | 25th percentile | 376.75 |
    | 75th percentile | 476 |
    | 95th percentile | 743.25 |
    | Maximum | 5902 |

    Sequence length is highly relevant both biologically and computationally. Differences in amino acid count can reflect different RuBisCO forms, fusion proteins, partial sequences, or annotation anomalies, and directly affect preprocessing and representation strategies used by ML models.

    The 2,000 analyzed sequences have mean length 448.8 aa and median 461 aa, very close values indicating that most samples concentrate around expected RuBisCO length. In addition, 50% of sequences lie between 376.8 and 476 aa, reflecting strong homogeneity in the main distribution.

    ![Sequence length histogram](assets/figures/datasets/sequence_length_distribution.png)

    However, the standard deviation (280 aa) and broad observed range (16 to 5,902 aa) show a small set of extreme values. The full histogram indicates that the distribution is heavily influenced by a few exceptionally long sequences, while the boxplot confirms many outliers, especially in the upper tail.

    ![Sequence length boxplot](assets/figures/datasets/sequence_length_boxplot.png)

    To provide a more representative visualization, we also show a histogram limited to the 99th percentile. This view removes the effect of the most extreme sequences and makes it clear that the vast majority of proteins are shorter than about 1,300 aa, mainly concentrated around 450 aa.

    ![Truncated sequence length histogram](assets/figures/datasets/sequence_length_distribution_cutoff.png)

    Although these atypical sequences are not automatically removed, identifying them is essential for future phases. They will be individually analyzed to determine whether they correspond to fusion proteins, incomplete annotations, biologically meaningful variants, or potential annotation errors in source databases.

    ### Temperature statistics

    Optimal operating temperature is the target variable in this first project phase. Using taxonomic information from TEMPURA, it was possible to assign an experimental optimal temperature value to **1,686** out of **2,000** collected sequences, which form the supervised training/evaluation dataset.

    | Statistic | Value |
    |:------------------------|--------:|
    | Number of observations | 1686 |
    | Mean | 52.93 |
    | Median | 28.2 |
    | Standard deviation | 30.77 |
    | Minimum | 20 |
    | 25th percentile | 25 |
    | 75th percentile | 85 |
    | 95th percentile | 100 |
    | Maximum | 105 |

    Values span a **wide range**, from **20 °C** to **105 °C**, reflecting the diversity of ecological niches of RuBisCO-containing organisms. The median is **28.2 °C**, while the mean rises to **52.9 °C**, a strong difference indicating a non-symmetric distribution strongly influenced by high-temperature-adapted organisms.

    ![Global optimal temperature histogram](assets/figures/datasets/topt_distribution.png)

    The histogram confirms multiple clear peaks instead of an approximately normal distribution. There is a strong concentration around 25 °C (mostly mesophiles), together with another large group around 80-90 °C (thermophilic and hyperthermophilic archaea). Smaller groups appear around 60 °C, 70 °C, and close to 100 °C, reflecting diverse adaptation strategies in nature.

    ![Optimal temperature boxplot](assets/figures/datasets/topt_boxplot.png)

    The boxplot complements this by showing broad dispersion. Unlike sequence length, no especially extreme outliers are observed; the broad thermal range itself is part of the biological variability.

    | Thermal_Group | count |
    |:----------------|------:|
    | Hyperthermophiles | 959 |
    | Mesophiles | 916 |
    | Thermophiles | 125 |

    A more intuitive interpretation groups proteins by thermal class of their source organism. The dataset is dominated by **hyperthermophiles (959 proteins)** and **mesophiles (916 proteins)**, while **thermophiles (125 proteins)** represent a much smaller fraction.

    In this work, thermal groups are defined by organism **optimal temperature (Topt)**: mesophiles **(<45 °C)**, thermophiles **(45-80 °C)**, and hyperthermophiles **(>=80 °C)**. Since the dataset has no organisms below 20 °C, psychrophiles are absent.

    This explains the multimodal structure in optimal temperatures and shows imbalance among thermal groups. Organisms adapted to very high and moderate temperatures are strongly represented, while thermophile-derived proteins are underrepresented. This must be considered during training/evaluation, as it can affect generalization in low-sample temperature regions.

    This imbalance implies that the model may predict temperatures near 25 °C and 85 °C well, but will likely **struggle to generalize in the intermediate range (45-80 °C)**. Predictions in this range should be interpreted with caution, and techniques such as resampling, class weighting, or stratified cross-validation should be considered.

    ![Optimal temperature distribution by domain](assets/figures/datasets/topt_violin_by_domain.png)

    Finally, separating observations by taxonomic domain shows that temperature distribution is tightly linked to phylogeny. Archaea concentrate most high temperatures, eukaryotic sequences cluster around 25 °C, and bacteria span a wider intermediate range from psychrophiles to moderate thermophiles.

    This strong taxonomy-temperature relationship confirms the importance of preserving phylogenetic information during preprocessing, since it is biologically meaningful for interpreting protein function and may provide useful context for future models.

    ### Taxonomic distribution

    Taxonomic composition is critical to understand model scope and limitations. Domain-level and genus-level analysis reveals a sampling structure strongly biased toward organisms with extreme adaptation strategies.

    ![Domain distribution](assets/figures/datasets/taxonomic_domain_distribution.png)

    Domain breakdown shows that **Eukaryota (892 proteins, 44.6%)** and **Archaea (771 proteins, 38.5%)** are the dominant groups and are represented in near-balanced proportions. Despite the ecological relevance and diversity of **Bacteria**, this domain contributes only **336 proteins (16.8%)** and, together with a single record classified as Other, completes the 2,000-sequence dataset.

    This becomes especially relevant when contrasted with the thermal distribution. The large number of Eukaryota sequences (mainly plants and algae) explains the peak around 25 °C, while abundant Archaea explains the second peak near 85 °C. Bacteria, though less numerous, act as a thermal bridge in the intermediate range, but with lower representation.

    ![Top 10 genera](assets/figures/datasets/top_10_genera.png)

    Analysis of the **10 most frequent genera** reinforces this pattern. The dataset is dominated by **hyperthermophilic and thermophilic archaeal** genera such as *Pyrococcus* (~190 sequences), *Methanocaldococcus* (~155), *Saccharolobus* (~135), and *Thermococcus* (~110), among others. This overrepresentation is a direct consequence of merging with TEMPURA, which historically has high coverage of microorganisms from geothermal environments.

    *Lupinus* and *Nostoc* are the only non-archaeal representatives in the top 10, likely due to high annotation frequency in UniProt or targeted sequencing efforts. *Nostoc* adds important evolutionary diversity as a **cyanobacterial** genus with nitrogen fixation and photosynthesis capabilities.

    This taxonomic distribution has direct consequences for ML performance. The strong bias toward thermal extremes and the underrepresentation of intermediate thermophiles (only 125 sequences) suggest reduced model generalization in the **45-80 °C** range. Even with broad dataset coverage at low/high ranges, predictions for moderate thermophiles must be interpreted cautiously.

    ### Missing-value analysis

    A critical requirement for supervised learning is availability of the target variable. After merging with TEMPURA, **314 of 2,000 sequences (15.7%)** could not be labeled with optimal temperature. Most of these records correspond to **eukaryotic organisms** (mainly plants and algae) and **non-thermophilic bacteria**.

    This lack of coverage is an inherent limitation of TEMPURA, which is specialized in prokaryotes from extreme environments. Consequently, the final training dataset (1,686 samples) is slightly overrepresented by prokaryotic organisms, which should be considered when interpreting results.

    ### Relationship between sequence length and temperature

    An additional exploratory analysis evaluates whether there is any relationship between RuBisCO protein length and the organism optimal temperature.

    ![Length vs. Optimal Temperature scatter plot](assets/figures/datasets/length_vs_topt_scatter.png)

    The scatter plot reveals interesting patterns. First, there is a **high point density between 400 and 500 amino acids**, consistent across the full temperature spectrum. This indicates that canonical RuBisCO length is preserved regardless of thermal niche. Vertical "columns" along the temperature axis are an artifact of discrete TEMPURA temperature values.

    A key finding is the **asymmetry toward longer lengths**. Mesophilic organisms (blue, T_opt < 45 °C) rarely exceed 500 aa, whereas hyperthermophiles (red, T_opt >= 80 °C) show a **long outlier tail**, with multiple sequences above 1,000 and even 1,800 aa. This suggests that fusion proteins or extended structural variants are almost exclusive to archaea adapted to extreme environments.

    ### Length distribution by taxonomic domain

    When sequence length is broken down by domain, substantial differences appear.

    ![Length by Domain boxplot](assets/figures/datasets/length_by_domain.png)

    The boxplot (Y-axis limited to 1,500 aa for central-box visibility) shows clear contrasts:

    - **Eukaryota:** Most homogeneous distribution. Very narrow box and median close to **450 aa**. Only a few outliers exceed 600 aa. This reflects high evolutionary stability of chloroplast RuBisCO in plants and algae.
    - **Bacteria:** Similar behavior to eukaryotes, with a tight box and median around **470 aa**. A few low-end outliers likely correspond to fragments or incomplete annotations.
    - **Archaea:** Main source of variability in the dataset. Median is lower, around **380 aa**, but interquartile range is much wider. It also shows a **very high number of upper outliers**, extending beyond 1,400 aa (and up to 5,902 aa globally). This extreme variability is consistent with fusion proteins and additional domains in thermophilic/hyperthermophilic archaea.

    Overall, the scatter plot and boxplot confirm that although most sequences follow a canonical ~450 aa pattern, the **bias toward hyperthermophilic archaea** introduces most of the variance and outliers in sequence-length distribution.

    ## Train / Validation / Test split

    Proper dataset partitioning is critical to guarantee objective evaluation and avoid overfitting. In this project, two split strategies were implemented, adapted to each model architecture.

    ### Split proportions and random seed

    For both ensemble-based models and the convolutional neural network (CNN), an **80/20** train/evaluation split was used.

    To guarantee reproducibility, a shared random seed (`random_state = 42`) was fixed across all partitioning functions. This ensures that rerunning the training pipeline yields identical subsets.

    ### Stratification

    Due to the **strong thermal-group imbalance** identified in the data-analysis section (especially underrepresentation of thermophiles), a simple random split could produce unbalanced test sets. To mitigate this, a custom stratification strategy was implemented in `stratified_split_simple` from `train_predict.py`.

    This strategy consists of:

    1. **Thermal-bin grouping:** Optimal temperatures are split into five bins: `0-20°C`, `20-40°C`, `40-60°C`, `60-80°C`, and `80-100°C`.
    2. **Proportional split by bin:** Within each bin, samples are randomly shuffled and a fixed fraction (`test_size = 0.2`) is sent to test, ensuring all thermal categories are represented in the same proportion as in the original dataset.
    3. **Safety fallback:** If any bin contains fewer than 2 samples (making stratification impossible), the system automatically falls back to simple random splitting.

    For the **positional CNN** (`5_Positional_training.ipynb`), this stratification was not applied. Instead, standard scikit-learn `train_test_split` random partitioning was used because the main objective was monitoring loss curves for early stopping.

    ### Data leakage prevention

    **Data leakage** is one of the most common modeling errors and occurs when test information leaks into training. To prevent it, the pipeline applies the following practices:

    - **Split before training:** Data are partitioned into disjoint sets before training starts, ensuring that no sequence or embedding used to fit model parameters is seen during evaluation.
    - **Feature scaling (SVR):** For SVR, `StandardScaler` is fitted only on training data (`X_train`). The same transformation is then applied to `X_test` via `.transform()`, preventing normalization parameters from being influenced by evaluation data.
    - **Alignment independence:** For positional models, sequence alignment is performed on the full dataset. However, because this is **pairwise alignment** against a fixed reference (not progressive MSA dependent on other sequences), each sequence positional information is independent from others. Therefore, alignment does not introduce train-test leakage.

    ### Internal cross-validation

    For classical models (Random Forest, Gradient Boosting, SVR, and Ensemble), the training set (80% of data) was additionally evaluated with 5-fold cross-validation (5-CV). This provides a more robust development-time estimate before final evaluation on the fully isolated 20% test split.

    ## Biases and limitations

    Despite efforts to build a broad and representative dataset, integration of UniProt and TEMPURA, together with publication patterns in the literature, introduces systematic biases that must be acknowledged before interpreting model performance. These biases do not invalidate the study, but they constrain conclusions and define where generalization is more or less reliable.

    ### Taxonomic bias (domain and genus)

    Taxonomic composition analysis revealed domain distribution that diverges from real biological diversity.

    Overrepresentation of Eukaryota and Archaea: Plant/algal sequences (Eukaryota, 892 proteins) and thermophilic archaeal sequences (Archaea, 771 proteins) together account for 83% of the dataset.

    Underrepresentation of Bacteria: Although bacteria represent much of prokaryotic diversity and carbon fixation in many ecosystems, they contribute only 336 proteins (16.8%).

    At genus level, bias is even more extreme. The dataset is dominated by hyperthermophilic archaeal genera (*Pyrococcus*, *Methanocaldococcus*, *Saccharolobus*, *Thermococcus*). *Lupinus* (plants) and *Nostoc* (cyanobacteria) are the only non-archaeal genera in the top 10.

    Implication: Models are expected to generalize better for plants and hyperthermophilic archaea than for mesophilic or moderately thermophilic bacteria.

    ### Thermal bias (the "missing middle")

    Taxonomic imbalance translates into strong target-variable imbalance. Optimal temperature distribution is clearly bimodal, with two dominant peaks:

    Mesophiles (<45 °C): 916 proteins.

    Hyperthermophiles (>=80 °C): 959 proteins.

    Thermophiles (45-80 °C): only 125 proteins.

    This underrepresentation in the intermediate range has major ML implications. Even if MSE averages errors globally, models tend to optimize where data are abundant (low and high ranges), sacrificing performance in the middle range.

    ### Sampling and publication bias

    Data origin, mostly curated literature in UniProt and TEMPURA measurements, introduces unavoidable publication bias.

    - **Classical biological models:** Eukaryota is overrepresented because plant RuBisCO (e.g., *Arabidopsis*, *Spinacia*) is heavily studied in plant physiology and biochemistry.
    - **Biotechnological interest:** Hyperthermophilic archaea (e.g., *Pyrococcus*, *Thermococcus*) are popular due to thermostable enzymes, explaining their high representation.
    - **Lower attention to mesophilic/thermophilic bacteria:** Many environmentally relevant bacteria are less sequenced/studied in protein thermostability context, explaining underrepresentation.

    This means results are especially relevant for model organisms and biotechnology-focused taxa, but may not extrapolate equally well to less-characterized bacterial diversity.

    ### Missing-data bias

    As noted above, **314 of 2,000 sequences (15.7%)** could not be assigned optimal temperature after TEMPURA merge. Most correspond to **eukaryotes** (mainly non-model algae/plants) and **non-thermophilic bacteria**.

    This has a double impact:

    1. It **reduces sample size** in an already limited intermediate temperature range.
    2. It **reinforces taxonomic bias**, since missing labels are concentrated in already underrepresented groups (Bacteria and non-model Eukaryota), increasing hyperthermophilic archaeal dominance in the final training data.

    ### Implications for model performance

    The combination of these biases has direct consequences for expected performance of trained models (Random Forest, Ensemble, and positional CNN):

    - **High precision at extremes:** Models should predict temperatures around 25 °C (plants) and 85-100 °C (hyperthermophilic archaea) with high accuracy because these regions are well sampled.
    - **Lower reliability in middle range (45-80 °C):** Predictions in thermophilic range should be interpreted with caution because there are too few examples to learn specific structure-function relationships.
    - **Taxonomic shortcut risk:** Models may infer temperature from phylogenetic patterns (e.g., archaea -> high temperature, plants -> low temperature) instead of intrinsic sequence physicochemical features. This shortcut-learning risk should be assessed with taxon-held-out validation.

    **Limitations conclusion:** This dataset is an excellent resource to explore sequence-thermal stability relationships in **model organisms and extremophiles**, but it is not fully representative of RuBisCO diversity in nature. Model conclusions should be treated as **robust biological hypotheses within these ecological niches**, not universal rules for all organisms.

    ## Licensing

    Respecting data-source terms of use and intellectual property is a core project principle. While the goal is a reproducible workflow, special care is taken to avoid violating source-database licensing.

    ### UniProt

    **UniProt** is distributed under **Creative Commons Attribution-NoDerivatives (CC BY-ND 4.0)**. This allows users to use, download, and redistribute data as long as source attribution is preserved and original annotations are not modified.

    In this project, data are retrieved only through the public UniProt REST API, strictly following terms of service and usage policies. All downloaded sequences are used solely for non-commercial research.

    ### TEMPURA

    **TEMPURA** (Database of Growth TEMPERatures of Usual and Rare Prokaryotes) is subject to a **specific academic license**. Although research use is generally allowed, **direct redistribution of the original dataset is restricted** by terms of use.

    ### Redistribution and data-access policy

    To ensure compliance with these licenses, this project follows these practices:

    - **No redistribution of original data:** The project repository **does not include and has never included original TEMPURA datasets or bulk UniProt dumps**.
    - **Script-based workflow:** Dataset construction is performed through scripts that query public APIs. Any researcher can reproduce the study from the same data sources while preserving traceability and avoiding redistribution of copyrighted materials.
    - **Method documentation:** Clear instructions are provided for environment setup and running download/preprocessing steps.

    ### Source code and documentation

    As part of the open-science strategy, generated **documentation** (including this descriptive file, dataset statistics, and preprocessing methodology) is shared publicly to support transparency and reproducibility.

    However, the **internal source code** for download/processing pipeline is private, as defined in initial project goals. The download scripts are documented conceptually here so other researchers can understand the process without direct access to proprietary implementation details. This balances intellectual-property protection with the commitment to share knowledge and methodology.

    ## Reproducibility

    Although the source code for integration and processing pipeline is private, the project is designed with a strong commitment to **methodological reproducibility**. Any researcher or student with intermediate Python/bioinformatics skills can replicate the presented results by following the protocol below.

    To guarantee reproducibility, best practices were applied for version tracking, data traceability, and environment specification. The workflow can be split into clearly documented stages:

    ### Environment and dependencies

    Development was conducted with Python 3.10.20. Core libraries and versions used for compatibility are:

    `pandas==2.2.3`

    `numpy==1.26.4`

    `scikit-learn==1.6.1`

    `torch==2.5.1` (with CUDA 12.1 support via the PyTorch index)

    `transformers==4.48.3`

    `biopython==1.87`

    `h5py==3.16.0`

    `matplotlib==3.9.3`

    `seaborn==0.13.2`

    `requests==2.32.3`

    `joblib==1.4.2`

    `tqdm==4.67.1`

    `plotly==6.9.0`

    !!! note "Note"
        For positional-model training in Google Colab, installing `torch` versions with proper CUDA support is recommended as described in notebook `5_Positional_training.ipynb`. The project `requirements.txt` includes detailed comments for correct installation by CUDA version (11.8, 12.1, 12.4, etc.).

    ### Original data acquisition and traceability

    To reproduce dataset construction, retrieve the following sources at their original access dates:

    - **UniProt (sequence download):**

        - Access date: **June 30, 2026**.
        - Script used: `src/utils/download_data.py`.
        - API endpoint: `https://rest.uniprot.org/uniprotkb/search`.
        - Query parameters: function `descargar_secuencias_uniprot` runs a composite query combining gene `rbcL`, protein name ("Ribulose bisphosphate carboxylase"), archaeal thermophile terms (*Pyrococcus*, *Thermococcus*, etc.), and explicit search for "RuBisCO type III".
        - The process downloaded, cleaned, and deduplicated entries using `Entry` identifier.

    - **TEMPURA (taxonomic merges):**

        - Access/downloaded version date: **June 30, 2026** (`200617_TEMPURA.csv`).
        - Database used exclusively to enrich sequences with `T_min`, `T_opt`, and `T_max`.

    ### Preprocessing and cleaning protocol

    The final dataset (2,000 sequences) was built through the following pipeline, documented conceptually in the Dataset construction section and implemented in `src/utils/data_enricher.py` and `src/utils/download_data.py`:

    1. **Initial cleaning:** Character normalization and space removal from sequences via `limpiar_y_tokenizar`.
    2. **Tokenization:** Separation of each amino acid by spaces for ESM-2 compatibility (`Tokenized_Sequence`).
    3. **Temperature assignment (hierarchical strategy):**
        - **Priority 1:** Exact species matching with TEMPURA (`genus_and_species`).
        - **Priority 2:** For Viridiplantae/Streptophyta organisms without exact match, assign conservative thermal range (`T_min=5.0`, `T_opt=25.0`, `T_max=35.0`).
        - **Priority 3:** Assign genus-mean temperature when species is unavailable.
    4. **Invalid-record removal:** Samples without assigned `T_opt` are filtered out (314 records excluded from supervised training).
    5. **Alignment (for positional models):** Pairwise alignment through `SequenceAligner` in `src/utils/alignment_utils.py`, using `BLOSUM62` and fixed length cropped to 1000 positions (as in `5_Positional_training.ipynb`) to control memory and ensure uniformity.
    6. **Stratified split:** 80/20 train-test split with fixed seed (`random_state = 42`) and manual temperature-bin stratification (`0-20`, `20-40`, `40-60`, `60-80`, `80-100` °C).

    ### Infrastructure and runtime

    For comparable runtimes, the following infrastructure is recommended. Total full-pipeline runtime is approximately **45-60 minutes** on standard mid-to-high-end hardware.

    - **Embedding extraction (ESM-2 `t6_8M`):** NVIDIA T4 GPU environment (Google Colab) or higher. Estimated runtime: **~15 minutes** for 2,000 sequences.
    - **Ensemble model training (Random Forest, SVR, GBR):** Modern CPU (8+ cores). Estimated runtime: **~2 minutes**.
    - **Positional CNN training:** NVIDIA T4 GPU. Estimated runtime: **~10 minutes** for 100 epochs.

    Following this protocol with the same parameters should produce a dataset with comparable statistics (length, domains, temperatures) and similar model performance to what is presented in this study.

    ## Future improvements

    While the current dataset and models are already a solid proof of concept for predicting RuBisCO optimal temperature from sequence, several improvements could expand project scope, reduce identified biases, and improve predictive performance.

    ### Dataset expansion and diversification

    The main limitation of the current dataset is strong taxonomic and thermal bias. Future iterations should focus on:

    - **Increasing bacterial representation:** Bacteria account for only 16.8% of the current dataset. A priority is to broaden UniProt queries to include more bacterial phyla, especially mesophilic and moderate-thermophilic groups.
    - **Reducing the "missing middle":** Underrepresentation of thermophiles (45-80 °C) limits generalization in this critical range. Complementary data sources such as **BacDive** or **NCBI BioSample** could be explored.
    - **Integrating non-canonical proteins:** The current dataset focuses on large subunit (`rbcL`). Including small subunit (`rbcS`) or less-studied RuBisCO isoforms could enrich feature space and reveal new thermal-stability patterns.

    ### Structural and functional characterization

    A promising next step is adding structural information. Although the current project focuses on primary sequence, future versions could include:

    - **AlphaFold DB integration:** Add structural embeddings (for example, Evoformer-derived representations) as extra features. Because 3D structure is tightly linked to thermal stability, this could significantly improve model accuracy.
    - **Functional and interaction annotations:** Enrich dataset with UniProt annotations related to post-translational modifications, cofactor-binding sites, or additional protein domains.

    ### Machine-learning model improvements

    Model architecture can also evolve substantially:

    - **Larger foundation models:** Current work uses ESM-2 `t6_8M` (8M parameters) for efficiency. Future iterations may evaluate larger variants such as `esm2_t36_3B_UR50D` (3B parameters), which capture richer sequence semantics.
    - **Multi-task learning:** Instead of predicting only `T_opt`, the model could jointly predict `T_min`, `T_opt`, and `T_max`, leveraging more information and forcing richer sequence-temperature relationship learning.
    - **Attention-based models (Transformers):** The positional 1D-CNN is a strong baseline, but a dedicated Transformer encoder over positional embeddings could capture long-range residue dependencies, a key factor in thermophilic protein stability.

    ### Experimental validation (benchmarking)

    Ultimately, predictive models in biology should be experimentally validated. Mid-term plans include:

    - **Wet-lab collaboration:** Select RuBisCO proteins with high-confidence and low-confidence predictions and validate them with enzyme activity assays at multiple temperatures.
    - **In-silico mutation analysis:** Use the positional model to identify residues contributing most to thermal stability and simulate point mutations to estimate impact on `T_opt`.

    These improvements are ambitious but realistic and could evolve the project from proof of concept into a robust reference tool for thermostable enzyme biotechnology.

=== "Español"

    ## Overview

    El acceso abierto a datos científicos de calidad ha transformado la forma en que se desarrolla la investigación. Gracias a iniciativas como UniProt y otras bases de datos públicas, hoy es posible que investigadores independientes, estudiantes y pequeños grupos de trabajo puedan desarrollar proyectos de alto impacto sin depender necesariamente de los recursos de grandes instituciones o empresas. La ciencia abierta no solo favorece la reproducibilidad y la transparencia, sino que también democratiza el acceso al conocimiento y acelera la innovación al permitir que cualquier persona pueda construir sobre el trabajo previo de la comunidad científica.

    Este proyecto nace precisamente aprovechando esa filosofía. Su objetivo es explorar hasta qué punto es posible predecir distintas propiedades fisicoquímicas de la enzima **RuBisCO** utilizando exclusivamente la información contenida en su secuencia primaria y otros metadatos biológicos asociados. Aunque el primer caso de estudio se centra en la predicción de la temperatura óptima de actividad, la arquitectura del proyecto está diseñada para extenderse en el futuro a otras propiedades de interés biotecnológico.

    Para ello se emplean dos tipos principales de información. Por un lado, las secuencias proteicas de RuBisCO, que constituyen la principal fuente de información sobre la estructura y función potencial de la proteína. Por otro, metadatos biológicos, principalmente información taxonómica de los organismos que expresan cada proteína y datos experimentales, que permiten asociar cada secuencia con la propiedad objetivo que se desea modelar, en este caso la **temperatura óptima de actividad**.

    Actualmente, el conjunto de datos se construye a partir de la integración de información procedente de **UniProt** y **TEMPURA**. No obstante, el proyecto está concebido como una plataforma en evolución, por lo que en futuras versiones podrán incorporarse nuevas fuentes de datos —como anotaciones funcionales, información estructural o bases de datos especializadas— con el objetivo de ampliar las capacidades predictivas y mejorar el rendimiento de los modelos.


    ## Fuentes de datos

    ### UniProt

    **UniProt** es una de las bases de datos de proteínas más completas y ampliamente utilizadas por la comunidad científica. Proporciona secuencias proteicas revisadas y anotaciones biológicas de alta calidad, convirtiéndose en una fuente de referencia para estudios de bioinformática, biología molecular y aprendizaje automático aplicado a proteínas.

    En este proyecto, UniProt constituye la fuente principal de información. A partir de ella se obtienen las secuencias de **RuBisCO**, junto con la información necesaria para identificar de forma unívoca cada proteína y relacionarla con el organismo del que procede.

    Los campos actualmente utilizados son:

    | Campo | Descripción |
    |:------|:------------|
    |Entry |	Identificador único de UniProt para cada proteína.|
    |Protein names	| Nombre anotado de la proteína. |
    |Organism	| Organismo del que procede la secuencia. |
    |Taxonomic lineage	| Clasificación taxonómica completa del organismo.|
    |Temperature dependence	| Anotaciones experimentales relacionadas con la dependencia de temperatura, cuando están disponibles.|
    |Sequence	| Secuencia primaria de aminoácidos utilizada como entrada principal de los modelos.|

    Aunque el proyecto almacena todos estos campos, la información más relevante durante esta primera fase corresponde a la **secuencia proteica**, el **organismo** y su **linaje taxonómico**, ya que permiten relacionar posteriormente cada proteína con información experimental procedente de otras fuentes.

    ### TEMPURA

    **TEMPURA** (Database of Growth TEMPERatures of Usual and Rare Prokaryotes) es una base de datos que recopila temperaturas de crecimiento determinadas experimentalmente para microorganismos, principalmente bacterias y arqueas.

    Mientras que UniProt proporciona información sobre las proteínas, TEMPURA aporta un tipo de información complementaria que resulta esencial para este proyecto: las condiciones térmicas en las que viven los organismos de los que proceden dichas proteínas.

    Mediante la información taxonómica compartida entre ambas bases de datos, las secuencias de RuBisCO obtenidas desde UniProt pueden enriquecerse con la temperatura media de crecimiento registrada experimentalmente para cada organismo. Esta temperatura se utiliza en la fase actual del proyecto como aproximación a la temperatura óptima de funcionamiento de la proteína y constituye la variable objetivo de los modelos de predicción.

    Además de la temperatura media, el proceso de integración conserva también las temperaturas mínima y máxima registradas para cada organismo. Aunque estos valores todavía no participan en el entrenamiento de los modelos, se almacenan con el objetivo de facilitar futuras líneas de investigación y permitir el desarrollo de nuevos enfoques predictivos.

    ### Integraciones futuras

    Uno de los objetivos a largo plazo del proyecto es enriquecer progresivamente el conjunto de datos incorporando nuevas fuentes de información biológica.

    Entre las posibles integraciones futuras se encuentra **AlphaFold DB**, cuya incorporación permitiría complementar la secuencia primaria con información estructural tridimensional predicha mediante inteligencia artificial. La inclusión de características derivadas de la estructura podría mejorar la capacidad predictiva de determinados modelos y abrir nuevas líneas de investigación centradas en la relación entre estructura y función proteica.

    No obstante, la incorporación de este tipo de información supone importantes retos computacionales y metodológicos, por lo que, por el momento, el proyecto se centra exclusivamente en información derivada de la secuencia primaria y de los metadatos biológicos asociados.

    A medida que el proyecto evolucione también se valorará la incorporación de otras bases de datos especializadas que permitan añadir anotaciones funcionales, estructurales o evolutivas, ampliando así el alcance de los modelos desarrollados.


    ## ¿Porqué estos datasets?

    La elección de **UniProt** y **TEMPURA** responde tanto a criterios científicos como a principios de reproducibilidad y accesibilidad.

    Desde un punto de vista técnico, ambas bases de datos aportan información altamente complementaria. UniProt proporciona secuencias proteicas cuidadosamente curadas y una extensa anotación biológica, mientras que TEMPURA incorpora datos experimentales sobre las condiciones térmicas de crecimiento de bacterias y arqueas. La integración de ambas fuentes permite construir un conjunto de datos consistente para estudiar la relación entre la secuencia primaria de RuBisCO y sus propiedades funcionales.

    Al mismo tiempo, estas bases de datos representan un excelente ejemplo del valor de la ciencia abierta. Su disponibilidad pública permite que investigadores, estudiantes y desarrolladores de cualquier parte del mundo puedan acceder a información de alta calidad sin depender de infraestructuras o recursos exclusivos de grandes organizaciones. Esta accesibilidad favorece la reproducibilidad de los resultados, facilita la validación independiente de los modelos y reduce las barreras de entrada para nuevas iniciativas de investigación.

    Este proyecto es, en gran medida, posible gracias a ese esfuerzo colectivo por compartir datos científicos de forma abierta. Del mismo modo, toda la metodología desarrollada aquí pretende ser transparente y reproducible para que cualquier persona interesada pueda comprenderla, replicarla y utilizarla como punto de partida para nuevas investigaciones.


    ## Construcción del dataset

    El conjunto de datos utilizado en este proyecto no procede de una única fuente, sino que se construye mediante un proceso de integración y enriquecimiento de datos procedentes de diferentes bases públicas. El objetivo es obtener un dataset consistente que combine la información de secuencia proporcionada por UniProt con información experimental sobre temperatura procedente de TEMPURA, aplicando posteriormente distintos procesos de limpieza, normalización y preparación para su utilización por los modelos de aprendizaje automático.

    ```mermaid
    flowchart TD

    A[UniProt REST API] --> B[Descarga de las\nsecuencias proteícas]

    B --> C[Limpieza de secuencias]
    C --> D[Tokenización de secuencias]

    D --> E[Cruce con TEMPURA]

    E --> F[Matching de especies]

    F --> G{¿Temperatura disponible?}

    G -->|Match exacto| H[Asignación de temperatura experimental]

    G -->|No| I[Normas fallback taxonómico]

    I --> J[Heurística de plantas]
    I --> K[Media de género]

    H --> L[Dataset enriquecido]
    J --> L
    K --> L

    L --> M[Alineamiento de secuencias]

    M --> N[Dataset final]
    ```


    ## Preprocesamiento

    Una vez descargados los datos desde las distintas fuentes, se aplica un proceso de preprocesamiento cuyo objetivo es construir un conjunto de datos consistente, reproducible y adecuado para el entrenamiento de los modelos de aprendizaje automático. Este proceso se compone de las siguientes etapas:

    ### 1. Descarga de secuencias

    La primera fase consiste en la obtención automática de secuencias de RuBisCO desde la API REST de UniProt.

    * Se realiza una búsqueda específica de proteínas RuBisCO utilizando tanto el nombre del gen (rbcL) como diferentes anotaciones de la proteína, ampliando la consulta para incluir variantes presentes en arqueas, especialmente especies termófilas e hipertermófilas.

    * La descarga se realiza de forma paginada para recuperar grandes volúmenes de información de manera eficiente y respetando las limitaciones de la API.

    * Una vez finalizada la descarga, se eliminan registros duplicados mediante el identificador único de UniProt (Entry), conservando únicamente una entrada por proteína.

    ### 2. Limpieza y preparación de secuencias

    Antes de utilizar las secuencias como entrada de los modelos, se aplican varias transformaciones destinadas a garantizar la calidad y homogeneidad de los datos.

    * Se eliminan todas aquellas entradas que no contienen una secuencia proteica válida, evitando incorporar muestras incompletas durante el entrenamiento.

    * Las secuencias se normalizan eliminando espacios, saltos de línea y otros caracteres de formato que puedan haberse introducido durante el proceso de descarga.

    * Finalmente, cada aminoácido se separa mediante espacios, generando una representación tokenizada compatible con modelos basados en transformadores, como ESM-2.

    ### 3. Enriquecimiento del conjunto de datos

    Las secuencias obtenidas desde UniProt se complementan con información experimental procedente de TEMPURA para incorporar la variable objetivo utilizada por los modelos.

    * Se normalizan previamente los nombres científicos de los organismos con el fin de facilitar el cruce entre ambas bases de datos y minimizar discrepancias derivadas del formato de escritura.

    * Cuando existe una coincidencia exacta entre especies, se asignan las temperaturas mínima, óptima y máxima registradas experimentalmente en TEMPURA.

    * Aunque en esta primera fase únicamente se utiliza la temperatura óptima como variable objetivo, también se almacenan las temperaturas mínima y máxima para facilitar futuras ampliaciones del proyecto.

    ### 4. Estrategia jerárquica de asignación

    No todas las especies presentes en UniProt disponen de información experimental en TEMPURA. Para aumentar la cobertura del conjunto de datos sin recurrir a asignaciones arbitrarias, se implementa una estrategia jerárquica basada en criterios biológicos.

    * La prioridad siempre es asignar la temperatura experimental correspondiente a la especie exacta cuando esta se encuentra disponible.

    * En el caso de organismos pertenecientes a Viridiplantae o Streptophyta, donde TEMPURA presenta una cobertura limitada, se asigna inicialmente un rango térmico conservador basado en las condiciones habituales de crecimiento de plantas fotosintéticas. Esta aproximación permite mantener estas secuencias en el conjunto de datos hasta disponer de información experimental más específica.

    * Cuando una especie no aparece en TEMPURA, se intenta recuperar la información a nivel de género utilizando los valores medios registrados para dicho grupo taxonómico.

    * Si ninguna de estas estrategias permite asignar una temperatura con suficiente confianza, la secuencia permanece sin etiquetar y queda excluida del entrenamiento supervisado.

    ### 5. Alineamiento de secuencias

    Como etapa final del preprocesamiento, las secuencias pueden alinearse respecto a una referencia común antes de emplearse en determinados modelos.

    * El alineamiento se realiza mediante algoritmos de alineamiento por pares utilizando la matriz de sustitución BLOSUM62 y penalizaciones configurables para la apertura y extensión de huecos (gaps).

    * Tras el alineamiento, todas las secuencias presentan una longitud uniforme, facilitando la comparación entre posiciones homólogas y la extracción de características dependientes de la posición.

    * La implementación también contempla la posibilidad de emplear algoritmos de alineamiento múltiple mediante herramientas externas como Clustal Omega o MAFFT, permitiendo adaptar el flujo de trabajo a distintos enfoques experimentales sin modificar el resto del pipeline.

    ## Dataset statistics

    | Métrica                            |   Valor |
    |:-----------------------------------|--------:|
    | Número de proteínas                |  2000   |
    | Número de organismos               |  1184   |
    | Número de géneros                  |   708   |
    | Número de dominios taxonómicos     |     4   |
    | Longitud media de secuencia (aa)   |   448.8 |
    | Longitud mediana de secuencia (aa) |   461   |
    | Longitud mínima (aa)               |    16   |
    | Longitud máxima (aa)               |  5902   |
    | Temperatura óptima media (°C)      |    52.9 |
    | Temperatura óptima mínima (°C)     |    20   |
    | Temperatura óptima máxima (°C)     |   105   |

    La tabla anterior resume las principales características del conjunto de datos utilizado en esta primera fase del proyecto. Tras el proceso de integración y preprocesamiento se dispone de **2.000 secuencias de RuBisCO**, pertenecientes a **1.184 organismos** distribuidos en **708 géneros** y **cuatro dominios taxonómicos**, proporcionando una representación amplia de la diversidad evolutiva presente en este tipo de enzimas.

    Las secuencias presentan una **longitud media de 448,8 aminoácidos**, muy próxima a la mediana (461 aminoácidos), lo que sugiere una distribución relativamente homogénea para la mayor parte del conjunto de datos. No obstante, también se observan algunos valores extremos, con secuencias significativamente más cortas o largas que la longitud habitual de una RuBisCO, cuya influencia será analizada en los apartados posteriores mediante el estudio de la distribución y la detección de posibles valores atípicos.

    En cuanto a la variable objetivo, las **temperaturas óptimas** abarcan un amplio rango experimental, **desde 20 °C hasta 105 °C, con una media de 52,9 °C**. Esta variabilidad resulta especialmente interesante desde el punto de vista del aprendizaje automático, ya que permite entrenar modelos sobre proteínas adaptadas a condiciones ambientales muy diferentes, incluyendo organismos mesófilos, termófilos e hipertermófilos.

    ### Estadisticas de las secuencias

    | Estadístico          |   Valor |
    |:---------------------|--------:|
    | Número de secuencias | 2000    |
    | Media                |  448.8  |
    | Mediana              |  461    |
    | Desviación estándar  |  280.02 |
    | Mínimo               |   16    |
    | Percentil 25         |  376.75 |
    | Percentil 75         |  476    |
    | Percentil 95         |  743.25 |
    | Máximo               | 5902    |


    La longitud de las secuencias constituye una característica de gran relevancia tanto desde el punto de vista biológico como computacional. Las diferencias en el número de aminoácidos pueden reflejar la existencia de distintas formas de RuBisCO, proteínas de fusión, secuencias parciales o posibles anomalías de anotación, además de influir directamente en las estrategias de preprocesamiento y representación empleadas por los modelos de aprendizaje automático.

    Las 2.000 secuencias analizadas presentan una longitud media de 448,8 aminoácidos y una mediana de 461 aminoácidos, valores muy próximos entre sí que indican que la mayor parte del conjunto de datos se concentra alrededor de la longitud esperada para las proteínas RuBisCO. Asimismo, el 50 % de las secuencias se sitúa entre 376,8 y 476 aminoácidos, lo que refleja una elevada homogeneidad en la distribución principal del conjunto de datos.

    ![Histograma de las longitudes](assets/figures/datasets/sequence_length_distribution.png)

    No obstante, la desviación estándar (280 aminoácidos) y el amplio rango observado, desde 16 hasta 5.902 aminoácidos, evidencian la presencia de un reducido número de valores extremos. El histograma completo muestra cómo la distribución está claramente dominada por unas pocas secuencias excepcionalmente largas, mientras que el diagrama de cajas confirma la existencia de numerosos valores atípicos, especialmente en el extremo superior de la distribución.

    ![Boxplot de las longitudes](assets/figures/datasets/sequence_length_boxplot.png)

    Para facilitar una interpretación más representativa del conjunto de datos, también se presenta un histograma limitado al percentil 99. Esta visualización elimina el efecto de las secuencias más extremas y permite apreciar con mayor claridad que la inmensa mayoría de las proteínas poseen una longitud inferior a aproximadamente 1.300 aminoácidos, concentrándose principalmente alrededor de los 450 aminoácidos.

    ![Histograma acotado de las longitudes](assets/figures/datasets/sequence_length_distribution_cutoff.png)

    Aunque estas secuencias atípicas no se eliminan automáticamente del conjunto de datos, su identificación resulta fundamental para futuras fases del proyecto, ya que serán objeto de un análisis individual con el fin de determinar si corresponden a proteínas de fusión, anotaciones incompletas, variantes biológicamente relevantes o posibles errores de anotación en las bases de datos de origen.

    ### Estadísticas de temperatura

    La temperatura óptima de funcionamiento constituye la variable objetivo de esta primera fase del proyecto. A partir de la información taxonómica obtenida de TEMPURA fue posible asignar un valor experimental de temperatura óptima a **1.686** de las **2.000** secuencias recopiladas, que conforman el conjunto utilizado para el entrenamiento y evaluación de los modelos.

    | Estadístico             |   Valor |
    |:------------------------|--------:|
    | Número de observaciones | 1686    |
    | Media                   |   52.93 |
    | Mediana                 |   28.2  |
    | Desviación estándar     |   30.77 |
    | Mínimo                  |   20    |
    | Percentil 25            |   25    |
    | Percentil 75            |   85    |
    | Percentil 95            |  100    |
    | Máximo                  |  105    |

    Los valores abarcan un **amplio rango**, desde **20 °C** hasta **105 °C**, reflejando la enorme diversidad de nichos ecológicos en los que habitan los organismos que contienen RuBisCO. La mediana se sitúa en **28,2 °C**, mientras que la media asciende a **52,9 °C**, una diferencia considerable que evidencia que la distribución no es simétrica y está fuertemente influenciada por organismos adaptados a temperaturas elevadas.

    ![Histograma global de temperatura óptima](assets/figures/datasets/topt_distribution.png)

    El histograma confirma que la distribución presenta varios picos claramente diferenciados, en lugar de seguir una distribución aproximadamente normal. Destaca una elevada concentración de organismos con temperaturas óptimas cercanas a 25 °C, correspondiente principalmente a especies mesófilas, junto con un segundo grupo muy numeroso situado entre 80 y 90 °C, característico de arqueas termófilas e hipertermófilas. También aparecen agrupaciones menores alrededor de 60 °C, 70 °C y próximas a 100 °C, reflejando la diversidad de estrategias adaptativas presentes en la naturaleza.

    ![Boxplot de distribución de temperaturas óptimas](assets/figures/datasets/topt_boxplot.png)

    El diagrama de cajas complementa esta información mostrando una gran dispersión de los datos. A diferencia de lo observado en la longitud de las secuencias, no se aprecian valores atípicos especialmente extremos; el amplio rango de temperaturas forma parte de la propia variabilidad biológica del conjunto de datos.

    | Thermal_Group | count | 
    |:----------------|--------:| 
    | Hipertermófilos | 959 | 
    | Mesófilos | 916 | 
    | Termófilos | 125 |

    Una forma más intuitiva de interpretar la distribución consiste en agrupar las proteínas según la clasificación térmica del organismo del que proceden. El conjunto de datos está dominado por secuencias procedentes de **hipertermófilos (959 proteínas)** y **mesófilos (916 proteínas)**, mientras que los **termófilos (125 proteínas)** representan una fracción considerablemente menor del conjunto.

    En este trabajo, los grupos térmicos se han definido a partir de la **temperatura óptima (Topt)** asociada a cada organismo: mesófilos **(<45 °C), termófilos (45–80 °C) e hipertermófilos (≥80 °C)**. Dado que el conjunto de datos no contiene organismos con temperaturas óptimas inferiores a 20 °C, no se identificaron representantes del grupo de los psicrófilos.

    Esta distribución explica la naturaleza multimodal observada en las temperaturas óptimas y pone de manifiesto un cierto desequilibrio entre grupos térmicos. En particular, los organismos adaptados a temperaturas extremadamente elevadas y a temperaturas moderadas están ampliamente representados, mientras que las proteínas procedentes de organismos termófilos constituyen una minoría. Este aspecto deberá tenerse en cuenta durante el entrenamiento y la evaluación de los modelos, ya que puede influir en su capacidad para generalizar correctamente sobre regiones menos representadas del espacio de temperaturas.

    Este desbalanceo implica que el modelo podría aprender muy bien a predecir temperaturas cercanas a 25 °C y 85 °C, pero probablemente **tendrá dificultades para generalizar en el rango intermedio (45–80 °C)**. Cualquier predicción en este rango deberá ser evaluada con especial cautela, y se recomienda considerar técnicas de remuestreo, ponderación de clases o validación cruzada estratificada durante el entrenamiento.

    ![Distribución de la temperatura óptima por dominio](assets/figures/datasets/topt_violin_by_domain.png)

    Finalmente, al separar las observaciones por dominio taxonómico se aprecia que la distribución de temperaturas está estrechamente relacionada con la filogenia de los organismos. Las arqueas concentran la mayor parte de las temperaturas elevadas, mientras que las secuencias eucariotas se agrupan casi exclusivamente alrededor de 25 °C. Las bacterias presentan una distribución mucho más amplia, ocupando un rango intermedio que incluye desde organismos psicrófilos hasta termófilos moderados.

    Esta fuerte relación entre taxonomía y temperatura confirma la importancia de conservar la información filogenética durante el preprocesamiento, ya que constituye una fuente de información biológicamente relevante para interpretar las propiedades funcionales de las proteínas y puede aportar contexto adicional durante el entrenamiento de futuros modelos.

    ### Distribución taxonómica

    La composición taxonómica del conjunto de datos es un factor determinante para comprender el alcance y las limitaciones de los modelos entrenados. El análisis de los dominios taxonómicos y los géneros más frecuentes revela una estructura de muestreo claramente sesgada hacia organismos con estrategias adaptativas extremas.

    ![Distribución por dominio](assets/figures/datasets/taxonomic_domain_distribution.png)

    El desglose por dominio taxonómico muestra que las secuencias de **Eukaryota (892 proteínas, 44.6 %)** y **Archaea (771 proteínas, 38.5 %)** son los grupos mayoritarios y están representados de forma casi equitativa. A pesar de la inmensa diversidad y relevancia ecológica de **Bacteria**, este dominio contribuye con **336 proteínas (16.8 %)** y, junto con un único registro clasificado como Other, completa el total de 2.000 secuencias.

    Esta distribución es especialmente relevante cuando se contrasta con la información térmica presentada anteriormente. La gran cantidad de secuencias de Eukaryota (principalmente plantas y algas) explica el pico de temperaturas óptimas en torno a los 25 °C, mientras que la abundante presencia de Archaea es la responsable directa del segundo pico alrededor de los 85 °C. El dominio Bacteria, aunque menos numeroso, actúa como un puente térmico que aporta la variabilidad en el rango intermedio (termófilo), aunque en menor proporción.

    ![Top 10 géneros](assets/figures/datasets/top_10_genera.png)

    El análisis de los **10 géneros más frecuentes** confirma aún más este patrón. El conjunto de datos está dominado casi por completo por géneros de **arqueas hipertermófilas y termófilas**, como *Pyrococcus* (~190 secuencias), *Methanocaldococcus* (~155), *Saccharolobus* (~135) o *Thermococcus* (~110), entre otros. Esta sobrerrepresentación de arqueas extremófilas es una consecuencia directa del cruce con la base de datos TEMPURA, la cual posee una cobertura históricamente alta para microorganismos de ambientes geotérmicos.

    *Lupinus* y *Nostoc* son los únicos representantes eucariotas y bacterianos que logran aparecer en el top 10, probablemente debido a una alta frecuencia de anotaciones en UniProt o a esfuerzos de secuenciación específicos. La presencia de *Nostoc* también añade una interesante diversidad evolutiva, ya que se trata de **cianobacterias** capaces de realizar fijación de nitrógeno y fotosíntesis.

    Esta distribución taxonómica tiene consecuencias directas en el rendimiento de los modelos de aprendizaje automático. El fuerte sesgo hacia los extremos térmicos (mesófilos e hipertermófilos) y la infrarrepresentación de los termófilos intermedios (que apenas representan 125 secuencias) sugiere que los modelos podrían tener una **menor capacidad de generalización en el rango de 45–80 °C**. Si bien el conjunto de datos es amplio y diverso para los rangos bajo y alto, cualquier predicción para organismos termófilos moderados debe ser interpretada con cautela.

    ### Análisis de valores ausentes

    Uno de los aspectos críticos para el entrenamiento de modelos supervisados es la disponibilidad de la variable objetivo. Tras el proceso de cruce con TEMPURA, **314 de las 2.000 secuencias (15,7 %)** no pudieron ser etiquetadas con una temperatura óptima. El análisis de estos registros revela que la gran mayoría corresponden a **organismos eucariotas** (principalmente plantas y algas) y a **bacterias** no termófilas. 

    Esta falta de cobertura es una limitación inherente a la naturaleza de TEMPURA, una base de datos especializada en procariotas de ambientes extremos. Por lo tanto, el conjunto de datos final utilizado para el entrenamiento (1.686 muestras) está ligeramente sobrerrepresentado por organismos procariotas, lo cual debe tenerse en cuenta al interpretar los resultados.

    ### Relación entre longitud de secuencia y temperatura

    Un análisis exploratorio adicional consiste en estudiar si existe alguna relación entre la longitud de las proteínas RuBisCO y la temperatura óptima del organismo que las expresa.

    ![Scatter plot de Longitud vs Temperatura Óptima](assets/figures/datasets/length_vs_topt_scatter.png)

    El gráfico de dispersión revela varios patrones interesantes. En primer lugar, se observa una **elevada densidad de puntos en el rango de 400 a 500 aminoácidos**, que se mantiene constante a lo largo de todo el espectro de temperaturas. Esto indica que la longitud canónica de la proteína RuBisCO se conserva independientemente del nicho térmico del organismo. Las "columnas" verticales que se aprecian en el eje de temperaturas son un artefacto de los valores discretos de temperatura registrados en la base de datos TEMPURA.

    Lo más destacado del gráfico es la **asimetría en la dispersión hacia longitudes mayores**. Mientras que los organismos mesófilos (representados en azul, T_opt < 45 °C) apenas presentan secuencias que superen los 500 aminoácidos, los organismos hipertermófilos (en rojo, T_opt ≥ 80 °C) muestran una **larga cola de valores atípicos**, con múltiples secuencias que superan los 1.000 e incluso los 1.800 aminoácidos. Esto sugiere que las proteínas de fusión o las variantes estructurales extendidas son un fenómeno casi exclusivo de arqueas adaptadas a ambientes extremos.

    ### Distribución de la longitud por dominio taxonómico

    Al desglosar la longitud de las secuencias por dominio, se aprecian diferencias significativas en la estructura de los datos.

    ![Boxplot de Longitud por Dominio](assets/figures/datasets/length_by_domain.png)

    El boxplot (con el eje Y limitado a 1.500 aa para apreciar la caja central) muestra diferencias notables:

    *   **Eukaryota:** Presenta la distribución más homogénea. La caja es extremadamente estrecha, con una mediana muy próxima a los **450 aminoácidos**. Apenas unos pocos valores atípicos superan los 600 aa. Esto refleja la alta estabilidad evolutiva de la RuBisCO en el cloroplasto de plantas y algas.
    *   **Bacteria:** Su comportamiento es similar al de los eucariotas, con una caja muy ajustada y una mediana en torno a los **470 aminoácidos**. Presenta algunos valores atípicos a la baja, posiblemente debidos a fragmentos de secuencia o anotaciones incompletas.
    *   **Archaea:** Este dominio es el principal responsable de la variabilidad observada en el conjunto de datos. Su mediana es sensiblemente inferior, situándose alrededor de los **380 aminoácidos**, pero su rango intercuartílico (el tamaño de la caja) es mucho más amplio. Sobre todo, destaca por la **enorme cantidad de outliers superiores**, que se extienden hasta más allá de los 1.400 aminoácidos (y que en el conjunto global alcanzan los 5.902 aa). Esta variabilidad extrema es consistente con la presencia de proteínas de fusión y dominios adicionales en arqueas termófilas e hipertermófilas.

    En conjunto, el scatter plot del apartado anterior y el boxplot confirman que, aunque la mayoría de las secuencias siguen el patrón canónico de ~450 aminoácidos, el **sesgo hacia arqueas hipertermófilas** es el que introduce la mayor parte de la variabilidad y los valores atípicos en la distribución de longitudes.


    ## Train / Validation / Test split

    La correcta partición del conjunto de datos es un paso crítico para garantizar la evaluación objetiva de los modelos y evitar el sobreajuste. En este proyecto, se han implementado dos estrategias de división diferentes, adaptadas a la naturaleza de cada arquitectura de modelo.

    ### Proporciones y semilla aleatoria

    Tanto para los modelos basados en ensemble como para la red neuronal convolucional (CNN), se ha empleado una división **80/20** entre el conjunto de entrenamiento y el de evaluación.

    Para garantizar la reproducibilidad de los experimentos, se fijó una semilla aleatoria común (`random_state = 42`) en todas las funciones de partición. Esta decisión asegura que, al re-ejecutar el pipeline de entrenamiento, se obtengan exactamente los mismos subconjuntos de datos para entrenar y evaluar los modelos.

    ### Estratificación

    Debido al **fuerte desbalanceo entre clases térmicas** identificado en la sección de análisis de datos (con una clara infrarrepresentación de los organismos termófilos), la simple división aleatoria podría haber producido conjuntos de prueba desequilibrados. Para mitigar este problema, se implementó una estrategia de estratificación personalizada en la función `stratified_split_simple` del módulo `train_predict.py`.

    Esta estrategia consiste en:

    1. **Agrupación por rangos térmicos:** Se dividen las temperaturas óptimas en cinco categorías (bins): `0-20°C`, `20-40°C`, `40-60°C`, `60-80°C` y `80-100°C`.

    2. **División proporcional por grupo:** Dentro de cada categoría, las muestras se mezclan aleatoriamente y se extrae un porcentaje fijo (test_size = 0.2) para formar parte del conjunto de test, garantizando que todas las categorías térmicas estén representadas en la evaluación en la misma proporción que en el conjunto original.

    3. **Fallback de seguridad:** En el caso improbable de que alguna categoría contenga menos de 2 muestras (lo que haría imposible la estratificación), el sistema recurre automáticamente a una división aleatoria simple para evitar errores.

    Cabe destacar que, para el entrenamiento del modelo **CNN posicional** (`5_Positional_training.ipynb`), no se aplicó esta estratificación, sino que se utilizó una división aleatoria simple mediante train_test_split de scikit-learn, dado que el objetivo principal era monitorizar la curva de pérdida para aplicar early stopping.

    ### Prevención del Data Leakage

    El **data leakage** (fuga de información) es uno de los errores más comunes en el desarrollo de modelos, y ocurre cuando información del conjunto de test se filtra al proceso de entrenamiento. Para evitarlo, se han seguido las siguientes buenas prácticas en el pipeline:

    * **Partición antes del entrenamiento:** La división de los datos en conjuntos disjuntos se realiza antes de iniciar el ciclo de entrenamiento. Esto garantiza que ninguna secuencia o embedding utilizado para ajustar los pesos del modelo haya sido visto durante la evaluación.

    * **Escalado de características (SVR):** En el caso específico del modelo SVR, el `StandardScaler` se ajusta (*fit*) exclusivamente sobre los datos de entrenamiento (`X_train`). Posteriormente, esa misma transformación se aplica a `X_test` utilizando el método `.transform()`, evitando que los parámetros de normalización se vean influenciados por los datos de evaluación.

    * **Independencia del alineamiento:** Para el modelo posicional, el alineamiento de secuencias se realiza sobre la totalidad del conjunto de datos. Sin embargo, dado que se trata de un **alineamiento por pares** contra una referencia fija (no un alineamiento múltiple progresivo que dependa de otras secuencias), la información posicional de cada secuencia es independiente del resto. Por lo tanto, el proceso de alineamiento no introduce fuga de información entre los conjuntos de train y test.

    ### Validación Cruzada Interna

    Un aspecto adicional a destacar es que, para los modelos clásicos (Random Forest, Gradient Boosting, SVR y Ensemble), el conjunto de entrenamiento (el 80% de los datos) se sometió a una validación cruzada de 5 folds (5-CV). Esta técnica divide el conjunto de entrenamiento en 5 subconjuntos, entrenando el modelo en 4 de ellos y validando en el restante de forma iterativa. Esto proporciona una estimación mucho más robusta del rendimiento del modelo durante el desarrollo, antes de realizar la evaluación final sobre el 20% de test completamente aislado.

    ## Sesgos y limitaciones

    A pesar de los esfuerzos por construir un conjunto de datos amplio y representativo, el proceso de integración entre UniProt y TEMPURA, unido a la naturaleza de la propia literatura científica, introduce una serie de sesgos sistemáticos que deben ser reconocidos antes de interpretar el rendimiento de los modelos. Estos sesgos no invalidan el estudio, pero sí acotan el alcance de sus conclusiones y determinan los escenarios en los que los modelos pueden generalizar con confianza.

    ### Sesgo taxonómico (Dominio y Género)

    El análisis de la composición taxonómica ha revelado una distribución de dominios que se aleja de la diversidad biológica real.

    Sobrerrepresentación de Eukaryota y Archaea: Las secuencias de plantas y algas (Eukaryota, 892 proteínas) y de arqueas termófilas (Archaea, 771 proteínas) constituyen conjuntamente el 83 % del dataset.

    Infrarrepresentación de Bacteria: A pesar de que las bacterias representan la mayor parte de la diversidad procariota y son responsables de gran parte de la fijación de carbono en ecosistemas marinos y terrestres, solo contribuyen con 336 proteínas (16,8 %).

    A nivel de género, el sesgo es aún más extremo. El dataset está dominado por géneros de arqueas hipertermófilas (Pyrococcus, Methanocaldococcus, Saccharolobus, Thermococcus), que acumulan la mayor parte de las secuencias. Lupinus (plantas) y Nostoc (cianobacterias) son los únicos representantes no arqueanos que logran colarse en el top 10.

    Implicación: Los modelos tendrán una capacidad de generalización mucho mayor para predecir la temperatura de plantas y arqueas hipertermófilas que para bacterias mesófilas o termófilas.

    ### Sesgo térmico (El "missing middle")

    El desequilibrio taxonómico se traduce directamente en un fuerte desbalanceo en la variable objetivo. La distribución de temperaturas óptimas es claramente bimodal, con dos picos principales:

    Mesófilos (< 45 °C): 916 proteínas.

    Hipertermófilos (≥ 80 °C): 959 proteínas.

    Termófilos (45–80 °C): Tan solo 125 proteínas.

    Esta marcada infrarrepresentación del rango intermedio (entre 45 °C y 80 °C) tiene implicaciones críticas para el aprendizaje automático. Aunque la función de pérdida (MSE) promediará los errores, los modelos tenderán a optimizar su rendimiento en los rangos bajo y alto, donde hay abundancia de datos, a costa de un peor desempeño en el rango intermedio.

    ### Sesgo de muestreo y publicación (Publication Bias)

    La fuente de los datos, principalmente la literatura científica curada en UniProt y las mediciones experimentales de TEMPURA, introduce un sesgo de publicación inevitable.

    * **Modelos biológicos clásicos:** Eukaryota está sobrerrepresentada porque la RuBisCO de plantas (como *Arabidopsis* o *Spinacia*) es uno de los modelos más estudiados en fisiología vegetal y bioquímica.

    * **Interés biotecnológico:** Las arqueas hipertermófilas (como *Pyrococcus* o *Thermococcus*) son extremadamente populares en biotecnología debido a sus enzimas termoestables, lo que explica la enorme cantidad de secuencias depositadas en UniProt.

    * **Menor atención a bacterias mesófilas/termófilas:** Muchas bacterias ambientales, a pesar de su importancia ecológica, han sido menos secuenciadas y estudiadas en el contexto de la termoestabilidad de sus proteínas, lo que explica su infrarrepresentación en este dataset.

    Este sesgo implica que los resultados de este estudio son particularmente relevantes para organismos modelo y de interés biotecnológico, pero podrían no extrapolarse con la misma precisión a la diversidad bacteriana menos caracterizada.

    ### Sesgo de datos faltantes (Missing Data)

    Como se mencionó en el análisis de valores ausentes, **314 de las 2.000 secuencias (15,7 %)** no pudieron ser etiquetadas con una temperatura óptima tras el cruce con TEMPURA. El análisis cualitativo de estos registros indica que la gran mayoría corresponden a **organismos eucariotas** (principalmente algas y plantas no modelo) y a **bacterias** no termófilas.

    La ausencia de estos datos tiene un doble efecto:

    1. **Reduce el tamaño muestral** ya limitado para el rango intermedio de temperaturas.

    2. **Refuerza el sesgo taxonómico**, ya que las muestras perdidas pertenecen en su mayoría a los grupos ya infrarrepresentados (Bacteria y Eucariotas no modelo), exacerbando la dominancia de las arqueas hipertermófilas en el conjunto de entrenamiento final.

    ### Implicaciones para el rendimiento de los modelos

    La combinación de los sesgos descritos tiene consecuencias directas en el rendimiento esperado de los modelos entrenados (Random Forest, Ensemble y CNN posicional):

    * **Alta precisión en los extremos:** Se espera que los modelos predigan con gran precisión temperaturas en torno a 25 °C (plantas) y 85–100 °C (arqueas hipertermófilas), ya que son las regiones mejor muestreadas.

    * **Baja fiabilidad en el rango medio (45–80 °C):** Cualquier predicción en el rango termófilo debe interpretarse con extrema cautela. Los modelos carecen de ejemplos suficientes para aprender las relaciones estructura-función específicas de este nicho térmico.

    * **Dependencia del grupo taxonómico:** Los modelos probablemente aprenderán a inferir la temperatura a partir de patrones filogenéticos (por ejemplo, si una secuencia es de una arquea, predecir alta temperatura; si es de una planta, predecir baja temperatura) en lugar de aprender propiedades fisicoquímicas intrínsecas de la secuencia. Esto es un riesgo de shortcut learning (atajo) que debe ser evaluado en experimentos de validación cruzada con taxones excluidos.

    **Conclusión de las limitaciones:** Este dataset es una herramienta excelente para explorar la relación entre secuencia y estabilidad térmica en **organismos modelo y extremófilos**, pero no es un conjunto representativo de toda la diversidad de la enzima RuBisCO en la naturaleza. Las conclusiones extraídas de los modelos deben ser consideradas como **hipótesis biológicas robustas dentro de estos nichos ecológicos**, en lugar de leyes universales aplicables a cualquier organismo.

    ## Licensing

    El respeto por los términos de uso de las fuentes de datos y la propiedad intelectual es un pilar fundamental de este proyecto. Aunque el objetivo es desarrollar un flujo de trabajo reproducible, se ha prestado especial atención a no infringir las licencias de las bases de datos utilizadas.

    ### UniProt

    **UniProt** se distribuye bajo la licencia **Creative Commons Attribution-NoDerivatives (CC BY-ND 4.0)**. Esto permite a los usuarios utilizar, descargar y redistribuir libremente los datos, siempre que se atribuya correctamente la fuente y no se modifiquen las anotaciones originales.
    En este proyecto, la obtención de datos se realiza exclusivamente a través de la API REST pública de UniProt, cumpliendo estrictamente con sus términos de servicio y políticas de uso. Todas las secuencias descargadas se utilizan únicamente con fines de investigación no comercial.

    ### TEMPURA

    **TEMPURA** (Database of Growth TEMPERatures of Usual and Rare Prokaryotes) está sujeta a una **licencia académica específica**. Aunque su uso con fines de investigación está generalmente permitido, la **redistribución directa del conjunto de datos original está restringida** por sus términos de uso.

    ### Política de redistribución y acceso a los datos

    Para garantizar el cumplimiento de las licencias mencionadas, este proyecto adopta las siguientes prácticas:

    * **No redistribución de datos originales:** El repositorio del proyecto **no incluye, ni ha incluido en ningún momento, copias de los datasets originales** de TEMPURA o de las descargas masivas desde UniProt.

    * **Flujo de trabajo basado en scripts:** Toda la construcción del dataset se realiza mediante scripts que actúan sobre las APIs públicas. De esta forma, cualquier investigador que desee replicar el estudio puede hacerlo utilizando las mismas fuentes de datos, manteniendo la trazabilidad y evitando la distribución de materiales sujetos a derechos de autor.

    * **Documentación de los métodos:** Se proporcionan instrucciones claras sobre cómo configurar el entorno y ejecutar el proceso de descarga y preprocesamiento.

    ### Código fuente y documentación

    Como parte de la estrategia de ciencia abierta, la **documentación generada** (incluyendo este archivo descriptivo, las estadísticas del dataset y la metodología de preprocesamiento) se comparte públicamente para garantizar la transparencia y la reproducibilidad de los resultados.

    Sin embargo, el **código fuente interno** del pipeline de descarga y procesamiento es de carácter privado, tal como se ha establecido en los objetivos iniciales del proyecto. Los scripts de descarga se documentan conceptualmente en esta guía, permitiendo a otros investigadores comprender el proceso sin necesidad de acceder al código subyacente. Esto equilibra la protección de la propiedad intelectual con el compromiso de compartir el conocimiento y la metodología con la comunidad científica.

    ## Reproductibility

    A pesar de que el código fuente del pipeline de integración y procesamiento es de carácter privado, este proyecto está diseñado bajo un firme compromiso con la **reproducibilidad metodológica**. Cualquier investigador o estudiante con conocimientos intermedios de Python y bioinformática puede replicar los resultados presentados en este estudio siguiendo el protocolo detallado a continuación.

    Para garantizar la reproducibilidad, se han seguido las mejores prácticas en cuanto al registro de versiones, trazabilidad de los datos y especificación del entorno. El flujo de trabajo se puede dividir en las siguientes etapas perfectamente documentadas:

    ### Entorno y dependencias
    El desarrollo se ha llevado a cabo utilizando Python 3.10.20. Las bibliotecas esenciales y sus versiones específicas para garantizar la compatibilidad son las siguientes (extraídas del archivo requirements.txt del proyecto):

    `pandas==2.2.3`

    `numpy==1.26.4`

    `scikit-learn==1.6.1`

    `torch==2.5.1` (con soporte CUDA 12.1 a través del índice de PyTorch)

    `transformers==4.48.3`

    `biopython==1.87`

    `h5py==3.16.0`

    `matplotlib==3.9.3`

    `seaborn==0.13.2`

    `requests==2.32.3`

    `joblib==1.4.2`

    `tqdm==4.67.1`

    `plotly==6.9.0`

    !!! note "Nota"
        Para el entrenamiento del modelo posicional en Google Colab, se recomienda instalar las versiones específicas de `torch` con soporte CUDA tal como se detalla en el notebook `5_Positional_training.ipynb`. El archivo `requirements.txt` del proyecto incluye comentarios detallados para la instalación correcta en función de la versión de CUDA disponible en el sistema (11.8, 12.1, 12.4, etc.).

    ### Adquisición y trazabilidad de los datos originales

    Para replicar la construcción del dataset, se deben obtener las siguientes fuentes en sus fechas de acceso originales:

    * **UniProt (descarga de secuencias):**

        * Fecha de acceso: **30 de junio de 2026**.

        * Script utilizado: `src/utils/download_data.py`.

        * Endpoint de la API: `https://rest.uniprot.org/uniprotkb/search`.

        * Parámetros de la consulta: La función `descargar_secuencias_uniprot` ejecuta una búsqueda compuesta que combina el gen `rbcL`, el nombre de la proteína ("Ribulose bisphosphate carboxylase"), términos específicos para arqueas termófilas (*Pyrococcus*, *Thermococcus*, etc.) y la búsqueda explícita de "RuBisCO type III".

        * El proceso descargó, limpió y eliminó duplicados basándose en el identificador `Entry`.

    * **TEMPURA (cruces taxonómicos):**

        * Fecha de acceso/versión descargada: **30 de junio de 2026** (archivo `200617_TEMPURA.csv`).

        * La base de datos se utilizó exclusivamente para enriquecer las secuencias con los valores `T_min`, `T_opt` y `T_max`.

    ### Protocolo de preprocesamiento y limpieza

    El conjunto de datos final (2.000 secuencias) se construyó siguiendo rigurosamente el siguiente pipeline, que ha sido documentado conceptualmente en la sección Construcción del dataset de esta guía y codificado en los scripts `src/utils/data_enricher.py` y `src/utils/download_data.py`:

    1. **Limpieza inicial:** Normalización de caracteres y eliminación de espacios en las secuencias mediante la función limpiar_y_tokenizar.

    2. **Tokenización:** Separación de cada aminoácido por un espacio para la compatibilidad con ESM-2 (Tokenized_Sequence).

    3. **Asignación de temperatura (Estrategia Jerárquica):**

        * **Prioridad 1:** Cruce exacto por especie con TEMPURA (basado en genus_and_species).

        * **Prioridad 2:** Para organismos clasificados como Viridiplantae o Streptophyta sin coincidencia exacta, asignación de un rango térmico conservador (`T_min=5.0`, `T_opt=25.0`, `T_max=35.0`).

        * **Prioridad 3:** Asignación por temperatura media del género cuando la especie no está disponible, utilizando la agrupación por género de TEMPURA.

    4. **Eliminación de registros no válidos:** Filtrado de muestras sin una `T_opt` asignada (314 registros descartados para el entrenamiento supervisado).

    5. ** Alineamiento (para modelos posicionales):** Alineamiento por pares utilizando la clase `SequenceAligner` de `src/utils/alignment_utils.py`, con matriz `BLOSUM62` y una longitud fija recortada a 1000 posiciones (tal como se implementa en `5_Positional_training.ipynb`) para controlar el uso de memoria y garantizar la uniformidad.

    6. **División estratificada:** Partición Train/Test 80/20 con semilla fija (`random_state = 42`) y estratificación manual basada en rangos de temperatura (`0-20`, `20-40`, `40-60`, `60-80`, `80-100` °C) para garantizar la representación de todos los nichos térmicos en el conjunto de evaluación.

    ### Infraestructura y tiempos de ejecución

    Para replicar los resultados con tiempos de ejecución similares, se recomienda la siguiente infraestructura. El tiempo total del pipeline completo es de aproximadamente **45–60 minutos** en hardware estándar de gama media-alta.

    * **Extracción de embeddings (ESM-2 `t6_8M`):** Entorno con GPU NVIDIA T4 (en Google Colab) o superior. Tiempo estimado: **~15 minutos** para 2.000 secuencias.

    * **Entrenamiento del modelo Ensemble (Random Forest, SVR, GBR):** CPU moderna (8+ núcleos). Tiempo estimado: **~2 minutos**.

    * **Entrenamiento del modelo CNN posicional:** GPU NVIDIA T4. Tiempo estimado: **~10 minutos** para 100 épocas.

    La ejecución de este protocolo, siguiendo exactamente los pasos y parámetros aquí descritos, debería conducir a un conjunto de datos con las mismas métricas estadísticas (distribución de longitudes, dominios y temperaturas) y un rendimiento de modelos análogo al presentado en este estudio.

    ## Mejoras Futuras

    Si bien el dataset actual y los modelos desarrollados han demostrado ser una prueba de concepto sólida para predecir la temperatura óptima de RuBisCO a partir de su secuencia, existen numerosas vías de mejora que permitirían ampliar el alcance del proyecto, reducir los sesgos identificados y aumentar la precisión de las predicciones.

    ### Expansión y diversificación del dataset

    La principal limitación del conjunto de datos actual es el fuerte sesgo taxonómico y térmico. Para abordarlo, las futuras iteraciones deberían centrarse en:

    * **Aumentar la representación bacteriana:** Dado que las bacterias representan solo el 16,8 % del dataset actual, una prioridad será ampliar la búsqueda en UniProt para incluir una mayor diversidad de filos bacterianos, especialmente aquellos que habitan en rangos de temperatura mesófilos y termófilos moderados.

    * **Reducir el "missing middle":** La infrarrepresentación de organismos termófilos (45–80 °C) limita la capacidad de generalización en este rango crítico. Se podría explorar el uso de bases de datos complementarias a TEMPURA, como **BacDive** o **NCBI BioSample**, que ofrecen información térmica de crecimiento para un espectro más amplio de microorganismos.

    * **Integración de proteínas no canónicas:** Actualmente el dataset se centra en la subunidad grande (rbcL). La inclusión de la subunidad pequeña (rbcS) o de isoformas menos estudiadas de RuBisCO podría enriquecer el espacio de características y revelar nuevos patrones de estabilidad térmica.

    ### Caracterización estructural y funcional

    Una de las vías más prometedoras es la incorporación de información estructural. Aunque el proyecto se ha centrado exclusivamente en la secuencia primaria, el futuro podría traer:

    * **Integración con AlphaFold DB:** Añadir embeddings estructurales (por ejemplo, utilizando los modelos de Evoformer) como características adicionales. La estructura tridimensional de una proteína está íntimamente relacionada con su estabilidad térmica, por lo que esta información podría aumentar significativamente la precisión de los modelos.

    * **Anotaciones funcionales y de interacción:** Enriquecer el dataset con anotaciones de UniProt relativas a modificaciones post-traduccionales, sitios de unión a cofactores o dominios proteicos adicionales.

    ### Mejora de los modelos de aprendizaje automático

    La arquitectura de los modelos también puede ser objeto de una evolución significativa:

    * **Modelos fundacionales más grandes**: Actualmente se utiliza el modelo ESM-2 `t6_8M` (8 millones de parámetros) por su eficiencia computacional. Futuras iteraciones podrían evaluar modelos más profundos y de mayor capacidad, como el `esm2_t36_3B_UR50D` (3.000 millones de parámetros), que capturan representaciones semánticas mucho más ricas de las secuencias proteicas.

    * **Aprendizaje multitarea:** En lugar de predecir únicamente la `T_opt`, se podría entrenar el modelo para predecir simultáneamente `T_min`, `T_opt` y `T_max`. Esto no solo aprovecharía toda la información disponible, sino que forzaría al modelo a aprender relaciones más complejas entre la secuencia y el espectro térmico completo del organismo.

    * **Modelos basados en atención (Transformers):** El modelo 1D-CNN posicional ya es un gran avance, pero se podría implementar un **Transformer encoder** directamente sobre los embeddings posicionales, permitiendo que el modelo aprenda relaciones de largo alcance entre residuos distantes, un factor clave en la estabilidad de proteínas termófilas.

    ### Validación experimental (Benchmarking)

    La validación final de cualquier modelo predictivo en biología debe ser experimental. A medio plazo, se plantea:

    * **Colaboración con laboratorios húmedos:** Seleccionar un conjunto de proteínas de RuBisCO cuyas temperaturas óptimas hayan sido predichas por el modelo con alta confianza (y otras con baja confianza) y validarlas experimentalmente mediante ensayos de actividad enzimática a diferentes temperaturas.

    * **Análisis de mutaciones _in silico_:** Utilizar el modelo posicional para identificar qué residuos específicos contribuyen más a la estabilidad térmica y simular mutaciones puntuales para predecir cómo afectarían a la `T_opt`.

    Estas mejoras, aunque ambiciosas, son perfectamente alcanzables y permitirían evolucionar el proyecto desde una prueba de concepto hasta una herramienta de predicción robusta y de referencia para la biotecnología de enzimas termoestables.