# 3.1 StatelessWidget

## 1. Суть концепции

`StatelessWidget` — виджет без внутреннего изменяемого состояния. Описывает UI на основе входных параметров (конструктора). Пересобирается только когда изменяется родитель и передаёт новые параметры.

Правило: если виджет не вызывает `setState()` — делай его `Stateless`.

---

## 2. Синтаксис

```dart
class UserCard extends StatelessWidget {
  final String name;
  final String avatarUrl;
  final VoidCallback onTap;

  const UserCard({
    super.key,
    required this.name,
    required this.avatarUrl,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: CircleAvatar(
          backgroundImage: NetworkImage(avatarUrl),
        ),
        title: Text(name),
        onTap: onTap,
      ),
    );
  }
}
```

---

## 3. Обязательное правило: `const` конструктор

```dart
// ✅ Правильно — const конструктор
class MyWidget extends StatelessWidget {
  final String title;
  const MyWidget({super.key, required this.title});

  @override
  Widget build(BuildContext context) => Text(title);
}

// Позволяет использовать:
const MyWidget(title: 'Hello') // не будет пересоздан при rebuild родителя
```

Без `const` конструктора Flutter не может оптимизировать повторное использование виджета.

---

## 4. Реальный пример: карточка товара

```dart
class ProductCard extends StatelessWidget {
  final Product product;
  final VoidCallback onAddToCart;

  const ProductCard({
    super.key,
    required this.product,
    required this.onAddToCart,
  });

  @override
  Widget build(BuildContext context) {
    final theme = Theme.of(context);

    return Card(
      clipBehavior: Clip.antiAlias,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          AspectRatio(
            aspectRatio: 16 / 9,
            child: Image.network(
              product.imageUrl,
              fit: BoxFit.cover,
              errorBuilder: (_, __, ___) => const Icon(Icons.broken_image),
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(12),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(product.name, style: theme.textTheme.titleMedium),
                const SizedBox(height: 4),
                Text(
                  '${product.price} ₽',
                  style: theme.textTheme.bodyLarge?.copyWith(
                    color: theme.colorScheme.primary,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                SizedBox(
                  width: double.infinity,
                  child: FilledButton(
                    onPressed: onAddToCart,
                    child: const Text('В корзину'),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 5. Что происходит под капотом

```
Родитель rebuild
      ↓
Flutter сравнивает тип виджета и ключ
      ↓
Тот же тип + тот же ключ → обновляет props в существующем Element
      ↓
Вызывает build() нового StatelessWidget
      ↓
Результат сравнивается с предыдущим render tree
```

При использовании `const` Flutter пропускает этот процесс полностью — если параметры не изменились.

---

## 6. Вынесение методов vs отдельные виджеты

```dart
// ❌ Антипаттерн — вынесенный метод, а не виджет
class MyScreen extends StatelessWidget {
  Widget _buildHeader() => Text('Header'); // вызывается каждый rebuild

  @override
  Widget build(BuildContext context) {
    return Column(children: [_buildHeader(), ...]);
  }
}

// ✅ Правильно — отдельный const-виджет
class _Header extends StatelessWidget {
  const _Header();

  @override
  Widget build(BuildContext context) => const Text('Header');
}
```

Приватный виджет (`_Header`) с `const` пропускается при rebuild родителя.

---

## 7. Типичные ошибки

| Ошибка                                | Проблема                                 | Решение                                               |
| ------------------------------------- | ---------------------------------------- | ----------------------------------------------------- |
| Нет `const` конструктора              | Нельзя использовать `const` при создании | Добавить `const` в конструктор                        |
| Логика в `build()`                    | Тяжёлые вычисления каждый rebuild        | Вынести в getter или precompute                       |
| Передача callback через много уровней | "Prop drilling"                          | Использовать InheritedWidget/Provider                 |
| `StatelessWidget` с изменяемым полем  | Противоречие архитектуре                 | Сделать `StatefulWidget` или перенести состояние выше |

---

## 8. Практические рекомендации

1. **Все поля `final`** — `StatelessWidget` должен быть полностью неизменяемым.
2. **`const` конструктор всегда** — это не опциональная практика, это обязательное правило.
3. **Вычисления вне `build()`** — если нужно преобразовать данные, делай это до передачи в виджет.
4. **Небольшие виджеты** — лучше 10 маленьких виджетов, чем один 100-строчный.
5. **`super.key`** — всегда передавай ключ через `super.key`, не игнорируй его.
