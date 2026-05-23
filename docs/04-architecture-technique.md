# Architecture technique — Sapior

## Vue d'ensemble

Sapior n'a pas de backend applicatif. Tout le contenu (audio, quiz, covers, résumés) est hébergé sur Cloudflare R2, un service de stockage de fichiers avec CDN mondial. L'app mobile gère l'interface, les téléchargements, la lecture audio et l'état de l'utilisateur en local.

Ce choix architectural n'est pas une économie de bout de chandelle — c'est une décision produit. Un backend applicatif ajoute de la complexité à maintenir, des points de défaillance, et un coût fixe même quand personne n'utilise l'app. Sans backend, il n'y a rien à maintenir en dehors des fichiers de contenu.

## Les grandes décisions

**Pourquoi stocker le catalogue dans un fichier versionné plutôt qu'en base de données ?**

Un catalogue en base de données nécessite un serveur, une API, des requêtes, une gestion des erreurs réseau. Pour des données qui changent rarement (quelques livres ajoutés par mois), c'est de la suringénierie.

La solution retenue : un fichier JSON versionné sur R2. Au lancement, l'app vérifie si la version distante est plus récente que la version locale. Si oui, elle la télécharge. Sinon, elle utilise ce qu'elle a déjà. Ça fonctionne hors connexion, ça se met à jour sans intervention de l'utilisateur, et ça permet d'ajouter un livre dans l'app le jour même sans passer par les stores.

**Pourquoi télécharger les audios plutôt que streamer ?**

Le streaming nécessite une connexion stable pendant 60 à 90 minutes. Dans le métro, dans un avion, dans une zone blanche — la connexion tombe. Avec le téléchargement local, le livre est sur l'appareil et la connexion n'a plus d'importance.

Le téléchargement se fait une seule fois. L'utilisateur télécharge un livre quand il est en wifi, et l'écoute ensuite où il veut. C'est le comportement qu'on attend d'une app de ce type — Spotify Premium, Audible et Blinkist fonctionnent tous comme ça.

**Pourquoi React Native plutôt qu'une app native ?**

Un seul codebase pour iOS et Android. Sur un projet solo, écrire et maintenir deux apps séparées n'est pas viable. React Native offre des performances adaptées à cet usage — lecture audio, navigation, affichage de listes — avec un accès complet aux APIs natives nécessaires (audio en arrière-plan, mode silencieux, stockage local).

**Pourquoi Zustand pour la gestion de l'état ?**

L'état de l'utilisateur dans Sapior est riche : livres terminés, scores de quiz, favoris (livres, citations, applications), reading list, position audio par livre, catalogue remote, cache des quiz. Tout ça doit persister d'une session à l'autre.

Zustand est une librairie de gestion d'état légère, dont la persistance locale s'active en quelques lignes. Il n'y a pas de compte utilisateur, pas de synchronisation cloud — tout vit sur l'appareil. C'est suffisant pour l'usage actuel et ça évite de gérer une infrastructure d'authentification et de sync.

## Les arbitrages par fonctionnalité

**La reprise de position audio**

La position est sauvegardée toutes les 5 secondes. Pas en temps réel (inutile et coûteux en batterie), pas toutes les minutes (trop de perte en cas de fermeture brutale). 5 secondes est le bon équilibre : l'utilisateur perd au maximum 5 secondes de lecture s'il ferme l'app sans y avoir pensé.

Au lancement, l'app recharge automatiquement le dernier livre écouté à la bonne position. L'utilisateur n'a pas à retrouver où il en était — c'est déjà fait.

**Le MiniPlayer dans la barre de navigation**

La barre de navigation en bas de l'écran contient les quatre onglets principaux ET un player compact qui s'affiche pendant une lecture en cours. Ce player disparaît automatiquement quand l'utilisateur est sur la page du livre en train d'être lu — il n'a pas besoin de deux interfaces de contrôle sur le même écran.

Cette intégration est non-standard. Les apps audio ont habituellement un player dans une modale ou sur un écran séparé. Le parti pris ici : le player ne doit jamais interrompre la navigation, il doit coexister avec elle. Un utilisateur peut consulter le catalogue, regarder ses favoris, ou modifier ses réglages sans quitter la lecture.

**Le chargement des quiz à la demande**

Les fichiers de quiz ne sont pas tous téléchargés au démarrage de l'app. Ils sont chargés quand l'utilisateur ouvre le quiz d'un livre précis, puis gardés en mémoire pour les accès suivants. Sur 45+ livres, télécharger tous les quiz au démarrage alourdirait le lancement sans bénéfice réel pour la majorité des utilisateurs.

## Ce qui a été écarté

**Les comptes utilisateurs.** Synchroniser la progression entre appareils est une vraie valeur ajoutée — mais ça nécessite un backend, une authentification, une gestion des conflits de données. En V1, tout est local. La perte de données en cas de changement de téléphone est le principal inconvénient. C'est un compromis acceptable pour éviter une infrastructure que le volume actuel ne justifie pas.

**Les notifications push.** "Tu n'as pas écouté depuis 3 jours" — ce type de notification est souvent plus agaçant qu'utile. Le choix de ne pas implémenter les notifications n'est pas un oubli. La rétention est assurée par la qualité du contenu et les mécaniques de progression (streak, stats, parcours), pas par des rappels intrusifs.

**Le streaming audio.** Plus rapide à démarrer, mais moins fiable. Le téléchargement local coûte un peu d'espace et un peu de temps au départ. Il garantit une expérience sans interruption pendant l'écoute. Pour un contenu de 60 à 90 minutes, la fiabilité prime.

## Ce que ça coûte à maintenir

Le coût principal est le stockage et la bande passante des fichiers audio sur R2. Cloudflare ne facture pas les téléchargements sortants — contrairement à AWS S3 qui facture chaque gigaoctet téléchargé. Pour une app dont le modèle repose sur des fichiers audio lourds écoutés souvent, c'est une économie significative.

Il n'y a pas de serveur à surveiller, pas de base de données à sauvegarder, pas de déploiement à gérer. La maintenance se résume à ajouter du contenu (de nouveaux livres) et à mettre à jour l'app quand les librairies évoluent.
