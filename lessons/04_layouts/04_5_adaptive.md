# 4.5 Адаптивный и отзывчивый дизайн

## 1. Суть концепции

**Адаптивный** — разные лейауты для разных форм-факторов (телефон / планшет / десктоп).  
**Отзывчивый** — UI гибко масштабируется под любой размер экрана.

Flutter изначально кроссплатформенный, поэтому адаптивность нужна не только для мобильных.

---

## 2. MediaQuery — размер и ориентация экрана

```dart
@override
Widget build(BuildContext context) {
  final size = MediaQuery.of(context).size;
  final orientation = MediaQuery.of(context).orientation;
  final padding = MediaQuery.of(context).padding;        // SafeArea отступы
  final viewInsets = MediaQuery.of(context).viewInsets; // высота клавиатуры
  final textScale = MediaQuery.of(context).textScaler;   // масштаб текста

  return Padding(
    padding: EdgeInsets.only(bottom: viewInsets.bottom), // над клавиатурой
    child: Column(
      children: [
        Text('Ширина: ${size.width.toInt()}'),
        if (orientation == Orientation.landscape)
          const Text('Ландшафтная ориентация'),
      ],
    ),
  );
}
```

---

## 3. LayoutBuilder — ограничения родителя

```dart
// LayoutBuilder даёт constraints родителя, а не всего экрана
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth > 600) {
      // планшет / широкий экран
      return const TwoColumnLayout();
    }
    // телефон
    return const SingleColumnLayout();
  },
)
```

---

## 4. Breakpoints — точки перелома лейаута

```dart
// Вспомогательный класс для breakpoints
class ScreenBreakpoints {
  static const double mobile = 600;
  static const double tablet = 900;
  static const double desktop = 1200;
}

// Адаптивная сетка
class AdaptiveGrid extends StatelessWidget {
  final List<Widget> children;
  const AdaptiveGrid({super.key, required this.children});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        int columns;
        if (constraints.maxWidth >= ScreenBreakpoints.desktop) {
          columns = 4;
        } else if (constraints.maxWidth >= ScreenBreakpoints.tablet) {
          columns = 3;
        } else if (constraints.maxWidth >= ScreenBreakpoints.mobile) {
          columns = 2;
        } else {
          columns = 1;
        }

        return GridView.count(
          crossAxisCount: columns,
          childAspectRatio: 1,
          crossAxisSpacing: 8,
          mainAxisSpacing: 8,
          shrinkWrap: true,
          physics: const NeverScrollableScrollPhysics(),
          children: children,
        );
      },
    );
  }
}
```

---

## 5. SafeArea — отступы для вырезов и закруглений

```dart
// SafeArea добавляет padding для notch, home indicator, status bar
Scaffold(
  body: SafeArea(
    top: true,
    bottom: true,
    child: Column(children: [...]),
  ),
)

// Для кастомного контента
Scaffold(
  body: Stack(
    children: [
      // Фоновое изображение на весь экран (без SafeArea)
      Positioned.fill(child: Image.network(url, fit: BoxFit.cover)),
      // Контент с SafeArea
      SafeArea(
        child: Column(children: [...]),
      ),
    ],
  ),
)
```

---

## 6. Адаптивный NavigationBar / NavigationRail / Drawer

```dart
class AdaptiveScaffold extends StatefulWidget {
  const AdaptiveScaffold({super.key});

  @override
  State<AdaptiveScaffold> createState() => _AdaptiveScaffoldState();
}

class _AdaptiveScaffoldState extends State<AdaptiveScaffold> {
  int _selectedIndex = 0;

  @override
  Widget build(BuildContext context) {
    final isWide = MediaQuery.of(context).size.width >= 600;

    return Scaffold(
      body: Row(
        children: [
          if (isWide)
            NavigationRail(
              selectedIndex: _selectedIndex,
              onDestinationSelected: (i) => setState(() => _selectedIndex = i),
              labelType: NavigationRailLabelType.all,
              destinations: const [
                NavigationRailDestination(icon: Icon(Icons.home), label: Text('Главная')),
                NavigationRailDestination(icon: Icon(Icons.search), label: Text('Поиск')),
                NavigationRailDestination(icon: Icon(Icons.person), label: Text('Профиль')),
              ],
            ),
          Expanded(child: _pages[_selectedIndex]),
        ],
      ),
      bottomNavigationBar: isWide ? null : NavigationBar(
        selectedIndex: _selectedIndex,
        onDestinationSelected: (i) => setState(() => _selectedIndex = i),
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home), label: 'Главная'),
          NavigationDestination(icon: Icon(Icons.search), label: 'Поиск'),
          NavigationDestination(icon: Icon(Icons.person), label: 'Профиль'),
        ],
      ),
    );
  }

  static const _pages = [
    Center(child: Text('Главная')),
    Center(child: Text('Поиск')),
    Center(child: Text('Профиль')),
  ];
}
```

---

## 7. Относительные размеры

```dart
// FractionallySizedBox — доля от родителя
FractionallySizedBox(
  widthFactor: 0.8,     // 80% ширины родителя
  child: FilledButton(onPressed: () {}, child: const Text('Кнопка')),
)

// Использование MediaQuery для вычислений
final screenWidth = MediaQuery.of(context).size.width;
Container(
  width: screenWidth * 0.5,  // 50% экрана
  ...
)
```

---

## 8. Практические рекомендации

1. **`LayoutBuilder` предпочтительнее `MediaQuery.size`** — даёт ограничения конкретного виджета, а не экрана.
2. **`SafeArea` на каждом экране** — особенно для экранов с bottom navigation.
3. **Breakpoints — не пиксели, а смысловые точки** — телефон / планшет / десктоп.
4. **`NavigationRail` для планшетов** и `BottomNavigationBar` для телефонов.
5. **Тестируй на разных размерах** — Flutter DevTools → Device Simulator.
6. **`FractionallySizedBox`** для кнопок, чтобы не задавать фиксированный `width`.
