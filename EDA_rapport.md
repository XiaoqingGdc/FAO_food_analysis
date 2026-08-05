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
Comparaison du nombre de zones par fichier : Vegetaux, Animaux et Population 
couvrent 175 pays chacun ; Cereales couvre 167 ; SousAlimentation couvre 204 zones.

**Vérification préalable :** les couvertures géographiques de `Vegetaux` et 
`Animaux` sont strictement identiques (mêmes 175 pays). La comparaison avec 
`Cereales` ci-dessous est donc valable indifféremment par rapport à l'un ou 
l'autre de ces deux fichiers.

**Détail Vegetaux/Animaux vs Cereales :** 8 pays présents dans Vegetaux et 
Animaux sont absents de Cereales : `Bermudes`, `Chine`, `Chine - RAS de Macao`, 
`Samoa`, `Polynésie française`, `Kiribati`, `Saint-Kitts-et-Nevis`, `Islande`. 
À l'inverse, aucun pays de Cereales n'est absent de Vegetaux/Animaux — Cereales 
constitue donc un **sous-ensemble strict** de leur couverture géographique.

**Hypothèse :** les pays manquants sont majoritairement de petits territoires 
insulaires à faible superficie agricole utile (Bermudes, Samoa, Kiribati, 
Saint-Kitts-et-Nevis, Polynésie française), ne disposant pas de production 
céréalière significative à déclarer à la FAO — une réalité agronomique plutôt 
qu'une lacune de collecte. Le cas de l'Islande s'explique par un climat peu 
propice à la céréaliculture. Le dédoublement `Chine` / `Chine - RAS de Macao` 
reflète une différence de granularité territoriale entre fichiers (cf. §2, 
où `Population` traite déjà la Chine comme un agrégat).

**Constat (Population vs SousAlimentation) :** l'ensemble des 175 pays de 
Population est intégralement inclus dans SousAlimentation (0 pays exclusif à 
Population). La relation d'inclusion est donc unidirectionnelle : 
Population ⊂ SousAlimentation. Les 29 zones supplémentaires de SousAlimentation 
s'expliquent par plusieurs causes distinctes :

- **micro-États insulaires** (Tuvalu, Nauru, Îles Cook, Îles Marshall, Tokélaou, 
  Samoa américaines, Micronésie, Nioué, Palaos, Comores, Seychelles, Andorre) : 
  population trop faible pour une estimation fiable côté Population, ou absence 
  de recensement standard
- **zones de conflit ou d'instabilité politique** (Libye, Soudan du Sud, 
  République arabe syrienne, Somalie, République démocratique du Congo, 
  Érythrée, Burundi) : capacités de collecte statistique limitées
- **territoires à statut politique particulier** (Palestine, Groenland, 
  Porto Rico, Tokélaou) : non comptabilisés comme pays indépendants dans 
  Population
- **économies à revenu élevé** (Singapour, Qatar, Bahreïn, Guinée équatoriale, 
  Bhoutan) : possiblement exclues de Population pour d'autres raisons de 
  méthodologie FAO à vérifier

Par conséquent, lors de la jointure finale (Étape 2), c'est la couverture de 
Population qui sera le facteur limitant — ces 29 zones seront naturellement 
exclues par un inner join sur Zone.

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
Fichier	Valeurs négatives	Valeurs atypiques élevées (> Q3+1.5×IQR)
Vegetaux	712	18 360
Animaux	35	6 338
Cereales	0	128
Population	0	19
SousAlimentation	0	56

Analyse des valeurs négatives (Vegetaux, 712 lignes ; Animaux, 35 lignes) :

Fichier	Variation de stock (légitime)	Disponibilité intérieure (suspect)	Anomalies avérées
Vegetaux	599 (84,1 %)	105 (14,7 %)	8 (1,1 %)
Animaux	29 (82,9 %)	5 (14,3 %)	1 (2,9 %)

La très large majorité des valeurs négatives correspond à l'Élément Variation de stock, pour lequel une valeur négative est cohérente sur le plan métier (déstockage net). La deuxième catégorie, Disponibilité intérieure, constitue une anomalie potentielle : un résultat négatif suggère des exportations disproportionnées par rapport à la production+import déclarés pour ce produit/pays.

Une minorité résiduelle constitue des anomalies avérées : 7 des 8 lignes concernées se rapportent à un seul enregistrement — Japon / Avoine — dont l'ensemble de la chaîne d'indicateurs dérivés (importations, exportations, nourriture, disponibilité par habitant) est négatif. Cela suggère une erreur en amont de la chaîne de calcul FAO (probablement sur les quantités importées/ exportées déclarées), propagée à tous les indicateurs qui en dépendent. La 8ᵉ ligne (Ouzbékistan / Fruits, Traitement = -4) constitue un point isolé indépendant. Le même schéma d'anomalie isolée est observé sur Animaux (1 ligne, Traitement, Hongrie/Viande de Bovins).

Décision de traitement (Étape 2) : les valeurs Variation de stock sont conservées. Les valeurs Disponibilité intérieure négatives sont marquées comme suspectes et exclues du calcul de la disponibilité calorique. L'enregistrement Japon / Avoine est exclu dans son intégralité du dataset (toute la chaîne d'indicateurs étant compromise), de même que les 2 lignes isolées (Ouzbékistan/Fruits, Hongrie/Viande de Bovins).

Analyse des valeurs atypiques élevées : un calcul IQR global détecte 18 360 valeurs atypiques (17,5 % du fichier). L'hypothèse initiale — un seuil trop sensible car mélangeant des Éléments à échelles incomparables — a été testée en recalculant l'IQR par Élément : le nombre de valeurs atypiques ne diminue que marginalement (16 798, soit 16,0 %), ce qui infirme largement cette hypothèse.

Conclusion révisée : la persistance d'un taux élevé d'atypiques même après segmentation par Élément indique que ces valeurs reflètent une caractéristique structurelle de la donnée elle-même : pour un Élément donné, un petit nombre de grands pays producteurs (ex. Chine, États-Unis, Inde) concentrent une part disproportionnée de la production/du commerce mondial, créant une distribution naturellement asymétrique (skewed) au sein même de chaque indicateur. Le critère IQR standard (Q3 + 1,5×IQR), conçu pour des distributions proches de la normale, n'est donc pas adapté pour qualifier ces valeurs d'"anomalies" — il capte surtout la concentration économique réelle du secteur agroalimentaire mondial plutôt que des erreurs de saisie. Une analyse au cas par cas (par Élément et par ordre de grandeur du pays) serait nécessaire pour distinguer d'éventuelles véritables erreurs au sein de ces 16 798 lignes, mais cela dépasse le cadre du diagnostic global de l'Étape 1.

## 8. Jointure — périmètre des fichiers sources et méthode retenue

Exclusion de Cereales : l'analyse de recoupement des produits (cf. §4) confirme que Cereales est un sous-ensemble de Vegetaux.

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

Le dataset global final compte 175 lignes × 8 colonnes — soit les 175 pays couverts en commun par Vegetaux, Animaux et Population, et 8 variables : 1 clé géographique (Zone) + 4 variables sources agrégées + 3 indicateurs dérivés calculés (Disponibilite_calorique_totale_kcal, Population_totale, Taux_sousnutrition_pct).

**Précision sur les 29 zones exclusives à `SousAlimentation` :** ces zones
 sont absentes de la base de 175 pays
(Vegetaux/Animaux/Population) et n'entrent donc jamais dans le dataset final,
quelle que soit la méthode de jointure retenue. 
Voici la liste de 29 pays exclus :{'Papouasie-Nouvelle-Guinée', 'Seychelles', 'Andorre', 'Palaos', 'Nauru', 'Guinée équatoriale', 'République démocratique du Congo', 'Nioué', 'Îles Cook', 'Soudan du Sud', 'Érythrée', 'Bahreïn', 'Îles Marshall', 'Somalie', 'Palestine', 'Samoa américaines', 'Micronésie (États fédérés de)', 'Bhoutan', 'Tokélaou', 'Burundi', 'Tuvalu', 'Libye', 'Porto Rico', 'Singapour', 'Comores', 'Qatar', 'République arabe syrienne', 'Tonga', 'Groenland'}
