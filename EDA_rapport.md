# Étape 1 — Diagnostic qualité des données

## 1. Vue d'ensemble des 5 fichiers

| Fichier | Lignes | Colonnes | Pays couverts | Doublons |
|---|---|---|---|---|
| Vegetaux | 104871 | 14 | 175 | 0 |
| Animaux | 37166 | 14 | 175 | 0 |
| Cereales | 891 | 14 | 167 | 0 |
| Population | 175 | 14 | 175 | 0 |
| SousAlimentation | 1 020 | 15 | 204| 0 |

**Constat général :** l'absence de doublons sur les 5 fichiers suggère une bonne qualité 
de collecte côté FAO (pas de double-saisie). Le fichier SousAlimentation se distingue 
par une colonne supplémentaire (`Note`), probablement destinée à des annotations 
méthodologiques sur le calcul du taux de sous-nutrition.

---

## 2. Valeurs manquantes

Vegetaux, Animaux et Cereales ne présentent **aucune valeur manquante** sur l'ensemble 
de leurs colonnes, ce qui est remarquable compte tenu du volume de données 
(plus de 100 000 lignes pour Vegetaux).

Le fichier **Population** présente en revanche un cas notable : la colonne `Symbole` 
est manquante sur **174 des 175 lignes (99,43 %)**. Hypothèse : contrairement aux 
fichiers de production/consommation, les données de population ne nécessitent 
généralement pas de code de fiabilité par ligne (une seule valeur par pays), 
d'où cette quasi-absence de remplissage — il ne s'agit probablement pas d'une 
anomalie de collecte mais d'une particularité structurelle de ce fichier.

résultat pour SousAlimentation  15 colonne concernée, nombre et 0 % de NaN, 
répartition par période si applicable.

---

## 3. Doublons

Aucun doublon n'a été détecté sur la ligne complète, ni sur la clé métier 
(`Zone`, `Produit`, `Élément`), pour l'ensemble des 5 fichiers. Cela confirme 
que chaque combinaison pays/produit/indicateur n'apparaît qu'une seule fois 
par fichier, ce qui est cohérent avec la structure attendue d'un export FAO.

---

## 4. Couverture géographique


**Détail Animaux vs Cereales :** 8 pays présents dans Animaux sont absents de 
Cereales (dont Bermudes [À COMPLÉTER la liste complète]). À l'inverse, aucun pays 
de Cereales n'est absent d'Animaux — Cereales constitue donc un **sous-ensemble strict** 
de la couverture géographique d'Animaux.

**Hypothèse :** les pays manquants sont probablement de petits territoires insulaires 
ou à faible superficie agricole utile, ne disposant pas de production céréalière 
significative à déclarer à la FAO. Cette absence reflèterait donc une réalité 
agronomique plutôt qu'une lacune de collecte.

[À COMPLÉTER : effectuer et documenter les comparaisons restantes, notamment avec 
Population et SousAlimentation]

---

## 5. Unités de mesure

Les unités identifiées dans les fichiers de production (Vegetaux, Animaux) sont : 
`Milliers de tonnes`, `kg`, `Kcal/personne/jour`, `g/personne/jour`. Ces unités 
varient selon l'Élément mesuré (production brute vs disponibilité par habitant), 
ce qui est cohérent avec la nature multi-indicateurs de ces fichiers.

[À COMPLÉTER : résultat de `coherence_element_unite()` — a-t-on détecté des 
Éléments sans unité associée, ou des incohérences entre le libellé et l'unité affichée ?]

---

## 6. Fiabilité des données (colonne Symbole)

[À COMPLÉTER : résultats de `fiabilite_symbole()` pour chaque fichier — 
% de données officielles (A) vs estimées (E) vs calculées (X) vs imputées (I). 
Conclure sur le niveau de confiance à accorder à chaque fichier.]

---

## 7. Valeurs extrêmes

| Fichier | Valeurs négatives | Valeurs atypiques élevées (> Q3+1.5×IQR) |
|---|---|---|
| Vegetaux | 712 | 18 360 |
| Animaux | 35 | 6 338 |
| Cereales | 0 | [À COMPLÉTER] |
| Population | 0 | 19 |
| SousAlimentation | [À COMPLÉTER] | [À COMPLÉTER] |

**Analyse des valeurs négatives (Vegetaux, 712 lignes) :** un examen des colonnes 
`Élément` associées montre que [À COMPLÉTER : après avoir affiché 
`extremes_vegetaux[['Zone','Produit','Élément','Valeur']].head(20)`, préciser si 
les négatifs correspondent majoritairement à un Élément du type "solde commercial" 
ou "variation de stock" — auquel cas ce sont des valeurs métier légitimes — ou à 
un Élément type "production"/"disponibilité", auquel cas ce sont des anomalies 
à traiter en Étape 2].

**Analyse des valeurs atypiques élevées :** le volume important (18 360 lignes pour 
Vegetaux, soit environ 17,5 % du fichier) suggère que le seuil IQR classique est 
probablement trop sensible sur une variable comme `Valeur`, dont la distribution 
est naturellement très étalée (un même Élément peut aller de quelques kg pour un 
petit pays à plusieurs millions de tonnes pour un grand pays producteur). 
**Conclusion :** la majorité de ces valeurs "atypiques" reflètent probablement 
des écarts de taille entre pays plutôt que des erreurs de saisie



---

### Unités de mesure — SousAlimentation

Contrairement aux fichiers Vegetaux/Animaux (plusieurs unités selon l'Élément), 
le fichier SousAlimentation ne contient qu'un seul type d'Élément (`Valeur`), 
exprimé en **millions** (de personnes). 

**Implication importante :** ce fichier fournit le **nombre absolu de personnes 
en sous-nutrition**, et non un taux/pourcentage directement exploitable. Le calcul 
du "taux de sous-nutrition (%)" demandé à l'Étape 2 nécessitera donc une jointure 
avec le fichier Population, selon la formule :

taux_sousnutrition (%) = (Valeur_sousalim en millions × 1 000 000) / Population totale × 100

 