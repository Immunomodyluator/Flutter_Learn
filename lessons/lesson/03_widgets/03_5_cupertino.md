# 3.5 Cupertino-виджеты

## 1. Суть концепции

Cupertino-виджеты — Flutter-реализация интерфейса iOS (Apple Human Interface Guidelines). Выглядят нативно на iOS, но работают на всех платформах.

**Когда использовать:**

- Приложение только для iOS
- Нужен нативный iOS-вид
- Адаптивный UI (разный вид на iOS и Android)

**Требование**: `CupertinoApp` или `CupertinoTheme` в корне.

---

## 2. CupertinoApp vs MaterialApp

```dart
// iOS-стиль для всего приложения
void main() => runApp(const CupertinoApp(
  home: CupertinoPageScaffold(
    navigationBar: CupertinoNavigationBar(middle: Text('Главная')),
    child: Center(child: Text('iOS')),
  ),
));

// Гибрид: MaterialApp + Cupertino-виджеты внутри
void main() => runApp(MaterialApp(
  home: Scaffold(
    body: CupertinoButton(
      onPressed: () {},
      child: const Text('iOS кнопка в Material-приложении'),
    ),
  ),
));
```

---

## 3. Основные Cupertino-виджеты

```dart
// CupertinoButton
CupertinoButton(
  onPressed: () {},
  child: const Text('Нажми'),
)
CupertinoButton.filled(
  onPressed: () {},
  child: const Text('Заполненная'),
)

// CupertinoTextField
CupertinoTextField(
  placeholder: 'Поиск',
  prefix: const Padding(
    padding: EdgeInsets.only(left: 8),
    child: Icon(CupertinoIcons.search, color: CupertinoColors.systemGrey),
  ),
  clearButtonMode: OverlayVisibilityMode.editing,
)

// CupertinoSwitch
bool _value = true;
CupertinoSwitch(
  value: _value,
  onChanged: (v) => setState(() => _value = v),
  activeColor: CupertinoColors.activeBlue,
)

// CupertinoSlider
double _speed = 0.5;
CupertinoSlider(
  value: _speed,
  onChanged: (v) => setState(() => _speed = v),
)

// CupertinoPicker (барабан выбора)
FixedExtentScrollController _scrollController = FixedExtentScrollController();
CupertinoPicker(
  scrollController: _scrollController,
  itemExtent: 40,
  onSelectedItemChanged: (index) => setState(() => _selectedIndex = index),
  children: ['Москва', 'Санкт-Петербург', 'Казань']
      .map((city) => Center(child: Text(city)))
      .toList(),
)
```

---

## 4. Навигация и диалоги

```dart
// CupertinoPageRoute — iOS-переход (слайд справа)
Navigator.push(context, CupertinoPageRoute(builder: (_) => DetailsPage()));

// CupertinoAlertDialog
showCupertinoDialog<void>(
  context: context,
  builder: (context) => CupertinoAlertDialog(
    title: const Text('Удалить?'),
    content: const Text('Это нельзя отменить'),
    actions: [
      CupertinoDialogAction(
        isDestructiveAction: true,
        onPressed: () => Navigator.pop(context),
        child: const Text('Удалить'),
      ),
      CupertinoDialogAction(
        onPressed: () => Navigator.pop(context),
        child: const Text('Отмена'),
      ),
    ],
  ),
);

// CupertinoActionSheet — список действий снизу
showCupertinoModalPopup<void>(
  context: context,
  builder: (context) => CupertinoActionSheet(
    title: const Text('Действия'),
    actions: [
      CupertinoActionSheetAction(
        isDestructiveAction: true,
        onPressed: () => Navigator.pop(context),
        child: const Text('Удалить'),
      ),
      CupertinoActionSheetAction(
        onPressed: () => Navigator.pop(context),
        child: const Text('Поделиться'),
      ),
    ],
    cancelButton: CupertinoActionSheetAction(
      onPressed: () => Navigator.pop(context),
      child: const Text('Отмена'),
    ),
  ),
);
```

---

## 5. Адаптивный UI (iOS + Android)

```dart
// Определяем платформу и показываем нужный виджет
import 'dart:io' show Platform;

Widget buildSwitch(bool value, ValueChanged<bool> onChanged) {
  if (Platform.isIOS) {
    return CupertinoSwitch(value: value, onChanged: onChanged);
  }
  return Switch(value: value, onChanged: onChanged);
}

// Или через фабричный паттерн для целых экранов
class AdaptiveAlertDialog extends StatelessWidget {
  final String title;
  final String message;
  const AdaptiveAlertDialog({super.key, required this.title, required this.message});

  @override
  Widget build(BuildContext context) {
    if (Theme.of(context).platform == TargetPlatform.iOS) {
      return CupertinoAlertDialog(
        title: Text(title),
        content: Text(message),
      );
    }
    return AlertDialog(
      title: Text(title),
      content: Text(message),
    );
  }
}
```

---

## 6. CupertinoIcons

```dart
// Иконки в iOS-стиле (пакет cupertino_icons уже в pubspec.yaml)
const Icon(CupertinoIcons.home)
const Icon(CupertinoIcons.search)
const Icon(CupertinoIcons.person)
const Icon(CupertinoIcons.settings)
const Icon(CupertinoIcons.back)
const Icon(CupertinoIcons.bell)
const Icon(CupertinoIcons.heart)
```

---

## 7. Типичные ошибки

| Ошибка                                    | Проблема                                 | Решение                                   |
| ----------------------------------------- | ---------------------------------------- | ----------------------------------------- |
| `CupertinoApp` + `Theme.of(context)`      | Material тема не работает в CupertinoApp | Использовать `CupertinoTheme.of(context)` |
| `showDialog` вместо `showCupertinoDialog` | Выглядит как Material в iOS-приложении   | Используй правильный диалог для платформы |
| `Platform.isIOS` в web                    | Throws `UnsupportedError`                | Проверять `kIsWeb` перед `Platform.isIOS` |

---

## 8. Практические рекомендации

1. **Не смешивай стили без причины** — либо весь экран Material, либо весь Cupertino.
2. **Адаптивный подход** — создавай обёртки `AdaptiveButton`, `AdaptiveSwitch` для повторного использования.
3. **`CupertinoIcons`** — уже включён в Flutter SDK через `cupertino_icons`.
4. **`CupertinoPageRoute`** для iOS-навигации — даёт правильный свайп назад.
5. **Тестируй на реальном устройстве** — Cupertino-виджеты выглядят иначе на разных версиях iOS.
