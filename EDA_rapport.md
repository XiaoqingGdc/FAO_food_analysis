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

Décision de traitement : compte tenu du taux de valeurs manquantes extrêmement élevé (99,43 %) et du fait qu'il ne s'agit pas d'une anomalie mais d'une particularité structurelle du fichier la colonne Symbole est supprimée du fichier Population avant intégration au dataset global

---

## 3. Doublons

Aucun doublon n'a été détecté sur la ligne complète, ni sur la clé métier 
(`Zone`, `Produit`, `Élément`), pour l'ensemble des 5 fichiers. Cela confirme 
que chaque combinaison pays/produit/indicateur n'apparaît qu'une seule fois 
par fichier, ce qui est cohérent avec la structure attendue d'un export FAO.

---

## 4. Couverture géographique
Comparaison du nombre de zones par fichier : Vegetaux, Animaux et Population 
couvrent 175 pays chacun ; Cereales couvre 167 ; SousAlimentation couvre 204 zones.


**Constat (Population vs SousAlimentation) :** l'ensemble des 175 pays de 
Population est intégralement inclus dans SousAlimentation.

Par conséquent, lors de la jointure finale (Étape 2), c'est la couverture de 
Population qui sera le facteur limitant — ces 29 zones seront naturellement 
exclues par un inner join sur Zone.

---

## 5. Unités de mesure

Les unités identifiées dans les fichiers de production (Vegetaux, Animaux) sont : 
`Milliers de tonnes`, `kg`, `Kcal/personne/jour`, `g/personne/jour`. 

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
Fichier	Valeurs négatives	Valeurs atypiques élevées (> Q3+1.5×IQR)
Vegetaux	712	18 360
Animaux	35	6 338
Cereales	0	128
Population	0	19
SousAlimentation	0	56

Analyse des valeurs négatives (Vegetaux, 712 lignes ; Animaux, 35 lignes).

La très large majorité des valeurs négatives correspond à l'Élément Variation de stock, pour lequel une valeur négative est cohérente sur le plan métier (déstockage net). La deuxième catégorie, Disponibilité intérieure, constitue une anomalie potentielle : un résultat négatif suggère des exportations disproportionnées par rapport à la production+import déclarés pour ce produit/pays.

## 8. Jointure — périmètre des fichiers sources et méthode retenue

Exclusion de Cereales : l'analyse de recoupement des produits confirme que Cereales est un sous-ensemble de Vegetaux.

Décision : seuls Vegetaux, Animaux, Population et SousAlimentation sont utilisés pour construire le dataset global.

Méthode de jointure retenue — jointure en deux temps :

Inner join entre Vegetaux, Animaux et Population : ces trois fichiers partagent une couverture géographique strictement identique (175 pays).

Left join de SousAlimentation sur cette base : le left join conserve donc l'intégralité des 175 pays ; les pays sans mesure fiable de sous-nutrition reçoivent une valeur NaN sur SousAlimentation_millions 

**Left join de `SousAlimentation` sur cette base : un inner join aurait donc supprimé des
pays disposant pourtant de données complètes sur les trois autres sources —
une perte injustifiée. Le left join conserve les 175 pays : ceux sans mesure
fiable reçoivent un `NaN` sur `SousAlimentation_millions` et
`Taux_sousnutrition_pct`.


## 9. Dataset final

Le dataset_global final compte 171 lignes — soit les 171 pays couverts en commun par Vegetaux, Animaux et Population, et 9 variables

**Précision sur les 33 zones exclusives à `SousAlimentation` :** ces zones sont absentes de la base de 171 pays
(Vegetaux/Animaux/Population) et n'entrent donc jamais dans le dataset final,
quelle que soit la méthode de jointure retenue. 
Voici la liste de 29 pays exclus :{'Papouasie-Nouvelle-Guinée', 'Seychelles', 'Andorre', 'Palaos', 'Nauru', 'Guinée équatoriale', 'République démocratique du Congo', 'Nioué', 'Îles Cook', 'Soudan du Sud', 'Érythrée', 'Bahreïn', 'Îles Marshall', 'Somalie', 'Palestine', 'Samoa américaines', 'Micronésie (États fédérés de)', 'Bhoutan', 'Tokélaou', 'Burundi', 'Tuvalu', 'Libye', 'Porto Rico', 'Singapour', 'Comores', 'Qatar', 'République arabe syrienne', 'Tonga', 'Groenland'}

Nettoyage complémentaire — doublon Chine : un examen du fichier Population révèle que la Chine apparaît sous 5 lignes distinctes : Chine (total), Chine, continentale, Chine - RAS de Hong-Kong, Chine - RAS de Macao, et Chine, Taiwan Province de. Une vérification arithmétique confirme que Population_totale de la ligne Chine est exactement la somme des 4 autres lignes (1 416 667 000 = 1 385 567 000 + 7 204 000 + 566 000 + 23 330 000) : la ligne Chine est donc un agrégat des 4 sous-zones, et non une zone indépendante. 

Décision : les 4 lignes de détail (Chine, continentale, Chine - RAS de Hong-Kong, Chine - RAS de Macao, Chine, Taiwan Province de) sont supprimées du dataset, au profit de la ligne agrégée Chine, qui dispose par ailleurs d'une valeur de Taux_sousnutrition_pct exploitable, 175-4 soit 171 zone final.