# 1.2: Коллекции

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 1 — Введение в Dart

---

### 1.2: Коллекции

🎯 **Цель шага:** Построить «витрину рецептов» FitMenu — сгруппировать рецепты по категориям через `Map`, хранить избранное через `Set`, применить spread-оператор и `collection if/for` для гибкого формирования списков.

📝 **Техническое задание:**

В `bin/collections_demo.dart` реализуй:

1. `List<Recipe>` из ≥ 5 рецептов с разными категориями (`'завтрак'`, `'обед'`, `'ужин'`)
2. `Set<String> favoriteIds` — ID избранного (добавь один ID дважды, убедись что дубликата нет)
3. `Map<String, List<Recipe>> byCategory` — группировка через `fold` или `forEach`
4. `List<Recipe> premiumCatalog` с `collection if` — добавляет VIP-рецепт только для Premium
5. `List<String> titles` через `collection for` — из каждого рецепта извлекаем строку с форматом `«Название: N ккал»`
6. Объедини завтраки и ужины через spread `...` в один `List<Recipe> lightMeals`

Простая модель `Recipe` прямо в файле: `id`, `title`, `category`, `calories`.

✅ **Критерии приёмки:**
- [ ] `Set` с дублирующимся ID содержит элемент только один раз
- [ ] `byCategory` правильно группирует по ключу категории
- [ ] `collection if` добавляет элемент только при `isPremium == true`
- [ ] `collection for` возвращает список строк того же размера что и исходный список
- [ ] Spread `...` объединяет два списка без `addAll`

💡 **Подсказка:** Для группировки: `recipes.fold<Map<String, List<Recipe>>>({}, (map, r) { map.putIfAbsent(r.category, () => []).add(r); return map; })`. Spread работает только с `Iterable`, не с `Map`.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
void main() {
  // 1. Полный каталог рецептов
  final List<Recipe> allRecipes = [
    Recipe(id: '1', title: 'Овсянка с ягодами', category: 'завтрак', calories: 320),
    Recipe(id: '2', title: 'Омлет с овощами',   category: 'завтрак', calories: 280),
    Recipe(id: '3', title: 'Куриная грудка',    category: 'обед',    calories: 480),
    Recipe(id: '4', title: 'Греческий салат',   category: 'обед',    calories: 200),
    Recipe(id: '5', title: 'Лосось на пару',    category: 'ужин',    calories: 350),
    Recipe(id: '6', title: 'Творог с фруктами', category: 'ужин',    calories: 180),
  ];

  // 2. Set избранных — дубликат '3' игнорируется
  final Set<String> favoriteIds = {'1', '3', '3', '5'};
  print('Избранных: ${favoriteIds.length}'); // 3, не 4

  // 3. Группировка по категории
  final Map<String, List<Recipe>> byCategory =
      allRecipes.fold({}, (map, r) {
        map.putIfAbsent(r.category, () => []).add(r);
        return map;
      });
  byCategory.forEach((cat, recipes) {
    print('$cat: ${recipes.map((r) => r.title).join(', ')}');
  });

  // 4. collection if — VIP рецепт только для Premium
  const bool isPremium = true;
  final List<Recipe> catalog = [
    ...allRecipes,
    if (isPremium)
      Recipe(id: '99', title: 'Авокадо-боул VIP', category: 'обед', calories: 420),
  ];
  print('\nКаталог (${catalog.length} рецептов):');

  // 5. collection for — заголовки с калориями
  final List<String> titles = [
    for (final r in catalog) '«${r.title}: ${r.calories} ккал»',
  ];
  titles.forEach(print);

  // 6. Spread — объединить завтраки и ужины
  final List<Recipe> lightMeals = [
    ...(byCategory['завтрак'] ?? []),
    ...(byCategory['ужин']   ?? []),
  ];
  print('\nЛёгкие блюда: ${lightMeals.map((r) => r.title).join(' | ')}');

  // Бонус: фильтрация избранных через Set
  final favorites = allRecipes.where((r) => favoriteIds.contains(r.id)).toList();
  print('Избранное: ${favorites.map((r) => r.title).join(', ')}');
}

class Recipe {
  final String id;
  final String title;
  final String category;
  final double calories;
  const Recipe({required this.id, required this.title,
      required this.category, required this.calories});
}
```

</details>
