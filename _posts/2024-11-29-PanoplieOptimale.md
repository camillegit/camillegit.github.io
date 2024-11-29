---
title: "Comment trouver la panoplie optimale pour un jeu en ligne ?"
author: camille              # for single entry
date: 2023-11-29 12:00:00 +0100
categories: [maths, jeux, optimisation, programmation]
tags: [maths, jeux, optimisation, programmation]   
---

### Comment trouver la panoplie optimale pour un jeu en ligne ? 

Comme beaucoup de jeunes adultes de ma génération, j'ai été marqué voilà une quinzaine d'année par un jeu en ligne massivement multijoueur. Après plusieurs années d’éloignement, j’ai récemment appris la sortie d'une refonte du jeu, Dofus 3.0, une version modernisée du jeu. Intrigué, je n’ai pas pu résister à l’envie de m’y replonger. Mais voilà, le jeu a énormément évolué : de nouveaux équipements, de nouvelles panoplies, des bonus inédits, des mécaniques qui me sont complètement inconnues. Plutôt que de me laisser submerger, j’ai décidé de transformer cette redécouverte en un petit projet ludique qui me permette de pratiquer les mathématiques et la programmation. L'idée est d'écrire un programme d’optimisation linéaire pour identifier les meilleurs équipements possibles selon des objectifs ciblés. Ce projet, en plus de me replonger dans l’univers de Dofus, me permettrait d’apprendre à mieux maîtriser les outils mathématiques que j’ai toujours voulu explorer.

## L'optimisation linéaire

Pour les noobies (les néophytes sur le sujet), l’optimisation linéaire, consiste à résoudre un problème où l’on cherche à maximiser ou minimiser une fonction (qu’on appelle la fonction objectif) tout en respectant des contraintes (l'espace dans lequel le problème existe) en agissant sur des variables. Pour faire court il s'agit de trouver la combinaison de variables qui maximise ou minimise la fonction objectif. Ces contraintes et la fonction sont toutes linéaires, ce qui simplifie les calculs, et permet d'appliquer des algorithmes de résolution efficients. Si l’on ajoute l’exigence que certaines variables doivent être des nombres entiers, on parle alors de programmation linéaire en nombres entiers (Integer Linear Programming, ILP pour les britons). C’est particulièrement adapté à ce projet : nous devons choisir un ensemble d’équipements discrets, et même binaires, (pas question de prendre 0,52 anneau ou 3 chapeaux) pour maximiser des caractéristiques de jeu comme la force ou l’intelligence, tout en respectant des contraintes comme le nombre d’emplacements disponibles ou des synergies spécifiques.


## La formulation du problème
# Les variables de décision

Dans ce problème, les **variables de décision** sont au cœur du modèle. Chaque variable représente un choix concret à effectuer. Par exemple :
- \( b_i \) : une variable binaire qui vaut 1 si un équipement \( i \) est sélectionné, et 0 sinon.
- \( y_{jk} \) : une variable représentant l’activation d’un bonus de panoplie (c'est à dire une combinaison d'équipements) spécifique de l'ensemble \( j \), disons la panoplie du Bouftou, pour \( k \) objets équipés.

Ces variables déterminent non seulement quels équipements sont choisis, mais aussi quels bonus sont activés en fonction des synergies entre équipements. Ces variables étant binaires, leur multiplication avec la somme pondérée de leurs caractéristiques propres nous donnera la performance combinée des équipements choisis.

# Les contraintes

Pour que le modèle reste cohérent avec les règles de Dofus, plusieurs **contraintes** s’appliquent aux variables :

1. **Limite du nombre d’emplacements**  
   Chaque catégorie d’équipement (chapeau, anneau, arme, etc.) a un emplacement dédié, et ne peut donc être équipé qu'en quantité limitée : on ne peut pas équiper deux chapeaux, par exemple ou plus de deux anneaux. Cela impose une contrainte comme :  
   \[
   \sum_{i \in \text{chapeaux}} b_i \leq 1
   \]

2. **Activation des bonus de panoplies**  
   Un bonus de panoplie \( y_{jk} \) ne peut être activé que si suffisamment d’éléments de cette panoplie sont sélectionnés :  
   \[
   y_{jk} \leq \frac{\sum_{i \in j} b_i}{k}
   \]  
   où \( k \) est le nombre minimal d’items nécessaires pour activer le bonus \( y_{jk} \). Les bonus seront cumulatifs (le bonus pour avoir équipé 3 équipements d'additionnera au bonus pour avoir équipé 2 équipements de la même panoplie), nous devrons nous assurer que notre base de donnée soit construite ainsi.


3. **Caractéristiques minimales**  
   Certaines contraintes peuvent donner un seuil pour une caractéristique pour les équipements sélectionnés, par exemple :  
   \[
   \sum_i b_i \cdot \text{caractéristique}_i \leq \text{seuil}
   \]

# La fonction objectif
 
   La fonction objectif combine les contributions des équipements \( b_i \) et des bonus \( y_{jk} \) pour maximiser les caractéristiques (\(\alpha_{i}\) ou \(\beta_{j,k}\)
) importantes comme la force, l’intelligence ou la vitalité, sous la forme :  
   \[
   \text{Maximiser } \sum_i \left(b_i \cdot \alpha_{i}\right) + \sum_{j,k} \left(y_{jk} \cdot \beta_{j,k}\right)
   \]

  Nous faisons un somme pondérée des caractéristiques d'intérêt pour définir leur importance relative dans l'objectif (selon ce qui est le plus pertinent pour les mécaniques de jeu).