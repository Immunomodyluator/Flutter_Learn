# 7.6: GetX — быстрый прототип с GetxController

> Project: FitMenu | Глава 7 — State Management

### 7.6: CartController с GetX для корзины ингредиентов

🎯 **Цель шага:** Реализовать корзину покупок ингредиентов (для рецептов из плана питания) через `GetxController` и реактивные переменные `Rx` — познакомиться с GetX-подходом и понять его отличия от Provider/Riverpod.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  get: ^4.6.0
```

Реализуй `lib/features/shopping/controllers/shopping_cart_controller.dart`.

**ShoppingItem:**
```dart
class ShoppingItem {
  ShoppingItem({required this.name, required this.amount, required this.unit, this.checked = false});
  final String name;
  final String amount;
  final String unit;
  bool checked;
}
```

**ShoppingCartController:**
```dart
class ShoppingCartController extends GetxController {
  final RxList<ShoppingItem> items = <ShoppingItem>[].obs;

  int get totalItems => items.length;
  int get checkedCount => items.where((i) => i.checked).length;

  void addItem(ShoppingItem item) => items.add(item);

  void removeItem(int index) => items.removeAt(index);

  void toggleChecked(int index) {
    items[index].checked = !items[index].checked;
    items.refresh(); // перерисовать список
  }

  void addFromRecipe(List<ShoppingItem> ingredients) {
    items.addAll(ingredients);
  }

  void clearChecked() => items.removeWhere((i) => i.checked);
}
```

**ShoppingCartScreen:**
- Инициализация: `Get.put(ShoppingCartController())`
- Список через `Obx(() => ListView(...))`
- Каждый item — `CheckboxListTile`
- AppBar badge с количеством товаров: `Obx(() => Badge(...))`
- FAB: добавить тестовые ингредиенты

**Навигация через GetX:**
```dart
Get.to(() => const ShoppingCartScreen());
Get.back();
```

---

✅ **Критерии приёмки:**
- [ ] `GetxController` с `RxList` реализован
- [ ] `Obx()` используется для реактивного UI (не `GetBuilder`)
- [ ] `items.refresh()` вызывается при изменении внутреннего состояния объекта
- [ ] Чекбоксы мгновенно обновляются при нажатии
- [ ] `clearChecked()` удаляет только отмеченные товары
- [ ] Понимаешь разницу: `Obx` (реактивный) vs `GetBuilder` (не реактивный)

---

💡 **Подсказка:** `RxList`, `RxString`, `RxBool` — реактивные типы GetX. `.obs` конвертирует обычное значение в реактивное. Проблема `items[i].checked = true` без `refresh()` — `Obx` не узнает об изменении внутри объекта (меняется поле, а не ссылка). `GetMaterialApp` нужен вместо обычного `MaterialApp` для полной работы GetX.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/shopping/controllers/shopping_cart_controller.dart

import 'package:get/get.dart';

class ShoppingItem {
  ShoppingItem({
    required this.name,
    required this.amount,
    required this.unit,
    this.checked = false,
  });

  final String name;
  final String amount;
  final String unit;
  bool checked;
}

class ShoppingCartController extends GetxController {
  final RxList<ShoppingItem> items = <ShoppingItem>[].obs;

  int get totalItems => items.length;
  int get checkedCount => items.where((i) => i.checked).length;
  int get uncheckedCount => totalItems - checkedCount;

  void addItem(ShoppingItem item) => items.add(item);

  void removeItem(int index) => items.removeAt(index);

  void toggleChecked(int index) {
    items[index].checked = !items[index].checked;
    items.refresh(); // уведомляем Obx об изменении внутри объекта
  }

  void addFromRecipe(List<ShoppingItem> ingredients) {
    // Не дублируем уже существующие
    final existingNames = items.map((i) => i.name.toLowerCase()).toSet();
    final newItems =
        ingredients.where((i) => !existingNames.contains(i.name.toLowerCase()));
    items.addAll(newItems);
  }

  void clearChecked() => items.removeWhere((i) => i.checked);

  void clearAll() => items.clear();
}
```

```dart
// lib/features/shopping/screens/shopping_cart_screen.dart

import 'package:flutter/material.dart';
import 'package:get/get.dart';

class ShoppingCartScreen extends StatelessWidget {
  const ShoppingCartScreen({super.key});

  ShoppingCartController get _ctrl => Get.find();

  @override
  Widget build(BuildContext context) {
    // Инициализируем контроллер, если не создан
    Get.put(ShoppingCartController());

    return Scaffold(
      appBar: AppBar(
        title: const Text('Список покупок'),
        actions: [
          Obx(
            () => Badge(
              label: Text('${_ctrl.uncheckedCount}'),
              isLabelVisible: _ctrl.uncheckedCount > 0,
              child: const Icon(Icons.shopping_cart),
            ),
          ),
          const SizedBox(width: 8),
          IconButton(
            icon: const Icon(Icons.delete_sweep),
            onPressed: _ctrl.clearChecked,
            tooltip: 'Удалить отмеченные',
          ),
        ],
      ),
      body: Obx(
        () => _ctrl.items.isEmpty
            ? const Center(child: Text('Список покупок пуст 🛒'))
            : ListView.builder(
                itemCount: _ctrl.items.length,
                itemBuilder: (context, index) {
                  final item = _ctrl.items[index];
                  return CheckboxListTile(
                    value: item.checked,
                    onChanged: (_) => _ctrl.toggleChecked(index),
                    title: Text(
                      item.name,
                      style: item.checked
                          ? const TextStyle(
                              decoration: TextDecoration.lineThrough,
                              color: Colors.grey,
                            )
                          : null,
                    ),
                    subtitle: Text('${item.amount} ${item.unit}'),
                    secondary: IconButton(
                      icon: const Icon(Icons.close, size: 18),
                      onPressed: () => _ctrl.removeItem(index),
                    ),
                  );
                },
              ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _ctrl.addFromRecipe([
          ShoppingItem(name: 'Куриное филе', amount: '200', unit: 'г'),
          ShoppingItem(name: 'Брокколи', amount: '150', unit: 'г'),
          ShoppingItem(name: 'Оливковое масло', amount: '1', unit: 'ст.л.'),
        ]),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

</details>
