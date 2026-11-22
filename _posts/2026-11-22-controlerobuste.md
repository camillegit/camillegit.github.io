---
title: "Contrôle robuste et routage de voilier"
author: camille              # for single entry
date: 2026-11-22 12:00:00 +0100
categories: [expérience]
tags: [maths,contrôle,décision,voile,programmation,recherche opérationnelle]
math: true
---

<!-- Les blocs [À toi] et [Résultats] sont à compléter avec le projet. -->

J'aime beaucoup la voile, et ce mois de novembre se courait la Route du Rhum, la transatlantique en solitaire entre Saint-Malo et Pointe-à-Pitre. En suivant la flotte sur la cartographie, je me suis demandé comment les skippers choisissaient leur route. Faut-il passer au nord d'une dépression ou plonger au sud chercher les alizés ? Sur cette édition, les multicoques (Ultim, Ocean Fifty et Vintage Multi) ont le droit d'être conseillés par une équipe à terre. Les skippers des monocoques IMOCA et Class40 décident seuls, avec les fichiers météo et le logiciel de routage qu'ils ont à bord.

J'ai voulu comprendre ce que fait ce logiciel, alors j'en ai écrit un petit. Une question plus intéressante est vite apparue. Un routeur sait calculer la route la plus rapide pour une météo donnée. Or la météo de demain reste incertaine, et les centres de prévision fournissent plusieurs scénarios. Choisir une route devient alors un problème de décision sous incertitude, celui que traite le **contrôle robuste**. Je pars ici des données d'entrée d'un routeur, puis je présente la méthode des isochrones et ses liens avec la programmation dynamique et l'optimisation linéaire, avant d'arriver au contrôle robuste et à la notion de regret.

<!-- [À toi] Une phrase sur le projet lui-même (langage, pourquoi ce format, ce qu'il produit). -->

## Les données d'entrée

Un routeur a besoin de deux choses. Il doit savoir ce dont le bateau est capable, et ce que le vent va faire.

### Les polaires

Chaque bateau a sa signature, ses **polaires**. Ce sont des tableaux qui donnent la vitesse du bateau selon deux grandeurs, la force du vent réel (TWS pour *true wind speed*) et l'angle entre le cap du bateau et la direction d'où vient le vent (TWA pour *true wind angle*).

$$ V = V(\text{TWS}, \text{TWA}) $$

On les trace en coordonnées polaires, d'où leur nom. Pour chaque force de vent on obtient une courbe, et la distance au centre donne la vitesse dans chaque direction. On y lit ce que tout marin sent à la barre. Face au vent le bateau s'arrête, il atteint sa vitesse maximale au travers ou au largue, et au vent arrière il va souvent plus vite en tirant des bords qu'en filant droit. Les architectes calculent les polaires avec des logiciels de prédiction de vitesse (VPP pour *velocity prediction program*), puis les équipes les corrigent avec les mesures faites en navigation. Le routeur lit ce tableau et interpole entre les valeurs.

<!-- [À toi] Source des polaires utilisées dans le projet (quel bateau, quelle classe). -->

<!-- [Figure] Une polaire tracée pour plusieurs forces de vent. -->

### Les fichiers météo

La météo arrive sous forme de fichiers GRIB. Ce sont des grilles qui donnent, en chaque point et à chaque pas de temps, la vitesse et la direction du vent prévues. Les grands centres de prévision les produisent à partir de leurs modèles numériques de l'atmosphère. Aux États-Unis, la NOAA publie gratuitement le modèle **GFS**, le plus utilisé par les plaisanciers. En Europe, le centre européen ECMWF fait tourner le modèle **IFS**, réputé le plus fiable à moyen terme. Météo-France produit **ARPEGE** à l'échelle du globe et **AROME** en haute résolution sur la France. Enfin des fournisseurs commerciaux retravaillent ces données ou font tourner leurs propres modèles.

Le problème est que l'atmosphère est un système chaotique. Edward Lorenz l'a montré en 1963 sur un modèle très simplifié de convection. Deux états initiaux presque identiques finissent par donner des évolutions complètement différentes. Les mesures de l'état actuel de l'atmosphère contiennent toujours des erreurs, et ces erreurs grossissent avec le temps. Au-delà de quelques jours, une prévision unique donne une fausse impression de certitude.

Les centres de prévision ont trouvé une parade, la **prévision d'ensemble**. Ils font tourner le même modèle plusieurs fois, en perturbant légèrement les conditions initiales. Chaque calcul s'appelle un membre et donne un scénario plausible. L'ensemble américain GEFS compte 31 membres (un contrôle et trente perturbés), l'ensemble européen ENS en compte 51. Quand les membres restent groupés, la prévision est fiable. Quand ils divergent, l'atmosphère hésite entre plusieurs évolutions, et le marin aussi.

<!-- [À toi] Quel ensemble est utilisé dans le projet, résolution, pas de temps, horizon. -->

<!-- [Figure] Le vent d'un même point selon les 31 membres, en fonction du temps (le « panache »). -->

## Calculer la meilleure route pour une météo donnée

Commençons par le cas simple où l'on connaît la météo. On cherche la suite de caps qui amène le bateau le plus vite possible à l'arrivée, sachant que le vent change selon le lieu et l'heure. Le mathématicien Ernst Zermelo a posé ce problème dès 1931, pour un navire pris dans un vent variable. Dans le langage de mon article sur [la pensée systémique](/posts/systemthinking/), c'est un système en espace d'état. L'état est la position du bateau, la commande est le cap.

### La méthode des isochrones

La méthode la plus répandue dans les logiciels de routage, et celle que j'ai programmée, est celle des **isochrones**. Elle a été formalisée par Hagiwara en 1989 pour le routage des navires. Son principe est intuitif. Depuis le point de départ, on essaie tous les caps possibles, par exemple tous les cinq degrés. Pour chaque cap, on lit le vent dans le fichier météo, on en déduit la vitesse grâce à la polaire, et on calcule où se trouve le bateau après un pas de temps $$ \Delta t $$.

$$ p_{k+1} = p_k + \Delta t \cdot V\big(\text{TWS}(p_k, t_k), \text{TWA}(\theta, p_k, t_k)\big) \cdot \vec{e}_\theta $$

Ici $$ p_k $$ est la position au pas $$ k $$, $$ \theta $$ le cap choisi et $$ \vec{e}_\theta $$ un vecteur de longueur un qui pointe dans la direction du cap. On obtient un nuage de points atteignables. On garde seulement son enveloppe extérieure, la courbe des points les plus éloignés dans chaque direction. Cette courbe est l'isochrone, littéralement l'ensemble des points atteignables en un même temps. On recommence depuis chacun de ses points, et les isochrones s'étendent comme des ronds dans l'eau déformés par le vent. Dès qu'une isochrone touche l'arrivée, on remonte la chaîne des points qui y mènent, et cette chaîne est la route optimale.

<!-- [À toi] Détails d'implémentation (pas de temps, nombre de caps testés, gestion des côtes, temps de calcul). -->

<!-- [Figure] Isochrones successives sur une carte, et la route remontée. -->

### Le lien avec la programmation dynamique

Pourquoi a-t-on le droit de jeter tous les points à l'intérieur de l'enveloppe ? La réponse tient dans une idée de Richard Bellman, publiée dans les années 1950 sous le nom de **principe d'optimalité**. Une route optimale est faite de morceaux eux-mêmes optimaux. Si la meilleure route vers Pointe-à-Pitre passe par les Açores, alors sa première partie est forcément la meilleure route vers les Açores. Sinon on pourrait la remplacer par un chemin plus rapide et gagner du temps sur l'ensemble.

Notons $$ T^\star(p) $$ le temps minimal pour atteindre le point $$ p $$ depuis le départ. Le principe de Bellman s'écrit

$$ T^\star(q) = \min_{p \,\to\, q} \Big( T^\star(p) + \tau(p, q) \Big) $$

où $$ \tau(p,q) $$ est le temps pour aller de $$ p $$ à $$ q $$, et le minimum porte sur tous les points $$ p $$ d'où l'on peut rejoindre $$ q $$. En mots, le meilleur temps pour arriver en $$ q $$ est le meilleur temps pour arriver juste avant, plus le dernier petit trajet. On calcule donc les temps de proche en proche, ce qui évite d'énumérer toutes les routes possibles, dont le nombre explose avec la distance. C'est le cœur de la **programmation dynamique**.

Les isochrones sont exactement cette récurrence. L'isochrone au temps $$ t $$ est la frontière de l'ensemble des points tels que $$ T^\star(p) \le t $$. C'est une courbe de niveau de la fonction $$ T^\star $$, comme une ligne d'altitude sur une carte de randonnée. Un point à l'intérieur de l'enveloppe a été atteint plus tôt par un autre chemin, il est déjà couvert par la frontière, d'où l'élagage. La programmation dynamique classique découperait la mer en cases et calculerait $$ T^\star $$ pour chacune. Les isochrones font le même calcul en suivant uniquement la frontière, ce qui les rend beaucoup plus rapides.

D'autres approches existent, comme la recherche de plus court chemin sur un graphe (algorithmes de Dijkstra ou A*), la commande optimale continue issue du principe de Pontryagin, ou des méthodes heuristiques comme les algorithmes génétiques. Elles reposent presque toutes sur la même idée de fond.

### Le lien avec l'optimisation linéaire

Dans mon article sur [l'équipement optimal dans Dofus](/posts/PanoplieOptimale/), j'avais utilisé l'optimisation linéaire. Le routage s'y rattache plus qu'on pourrait le croire. Découpons la mer en un graphe de points reliés par des arcs, chaque arc $$ (i,j) $$ ayant un temps de parcours $$ c_{ij} $$. Trouver le plus court chemin revient à envoyer une unité de « bateau » du départ à l'arrivée en minimisant le temps total.

$$ \min \sum_{(i,j)} c_{ij}\, f_{ij} \quad \text{avec} \quad \sum_{j} f_{ij} - \sum_{j} f_{ji} = \begin{cases} 1 & \text{au départ} \\ -1 & \text{à l'arrivée} \\ 0 & \text{ailleurs} \end{cases}, \qquad f_{ij} \ge 0 $$

La variable $$ f_{ij} $$ vaut 1 si la route emprunte l'arc $$ (i,j) $$. La contrainte dit simplement que tout ce qui entre en un point en ressort, sauf au départ et à l'arrivée.

Tout programme linéaire a un jumeau appelé son **dual**. Ici le dual attribue à chaque point $$ i $$ une valeur $$ d_i $$ et cherche à écarter le plus possible le départ de l'arrivée, avec une seule règle.

$$ d_j - d_i \le c_{ij} \quad \text{pour chaque arc} $$

Une image classique aide à le comprendre. Imaginons un modèle en ficelles où chaque arc est une ficelle de longueur $$ c_{ij} $$. On tient le départ d'une main, l'arrivée de l'autre, et on tire. Les ficelles qui se tendent dessinent le plus court chemin, et la hauteur de chaque nœud $$ d_i $$ donne son temps d'accès depuis le départ. À l'optimum, chaque point du chemin vérifie $$ d_j = \min_i (d_i + c_{ij}) $$, ce qui est exactement l'équation de Bellman. Programmation dynamique et optimisation linéaire résolvent donc le même problème. La première exploite la structure séquentielle du trajet, d'où sa rapidité.

Cette dualité a une histoire qui nous servira plus loin. En octobre 1947, George Dantzig, qui venait d'inventer l'algorithme du simplexe, rend visite à John von Neumann à Princeton pour lui présenter l'optimisation linéaire. Von Neumann comprend tout de suite et lui expose la théorie de la dualité, qu'il relie à la théorie des jeux qu'il a développée vingt ans plus tôt.

## Combiner les isochrones et les scénarios météo

Revenons à l'incertitude. Avec un ensemble de 31 membres, je lance le calcul des isochrones 31 fois, une fois par scénario. J'obtiens 31 routes optimales $$ r_1, \dots, r_{31} $$. Quand on les superpose sur une carte, on obtient un plat de spaghettis. Certaines passent au nord, d'autres au sud, certaines se ressemblent beaucoup.

<!-- [Figure] Les 31 routes superposées. -->

Chacune de ces routes est la meilleure dans son propre monde. Pour savoir ce qu'elle vaut ailleurs, je fais naviguer chaque route $$ r_i $$ dans chaque scénario $$ s_j $$. J'obtiens une matrice de temps de parcours $$ T_{ij} $$, le temps de la route $$ i $$ quand la météo est celle du scénario $$ j $$. Par construction, la meilleure valeur de chaque colonne se trouve sur la diagonale, puisque la route $$ r_j $$ a été calculée pour le scénario $$ j $$.

Il faut préciser ce que veut dire « faire naviguer une route dans un autre scénario ». On peut suivre la trace géographique, suivre la suite de caps, ou suivre la route en s'autorisant des corrections. Ce choix change la matrice.

<!-- [À toi] Choix retenu dans le projet. -->

Reste la vraie question. Quelle ligne de la matrice choisir ?

## Le contrôle robuste

### Risque et incertitude

L'économiste Frank Knight distinguait en 1921 deux situations. Dans le **risque**, on connaît les probabilités des différents événements, comme au casino. Dans l'**incertitude**, on les ignore, ou on s'en méfie. Les membres d'un ensemble météo sont censés être équiprobables, et les prévisionnistes eux-mêmes doutent que chaque scénario ait exactement une chance sur 31 de se réaliser. On se trouve quelque part entre les deux, et c'est cette zone grise qui rend le choix intéressant.

### Le minimax de von Neumann

En 1928, le jeune John von Neumann publie à Göttingen un article sur la théorie des jeux de société. Il y étudie les jeux à deux joueurs où ce que l'un gagne, l'autre le perd. Pierre-feuille-ciseaux ou le tir au but au football en sont de bons exemples. Chaque joueur sait que l'autre cherche à lui nuire. La stratégie prudente consiste alors à choisir le coup qui donne le meilleur résultat dans le pire des cas, c'est-à-dire à minimiser sa perte maximale. D'où le nom de **minimax**.

Le théorème de von Neumann affirme que dans ces jeux, si les joueurs ont le droit de tirer leur coup au hasard, les deux points de vue se rejoignent. Le mieux que je puisse garantir en me protégeant du pire est égal au pire que mon adversaire puisse m'infliger en jouant au mieux. Il existe une valeur du jeu.

$$ \min_{x} \max_{y} \; x^\top A\, y \;=\; \max_{y} \min_{x} \; x^\top A\, y $$

Ici $$ A $$ est la table des gains, $$ x $$ et $$ y $$ sont les probabilités avec lesquelles chaque joueur choisit ses coups. Pour le tir au but, cela explique pourquoi un tireur a intérêt à varier ses côtés de façon imprévisible. C'est ce théorème que von Neumann reconnaîtra en 1947 dans la dualité de Dantzig. En 1944, il en fait avec l'économiste Oskar Morgenstern la base d'un livre, *Theory of Games and Economic Behavior*, qui fonde la théorie des jeux moderne.

Ce minimax a des racines viennoises. Morgenstern dirigeait à Vienne l'Institut autrichien de recherche sur la conjoncture et fréquentait le colloque mathématique de Karl Menger, lui-même membre du Cercle de Vienne, celui qui lisait ligne à ligne le [*Tractatus* de Wittgenstein](/posts/tractatus/). C'est par Menger que Morgenstern recrute un jeune mathématicien, Abraham Wald. Von Neumann croisait aussi ce monde. En 1930, à la conférence de Königsberg coorganisée par le Cercle de Vienne, il défendait le programme formaliste quand Kurt Gödel y a annoncé son théorème d'incomplétude, et il fut l'un des premiers à en saisir la portée.

<!-- [À toi] Lien avec l'article à venir sur Wittgenstein et le Cercle de Vienne. -->

Wald, émigré aux États-Unis en 1938, transpose le minimax à la statistique. Il imagine un jeu entre le décideur et la Nature. La Nature choisit l'état du monde, le décideur choisit son action, et faute de connaître les intentions de la Nature, il se prémunit contre le pire. Pour notre routeur, la Nature choisit le scénario météo, et le skipper choisit sa route.

### Quatre règles de décision

Formalisons. On dispose d'actions $$ a $$ (les routes), de scénarios $$ s $$ (les membres de l'ensemble) et d'un coût $$ C(a,s) $$ (le temps de parcours). Plusieurs règles permettent de choisir.

- **Laplace** fait la moyenne sur les scénarios, en les supposant équiprobables, et choisit la route la plus rapide en moyenne.

$$ \min_a \frac{1}{|\mathcal{S}|} \sum_s C(a,s) $$

- **Wald** applique le minimax et choisit la route dont le pire temps est le plus court.

$$ \min_a \max_s C(a,s) $$

- **Hurwicz** mélange le meilleur et le pire cas avec un coefficient d'optimisme $$ \alpha $$ compris entre 0 et 1.

$$ \min_a \Big( \alpha \min_s C(a,s) + (1-\alpha) \max_s C(a,s) \Big) $$

- **Savage** introduit le **regret**, en 1951, en réponse à Wald.

Le regret mesure l'écart entre ce qu'on a fait et ce qu'on aurait fait si l'on avait connu le scénario à l'avance.

$$ R(a,s) = C(a,s) - \min_{a'} C(a',s) $$

La règle de Savage choisit l'action dont le plus grand regret est le plus petit.

$$ a^\star = \arg\min_a \max_s R(a,s) $$

La différence avec Wald est subtile et importante. Si une tempête ralentit toute la flotte, le retard vient de la météo, et le regret est nul pour qui a pris la meilleure route possible. Le regret compare chaque décision à la meilleure décision possible dans le même monde. Il isole donc la part du résultat qui dépend vraiment du choix. Wald, lui, se concentre sur le pire scénario dans l'absolu, et finit souvent par choisir une route médiocre partout.

Un petit exemple rend les choses concrètes. Prenons trois routes (nord, directe, sud) et quatre scénarios, avec des temps de parcours en heures.

| Route | $$ s_1 $$ | $$ s_2 $$ | $$ s_3 $$ | $$ s_4 $$ | Moyenne | Pire cas |
|---|---|---|---|---|---|---|
| Nord | 195 | 200 | 205 | 265 | **216,25** | 265 |
| Directe | 210 | 212 | 214 | 232 | 217 | 232 |
| Sud | 225 | 226 | 228 | 228 | 226,75 | **228** |

La route nord est la meilleure en moyenne et la route sud la meilleure dans le pire cas. Calculons maintenant les regrets en retranchant à chaque colonne son minimum.

| Route | $$ s_1 $$ | $$ s_2 $$ | $$ s_3 $$ | $$ s_4 $$ | Regret maximal |
|---|---|---|---|---|---|
| Nord | 0 | 0 | 0 | 37 | 37 |
| Directe | 15 | 12 | 9 | 4 | **15** |
| Sud | 30 | 26 | 23 | 0 | 30 |

La règle de Savage choisit la route directe. Elle perd dans chaque scénario, mais toujours de peu, au plus 15 heures. Trois règles, trois routes différentes. C'est ce que j'entends par **dérisquer** une décision. On accepte de perdre un peu en moyenne pour borner l'écart avec la bonne décision.

Le regret a une limite qu'il faut connaître. Ajouter une route candidate, même mauvaise, peut modifier le minimum d'une colonne et donc changer la décision. La liste des routes qu'on accepte de comparer fait partie de la décision elle-même.

### Retour à l'optimisation linéaire

Ces règles s'écrivent très bien comme des programmes linéaires, avec les mêmes outils que pour Dofus. Introduisons une variable binaire $$ b_i $$ qui vaut 1 si l'on choisit la route $$ i $$, et une variable $$ t $$ qui représente le pire regret. Notons $$ T^\star_j $$ le meilleur temps possible dans le scénario $$ j $$. Le regret minimax devient

$$ \min t \quad \text{avec} \quad \sum_i b_i\, T_{ij} - T^\star_j \le t \;\text{ pour chaque scénario } j, \qquad \sum_i b_i = 1 $$

Chaque scénario ajoute simplement une contrainte. Le maximum, difficile à manipuler, disparaît au profit d'une variable qui doit dominer tous les cas. C'est l'astuce de base de l'**optimisation robuste**, développée notamment par Ben-Tal, El Ghaoui et Nemirovski. On peut aussi remplacer le choix d'une route parmi 31 par des routes construites directement par l'optimiseur, et le problème reste du même type.

### Du choix d'une route au contrôle robuste

Tout ce qui précède choisit une route une fois pour toutes. Un marin, lui, ajuste sa trajectoire à chaque nouveau fichier météo. C'est ici qu'intervient le **contrôle robuste**. On reprend le système de l'article sur la pensée systémique, et on y ajoute une perturbation inconnue $$ w $$, ici le vent réel.

$$ \dot{x} = f(x, u, w), \qquad w \in \mathcal{W} $$

Le contrôle stochastique suppose connue la loi de $$ w $$ et minimise le coût moyen. Le contrôle robuste suppose seulement que $$ w $$ reste dans un ensemble $$ \mathcal{W} $$ et minimise le coût dans le pire cas, ou le regret. On retrouve le jeu de von Neumann, entre le contrôleur et une Nature adversaire. Les automaticiens ont développé cette idée à partir des années 1980 avec la commande $$ H_\infty $$, qui cherche à limiter l'effet de la pire perturbation possible sur l'erreur.

La pratique du skipper correspond à la **commande prédictive**, ou commande à horizon glissant. On optimise sur tout l'horizon de la prévision, on applique seulement le premier bord, puis on recommence à l'arrivée du fichier suivant, toutes les six heures pour GEFS. Cette façon de faire introduit la notion de **recours**. Une bonne décision robuste garde de bonnes options ouvertes pour la suite. Choisir tôt un côté d'une dorsale est irréversible, et ce coût de l'irréversibilité devrait entrer dans le calcul.

<!-- SECTION À COMPLÉTER
## Résultats

 [Résultats] Tableau des routes choisies par Laplace, Wald et Savage sur un départ de la Route du Rhum. 

 [Résultats] Carte de chaleur de la matrice des regrets. 

 [Résultats] Distribution des temps de parcours de la route retenue sur les 31 scénarios. 

 [Résultats] Comparaison avec la route réellement suivie par les skippers, si les traces sont accessibles. 
-->

## Conclusion

Partir d'une question de voile m'a fait traverser plusieurs domaines qui se répondent. Les isochrones sont de la programmation dynamique, la programmation dynamique est le dual d'un programme linéaire, et la dualité linéaire est le théorème du minimax de von Neumann. Le contrôle robuste reprend ce minimax pour décider face à une Nature dont on ignore les intentions. Le regret de Savage en donne une version plus fine, qui compare chaque décision à ce qu'on aurait pu faire de mieux. Dérisquer, c'est accepter un peu moins de performance moyenne pour rester toujours proche de la bonne décision. L'idée dépasse largement la voile, elle vaut pour un investissement ou un choix technologique, partout où l'on dispose de scénarios sans probabilités crédibles.

## Références (à vérifier)

- Zermelo, E. (1931). « Über das Navigationsproblem bei ruhender oder veränderlicher Windverteilung ». *ZAMM*, 11.
- Hagiwara, H. (1989). *Weather Routing of (Sail-Assisted) Motor Vessels*. Thèse, Delft.
- Bellman, R. (1957). *Dynamic Programming*. Princeton University Press.
- Lorenz, E. N. (1963). « Deterministic Nonperiodic Flow ». *Journal of the Atmospheric Sciences*, 20.
- von Neumann, J. (1928). « Zur Theorie der Gesellschaftsspiele ». *Mathematische Annalen*, 100.
- von Neumann, J., Morgenstern, O. (1944). *Theory of Games and Economic Behavior*. Princeton University Press.
- Dantzig, G. B. (1982). « Reminiscences about the origins of linear programming ». *Operations Research Letters*, 1.
- Knight, F. H. (1921). *Risk, Uncertainty and Profit*.
- Wald, A. (1950). *Statistical Decision Functions*. Wiley.
- Savage, L. J. (1951). « The Theory of Statistical Decision ». *JASA*, 46.
- Ben-Tal, A., El Ghaoui, L., Nemirovski, A. (2009). *Robust Optimization*. Princeton University Press.
- Zhou, K., Doyle, J. C., Glover, K. (1996). *Robust and Optimal Control*. Prentice Hall.
- NOAA, Global Ensemble Forecast System (GEFS). ECMWF, Ensemble Prediction System (ENS).
