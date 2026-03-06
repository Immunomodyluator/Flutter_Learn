# 6. Навигация и маршрутизация

## 1. Суть концепции

Навигация во Flutter — управление стеком экранов. Flutter предоставляет три подхода:

1. **Navigator 1.0** — императивный (push/pop)
2. **Navigator 2.0** — декларативный (Router API)
3. **GoRouter** — пакет поверх Router API, де-факто стандарт

---

## 2. Три подхода — сравнение

|                     | Navigator 1.0 | Named Routes          | GoRouter      |
| ------------------- | ------------- | --------------------- | ------------- |
| Синтаксис           | Императивный  | Императивный + строки | Декларативный |
| Deep links          | Ограничено    | Ограничено            | ✓             |
| Web URLs            | Нет           | Частично              | ✓             |
| Вложенная навигация | Сложно        | Сложно                | ✓             |
| Редиректы           | Нет           | Нет                   | ✓             |
| Типизация           | Нет           | Нет                   | Частично      |
| Сложность           | Низкая        | Средняя               | Средняя       |

---

## 3. Navigator 1.0 — базовые операции

```dart
// Перейти на новый экран
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => DetailsScreen(id: item.id)),
);

// Вернуться назад
Navigator.pop(context);

// Вернуться с результатом
Navigator.pop(context, 'result');

// Ждать результата
final result = await Navigator.push<String>(
  context,
  MaterialPageRoute(builder: (_) => EditScreen()),
);
print('Результат: $result');

// Заменить текущий экран (без возможности вернуться)
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (_) => HomeScreen()),
);

// Очистить стек и перейти (например, после логина)
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (_) => HomeScreen()),
  (route) => false,  // удалить все маршруты
);
```

---

## 4. Передача данных между экранами

```dart
// Передача через конструктор (рекомендуется)
class ProductDetailsScreen extends StatelessWidget {
  final Product product;
  const ProductDetailsScreen({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(product.name)),
      body: Text(product.description),
    );
  }
}

// Вызов
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (_) => ProductDetailsScreen(product: selectedProduct),
  ),
);
```

---

## 5. Именованные маршруты

```dart
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (_) => const HomeScreen(),
    '/products': (_) => const ProductListScreen(),
    '/profile': (_) => const ProfileScreen(),
  },
  // Для передачи аргументов
  onGenerateRoute: (settings) {
    if (settings.name == '/product-detail') {
      final product = settings.arguments as Product;
      return MaterialPageRoute(
        builder: (_) => ProductDetailsScreen(product: product),
      );
    }
    return null;
  },
)

// Навигация по имени
Navigator.pushNamed(context, '/products');
Navigator.pushNamed(context, '/product-detail', arguments: someProduct);
```

---

## 6. GoRouter — современный подход

```yaml
# pubspec.yaml
dependencies:
  go_router: ^13.0.0
```

```dart
import 'package:go_router/go_router.dart';

final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(path: '/', builder: (_, __) => const HomeScreen()),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductDetailsScreen(productId: id);
      },
    ),
    GoRoute(path: '/profile', builder: (_, __) => const ProfileScreen()),
  ],
);

// В MaterialApp
MaterialApp.router(routerConfig: router)

// Навигация
context.go('/product/123');        // заменить стек
context.push('/product/123');      // добавить в стек
context.pop();                     // назад
context.goNamed('home');           // по имени
```

---

## 7. Типичные ошибки

| Ошибка                                        | Проблема               | Решение                                     |
| --------------------------------------------- | ---------------------- | ------------------------------------------- |
| `Navigator.pop()` без проверки                | Crash если стек пустой | `if (Navigator.canPop(context))`            |
| Глобальный `Navigator.of(context)` из сервиса | context может устареть | Использовать ключ навигатора `navigatorKey` |
| Передача данных через `arguments` только      | Не типизировано        | передача через конструктор или GoRouter     |

---

## 8. Практические рекомендации

1. **GoRouter для новых проектов** — поддержка deep links, web, редиректы.
2. **Navigator 1.0** достаточно для простых приложений без web/deep links.
3. **Передавай данные через конструктор** — типобезопасно и понятно.
4. **`pushReplacement`** после логина/онбординга — нельзя вернуться назад.
5. **`pushAndRemoveUntil`** для выхода из аккаунта — очищает весь стек.
