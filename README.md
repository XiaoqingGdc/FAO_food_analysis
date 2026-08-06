# FAO Food Analysis — Sous-nutrition mondiale


## Structure du projet
- `FAO/` : fichiers sources bruts (5 CSV)
- `data/` : datasets nettoyés/dérivés (dataset global, version imputée)
- `img/` : cartographies exportées (avant/après correction des valeurs négatives)
- `FAO_datalab_analysis.ipynb` : Étape 1 (J1) — diagnostic qualité + jointure
- `FAO_datalab_exploration.ipynb` : Étape 2 (J2) — analyse exploratoire
- `FAO_datalab_modelisation.ipynb` : Étape 3 (J3) — régression + ACP
- `FAO_datalab_clustering.ipynb` : Étape 4 (J4) — clustering K-Means + restitution
- `FAO_profiling_auto.ipynb` : profiling automatisé complémentaire (Colab)
- `EDA_rapport.md` : rapport de diagnostic détaillé
- `rapport_eda_auto.html` : profiling automatisé (ydata-profiling)

```
FAO_food_analysis/
├── FAO/ # Fichiers CSV bruts (5 fichiers sources)
├── data/
│ ├── dataset_global_fao.csv # Dataset global nettoyé (Étape 1)
│ └── dataset_global_fao_impute.csv # Dataset avec Taux_sousnutrition_pct imputé (Étape 3)
├── img/
│ ├── map_kcal.png # Carte disponibilité calorique
│ └── map_sousnutrition.png # Carte taux de sous-nutrition
├── .gitignore # Exclut le dossier venv
├── EDA_rapport.md # Rapport de diagnostic — Étape 1 (J1)
├── FAO_datalab_analysis.ipynb # Notebook principal — Étape 1 (J1)
├── FAO_datalab_exploration.ipynb # Notebook d'analyse exploratoire — Étape 2 (J2)
├── FAO_datalab_modelisation.ipynb # Notebook régression + ACP — Étape 3 (J3)
├── FAO_datalab_clustering.ipynb # Notebook clustering + restitution — Étape 4 (J4)
├── FAO_profiling_auto.ipynb # Complément : profiling automatisé (Colab)
├── rapport_eda_auto.html # Rapport EDA automatisé (ydata-profiling)
└── requirements.txt # Dépendances du projet
```

## Installation

```bash
pip install -r requirements.txt
```