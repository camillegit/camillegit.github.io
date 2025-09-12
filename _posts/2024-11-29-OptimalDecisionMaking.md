---
title: "Finding the optimal equipment in a video game"
author: camille              # for single entry
date: 2024-11-29 12:00:00 +0100
categories: [experiment]
tags: [maths, optimization, programming, operations research]   
math: true
---

Like many young adults of my generation, I was profoundly influenced about fifteen years ago by a massively multiplayer online game. After several years away, I recently learned about the release of a revamped version of the game, **Dofus 3.0**, a modernized edition. Intrigued, I couldn’t resist diving back in. However, the game had evolved significantly with new equipment, new sets, unique bonuses, and mechanics that were completely unfamiliar to me. Rather than feeling overwhelmed, I decided to turn this rediscovery into a small, playful project that would allow me to practice mathematics and programming. The idea was to write a **linear optimization program** to identify the best possible equipment according to targeted objectives. These methods are highly valued in the field of **operations research**, where they are used to make decisions considered optimal in organizational operations. This project, in addition to immersing me back into the world of Dofus, allowed me to apply a few mathematical tools I had always wanted to explore.  

---

## Game Mechanics

Dofus is an online role-playing game where players control a character who must confront numerous fantastical and quirky creatures. Progression is partly based on obtaining items and equipment that enable the player to face increasingly tough opponents. These *equipments* are divided into categories (hats, capes, rings, etc.) and grant the character specific *attributes* (strength, intelligence, agility, etc.) that enhance their abilities in combat. Equipment sometimes requires certain conditions to be equipped. Moreover, combining certain pieces of equipment from the same *set* grants additional attribute bonuses. The problem becomes non-trivial, it’s not just about finding the optimal equipment for each slot, but rather the most strategic combination. Having equipment that provides the most advantageous attributes gives your character a significant competitive edge!  

---

## Linear Optimization

For beginners, **linear optimization** is about solving a problem where you aim to maximize or minimize a function (called the objective function) while respecting constraints (the space in which the problem exists) by acting on variables. Simply put, it’s finding the combination of variables that maximizes or minimizes the objective function. Both the constraints and the objective are linear, which simplifies calculations and allows efficient algorithms to be applied. If some variables must be integers, we speak of **integer linear programming (ILP)**. This is particularly suitable for this project, we must choose a discrete, and even binary, set of equipment (you can’t take 0.52 rings or 3 hats) to maximize game attributes like strength or intelligence, while respecting constraints such as the number of available slots or specific synergies. These synergies are called *sets* (*panoplies*); when a certain number of items from a given set are equipped, they grant a bonus.  

---

## Problem Formulation

### Decision Variables

In this problem, **decision variables** are at the core of the model. Each variable represents a concrete choice. For example :  

- $$ b_i $$: a binary variable equal to 1 if equipment $$ i $$ is selected, 0 otherwise.  
- $$ y_{jk} $$: also a binary variable representing the activation of a specific set bonus for set $$ j $$ (e.g., Bouftou set) when $$ k $$ items are equipped.  

These variables determine not only which equipment is chosen but also which bonuses are activated based on equipment synergies. Multiplying these binary variables by the weighted sum of their attributes gives the combined performance of the selected equipment.  

### Constraints

To ensure the model aligns with Dofus rules, several **constraints** apply :  

- **Slot limits**  
  Each equipment category (hat, ring, weapon, etc.) has a dedicated slot, limiting how many can be equipped. For example :   

$$
 \vphantom{\sum}
 \sum_{i \in \text{hats}} b_i \leq 1 $$

- **Set bonus activation**  
  A set bonus $$ y_{jk} $$ can only be activated if enough items from that set are selected :  

$$
  \vphantom{\sum}
   y_{jk} \leq \frac{\sum_{i \in j} b_i}{k} $$

Here, $$ k $$ is the minimum number of items required to activate bonus $$ y_{jk} $$ specific to set $$ j $$. Bonuses are cumulative (the bonus for equipping 3 items adds to the bonus for equipping 2 items of the same set), so the database must reflect this.  

- **Minimum attributes**  
  Certain constraints may require a minimum threshold for an attribute $$\alpha_{i}$$, applied to all selected equipment and their associated set bonuses :  

$$

  \vphantom{\sum} 
  \text{threshold} \leq \sum_i \left(b_i \cdot \alpha_{i}\right) + \sum_{jk} \left(y_{jk} \cdot \beta_{jk}\right) $$

---

### Objective Function

The **objective function** combines the contributions of equipment $$ b_i $$ and bonuses $$ y_{jk} $$ to maximize important attributes $$\alpha_{i}$$ or $$\beta_{jk}$$ such as strength, intelligence, or vitality :  

$$

	\vphantom{\sum}
	\text{Maximise }_{b_i,y_{jk}} \: \sum_i \left(b_i \cdot \alpha_{i}\right) + \sum_{jk} \left(y_{jk} \cdot \beta_{jk}\right)
$$


We use a weighted sum of relevant attributes to reflect their relative importance for gameplay mechanics.  

---

## Implementation

### Data Collection

To solve this equipment optimization problem, it was first necessary to collect useful data. After developing a scraping program and discovering the obsolescence of the database in use, the game data was ultimately retrieved via the **DofusDB API**.  

### Data Cleaning

A crucial step in any project is cleaning and processing the data. Steps included :  

- **Eliminate unobtainable equipment**: Filter out items present in the database but not obtainable in-game.  
- **Recalculate set bonuses**: Sets offer incremental bonuses depending on the number of items selected. An incremental method was developed to calculate bonuses as new equipment is added.  
- **Standardize data formats**: Ensure data is in formats suitable for Python implementation.  

### Programming the Problem

Once the data was ready, the linear programming model was implemented using Python’s **PuLP** library. PuLP simplifies formulating and solving linear optimization problems using well-known solvers (e.g., simplex or interior point methods). The implementation followed the steps outlined above.  

---

## Conclusion and Perspectives

With the optimization model developed, it can now be tested and refined. Results are promising, providing optimized equipment sets according to various criteria. This was useful for approximating the most competitive in-game equipment, independently of the database. Some adaptations are needed to account for additional parameters, such as how easily equipment can be obtained or its in-game price. These could be included in the objective function or constraints if the data is available. Additionally, the objective could aim to maximize ultimate combat variables like damage, though this reaches the limits of problem formulation due to the game’s non-transitive mechanics and complex combat options. This is where the real fun lies, finding strategies suited to each situation and opponent. The program is available on my GitHub, and feedback is welcome.  

While using this algorithm for this purpose might seem trivial, **linear, quadratic, stochastic, and dynamic optimization** have many applications in robotics and management via operations research. For example, in **quadratic optimization**, the **Markowitz model** is stated as :  

$$

  \vphantom{\sum}

  \text{Maximise}_\mathbf{x} \: \mathbf{\mu}^\top \mathbf{x} - \gamma \mathbf{x}^\top \Sigma \mathbf{x}

$$

$$
\mathbf{1}^\top \mathbf{x} = 1
$$


Where $$\mathbf{x}$$ represents normalized asset allocation, $$\mathbf{\mu}$$ is the expected return, $$\gamma$$ is a weighting factor between risk and return, and $$\Sigma$$ is the asset covariance matrix. This normalization constraint is analogous to the equipment slot constraints above.  

Though theoretically interesting, the model is highly simplified in practice. Assumptions about future returns or risks mirroring past behavior are questionable, as are assumptions about normal distributions. As in our Dofus example, the usefulness of these solution methods depends on the chosen model and available data.  
