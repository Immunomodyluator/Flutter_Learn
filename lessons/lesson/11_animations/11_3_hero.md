# 11.3 Hero-анимации

## 1. Суть

Hero — анимация "общего элемента" при переходе между экранами. Виджет как будто "летит" с одного экрана на другой. Реализуется простым оборачиванием в `Hero` с одинаковым `tag` на обоих экранах.

---

## 2. Базовый синтаксис

```dart
// Экран A — список
class ProductListScreen extends StatelessWidget {
  const ProductListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: products.length,
      itemBuilder: (context, index) {
        final product = products[index];
        return GestureDetector(
          onTap: () => Navigator.push(
            context,
            MaterialPageRoute(
              builder: (_) => ProductDetailScreen(product: product),
            ),
          ),
          child: Hero(
            tag: 'product-image-${product.id}', // уникальный тег
            child: Image.network(
              product.imageUrl,
              width: 80,
              height: 80,
              fit: BoxFit.cover,
            ),
          ),
        );
      },
    );
  }
}

// Экран B — детальный
class ProductDetailScreen extends StatelessWidget {
  final Product product;

  const ProductDetailScreen({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          Hero(
            tag: 'product-image-${product.id}', // тот же тег!
            child: Image.network(
              product.imageUrl,
              width: double.infinity,
              height: 300,
              fit: BoxFit.cover,
            ),
          ),
          Padding(
            padding: const EdgeInsets.all(16),
            child: Text(product.title),
          ),
        ],
      ),
    );
  }
}
```

---

## 3. Hero с GoRouter

```dart
// GoRouter поддерживает Hero без дополнительных настроек
GoRouter(
  routes: [
    GoRoute(
      path: '/products',
      builder: (context, state) => const ProductListScreen(),
    ),
    GoRoute(
      path: '/products/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductDetailScreen(productId: id);
      },
    ),
  ],
)
```

---

## 4. Кастомный flightShuttleBuilder

```dart
// Настройка виджета во время полёта
Hero(
  tag: 'avatar-${user.id}',
  flightShuttleBuilder: (
    BuildContext flightContext,
    Animation<double> animation,
    HeroFlightDirection flightDirection,
    BuildContext fromHeroContext,
    BuildContext toHeroContext,
  ) {
    // Возвращаем виджет, который будет показан во время перехода
    return ScaleTransition(
      scale: animation,
      child: Material(
        type: MaterialType.transparency,
        child: ClipOval(
          child: Image.network(user.avatarUrl, fit: BoxFit.cover),
        ),
      ),
    );
  },
  child: CircleAvatar(
    backgroundImage: NetworkImage(user.avatarUrl),
    radius: 30,
  ),
)
```

---

## 5. Реальный пример — карточка товара → детали

```dart
class ProductCard extends StatelessWidget {
  final Product product;

  const ProductCard({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => context.push('/products/${product.id}'),
      child: Card(
        clipBehavior: Clip.antiAlias,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Hero(
              tag: 'product-img-${product.id}',
              child: Image.network(
                product.imageUrl,
                height: 150,
                width: double.infinity,
                fit: BoxFit.cover,
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Анимируем и заголовок!
                  Hero(
                    tag: 'product-title-${product.id}',
                    child: Material(
                      type: MaterialType.transparency,
                      child: Text(
                        product.title,
                        style: const TextStyle(fontWeight: FontWeight.bold),
                      ),
                    ),
                  ),
                  Text('${product.price} ₽'),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class ProductDetailScreen extends StatelessWidget {
  final String productId;

  const ProductDetailScreen({super.key, required this.productId});

  @override
  Widget build(BuildContext context) {
    // Получаем продукт (упрощённо)
    final product = getProductById(productId);

    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 300,
            flexibleSpace: FlexibleSpaceBar(
              background: Hero(
                tag: 'product-img-${product.id}',
                child: Image.network(product.imageUrl, fit: BoxFit.cover),
              ),
            ),
          ),
          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Hero(
                    tag: 'product-title-${product.id}',
                    child: Material(
                      type: MaterialType.transparency,
                      child: Text(
                        product.title,
                        style: Theme.of(context).textTheme.headlineMedium,
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 6. Под капотом

- Flutter создаёт "overlay" — временный слой поверх обоих экранов.
- Hero-виджет "вырезается" из исходного экрана, анимируется в overlay, "вставляется" в целевой экран.
- `tag` должен быть уникальным на всём дереве навигации — иначе `AssertionError`.
- По умолчанию Flutter интерполирует размер и позицию с `Curves.fastOutSlowIn`.

---

## 7. Типичные ошибки

| Ошибка                      | Причина                                | Решение                                                        |
| --------------------------- | -------------------------------------- | -------------------------------------------------------------- |
| Hero не анимирует           | Разные `tag` на двух экранах           | Используй строго одинаковый `tag`                              |
| Текст в Hero обрезается     | Нет `Material` обёртки                 | Оборачивай текст в `Material(type: MaterialType.transparency)` |
| `There are multiple heroes` | Один `tag` у нескольких Hero на экране | Убедись что `tag` уникален (используй id элемента)             |
| Hero исчезает при анимации  | Клип родителя обрезает                 | Добавь `Clip.none` или `overflow: Overflow.visible` родителю   |

---

## 8. Рекомендации

1. **Уникальный `tag`** — используй `'hero-${item.id}'`, не строку без id.
2. **Текст в Hero** — оборачивай в `Material(type: MaterialType.transparency)`.
3. **`flightShuttleBuilder`** — для кастомного виджета во время полёта (скругление → прямоугольник).
4. **Hero работает с любым виджетом** — изображения, карточки, кнопки.
5. **Не злоупотребляй** — Hero на несколько элементов одновременно может выглядеть перегружено.
