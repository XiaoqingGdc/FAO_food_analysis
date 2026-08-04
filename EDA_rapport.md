# Étape 1 — Diagnostic qualité des données

## 1. Vue d'ensemble des 5 fichiers

| Fichier | Lignes | Colonnes | Pays couverts | Doublons |Valeur manquante
|---|---|---|---|---|---|
| Vegetaux | 104871 | 14 | 175 | 0 |0
| Animaux | 37166 | 14 | 175 | 0 |0
| Cereales | 891 | 14 | 167 | 0 |0
| Population | 175 | 14 | 175 | 0 |174
| SousAlimentation | 1 020 | 15 | 204| 0 |415

**Constat général :**: l'absence de doublons sur les 5 fichiers suggère une bonne qualité de collecte côté FAO (pas de double-saisie). Le fichier SousAlimentation se distingue par une colonne supplémentaire (Note), probablement destinée à des annotations méthodologiques sur le calcul du taux de sous-nutrition, et par un format d'année en période triennale (ex. 2012-2014) plutôt qu'en année unique — à la différence de Vegetaux et Population, qui ne couvrent chacun qu'une seule année (2013). Ce point est déterminant pour la jointure prévue à l'Étape 2 : la période de référence 2012-2014 de SousAlimentation a été retenue par le corps enseignant comme la tranche à utiliser pour le rapprochement avec les données 2013 des autres fichiers.

---

## 2. Valeurs manquantes

Vegetaux, Animaux et Cereales ne présentent **aucune valeur manquante** 

Le fichier **Population** présente en revanche un cas notable : la colonne `Symbole` 
est manquante sur **174 des 175 lignes (99,43 %)**. 

Décision de traitement (Étape 2) : compte tenu du taux de valeurs manquantes extrêmement élevé (99,43 %) et du fait qu'il ne s'agit pas d'une anomalie mais d'une particularité structurelle du fichier la colonne Symbole est supprimée du fichier Population avant intégration au dataset global


SousAlimentation — colonne Valeur : sur 1 020 lignes, 415(230+185) présentent une valeur manquante (40,7 %), entièrement expliquées par la colonne Symbole.

Symbole	Nombre de lignes	Signification
F	605	Valeur estimée (exploitable)
NR	230	Non rapporté — pas de donnée disponible
NV	185	Valeur non disponible


Sur la période de référence retenue (2012-2014):
F     120
NR     47
NV     37
Lignes avec valeur exploitable (Symbole='F') : 120
---

## 3. Doublons

Aucun doublon n'a été détecté sur la ligne complète, ni sur la clé métier 
(`Zone`, `Produit`, `Élément`), pour l'ensemble des 5 fichiers. Cela confirme 
que chaque combinaison pays/produit/indicateur n'apparaît qu'une seule fois 
par fichier, ce qui est cohérent avec la structure attendue d'un export FAO.

---

## 4. Couverture géographique
Comparaison du nombre de zones par fichier:Vegetaux, Animaux et Population couvrent 175 pays chacun ; Cereales couvre 167, SousAlimentation couvre 204 zones.

**Détail Animaux vs Cereales :** 8 pays présents dans Animaux sont absents de 
Cereales. À l'inverse, aucun pays 
de Cereales n'est absent d'Animaux — Cereales constitue donc un **sous-ensemble strict** 
de la couverture géographique d'Animaux.

**Hypothèse :** les pays manquants sont probablement de petits territoires insulaires 
ou à faible superficie agricole utile, ne disposant pas de production céréalière 
significative à déclarer à la FAO. Cette absence reflèterait donc une réalité 
agronomique plutôt qu'une lacune de collecte.

**HConstat :**  l'ensemble des 175 pays de Population est intégralement inclus
dans SousAlimentation (0 pays exclusif à Population). La relation
d'inclusion est donc unidirectionnelle : Population ⊂ SousAlimentation.
Par conséquent, lors de la jointure finale (Étape 2), c'est la couverture de
Population qui sera le facteur limitant — les 29 zones supplémentaires de
SousAlimentation (micro-États, zones de conflit, territoires à statut
particulier) seront naturellement exclues par un inner join sur Zone.

---

## 5. Unités de mesure

Les unités identifiées dans les fichiers de production (Vegetaux, Animaux) sont : 
`Milliers de tonnes`, `kg`, `Kcal/personne/jour`, `g/personne/jour`. Ces unités 
varient selon l'Élément mesuré (production brute vs disponibilité par habitant), 
ce qui est cohérent avec la nature multi-indicateurs de ces fichiers.

Population : unité unique, 1000 personnes — cohérent avec un fichier à un seul type d'indicateur.

SousAlimentation : contrairement aux fichiers Vegetaux/Animaux (plusieurs unités selon l'Élément), ce fichier ne contient qu'un seul type d'Élément (Valeur), exprimé en millions de personnes.
---

## 6. Fiabilité des données (colonne Symbole)
La colonne `Symbole` indique la nature de chaque donnée selon la nomenclature FAO :
`S` = Standardisé (donnée officielle directement rapportée), `Fc` = Calculé
(estimation dérivée par la FAO, ex. via un bilan production/import/export), et
`A` = Agrégat (valeur combinant potentiellement plusieurs sources).

| Fichier | S (Standardisé) | Fc (Calculé) | A (Agrégat) |
|---|---|---|---|
| Vegetaux | 64,06 % | 35,37 % | 0,57 % |
| Animaux | 59,53 % | 40,01 % | 0,46 % |
| Cereales | 100,0 % | — | — |


---

## 7. Valeurs extrêmes

| Fichier | Valeurs négatives | Valeurs atypiques élevées (> Q3+1.5×IQR) |
|---|---|---|
| Vegetaux | 712 | 18 360 |
| Animaux | 35 | 6 338 |
| Cereales | 0 | 128 |
| Population | 0 | 19 |
| SousAlimentation | 0 | 56 |

**Analyse des valeurs négatives (Vegetaux, 712 lignes) :** un examen des colonnes 


**Analyse des valeurs atypiques élevées :** le volume important (18 360 lignes pour 
Vegetaux, soit environ 17,5 % du fichier) suggère que le seuil IQR classique est 
probablement trop sensible sur une variable comme `Valeur`, dont la distribution 
est naturellement très étalée (un même Élément peut aller de quelques kg pour un 
petit pays à plusieurs millions de tonnes pour un grand pays producteur). 
**Conclusion :** la majorité de ces valeurs "atypiques" reflètent probablement 
des écarts de taille entre pays plutôt que des erreurs de saisie



 