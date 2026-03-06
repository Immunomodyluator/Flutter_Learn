# 6.2 Именованные маршруты

## 1. Суть концепции

Именованные маршруты — навигация по строковым именам (`'/home'`, `'/product/detail'`). Централизованное определение маршрутов упрощает поддержку и позволяет использовать deep links.

**Ограничения**: строки не типизированы, передача данных через `arguments` не безопасна по типам. Для сложных приложений предпочтительнее GoRouter.

---

## 2. Базовая настройка

```dart
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => const HomeScreen(),
    '/products': (context) => const ProductListScreen(),
    '/cart': (context) => const CartScreen(),
    '/profile': (context) => const ProfileScreen(),
    '/settings': (context) => const SettingsScreen(),
  },
)
```

---

## 3. Навигация по имени

```dart
// Перейти на экран
Navigator.pushNamed(context, '/products');

// Вернуться назад
Navigator.pop(context);

// Заменить текущий экран
Navigator.pushReplacementNamed(context, '/home');

// Очистить стек
Navigator.pushNamedAndRemoveUntil(context, '/home', (_) => false);

// Вернуться до именованного маршрута
Navigator.popUntil(context, ModalRoute.withName('/home'));
```

---

## 4. Передача аргументов

```dart
// Отправка
Navigator.pushNamed(
  context,
  '/product-detail',
  arguments: product, // или Map, или любой объект
);

// Получение в нужном экране
class ProductDetailScreen extends StatelessWidget {
  const ProductDetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final product = ModalRoute.of(context)!.settings.arguments as Product;
    return Scaffold(
      appBar: AppBar(title: Text(product.name)),
      body: Text(product.description),
    );
  }
}
```

---

## 5. onGenerateRoute — динамические маршруты

```dart
MaterialApp(
  onGenerateRoute: (settings) {
    // Разбираем URL вручную
    final uri = Uri.parse(settings.name ?? '');

    if (uri.pathSegments.length == 2 && uri.pathSegments.first == 'product') {
      final id = uri.pathSegments[1];
      return MaterialPageRoute(
        builder: (_) => ProductDetailScreen(productId: id),
        settings: settings,
      );
    }

    if (settings.name == '/') {
      return MaterialPageRoute(builder: (_) => const HomeScreen());
    }

    // 404 — неизвестный маршрут
    return MaterialPageRoute(builder: (_) => const NotFoundScreen());
  },
  // onUnknownRoute — запасной вариант
  onUnknownRoute: (settings) => MaterialPageRoute(
    builder: (_) => const NotFoundScreen(),
  ),
)
```

---

## 6. Типизированные маршруты — паттерн с константами

```dart
// Централизованный файл маршрутов
abstract class AppRoutes {
  static const home = '/';
  static const products = '/products';
  static const productDetail = '/product-detail';
  static const cart = '/cart';
  static const profile = '/profile';
}

// Использование
Navigator.pushNamed(context, AppRoutes.productDetail, arguments: product);
```

---

## 7. Обёртка для типобезопасной навигации

```dart
// Расширение для Navigation
extension NavigatorX on BuildContext {
  Future<void> goToProductDetail(Product product) => Navigator.pushNamed(
    this,
    AppRoutes.productDetail,
    arguments: product,
  );

  Future<void> goHome() => Navigator.pushNamedAndRemoveUntil(
    this,
    AppRoutes.home,
    (_) => false,
  );
}

// Использование
context.goToProductDetail(product);
context.goHome();
```

---

## 8. Типичные ошибки

| Ошибка                                | Проблема                      | Решение                                   |
| ------------------------------------- | ----------------------------- | ----------------------------------------- |
| Опечатка в строке маршрута            | Навигация не срабатывает      | Использовать константы `AppRoutes.xxx`    |
| Каст без проверки `as Product`        | Exception если null           | Проверять тип: `if (args is Product)`     |
| Маршрут не зарегистрирован            | Ошибка или 404                | Добавить в `routes` или `onGenerateRoute` |
| `routes` + `onGenerateRoute` конфликт | Прямые routes имеют приоритет | Переносить всё в `onGenerateRoute`        |

---

## 9. Когда переходить на GoRouter

Именованные маршруты достаточны если:

- Небольшое приложение (< 10 экранов)
- Нет web-платформы
- Нет deep links
- Нет сложной вложенной навигации

Переходи на GoRouter если:

- Нужны URL в браузере (Flutter Web)
- Нужны deep links
- Нужны редиректы (auth guard)
- Есть вложенные навигаторы (tabs с own стеком)

---

## 10. Практические рекомендации

1. **Константы маршрутов** — класс `AppRoutes` с константными строками.
2. **`onGenerateRoute`** для параметризованных маршрутов (ID в URL).
3. **`onUnknownRoute`** — всегда добавляй обработку неизвестных маршрутов.
4. **Типизированные обёртки** через extension для безопасной навигации.
5. **Планируй переход на GoRouter** если приложение будет расти.
