---
title: "Le langage, du Shannon à Wittgenstein"
author: camille              # for single entry
date: 2025-04-08 12:00:00 +0100
categories: [notes de lecture]
tags: [langage,nlp,maths]   
---


## 1. Pourquoi parler du langage comme système d'information ?

Dans une époque saturée de communication, où des milliards de messages circulent chaque jour via des canaux numériques, on pourrait croire que le langage va de soi. Mais plus nous parlons, plus il devient essentiel de comprendre **ce que signifie vraiment communiquer**. Comment une idée, une sensation, un concept né dans l'esprit d'une personne peut-il être transmis à une autre ? Comment un univers mental se laisse-t-il encoder dans une chaîne de symboles ? Et qu'est-ce que cela implique pour notre manière de concevoir le langage, les machines qui le manipulent, et notre compréhension même de la pensée ?

Pour explorer ces questions, nous allons remonter aux fondements de la **théorie mathématique de la communication** élaborée par Claude Shannon en 1948. Cette théorie, conçue à l'origine pour optimiser les transmissions sur les réseaux téléphoniques, nous offre un modèle conceptuel puissant pour repenser le langage comme un système d'information. Nous verrons comment ce cadre peut être étendu à l'étude des langues naturelles, de la linguistique à l'intelligence artificielle, et même à une réflexion philosophique sur le sens, la pensée et la compréhension.

---

## 2. Le modèle de Shannon : canal, symboles et espace sémantique

Le modèle de communication de Shannon repose sur une structure simple :

Source → Encodeur → Canal → Décodeur → Destinataire

La **source** contient un message à transmettre. Ce message est encodé sous forme d'une séquence de symboles appartenant à un **ensemble fini** (l'alphabet du code). Ce signal est transmis par un canal, potentiellement bruité, et reçu par un décodeur qui tente de reconstruire le message.

Ce modèle repose sur une abstraction forte : il ne s'intéresse pas au sens, mais à la structure formelle de la transmission. Pourtant, si l'on veut penser le **langage humain** sous cet angle, il faut faire une analogie essentielle :

> **Chaque individu dispose d'un espace sémantique interne** (un répertoire de concepts, d'émotions, de représentations). Le langage sert de médiateur entre ces espaces : il transforme une structure mentale en une suite de symboles que l'autre devra retransformer en structure mentale.

Le langage devient ainsi un **espace de communication**, une couche d’interfaçage entre deux mondes sémantiques. Mais pour que cela fonctionne, il faut un alphabet commun, une grammaire partagée, une interprétation compatible : c’est toute la complexité des langues naturelles.

---

## 3. Économie du signal, probabilités et naissance du traitement automatique du langage

Transmettre de l'information est coûteux : en temps, en énergie, en bande passante. L'économie du signal repose sur une idée centrale : **réduire la redondance sans perdre le sens**. D'où l'importance des **statistiques** dans le langage.

Dans la théorie de Shannon, chaque symbole a une probabilité d’apparition. Un système efficace alloue moins de ressources aux symboles fréquents (ex. : en morse, le "E" est le plus court), et plus aux symboles rares. Ce principe sous-tend les systèmes de compression, mais aussi les modèles de langage.

C'est exactement ce qu'ont fait les premiers chercheurs en **traitement automatique du langage (NLP)** : ils ont modélisé le langage comme une chaîne de probabilités. Les **modèles n-grams** comptaient la fréquence des successions de mots :

> P("gâteau au chocolat") = P("gâteau") * P("au"|"gâteau") * P("chocolat"|"au")

Ces modèles sont simples, mais puissants. Ils ont permis la première vague de traduction automatique, de correction orthographique, de suggestion de texte.

Aujourd’hui, les modèles comme **BERT, GPT ou LLaMA** vont bien plus loin : ils estiment la probabilité d’apparition d’un mot ou d’une phrase à partir d’un contexte immense, grâce à l'entraînement sur des corpus massifs. Pourtant, le fondement reste shannonien : **prédire le plus probable**.

---

## 4. Entropie, redondance et richesse expressive

L’un des apports majeurs de Shannon est la notion d’**entropie informationnelle**. C’est une mesure de l’incertitude moyenne d’une source. En langage simple : plus un message est **imprévisible**, plus il est informatif.

Une langue très prévisible (comme un langage très répétitif ou codifié) a une **entropie faible**. Une langue très riche, où chaque phrase surprend, a une **entropie élevée**.

Prenons trois exemples contrastés :

- **James Joyce** : dans *Finnegans Wake*, chaque mot est un jeu, une invention, une surprise. L'entropie est maximale. C’est difficile à déchiffrer, mais chaque ligne est éminemment dense.
- **Langage marin** : extrêmement précis, chaque mot désigne une réalité physique bien délimitée. Peu de redondance, mais une entropie forte, car peu d’ambiguïté.
- **Global English** : langue simplifiée pour la communication internationale, riche en redondance, pauvre en subtilités, mais très robuste au bruit.

Cette **redondance** est essentielle pour la communication humaine : elle compense les erreurs, le bruit, l’inattention. Elle rend le langage **tolérant à la perte d’information**. Un texte entièrement optimisé pour l’efficacité serait inhumainement difficile à décoder.

---

## 5. Transmission de concepts : de Shannon à Wittgenstein

Le modèle de Shannon ne traite pas du **sens**. Il ne sait pas si le message parle d’amour ou de mécanique quantique. Pourtant, c’est précisément cela qui nous intéresse : comment un **concept**, une **pensée**, une **sensation** peut-elle voyager d’un esprit à un autre par le biais du langage ?

C’est ici que la communication rejoint la **philosophie du langage**. Wittgenstein, dans ses *Recherches philosophiques*, insiste sur le fait que le sens d’un mot est son **usage dans une forme de vie**. Parler, c’est jouer un **jeu de langage**. Comprendre un mot, c’est savoir comment il fonctionne dans des contextes partagés.

Dans cette perspective, les espaces sémantiques ne sont pas figés : ils sont **dynamiques, sociaux, culturels**. Le langage est un outil d’alignement, de synchronisation entre des univers mentaux différents. On ne transmet pas seulement des données, mais des **expériences vécues**, **des représentations subjectives**.

Or, c’est justement ce que les modèles modernes de traitement du langage essaient de capturer : non plus seulement les suites probables de mots, mais les **représentations latentes** qui sous-tendent les usages. Derrière "chat", il y a une forme, un son, un comportement, un imaginaire.

On passe alors d’un langage comme code à un langage comme **interface floue** entre deux subjectivités. Et là, le modèle de Shannon cède la place à une modélisation plus riche, à la frontière des mathématiques, des sciences cognitives et de la philosophie.

---

## Conclusion : parler, c’est coder une pensée sous incertitude

Le langage humain est un miracle d’équilibre : il est à la fois redondant et expressif, bruité et précis, statistique et symbolique. La théorie de Shannon nous donne un socle rigoureux pour penser **la transmission de l’information**, mais elle ne suffit pas à expliquer la transmission du **sens**.

C’est dans l’articulation entre les deux — transmission technique et interprétation sémantique — que réside toute la complexité du langage. Et c’est là que se joue le défi des intelligences artificielles : non seulement parler comme nous, mais **comprendre ce que signifie parler**.
