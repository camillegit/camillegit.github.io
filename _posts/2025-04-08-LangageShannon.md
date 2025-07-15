---
title: "Le langage, du Shannon à Wittgenstein"
author: camille              # for single entry
date: 2025-04-08 12:00:00 +0100
categories: [notes de lecture]
tags: [langage,nlp,maths]   
---

Avec l'explosion récente de la popularité des modèles de langages, j'ai ressenti le besoin de me replonger dans l'oeuvre majeure de Claude Shannon, la théorie mathématique de la communication. L'auteur ne sera pas inconnu à tout ingénieur qui se sera frotté un jour au traitement de signal, notamment pour son théorème de l'échantillonage. J'aimerais explorer et partager ici les liens entre l'ouvrage de Shannon et les fondement du langage dans lequel nous nous exprimons en tant que concept mathématique. Cela permet de comprendre l'évolution des modèles de génération de langage automatique mais aussi de donner quelques perspectives philosophiques sur notre principal moyen d'expression et le sens profond de la communication.

## 1. Le langage, un message comme un autre ?

L'ouvrage de Claude Shannon compile pas loin de 10 ans de travail. L'enjeu est de formaliser la transmission d'un signal et fait avancer un nombre considérable de sujet d'importance cruciale dans les communications numériques modernes comme la compression, la robustesse au bruit et les limites de leur performance. Pour Shannon, un message en tant qu'objet de la transmission est un élément sélectionné dans un ensemble fini de messages possibles. Cet ensemble constitue ce que l’on peut appeler un espace de communication. Chaque message correspond à une suite de symboles, par exemple, des bits 0 et 1 dans notre ordinateur, qui sera codée, transmise via un canal, puis décodée à l’autre extrémité. De ce point de vue, le langage humain peut lui aussi être vu comme une suite de symboles choisis dans un espace fini, constitué de mots, eux-mêmes constitués de syllabes, elles-même constituées de lettres.  Une phrase que nous prononçons est ainsi l’équivalent d’un message binaire : une sélection unique et délibérée parmi une multitude de combinaisons possibles. 

Cependant le message n'existe pas de façon immanente, pour lui-même. Il intervient pour transmettre une information, une sensation, un concept entre les interlocuteurs. Ce que le message s'efforce de transmettre appartient à l'espace sémantique, c'est sa signification. Cette significaiton se réfère ou est corrélée selon certaines modalités à des entités physiques ou conceptuelles. Imaginons ici que le langage sert tout à la fois à nous conter l'épopée d'Enée ou transmettre la connaissance de la physique quantique.

Le modèle de communication de Shannon s'énonce assez simplement comme suit : 

Source → Encodeur → Canal → Décodeur → Destinataire

La source et le destinataire, dans le contexte du langage, souhaitent échanger un élément appartenant à l'espace sémantique, mettons une sensation, la mélancolie. Ainsi, Enée pourrait encoder cet élément sémantique dans le langage de la façon suivante "Sunt lacrimae rerum", peu de mots pour porter l'immense tragédie de la chute de Troie. Le destinateur, pour peu que celui-ci soit latiniste transcrira les mots dans son propre espace sémantique, espace très certainement différent d'Enée, la source compte tenu des différences d'expériences vécues, de culture. Il relève presque du miracle dès lors que le concept sémantique transmis arrive inchangé au destinataire compte tenu de toutes les transformations nécessaires au passages d'un esapce à l'autre, comme autant de diffraction. De plus il est intéressant de noter que compte tenu de l'encodage choisi, c'est à dire des mots retenus, le message est intrinsèquement ambigu. Cette ambiguité apparaît magnifiquement en français, où les traductions acceptées pourraient être "les choses sont des larmes" ou bien "il y a des larmes dans les choses". Loin de s'exclurent, ces interprétations se superposent comme des états quantiques et enrichissent encore la portée sémantique du message.


## 2. Economie de signal et probabilisme.

