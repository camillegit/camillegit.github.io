---
title: "Comment trouver la panoplie optimale pour un jeu en ligne ?"
author: camille              # for single entry
date: 2024-11-29 12:00:00 +0100
categories: [maths, jeux, optimisation, programmation]
tags: [maths, jeux, optimisation, programmation]   
math: true
---

Comme beaucoup de jeunes adultes de ma génération, j'ai été marqué voilà une quinzaine d'année par un jeu en ligne massivement multijoueur. Après plusieurs années d’éloignement, j’ai récemment appris la sortie d'une refonte du jeu, Dofus 3.0, une version modernisée du jeu. Intrigué, je n’ai pas pu résister à l’envie de m’y replonger. Mais voilà, le jeu a énormément évolué : de nouveaux équipements, de nouvelles panoplies, des bonus inédits, des mécaniques qui me sont complètement inconnues. Plutôt que de me laisser submerger, j’ai décidé de transformer cette redécouverte en un petit projet ludique qui me permette de pratiquer les mathématiques et la programmation. L'idée est d'écrire un programme d’optimisation linéaire pour identifier les meilleurs équipements possibles selon des objectifs ciblés. Ce projet, en plus de me replonger dans l’univers de Dofus, me permettrait d’apprendre à mieux maîtriser les outils mathématiques que j’ai toujours voulu explorer.

## L'optimisation linéaire

Pour les noobies (les néophytes sur le sujet), l’optimisation linéaire, consiste à résoudre un problème où l’on cherche à maximiser ou minimiser une fonction (qu’on appelle la fonction objectif) tout en respectant des contraintes (l'espace dans lequel le problème existe) en agissant sur des variables. Pour faire court il s'agit de trouver la combinaison de variables qui maximise ou minimise la fonction objectif. Ces contraintes et la fonction sont toutes linéaires, ce qui simplifie les calculs, et permet d'appliquer des algorithmes de résolution efficients. Si l’on ajoute l’exigence que certaines variables doivent être des nombres entiers, on parle alors d'optimisation (ou programmation) linéaire en nombres entiers (Integer Linear Programming, ILP pour les britons). C’est particulièrement adapté à ce projet : nous devons choisir un ensemble d’équipements discrets, et même binaires, (pas question de prendre 0,52 anneau ou 3 chapeaux) pour maximiser des caractéristiques de jeu comme la force ou l’intelligence, tout en respectant des contraintes comme le nombre d’emplacements disponibles ou des synergies spécifiques.


## La formulation du problème
### Les variables de décision

Dans ce problème, les **variables de décision** sont au cœur du modèle. Chaque variable représente un choix concret à effectuer. Par exemple :
- $$ b_i $$ : une variable binaire qui vaut 1 si un équipement $$ i $$ est sélectionné, et 0 sinon.
- $$ y_{jk} $$ : une variable, également binaire, représentant l’activation d’un bonus de panoplie (c'est à dire une combinaison d'équipements) spécifique de l'ensemble $$ j $$, disons la panoplie du Bouftou, pour $$ k $$ objets équipés.

Ces variables déterminent non seulement quels équipements sont choisis, mais aussi quels bonus sont activés en fonction des synergies entre équipements. Ces variables étant binaires, leur multiplication avec la somme pondérée de leurs caractéristiques propres nous donnera la performance combinée des équipements choisis.

### Les contraintes

Pour que le modèle reste cohérent avec les règles de Dofus, plusieurs **contraintes** s’appliquent aux variables :

1. **Limite du nombre d’emplacements**  
   Chaque catégorie d’équipement (chapeau, anneau, arme, etc.) a un emplacement dédié, et ne peut donc être équipé qu'en quantité limitée : on ne peut pas équiper deux chapeaux, par exemple ou plus de deux anneaux. Cela impose une contrainte comme : 
 
	$$ \sum_{i \in \text{chapeaux}} b_i \leq 1 $$

2. **Activation des bonus de panoplies**  
   Un bonus de panoplie $$ y_{jk} $$ ne peut être activé que si suffisamment d’éléments de cette panoplie sont sélectionnés :  

   $$ y_{jk} \leq \frac{\sum_{i \in j} b_i}{k} $$
  
   où $$ k $$ est le nombre minimal d’équipements nécessaires pour activer le bonus $$ y_{jk} $$ spécifique à la panoplie $$ j $$. Les bonus seront cumulatifs (le bonus pour avoir équipé 3 équipements d'additionnera au bonus pour avoir équipé 2 équipements de la même panoplie), nous devrons nous assurer que notre base de donnée soit construite ainsi.


3. **Caractéristiques minimales**  
   Certaines contraintes peuvent donner un seuil pour une caractéristique $$\alpha_{i}$$ donnée, appliquée à tous les équipements sélectionnés, par exemple :  
   
	$$  \text{seuil} \leq \sum_i b_i \cdot \alpha_i $$

### La fonction objectif
 
   La fonction objectif combine les contributions des équipements $$ b_i $$ et des bonus $$ y_{jk} $$ pour maximiser les caractéristiques $$\alpha_{i}$$ ou $$\beta_{j,k}$$ importantes comme la force, l’intelligence ou la vitalité, sous la forme :  
   
$$
\text{Maximiser } \sum_i \left(b_i \cdot \alpha_{i}\right) + \sum_{j,k} \left(y_{jk} \cdot \beta_{j,k}\right)
$$

  Nous faisons un somme pondérée des caractéristiques d'intérêt pour définir leur importance relative dans l'objectif (selon ce qui est le plus pertinent pour les mécaniques de jeu).

## L'implémentation

### Collecte des données

Pour résoudre ce problème d'optimisation des équipements, nous avons d'abord dû récolter les données nécessaires à la modélisation du problème. La première étape de notre quête a été de programmer un **scrapper** avec **Beautiful Soup** en Python, pour extraire automatiquement des informations de pages web. L'objectif était de collecter les données des équipements depuis l'encyclopédie du jeu disponible sur le site officiel de **Dofus**. À l'époque, cette encyclopédie regorgeait d'informations détaillées sur les équipements, les bonus de panoplie et les caractéristiques associées. 

Cependant, au détour de la lecture d'une note de mise à jour, une mauvaise nouvelle est tombée : l'encyclopédie du site de Dofus allait être fermée à l'occasion de la refonte du jeu. Heureusement, de rares alternatives existent, comme l'API de DofusDB. Cette API permet de collecter les informations détaillées sur les équipements et les mécanismes du jeu.

### Traitement et nettoyage des données

Une fois les données collectées, une étape cruciale de tout projet a été le **nettoyage** et le **traitement** des informations. Les traitements effectuées incluent notamment :

- **Éliminer les équipements non obtenables en jeu** : Nous avons filtré les objets qui ne pouvaient être obtenus dans le jeu, comme ceux obtenus uniquement par des évenements ou autres.
  
- **Recalculer les bonus de panoplie** : Les panoplies offrent des bonus croissants en fonction du nombre d'objets sélectionnés. Il faut donc développer une méthode **incrémentale** pour calculer les bonus activés à mesure que de nouveaux équipements étaient ajoutés à la sélection.

- **Uniformiser le format des données** : Enfin il faut faire en sorte que les données soient présentes dans des formats qui seront utilisables plus facilement pour l'implémentation du problème en Python.

### Programmation du problème

Une fois les données prêtes, il a fallu aborder la partie **programmation linéaire** du projet. Heureusement, grâce à la bibliothèque **PuLP** en Python, l’implémentation du modèle d'optimisation a été relativement simple. PuLP permet de formuler simplement et de résoudre des problèmes d'optimisation linéaire, en utilisant les **Solvers** populaires (les algorithmes de résolution des problèmes à proprement parler). L'implémentation se fait en suivant les étapes du problèmes décrites précédemment.

## Conclusion

Une fois le modèle d'optimisation développé, il ne restait plus qu'à le tester et à l'affiner. Les résultats sont satisfaisants et permettent d'obtenir des ensembles d'équipements optimisés selon divers critères. Le programme est disponible sur mon **GitHub**, n'hésitez pas à partager vos propositions d'amélioration.


