# Changelog

Toutes les modifications notables apportées à ce fork sont documentées ici.

Ce fork est basé sur [pyrech/homewizard_cloud_watermeter](https://github.com/pyrech/homewizard_cloud_watermeter)
(dernière synchronisation upstream : commit `48008d6`). Les entrées ci-dessous
décrivent uniquement les ajouts/modifications apportés dans **ce fork**
(`damien-er/homewizard_cloud_watermeter`) — pour l'historique upstream, voir
le dépôt d'origine.

La numérotation suit [Semantic Versioning](https://semver.org/lang/fr/).
`v1.x.x` couvre l'ensemble des ajouts liés au capteur flow_rate (solution
transitoire tant que le Watermeter reste alimenté par pile). `v2.0.0` sera
réservé à une éventuelle refonte majeure ou à la bascule vers l'intégration
officielle temps réel (une fois le device raccordé au secteur).

## [v1.3.0] - 2026-09-14

### Ajouté
- Historique détaillé du débit en Long Term Statistics, sous un nouveau
  `statistic_id` (`homewizard_cloud_watermeter:<device>_flow_rate_5min`),
  consultable via le panneau Historique de Home Assistant ou une carte
  `statistics-graph`.
- Résolution alignée sur 5 minutes, pour correspondre à la résolution du
  graphique "Jour" de l'application HomeWizard (confirmé via la documentation
  officielle HomeWizard).
- Injection de chaque tranche de 5 min, y compris les tranches à
  consommation nulle, pour obtenir un graphique continu avec de vrais creux
  plutôt qu'un simple relevé de pics.

### Modifié
- `api.py` : paramètre `gb` passé de `"15m"` à `"5m"` dans la requête à
  l'API cloud HomeWizard.
- `coordinator.py` : tous les calculs de débit (`L/bucket → L/min`) ajustés
  en conséquence (division par 5 au lieu de 15).

### Note technique
Le device physique, sur pile, ne synchronise avec le cloud que ~4 fois par
jour. Les points à 5 min entre deux synchronisations réelles restent donc
de l'interpolation linéaire côté API cloud (`"fill": "linear"`) — comme
dans l'application HomeWizard elle-même — et non une mesure indépendante
toutes les 5 minutes.

## [v1.2.0] - 2026-09-13

### Corrigé
- Le capteur `flow_rate` restait bloqué à `0.0 L/min` après minuit malgré
  une consommation réelle la veille. Cause : la recherche du dernier bucket
  non-nul s'arrêtait sur la première tranche à `0.0` rencontrée côté
  aujourd'hui (remplie par interpolation, donc non-`null`), sans regarder
  plus loin dans les données d'hier.
- Séparation de la logique en deux boucles distinctes : une pour
  `last_sync_at` (comportement inchangé, inclut les zéros), une nouvelle
  pour `last_flow_rate`/`last_flow_rate_at` qui ignore les buckets à zéro
  pour retrouver la dernière consommation réelle, même si elle date de la
  veille.

## [v1.1.0] - 2026-09-12

### Corrigé
- Trois automatisations Home Assistant recalibrées pour le rythme de
  synchronisation réel du device sur pile (~4x/jour, toutes les ~6h) :
  détection de capteur silencieux, alerte fuite d'eau (seuils reformulés
  en L/h), alerte fuite d'eau critique.

## [v1.0.0] - 2026-09-XX

### Ajouté
- Nouveau capteur `HomeWizardFlowRateSensor` (device_class
  `volume_flow_rate`, unité L/min, state_class `measurement`), calculé à
  partir du dernier bucket de 15 min non-nul renvoyé par l'API cloud.
- Capteur diagnostic associé `HomeWizardFlowRateTimestampSensor`
  (désactivé par défaut), horodatage du dernier calcul.
- Aucune modification de `api.py` à ce stade (`gb: "15m"` conservé tel
  quel depuis l'upstream).
