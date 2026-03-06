# 3.5: Cupertino-виджеты

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 3 — Основы виджетов

---

### 3.5: Cupertino-виджеты

🎯 **Цель шага:** Добавить в FitMenu адаптивные диалоги и контролы в iOS-стиле: `CupertinoActionSheet` для действий с рецептом, `CupertinoAlertDialog` для подтверждения удаления, `CupertinoPicker` для выбора порции.

📝 **Техническое задание:**

Создай виджет `RecipeActionsButton` с логикой «правильный диалог для правильной платформы»:

1. На iOS (`defaultTargetPlatform == TargetPlatform.iOS`) — `showCupertinoModalPopup` + `CupertinoActionSheet` с тремя вариантами: «В план», «Редактировать», «Удалить» (деструктивный — красный)
2. На Android — `showModalBottomSheet` с теми же `ListTile`
3. «Удалить» → `CupertinoAlertDialog` (iOS) / `AlertDialog` (Android) — «Вы уверены?»
4. «В план» → `CupertinoPicker` для выбора порции 0.5–5.0 с шагом 0.5
5. Используй `defaultTargetPlatform` из `flutter/foundation.dart`

✅ **Критерии приёмки:**
- [ ] На iOS показывается `CupertinoActionSheet`, на Android — `BottomSheet`
- [ ] Деструктивное действие «Удалить» красного цвета в `CupertinoActionSheet`
- [ ] `CupertinoAlertDialog` имеет кнопки «Отмена» и «Удалить»
- [ ] `CupertinoPicker` возвращает выбранное значение через колбэк
- [ ] Никакой дублированной логики — платформенная ветка только в одном месте

💡 **Подсказка:** `showCupertinoModalPopup` — для `ActionSheet` и `Picker`. `showCupertinoDialog` — для `AlertDialog`. Оба требуют `useRootNavigator: true` если используются во вложенном Navigator.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
import 'package:flutter/cupertino.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

class RecipeActionsButton extends StatelessWidget {
  final String recipeTitle;
  final VoidCallback onEdit;
  final VoidCallback onDelete;
  final void Function(double servings) onAddToPlan;

  const RecipeActionsButton({
    super.key,
    required this.recipeTitle,
    required this.onEdit,
    required this.onDelete,
    required this.onAddToPlan,
  });

  bool get _isIOS => defaultTargetPlatform == TargetPlatform.iOS;

  @override
  Widget build(BuildContext context) {
    return IconButton(
      icon: const Icon(Icons.more_vert),
      onPressed: () => _isIOS
          ? _showCupertinoSheet(context)
          : _showMaterialSheet(context),
    );
  }

  void _showCupertinoSheet(BuildContext context) {
    showCupertinoModalPopup<void>(
      context: context,
      builder: (_) => CupertinoActionSheet(
        title: Text(recipeTitle),
        actions: [
          CupertinoActionSheetAction(
            onPressed: () { Navigator.pop(context); _showPicker(context); },
            child: const Text('Добавить в план питания'),
          ),
          CupertinoActionSheetAction(
            onPressed: () { Navigator.pop(context); onEdit(); },
            child: const Text('Редактировать'),
          ),
          CupertinoActionSheetAction(
            isDestructiveAction: true, // красный цвет
            onPressed: () { Navigator.pop(context); _confirmDelete(context); },
            child: const Text('Удалить'),
          ),
        ],
        cancelButton: CupertinoActionSheetAction(
          onPressed: () => Navigator.pop(context),
          child: const Text('Отмена'),
        ),
      ),
    );
  }

  void _showMaterialSheet(BuildContext context) {
    showModalBottomSheet<void>(
      context: context,
      builder: (_) => SafeArea(child: Column(mainAxisSize: MainAxisSize.min,
        children: [
          ListTile(leading: const Icon(Icons.add_circle_outline),
              title: const Text('В план питания'),
              onTap: () { Navigator.pop(context); _showPicker(context); }),
          ListTile(leading: const Icon(Icons.edit_outlined),
              title: const Text('Редактировать'),
              onTap: () { Navigator.pop(context); onEdit(); }),
          ListTile(
              leading: const Icon(Icons.delete_outline, color: Colors.red),
              title: const Text('Удалить', style: TextStyle(color: Colors.red)),
              onTap: () { Navigator.pop(context); _confirmDelete(context); }),
        ],
      )),
    );
  }

  void _confirmDelete(BuildContext context) {
    if (_isIOS) {
      showCupertinoDialog<void>(
        context: context,
        builder: (_) => CupertinoAlertDialog(
          title: const Text('Удалить рецепт?'),
          content: Text('«$recipeTitle» будет удалён.'),
          actions: [
            CupertinoDialogAction(
              onPressed: () => Navigator.pop(context),
              child: const Text('Отмена'),
            ),
            CupertinoDialogAction(
              isDestructiveAction: true,
              onPressed: () { Navigator.pop(context); onDelete(); },
              child: const Text('Удалить'),
            ),
          ],
        ),
      );
    } else {
      showDialog<void>(
        context: context,
        builder: (_) => AlertDialog(
          title: const Text('Удалить рецепт?'),
          content: Text('«$recipeTitle» будет удалён.'),
          actions: [
            TextButton(onPressed: () => Navigator.pop(context),
                child: const Text('Отмена')),
            TextButton(
              style: TextButton.styleFrom(foregroundColor: Colors.red),
              onPressed: () { Navigator.pop(context); onDelete(); },
              child: const Text('Удалить'),
            ),
          ],
        ),
      );
    }
  }

  void _showPicker(BuildContext context) {
    double selected = 1.0;
    final options = [0.5, 1.0, 1.5, 2.0, 2.5, 3.0, 4.0, 5.0];

    showCupertinoModalPopup<void>(
      context: context,
      builder: (_) => Container(
        height: 300,
        color: CupertinoColors.systemBackground.resolveFrom(context),
        child: Column(children: [
          Row(mainAxisAlignment: MainAxisAlignment.spaceBetween, children: [
            CupertinoButton(
                child: const Text('Отмена'),
                onPressed: () => Navigator.pop(context)),
            CupertinoButton(
                child: const Text('Готово'),
                onPressed: () {
                  Navigator.pop(context);
                  onAddToPlan(selected);
                }),
          ]),
          Expanded(child: CupertinoPicker(
            itemExtent: 40,
            onSelectedItemChanged: (i) => selected = options[i],
            children: options.map((s) =>
                Center(child: Text('$s порц.'))).toList(),
          )),
        ]),
      ),
    );
  }
}
```

</details>
