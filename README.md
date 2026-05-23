# <img src="./public/images/icon.png" alt="Sapior" width="50" style="vertical-align: middle;"/> Sapior - Case Study

> La philosophie, enfin accessible — livres audio gamifiés pour cultiver votre pensée

<div align="center">

[![App Store](https://img.shields.io/badge/App_Store-0D96F6?style=for-the-badge&logo=app-store&logoColor=white)](https://apps.apple.com/fr/app/sapior/id6761835272)
[![Google Play](https://img.shields.io/badge/Google_Play-414141?style=for-the-badge&logo=google-play&logoColor=white)](https://play.google.com/store/apps/details?id=fr.goethals.sapior)
[![React Native](https://img.shields.io/badge/React_Native-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactnative.dev/)
[![Expo](https://img.shields.io/badge/Expo-000020?style=for-the-badge&logo=expo&logoColor=white)](https://expo.dev/)
[![Cloudflare](https://img.shields.io/badge/Cloudflare_R2-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)](https://cloudflare.com/)

**[📱 App Store](https://apps.apple.com/fr/app/sapior/id6761835272) · [🤖 Google Play](https://play.google.com/store/apps/details?id=fr.goethals.sapior) · [🌐 Site Web](https://goethals.fr/sapior/)**

</div>

---

<div align="center">
  <img src="./public/images/mockup.png" alt="Sapior - Design System" width="80%" />
</div>

---

## 📑 Sommaire

- [👋 Vision Produit](#-vision-produit)
- [🎯 Le Problème Résolu](#-le-problème-résolu)
- [🚀 Innovations Clés](#-innovations-clés)
- [📚 Le Catalogue](#-le-catalogue)
- [🎮 Système de Gamification](#-système-de-gamification)
- [🎨 Design System](#-design-system)
- [🏗️ Architecture Technique](#️-architecture-technique)
- [🛠️ Stack Technique](#️-stack-technique)
- [🏆 Compétences Mises en Oeuvre](#-compétences-mises-en-oeuvre)
- [📊 Métriques & Publication](#-métriques--publication)
- [🚀 Roadmap](#-roadmap)

---

## 👋 Vision Produit

**Sapior rend la philosophie aussi accessible qu'un podcast, aussi engageante qu'un jeu.**

La philosophie est l'une des disciplines intellectuelles les plus riches qui soit — et pourtant, elle reste inaccessible pour la majorité. Les livres classiques sont denses, le vocabulaire technique, le contexte historique nécessaire. Les alternatives existantes (Blinkist, Audible) sont généralistes et ne créent pas d'engagement réel.

Sapior part d'un postulat simple : **si vous pouviez écouter Platon pendant votre trajet du matin, puis tester votre compréhension avec un quiz, et partager la citation qui vous a touché — la philosophie deviendrait une habitude quotidienne.**

> *"Parcours guidé de la pensée — de Socrate à Nietzsche, dans votre poche."*

---

## 🎯 Le Problème Résolu

### Le marché de la philosophie grand public est mal servi

| Problème | Solution Sapior |
|---------|-----------------|
| **Livres indigestes** | Scripts audio structurés, 45-90 min par oeuvre |
| **Pas de validation** | Quiz QCM après chaque écoute |
| **Isolation intellectuelle** | Citations partageables visuellement |
| **Manque de progression** | Suivi par livre, par catégorie, badges |
| **Contenu non curé** | 32 catégories, chaque oeuvre sélectionnée manuellement |
| **Offline impossible** | Téléchargement préalable, lecture sans réseau |

### Positionnement vs. concurrents

```
                    Philosophie     Gamification    Offline    Prix
Sapior                  ✅               ✅            ✅       💰
Audible                 ⚠️               ❌            ✅       💰💰
Blinkist                ⚠️               ❌            ✅       💰💰
Podcasts                ⚠️               ❌            ⚠️       Gratuit
YouTube                 ⚠️               ❌            ❌       Gratuit

✅ = excellent   ⚠️ = partiel   ❌ = absent
```

---

## 🚀 Innovations Clés

### 1. MiniPlayer natif glassmorphism intégré à la navigation

Le player audio n'interrompt pas la navigation. Il s'intègre directement dans la barre de navigation (CustomTabBar) et se slide automatiquement hors de vue quand on consulte le BookScreen du livre en cours.

```
TabBar pill glassmorphism (BlurView)
├── Tab: Accueil
├── Tab: QCM Infini
├── Tab: Stats
├── Tab: Paramètres
└── MiniPlayer (slide in/out automatique)
    ├── Cover + Titre + Auteur
    ├── Play/Pause
    └── Barre de progression
```

### 2. Catalogue dynamique depuis Cloudflare R2

Le catalogue n'est pas hardcodé. Un `r2-catalog.json` versionné est fetché au démarrage. Si la version distante > version locale, le catalogue se met à jour — sans mise à jour App Store.

### 3. QCM auto-générés par livre

Chaque livre dispose d'un `qcm.json` sur R2, chargé lazily au premier accès. Le mode "QCM Infini" pioche dans tous les livres consultés, génère des streaks et un score best-of persisté.

### 4. Citations visuelles partageables

Long-press sur une citation → capture d'écran d'une carte personnalisée (CitationShareCard) → Share API native. L'utilisateur partage une belle carte formatée, pas juste du texte.

### 5. Téléchargement offline intelligent

```typescript
// Téléchargement → stockage local FileSystem
FileSystem.Paths.document + '/audio/{folder}.mp3'

// Position sauvegardée toutes les 5s
AsyncStorage.setItem('@sapior_pos_<bookId>', position)

// Restauration automatique au démarrage du dernier livre écouté
// via lastLoadedBookId dans appStore
```

---

## 📚 Le Catalogue

### 32 catégories, 30+ oeuvres

```
Philosophie           Développement personnel    Économie & Finance
Psychologie           Santé & Bien-être          Histoire
Spiritualité          Littérature                Sciences
Art & Esthétique      Éthique                    Politique
Sociologie            Anthropologie              Linguistique
...
```

### Format par livre (sur Cloudflare R2)

```
r2-bucket/
└── {folder}/
    ├── cover.jpg              # Couverture (400×600)
    ├── script_audio_long.mp3  # Audio complet (45-90 min)
    ├── resume.md              # Résumé textuel
    ├── script_audio_long.md   # Transcription complète
    ├── qcm.json               # Quiz QCM (10-20 questions)
    └── application.json       # Applications pro/perso des concepts
```

### Application JSON — innovations produit

```json
{
  "applications": [
    {
      "section": "pro",
      "title": "La méthode socratique en réunion",
      "content": "Posez des questions ouvertes plutôt que d'affirmer...",
      "action_immediate": "Lors de votre prochaine réunion, remplacez une affirmation par une question"
    },
    {
      "section": "perso",
      "title": "Questionner vos certitudes quotidiennes",
      "content": "Identifiez une conviction forte et demandez-vous...",
      "action_immediate": "Ce soir, choisissez une croyance et cherchez 3 contre-exemples"
    }
  ]
}
```

---

## 🎮 Système de Gamification

### Progression multi-niveaux

```
Livre individuel
├── % avancement audio (position / durée totale)
├── Score quiz (0-100%)
├── Statut : À lire | En cours | Terminé
└── En favoris ✓/✗

Par catégorie
└── Nombre livres terminés / total

Global
├── Streak QCM infini (bestQCMStreak)
├── Badges déblocables
└── StatsScreen : temps d'écoute, quiz, favoris
```

### QCM Infini

Mode spécial sans limite de temps — questions piochées dans tous les livres consultés. Streak visuel, feedback immédiat (bonne/mauvaise réponse + explication), score persisté.

### Journeys thématiques

Parcours guidés multi-livres autour d'un thème (ex : "Les grands stoïciens", "L'école de Francfort"). Le JourneyScreen présente les étapes et la progression globale du parcours.

---

## 🎨 Design System

### "Parcours guidé de la pensée"

Inspiré de l'esthétique Renaissance — or, noir profond, portraits philosophiques. L'app doit évoquer la gravité intellectuelle sans être austère.

### Palette

| Token | Valeur | Usage |
|-------|--------|-------|
| `bg` | `#0E0E12` | Fond principal |
| `accent` | `#6C5CE7` | Violet — accent principal, progression |
| `gold` | `#F0C27A` | Or — éléments premium, badges |
| `green` | `#00B894` | Vert — progression, succès, quiz correct |
| `red` | `#FF6B6B` | Erreurs quiz, alertes |

### Composants signature

**CustomTabBar** — pill glassmorphism avec `BlurView` et `MiniPlayer` intégré
```
┌─────────────────────────────────────────────┐
│  [Cover] ── Apologie de Socrate ──  [▶/⏸]  │  ← MiniPlayer
├──────────┬──────────┬──────────┬────────────┤
│    🏠    │    ❓    │    📊    │    ⚙️      │  ← 4 tabs
└──────────┴──────────┴──────────┴────────────┘
     glassmorphism pill (BlurView)
```

**CitationCard** — double variant (sheet + list), long-press → capture → partage
**BookRow** — long-press → BottomSheet (favoris, liste, téléchargement, partage)
**GoldenParticles** — particules flottantes animées sur HomeScreen
**ProgressBar** — animée, color-coded selon completion
**QCMCard** — feedback animé bonne/mauvaise réponse avec explication

---

## 🏗️ Architecture Technique

### Structure

```
src/
├── screens/
│   ├── HomeScreen.tsx           # Catalogue + recherche
│   ├── CategoryScreen.tsx       # Liste livres par catégorie
│   ├── BookScreen.tsx           # Fiche livre + player étendu
│   ├── AuthorScreen.tsx         # Biographie auteur
│   ├── KeyPointsScreen.tsx      # Points clés du livre
│   ├── CitationsScreen.tsx      # Citations (favoris, long-press partage)
│   ├── QuizScreen.tsx           # Quiz validation
│   ├── TranscriptScreen.tsx     # Transcription complète
│   ├── ReadingListScreen.tsx    # Liste de lecture personnelle
│   ├── InfiniteQCMScreen.tsx    # Mode QCM infini (streak)
│   ├── StatsScreen.tsx          # Statistiques globales
│   ├── DownloadedBooksScreen.tsx # Gestion téléchargements
│   ├── Journey/JourneyScreen.tsx # Parcours thématiques
│   └── Achievement/             # Badges et récompenses
│
├── hooks/
│   ├── useAudio.ts              # Logique audio complète
│   ├── useR2Catalog.ts          # Fetch catalog R2 + versioning
│   ├── useIsAudioDownloaded.ts  # Vérification fichier local
│   ├── useScanDownloads.ts      # Sync filesystem → store
│   ├── useSwipeBack.ts          # Geste iOS swipe-back
│   └── useCitationShare.ts      # Capture carte + Share API
│
├── store/
│   └── appStore.ts              # Zustand (persisté AsyncStorage)
│
├── context/
│   └── AppContext.tsx           # Audio + quiz state partagés
│
├── navigation/
│   ├── TabNavigator.tsx         # CustomTabBar glassmorphism
│   └── RootNavigator.tsx
│
├── data/
│   ├── catalog.ts               # Book, Category, getBooks()
│   └── bookAssets.ts            # Résolution URLs R2
│
└── theme/
    └── tokens.ts                # Design tokens + haptics + storage keys
```

### State Management (Zustand)

```typescript
// appStore.ts — persisté AsyncStorage clé 'sapior-storage'
interface AppStore {
  // Onboarding
  hasSeenOnboarding: boolean;

  // Progression
  completedIds: string[];
  scores: Record<string, number>;
  lastReadAt: Record<string, string>;

  // Quiz
  quizAnswersByBook: Record<string, any>;
  quizPassedIds: string[];
  bestQCMStreak: number;

  // Audio
  lastLoadedBookId: string | null;

  // Favoris
  favoriteBookIds: string[];
  favoriteCitationIds: string[];      // clé "bookId::idx"
  favoriteApplicationIds: string[];   // clé "bookId::idx"
  readingListIds: string[];

  // Downloads (non-persisté, dérivé filesystem)
  downloadedFolders: string[];

  // Catalog (persisté pour offline)
  r2Catalog: R2Catalog | null;
  r2CatalogVersion: number;
  qcmCache: Record<string, any>;
}
```

### Audio Engine

```
useAudio.ts
├── loadAudio(bookId)
│   ├── Download si absent (FileSystem)
│   ├── Create expo-audio player
│   ├── Seek à position sauvegardée
│   └── Autosave position toutes les 5s
│
├── Contrôles : play/pause, seekTo, setRate
├── Vitesses : 0.75x | 1x | 1.25x | 1.5x | 2x
└── deleteAudio(bookId) → unload + suppression fichier
```

---

## 🛠️ Stack Technique

| Couche | Technologie | Raison |
|--------|------------|--------|
| **Framework** | React Native + Expo SDK 54 | Cross-platform, DX rapide |
| **Langage** | TypeScript | Type safety, refactoring sûr |
| **State** | Zustand + AsyncStorage | Léger, typé, persisté |
| **Audio** | expo-audio | Contrôle complet, mode silencieux, background |
| **Storage** | Cloudflare R2 | CDN mondial, pricing agressif, object storage |
| **Navigation** | React Navigation (Stack + Tab) | Standard RN, performant |
| **UI** | BlurView, expo-haptics | Glassmorphism natif iOS |
| **Partage** | expo-sharing + expo-file-system | Capture + share natif |
| **Catalogue** | JSON versionné sur R2 | Mise à jour sans release App Store |

---

## 🏆 Compétences Mises en Oeuvre

### Product Strategy
- **Niche bien définie** : philosophie pour grand public, pas edtech généraliste
- **Contenu curé manuellement** : chaque livre sélectionné, chaque script révisé
- **Gamification non-intrusive** : la progression enrichit sans gamifier à outrance
- **Modèle sans serveur applicatif** : tout sur R2 + client, coûts quasi-nuls à l'échelle

### Architecture Technique
- **Catalog versioning sans backend** : `r2-catalog.json` versionné élimine les serveurs
- **Offline first** : téléchargement préalable, position persistée, restore automatique
- **MiniPlayer intégré** : solution non-standard (player dans TabBar) pour UX premium
- **Cache QCM lazy** : chargé à la demande, stocké en mémoire, évite les requêtes répétées

### Design UX
- **Navigation fluide** : MiniPlayer ne bloque jamais la navigation principale
- **Haptics** : retours tactiles sur chaque interaction significative (désactivables)
- **Dark mode complet** : chaque écran pensé pour dark mode par défaut
- **Citations shareable** : transformation d'une feature "lecture" en feature "social"

### Publication App Store
- EAS Build (Expo Application Services)
- Gestion audio background iOS (`UIBackgroundModes: audio`)
- Permissions audio (RECORD_AUDIO, MODIFY_AUDIO_SETTINGS)
- Version 1.3.0 (build 15) — 15 itérations d'amélioration

---

## 📊 Métriques & Publication

| Indicateur | Valeur |
|-----------|--------|
| **Version** | 1.3.0 (build 15) |
| **Bundle ID** | `fr.goethals.sapior` |
| **Plateforme** | iOS (iPhone) |
| **Orientation** | Portrait uniquement |
| **Interface** | Dark mode forcé |
| **Audio background** | Activé |
| **Publication** | App Store (iOS) + Google Play (Android) |
| **Catalogue** | 30+ oeuvres, 32 catégories |

---

## 🚀 Roadmap

### V1.3 (actuelle) ✅
- Catalogue audio 30+ oeuvres
- Quiz QCM + QCM Infini
- Citations partageables visuellement
- MiniPlayer glassmorphism dans navigation
- Téléchargement offline
- Journeys thématiques
- Stats et progression

### V2 (prévue)
- [ ] Mode "Discussion" : débattre d'une thèse philosophique avec l'IA
- [ ] Notes personnelles par livre
- [ ] Recommandations personnalisées basées sur progression
- [ ] Notifications "rappel lecture" intelligentes
- [ ] Partage de progression entre amis

### V3 (vision)
- [ ] Création de parcours personnalisés
- [ ] Mode "Débat" : deux positions sur une question philosophique
- [ ] API pour institutions éducatives
- [ ] Android

---

## 📚 Documentation Détaillée

- **[01 - Vision Produit](./docs/01-vision-produit.md)** — Marché, personas, positionnement vs concurrents
- **[02 - Catalogue & Audio](./docs/02-catalogue-et-audio.md)** — Architecture R2, formats données, pipeline audio
- **[03 - Gamification](./docs/03-gamification.md)** — Système de progression, quiz, citations, journeys
- **[04 - Architecture Technique](./docs/04-architecture-technique.md)** — Choix techniques et trade-offs

---

<div align="center">

**Sapior** — Développé de la vision produit à la publication App Store par [Antoine Goethals](https://github.com/GtAntoine)

*React Native · Expo · Cloudflare R2 · Zustand · TypeScript*

</div>
