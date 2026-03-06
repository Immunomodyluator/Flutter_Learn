# 15.3: Размер приложения — деferred loading и tree shaking

> Project: FitMenu | Глава 15 — Performance

### 15.3: Уменьшение размера APK/IPA

🎯 **Цель шага:** Уменьшить размер релизного APK FitMenu через deferred (отложенную) загрузку экрана аналитики, иконки Material Symbols и tree shaking неиспользуемого кода.

---

📝 **Техническое задание:**

**1. Анализ размера:**
```bash
# Сборка split-APK с анализом
flutter build apk --analyze-size
flutter build appbundle --analyze-size

# Результат: .flutter-size-<platform>.json
# Открыть в: https://flutter.github.io/devtools/app-size
```

**2. Deferred загрузка тяжёлого экрана:**
```dart
// Импортируем экран аналитики отложенно
import 'package:fitmenu/features/analytics/analytics_screen.dart' deferred as analytics;

class _HomeState extends State<HomeScreen> {
  bool _analyticsLoaded = false;

  Future<void> _openAnalytics() async {
    if (!_analyticsLoaded) {
      await analytics.loadLibrary();   // скачивается при первом открытии
      _analyticsLoaded = true;
    }
    if (mounted) {
      Navigator.push(context, MaterialPageRoute(
        builder: (_) => analytics.AnalyticsScreen(),
      ));
    }
  }
}
```

**3. Иконки — используй только нужные:**
```yaml
# pubspec.yaml
flutter:
  uses-material-design: true
  # НЕ включай весь пакет если нужно 5 иконок
  # Используй flutter_svg + свои SVG иконки
```

**4. ProGuard для Android (минификация):**
```
# android/app/proguard-rules.pro
-keep class io.flutter.** { *; }
-dontwarn io.flutter.embedding.**
```

**5. Задание:**
- Собери `flutter build apk --release --split-per-abi`
- Запусти `flutter build apk --analyze-size`
- Найди топ-3 самых тяжёлых компонента
- Реализуй deferred loading для AnalyticsScreen
- Сравни размер до/после

---

✅ **Критерии приёмки:**
- [ ] `deferred as` используется для AnalyticsScreen
- [ ] `loadLibrary()` вызывается перед первым использованием
- [ ] `flutter build apk --split-per-abi` создаёт 3 APK меньшего размера
- [ ] `--analyze-size` не показывает неиспользуемые зависимости > 1MB
- [ ] Релизная сборка использует `--obfuscate --split-debug-info=`

---

💡 **Подсказка:** `--split-per-abi` создаёт отдельные APK для arm64, arm32, x86_64 — каждый на 30-40% меньше универсального. `flutter build appbundle` для Play Store (Google сам делает split). Deferred loading работает только в release/profile режиме.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/analytics/analytics_screen.dart
// (обычный файл — ничего особенного)

class AnalyticsScreen extends StatelessWidget {
  const AnalyticsScreen({super.key});
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Аналитика питания')),
      body: const _AnalyticsContent(),
    );
  }
}
```

```dart
// lib/features/home/presentation/home_screen.dart

import 'package:fitmenu/features/analytics/analytics_screen.dart' deferred as analytics;

class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});
  @override State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  bool _analyticsLibraryLoaded = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('FitMenu'),
        actions: [
          IconButton(
            icon: const Icon(Icons.bar_chart_rounded),
            tooltip: 'Аналитика',
            onPressed: _navigateToAnalytics,
          ),
        ],
      ),
      body: const _HomeContent(),
    );
  }

  Future<void> _navigateToAnalytics() async {
    // Показываем индикатор загрузки во время скачивания библиотеки
    if (!_analyticsLibraryLoaded) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Загрузка аналитики...'),
          duration: Duration(seconds: 1),
        ),
      );
      await analytics.loadLibrary();
      _analyticsLibraryLoaded = true;
    }

    if (!mounted) return;

    Navigator.push<void>(
      context,
      MaterialPageRoute(
        builder: (_) => analytics.AnalyticsScreen(),
      ),
    );
  }
}
```

```bash
# Команды для анализа размера
flutter build apk --release --split-per-abi
flutter build apk --analyze-size --target-platform android-arm64
flutter build appbundle --release --obfuscate --split-debug-info=build/debug-info/
```

</details>
