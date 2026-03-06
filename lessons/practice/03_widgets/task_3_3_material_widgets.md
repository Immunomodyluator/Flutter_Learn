# 3.3: Базовые Material-виджеты

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 3 — Основы виджетов

---

### 3.3: Базовые Material-виджеты

🎯 **Цель шага:** Собрать главный экран FitMenu используя весь арсенал Material-виджетов: `AppBar` с аватаром и поиском, список рецептов, кнопки, `Image.network` с fallback и `SnackBar`.

📝 **Техническое задание:**

Создай `RecipeListScreen`:

1. `AppBar` — заголовок «FitMenu», `actions`: `IconButton` поиска + `CircleAvatar` пользователя
2. Список карточек рецептов через `ListView.builder`
3. На каждой карточке: `ElevatedButton` «В план», `TextButton` «Подробнее», `IconButton` избранное
4. `Image.network` для фото с `loadingBuilder` (показывает `CircularProgressIndicator`) и `errorBuilder` (показывает placeholder-иконку)
5. `SnackBar` через `ScaffoldMessenger.of(context)` при нажатии «В план»
6. `FloatingActionButton.extended` — «Новый рецепт»

✅ **Критерии приёмки:**
- [ ] `ScaffoldMessenger.of(context).showSnackBar` (не устаревший `Scaffold.of`)
- [ ] `Image.network` показывает индикатор загрузки и иконку при ошибке
- [ ] Нет устаревших виджетов: не `RaisedButton`, не `FlatButton`
- [ ] Все `onPressed` обработчики не `null`
- [ ] На узком экране (320dp) нет Overflow ошибок

💡 **Подсказка:** `SnackBar` с `behavior: SnackBarBehavior.floating` выглядит лучше и не перекрывает FAB. Оберни `Image.network` в `AspectRatio` с коэффициентом `16/9` — это предотвращает прыжки layout при загрузке.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
import 'package:flutter/material.dart';

class RecipeListScreen extends StatelessWidget {
  const RecipeListScreen({super.key});

  static const _recipes = [
    (id: '1', title: 'Овсянка с ягодами',    cal: 320.0, cat: 'завтрак',
     img: 'https://picsum.photos/seed/oat/400/200'),
    (id: '2', title: 'Куриная грудка с рисом', cal: 480.0, cat: 'обед',
     img: 'https://picsum.photos/seed/chick/400/200'),
    (id: '3', title: 'Греческий салат',        cal: 200.0, cat: 'обед',
     img: 'https://picsum.photos/seed/salad/400/200'),
  ];

  void _addToPlan(BuildContext context, String title) {
    ScaffoldMessenger.of(context)
      ..hideCurrentSnackBar()
      ..showSnackBar(SnackBar(
        content: Text('«$title» добавлен в план'),
        behavior: SnackBarBehavior.floating,
        action: SnackBarAction(label: 'Отмена', onPressed: () {}),
        duration: const Duration(seconds: 2),
      ));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('FitMenu'),
        actions: [
          IconButton(
            icon: const Icon(Icons.search),
            tooltip: 'Поиск',
            onPressed: () {},
          ),
          Padding(
            padding: const EdgeInsets.only(right: 8),
            child: CircleAvatar(
              radius: 16,
              backgroundImage: const NetworkImage('https://i.pravatar.cc/40'),
              onBackgroundImageError: (_, __) {},
            ),
          ),
        ],
      ),
      body: ListView.builder(
        padding: const EdgeInsets.all(16),
        itemCount: _recipes.length,
        itemBuilder: (ctx, i) {
          final r = _recipes[i];
          return _RecipeTile(
            title: r.title,
            calories: r.cal,
            category: r.cat,
            imageUrl: r.img,
            onAddToPlan: () => _addToPlan(ctx, r.title),
          );
        },
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: () {},
        icon: const Icon(Icons.add),
        label: const Text('Новый рецепт'),
      ),
    );
  }
}

class _RecipeTile extends StatefulWidget {
  final String title, category, imageUrl;
  final double calories;
  final VoidCallback onAddToPlan;
  const _RecipeTile({required this.title, required this.calories,
      required this.category, required this.imageUrl,
      required this.onAddToPlan});
  @override State<_RecipeTile> createState() => _RecipeTileState();
}

class _RecipeTileState extends State<_RecipeTile> {
  bool _fav = false;

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      margin: const EdgeInsets.only(bottom: 12),
      child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
        // Изображение с loadingBuilder + errorBuilder
        AspectRatio(
          aspectRatio: 16 / 9,
          child: Image.network(
            widget.imageUrl, fit: BoxFit.cover,
            loadingBuilder: (_, child, progress) => progress == null ? child
                : const Center(child: CircularProgressIndicator()),
            errorBuilder: (_, __, ___) => ColoredBox(
              color: Colors.green.shade100,
              child: const Center(child: Icon(Icons.broken_image, size: 48)),
            ),
          ),
        ),
        Padding(
          padding: const EdgeInsets.all(12),
          child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
            Text(widget.title,
                style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
            Text('${widget.calories.toStringAsFixed(0)} ккал · ${widget.category}',
                style: const TextStyle(color: Colors.grey)),
            const SizedBox(height: 8),
            Row(children: [
              ElevatedButton.icon(
                onPressed: widget.onAddToPlan,
                icon: const Icon(Icons.add, size: 16),
                label: const Text('В план'),
              ),
              const SizedBox(width: 8),
              TextButton(onPressed: () {}, child: const Text('Подробнее')),
              const Spacer(),
              IconButton(
                icon: Icon(_fav ? Icons.favorite : Icons.favorite_border,
                    color: _fav ? Colors.red : null),
                onPressed: () => setState(() => _fav = !_fav),
              ),
            ]),
          ]),
        ),
      ]),
    );
  }
}
```

</details>
