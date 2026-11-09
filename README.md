# Segmentation clients pour le ciblage marketing

Segmentation non supervisée de 2 240 clients d'une enseigne de distribution, afin d'adapter les campagnes marketing à chaque profil de client.

**Techniques :** feature engineering, transformation logarithmique, PCA, KMeans, classification ascendante hiérarchique (CAH), DBSCAN, évaluation par métriques internes et stabilité (ARI), pipeline scikit-learn.

## Contexte

L'enseigne envoie les mêmes campagnes à tous ses clients, alors que leurs habitudes d'achat sont très différentes. L'objectif est d'identifier des groupes de clients au comportement similaire, puis de traduire ces groupes en recommandations marketing concrètes.

## Données

[Customer Personality Analysis](https://www.kaggle.com/datasets/imakash3011/customer-personality-analysis) (Kaggle) : 2 240 clients, 29 variables (profil socio-démographique, dépenses par catégorie sur 2 ans, canaux d'achat, réponses à 6 campagnes).

Le fichier n'est pas inclus dans le dépôt. Pour reproduire le projet, téléchargez `marketing_campaign.csv` et placez-le dans `data/`.

## Démarche

1. **Nettoyage :** imputation du revenu par la médiane, suppression de 4 clients aberrants, regroupement des catégories rares.
2. **Feature engineering :** âge, ancienneté, dépenses totales, composition du foyer, nombre de campagnes acceptées.
3. **Choix des variables :** clustering uniquement sur les 13 variables **comportementales** ; les variables socio-démographiques sont réservées au profilage, ce qui permet de valider les segments de façon indépendante.
4. **Préparation :** transformation `log1p` des variables asymétriques, standardisation, PCA (5 composantes, 82 % de la variance).
5. **Modélisation et comparaison :** KMeans, CAH et DBSCAN, évalués par silhouette, Davies-Bouldin, Calinski-Harabasz et stabilité (bootstrap, ARI).
6. **Profilage** des segments et recommandations marketing.

## Résultats

**Modèle retenu :** KMeans, k = 4 (silhouette 0,28 ; stabilité bootstrap : ARI moyen 0,97).

| Segment | Clients | Chiffre d'affaires | Profil |
|---|---|---|---|
| Petits acheteurs | 37 % | ≈ 4 % | Jeunes familles, revenu le plus faible, achats en magasin, insensibles aux campagnes |
| Chasseurs de promos | 23 % | ≈ 19 % | Familles avec adolescents, panier dominé par le vin, champions des promotions et du web |
| Gros acheteurs autonomes | 25 % | ≈ 42 % | Revenu élevé, achats en magasin, imperméables aux campagnes |
| Premium engagés | 14 % | ≈ 35 % | Revenu le plus élevé, peu d'enfants, catalogue, seuls réceptifs aux campagnes (jusqu'à 57 %) |

**Principaux enseignements :**
- 39 % des clients génèrent environ 77 % du chiffre d'affaires.
- Les campagnes actuelles ne fonctionnent que sur le segment premium ; le segment qui pèse le plus en chiffre d'affaires n'y répond pas.
- Les variables socio-démographiques, absentes du clustering, diffèrent nettement entre segments, ce qui valide leur pertinence.

## Choix techniques notables

- **k = 4 plutôt que le meilleur silhouette (k = 2) :** sur des données en continuum, les métriques internes favorisent toujours le plus petit nombre de segments. k = 4 est le dernier k avant la chute du silhouette, et il isole le segment premium réceptif aux campagnes.
- **Stabilité :** avec une seule initialisation, KMeans k = 4 tombe souvent dans des minimums locaux (ARI minimum 0,46) ; avec `n_init=10`, la segmentation devient reproductible.
- **DBSCAN non retenu :** le nuage de clients est continu, sans zones séparées par du vide. DBSCAN produit soit un seul segment, soit une fragmentation ; il a été utilisé pour repérer les clients atypiques.

## Structure du projet

~~~text
├── data/                            # données (non versionnées)
├── models/                          # pipeline sauvegardé (joblib)
├── notebooks/
│   └── segmentation_clients.ipynb   # analyse complète, du nettoyage au pipeline
├── requirements.txt                 # dépendances
└── README.md
~~~


## Installation et utilisation

1. Cloner le dépôt :
   ```bash
   git clone https://github.com/hilaryChoco/customer-segmentation-clustering.git
   cd customer-segmentation-clustering
   ```

2. Créer un environnement et installer les dépendances :

   **Avec conda :**
   ```bash
   conda create -n segmentation python=3.11.15
   conda activate segmentation
   pip install -r requirements.txt
   ```

   **Ou avec venv :**
   ```bash
   python -m venv .venv
   .venv\Scripts\activate        # Windows
   source .venv/bin/activate     # macOS / Linux
   pip install -r requirements.txt
   ```

3. Télécharger `marketing_campaign.csv` depuis [Kaggle](https://www.kaggle.com/datasets/imakash3011/customer-personality-analysis) et le placer dans `data/`.

4. Lancer Jupyter et ouvrir `notebooks/segmentation_clients.ipynb` :
   ```bash
   jupyter lab
   ```

Pour affecter de nouveaux clients à un segment, voir la fonction `predire_segment` (section XIII).

## Limites

- Les segments découpent un continuum : un client proche d'une frontière peut changer de segment.
- Données d'une seule enseigne sur 2 ans, à mettre à jour régulièrement.
- Les recommandations sont des hypothèses à valider par A/B testing.
