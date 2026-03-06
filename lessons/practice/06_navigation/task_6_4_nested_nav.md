# 6.4: Вложенная навигация — табы внутри экрана

> Project: FitMenu | Глава 6 — Navigation

### 6.4: Nested Navigation — табы "Завтрак / Обед / Ужин" в плане питания

🎯 **Цель шага:** Реализовать вложенную навигацию внутри экрана "План питания": три вкладки (Завтрак, Обед, Ужин) с отдельным `Navigator` или `TabBarView`, каждая из которых содержит список приёмов пищи с возможностью перейти в детали.

---

📝 **Техническое задание:**

Реализуй экран `lib/features/meal_plan/screens/meal_plan_screen.dart`.

**Структура:**
```
MealPlanScreen (StatefulWidget + TickerProviderStateMixin)
  ├── AppBar + TabBar [Завтрак | Обед | Ужин]
  └── TabBarView
        ├── MealTab(type: MealType.breakfast)
        ├── MealTab(type: MealType.lunch)
        └── MealTab(type: MealType.dinner)
```

**MealType enum:**
```dart
enum MealType {
  breakfast('Завтрак', Icons.wb_sunny_outlined),
  lunch('Обед', Icons.wb_sunny),
  dinner('Ужин', Icons.nightlight_outlined);

  const MealType(this.label, this.icon);
  final String label;
  final IconData icon;
}
```

**MealTab виджет:**
- Список блюд (`ListView.builder`)
- Каждый элемент — `ListTile` с названием, калориями, иконкой удаления
- FAB: "+" для добавления рецепта (открывает `showModalBottomSheet` со списком рецептов)
- Пустое состояние: иконка + "Нажми + чтобы добавить блюдо"

**TabController:**
- `TabController(length: 3, vsync: this)`
- `TabBar` располагается в `AppBar.bottom`
- Вкладки с иконками и подписями

**ModalBottomSheet для добавления:**
- Высота: `DraggableScrollableSheet`
- Список рецептов для выбора
- При выборе закрывает sheet и добавляет блюдо в список

---

✅ **Критерии приёмки:**
- [ ] `TabBar` с тремя вкладками работает в `AppBar.bottom`
- [ ] `TabBarView` переключается плавно, состояние вкладок сохраняется (использован `AutomaticKeepAliveClientMixin`)
- [ ] FAB "+" открывает `showModalBottomSheet`
- [ ] `DraggableScrollableSheet` позволяет растягивать sheet
- [ ] Пустое состояние отображается при отсутствии блюд
- [ ] `MealType` реализован как enum с named-параметрами

---

💡 **Подсказка:** Чтобы сохранить состояние вкладки при переключении, добавь `AutomaticKeepAliveClientMixin` в State класс вкладки и верни `true` из `wantKeepAlive`. `DraggableScrollableSheet` принимает `initialChildSize`, `minChildSize`, `maxChildSize` — попробуй `0.5 / 0.3 / 0.9`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/meal_plan/screens/meal_plan_screen.dart

import 'package:flutter/material.dart';

enum MealType {
  breakfast('Завтрак', Icons.wb_sunny_outlined),
  lunch('Обед', Icons.wb_sunny),
  dinner('Ужин', Icons.nightlight_outlined);

  const MealType(this.label, this.icon);
  final String label;
  final IconData icon;
}

class MealPlanScreen extends StatefulWidget {
  const MealPlanScreen({super.key});

  @override
  State<MealPlanScreen> createState() => _MealPlanScreenState();
}

class _MealPlanScreenState extends State<MealPlanScreen>
    with TickerProviderStateMixin {
  late final TabController _tabController;

  @override
  void initState() {
    super.initState();
    _tabController = TabController(length: MealType.values.length, vsync: this);
  }

  @override
  void dispose() {
    _tabController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('План питания'),
        bottom: TabBar(
          controller: _tabController,
          tabs: MealType.values
              .map((type) => Tab(icon: Icon(type.icon), text: type.label))
              .toList(),
        ),
      ),
      body: TabBarView(
        controller: _tabController,
        children: MealType.values
            .map((type) => MealTab(mealType: type))
            .toList(),
      ),
    );
  }
}

// --- MealTab ---

class MealTab extends StatefulWidget {
  const MealTab({super.key, required this.mealType});
  final MealType mealType;

  @override
  State<MealTab> createState() => _MealTabState();
}

class _MealTabState extends State<MealTab>
    with AutomaticKeepAliveClientMixin {
  // Сохраняем состояние вкладки при переключении
  @override
  bool get wantKeepAlive => true;

  final List<String> _meals = [];

  void _addMeal(String name) {
    setState(() => _meals.add(name));
  }

  void _removeMeal(int index) {
    setState(() => _meals.removeAt(index));
  }

  void _showAddDialog() {
    showModalBottomSheet<void>(
      context: context,
      isScrollControlled: true,
      builder: (context) => DraggableScrollableSheet(
        initialChildSize: 0.5,
        minChildSize: 0.3,
        maxChildSize: 0.9,
        expand: false,
        builder: (context, scrollController) => Column(
          children: [
            Padding(
              padding: const EdgeInsets.all(16),
              child: Text(
                'Выберите рецепт',
                style: Theme.of(context).textTheme.titleMedium,
              ),
            ),
            Expanded(
              child: ListView.builder(
                controller: scrollController,
                itemCount: 10,
                itemBuilder: (context, index) => ListTile(
                  title: Text('Рецепт ${index + 1}'),
                  subtitle: Text('${200 + index * 30} ккал'),
                  onTap: () {
                    _addMeal('Рецепт ${index + 1}');
                    Navigator.pop(context);
                  },
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    super.build(context); // обязательно для AutomaticKeepAliveClientMixin

    if (_meals.isEmpty) {
      return Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Icon(widget.mealType.icon, size: 48, color: Colors.grey),
            const SizedBox(height: 8),
            Text('Нажми + чтобы добавить блюдо',
                style: Theme.of(context).textTheme.bodyMedium?.copyWith(
                      color: Colors.grey,
                    )),
          ],
        ),
      );
    }

    return Scaffold(
      body: ListView.builder(
        itemCount: _meals.length,
        itemBuilder: (context, index) => ListTile(
          leading: const Icon(Icons.restaurant),
          title: Text(_meals[index]),
          trailing: IconButton(
            icon: const Icon(Icons.delete_outline),
            onPressed: () => _removeMeal(index),
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _showAddDialog,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

</details>
