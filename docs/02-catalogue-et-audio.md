# Catalogue et expérience audio — Sapior

## Comment le catalogue est construit

Le catalogue Sapior est curé manuellement. Chaque titre est choisi selon trois questions : est-ce que cette œuvre a influencé la façon dont les gens pensent aujourd'hui ? Peut-on l'expliquer clairement à quelqu'un sans formation ? Y a-t-il des applications pratiques dans la vie quotidienne ?

Cette sélection exclut des textes importants académiquement mais difficilement accessibles (Hegel, Heidegger dans leur intégralité), et inclut des œuvres récentes qui ne sont pas de la philosophie classique mais qui répondent aux mêmes critères (Nassim Taleb, Daniel Kahneman, Steven Levitt).

Le catalogue couvre aujourd'hui 45+ titres, organisés en cinq catégories principales : philosophie, développement personnel, psychologie, économie, et santé/bien-être.

## Ce que contient chaque livre

Pour chaque titre, l'app propose quatre types de contenu :

**L'audio** dure entre 45 et 90 minutes. Ce n'est pas une lecture du texte original — c'est un script rédigé pour être écouté, qui reformule les idées clés dans un langage accessible, avec un fil narratif clair.

**Le résumé** est un document court (500 à 800 mots) qui capture l'essentiel de l'œuvre. Il sert à réviser après l'écoute ou à décider si on veut écouter le livre complet.

**La transcription** est le script intégral de l'audio. Certains utilisateurs préfèrent lire, ou alterner entre lecture et écoute.

**Les applications** sont la partie la plus originale. Pour chaque livre, deux ou trois situations concrètes sont décrites — une dans un contexte professionnel, une dans un contexte personnel — avec une action à faire le jour même. L'idée : transformer une écoute en quelque chose d'actionnable dans les 24 heures.

## La mécanique audio

L'audio est hébergé sur Cloudflare R2, un service de stockage de fichiers. L'utilisateur télécharge le fichier avant de l'écouter — ça prend quelques secondes selon la connexion, et ensuite le livre est disponible hors réseau.

La position de lecture est sauvegardée automatiquement toutes les 5 secondes. Quand l'utilisateur rouvre l'app, il reprend exactement là où il s'est arrêté — même s'il a fermé l'app, reçu un appel, ou redémarré son téléphone.

L'audio fonctionne en arrière-plan et en mode silencieux sur iOS. Écouter Sapior avec les AirPods pendant une réunion en sourdine — c'est un usage réel, pas un edge case.

La vitesse de lecture est réglable : 0.75x pour les passages denses, 1.5x ou 2x pour les redites. L'utilisateur qui a déjà des bases sur Nietzsche peut accélérer les parties introductives.

## Le téléchargement : pourquoi offline plutôt que streaming

Le choix du téléchargement local plutôt que du streaming en continu est délibéré. Dans le métro, dans un avion, dans une zone blanche — le streaming tombe. Un livre téléchargé ne tombe jamais. Pour un contenu qu'on écoute pendant 60 minutes d'une traite, la continuité est plus importante que la légèreté du chargement.

L'app gère les téléchargements dans un écran dédié : liste des livres téléchargés, espace utilisé, possibilité de supprimer les fichiers qu'on ne réécoute plus.

## La mise à jour du catalogue sans mise à jour de l'app

Un catalogue qui grossit normalement force les utilisateurs à mettre à jour l'app pour voir les nouveaux titres. C'est une friction inutile.

Le catalogue est géré dans un fichier de configuration versionné, hébergé sur le même service que les audios. Au lancement de l'app, si une nouvelle version du catalogue est disponible, elle est chargée automatiquement. L'utilisateur voit les nouveaux titres sans rien faire.

Ce choix a un effet concret : un livre peut être ajouté et visible dans l'app le jour même, sans passer par le processus de validation des stores (qui prend en moyenne 24 à 48 heures pour iOS).

## Le bouton "Demander l'audio"

Quand un utilisateur cherche un titre et ne le trouve pas, il voit un bouton "Demander l'audio". Il peut signaler qu'il veut ce livre.

Ces demandes servent directement à prioriser le catalogue. Si dix personnes demandent le même livre en deux semaines, c'est une indication claire de ce qui doit être produit en priorité. C'est plus fiable que de décider seul ce qui va plaire.
