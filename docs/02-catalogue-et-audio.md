# 02 — Catalogue & Architecture Audio : Sapior

## Architecture du Catalogue

### Principe : Catalog-as-JSON sur CDN

Le catalogue Sapior n'est pas géré par un serveur applicatif. Toutes les métadonnées vivent dans un fichier JSON versionné sur Cloudflare R2, fetché au démarrage de l'app.

```
App launch
    ↓
useR2Catalog hook
    ↓
GET r2-catalog.json (R2 public)
    ↓
if remote.version > local.r2CatalogVersion
    → update store (appStore.r2Catalog)
    → save version
else
    → use cached catalog (offline-safe)
```

**Avantage majeur** : ajouter un nouveau livre au catalogue ne nécessite pas de mise à jour App Store. Il suffit de mettre à jour `r2-catalog.json` et d'uploader les fichiers audio/data.

### Structure du R2 Bucket

```
r2-bucket/
├── r2-catalog.json           # Index versionné (fetché au démarrage)
│
├── platon-apologie-de-socrate/
│   ├── cover.jpg             # Image couverture (400×600px, WebP)
│   ├── script_audio_long.mp3 # Audio complet (mono, 128kbps, ~45-90 min)
│   ├── resume.md             # Résumé textuel (500-800 mots)
│   ├── script_audio_long.md  # Transcription complète
│   ├── qcm.json              # Quiz (10-20 questions)
│   └── application.json      # Applications pro/perso
│
├── descartes-discours-de-la-methode/
│   └── [même structure]
│
└── ...
```

### Format r2-catalog.json

```json
{
  "version": 42,
  "books": [
    {
      "id": "platon-apologie-de-socrate",
      "title": "Apologie de Socrate",
      "author": "Platon",
      "categoryId": "philosophie",
      "year": "-399",
      "folder": "platon-apologie-de-socrate",
      "durationSeconds": 3240,
      "available": true
    },
    {
      "id": "nietzsche-ainsi-parlait",
      "title": "Ainsi parlait Zarathoustra",
      "author": "Friedrich Nietzsche",
      "categoryId": "existentialisme",
      "year": "1883",
      "folder": "nietzsche-ainsi-parlait",
      "durationSeconds": 5400,
      "available": true
    }
  ]
}
```

## Architecture Audio

### Pipeline de téléchargement

```typescript
// hooks/useAudio.ts

async function loadAudio(bookId: string) {
  const folder = getBookFolder(bookId);
  const localPath = `${FileSystem.Paths.document}/audio/${folder}.mp3`;

  // 1. Vérifier si déjà téléchargé
  const fileInfo = await FileSystem.getInfoAsync(localPath);

  if (!fileInfo.exists) {
    // 2. Télécharger depuis R2
    const r2Url = getAudioUrl(folder);
    await FileSystem.downloadAsync(r2Url, localPath, {
      headers: { 'Cache-Control': 'no-cache' }
    });
  }

  // 3. Créer le player expo-audio
  player = createAudioPlayer({ uri: localPath });

  // 4. Restaurer la position sauvegardée
  const savedPosition = await AsyncStorage.getItem(`@sapior_pos_${bookId}`);
  if (savedPosition) {
    await player.seekTo(parseFloat(savedPosition));
  }

  // 5. Démarrer l'autosave (toutes les 5 secondes)
  startAutoSave(bookId, player);
}
```

### Persistance de position

```typescript
// Sauvegarde automatique toutes les 5 secondes
function startAutoSave(bookId: string, player: AudioPlayer) {
  const interval = setInterval(async () => {
    const position = player.currentTime;
    const duration = player.duration;

    await AsyncStorage.setItem(`@sapior_pos_${bookId}`, String(position));
    await AsyncStorage.setItem(`@sapior_dur_${bookId}`, String(duration));
  }, 5000);

  return () => clearInterval(interval);
}

// Auto-restore au démarrage : dernier livre écouté (lastLoadedBookId du store)
// → loadAudio(lastLoadedBookId) → MiniPlayer visible dans TabBar
```

### Mode silencieux iOS

```typescript
await Audio.setAudioModeAsync({
  staysActiveInBackground: true,      // Lecture en arrière-plan
  playsInSilentModeIOS: true,         // Lecture même en mode silencieux
  shouldDuckAndroid: true,            // Réduction volume autres apps
});
```

### Contrôles de vitesse

```typescript
const SPEED_OPTIONS = [0.75, 1, 1.25, 1.5, 2]; // Disponibles via cycleRate()

// Cycle automatique sur le bouton vitesse
function cycleRate() {
  const currentIdx = SPEED_OPTIONS.indexOf(currentRate);
  const nextIdx = (currentIdx + 1) % SPEED_OPTIONS.length;
  player.setRate(SPEED_OPTIONS[nextIdx]);
}
```

## Gestion des Téléchargements

### Structure locale

```
device-filesystem/
└── documents/
    └── audio/
        ├── platon-apologie-de-socrate.mp3    # ~45MB
        ├── nietzsche-aussi-parlait.mp3        # ~75MB
        └── ...
```

### useScanDownloads

Au démarrage, ce hook scanne le filesystem pour synchroniser la liste des livres téléchargés avec le store (downloadedFolders). Permet de gérer les téléchargements même après une réinstallation ou une mise à jour.

```typescript
async function scanDownloads(): Promise<string[]> {
  const audioDir = `${FileSystem.Paths.document}/audio`;
  const files = await FileSystem.readDirectoryAsync(audioDir);
  return files
    .filter(f => f.endsWith('.mp3'))
    .map(f => f.replace('.mp3', ''));
}
```

### DownloadedBooksScreen

Interface de gestion : liste des livres téléchargés, espace utilisé, suppression individuelle.

```typescript
// Suppression propre
async function deleteAudio(bookId: string) {
  const path = `${FileSystem.Paths.document}/audio/${getBookFolder(bookId)}.mp3`;
  await FileSystem.deleteAsync(path, { idempotent: true });

  // Unload le player si c'est le livre actuellement chargé
  if (currentBookId === bookId) {
    await player?.remove();
    setCurrentBookId(null);
  }

  // Mise à jour du store
  store.setDownloadedFolders(prev => prev.filter(f => f !== bookId));
}
```

## Optimisations Techniques

### Lazy Loading QCM

Les fichiers `qcm.json` ne sont chargés que lors du premier accès, puis mis en cache en mémoire (store.qcmCache). Évite de charger 30+ fichiers JSON au démarrage.

```typescript
async function getQuizData(bookId: string): Promise<QCMData | null> {
  // 1. Vérifier le cache en mémoire
  if (store.qcmCache[bookId]) return store.qcmCache[bookId];

  // 2. Charger depuis R2
  const url = getQcmUrl(getBookFolder(bookId));
  const data = await fetch(url).then(r => r.json());

  // 3. Mettre en cache
  store.setQcmCache(bookId, data);
  return data;
}
```

### Cloudflare R2 vs S3

| Critère | R2 | S3 |
|--------|----|----|
| Egress fees | Gratuit | $0.09/GB |
| CDN global | Inclus | Via CloudFront (coût) |
| Latence EU | ~20ms | ~30-50ms |
| Compatibilité S3 API | ✅ | ✅ |

**Choix R2** : à notre échelle (< 100GB), R2 est significativement moins cher qu'AWS S3 + CloudFront, tout en offrant des performances similaires.
