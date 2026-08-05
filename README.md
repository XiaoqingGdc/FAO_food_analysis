# FAO Food Analysis — Sous-nutrition mondiale

## Structure du projet
- `FAO/` : fichiers sources bruts (5 CSV)
- `FAO_datalab_analysis.ipynb` : Étape 1 — diagnostic qualité + jointure
- `FAO_datalab_exploration.ipynb` : Étape 2 — analyse exploratoire
- `EDA_rapport.md` : rapport de diagnostic détaillé
- `dataset_global_fao.csv` : dataset final nettoyé (171 pays)
- `rapport_eda_auto.html` : profiling automatisé complémentaire (ydata-profiling)

```
FAO_food_analysis/
├── FAO/                          # Fichiers CSV bruts (5 fichiers sources)
├── .gitignore                    # Exclut le dossier venv
├── EDA_rapport.md                # Rapport de diagnostic — Étape 1 (J1)
├── FAO_datalab_analysis.ipynb    # Notebook principal — Étape 1 (J1)
├── FAO_datalab_exploration.ipynb # Notebook d'analyse exploratoire — Étape 2 (J2)
├── FAO_profiling_auto.ipynb      # Complément : profiling automatisé (Colab)
├── dataset_global_fao.csv        # Dataset global nettoyé (171 pays)
├── rapport_eda_auto.html         # Rapport EDA automatisé (ydata-profiling)
└── requirements.txt              # Dépendances du projet
```

## Installation

```bash
pip install -r requirements.txt
```