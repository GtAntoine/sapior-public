# 03 — Système de Gamification : Sapior

## Philosophie

La gamification de Sapior est **intentionnellement non-intrusive**. L'objectif n'est pas de rendre l'app addictive au sens négatif, mais de créer des boucles de renforcement positives qui encouragent la régularité et valident la compréhension.

**Principe directeur** : la progression doit refléter une vraie compréhension, pas juste du temps passé.

## Niveaux de Progression

### 1. Progression par Livre

```
État d'un livre :
├── À lire (dans reading list ou pas)
├── En cours (position audio > 0)
│   └── % d'avancement (position / durée totale)
├── Terminé (audio 100% + quiz passé)
└── Favori ⭐

Stockage :
completedIds: string[]          → livres 100% + quiz
scores: Record<string, number>  → score quiz par livre
lastReadAt: Record<string, string> → timestamp dernière écoute
```

### 2. Progression par Catégorie

```typescript
// Sur CategoryScreen : affichage dynamique
const categoryProgress = {
  completed: books.filter(b => completedIds.includes(b.id)).length,
  total: books.length,
  percentage: Math.round((completed / total) * 100)
}
```

### 3. Statistiques Globales (StatsScreen)

```
StatsScreen
├── Livres terminés : X / Y total
├── Temps d'écoute total : Xh Xmin
├── Quiz complétés : X
├── Score moyen quiz : X%
├── Streak QCM infini actuel : X
├── Best streak QCM : X (persisté)
├── Citations en favoris : X
└── Répartition par catégorie (graphique)
```

## Système de Quiz

### Quiz par Livre (QuizScreen)

Après avoir écouté un livre, l'utilisateur peut valider sa compréhension :
- 10-20 questions QCM (chargées depuis `qcm.json` sur R2)
- 4 choix par question
- Feedback immédiat (correct/incorrect + explication)
- Score final : X/Y → stocké dans `scores[bookId]`
- Seuil de validation : 70% → livre marqué `passed`

```typescript
// Format qcm.json
{
  "questions": [
    {
      "id": "q1",
      "question": "Quelle est la thèse principale de l'Apologie ?",
      "options": [
        "Socrate est innocent et défend la philosophie",
        "Socrate reconnaît sa culpabilité",
        "La démocratie athénienne est juste",
        "Les dieux n'existent pas"
      ],
      "correctIndex": 0,
      "explanation": "Socrate défend son activité philosophique..."
    }
  ]
}
```

### QCM Infini (InfiniteQCMScreen)

Mode sans limite — questions piochées aléatoirement dans tous les livres consultés :

```typescript
// Algorithme de sélection
function pickNextQuestion(
  allBooks: Book[],
  visitedBooks: string[],
  qcmCache: QCMCache,
  excludeRecent: string[]  // Évite les répétitions immédiates
): Question {
  const availableBooks = visitedBooks.filter(id => qcmCache[id]);
  const book = randomPick(availableBooks);
  const bookQuestions = qcmCache[book].questions.filter(
    q => !excludeRecent.includes(q.id)
  );
  return randomPick(bookQuestions);
}
```

**Streak System :**
- Bonne réponse → streak + 1
- Mauvaise réponse → streak reset à 0
- `bestQCMStreak` persisté dans store (Zustand → AsyncStorage)
- Affichage visuel du streak actuel (🔥 avec compteur)

## Citations Partageables

### Flow utilisateur

```
CitationsScreen (liste des citations du livre)
    ↓ Long press sur une citation
CitationCard (preview animée)
    ↓ Capture d'écran (expo-gl ou react-native-view-shot)
CitationShareCard (composant off-screen)
    ↓ Share API native (iOS share sheet)
[Image partagée : carte mise en page avec citation + auteur + livre]
```

### Format de la carte partagée

```
┌─────────────────────────────────┐
│                                 │
│  "La vraie sagesse consiste à   │
│   savoir que l'on ne sait       │
│   rien."                        │
│                                 │
│                    — Socrate    │
│               Apologie de      │
│               Socrate · Platon  │
│                                 │
│  ♦ Sapior                       │
└─────────────────────────────────┘
```

### useCitationShare

```typescript
// hooks/useCitationShare.ts
async function shareCitation(citation: Citation, book: Book) {
  // 1. Rendre CitationShareCard off-screen
  // 2. Capturer avec react-native-view-shot (PNG)
  const imageUri = await captureRef(cardRef, { format: 'png', quality: 1 });

  // 3. Partager via Share API iOS
  await Share.share({
    url: imageUri,
    message: `"${citation.text}" — ${citation.author} · ${book.title} (via Sapior)`
  });
}
```

## Favoris & Listes

### 4 systèmes de favoris distincts

```typescript
// 1. Livres favoris (BookRow → long press → "Ajouter aux favoris")
favoriteBookIds: string[]

// 2. Citations favorites (CitationsScreen → ❤️)
favoriteCitationIds: string[]  // clé "bookId::citationIndex"

// 3. Applications favorites (ApplicationsSheet → ❤️)
favoriteApplicationIds: string[]  // clé "bookId::appIndex"

// 4. Reading List (BookRow → long press → "À lire plus tard")
readingListIds: string[]  // bookmark, pas encore lu
```

### FavoritesScreen (ReadingListScreen)
Affiche les citations et applications favorites avec navigation vers le livre source.

## Journeys Thématiques

### Concept

Un Journey est un parcours guidé multi-livres autour d'un thème philosophique :

```json
{
  "id": "stoiciens",
  "title": "Les grands stoïciens",
  "description": "De Zénon à Marc Aurèle — comprendre le stoïcisme par ses sources",
  "steps": [
    { "bookId": "zenon-stoicisme-fondements", "order": 1 },
    { "bookId": "epictete-entretiens", "order": 2 },
    { "bookId": "marc-aurele-pensees", "order": 3 }
  ]
}
```

### JourneyScreen

```
Journey "Les grands stoïciens"
├── Progression : 1/3 livres complétés
│
├── ① Fondements du stoïcisme (Zénon)    ✅ Terminé
├── ② Entretiens (Épictète)              🔵 En cours (67%)
└── ③ Pensées pour moi-même (Marc Aurèle) ⬜ À démarrer
```

## Applications Pratiques

### Format application.json

Chaque livre sur R2 contient des applications concrètes en deux dimensions :

```json
{
  "applications": [
    {
      "section": "pro",
      "title": "La méthode socratique en réunion",
      "content": "Au lieu d'affirmer vos idées, questionnez vos interlocuteurs...",
      "action_immediate": "Lors de votre prochaine réunion, posez 3 questions ouvertes"
    },
    {
      "section": "perso",
      "title": "Examiner ses certitudes",
      "content": "Identifiez une conviction forte que vous n'avez jamais questionnée...",
      "action_immediate": "Ce soir, choisissez une croyance et cherchez 3 contre-exemples"
    }
  ]
}
```

### ApplicationsSheet

Bottom sheet organisé en 2 onglets (Pro / Perso). Chaque application est une card avec :
- Titre
- Contenu développé
- **Action immédiate** (l'élément le plus important — quelque chose à faire aujourd'hui)
- ❤️ Favoris

Cette feature est une différenciation forte de Sapior vs Audible/Blinkist : pas seulement comprendre, mais *appliquer*.
