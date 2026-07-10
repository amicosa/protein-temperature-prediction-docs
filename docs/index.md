# 🧬 Protein Thermal Stability Prediction using ESM-2 Embeddings

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
    C --> D[Alineamiento múltiple<br>+ proyección posicional]
    D --> E[Extracción ESM-2<br>embeddings por residuo]
    E --> F[Entrenamiento de modelos<br>RF, GB, SVR, Ensemble]
    F --> G[Evaluación y visualización]
```

---

## 🧪 Resultados clave (avance)

Dataset: ~2000 secuencias de RuBisCO curadas con datos térmicos.

Embeddings: Vectores de 768 dimensiones por residuo, proyectados sobre alineamiento de longitud fija.

Mejor modelo: Ensemble (RF + GB + SVR) con:

RMSE: ±8.5 °C (en validación)

R²: 0.72 (en test)