# 6.3 GoRouter

## 1. Суть концепции

GoRouter — официально рекомендованный пакет для навигации во Flutter (от Flutter team). Построен поверх Navigator 2.0 (Router API). Поддерживает deep links, URL-навигацию (web), редиректы, вложенные навигаторы и shell routes.

```yaml
dependencies:
  go_router: ^13.0.0
```

---

## 2. Базовая настройка

```dart
import 'package:go_router/go_router.dart';

final GoRouter router = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,  // логировать переходы в debug
  routes: [
    GoRoute(
      path: '/',
      name: 'home',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      name: 'product-detail',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductDetailScreen(productId: id);
      },
    ),
    GoRoute(
      path: '/search',
      builder: (context, state) {
        final query = state.uri.queryParameters['q'] ?? '';
        return SearchScreen(query: query);
      },
    ),
  ],
);

// В main.dart
void main() => runApp(MaterialApp.router(routerConfig: router));
```

---

## 3. Навигация

```dart
// Перейти (заменить стек)
context.go('/product/123');
context.goNamed('product-detail', pathParameters: {'id': '123'});

// Добавить в стек (push)
context.push('/product/123');
context.pushNamed('product-detail', pathParameters: {'id': '42'});

// Назад
context.pop();
context.pop<String>('result');  // вернуть значение

// С query параметрами
context.go('/search?q=flutter');

// Проверить можно ли pop
if (context.canPop()) context.pop();
```

---

## 4. Редиректы — авторизационный guard

```dart
final GoRouter router = GoRouter(
  initialLocation: '/',
  redirect: (context, state) {
    final isLoggedIn = AuthService.isLoggedIn;
    final isLoginPage = state.matchedLocation == '/login';

    if (!isLoggedIn && !isLoginPage) return '/login';  // перенаправить на логин
    if (isLoggedIn && isLoginPage) return '/';          // уже вошёл — на главную

    return null;  // null = не перенаправлять
  },
  routes: [
    GoRoute(path: '/', builder: (_, __) => const HomeScreen()),
    GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
    GoRoute(path: '/profile', builder: (_, __) => const ProfileScreen()),
  ],
)
```

---

## 5. ShellRoute — вложенная навигация с Tab Bar

```dart
final GoRouter router = GoRouter(
  initialLocation: '/home',
  routes: [
    ShellRoute(
      builder: (context, state, child) => MainShell(child: child),
      routes: [
        GoRoute(path: '/home', builder: (_, __) => const HomeTab()),
        GoRoute(path: '/search', builder: (_, __) => const SearchTab()),
        GoRoute(
          path: '/profile',
          builder: (_, __) => const ProfileTab(),
          routes: [
            // Вложенный маршрут внутри вкладки
            GoRoute(
              path: 'settings',
              builder: (_, __) => const ProfileSettingsScreen(),
            ),
          ],
        ),
      ],
    ),
  ],
);

// Shell — Scaffold с NavigationBar
class MainShell extends StatelessWidget {
  final Widget child;
  const MainShell({super.key, required this.child});

  @override
  Widget build(BuildContext context) {
    final location = GoRouterState.of(context).matchedLocation;

    int currentIndex = switch (location) {
      String s when s.startsWith('/home') => 0,
      String s when s.startsWith('/search') => 1,
      String s when s.startsWith('/profile') => 2,
      _ => 0,
    };

    return Scaffold(
      body: child,
      bottomNavigationBar: NavigationBar(
        selectedIndex: currentIndex,
        onDestinationSelected: (i) {
          switch (i) {
            case 0: context.go('/home');
            case 1: context.go('/search');
            case 2: context.go('/profile');
          }
        },
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

## 6. Передача объектов через extra

```dart
// Передача
context.push('/product-detail', extra: product);

// Получение
GoRoute(
  path: '/product-detail',
  builder: (context, state) {
    final product = state.extra as Product;
    return ProductDetailScreen(product: product);
  },
)
```

**Внимание**: `extra` не сохраняется при deep link — используй только для in-app навигации.

---

## 7. Типичные ошибки

| Ошибка                                    | Проблема                                    | Решение                                     |
| ----------------------------------------- | ------------------------------------------- | ------------------------------------------- |
| `context.go()` вместо `context.push()`    | Убирает стек, нельзя вернуться              | Используй `push` для деталей                |
| `extra` в deep links                      | Не восстанавливается при cold start         | Передавай ID через path, объект грузи по ID |
| Нет `errorBuilder`                        | Приложение падает для неизвестных маршрутов | Добавить `errorBuilder` в GoRouter          |
| `ShellRoute` без сохранения стека вкладок | Вкладка сбрасывается при переключении       | Использовать `StatefulShellRoute`           |

---

## 8. Практические рекомендации

1. **`go` для смены раздела** (переход между вкладками), **`push` для деталей** (стек).
2. **`redirect`** для auth guard — одно место для логики авторизации.
3. **`ShellRoute`** для NavigationBar с сохранением состояния вкладок.
4. **`StatefulShellRoute`** если нужно сохранять состояние каждой вкладки.
5. **`debugLogDiagnostics: true`** во время разработки для логирования навигации.
6. **Именованные маршруты** (`name:`) вместо строк при использовании `goNamed`.
