# 3.2: StatefulWidget и State

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 3 — Основы виджетов

---

### 3.2: StatefulWidget и State

🎯 **Цель шага:** Реализовать `MealLogScreen` — экран дневника питания с живым пересчётом КБЖУ при каждом добавлении/удалении блюда, правильным жизненным циклом State и защитой от async-краша.

📝 **Техническое задание:**

Создай `lib/features/meal_log/screens/meal_log_screen.dart`:

1. State хранит `List<Recipe> _meals` (добавленные блюда)
2. Вычисляемые геттеры `_totalCalories` и `_totalProtein` — не дублировать в State, считать на лету
3. FAB открывает `showModalBottomSheet` со списком рецептов для добавления
4. Кнопка удаления у каждого блюда через `setState(() => _meals.removeAt(index))`
5. `initState` симулирует загрузку данных через `Future.delayed` — обязательно проверяй `mounted` перед `setState`
6. `debugPrint` в `initState`, `didUpdateWidget`, `dispose`

✅ **Критерии приёмки:**
- [ ] `initState` вызывается один раз при создании (проверить через `debugPrint`)
- [ ] `dispose` вызывается при уходе с экрана
- [ ] `if (!mounted) return` стоит перед `setState` в async методе
- [ ] Панель КБЖУ обновляется мгновенно после каждого добавления/удаления
- [ ] Нет `setState(() {})` с пустым телом

💡 **Подсказка:** Вычисляемые значения (сумма калорий) лучше не хранить в State, а считать через геттер — иначе нужно синхронизировать два состояния и легко ошибиться. `_totalCalories` = `_meals.fold(0.0, (sum, r) => sum + r.calories)`.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
import 'package:flutter/material.dart';
import '../../../core/domain/models/recipe.dart';

class MealLogScreen extends StatefulWidget {
  const MealLogScreen({super.key});

  @override
  State<MealLogScreen> createState() => _MealLogScreenState();
}

class _MealLogScreenState extends State<MealLogScreen> {
  final List<Recipe> _meals = [];

  // Вычисляемые геттеры — не дублируем значения в State
  double get _totalCalories => _meals.fold(0, (s, r) => s + r.calories);
  double get _totalProtein  => _meals.fold(0, (s, r) => s + r.protein);

  @override
  void initState() {
    super.initState();
    debugPrint('🟢 MealLogScreen initState');
    _loadInitial();
  }

  @override
  void didUpdateWidget(MealLogScreen old) {
    super.didUpdateWidget(old);
    debugPrint('🔄 didUpdateWidget');
  }

  @override
  void dispose() {
    debugPrint('🔴 MealLogScreen dispose');
    super.dispose();
  }

  Future<void> _loadInitial() async {
    await Future.delayed(const Duration(milliseconds: 300));
    if (!mounted) return; // виджет уже удалён!
    setState(() {
      _meals.add(Recipe(id: '0', title: 'Завтрак (пример)',
          category: 'завтрак', calories: 320, protein: 12, fat: 6, carbs: 54));
    });
  }

  void _remove(int index) => setState(() => _meals.removeAt(index));

  void _add(Recipe recipe) {
    setState(() => _meals.add(recipe));
    Navigator.pop(context);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Дневник питания')),
      body: Column(children: [
        // Панель КБЖУ
        ColoredBox(
          color: Colors.green.shade50,
          child: Padding(
            padding: const EdgeInsets.symmetric(vertical: 12, horizontal: 24),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              children: [
                _stat('Калории', '${_totalCalories.toStringAsFixed(0)} ккал'),
                _stat('Белки', '${_totalProtein.toStringAsFixed(1)} г'),
                _stat('Блюд', '${_meals.length}'),
              ],
            ),
          ),
        ),
        // Список
        Expanded(
          child: _meals.isEmpty
              ? const Center(child: Text('Добавьте первое блюдо 🥗'))
              : ListView.builder(
                  itemCount: _meals.length,
                  itemBuilder: (_, i) => ListTile(
                    leading: const Icon(Icons.restaurant),
                    title: Text(_meals[i].title),
                    subtitle: Text('${_meals[i].calories.toStringAsFixed(0)} ккал'),
                    trailing: IconButton(
                      icon: const Icon(Icons.delete_outline, color: Colors.red),
                      onPressed: () => _remove(i),
                    ),
                  ),
                ),
        ),
      ]),
      floatingActionButton: FloatingActionButton(
        onPressed: () => showModalBottomSheet(
          context: context,
          builder: (_) => _AddSheet(onAdd: _add),
        ),
        child: const Icon(Icons.add),
      ),
    );
  }

  Widget _stat(String label, String value) => Column(children: [
    Text(value, style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
    Text(label, style: const TextStyle(color: Colors.grey, fontSize: 12)),
  ]);
}

class _AddSheet extends StatelessWidget {
  final void Function(Recipe) onAdd;
  const _AddSheet({required this.onAdd});

  static final _options = [
    Recipe(id:'a', title:'Куриная грудка', category:'обед',
        calories:480, protein:52, fat:8, carbs:44),
    Recipe(id:'b', title:'Греческий салат', category:'обед',
        calories:200, protein:5, fat:14, carbs:12),
  ];

  @override
  Widget build(BuildContext context) => ListView(
    shrinkWrap: true,
    children: _options.map((r) => ListTile(
      title: Text(r.title),
      subtitle: Text('${r.calories.toStringAsFixed(0)} ккал'),
      onTap: () => onAdd(r),
    )).toList(),
  );
}
```

</details>
