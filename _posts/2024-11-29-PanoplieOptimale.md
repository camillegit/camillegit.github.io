---
title: "Comment trouver l'équipement optimal pour un jeu en ligne ?"
author: camille              # for single entry
date: 2024-11-29 12:00:00 +0100
categories: [expérience]
tags: [maths, jeux, optimisation, programmation, recherche opérationnelle]   
math: true
---

Comme beaucoup de jeunes adultes de ma génération, j'ai été marqué voilà une quinzaine d'année par un jeu en ligne massivement multijoueur. Après plusieurs années d’éloignement, j’ai récemment appris la sortie d'une refonte du jeu, Dofus 3.0, une version modernisée du jeu. Intrigué, je n’ai pas pu résister à l’envie de m’y replonger. Mais voilà, le jeu a énormément évolué : de nouveaux équipements, de nouvelles panoplies (des ensembles d'équipements spécifiques), des bonus inédits, des mécaniques qui me sont complètement inconnues. Plutôt que de me laisser submerger, j’ai décidé de transformer cette redécouverte en un petit projet ludique qui me permette de pratiquer les mathématiques et la programmation. L'idée est d'écrire un programme d’optimisation linéaire pour identifier les meilleurs équipements possibles selon des objectifs ciblés. Ces méthodes sont très prisées dans le domaine de la **recherche opérationnelle**, où elles visent à adopter des décisions considérées comme optimales dans les opérations d'une organisation. Ce projet, en plus de me replonger dans l’univers de Dofus, me permettrait d’appliquer ces quelques outils mathématiques que j’ai toujours voulu explorer.

## La mécanique du jeu

Dofus est un jeu de rôle en ligne, où le joueur incarne un personnage qui devra affronter nombre de créatures fantastiques et loufoques. La progression est basée entre autre sur l'obtention d'objets et d'équipements qui permettront au joueur d'affronter des adversaires toujours plus coriaces. Ces *équipements* sont réparties en différentes catégories (chapeaux, capes, anneaux ...) et octroient au personnage des *caractéristiques* partciulières (force, intelligence, agilité ...) qui augmentent ses aptitudes lors des affrontements. Les équipements doivent cependant parfois répondre à certaines conditions pour être équipés. De plus la combinaison de certains équipements partciculiers appartenant à une même *panoplie* donnera des bonus de caractéristiques. Le problème devient moins trivial, il ne s'agit pas seulement de trouver l'équipement optimal à chaque position mais bien la combinaison la plus judicieuse. Disposer d'un équipement donnant les caractéristiques les plus avantageuses donnera alors un avantage compétitif non négligeable à notre personnage !

## L'optimisation linéaire

Pour les noobies (les néophytes sur le sujet), l’optimisation linéaire, consiste à résoudre un problème où l’on cherche à maximiser ou minimiser une fonction (qu’on appelle la fonction objectif) tout en respectant des contraintes (l'espace dans lequel le problème existe) en agissant sur des variables. Pour faire court il s'agit de trouver la combinaison de variables qui maximise ou minimise la fonction objectif. Ces contraintes et la fonction sont toutes linéaires, ce qui simplifie les calculs, et permet d'appliquer des algorithmes de résolution efficients. Si l’on ajoute l’exigence que certaines variables doivent être des nombres entiers, on parle alors d'optimisation (ou programmation) linéaire en nombres entiers (Integer Linear Programming, ILP pour les britons). C’est particulièrement adapté à ce projet : nous devons choisir un ensemble d’équipements discrets, et même binaires, (pas question de prendre 0,52 anneau ou 3 chapeaux) pour maximiser des caractéristiques de jeu comme la force ou l’intelligence, tout en respectant des contraintes comme le nombre d’emplacements disponibles ou des synergies spécifiques. Ces synergies sont appelées des panoplies, lorsqu'un certain nombre d'équipements d'un ensemble donné sont équipées, ceux-ci donnent droit à un bonus.


## La formulation du problème
### Les variables de décision

Dans ce problème, les **variables de décision** sont au cœur du modèle. Chaque variable représente un choix concret à effectuer. Par exemple :
- $$ b_i $$ : une variable booléenne ou autrement dit binaire qui vaut 1 si un équipement $$ i $$ est sélectionné, et 0 sinon.
- $$ y_{jk} $$ : une variable, également booléenne, représentant l’activation d’un bonus de panoplie (c'est à dire une combinaison d'équipements) spécifique de l'ensemble d'équipement de la panoplie $$ j $$, disons la panoplie du Bouftou, pour $$ k $$ objets équipés. Si $$ y_{jk} $$ vaut 1 cela signifie qu'un nombre $$ k $$ ou supérieur d'équipements de cette panoplie a été équipé.

Ces variables déterminent non seulement quels équipements sont choisis, mais aussi quels bonus sont activés en fonction des synergies entre équipements. Ces variables étant binaires, leur multiplication avec la somme pondérée de leurs caractéristiques propres nous donnera la performance combinée des équipements choisis.

### Les contraintes

Pour que le modèle reste cohérent avec les règles de Dofus, plusieurs **contraintes** s’appliquent aux variables :

- **Limite du nombre d’emplacements**  
   Chaque catégorie d’équipement (chapeau, anneau, arme, etc.) a un emplacement dédié, et ne peut donc être équipé qu'en quantité limitée : on ne peut pas équiper deux chapeaux, par exemple ou plus de deux anneaux. Cela impose une contrainte comme : 
 
$$
 \vphantom{\sum}
 \sum_{i \in \text{chapeaux}} b_i \leq 1 $$

- **Activation des bonus de panoplies**  
   Un bonus de panoplie $$ y_{jk} $$ ne peut être activé que si suffisamment d’éléments de cette panoplie sont sélectionnés :  

$$
  \vphantom{\sum}
   y_{jk} \leq \frac{\sum_{i \in j} b_i}{k} $$
  
   où $$ k $$ est le nombre minimal d’équipements nécessaires pour activer le bonus $$ y_{jk} $$ correspondant à la panoplie $$ j $$ pour $$ k $$ équipements équipés. Les bonus seront cumulatifs (le bonus pour avoir équipé 3 équipements s'additionnera au bonus pour avoir équipé 2 équipements de la même panoplie), nous devrons nous assurer que notre base de donnée soit construite ainsi.


- **Caractéristiques minimales**  
   Certaines contraintes de notre problème peuvent nous permettre d'obtenir un seuil pour une caractéristique donnée, appliquées à tous les équipements sélectionnés et leurs bonus de panoplie associés. Dans le jeu cela est particulièrement utile pour choisir des équipements donnant un seuil minimum de "points d'action" ou "points de mouvements" qui permettent à chaque tour d'effectuer certaines actions et déplacements et ainsi mettre en place certaines stratégies de jeu. Les caractéristiques d'intérêt pour le seuil sont renseignées dans les vecteurs $$\eta_{i}$$ ou $$\zeta_{jk}$$ :  
   
$$

  \vphantom{\sum} 
  \text{Seuil de caractéristique} \leq \sum_i \left(b_i \cdot \eta_{i}\right) + \sum_{j, k} \left(y_{jk} \cdot \zeta_{jk}\right) $$


- **Conditions d'équipement sur les caractéristiques**

     Certains équipements ne peuvent être équipé que sous certaines conditions de caractéristiques. Par exemple la bien nommée "baguette des limbes" ne pourra être équipée par le joueur qu'à la condition que sa caractéristique de "points d'actions" soit strictement inférieure à 12 et sa caractéristique de "sagesse" soit strictement supérieure à 99. En optimisation linéaire, on ne peut pas traiter directement de clause "SI/ALORS" directement. La méthode du **Big M** va nous aider en introduisant une constante $$M$$ très grande qui sert de commutateur qui permet d'activer ou de désactiver une contrainte selon la valeur d'une variable binaire $z_{n}$. Les vecteurs $$\khi_{i}$$ ou $$\psi_{jk}$$ contiennent les valeurs de la caractéristique concernée pour le seuil en question.
      On va donc représenter chaque condition unique par par deux équation de contraintes. Premièrement, une équation introduit le commutateur binaire $$z_{n}$$, où $$n$$ représente chaque contrainte unique dans l'ensemble des équipements. $$z_{n}$$ indique si le seuil est respecté ou non.
      Ensuite une seconde équation vient condtionner la possibilité d'intégrer les N équipements $$b_i$$ concernés par cette condition unique selon la valeur du commutateur $$z_{n}$$ indiquant si la contrainte est respectée. Ces équations sont :

$$

  \vphantom{\sum}  
  \sum_i \left(b_i \cdot \khi_{i}\right) + \sum_{j, k} \left(y_{jk} \cdot \psi_{jk}\right) + \text{Base} \geq \text{Seuil condition} - M \cdot (1 - z_{n})$$

$$

  \vphantom{\sum}
  z_{n} \cdot N \geq \sum_{i \in \text{condition n}} b_i $$
 
- **Conditions d'équipement sur les bonus de panoplies**

	Une méthode similaire à la précédente est utilisée pour respecter les conditions sur les bonus de panoplies appliquées à certains équipements. Certains équipements ne peuvent être utilisés qu'à la condition que le joueur ne possède pas de bonus de panoplie actifs pour 2 équipements par exemple. Ces bonus ont été définis déjà dans les variables $$y_{jk}$$.

$$

  \vphantom{\sum}
  \sum_{k \geq V} y_{jk} \geq 1 - M \cdot (1 - z_n)$$


### La fonction objectif
 
   La **fonction objectif** combine les contributions des équipements $$ b_i $$ et des bonus $$ y_{jk} $$ pour maximiser les caractéristiques $$\alpha_{i}$$ ou $$\beta_{jk}$$ importantes comme la force, l’intelligence ou la vitalité, sous la forme :  
   
$$

	\vphantom{\sum}
	\text{Maximiser }_{b_i,y_{jk}} \: \sum_i \left(b_i \cdot \alpha_{i}\right) + \sum_{j, k} \left(y_{jk} \cdot \beta_{jk}\right)
$$

  Nous faisons un somme pondérée des caractéristiques d'intérêt pour définir leur importance relative dans l'objectif (selon ce qui est le plus pertinent pour les mécaniques de jeu).

## L'implémentation

### Collecte des données

Pour résoudre ce problème d'optimisation des équipements, il était d'abord nécessaire de récolter les données utiles à la modélisation du problème. Après le développement d'un programme de scrapping (lecture d'information sur le code source des pages internet), puis le contretemps d'apprendre l'obsolescence prochaine de la base de donnée utilisée, les données du jeu ont pu être récupérées à travers l'API (l'interface machine) de DofusDB.

### Traitement et nettoyage des données

Après collecte des données, une étape cruciale de tout projet est le nettoyage et le traitement des informations. Les traitements effectuées incluent notamment :

- **Éliminer les équipements non obtenables en jeu** : Les objets présents dans la base de données mais non obtenables en jeu ont été filtrés.
  
- **Recalculer les bonus de panoplie** : Les panoplies offrent des bonus croissants en fonction du nombre d'objets sélectionnés. Il faut donc développer une méthode incrémentale pour calculer les bonus activés à mesure que de nouveaux équipements étaient ajoutés à la sélection.

- **Uniformiser le format des données** : Enfin il faut faire en sorte que les données soient présentes dans des formats qui seront utilisables plus facilement pour l'implémentation du problème en Python.

### Programmation du problème

Une fois les données prêtes, il a fallu aborder la partie programmation linéaire du projet. Heureusement, grâce à la bibliothèque PuLP en Python, l’implémentation du modèle d'optimisation a été relativement simple. PuLP permet de formuler simplement et de résoudre des problèmes d'optimisation linéaire, en utilisant les solvers populaires (les algorithmes de résolution des problèmes à proprement parler, qui se basent sur des algorithmes de résolutions bien connus comme la méthode simplexe ou le point intérieur). L'implémentation se fait en suivant les étapes du problème décrites précédemment. 

## Conclusion et perspectives

Le modèle d'optimisation développé, il ne reste plus qu'à le tester et à l'affiner. Les résultats sont satisfaisants et permettent d'obtenir des ensembles d'équipements optimisés selon divers critères. Cela a été passablement utile pour avoir une première approximation des équipements les plus compétitifs dans le jeu, en étant totalement agnostique de la base de donnée des équipements. Certaines adaptations sont seulement nécessaires pour prendre en compte davantage de paramètres comme la facilité d'obtention des équipements, et découlant de cela leur prix en jeu. Ces paramètres pourraient être pris en compte dans la fonction d'objectif ou les contraintes dans la mesure où les données seraient disponibles. De plus la fonction d'objectif pourrait maximiser notamment la variable ultime que sont les dégats en combat, ici on arrivera rapidement à la limite de la formulation de problème car le jeu n'est pas transitif et les actions possibles en combat si variées qu'il serait vain de vouloir toutes les maximiser. C'est là que le plaisir du jeu se manifeste, en trouvant les stratégies de combat adaptées à chaque situation, chaque adversaire. Le programme est disponible sur mon GitHub, n'hésitez pas à partager vos propositions d'amélioration.

Si la mise en place de cet algorithme pour ces fins parait futile, l'optimisation linéaire, quadratique, stochastique, dynamique trouve de nombreuses applications en robotique, ou en gestion grâce à la recherche opérationnelle. J'en veux pour exemple en optimisation quadratique le **modèle de Markowitz** qui s'énonce comme suit : 

$$

  \vphantom{\sum}
  \text{Maximiser}_\mathbf{x} \: \mathbf{\mu}^\top \mathbf{x} - \gamma \mathbf{x}^\top \Sigma \mathbf{x}

$$

$$
\mathbf{1}^\top \mathbf{x} = 1
$$

Où $$\mathbf{x}$$ serait un vecteur représentant l'allocation d'actifs normalisée par le total des actifs, $$\mathbf{\mu}$$ serait le rendement moyen passé de ces actifs (hypothèse pour le rendement futur), $$\gamma$$ un facteur de pondération entre le risque et le retour, $$\Sigma$$ la matrice de covariance des actifs (hypothèse pour le risque futur). On retrouve notre contrainte pour la normalisation de la distribution, de manière analogue à la contrainte sur les types d'équipements plus haut avec $$\mathbf{1}^\top$$ le transposé du vecteur identité. 

Ce modèle, bien qu'intéressant sur le plan théorique est très simplificateur en pratique. Les hypothèses que le rendement ou le risque futurs sont similaires au passé sont sujettes à caution, tout comme les présupposés sur les distributions normales utilisées. Comme dans notre exemple, l'utilité des méthodes de résolutions est tributaire du modèle choisi ainsi que des données disponibles.
