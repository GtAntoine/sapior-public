# <img src="./public/images/icon.png" alt="Sapior" width="50" style="vertical-align: middle;"/> Sapior

> Livres audio et quiz pour comprendre les grandes idées, de Platon à Nassim Taleb

<div align="center">

[![App Store](https://img.shields.io/badge/App_Store-0D96F6?style=for-the-badge&logo=app-store&logoColor=white)](https://apps.apple.com/fr/app/sapior/id6761835272)
[![Google Play](https://img.shields.io/badge/Google_Play-414141?style=for-the-badge&logo=google-play&logoColor=white)](https://play.google.com/store/apps/details?id=fr.goethals.sapior)
[![React Native](https://img.shields.io/badge/React_Native-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactnative.dev/)
[![Expo](https://img.shields.io/badge/Expo-000020?style=for-the-badge&logo=expo&logoColor=white)](https://expo.dev/)

**[📱 App Store](https://apps.apple.com/fr/app/sapior/id6761835272) · [🤖 Google Play](https://play.google.com/store/apps/details?id=fr.goethals.sapior) · [🌐 Site Web](https://goethals.fr/sapior/)**

</div>

---

<div align="center">
  <img src="./public/images/screenshot-0.jpg" alt="Sapior - Accueil" width="33%" />
  <img src="./public/images/screenshot-1.jpg" alt="Sapior - Player" width="33%" />
  <img src="./public/images/screenshot-2.jpg" alt="Sapior - Quiz" width="33%" />
</div>

<div align="center">
  <img src="./public/images/screenshot-3.jpg" alt="Sapior - Stats" width="33%" />
  <img src="./public/images/screenshot-4.jpg" alt="Sapior - Recherche" width="33%" />
  <img src="./public/images/screenshot-5.jpg" alt="Sapior - Philo" width="33%" />
</div>

---

## 📚 Documentation détaillée

- **[01 - Vision & marché](./docs/01-vision-produit.md)** - Positionnement, personas, comparatif concurrents
- **[02 - Catalogue & audio](./docs/02-catalogue-et-audio.md)** - Architecture R2, téléchargement offline, persistance
- **[03 - Quiz & progression](./docs/03-gamification.md)** - Système de quiz, citations partageables, journeys
- **[04 - Architecture technique](./docs/04-architecture-technique.md)** - Choix techniques et arbitrages

---

## C'est quoi

Sapior est une application mobile iOS et Android. Elle propose des livres audio en philosophie, développement personnel, psychologie et économie, avec des quiz pour valider ce qu'on retient.

L'idée de départ : les gens achètent des livres qu'ils ne finissent pas. Le format audio avec quiz change l'équation. On écoute pendant son trajet, on répond à des questions, on suit sa progression livre par livre.

**Ce que l'utilisateur trouve dans l'app :**
- Un catalogue de 100+ titres (Platon, Nietzsche, Nassim Taleb, Hal Elrod...)
- Un player audio avec contrôle de vitesse et reprise automatique à la position précédente
- Un quiz après chaque livre pour ancrer les idées clés
- Un mode Quiz Infini avec streak (204 questions disponibles, record personnel à 40 consécutives)
- Un parcours chronologique de la philosophie, de l'Antiquité à aujourd'hui
- La possibilité de demander un audio manquant depuis l'appli

---

## Ce que ça démontre

Sapior est publiée sur deux stores, avec de vrais utilisateurs, un catalogue curé manuellement et une architecture pensée pour tenir dans la durée.

| Compétence | Ce qui le prouve |
|-----------|-----------------|
| Vision produit | Identifier un marché (edtech/culture) sous-servi, choisir un angle précis (philo + quiz) |
| UX mobile | Player avec MiniPlayer intégré dans la barre de navigation, reprise de position, vitesse variable |
| Architecture sans backend | Catalogue versionné sur CDN (Cloudflare R2), mis à jour sans passer par les stores |
| Offline | Téléchargement local des audios, lecture sans réseau |
| Rétention | Quiz avec streak, stats de progression, partage de citations |
| Publication | iOS App Store + Google Play, gestion des cycles de release |

---

## Stack

| Couche | Choix |
|--------|-------|
| Framework | React Native + Expo SDK 54 |
| Langage | TypeScript |
| State | Zustand persisté (AsyncStorage) |
| Audio | expo-audio (background + mode silencieux iOS) |
| Contenu | Cloudflare R2 (audio MP3, covers, quiz JSON) |
| Navigation | React Navigation, Stack + Tab avec CustomTabBar |
| UI | BlurView, expo-haptics |

---

## Comment le contenu est structuré

Chaque livre sur R2 contient quatre fichiers : l'audio (45 à 90 min), le résumé, la transcription complète, et un fichier JSON de quiz (10 à 20 questions avec explications). Le catalogue général (`r2-catalog.json`) est versionné. Quand on ajoute un livre, l'app le détecte au prochain lancement sans mise à jour sur les stores.

Les utilisateurs peuvent aussi demander un titre manquant depuis l'app. Ça donne une liste d'attente concrète pour prioriser le catalogue.

---

## Roadmap

**Fait**
- 100+ livres audio
- Quiz par livre + Quiz Infini avec streak
- Téléchargement offline
- Parcours philosophique chronologique
- Stats détaillées
- Publication iOS + Android

**Prévu**
- [ ] Notes personnelles par livre
- [ ] Partage de progression entre amis
- [ ] Recommandations selon l'historique d'écoute
