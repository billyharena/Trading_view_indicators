# Indicateurs TradingView

## XAUUSD : aide à la décision en scalping

`xauusd_proba.pine` est un indicateur Pine Script v6 dans un panneau séparé.
Il conserve la régression logistique en ligne initiale et lui ajoute une décision
filtrée, des marqueurs et des alertes à la clôture.

### Installation et lecture

1. Copier le script dans l'éditeur Pine de TradingView, enregistrer et ajouter au graphique XAUUSD.
2. Utiliser des bougies temporelles standard ; M1 et M5 sont des unités à évaluer en simulation, pas des réglages optimisés.
3. Attendre la normalisation des cinq variables, puis 100 exemples d'apprentissage par défaut.
4. Lire la première ligne du tableau : **ACHAT**, **VENTE** ou **ATTENDRE**, suivie du motif.
5. Créer les alertes « Scalp achat confirmé » et « Scalp vente confirmé », avec la fréquence « Une fois par clôture de barre ».

La décision concerne uniquement le mouvement entre la clôture du signal et la
clôture suivante. Pendant une bougie ouverte, le tableau conserve la dernière
décision clôturée et son heure UTC ; elle ne constitue pas un nouveau signal.
Les alertes, marqueurs et statistiques utilisent exactement les mêmes signaux.
Recréer les alertes TradingView après modification du code ou des paramètres.

### Conditions d'émission

| Condition | Valeur initiale |
| --- | --- |
| Score du modèle | Hausse ≥ 55 % pour achat ; hausse ≤ 45 % pour vente |
| Tendance achat | Clôture > EMA 20 > EMA 50 |
| Tendance vente | Clôture < EMA 20 < EMA 50 |
| Extension maximale | Distance absolue clôture–EMA 20 ≤ 1,5 ATR |
| Amplitude maximale | Haut–bas ≤ 2,5 ATR |
| Pause | 3 barres complètes sans signal après chaque émission, dans les deux sens |

Le filtre de tendance peut être désactivé. Les limites d'extension et d'amplitude
sont réglables et utilisent l'ATR 14 de la barre précédente pour qu'une bougie
extrême ne relève pas son propre seuil. À contexte identique, un signal peut
être réémis après la pause ; il n'est pas limité aux croisements de seuil.
Ces paramètres sont des valeurs de départ, sans optimisation sur des données XAUUSD.

### Corrections et portée des statistiques

- L'apprentissage et les décisions ne se font qu'à la clôture. Les variables de
  la barre précédente sont associées au résultat désormais connu, puis le modèle
  prédit la barre suivante. Il n'utilise aucune donnée future.
- La disponibilité réelle des cinq variables remplace un délai arbitraire ;
  les z-scores sont bornés à ±3 et l'entrée de la sigmoïde à ±30 pour limiter
  l'effet des valeurs extrêmes et les débordements numériques.
- Une clôture inchangée reçoit une cible neutre de 0,5 pendant l'apprentissage.
  Elle ne compte jamais comme succès directionnel dans les statistiques.
- La précision des signaux mesure seulement les alertes réellement émises,
  filtres et pause compris, une fois leur bougie cible clôturée. L'effectif
  indique le nombre de signaux évalués, pas le nombre de trades gagnants.
- La précision globale et la fréquence de hausse utilisent le même ensemble
  de prédictions évaluables. Les statistiques couvrent les dernières 300 barres
  par défaut ; « — » signifie historique insuffisant ou absence d'échantillon.

Le score logistique n'est pas une probabilité de gain calibrée. La précision
directionnelle ne mesure ni les frais, ni le spread, ni le slippage, ni le
rapport gain/perte. Aucun stop, objectif ou résultat de backtest n'est simulé.
L'indicateur n'intègre ni calendrier économique ni spread temps réel. Le modèle
est réappris sur l'historique chargé : changer de flux, d'unité, de paramètres
ou de profondeur d'historique peut changer les résultats.

### Validation dans TradingView

La compilation Pine et les performances sur XAUUSD restent à vérifier dans
TradingView ; aucun compilateur Pine local n'est fourni dans ce dépôt.

- Compiler en v6 et vérifier le graphique sur M1 puis M5.
- En Bar Replay, vérifier l'absence de signaux pendant l'apprentissage et
  contrôler les motifs d'attente, puis les marqueurs sur les clôtures.
- En temps réel, vérifier que le tableau reste figé pendant la bougie et que
  les alertes ne se déclenchent qu'à sa clôture.
- Avec une pause de 3, vérifier qu'un signal en barre N ne peut être suivi d'un
  autre avant N+4 ; avec 0, deux clôtures consécutives sont autorisées.
- Comparer manuellement quelques signaux à la clôture suivante : hausse pour
  achat, baisse pour vente, égalité comptée comme échec ; vérifier leur entrée
  dans l'effectif seulement à la clôture suivante.
- Comparer en simulation des périodes distinctes, avec les coûts du courtier,
  avant toute conclusion sur l'utilité des filtres.

Référence : [TradingView — Bar states](https://www.tradingview.com/pine-script-docs/concepts/bar-states/)
pour la confirmation des bougies et le comportement en temps réel.

## XAUUSD MTF : confluence pour graphique M5

`xauusd_mtf.pine` affiche les tendances M5, M15, H1 et H4 sur le graphique.
Il conserve les cinq critères (prix/EMA lente, EMA rapide/lente, RSI, MACD,
DI+/DI−), le filtre ADX et les poids 1/2/3/4 du script initial.
Les égalités sont désormais neutres et toutes les composantes doivent être
disponibles. La période de l'EMA rapide doit être inférieure à celle de l'EMA lente.

### Décision et signaux

- Utiliser un graphique **5 minutes en bougies standard**. Les autres unités
  provoquent une erreur explicite, comme dans la version initiale.
- Le score global signé va de −50 à +50 ; le seuil initial reste ±25.
  Il décrit une confluence technique, pas une probabilité de réussite.
- Pour un achat, le biais global doit être haussier et **M5 et M15** doivent
  chacun atteindre le score minimum (+3 par défaut). Pour une vente, les
  trois conditions sont inversées (−3 pour M5/M15).
- L'amplitude de la bougie doit rester ≤ 2,5 ATR et sa clôture à ≤ 1,5 ATR de
  l'EMA rapide. Ces limites utilisent l'ATR 14 M5 précédent et sont réglables.
- Les alertes et marqueurs se déclenchent uniquement à la clôture M5. Le score
  M5 utilise alors la bougie qui vient de se terminer, supprimant le retard
  supplémentaire d'une barre de la version initiale.
- Une seule alerte est émise par séquence de conditions valides de même sens.
  Une clôture sans configuration valide ou un changement de sens réarme le
  signal. Après une émission en N, la pause initiale de 3 barres interdit toute
  nouvelle alerte avant N+4, même dans le sens opposé. Une nouvelle séquence
  commencée pendant la pause peut émettre à son expiration si elle reste valide.

Le tableau distingue le **biais GLOBAL** du **SIGNAL M5** et explique pourquoi
il faut attendre. « Signal déjà émis » ne signifie pas une sortie de position.
Le tableau conserve son dernier état clôturé pendant la bougie ouverte, tandis
que les EMA tracées peuvent évoluer avec le prix. La colonne « CLÔTURE UTC »
identifie la bougie effectivement utilisée pour chaque unité.

### Données supérieures confirmées

Les requêtes M15/H1/H4 conservent le décalage `[1]` avec `lookahead_on` : elles
utilisent la dernière bougie supérieure confirmée disponible au début de la
barre M5. Cela évite de baser les signaux sur une bougie supérieure en formation.
À une frontière H1, par exemple, la nouvelle valeur H1 est utilisée dans la
barre M5 suivante ; la prochaine décision intervient à sa clôture. Cette
latence est intentionnelle et les horaires du tableau la rendent visible.
Voir [la documentation TradingView](https://www.tradingview.com/pine-script-docs/concepts/other-timeframes-and-data/).

### Vérifications à effectuer dans TradingView

La relecture et les contrôles statiques locaux ne remplacent pas la compilation
Pine, le Bar Replay ou la validation en temps réel, qui restent à effectuer.

1. Compiler en Pine v6 sur M5 et créer les nouvelles alertes « Achat MTF
   confirmé » / « Vente MTF confirmée », une fois par clôture. Recréer les
   anciennes alertes pour leur faire utiliser ce code.
2. En Replay, observer une frontière M15/H1/H4 : vérifier les horaires UTC et
   la mise à jour du score supérieur dans la barre M5 suivante.
3. Vérifier qu'un biais global haussier avec M5 ou M15 baissier/neutre n'émet
   aucun achat ; vérifier le cas symétrique pour une vente.
4. Vérifier qu'une confluence persistante ne répète pas les alertes, qu'une
   invalidation réarme le signal et que la pause s'applique dans les deux sens.
5. Vérifier qu'un score indisponible, une extension excessive ou une bougie trop
   ample bloque le signal ; les motifs sont affichés par ordre de priorité.
6. En direct, vérifier que tableau, marqueur et alerte correspondent à la même
   clôture M5. Comparer les marqueurs après rechargement avec le même historique.

Ces filtres n'ont pas été optimisés ou backtestés sur XAUUSD. Ce script ne gère
ni les sorties, ni les coûts d'exécution, ni les annonces économiques ; il ne
mesure pas la rentabilité et ne fixe pas de durée de détention.
