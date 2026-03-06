# 6.4 Вложенная навигация

## 1. Суть концепции

Вложенная навигация — несколько независимых стеков навигации в одном приложении. Типичный пример: приложение с `BottomNavigationBar`, где каждая вкладка имеет свой стек экранов.

---

## 2. IndexedStack + Navigator на каждой вкладке

```dart
class MainScreen extends StatefulWidget {
  const MainScreen({super.key});

  @override
  State<MainScreen> createState() => _MainScreenState();
}

class _MainScreenState extends State<MainScreen> {
  int _currentTab = 0;

  // Ключ для каждого Navigator — сохраняет стек
  final _navigatorKeys = [
    GlobalKey<NavigatorState>(),
    GlobalKey<NavigatorState>(),
    GlobalKey<NavigatorState>(),
  ];

  @override
  Widget build(BuildContext context) {
    return PopScope(
      canPop: false,
      onPopInvoked: (didPop) {
        // По кнопке "Назад" — возвращаемся в стеке текущей вкладки
        final navigator = _navigatorKeys[_currentTab].currentState;
        if (navigator != null && navigator.canPop()) {
          navigator.pop();
        }
      },
      child: Scaffold(
        body: IndexedStack(
          index: _currentTab,
          children: [
            _TabNavigator(navigatorKey: _navigatorKeys[0], tab: 'home'),
            _TabNavigator(navigatorKey: _navigatorKeys[1], tab: 'search'),
            _TabNavigator(navigatorKey: _navigatorKeys[2], tab: 'profile'),
          ],
        ),
        bottomNavigationBar: NavigationBar(
          selectedIndex: _currentTab,
          onDestinationSelected: (i) => setState(() => _currentTab = i),
          destinations: const [
            NavigationDestination(icon: Icon(Icons.home), label: 'Главная'),
            NavigationDestination(icon: Icon(Icons.search), label: 'Поиск'),
            NavigationDestination(icon: Icon(Icons.person), label: 'Профиль'),
          ],
        ),
      ),
    );
  }
}

class _TabNavigator extends StatelessWidget {
  final GlobalKey<NavigatorState> navigatorKey;
  final String tab;

  const _TabNavigator({required this.navigatorKey, required this.tab});

  @override
  Widget build(BuildContext context) {
    return Navigator(
      key: navigatorKey,
      onGenerateRoute: (settings) {
        return MaterialPageRoute(
          builder: (_) => switch (tab) {
            'home' => const HomeTab(),
            'search' => const SearchTab(),
            'profile' => const ProfileTab(),
            _ => const HomeTab(),
          },
        );
      },
    );
  }
}
```

---

## 3. StatefulShellRoute в GoRouter (рекомендуется)

```dart
final GoRouter router = GoRouter(
  initialLocation: '/home',
  routes: [
    StatefulShellRoute.indexedStack(
      builder: (context, state, navigationShell) {
        return MainShell(navigationShell: navigationShell);
      },
      branches: [
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/home',
              builder: (_, __) => const HomeTab(),
              routes: [
                GoRoute(
                  path: 'detail/:id',
                  builder: (_, state) => HomeDetailScreen(
                    id: state.pathParameters['id']!,
                  ),
                ),
              ],
            ),
          ],
        ),
        StatefulShellBranch(
          routes: [
            GoRoute(path: '/search', builder: (_, __) => const SearchTab()),
          ],
        ),
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/profile',
              builder: (_, __) => const ProfileTab(),
              routes: [
                GoRoute(
                  path: 'settings',
                  builder: (_, __) => const ProfileSettingsScreen(),
                ),
              ],
            ),
          ],
        ),
      ],
    ),
  ],
);

class MainShell extends StatelessWidget {
  final StatefulNavigationShell navigationShell;
  const MainShell({super.key, required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,  // автоматически отображает нужную ветку
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (i) => navigationShell.goBranch(
          i,
          initialLocation: i == navigationShell.currentIndex,
          // initialLocation: true — вернуться к началу вкладки при повторном тапе
        ),
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home), label: 'Главная'),
          NavigationDestination(icon: Icon(Icons.search), label: 'Поиск'),
          NavigationDestination(icon: Icon(Icons.person), label: 'Профиль'),
        ],
      ),
    );
  }
}
```

---

## 4. Навигация между вкладками

```dart
// Из любого вложенного экрана перейти на другую вкладку
context.go('/search?q=flutter');

// Перейти вглубь текущей вкладки
context.push('/home/detail/42');

// При нажатии на иконку уже активной вкладки — вернуться в начало
navigationShell.goBranch(
  currentIndex,
  initialLocation: true, // сбросить стек вкладки
);
```

---

## 5. Типичные ошибки

| Ошибка                                     | Проблема               | Решение                                           |
| ------------------------------------------ | ---------------------- | ------------------------------------------------- |
| `IndexedStack` без `GlobalKey` у Navigator | Стек не сохраняется    | Каждому вложенному Navigator нужен GlobalKey      |
| Кнопка "Назад" не работает в вкладках      | Нет обработки PopScope | Добавить PopScope с проверкой текущего навигатора |
| `context.go()` из вложенного экрана        | Убирает весь стек      | Использовать `context.push()`                     |
| `ShellRoute` без состояния                 | Вкладки сбрасываются   | Использовать `StatefulShellRoute`                 |

---

## 6. Практические рекомендации

1. **`StatefulShellRoute.indexedStack`** в GoRouter — лучший подход для приложений с вкладками.
2. **Сохранение стека вкладок** важно для UX — пользователь не должен терять навигацию при переключении вкладок.
3. **Двойной тап на иконку вкладки** — возврат в начало вкладки (`initialLocation: true`).
4. **`IndexedStack`** сохраняет состояние всех вкладок в памяти — осторожно с памятью.
5. **Тестируй deep links** — убедись что URL восстанавливает правильное состояние стека.
