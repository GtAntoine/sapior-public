# 04 — Architecture Technique : Sapior

## Vue d'Ensemble

Sapior adopte une architecture **client-first, serverless-light**. Pas de backend applicatif — toutes les données vivent sur Cloudflare R2 (CDN), le state est géré côté client (Zustand), et les fonctions cloud se limitent à la livraison de contenu statique.

```
┌─────────────────────────────────────────────────────────┐
│                    Client iOS (React Native)              │
│                                                           │
│  HomeScreen → BookScreen → AudioPlayer (expo-audio)      │
│       ↑             ↓           ↓                         │
│  useR2Catalog  QCM lazy    FileSystem                     │
│  (catalog)     load        (audio cache)                  │
└─────────────────────────────────────────────────────────┘
                             │ HTTPS (public R2 URLs)
┌────────────────────────────▼────────────────────────────┐
│                    Cloudflare R2                          │
│                                                           │
│  r2-catalog.json (versionné)                             │
│  {folder}/cover.jpg                                      │
│  {folder}/script_audio_long.mp3                          │
│  {folder}/qcm.json                                       │
│  {folder}/application.json                               │
│  {folder}/resume.md                                      │
└─────────────────────────────────────────────────────────┘
```

## Choix Techniques Majeurs

### 1. Catalog-as-JSON vs Backend API

**Choix** : JSON versionné sur R2

**Avantages** :
- Zéro coût serveur pour lire le catalogue
- Mise à jour catalogue sans release App Store
- Cache offline natif (persisté dans Zustand)
- CDN mondial Cloudflare → latence < 50ms en EU

**Trade-off** :
- Impossible de personnaliser le catalogue par utilisateur
- Pas de recherche full-text côté serveur

**Décision** : acceptable au stade actuel (catalogue curé, pas de contenu généré)

### 2. expo-audio vs react-native-track-player

**Choix** : expo-audio

**Raison** :
- react-native-track-player nécessite un build natif (incompatible Expo Go pendant le développement)
- expo-audio supporte `setAudioModeAsync` (background + silent mode iOS)
- API suffisante pour notre use case (lecture simple avec position save)

**Trade-off** :
- Pas de contrôle depuis le lockscreen iOS (limité vs RNTD)
- Background audio moins robuste

**Plan V2** : migration vers react-native-track-player pour les contrôles système

### 3. Zustand vs Context API pour le state

**Choix** : Zustand (store principal) + AppContext (state audio partagé)

**Pattern hybride** :
- **Zustand** (`appStore.ts`) : état persisté (progression, favoris, catalogue, QCM cache)
- **AppContext** (`AppContext.tsx`) : état audio volatile (partagé sans persistance)

```typescript
// Ce qui va dans Zustand (persisté)
{ completedIds, scores, favoriteBookIds, r2Catalog, qcmCache, ... }

// Ce qui va dans AppContext (non-persisté)
{ currentPlayer, isPlaying, currentPosition, seekingProgress }
```

### 4. React Navigation vs Expo Router

**Choix** : React Navigation (Stack + Tab)

**Raison** :
- Expo Router (file-based routing) était en beta lors du développement
- React Navigation offre un contrôle plus fin sur les transitions
- CustomTabBar avec MiniPlayer nécessite des customisations profondes

### 5. Cloudflare R2 vs AWS S3

**Choix** : Cloudflare R2

| Critère | R2 | S3 + CloudFront |
|--------|----|----|
| Egress (per GB) | $0 | $0.09 |
| Storage (per GB/mois) | $0.015 | $0.023 |
| CDN global | ✅ inclus | ✅ via CloudFront |
| Compatibilité S3 API | ✅ | ✅ |

**Économie estimée** : pour 100GB de fichiers audio + 50k téléchargements/mois → ~$40/mois d'économie vs S3

## Navigation — Architecture Complète

```
App
├── OnboardingScreen (affiché une seule fois)
│
└── NavigationContainer
    └── TabNavigator (CustomTabBar glassmorphism)
        │
        ├── HomeTab (HomeStack)
        │   ├── HomeScreen (catalogue + search)
        │   ├── CategoryScreen
        │   ├── BookScreen (player étendu)
        │   ├── AuthorScreen
        │   ├── KeyPointsScreen
        │   ├── CitationsScreen
        │   ├── QuizScreen
        │   ├── TranscriptScreen
        │   ├── ReadingListScreen
        │   ├── InProgressScreen
        │   ├── NewBooksScreen
        │   ├── Journey/JourneyScreen
        │   └── Achievement/AchievementScreen
        │
        ├── InfiniteQCM (écran direct)
        │
        ├── Stats (écran direct)
        │
        └── SettingsTab (SettingsStack)
            ├── SettingsScreen
            └── DownloadedBooksScreen
```

### CustomTabBar

```typescript
// components/TabNavigator.tsx

// Pill glassmorphism avec BlurView
// MiniPlayer intégré (slide in/out selon BookScreen actif)
// Haptic feedback sur chaque tab press

const CustomTabBar = ({ state, navigation }) => {
  const { currentBookId } = useAudio();
  const currentRoute = state.routes[state.index];

  // MiniPlayer visible sauf si BookScreen du livre en cours est affiché
  const showMiniPlayer =
    currentBookId &&
    !(currentRoute.name === 'Book' && currentRoute.params?.bookId === currentBookId);

  return (
    <BlurView intensity={80} style={styles.tabBar}>
      {showMiniPlayer && <MiniPlayer />}
      {state.routes.map((route, idx) => (
        <TabButton
          key={route.key}
          active={state.index === idx}
          onPress={() => navigation.navigate(route.name)}
        />
      ))}
    </BlurView>
  );
};
```

## State Management — Détail Complet

### appStore.ts (Zustand + AsyncStorage)

```typescript
// Persisté : clé 'sapior-storage'
interface AppStore {
  // Onboarding (une seule fois)
  hasSeenOnboarding: boolean;

  // Progression
  completedIds: string[];         // Livres complétés (audio + quiz)
  scores: Record<string, number>; // Score quiz par livre
  lastReadAt: Record<string, string>; // Timestamp

  // Quiz
  quizAnswersByBook: Record<string, QuizAnswer[]>;
  quizPassedIds: string[];        // Quiz validés (score ≥ 70%)
  bestQCMStreak: number;          // Record QCM infini

  // Audio
  lastLoadedBookId: string | null; // Restore au démarrage

  // Favoris
  favoriteBookIds: string[];
  favoriteCitationIds: string[];  // "bookId::idx"
  favoriteApplicationIds: string[];

  // Reading list
  readingListIds: string[];

  // Catalog (persisté pour offline)
  r2Catalog: R2Catalog | null;
  r2CatalogVersion: number;

  // QCM cache (lazy loaded, persisté)
  qcmCache: Record<string, QCMData>;

  // Downloads (non-persisté, dérivé filesystem)
  downloadedFolders: string[];
}
```

## Audio — Cycle de Vie Complet

```
App Launch
    ↓
useScanDownloads (scan filesystem → downloadedFolders)
    ↓
if lastLoadedBookId exists
    → loadAudio(lastLoadedBookId)  [position restore]
    → MiniPlayer visible

User opens BookScreen
    ↓
useIsAudioDownloaded(bookId) → boolean
    ↓
[Si non téléchargé]
Download button → FileSystem.downloadAsync(r2Url, localPath)
    ↓
[Si téléchargé]
loadAudio(bookId)
    ├── FileSystem check
    ├── createAudioPlayer({ uri: localPath })
    ├── setAudioModeAsync (background + silent)
    ├── seekTo(savedPosition)
    └── startAutoSave(5s interval)

User navigates away
    → MiniPlayer appears in TabBar (slide animation)

User deletes audio
    → deleteAudio(bookId)
    → unload player
    → FileSystem.deleteAsync(localPath)
    → downloadedFolders updated
```

## Performance

### Optimisations clés

1. **useR2Catalog** : ne refetch que si `remote.version > local.version` (pas à chaque lancement)
2. **QCM cache lazy** : chargé au premier accès, conservé en mémoire
3. **Audio download** : vérifié via `FileSystem.getInfoAsync` avant tout téléchargement
4. **Images covers** : chargées lazily depuis R2 via `getCoverSource(folder)`
5. **FlatList avec getItemLayout** : performances constantes sur grande liste

### Points d'amélioration (V2)

- react-native-track-player pour les contrôles système iOS
- Background audio download (télécharger le suivant pendant l'écoute)
- Pagination du catalogue (> 100 livres)
- Streaming audio (ne pas télécharger l'intégralité avant lecture)

## Configuration iOS pour Audio Background

```json
// app.json
"ios": {
  "infoPlist": {
    "UIBackgroundModes": ["audio"]
  }
}
```

Sans cette clé, iOS coupe la lecture audio dès que l'app passe en arrière-plan.
