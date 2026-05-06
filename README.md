# 🎮 Projet Steam — Analyse du marché des jeux vidéo

## 📇 Contexte

Ce projet a été réalisé pour **Ubisoft**, un éditeur français de jeux vidéo souhaitant lancer un nouveau jeu sur Steam. L'objectif est de conduire une analyse globale du marché des jeux vidéo disponibles sur Steam afin de mieux comprendre l'écosystème et les tendances actuelles.

L'analyse a été réalisée avec **Databricks** et **PySpark** sur un dataset de ~55 000 jeux Steam.

> Les notebooks sont accessibles directement sur Databricks via ce lien :
[Accéder au Workspace Databricks](https://dbc-ab2288c0-e937.cloud.databricks.com/browse/folders/1120015630262297?o=7474645418564237)

---

## 📁 Structure du projet

```
projet-steam-bloc2/
├── notebooks/
│   ├── 01_loading_cleaning.ipynb        # Notebook 01 : Chargement & Nettoyage
│   └── 02_eda_visualization.ipynb       # Notebook 02 : EDA & Visualisations
├── data/                                # Emplacement pour données locales éventuelles
├── images/
│   └── databricks-notebooks.png         # Capture du workspace Databricks (README)
├── outputs/
│   └── images/                          # Visualisations exportées depuis Databricks
│       ├── 1_1_top_publishers.png
│       ├── 1_2_best_rated_games.png
│       ├── 1_3_releases_by_year.png
│       ├── 1_4_free_vs_paid.png
│       ├── 1_4_price_distribution.png
│       ├── 1_4_discount.png
│       ├── 1_5_top_languages.png
│       ├── 1_6_age_restriction.png
│       ├── 2_1_top_genres.png
│       ├── 2_2_genres_positive_ratio.png
│       ├── 2_3_publisher_genres.png
│       ├── 2_4_genres_price.png
│       ├── 3_1_platforms.png
│       ├── 3_2_genres_platforms.png
│       ├── 4_1_genres_evolution.png
│       ├── 4_2_price_vs_reviews.png
│       ├── 4_3_top_games_owners.png
│       └── 4_4_multiplatform.png
├── .gitignore
├── LICENSE
└── README.md
```

---

## 🗂️ Dataset

- **Source** : `s3://full-stack-bigdata-datasets/Big_Data/Project_Steam/steam_game_output.json`
- **Format** : JSON semi-structuré (imbriqué)
- **Volume** : ~55 691 enregistrements
- **Période couverte** : 1997 — 2023

---

## 🔧 Notebook 01 — Data Exploration & Cleaning

### Transformations appliquées

| Champ brut | Problème détecté | Traitement appliqué |
|---|---|---|
| `data.*` | JSON imbriqué | `col("data").getField("champ")` |
| `platforms.*` | Struct doublement imbriqué | `getField("platforms").getField("windows")` |
| `price` / `initialprice` / `discount` | String | `cast(Float)` |
| `release_date` | Multi-formats : `"2000/11/1"`, `"2019/01"`, `"2019"` | `try_to_date` + `coalesce` |
| `required_age` | String avec formats variés (`"MA 15+"`) | `regexp_extract(\d+)` + `cast(Int)` |
| `owners` | Plage texte `"0 .. 20,000"` | `regexp_extract` + `cast(Long)` |
| `positive` / `negative` | Entiers bruts | Ratio calculé : `positive / total_reviews` |
| `genre` | String multi-valeurs `"Action, Indie"` | `split()` → `explode()` |
| `languages` | String multi-valeurs | `split()` → `array_size()` + `explode()` |
| `categories` | `ArrayType` JSON | `explode()` direct |

### DataFrames créés et sauvegardés en Delta Tables

| Table | Description |
|---|---|
| `steam_games` | DataFrame principal nettoyé (~55 690 jeux) |
| `steam_genres` | Une ligne par genre (via `explode`) |
| `steam_languages` | Une ligne par langue (via `explode`) |
| `steam_categories` | Une ligne par catégorie (via `explode`) |
| `steam_platforms` | Format long Windows / Mac / Linux (via `stack`) |
| `steam_publishers` | Agrégations par éditeur |

---

## 📊 Notebook 02 — EDA & Analyses

### Partie 1 — Analyse Macro

#### 1.1 Top Publishers
Ce visuel présente les éditeurs qui publient le plus grand nombre de jeux sur Steam.
![Top Publishers](outputs/images/1_1_top_publishers.png)
**Interprétation :** la visibilité est fortement captée par les éditeurs qui publient le plus de titres. Pour un nouveau jeu, il vaut mieux éviter la compétition frontale avec ces acteurs. Une proposition différenciante et une communication ciblée sont plus efficaces.

#### 1.2 Jeux les mieux notés
Ce visuel met en avant les jeux qui obtiennent les meilleurs retours des joueurs.
![Best Rated Games](outputs/images/1_2_best_rated_games.png)
**Interprétation :** les meilleurs jeux en note concentrent une forte confiance des joueurs. La priorité doit être la qualité perçue dès le lancement. De bons avis initiaux améliorent la conversion et la visibilité organique.

#### 1.3 Releases par année (Covid 2020-2021)
Ce visuel montre l'évolution du nombre de sorties de jeux par année.
![Releases by Year](outputs/images/1_3_releases_by_year.png)
**Interprétation :** le volume de sorties augmente au fil du temps, ce qui renforce la pression concurrentielle. Le choix de la fenêtre de lancement devient stratégique. Il faut accompagner la sortie avec un plan marketing solide.

#### 1.4 Distribution des prix
Ce premier graphique compare la part des jeux gratuits et payants.
![Free vs Paid](outputs/images/1_4_free_vs_paid.png)
**Interprétation :** la coexistence du gratuit et du payant révèle des attentes différentes selon les segments de joueurs. Un modèle free-to-play favorise l'acquisition massive. Un jeu payant doit afficher une valeur claire dès la page produit.

Ce deuxième graphique illustre la répartition des prix des jeux payants.
![Price Distribution](outputs/images/1_4_price_distribution.png)
**Interprétation :** la majorité des jeux se concentre sur une plage de prix spécifique. Se placer dans cette zone limite la friction à l'achat. Un prix plus élevé exige un positionnement premium bien justifié.

Ce troisième graphique synthétise la distribution des niveaux de promotion.
![Discount](outputs/images/1_4_discount.png)
**Interprétation :** les remises sont fréquentes et structurent les comportements d'achat sur Steam. Il faut prévoir des promotions régulières mais maîtrisées. Trop de remises peut affaiblir la perception de valeur du jeu.

#### 1.5 Langues les plus représentées
Ce visuel présente les langues les plus proposées sur les fiches de jeux.
![Top Languages](outputs/images/1_5_top_languages.png)
**Interprétation :** certaines langues dominent nettement la distribution des jeux. Prioriser ces langues augmente rapidement la portée internationale. Ensuite, une extension progressive optimise le budget de localisation.

#### 1.6 Restrictions d'âge
Ce visuel décrit la répartition des jeux selon les restrictions d'âge.
![Age Restriction](outputs/images/1_6_age_restriction.png)
**Interprétation :** la répartition des classes d'âge montre un équilibre entre contenus grand public et contenus matures. Viser un public large facilite l'acquisition. Un public mature peut mieux convertir si le positionnement est clair et assumé.

---

### Partie 2 — Analyse Genres

#### 2.1 Genres les plus représentés
Ce visuel classe les genres les plus présents dans le catalogue Steam.
![Top Genres](outputs/images/2_1_top_genres.png)
**Interprétation :** les genres les plus présents concentrent aussi le plus fort niveau de concurrence. Entrer sur ces genres demande une vraie différenciation. Sinon, cibler une niche peut offrir une meilleure traction initiale.

#### 2.2 Genres avec le meilleur ratio de reviews positives
Ce visuel compare les genres qui génèrent le plus de satisfaction joueur.
![Genres Positive Ratio](outputs/images/2_2_genres_positive_ratio.png)
**Interprétation :** certains genres génèrent un niveau de satisfaction joueur plus élevé. Miser sur ces genres peut améliorer la rétention et le bouche-à-oreille. C'est un bon signal pour la performance long terme.

#### 2.3 Genres favoris par publisher
Ce visuel met en relation les éditeurs et leurs genres de prédilection.
![Publisher Genres](outputs/images/2_3_publisher_genres.png)
**Interprétation :** les éditeurs se répartissent sur des genres où ils ont déjà une légitimité forte. Cette cartographie aide à éviter les zones trop verrouillées. Elle permet d'identifier des espaces de marché plus accessibles.

#### 2.4 Genres les plus lucratifs
Ce visuel compare les genres selon leur potentiel de revenus.
![Genres Price](outputs/images/2_4_genres_price.png)
**Interprétation :** tous les genres n'ont pas le même potentiel de monétisation. Les genres les plus rentables peuvent supporter plus d'investissement marketing. Il faut toutefois garder la cohérence entre prix, promesse et qualité perçue.

---

### Partie 3 — Analyse Plateformes

#### 3.1 Répartition Windows / Mac / Linux
Ce visuel montre la part des jeux compatibles avec chaque système.
![Platforms](outputs/images/3_1_platforms.png)
**Interprétation :** Windows domine largement la compatibilité des jeux sur Steam. Un lancement Windows-first est souvent le plus efficace. Le support Mac/Linux peut venir ensuite pour élargir la base utilisateurs.

#### 3.2 Genres par plateforme
Ce visuel détaille les genres les plus présents sur chaque plateforme.
![Genres Platforms](outputs/images/3_2_genres_platforms.png)
**Interprétation :** la distribution des genres varie selon la plateforme ciblée. Adapter le message marketing par plateforme améliore la pertinence. Le couple genre + plateforme doit guider le ciblage.

---

### Partie 4 — Analyses Complémentaires

#### 4.1 Evolution des genres dans le temps
Ce visuel retrace la dynamique des genres sur plusieurs années.
![Genres Evolution](outputs/images/4_1_genres_evolution.png)
**Interprétation :** certains genres progressent dans le temps alors que d'autres stagnent ou reculent. Il est plus sûr de se positionner sur une tendance en croissance. Cela augmente les chances de traction au lancement.

#### 4.2 Corrélation prix vs reviews positives
Ce visuel explore la relation entre niveau de prix et avis positifs.
![Price vs Reviews](outputs/images/4_2_price_vs_reviews.png)
**Interprétation :** la relation entre prix et avis positifs n'est pas linéaire. Un prix élevé peut fonctionner, mais seulement avec une forte proposition de valeur. Le positionnement tarifaire doit être cohérent avec l'expérience perçue.

#### 4.3 Top jeux par nombre de propriétaires
Ce visuel présente les jeux les plus diffusés en nombre de propriétaires.
![Top Games Owners](outputs/images/4_3_top_games_owners.png)
**Interprétation :** quelques titres concentrent une part importante des propriétaires. La visibilité est très polarisée autour des leaders. Un nouveau jeu doit compenser avec une activation acquisition plus intense.

#### 4.4 Jeux multi-plateformes vs popularité
Ce visuel compare la popularité des jeux selon leur couverture multi-plateformes.
![Multiplatform](outputs/images/4_4_multiplatform.png)
**Interprétation :** les jeux multi-plateformes tendent à toucher une audience plus large. Elargir la compatibilité peut renforcer la performance commerciale globale. Le coût de portage doit cependant rester aligné avec le potentiel de ventes.

---
🖥️ Aperçu des notebooks sur Databricks
> Les notebooks ont été développés et exécutés sur Databricks Free Edition. Voici un aperçu de l'environnement de travail :
![Aperçu des notebooks](images/databricks-notebooks.png)

> Vue du Workspace Databricks avec les deux notebooks 01_loading_cleaning et 02_eda_visualization dans le dossier projet-steam.
---

## 🛠️ Stack technique

- **Databricks** (Free Edition)
- **Apache Spark / PySpark**
- **Delta Tables**
- **AWS S3**

---

## 👤 Auteur

Projet réalisé dans le cadre du **Bloc 2 — Big Data** à Jedha Bootcamp.

- By RANJAKASOA Raphaël Marcellin
