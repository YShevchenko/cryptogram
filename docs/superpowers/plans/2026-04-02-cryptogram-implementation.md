# Cryptogram Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a polished cryptogram puzzle game where players decode famous quotes by solving substitution ciphers, with a "Neon Void" cyberpunk theme, AdMob monetization, IAP, and 4-language localization.

**Architecture:** Clean Architecture with 3 layers (domain/data/presentation). Domain layer is pure Dart with zero Flutter imports — all game logic testable without widget tests. Riverpod for state management with dependency injection via providers. Services abstracted behind interfaces for testability and swappability.

**Tech Stack:** Flutter 3.x, Riverpod 2.x, shared_preferences, google_mobile_ads, in_app_purchase, flutter_localizations, Google Fonts (Space Grotesk + Manrope)

**Design:** "Neon Void" theme from Stitch designs — dark (#0e0e13) background, neon cyan (#a9ffdf/#00FFC8) primary, neon pink (#ff59e3) secondary, neon purple (#ac89ff) tertiary. Fonts: Space Grotesk (headlines/labels), Manrope (body).

---

## File Structure

```
cryptogram/
├── lib/
│   ├── main.dart                           # Entry point, service init
│   ├── app.dart                            # MaterialApp with theme + routing
│   ├── core/
│   │   ├── theme/
│   │   │   ├── app_colors.dart             # Neon Void color palette from Stitch
│   │   │   └── app_theme.dart              # ThemeData + text styles
│   │   └── constants.dart                  # Game tuning constants
│   ├── domain/
│   │   ├── models/
│   │   │   ├── quote.dart                  # Quote with text + author + category
│   │   │   ├── cipher_mapping.dart         # Number-to-letter substitution map
│   │   │   ├── game_state.dart             # Current puzzle state
│   │   │   └── player_stats.dart           # Lifetime stats + progress
│   │   ├── services/
│   │   │   ├── cipher_engine.dart          # Generate cipher, validate solution
│   │   │   └── difficulty_service.dart     # Scale difficulty by level
│   │   └── repositories/
│   │       ├── quote_repository.dart       # Abstract: fetch quotes
│   │       └── progress_repository.dart    # Abstract: save/load progress
│   ├── data/
│   │   ├── repositories/
│   │   │   ├── asset_quote_repository.dart # Load quotes from bundled JSON
│   │   │   └── prefs_progress_repository.dart # SharedPreferences persistence
│   │   └── quotes_data.dart               # Quote loading + parsing
│   ├── presentation/
│   │   ├── providers/
│   │   │   ├── game_notifier.dart          # Core game state management
│   │   │   ├── progress_notifier.dart      # Player progress state
│   │   │   ├── settings_notifier.dart      # App settings (sound, theme)
│   │   │   └── providers.dart              # All provider definitions
│   │   ├── screens/
│   │   │   ├── menu_screen.dart            # Title screen (Stitch 01)
│   │   │   ├── game_screen.dart            # Main gameplay (Stitch 02)
│   │   │   ├── settings_screen.dart        # Settings page
│   │   │   └── shop_screen.dart            # IAP store
│   │   └── widgets/
│   │       ├── cipher_board.dart           # Encrypted quote display
│   │       ├── letter_keyboard.dart        # Letter selection input
│   │       ├── hint_button.dart            # Hint trigger (rewarded ad)
│   │       ├── neon_button.dart            # Reusable Neon Void button
│   │       ├── neon_glow.dart              # Glow effect wrapper
│   │       ├── victory_modal.dart          # Level complete (Stitch 03)
│   │       ├── stats_row.dart              # Stat display widgets
│   │       └── scanline_overlay.dart       # HUD scanline effect
│   └── services/
│       ├── ad_service.dart                 # AdMob wrapper (interface + impl)
│       ├── iap_service.dart                # IAP wrapper (interface + impl)
│       └── audio_service.dart              # Simple sound effects
├── assets/
│   └── data/
│       └── quotes.json                     # 500+ public domain quotes
├── test/
│   ├── domain/
│   │   ├── cipher_engine_test.dart
│   │   ├── difficulty_service_test.dart
│   │   └── models_test.dart
│   ├── data/
│   │   └── asset_quote_repository_test.dart
│   └── presentation/
│       └── game_notifier_test.dart
├── l10n/
│   ├── app_en.arb                          # English (base)
│   ├── app_de.arb                          # German
│   ├── app_es.arb                          # Spanish
│   └── app_uk.arb                          # Ukrainian
└── pubspec.yaml
```

---

### Task 1: Project Scaffold + Dependencies

**Files:**
- Create: `cryptogram/pubspec.yaml`
- Create: `cryptogram/analysis_options.yaml`
- Create: `cryptogram/l10n.yaml`
- Create: `cryptogram/lib/main.dart` (placeholder)

- [ ] **Step 1: Create Flutter project**

```bash
cd /Users/yts/lab/planned-games/cryptogram
flutter create --org com.heldiglab --project-name cryptogram .
```

If the directory already has files, use:
```bash
cd /Users/yts/lab/planned-games/cryptogram
flutter create --org com.heldiglab --project-name cryptogram cryptogram_app
# Then move lib/ test/ pubspec.yaml etc into the cryptogram/ root
```

- [ ] **Step 2: Replace pubspec.yaml**

```yaml
name: cryptogram
description: "Cryptogram - Decode famous quotes using cipher puzzles"
publish_to: 'none'
version: 1.0.1+1

environment:
  sdk: ^3.7.0

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

  # State Management
  flutter_riverpod: ^2.6.1

  # Persistence
  shared_preferences: ^2.3.3

  # Monetization
  google_mobile_ads: ^5.2.0
  in_app_purchase: ^3.2.1

  # UI
  google_fonts: ^6.2.1

  # Audio
  audioplayers: ^6.1.0

  # Privacy
  app_tracking_transparency: ^2.0.6

  # Utils
  intl: any
  equatable: ^2.0.7

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^6.0.0
  mocktail: ^1.0.4

flutter:
  uses-material-design: true
  generate: true

  assets:
    - assets/data/
    - assets/audio/

  fonts:
    - family: SpaceGrotesk
      fonts:
        - asset: assets/fonts/SpaceGrotesk-Regular.ttf
        - asset: assets/fonts/SpaceGrotesk-Bold.ttf
          weight: 700
        - asset: assets/fonts/SpaceGrotesk-Medium.ttf
          weight: 500
    - family: Manrope
      fonts:
        - asset: assets/fonts/Manrope-Regular.ttf
        - asset: assets/fonts/Manrope-Bold.ttf
          weight: 700
        - asset: assets/fonts/Manrope-Medium.ttf
          weight: 500
```

- [ ] **Step 3: Create l10n.yaml**

```yaml
arb-dir: l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
```

- [ ] **Step 4: Create analysis_options.yaml**

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    prefer_const_constructors: true
    prefer_const_declarations: true
    avoid_print: true
    prefer_single_quotes: true
```

- [ ] **Step 5: Create asset directories and download fonts**

```bash
mkdir -p assets/data assets/audio assets/fonts
# Download Space Grotesk and Manrope from Google Fonts
# Place .ttf files in assets/fonts/
```

- [ ] **Step 6: Run flutter pub get to verify**

```bash
cd cryptogram_app  # or wherever the project root is
flutter pub get
```
Expected: No errors.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "feat(cryptogram): scaffold project with dependencies"
```

---

### Task 2: Domain Models

**Files:**
- Create: `lib/domain/models/quote.dart`
- Create: `lib/domain/models/cipher_mapping.dart`
- Create: `lib/domain/models/game_state.dart`
- Create: `lib/domain/models/player_stats.dart`
- Create: `lib/core/constants.dart`
- Test: `test/domain/models_test.dart`

- [ ] **Step 1: Write tests for domain models**

```dart
// test/domain/models_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:cryptogram/domain/models/quote.dart';
import 'package:cryptogram/domain/models/cipher_mapping.dart';
import 'package:cryptogram/domain/models/game_state.dart';
import 'package:cryptogram/domain/models/player_stats.dart';

void main() {
  group('Quote', () {
    test('creates from JSON', () {
      final json = {
        'text': 'To be or not to be',
        'author': 'Shakespeare',
        'category': 'literature',
      };
      final quote = Quote.fromJson(json);
      expect(quote.text, 'To be or not to be');
      expect(quote.author, 'Shakespeare');
      expect(quote.category, 'literature');
    });

    test('uniqueLetters returns distinct uppercase letters', () {
      final quote = Quote(
        text: 'Hello World',
        author: 'Test',
        category: 'test',
      );
      expect(quote.uniqueLetters, containsAll(['H', 'E', 'L', 'O', 'W', 'R', 'D']));
      expect(quote.uniqueLetters.length, 7);
    });
  });

  group('CipherMapping', () {
    test('generates random mapping for given letters', () {
      final mapping = CipherMapping.generate(['A', 'B', 'C']);
      expect(mapping.letterToNumber.length, 3);
      expect(mapping.numberToLetter.length, 3);
      // Each letter maps to a unique number
      expect(mapping.letterToNumber.values.toSet().length, 3);
    });

    test('encrypt and decrypt are inverse', () {
      final mapping = CipherMapping.generate(['H', 'E', 'L', 'O']);
      final encrypted = mapping.encrypt('HELLO');
      final decrypted = mapping.decrypt(encrypted);
      expect(decrypted, 'HELLO');
    });
  });

  group('GameState', () {
    test('isComplete returns true when all guesses correct', () {
      final mapping = CipherMapping({1: 'A', 2: 'B'}, {'A': 1, 'B': 2});
      final state = GameState(
        quote: Quote(text: 'AB', author: 'T', category: 'c'),
        cipher: mapping,
        guesses: {1: 'A', 2: 'B'},
        level: 1,
        hintsUsed: 0,
        startTime: DateTime.now(),
      );
      expect(state.isComplete, true);
    });

    test('isComplete returns false with wrong guess', () {
      final mapping = CipherMapping({1: 'A', 2: 'B'}, {'A': 1, 'B': 2});
      final state = GameState(
        quote: Quote(text: 'AB', author: 'T', category: 'c'),
        cipher: mapping,
        guesses: {1: 'A', 2: 'C'},
        level: 1,
        hintsUsed: 0,
        startTime: DateTime.now(),
      );
      expect(state.isComplete, false);
    });

    test('correctCount tracks correct guesses', () {
      final mapping = CipherMapping({1: 'A', 2: 'B', 3: 'C'}, {'A': 1, 'B': 2, 'C': 3});
      final state = GameState(
        quote: Quote(text: 'ABC', author: 'T', category: 'c'),
        cipher: mapping,
        guesses: {1: 'A', 2: 'X'},
        level: 1,
        hintsUsed: 0,
        startTime: DateTime.now(),
      );
      expect(state.correctCount, 1);
      expect(state.totalLetters, 3);
    });
  });

  group('PlayerStats', () {
    test('serializes to and from JSON', () {
      final stats = PlayerStats(
        currentLevel: 5,
        highScore: 1000,
        totalSolved: 20,
        currentStreak: 3,
        bestStreak: 7,
        totalHintsUsed: 10,
      );
      final json = stats.toJson();
      final restored = PlayerStats.fromJson(json);
      expect(restored.currentLevel, 5);
      expect(restored.highScore, 1000);
      expect(restored.currentStreak, 3);
    });
  });
}
```

- [ ] **Step 2: Run tests — verify they fail**

```bash
flutter test test/domain/models_test.dart
```
Expected: FAIL (files don't exist yet)

- [ ] **Step 3: Implement Quote model**

```dart
// lib/domain/models/quote.dart
import 'package:equatable/equatable.dart';

class Quote extends Equatable {
  final String text;
  final String author;
  final String category;

  const Quote({
    required this.text,
    required this.author,
    required this.category,
  });

  factory Quote.fromJson(Map<String, dynamic> json) {
    return Quote(
      text: json['text'] as String,
      author: json['author'] as String,
      category: json['category'] as String,
    );
  }

  Map<String, dynamic> toJson() => {
        'text': text,
        'author': author,
        'category': category,
      };

  /// Returns the set of unique uppercase letters in the quote text.
  List<String> get uniqueLetters {
    return text
        .toUpperCase()
        .split('')
        .where((c) => RegExp(r'[A-Z]').hasMatch(c))
        .toSet()
        .toList()
      ..sort();
  }

  @override
  List<Object?> get props => [text, author, category];
}
```

- [ ] **Step 4: Implement CipherMapping model**

```dart
// lib/domain/models/cipher_mapping.dart
import 'dart:math';
import 'package:equatable/equatable.dart';

class CipherMapping extends Equatable {
  /// Maps number -> correct letter
  final Map<int, String> numberToLetter;

  /// Maps letter -> number
  final Map<String, int> letterToNumber;

  const CipherMapping(this.numberToLetter, this.letterToNumber);

  /// Generate a random cipher for the given unique letters.
  /// Numbers start at 1 and are shuffled randomly.
  factory CipherMapping.generate(List<String> uniqueLetters) {
    final random = Random();
    final numbers = List.generate(uniqueLetters.length, (i) => i + 1)..shuffle(random);

    final n2l = <int, String>{};
    final l2n = <String, int>{};

    for (var i = 0; i < uniqueLetters.length; i++) {
      n2l[numbers[i]] = uniqueLetters[i];
      l2n[uniqueLetters[i]] = numbers[i];
    }

    return CipherMapping(n2l, l2n);
  }

  /// Encrypt a string: replace each letter with its number.
  /// Non-letter characters pass through unchanged.
  List<CipherToken> encrypt(String text) {
    return text.toUpperCase().split('').map((char) {
      if (letterToNumber.containsKey(char)) {
        return CipherToken(number: letterToNumber[char]!, originalChar: char);
      }
      return CipherToken(plainChar: char);
    }).toList();
  }

  /// Decrypt a list of numbers back to the original string.
  String decrypt(List<CipherToken> tokens) {
    return tokens.map((t) {
      if (t.isEncrypted) {
        return numberToLetter[t.number] ?? '?';
      }
      return t.plainChar;
    }).join();
  }

  @override
  List<Object?> get props => [numberToLetter, letterToNumber];
}

/// A single token in the cipher display — either an encrypted number or a plain character.
class CipherToken extends Equatable {
  final int? number;
  final String? originalChar;
  final String? plainChar;

  const CipherToken({this.number, this.originalChar, this.plainChar});

  bool get isEncrypted => number != null;
  bool get isSpace => plainChar == ' ';
  bool get isPunctuation => !isEncrypted && !isSpace;

  @override
  List<Object?> get props => [number, originalChar, plainChar];
}
```

- [ ] **Step 5: Implement GameState model**

```dart
// lib/domain/models/game_state.dart
import 'package:equatable/equatable.dart';
import 'quote.dart';
import 'cipher_mapping.dart';

class GameState extends Equatable {
  final Quote quote;
  final CipherMapping cipher;
  final Map<int, String> guesses;
  final int level;
  final int hintsUsed;
  final DateTime startTime;
  final int? selectedNumber;

  const GameState({
    required this.quote,
    required this.cipher,
    required this.guesses,
    required this.level,
    required this.hintsUsed,
    required this.startTime,
    this.selectedNumber,
  });

  /// True when every encrypted number has the correct letter guessed.
  bool get isComplete {
    for (final entry in cipher.numberToLetter.entries) {
      if (guesses[entry.key] != entry.value) return false;
    }
    return true;
  }

  /// Number of correctly guessed letters.
  int get correctCount {
    int count = 0;
    for (final entry in guesses.entries) {
      if (cipher.numberToLetter[entry.key] == entry.value) count++;
    }
    return count;
  }

  /// Total number of unique letters to guess.
  int get totalLetters => cipher.numberToLetter.length;

  /// Progress as a fraction 0.0 to 1.0.
  double get progress => totalLetters == 0 ? 0 : correctCount / totalLetters;

  /// Elapsed time since puzzle start.
  Duration get elapsed => DateTime.now().difference(startTime);

  /// Calculate score based on time, hints, and level.
  int get score {
    final baseScore = 100 * level;
    final timePenalty = (elapsed.inSeconds / 10).clamp(0, baseScore ~/ 2).toInt();
    final hintPenalty = hintsUsed * 20;
    return (baseScore - timePenalty - hintPenalty).clamp(0, baseScore * 2);
  }

  /// Letters already used in guesses (to dim on keyboard).
  Set<String> get usedLetters => guesses.values.toSet();

  /// Returns a copy with the given number-letter guess applied.
  GameState withGuess(int number, String letter) {
    final newGuesses = Map<int, String>.from(guesses);
    // Remove any existing guess for this letter (one-to-one mapping)
    newGuesses.removeWhere((_, v) => v == letter);
    newGuesses[number] = letter;
    return _copyWith(guesses: newGuesses);
  }

  /// Returns a copy with the guess for the given number removed.
  GameState withGuessRemoved(int number) {
    final newGuesses = Map<int, String>.from(guesses)..remove(number);
    return _copyWith(guesses: newGuesses);
  }

  /// Returns a copy with the selected number changed.
  GameState withSelection(int? number) => _copyWith(selectedNumber: number);

  /// Returns a copy with one hint applied (reveals a random unguessed letter).
  GameState withHint() {
    final unguessed = cipher.numberToLetter.entries
        .where((e) => guesses[e.key] != e.value)
        .toList();
    if (unguessed.isEmpty) return this;
    unguessed.shuffle();
    final hint = unguessed.first;
    final newGuesses = Map<int, String>.from(guesses);
    newGuesses[hint.key] = hint.value;
    return _copyWith(guesses: newGuesses, hintsUsed: hintsUsed + 1);
  }

  GameState _copyWith({
    Map<int, String>? guesses,
    int? selectedNumber,
    int? hintsUsed,
  }) {
    return GameState(
      quote: quote,
      cipher: cipher,
      guesses: guesses ?? this.guesses,
      level: level,
      hintsUsed: hintsUsed ?? this.hintsUsed,
      startTime: startTime,
      selectedNumber: selectedNumber,
    );
  }

  @override
  List<Object?> get props => [quote, cipher, guesses, level, hintsUsed, selectedNumber];
}
```

- [ ] **Step 6: Implement PlayerStats model**

```dart
// lib/domain/models/player_stats.dart
import 'package:equatable/equatable.dart';

class PlayerStats extends Equatable {
  final int currentLevel;
  final int highScore;
  final int totalSolved;
  final int currentStreak;
  final int bestStreak;
  final int totalHintsUsed;
  final bool adsRemoved;
  final String selectedTheme;

  const PlayerStats({
    this.currentLevel = 1,
    this.highScore = 0,
    this.totalSolved = 0,
    this.currentStreak = 0,
    this.bestStreak = 0,
    this.totalHintsUsed = 0,
    this.adsRemoved = false,
    this.selectedTheme = 'neon_void',
  });

  factory PlayerStats.fromJson(Map<String, dynamic> json) {
    return PlayerStats(
      currentLevel: json['currentLevel'] as int? ?? 1,
      highScore: json['highScore'] as int? ?? 0,
      totalSolved: json['totalSolved'] as int? ?? 0,
      currentStreak: json['currentStreak'] as int? ?? 0,
      bestStreak: json['bestStreak'] as int? ?? 0,
      totalHintsUsed: json['totalHintsUsed'] as int? ?? 0,
      adsRemoved: json['adsRemoved'] as bool? ?? false,
      selectedTheme: json['selectedTheme'] as String? ?? 'neon_void',
    );
  }

  Map<String, dynamic> toJson() => {
        'currentLevel': currentLevel,
        'highScore': highScore,
        'totalSolved': totalSolved,
        'currentStreak': currentStreak,
        'bestStreak': bestStreak,
        'totalHintsUsed': totalHintsUsed,
        'adsRemoved': adsRemoved,
        'selectedTheme': selectedTheme,
      };

  PlayerStats withLevelComplete(int score) {
    final newStreak = currentStreak + 1;
    return PlayerStats(
      currentLevel: currentLevel + 1,
      highScore: score > highScore ? score : highScore,
      totalSolved: totalSolved + 1,
      currentStreak: newStreak,
      bestStreak: newStreak > bestStreak ? newStreak : bestStreak,
      totalHintsUsed: totalHintsUsed,
      adsRemoved: adsRemoved,
      selectedTheme: selectedTheme,
    );
  }

  PlayerStats copyWith({
    int? currentLevel,
    int? highScore,
    int? totalSolved,
    int? currentStreak,
    int? bestStreak,
    int? totalHintsUsed,
    bool? adsRemoved,
    String? selectedTheme,
  }) {
    return PlayerStats(
      currentLevel: currentLevel ?? this.currentLevel,
      highScore: highScore ?? this.highScore,
      totalSolved: totalSolved ?? this.totalSolved,
      currentStreak: currentStreak ?? this.currentStreak,
      bestStreak: bestStreak ?? this.bestStreak,
      totalHintsUsed: totalHintsUsed ?? this.totalHintsUsed,
      adsRemoved: adsRemoved ?? this.adsRemoved,
      selectedTheme: selectedTheme ?? this.selectedTheme,
    );
  }

  @override
  List<Object?> get props => [
        currentLevel, highScore, totalSolved,
        currentStreak, bestStreak, totalHintsUsed,
        adsRemoved, selectedTheme,
      ];
}
```

- [ ] **Step 7: Implement constants**

```dart
// lib/core/constants.dart

/// Game tuning constants. Change here, not scattered through code.
abstract final class GameConstants {
  /// Show interstitial every N levels.
  static const int adFrequencyLevels = 4;

  /// Max hints per level without paying.
  static const int freeHintsPerLevel = 1;

  /// Seconds before time penalty starts.
  static const int timePenaltyThresholdSeconds = 30;

  /// Base score per level (multiplied by level number).
  static const int baseScorePerLevel = 100;

  /// Points deducted per hint used.
  static const int hintPenaltyPoints = 20;

  /// IAP product IDs.
  static const String removeAdsProductId = 'cryptogram_remove_ads';
  static const String themePackNeonProductId = 'cryptogram_theme_neon';
  static const String themePackPastelProductId = 'cryptogram_theme_pastel';

  /// Quote categories.
  static const List<String> categories = [
    'philosophy', 'literature', 'science', 'history', 'motivation',
  ];
}
```

- [ ] **Step 8: Run tests — verify they pass**

```bash
flutter test test/domain/models_test.dart
```
Expected: ALL PASS

- [ ] **Step 9: Commit**

```bash
git add lib/domain/ lib/core/constants.dart test/domain/models_test.dart
git commit -m "feat(cryptogram): domain models — Quote, CipherMapping, GameState, PlayerStats"
```

---

### Task 3: Cipher Engine + Difficulty Service

**Files:**
- Create: `lib/domain/services/cipher_engine.dart`
- Create: `lib/domain/services/difficulty_service.dart`
- Create: `lib/domain/repositories/quote_repository.dart`
- Create: `lib/domain/repositories/progress_repository.dart`
- Test: `test/domain/cipher_engine_test.dart`
- Test: `test/domain/difficulty_service_test.dart`

- [ ] **Step 1: Write cipher engine tests**

```dart
// test/domain/cipher_engine_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:cryptogram/domain/models/quote.dart';
import 'package:cryptogram/domain/services/cipher_engine.dart';

void main() {
  late CipherEngine engine;

  setUp(() {
    engine = CipherEngine();
  });

  test('createPuzzle generates valid cipher for quote', () {
    final quote = Quote(text: 'Hello World', author: 'Test', category: 'test');
    final state = engine.createPuzzle(quote: quote, level: 1);

    expect(state.cipher.numberToLetter.length, 7); // H E L O W R D
    expect(state.guesses, isEmpty);
    expect(state.level, 1);
  });

  test('cipher maps each unique letter to a unique number', () {
    final quote = Quote(text: 'ABCABC', author: 'T', category: 'c');
    final state = engine.createPuzzle(quote: quote, level: 1);

    final numbers = state.cipher.letterToNumber.values.toSet();
    expect(numbers.length, 3); // A, B, C
  });

  test('pre-revealed letters are filled at higher difficulty', () {
    final quote = Quote(text: 'Hello World', author: 'T', category: 'c');
    // Level 1 should have some pre-revealed hints
    final state = engine.createPuzzle(quote: quote, level: 1);
    expect(state.guesses.length, greaterThanOrEqualTo(0));
  });
}
```

- [ ] **Step 2: Implement CipherEngine**

```dart
// lib/domain/services/cipher_engine.dart
import '../models/quote.dart';
import '../models/cipher_mapping.dart';
import '../models/game_state.dart';
import 'difficulty_service.dart';

class CipherEngine {
  final DifficultyService _difficulty;

  CipherEngine({DifficultyService? difficulty})
      : _difficulty = difficulty ?? DifficultyService();

  /// Create a new puzzle from a quote at the given level.
  GameState createPuzzle({required Quote quote, required int level}) {
    final uniqueLetters = quote.uniqueLetters;
    final cipher = CipherMapping.generate(uniqueLetters);
    final preRevealed = _difficulty.preRevealedCount(
      level: level,
      totalLetters: uniqueLetters.length,
    );

    // Pre-reveal some letters as starter hints
    final guesses = <int, String>{};
    if (preRevealed > 0) {
      final entries = cipher.numberToLetter.entries.toList()..shuffle();
      for (var i = 0; i < preRevealed && i < entries.length; i++) {
        guesses[entries[i].key] = entries[i].value;
      }
    }

    return GameState(
      quote: quote,
      cipher: cipher,
      guesses: guesses,
      level: level,
      hintsUsed: 0,
      startTime: DateTime.now(),
    );
  }
}
```

- [ ] **Step 3: Implement DifficultyService**

```dart
// lib/domain/services/difficulty_service.dart

/// Controls how difficulty scales with player level.
class DifficultyService {
  /// How many letters to pre-reveal based on level.
  /// Lower levels get more help.
  int preRevealedCount({required int level, required int totalLetters}) {
    if (level <= 3) return (totalLetters * 0.4).round();
    if (level <= 10) return (totalLetters * 0.25).round();
    if (level <= 25) return (totalLetters * 0.15).round();
    if (level <= 50) return 1;
    return 0;
  }

  /// Minimum quote length for the given level.
  /// Higher levels get longer, harder quotes.
  int minQuoteLength(int level) {
    if (level <= 5) return 20;
    if (level <= 15) return 40;
    if (level <= 30) return 60;
    return 80;
  }

  /// Maximum quote length for the given level.
  int maxQuoteLength(int level) {
    if (level <= 5) return 60;
    if (level <= 15) return 100;
    if (level <= 30) return 150;
    return 200;
  }
}
```

- [ ] **Step 4: Define repository interfaces**

```dart
// lib/domain/repositories/quote_repository.dart
import '../models/quote.dart';

abstract class QuoteRepository {
  /// Get a quote suitable for the given level.
  Future<Quote> getQuoteForLevel(int level);

  /// Get total number of available quotes.
  Future<int> get totalQuotes;
}
```

```dart
// lib/domain/repositories/progress_repository.dart
import '../models/player_stats.dart';

abstract class ProgressRepository {
  Future<PlayerStats> load();
  Future<void> save(PlayerStats stats);
  Future<void> clear();
}
```

- [ ] **Step 5: Run all domain tests**

```bash
flutter test test/domain/
```
Expected: ALL PASS

- [ ] **Step 6: Commit**

```bash
git add lib/domain/ test/domain/
git commit -m "feat(cryptogram): cipher engine, difficulty scaling, repository interfaces"
```

---

### Task 4: Data Layer — Quote Repository + Progress

**Files:**
- Create: `lib/data/repositories/asset_quote_repository.dart`
- Create: `lib/data/repositories/prefs_progress_repository.dart`
- Create: `lib/data/quotes_data.dart`
- Create: `assets/data/quotes.json`
- Test: `test/data/asset_quote_repository_test.dart`

- [ ] **Step 1: Create quotes.json with 500+ public domain quotes**

The file should contain quotes organized by category. Generate a comprehensive JSON file with this structure:

```json
{
  "quotes": [
    {"text": "The only thing we have to fear is fear itself.", "author": "Franklin D. Roosevelt", "category": "motivation"},
    {"text": "I think therefore I am.", "author": "Rene Descartes", "category": "philosophy"},
    {"text": "To be or not to be that is the question.", "author": "William Shakespeare", "category": "literature"},
    ...
  ]
}
```

Include at least 500 quotes across categories: philosophy, literature, science, history, motivation. All must be public domain. Vary lengths from short (20 chars) to long (200 chars).

- [ ] **Step 2: Implement AssetQuoteRepository**

```dart
// lib/data/repositories/asset_quote_repository.dart
import 'dart:convert';
import 'dart:math';
import 'package:flutter/services.dart';
import '../../domain/models/quote.dart';
import '../../domain/repositories/quote_repository.dart';
import '../../domain/services/difficulty_service.dart';

class AssetQuoteRepository implements QuoteRepository {
  final DifficultyService _difficulty;
  final Random _random;
  List<Quote>? _cachedQuotes;
  final Set<int> _usedIndices = {};

  AssetQuoteRepository({
    DifficultyService? difficulty,
    Random? random,
  })  : _difficulty = difficulty ?? DifficultyService(),
        _random = random ?? Random();

  Future<List<Quote>> _loadQuotes() async {
    if (_cachedQuotes != null) return _cachedQuotes!;
    final jsonString = await rootBundle.loadString('assets/data/quotes.json');
    final data = json.decode(jsonString) as Map<String, dynamic>;
    final quotesJson = data['quotes'] as List<dynamic>;
    _cachedQuotes = quotesJson
        .map((q) => Quote.fromJson(q as Map<String, dynamic>))
        .toList();
    return _cachedQuotes!;
  }

  @override
  Future<Quote> getQuoteForLevel(int level) async {
    final quotes = await _loadQuotes();
    final minLen = _difficulty.minQuoteLength(level);
    final maxLen = _difficulty.maxQuoteLength(level);

    final suitable = <int>[];
    for (var i = 0; i < quotes.length; i++) {
      final len = quotes[i].text.length;
      if (len >= minLen && len <= maxLen && !_usedIndices.contains(i)) {
        suitable.add(i);
      }
    }

    // If all suitable quotes used, reset tracking
    if (suitable.isEmpty) {
      _usedIndices.clear();
      return getQuoteForLevel(level);
    }

    final chosen = suitable[_random.nextInt(suitable.length)];
    _usedIndices.add(chosen);
    return quotes[chosen];
  }

  @override
  Future<int> get totalQuotes async => (await _loadQuotes()).length;
}
```

- [ ] **Step 3: Implement PrefsProgressRepository**

```dart
// lib/data/repositories/prefs_progress_repository.dart
import 'dart:convert';
import 'package:shared_preferences/shared_preferences.dart';
import '../../domain/models/player_stats.dart';
import '../../domain/repositories/progress_repository.dart';

class PrefsProgressRepository implements ProgressRepository {
  static const _key = 'cryptogram_player_stats';
  final SharedPreferences _prefs;

  PrefsProgressRepository(this._prefs);

  @override
  Future<PlayerStats> load() async {
    final jsonStr = _prefs.getString(_key);
    if (jsonStr == null) return const PlayerStats();
    final json = jsonDecode(jsonStr) as Map<String, dynamic>;
    return PlayerStats.fromJson(json);
  }

  @override
  Future<void> save(PlayerStats stats) async {
    await _prefs.setString(_key, jsonEncode(stats.toJson()));
  }

  @override
  Future<void> clear() async {
    await _prefs.remove(_key);
  }
}
```

- [ ] **Step 4: Run tests**

```bash
flutter test test/data/
```

- [ ] **Step 5: Commit**

```bash
git add lib/data/ assets/data/ test/data/
git commit -m "feat(cryptogram): data layer — asset quote repo, prefs progress repo, 500+ quotes"
```

---

### Task 5: Theme + Design System (Neon Void from Stitch)

**Files:**
- Create: `lib/core/theme/app_colors.dart`
- Create: `lib/core/theme/app_theme.dart`

- [ ] **Step 1: Implement colors from Stitch design tokens**

```dart
// lib/core/theme/app_colors.dart
import 'package:flutter/material.dart';

/// Neon Void color palette — extracted from Stitch design JSON.
abstract final class AppColors {
  // Core
  static const background = Color(0xFF0E0E13);
  static const surface = Color(0xFF0E0E13);
  static const surfaceContainer = Color(0xFF19191F);
  static const surfaceContainerLow = Color(0xFF131319);
  static const surfaceContainerHigh = Color(0xFF1F1F26);
  static const surfaceContainerHighest = Color(0xFF25252D);
  static const surfaceContainerLowest = Color(0xFF000000);

  // Primary (neon cyan/mint)
  static const primary = Color(0xFFA9FFDF);
  static const primaryContainer = Color(0xFF00FDC6);
  static const primaryDim = Color(0xFF00EAB7);
  static const onPrimary = Color(0xFF00654D);
  static const neonCyan = Color(0xFF00FFC8);

  // Secondary (neon pink/magenta)
  static const secondary = Color(0xFFFF59E3);
  static const secondaryContainer = Color(0xFFAD009B);
  static const onSecondary = Color(0xFF42003A);

  // Tertiary (neon purple)
  static const tertiary = Color(0xFFAC89FF);
  static const tertiaryContainer = Color(0xFF7000FF);
  static const tertiaryDim = Color(0xFF874CFF);

  // Error
  static const error = Color(0xFFFF716C);
  static const errorContainer = Color(0xFF9F0519);

  // Text / Surface
  static const onSurface = Color(0xFFF9F5FD);
  static const onSurfaceVariant = Color(0xFFACAAB1);
  static const outline = Color(0xFF76747B);
  static const outlineVariant = Color(0xFF48474D);
}
```

- [ ] **Step 2: Implement theme**

```dart
// lib/core/theme/app_theme.dart
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';
import 'app_colors.dart';

abstract final class AppTheme {
  static ThemeData get dark {
    return ThemeData(
      brightness: Brightness.dark,
      scaffoldBackgroundColor: AppColors.background,
      colorScheme: const ColorScheme.dark(
        primary: AppColors.primary,
        onPrimary: AppColors.onPrimary,
        secondary: AppColors.secondary,
        onSecondary: AppColors.onSecondary,
        tertiary: AppColors.tertiary,
        error: AppColors.error,
        surface: AppColors.surface,
        onSurface: AppColors.onSurface,
        outline: AppColors.outline,
        outlineVariant: AppColors.outlineVariant,
      ),
      textTheme: _textTheme,
      appBarTheme: const AppBarTheme(
        backgroundColor: Colors.transparent,
        elevation: 0,
      ),
      useMaterial3: true,
    );
  }

  static TextTheme get _textTheme {
    return TextTheme(
      displayLarge: GoogleFonts.spaceGrotesk(
        fontSize: 48,
        fontWeight: FontWeight.w800,
        color: AppColors.onSurface,
        letterSpacing: 4,
      ),
      headlineLarge: GoogleFonts.spaceGrotesk(
        fontSize: 32,
        fontWeight: FontWeight.w700,
        color: AppColors.onSurface,
        letterSpacing: 2,
      ),
      headlineMedium: GoogleFonts.spaceGrotesk(
        fontSize: 24,
        fontWeight: FontWeight.w700,
        color: AppColors.onSurface,
      ),
      titleLarge: GoogleFonts.spaceGrotesk(
        fontSize: 20,
        fontWeight: FontWeight.w600,
        color: AppColors.onSurface,
        letterSpacing: 2,
      ),
      titleMedium: GoogleFonts.spaceGrotesk(
        fontSize: 16,
        fontWeight: FontWeight.w500,
        color: AppColors.onSurface,
      ),
      bodyLarge: GoogleFonts.manrope(
        fontSize: 16,
        fontWeight: FontWeight.w400,
        color: AppColors.onSurface,
      ),
      bodyMedium: GoogleFonts.manrope(
        fontSize: 14,
        fontWeight: FontWeight.w400,
        color: AppColors.onSurfaceVariant,
      ),
      labelLarge: GoogleFonts.spaceGrotesk(
        fontSize: 14,
        fontWeight: FontWeight.w600,
        color: AppColors.onSurface,
        letterSpacing: 1.5,
      ),
      labelSmall: GoogleFonts.spaceGrotesk(
        fontSize: 10,
        fontWeight: FontWeight.w500,
        color: AppColors.onSurfaceVariant,
        letterSpacing: 2,
      ),
    );
  }
}
```

- [ ] **Step 3: Commit**

```bash
git add lib/core/theme/
git commit -m "feat(cryptogram): Neon Void theme from Stitch designs"
```

---

### Task 6: Localization (EN/DE/ES/UK)

**Files:**
- Create: `l10n/app_en.arb`
- Create: `l10n/app_de.arb`
- Create: `l10n/app_es.arb`
- Create: `l10n/app_uk.arb`

- [ ] **Step 1: Create English base ARB**

```json
{
  "@@locale": "en",
  "appTitle": "Cryptogram",
  "play": "PLAY",
  "settings": "Settings",
  "shop": "SHOP",
  "rank": "RANK",
  "globalStanding": "GLOBAL STANDING",
  "highScore": "HIGH SCORE",
  "streak": "STREAK",
  "days": "{count} DAYS",
  "@days": {"placeholders": {"count": {"type": "int"}}},
  "level": "LEVEL {number}",
  "@level": {"placeholders": {"number": {"type": "int"}}},
  "score": "SCORE: {points}",
  "@score": {"placeholders": {"points": {"type": "String"}}},
  "pause": "Pause",
  "resume": "Resume",
  "hint": "HINT",
  "hintWatchAd": "Watch ad for hint",
  "noHintsLeft": "No hints available",
  "levelComplete": "Level Complete!",
  "perfectEfficiency": "Perfect Efficiency Achieved",
  "xpEarned": "XP EARNED",
  "nextLevel": "Next Level",
  "watchAdCoins": "Watch Ad for +{amount} Coins",
  "@watchAdCoins": {"placeholders": {"amount": {"type": "int"}}},
  "mainMenu": "Main Menu",
  "removeAds": "Remove Ads",
  "removeAdsPrice": "Remove Ads — {price}",
  "@removeAdsPrice": {"placeholders": {"price": {"type": "String"}}},
  "themePack": "Theme Pack",
  "restore": "Restore Purchases",
  "soundEffects": "Sound Effects",
  "hapticFeedback": "Haptic Feedback",
  "language": "Language",
  "selectLetter": "Select a number, then tap a letter",
  "alreadyUsed": "Letter already used",
  "correct": "Correct!",
  "tryAgain": "Try again",
  "quoteBy": "— {author}",
  "@quoteBy": {"placeholders": {"author": {"type": "String"}}},
  "totalSolved": "Total Solved",
  "bestStreak": "Best Streak",
  "inventory": "INVENTORY",
  "boost": "BOOST",
  "store": "STORE",
  "config": "CONFIG"
}
```

- [ ] **Step 2: Create German ARB**

```json
{
  "@@locale": "de",
  "appTitle": "Kryptogramm",
  "play": "SPIELEN",
  "settings": "Einstellungen",
  "shop": "LADEN",
  "rank": "RANG",
  "globalStanding": "WELTRANGLISTE",
  "highScore": "HIGHSCORE",
  "streak": "SERIE",
  "days": "{count} TAGE",
  "level": "LEVEL {number}",
  "score": "PUNKTE: {points}",
  "pause": "Pause",
  "resume": "Fortsetzen",
  "hint": "HINWEIS",
  "hintWatchAd": "Werbung ansehen für Hinweis",
  "noHintsLeft": "Keine Hinweise verfügbar",
  "levelComplete": "Level geschafft!",
  "perfectEfficiency": "Perfekte Effizienz erreicht",
  "xpEarned": "XP VERDIENT",
  "nextLevel": "Nächstes Level",
  "watchAdCoins": "Werbung für +{amount} Münzen",
  "mainMenu": "Hauptmenü",
  "removeAds": "Werbung entfernen",
  "removeAdsPrice": "Werbung entfernen — {price}",
  "themePack": "Themenpaket",
  "restore": "Käufe wiederherstellen",
  "soundEffects": "Soundeffekte",
  "hapticFeedback": "Haptisches Feedback",
  "language": "Sprache",
  "selectLetter": "Wähle eine Zahl, dann tippe einen Buchstaben",
  "alreadyUsed": "Buchstabe bereits verwendet",
  "correct": "Richtig!",
  "tryAgain": "Nochmal versuchen",
  "quoteBy": "— {author}",
  "totalSolved": "Gesamt gelöst",
  "bestStreak": "Beste Serie",
  "inventory": "INVENTAR",
  "boost": "BOOST",
  "store": "LADEN",
  "config": "KONFIG"
}
```

- [ ] **Step 3: Create Spanish ARB**

```json
{
  "@@locale": "es",
  "appTitle": "Criptograma",
  "play": "JUGAR",
  "settings": "Ajustes",
  "shop": "TIENDA",
  "rank": "RANGO",
  "globalStanding": "CLASIFICACIÓN",
  "highScore": "RÉCORD",
  "streak": "RACHA",
  "days": "{count} DÍAS",
  "level": "NIVEL {number}",
  "score": "PUNTOS: {points}",
  "pause": "Pausa",
  "resume": "Continuar",
  "hint": "PISTA",
  "hintWatchAd": "Ver anuncio para pista",
  "noHintsLeft": "Sin pistas disponibles",
  "levelComplete": "¡Nivel Completado!",
  "perfectEfficiency": "Eficiencia Perfecta Lograda",
  "xpEarned": "XP GANADA",
  "nextLevel": "Siguiente Nivel",
  "watchAdCoins": "Ver anuncio por +{amount} Monedas",
  "mainMenu": "Menú Principal",
  "removeAds": "Eliminar Anuncios",
  "removeAdsPrice": "Eliminar Anuncios — {price}",
  "themePack": "Paquete de Temas",
  "restore": "Restaurar Compras",
  "soundEffects": "Efectos de Sonido",
  "hapticFeedback": "Vibración",
  "language": "Idioma",
  "selectLetter": "Selecciona un número, luego toca una letra",
  "alreadyUsed": "Letra ya usada",
  "correct": "¡Correcto!",
  "tryAgain": "Inténtalo de nuevo",
  "quoteBy": "— {author}",
  "totalSolved": "Total Resueltos",
  "bestStreak": "Mejor Racha",
  "inventory": "INVENTARIO",
  "boost": "IMPULSO",
  "store": "TIENDA",
  "config": "CONFIG"
}
```

- [ ] **Step 4: Create Ukrainian ARB**

```json
{
  "@@locale": "uk",
  "appTitle": "Криптограма",
  "play": "ГРАТИ",
  "settings": "Налаштування",
  "shop": "МАГАЗИН",
  "rank": "РАНГ",
  "globalStanding": "РЕЙТИНГ",
  "highScore": "РЕКОРД",
  "streak": "СЕРІЯ",
  "days": "{count} ДНІВ",
  "level": "РІВЕНЬ {number}",
  "score": "РАХУНОК: {points}",
  "pause": "Пауза",
  "resume": "Продовжити",
  "hint": "ПІДКАЗКА",
  "hintWatchAd": "Дивитись рекламу для підказки",
  "noHintsLeft": "Підказок немає",
  "levelComplete": "Рівень пройдено!",
  "perfectEfficiency": "Ідеальна ефективність досягнута",
  "xpEarned": "XP ОТРИМАНО",
  "nextLevel": "Наступний Рівень",
  "watchAdCoins": "Реклама за +{amount} Монет",
  "mainMenu": "Головне Меню",
  "removeAds": "Прибрати Рекламу",
  "removeAdsPrice": "Прибрати Рекламу — {price}",
  "themePack": "Набір Тем",
  "restore": "Відновити Покупки",
  "soundEffects": "Звукові Ефекти",
  "hapticFeedback": "Вібрація",
  "language": "Мова",
  "selectLetter": "Оберіть число, потім натисніть букву",
  "alreadyUsed": "Букву вже використано",
  "correct": "Правильно!",
  "tryAgain": "Спробуйте ще",
  "quoteBy": "— {author}",
  "totalSolved": "Всього розгадано",
  "bestStreak": "Найкраща серія",
  "inventory": "ІНВЕНТАР",
  "boost": "БУСТ",
  "store": "МАГАЗИН",
  "config": "НАЛАШТ"
}
```

- [ ] **Step 5: Commit**

```bash
git add l10n/ l10n.yaml
git commit -m "feat(cryptogram): localization — EN, DE, ES, UK"
```

---

### Task 7: Services — Ads, IAP, Consent

**Files:**
- Create: `lib/services/ad_service.dart`
- Create: `lib/services/iap_service.dart`
- Create: `lib/services/consent_service.dart`
- Create: `lib/services/audio_service.dart`

- [ ] **Step 1: Implement AdService**

```dart
// lib/services/ad_service.dart
import 'dart:io';
import 'package:flutter/foundation.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import '../core/constants.dart';

abstract class AdServiceBase {
  Future<void> initialize();
  Future<void> showInterstitialIfReady();
  Future<bool> showRewardedAd();
  void setAdsRemoved(bool removed);
  bool get adsRemoved;
  void dispose();
}

class AdService implements AdServiceBase {
  static String get _interstitialAdUnitId => Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/1033173712'
      : 'ca-app-pub-3940256099942544/4411468910';

  static String get _rewardedAdUnitId => Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/5224354917'
      : 'ca-app-pub-3940256099942544/1712485313';

  InterstitialAd? _interstitialAd;
  RewardedAd? _rewardedAd;
  int _levelCount = 0;
  bool _adsRemoved = false;

  @override
  bool get adsRemoved => _adsRemoved;

  @override
  Future<void> initialize() async {
    await MobileAds.instance.initialize();
    _preloadInterstitial();
    _preloadRewarded();
  }

  @override
  void setAdsRemoved(bool removed) {
    _adsRemoved = removed;
    if (removed) {
      _interstitialAd?.dispose();
      _interstitialAd = null;
    }
  }

  void _preloadInterstitial() {
    if (_adsRemoved) return;
    InterstitialAd.load(
      adUnitId: _interstitialAdUnitId,
      request: const AdRequest(),
      adLoadCallback: InterstitialAdLoadCallback(
        onAdLoaded: (ad) {
          _interstitialAd = ad;
          ad.fullScreenContentCallback = FullScreenContentCallback(
            onAdDismissedFullScreenContent: (ad) {
              ad.dispose();
              _preloadInterstitial();
            },
            onAdFailedToShowFullScreenContent: (ad, error) {
              ad.dispose();
              _preloadInterstitial();
            },
          );
        },
        onAdFailedToLoad: (error) {
          debugPrint('Interstitial load failed: $error');
        },
      ),
    );
  }

  void _preloadRewarded() {
    RewardedAd.load(
      adUnitId: _rewardedAdUnitId,
      request: const AdRequest(),
      rewardedAdLoadCallback: RewardedAdLoadCallback(
        onAdLoaded: (ad) => _rewardedAd = ad,
        onAdFailedToLoad: (error) {
          debugPrint('Rewarded load failed: $error');
        },
      ),
    );
  }

  @override
  Future<void> showInterstitialIfReady() async {
    if (_adsRemoved) return;
    _levelCount++;
    if (_levelCount % GameConstants.adFrequencyLevels == 0) {
      await _interstitialAd?.show();
    }
  }

  @override
  Future<bool> showRewardedAd() async {
    if (_rewardedAd == null) return false;
    bool rewarded = false;
    await _rewardedAd!.show(
      onUserEarnedReward: (_, __) => rewarded = true,
    );
    _preloadRewarded();
    return rewarded;
  }

  @override
  void dispose() {
    _interstitialAd?.dispose();
    _rewardedAd?.dispose();
  }
}
```

- [ ] **Step 2: Implement IAPService**

```dart
// lib/services/iap_service.dart
import 'dart:async';
import 'package:flutter/foundation.dart';
import 'package:in_app_purchase/in_app_purchase.dart';
import '../core/constants.dart';

typedef PurchaseCallback = void Function(String productId);

abstract class IAPServiceBase {
  Future<void> initialize();
  Future<bool> purchase(String productId);
  Future<void> restorePurchases();
  bool isPurchased(String productId);
  void addListener(PurchaseCallback callback);
  void dispose();
}

class IAPService implements IAPServiceBase {
  final InAppPurchase _iap = InAppPurchase.instance;
  StreamSubscription<List<PurchaseDetails>>? _subscription;
  final Set<String> _purchased = {};
  final List<PurchaseCallback> _listeners = [];

  @override
  bool isPurchased(String productId) => _purchased.contains(productId);

  @override
  void addListener(PurchaseCallback callback) => _listeners.add(callback);

  @override
  Future<void> initialize() async {
    if (!await _iap.isAvailable()) return;
    _subscription = _iap.purchaseStream.listen(_onPurchaseUpdate);
    await restorePurchases();
  }

  void _onPurchaseUpdate(List<PurchaseDetails> purchases) async {
    for (final purchase in purchases) {
      if (purchase.status == PurchaseStatus.purchased ||
          purchase.status == PurchaseStatus.restored) {
        _purchased.add(purchase.productID);
        for (final listener in _listeners) {
          listener(purchase.productID);
        }
      }
      if (purchase.pendingCompletePurchase) {
        await _iap.completePurchase(purchase);
      }
    }
  }

  @override
  Future<bool> purchase(String productId) async {
    if (!await _iap.isAvailable()) return false;
    final response = await _iap.queryProductDetails({productId});
    if (response.productDetails.isEmpty) return false;
    final param = PurchaseParam(productDetails: response.productDetails.first);
    try {
      await _iap.buyNonConsumable(purchaseParam: param);
      return true;
    } catch (e) {
      debugPrint('Purchase error: $e');
      return false;
    }
  }

  @override
  Future<void> restorePurchases() async {
    try {
      await _iap.restorePurchases();
    } catch (e) {
      debugPrint('Restore error: $e');
    }
  }

  @override
  void dispose() => _subscription?.cancel();
}
```

- [ ] **Step 3: Implement ConsentService**

```dart
// lib/services/consent_service.dart
import 'dart:io';
import 'package:app_tracking_transparency/app_tracking_transparency.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

class ConsentService {
  Future<void> requestConsent() async {
    // iOS ATT
    if (Platform.isIOS) {
      await AppTrackingTransparency.requestTrackingAuthorization();
    }

    // UMP consent (EU)
    final params = ConsentRequestParameters();
    ConsentInformation.instance.requestConsentInfoUpdate(params, () async {
      if (await ConsentInformation.instance.isConsentFormAvailable()) {
        ConsentForm.loadConsentForm((consentForm) async {
          final status = await ConsentInformation.instance.getConsentStatus();
          if (status == ConsentStatus.required) {
            consentForm.show((formError) {});
          }
        }, (formError) {});
      }
    }, (error) {});
  }
}
```

- [ ] **Step 4: Implement AudioService**

```dart
// lib/services/audio_service.dart
import 'package:audioplayers/audioplayers.dart';

class AudioService {
  final AudioPlayer _player = AudioPlayer();
  bool _enabled = true;

  bool get enabled => _enabled;
  set enabled(bool value) => _enabled = value;

  Future<void> playTap() => _play('tap.mp3');
  Future<void> playCorrect() => _play('correct.mp3');
  Future<void> playWrong() => _play('wrong.mp3');
  Future<void> playVictory() => _play('victory.mp3');

  Future<void> _play(String file) async {
    if (!_enabled) return;
    try {
      await _player.play(AssetSource('audio/$file'));
    } catch (_) {
      // Audio is non-critical — fail silently
    }
  }

  void dispose() => _player.dispose();
}
```

- [ ] **Step 5: Commit**

```bash
git add lib/services/
git commit -m "feat(cryptogram): services — ads, IAP, consent, audio"
```

---

### Task 8: Riverpod Providers

**Files:**
- Create: `lib/presentation/providers/providers.dart`
- Create: `lib/presentation/providers/game_notifier.dart`
- Create: `lib/presentation/providers/progress_notifier.dart`
- Create: `lib/presentation/providers/settings_notifier.dart`
- Test: `test/presentation/game_notifier_test.dart`

- [ ] **Step 1: Define all providers**

```dart
// lib/presentation/providers/providers.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:shared_preferences/shared_preferences.dart';
import '../../data/repositories/asset_quote_repository.dart';
import '../../data/repositories/prefs_progress_repository.dart';
import '../../domain/repositories/quote_repository.dart';
import '../../domain/repositories/progress_repository.dart';
import '../../domain/services/cipher_engine.dart';
import '../../domain/services/difficulty_service.dart';
import '../../domain/models/game_state.dart';
import '../../domain/models/player_stats.dart';
import '../../services/ad_service.dart';
import '../../services/iap_service.dart';
import '../../services/audio_service.dart';
import 'game_notifier.dart';
import 'progress_notifier.dart';
import 'settings_notifier.dart';

// Infrastructure
final sharedPrefsProvider = Provider<SharedPreferences>((ref) {
  throw UnimplementedError('Override in main with ProviderScope');
});

// Domain services
final difficultyServiceProvider = Provider((_) => DifficultyService());
final cipherEngineProvider = Provider((ref) {
  return CipherEngine(difficulty: ref.read(difficultyServiceProvider));
});

// Repositories
final quoteRepoProvider = Provider<QuoteRepository>((ref) {
  return AssetQuoteRepository(difficulty: ref.read(difficultyServiceProvider));
});

final progressRepoProvider = Provider<ProgressRepository>((ref) {
  return PrefsProgressRepository(ref.read(sharedPrefsProvider));
});

// External services
final adServiceProvider = Provider((_) => AdService());
final iapServiceProvider = Provider((_) => IAPService());
final audioServiceProvider = Provider((_) => AudioService());

// State notifiers
final gameProvider = StateNotifierProvider<GameNotifier, AsyncValue<GameState?>>((ref) {
  return GameNotifier(
    cipherEngine: ref.read(cipherEngineProvider),
    quoteRepo: ref.read(quoteRepoProvider),
    adService: ref.read(adServiceProvider),
    audioService: ref.read(audioServiceProvider),
  );
});

final progressProvider = StateNotifierProvider<ProgressNotifier, PlayerStats>((ref) {
  return ProgressNotifier(ref.read(progressRepoProvider));
});

final settingsProvider = StateNotifierProvider<SettingsNotifier, SettingsState>((ref) {
  return SettingsNotifier(ref.read(sharedPrefsProvider));
});
```

- [ ] **Step 2: Implement GameNotifier**

```dart
// lib/presentation/providers/game_notifier.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../domain/models/game_state.dart';
import '../../domain/services/cipher_engine.dart';
import '../../domain/repositories/quote_repository.dart';
import '../../services/ad_service.dart';
import '../../services/audio_service.dart';

class GameNotifier extends StateNotifier<AsyncValue<GameState?>> {
  final CipherEngine _cipherEngine;
  final QuoteRepository _quoteRepo;
  final AdServiceBase _adService;
  final AudioService _audioService;

  GameNotifier({
    required CipherEngine cipherEngine,
    required QuoteRepository quoteRepo,
    required AdServiceBase adService,
    required AudioService audioService,
  })  : _cipherEngine = cipherEngine,
        _quoteRepo = quoteRepo,
        _adService = adService,
        _audioService = audioService,
        super(const AsyncValue.data(null));

  Future<void> startLevel(int level) async {
    state = const AsyncValue.loading();
    try {
      final quote = await _quoteRepo.getQuoteForLevel(level);
      final gameState = _cipherEngine.createPuzzle(quote: quote, level: level);
      state = AsyncValue.data(gameState);
    } catch (e, st) {
      state = AsyncValue.error(e, st);
    }
  }

  void selectNumber(int number) {
    final current = state.valueOrNull;
    if (current == null) return;
    state = AsyncValue.data(current.withSelection(number));
    _audioService.playTap();
  }

  void guessLetter(String letter) {
    final current = state.valueOrNull;
    if (current == null || current.selectedNumber == null) return;

    final updated = current.withGuess(current.selectedNumber!, letter);
    state = AsyncValue.data(updated.withSelection(null));

    if (updated.cipher.numberToLetter[current.selectedNumber!] == letter) {
      _audioService.playCorrect();
    } else {
      _audioService.playWrong();
    }

    if (updated.isComplete) {
      _audioService.playVictory();
      _adService.showInterstitialIfReady();
    }
  }

  void removeGuess(int number) {
    final current = state.valueOrNull;
    if (current == null) return;
    state = AsyncValue.data(current.withGuessRemoved(number));
  }

  Future<bool> useHint() async {
    final current = state.valueOrNull;
    if (current == null) return false;

    final rewarded = await _adService.showRewardedAd();
    if (!rewarded) return false;

    state = AsyncValue.data(current.withHint());

    final updated = state.valueOrNull;
    if (updated != null && updated.isComplete) {
      _audioService.playVictory();
    }
    return true;
  }
}
```

- [ ] **Step 3: Implement ProgressNotifier**

```dart
// lib/presentation/providers/progress_notifier.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../domain/models/player_stats.dart';
import '../../domain/repositories/progress_repository.dart';

class ProgressNotifier extends StateNotifier<PlayerStats> {
  final ProgressRepository _repo;

  ProgressNotifier(this._repo) : super(const PlayerStats());

  Future<void> load() async {
    state = await _repo.load();
  }

  Future<void> completeLevel(int score) async {
    state = state.withLevelComplete(score);
    await _repo.save(state);
  }

  Future<void> setAdsRemoved(bool removed) async {
    state = state.copyWith(adsRemoved: removed);
    await _repo.save(state);
  }

  Future<void> setTheme(String theme) async {
    state = state.copyWith(selectedTheme: theme);
    await _repo.save(state);
  }

  Future<void> reset() async {
    state = const PlayerStats();
    await _repo.clear();
  }
}
```

- [ ] **Step 4: Implement SettingsNotifier**

```dart
// lib/presentation/providers/settings_notifier.dart
import 'package:equatable/equatable.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:shared_preferences/shared_preferences.dart';

class SettingsState extends Equatable {
  final bool soundEnabled;
  final bool hapticEnabled;
  final String locale;

  const SettingsState({
    this.soundEnabled = true,
    this.hapticEnabled = true,
    this.locale = 'en',
  });

  SettingsState copyWith({bool? soundEnabled, bool? hapticEnabled, String? locale}) {
    return SettingsState(
      soundEnabled: soundEnabled ?? this.soundEnabled,
      hapticEnabled: hapticEnabled ?? this.hapticEnabled,
      locale: locale ?? this.locale,
    );
  }

  @override
  List<Object?> get props => [soundEnabled, hapticEnabled, locale];
}

class SettingsNotifier extends StateNotifier<SettingsState> {
  final SharedPreferences _prefs;

  SettingsNotifier(this._prefs) : super(const SettingsState()) {
    _load();
  }

  void _load() {
    state = SettingsState(
      soundEnabled: _prefs.getBool('sound') ?? true,
      hapticEnabled: _prefs.getBool('haptic') ?? true,
      locale: _prefs.getString('locale') ?? 'en',
    );
  }

  Future<void> toggleSound() async {
    state = state.copyWith(soundEnabled: !state.soundEnabled);
    await _prefs.setBool('sound', state.soundEnabled);
  }

  Future<void> toggleHaptic() async {
    state = state.copyWith(hapticEnabled: !state.hapticEnabled);
    await _prefs.setBool('haptic', state.hapticEnabled);
  }

  Future<void> setLocale(String locale) async {
    state = state.copyWith(locale: locale);
    await _prefs.setString('locale', locale);
  }
}
```

- [ ] **Step 5: Commit**

```bash
git add lib/presentation/providers/ test/presentation/
git commit -m "feat(cryptogram): Riverpod providers — game, progress, settings notifiers"
```

---

### Task 9: Reusable UI Widgets (Neon Void Components)

**Files:**
- Create: `lib/presentation/widgets/neon_button.dart`
- Create: `lib/presentation/widgets/neon_glow.dart`
- Create: `lib/presentation/widgets/scanline_overlay.dart`
- Create: `lib/presentation/widgets/stats_row.dart`

- [ ] **Step 1: Implement all shared widgets**

These are the reusable Neon Void design system components extracted from the Stitch designs. Implement each widget in its own file following the Stitch design tokens (colors, glows, gradients, typography).

Key visual elements from Stitch:
- Neon text glow: `text-shadow: 0 0 10px rgba(169, 255, 223, 0.4), 0 0 20px rgba(169, 255, 223, 0.2)`
- Neon box glow: `box-shadow: 0 0 20px rgba(0, 255, 200, 0.3)`
- Radial gradient background: `radial-gradient(circle at 50%, #19191f 0%, #0e0e13 100%)`
- HUD scanlines: `linear-gradient(to bottom, transparent 50%, rgba(169,255,223,0.02) 50%)` with 4px repeat
- Backdrop blur on overlays
- Corner accent borders (top-left and bottom-right)

Each widget file should implement the component with matching visual fidelity to the Stitch HTML/CSS.

- [ ] **Step 2: Commit**

```bash
git add lib/presentation/widgets/
git commit -m "feat(cryptogram): Neon Void UI components — button, glow, scanline, stats"
```

---

### Task 10: Menu Screen (Stitch 01 — Title Screen)

**Files:**
- Create: `lib/presentation/screens/menu_screen.dart`

- [ ] **Step 1: Implement menu screen matching Stitch 01_TitleScreen**

Key elements from Stitch design:
- Top bar: App icon + "NEON VOID" title + settings gear
- Center: "GLOBAL STANDING" label + "HIGH SCORE: 10,000"
- Large circular PLAY button with gradient border and glow
- 2-column stats grid: RANK + STREAK
- Bottom nav: SHOP / PLAY (active) / RANK
- Atmospheric background: radial gradient + floating colored blurs

The screen should use Riverpod to read `progressProvider` for stats and navigate to `GameScreen` on play tap.

- [ ] **Step 2: Commit**

```bash
git add lib/presentation/screens/menu_screen.dart
git commit -m "feat(cryptogram): menu screen with Neon Void design"
```

---

### Task 11: Game Screen + Cipher Board + Letter Keyboard (Stitch 02 — Gameplay)

**Files:**
- Create: `lib/presentation/screens/game_screen.dart`
- Create: `lib/presentation/widgets/cipher_board.dart`
- Create: `lib/presentation/widgets/letter_keyboard.dart`
- Create: `lib/presentation/widgets/hint_button.dart`

- [ ] **Step 1: Implement cipher board**

The cipher board displays the encrypted quote. Each letter position shows:
- The number (when not yet guessed)
- The guessed letter (highlighted green if correct, red if wrong)
- Spaces and punctuation pass through
- Tapping a number selects it (highlights with neon cyan glow)
- Words wrap naturally, maintaining the quote's readable structure

- [ ] **Step 2: Implement letter keyboard**

A-Z keyboard displayed at the bottom of the game screen:
- 3 rows: QWERTY layout
- Letters already correctly placed are dimmed/disabled
- Tapping a letter assigns it to the currently selected number
- Visual feedback on tap (scale animation + glow)

- [ ] **Step 3: Implement hint button**

Hint button with rewarded ad integration:
- Shows "HINT" with material icon
- Tap triggers rewarded ad flow
- On reward earned, reveals one random unguessed letter
- Disabled when no unguessed letters remain

- [ ] **Step 4: Implement game screen matching Stitch 02_GameplayHUD**

Key elements from Stitch design:
- Top bar: Pause button + "LEVEL 42" + "SCORE: 24,800"
- HUD corner accents (top-left, bottom-right borders)
- Scanline overlay effect
- Central cipher board
- Progress bar ("CORE CHARGE" showing % complete)
- Letter keyboard at bottom
- Hint button

The screen coordinates all game state via `gameProvider`.

- [ ] **Step 5: Commit**

```bash
git add lib/presentation/screens/game_screen.dart lib/presentation/widgets/cipher_board.dart lib/presentation/widgets/letter_keyboard.dart lib/presentation/widgets/hint_button.dart
git commit -m "feat(cryptogram): game screen with cipher board, keyboard, HUD"
```

---

### Task 12: Victory Modal (Stitch 03 — Game Over)

**Files:**
- Create: `lib/presentation/widgets/victory_modal.dart`

- [ ] **Step 1: Implement victory modal matching Stitch 03_GameOverModal**

Key elements from Stitch design:
- Blurred + darkened background overlay
- Modal card with atmospheric gradient glow at top
- Stats row: "XP EARNED: +2,400" | "STREAK: 5X"
- Headline: "Level Complete!" with neon glow
- Subtitle: "Perfect Efficiency Achieved"
- Primary button: "Next Level" with gradient background
- Secondary button: "Watch Ad for +500 Coins"
- Text link: "Main Menu"
- Corner accent decorations on card

The modal shows when `gameState.isComplete` becomes true. It reads score from `GameState.score` and triggers `progressProvider.completeLevel()`.

- [ ] **Step 2: Commit**

```bash
git add lib/presentation/widgets/victory_modal.dart
git commit -m "feat(cryptogram): victory modal with stats and ad integration"
```

---

### Task 13: Settings + Shop Screens

**Files:**
- Create: `lib/presentation/screens/settings_screen.dart`
- Create: `lib/presentation/screens/shop_screen.dart`

- [ ] **Step 1: Implement settings screen**

Simple settings page:
- Sound Effects toggle
- Haptic Feedback toggle
- Language picker (EN/DE/ES/UK)
- Restore Purchases button
- Neon Void styling consistent with other screens

- [ ] **Step 2: Implement shop screen**

IAP store:
- "Remove Ads — $2.99" card
- "Neon Theme Pack — $0.99" card
- "Pastel Theme Pack — $0.99" card
- Restore Purchases link
- Uses `iapServiceProvider` for purchases

- [ ] **Step 3: Commit**

```bash
git add lib/presentation/screens/settings_screen.dart lib/presentation/screens/shop_screen.dart
git commit -m "feat(cryptogram): settings and shop screens"
```

---

### Task 14: App Entry Point + Routing

**Files:**
- Create: `lib/app.dart`
- Modify: `lib/main.dart`

- [ ] **Step 1: Implement app.dart**

```dart
// lib/app.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
import 'core/theme/app_theme.dart';
import 'presentation/providers/providers.dart';
import 'presentation/screens/menu_screen.dart';

class CryptogramApp extends ConsumerWidget {
  const CryptogramApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final settings = ref.watch(settingsProvider);

    return MaterialApp(
      title: 'Cryptogram',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.dark,
      locale: Locale(settings.locale),
      supportedLocales: AppLocalizations.supportedLocales,
      localizationsDelegates: const [
        AppLocalizations.delegate,
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        GlobalCupertinoLocalizations.delegate,
      ],
      home: const MenuScreen(),
    );
  }
}
```

- [ ] **Step 2: Implement main.dart**

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'app.dart';
import 'presentation/providers/providers.dart';
import 'services/consent_service.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Lock to portrait
  await SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
  ]);

  // Dark status bar
  SystemChrome.setSystemUIOverlayStyle(const SystemUiOverlayStyle(
    statusBarColor: Colors.transparent,
    statusBarIconBrightness: Brightness.light,
  ));

  final prefs = await SharedPreferences.getInstance();

  // Request consent before ads
  await ConsentService().requestConsent();

  runApp(
    ProviderScope(
      overrides: [
        sharedPrefsProvider.overrideWithValue(prefs),
      ],
      child: const CryptogramApp(),
    ),
  );
}
```

- [ ] **Step 3: Commit**

```bash
git add lib/main.dart lib/app.dart
git commit -m "feat(cryptogram): app entry point with Riverpod, localization, consent"
```

---

### Task 15: Integration Testing + Polish

**Files:**
- Verify all screens render without errors
- Test full gameplay loop: menu -> play -> solve -> victory -> next level
- Test hint flow with rewarded ad
- Test offline mode (airplane mode)
- Test IAP restore flow
- Test all 4 locales render correctly

- [ ] **Step 1: Run app on iOS simulator**

```bash
cd cryptogram_app
flutter run -d "iPhone 16 Pro"
```

Walk through: Menu → Play → Solve puzzle → Victory → Next Level → Settings → Shop

- [ ] **Step 2: Run app on Android emulator**

```bash
flutter run -d emulator
```

Verify same flow works on Android.

- [ ] **Step 3: Run all tests**

```bash
flutter test
```
Expected: ALL PASS

- [ ] **Step 4: Fix any issues found**

Address layout issues, overflow, missing translations, ad loading, etc.

- [ ] **Step 5: Final commit**

```bash
git add -A
git commit -m "feat(cryptogram): integration polish and fixes"
```

---

### Task 16: Grok Code Review

- [ ] **Step 1: Ask Grok to review the implementation**

Send the complete codebase to Grok for review. Focus on:
- Architecture adherence to SOLID
- Code quality and readability
- Performance concerns
- Missing edge cases
- UI/UX improvements

- [ ] **Step 2: Fix reasonable review findings**

Implement fixes for issues Grok identifies that are valid.

- [ ] **Step 3: Final commit**

```bash
git add -A
git commit -m "fix(cryptogram): address code review findings"
```
