---
title: "Comment trouver la panoplie optimale pour un jeu en ligne ?"
author: camille              # for single entry
date: 2024-11-29 12:00:00 +0100
categories: [expérience]
tags: [maths, jeux, optimisation, programmation]   
math: true
---

Comme beaucoup de jeunes adultes de ma génération, j'ai été marqué voilà une quinzaine d'année par un jeu en ligne massivement multijoueur. Après plusieurs années d’éloignement, j’ai récemment appris la sortie d'une refonte du jeu, Dofus 3.0, une version modernisée du jeu. Intrigué, je n’ai pas pu résister à l’envie de m’y replonger. Mais voilà, le jeu a énormément évolué : de nouveaux équipements, de nouvelles panoplies, des bonus inédits, des mécaniques qui me sont complètement inconnues. Plutôt que de me laisser submerger, j’ai décidé de transformer cette redécouverte en un petit projet ludique qui me permette de pratiquer les mathématiques et la programmation. L'idée est d'écrire un programme d’optimisation linéaire pour identifier les meilleurs équipements possibles selon des objectifs ciblés. Ce projet, en plus de me replonger dans l’univers de Dofus, me permettrait d’appliquer quelques outils mathématiques que j’ai toujours voulu explorer.

## L'optimisation linéaire

Pour les noobies (les néophytes sur le sujet), l’optimisation linéaire, consiste à résoudre un problème où l’on cherche à maximiser ou minimiser une fonction (qu’on appelle la fonction objectif) tout en respectant des contraintes (l'espace dans lequel le problème existe) en agissant sur des variables. Pour faire court il s'agit de trouver la combinaison de variables qui maximise ou minimise la fonction objectif. Ces contraintes et la fonction sont toutes linéaires, ce qui simplifie les calculs, et permet d'appliquer des algorithmes de résolution efficients. Si l’on ajoute l’exigence que certaines variables doivent être des nombres entiers, on parle alors d'optimisation (ou programmation) linéaire en nombres entiers (Integer Linear Programming, ILP pour les britons). C’est particulièrement adapté à ce projet : nous devons choisir un ensemble d’équipements discrets, et même binaires, (pas question de prendre 0,52 anneau ou 3 chapeaux) pour maximiser des caractéristiques de jeu comme la force ou l’intelligence, tout en respectant des contraintes comme le nombre d’emplacements disponibles ou des synergies spécifiques. Ces synergies sont appelées des panoplies, lorsqu'un certain nombre d'équipements d'un ensemble donné sont équipées, ceux-ci donnent droit à un bonus.


## La formulation du problème
### Les variables de décision

Dans ce problème, les **variables de décision** sont au cœur du modèle. Chaque variable représente un choix concret à effectuer. Par exemple :
- $$ b_i $$ : une variable binaire qui vaut 1 si un équipement $$ i $$ est sélectionné, et 0 sinon.
- $$ y_{jk} $$ : une variable, également binaire, représentant l’activation d’un bonus de panoplie (c'est à dire une combinaison d'équipements) spécifique de l'ensemble $$ j $$, disons la panoplie du Bouftou, pour $$ k $$ objets équipés.

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
  
   où $$ k $$ est le nombre minimal d’équipements nécessaires pour activer le bonus $$ y_{jk} $$ spécifique à la panoplie $$ j $$. Les bonus seront cumulatifs (le bonus pour avoir équipé 3 équipements d'additionnera au bonus pour avoir équipé 2 équipements de la même panoplie), nous devrons nous assurer que notre base de donnée soit construite ainsi.


- **Caractéristiques minimales**  
   Certaines contraintes peuvent donner un seuil pour une caractéristique $$\alpha_{i}$$ donnée, appliquée à tous les équipements sélectionnés, par exemple :  
   
$$

  \vphantom{\sum} 
  \text{seuil} \leq \sum_i \left(b_i \cdot \alpha_{i}\right) + \sum_{jk} \left(y_{jk} \cdot \beta_{jk}\right) $$

- **Condition d'équipement**  
   Enfin, certains équipements ne sont équipables que sous certaines conditions de caractéristiques sur l'ensemble équipé, par exemple d'avoir une Force supérieure à un certain seuil. Ces conditions peuvent être multiples et obéissent aux règles de la logique combinatoire. La formulation de ce dernier type de contrainte est un approximation pour linéariser le problème, qui restera valide dans l'espace des possibilités connues déterminé par les règles du jeu. Pour simplifier je présente ici l'exemple d'implémentation d'une unique condition  d'infériorité à $$ \khi_{i_1} $$ pour un équipement $$ b_i_1 $$, par exemple faisant partie d'un ensemble de *AND* logiques :  
   
$$

  \vphantom{\sum}
  b_i_1 \cdot \cdot \khi_{i_1} + \sum_i \left(b_i \cdot \alpha_{i}\right) + \sum_{jk} \left(y_{jk} \cdot \beta_{jk}\right) \leq 2 \cdot \khi_{i_1} $$

### La fonction objectif
 
   La **fonction objectif** combine les contributions des équipements $$ b_i $$ et des bonus $$ y_{jk} $$ pour maximiser les caractéristiques $$\alpha_{i}$$ ou $$\beta_{jk}$$ importantes comme la force, l’intelligence ou la vitalité, sous la forme :  
   
$$

  \vphantom{\sum}
\text{Maximiser }_{b_i,y_{jk}} \: \sum_i \left(b_i \cdot \alpha_{i}\right) + \sum_{jk} \left(y_{jk} \cdot \beta_{jk}\right)
$$

  Nous faisons un somme pondérée des caractéristiques d'intérêt pour définir leur importance relative dans l'objectif (selon ce qui est le plus pertinent pour les mécaniques de jeu).

## L'implémentation

### Collecte des données

Pour résoudre ce problème d'optimisation des équipements, nous avons d'abord dû récolter les données nécessaires à la modélisation du problème. La première étape de notre quête a été de programmer un scrapper avec Beautiful Soup en Python, pour extraire automatiquement des informations de pages web. L'objectif était de collecter les données des équipements depuis l'encyclopédie du jeu disponible sur le site officiel de Dofus. À l'époque, cette encyclopédie regorgeait d'informations détaillées sur les équipements, les bonus de panoplie et les caractéristiques associées. 

Cependant, au détour de la lecture d'une note de mise à jour, une mauvaise nouvelle est tombée : l'encyclopédie du site de Dofus allait être fermée à l'occasion de la refonte du jeu. Heureusement, de rares alternatives existent, comme l'API de DofusDB. Cette API permet de collecter les informations détaillées sur les équipements et les mécanismes du jeu.

### Traitement et nettoyage des données

Une fois les données collectées, une étape cruciale de tout projet est le nettoyage et le traitement des informations. Les traitements effectuées incluent notamment :

- **Éliminer les équipements non obtenables en jeu** : Nous avons filtré les objets qui ne pouvaient être obtenus dans le jeu, comme ceux obtenus uniquement par des évenements ou autres.
  
- **Recalculer les bonus de panoplie** : Les panoplies offrent des bonus croissants en fonction du nombre d'objets sélectionnés. Il faut donc développer une méthode incrémentale pour calculer les bonus activés à mesure que de nouveaux équipements étaient ajoutés à la sélection.

- **Uniformiser le format des données** : Enfin il faut faire en sorte que les données soient présentes dans des formats qui seront utilisables plus facilement pour l'implémentation du problème en Python.

### Programmation du problème

Une fois les données prêtes, il a fallu aborder la partie programmation linéaire du projet. Heureusement, grâce à la bibliothèque PuLP en Python, l’implémentation du modèle d'optimisation a été relativement simple. PuLP permet de formuler simplement et de résoudre des problèmes d'optimisation linéaire, en utilisant les solvers populaires (les algorithmes de résolution des problèmes à proprement parler, qui se basent sur des algorithmes de résolutions bien connus comme la méthode simplexe ou le point intérieur). L'implémentation se fait en suivant les étapes du problème décrites précédemment.

## Conclusion et perspectives

Une fois le modèle d'optimisation développé, il ne reste plus qu'à le tester et à l'affiner. Les résultats sont satisfaisants et permettent d'obtenir des ensembles d'équipements optimisés selon divers critères. Cela a été passablement utile pour avoir une première approximation des équipements les plus compétitifs dans le jeu, en étant totalement agnostique de la base de donnée des équipements. Certaines adaptations sont seulement nécessaires pour prendre en compte davantage de paramètres comme la facilité d'obtention des équipements, et découlant de cela leur prix en jeu. Ces paramètres pourraient être pris en compte dans la fonction d'objectif ou les contraintes dans la mesure où les données seraient disponibles. De plus la fonction d'objectif pourrait maximiser notamment la variable ultime que sont les dégats en combat, ici on arrivera rapidement à la limite de la formulation de problème car le jeu n'est pas transitif et les actions possibles en combat si variées qu'il serait vain de vouloir toutes les maximiser. C'est là que le plaisir du jeu se manifeste, en trouvant les stratégies de combat adaptées à chaque situation, chaque adversaire. Le programme est disponible sur mon GitHub, n'hésitez pas à partager vos propositions d'amélioration.

Si la mise en place de cet algorithme pour ces fins parait futile, l'optimisation linéaire, quadratique, stochastique, dynamique trouve de nombreuses applications en robotique, ou en gestion. J'en veux pour exemple en optimisation quadratique le **modèle de Markowitz** qui s'énonce comme suit : 

$$

  \vphantom{\sum}

  \text{Maximiser}_\mathbf{x} \: \mathbf{\mu}^\top \mathbf{x} - \gamma \mathbf{x}^\top \Sigma \mathbf{x}

$$

$$
\mathbf{1}^\top \mathbf{x} = 1
$$

Où $$\mathbf{x}$$ serait un vecteur représentant l'allocation d'actifs normalisée par le total des actifs, $$\mathbf{\mu}$$ serait le rendement moyen passé de ces actifs (hypothèse pour le rendement futur), $$\gamma$$ un facteur de pondération entre le risque et le retour, $$\Sigma$$ la matrice de covariance des actifs (hypothèse pour le risque futur). On retrouve notre contrainte pour la normalisation de la distribution, de manière analogue à la contrainte sur les types d'équipements plus haut avec $$\mathbf{1}^\top$$ le transposé du vecteur identité. 

Ce modèle, bien qu'intéressant sur le plan théorique est très simplificateur en pratique. Les hypothèses que le rendement ou le risque futurs sont similaires au passé sont sujettes à caution, tout comme les présupposés sur les distributions normales utilisées.
