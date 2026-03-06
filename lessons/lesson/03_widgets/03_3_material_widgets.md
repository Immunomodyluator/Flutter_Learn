# 3.3 Базовые Material-виджеты

## 1. Суть концепции

Flutter реализует [Material Design 3](https://m3.material.io/) — систему дизайна Google. Базовые Material-виджеты — готовые компоненты UI, которые автоматически адаптируются к теме приложения.

Для использования нужен `MaterialApp` в корне дерева виджетов.

---

## 2. Scaffold — базовая структура экрана

```dart
Scaffold(
  appBar: AppBar(
    leading: IconButton(icon: const Icon(Icons.arrow_back), onPressed: () => Navigator.pop(context)),
    title: const Text('Заголовок'),
    actions: [
      IconButton(icon: const Icon(Icons.search), onPressed: () {}),
      PopupMenuButton<String>(
        onSelected: (value) {},
        itemBuilder: (_) => [
          const PopupMenuItem(value: 'settings', child: Text('Настройки')),
        ],
      ),
    ],
  ),
  body: const Center(child: Text('Контент')),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: const Icon(Icons.add),
  ),
  bottomNavigationBar: BottomNavigationBar(
    items: const [
      BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Главная'),
      BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Профиль'),
    ],
    currentIndex: 0,
    onTap: (index) {},
  ),
  drawer: Drawer(child: ListView(children: [])),
)
```

---

## 3. Кнопки

```dart
// FilledButton — основное действие (Material 3)
FilledButton(onPressed: () {}, child: const Text('Сохранить'))
FilledButton.tonal(onPressed: () {}, child: const Text('Вторичное'))

// OutlinedButton — второстепенное действие
OutlinedButton(onPressed: () {}, child: const Text('Отмена'))

// TextButton — третичное / в диалогах
TextButton(onPressed: () {}, child: const Text('Подробнее'))

// ElevatedButton — устарел в M3, но ещё используется
ElevatedButton(onPressed: null, child: const Text('Неактивна')) // null = disabled

// IconButton
IconButton(
  icon: const Icon(Icons.favorite),
  color: Colors.red,
  onPressed: () {},
)

// FloatingActionButton
FloatingActionButton.extended(
  onPressed: () {},
  icon: const Icon(Icons.add),
  label: const Text('Создать'),
)
```

---

## 4. Списки и карточки

```dart
// Card
Card(
  elevation: 2,
  margin: const EdgeInsets.all(8),
  child: Padding(
    padding: const EdgeInsets.all(16),
    child: Text('Содержимое карточки'),
  ),
)

// ListTile — строка списка
ListTile(
  leading: const CircleAvatar(child: Icon(Icons.person)),
  title: const Text('Иван Иванов'),
  subtitle: const Text('Пользователь'),
  trailing: const Icon(Icons.chevron_right),
  onTap: () {},
)

// ListView.builder — для длинных списков
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    final item = items[index];
    return ListTile(
      title: Text(item.name),
      onTap: () => Navigator.pushNamed(context, '/detail', arguments: item),
    );
  },
)
```

---

## 5. Диалоги и уведомления

```dart
// AlertDialog
showDialog<bool>(
  context: context,
  builder: (context) => AlertDialog(
    title: const Text('Удалить?'),
    content: const Text('Это действие необратимо'),
    actions: [
      TextButton(
        onPressed: () => Navigator.pop(context, false),
        child: const Text('Отмена'),
      ),
      FilledButton(
        onPressed: () => Navigator.pop(context, true),
        child: const Text('Удалить'),
      ),
    ],
  ),
);

// SnackBar
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: const Text('Сохранено'),
    action: SnackBarAction(label: 'Отмена', onPressed: () {}),
    duration: const Duration(seconds: 3),
  ),
);

// BottomSheet
showModalBottomSheet<void>(
  context: context,
  isScrollControlled: true, // для высоких sheet
  shape: const RoundedRectangleBorder(
    borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
  ),
  builder: (context) => Padding(
    padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom),
    child: const SizedBox(height: 300, child: Center(child: Text('Содержимое'))),
  ),
);
```

---

## 6. Индикаторы

```dart
// Линейный прогресс
LinearProgressIndicator(value: 0.7) // null = бесконечный
const LinearProgressIndicator()

// Круговой прогресс
const CircularProgressIndicator()
CircularProgressIndicator(value: 0.5)

// Паттерн загрузки
if (_isLoading)
  const Center(child: CircularProgressIndicator())
else
  ContentWidget()
```

---

## 7. Типичные ошибки

| Ошибка                                      | Проблема                     | Решение                                                           |
| ------------------------------------------- | ---------------------------- | ----------------------------------------------------------------- |
| `showSnackBar` без `mounted`                | Ошибка если виджет удалён    | `if (mounted) ScaffoldMessenger.of(context)...`                   |
| `showDialog` возвращает `null`              | Не обработан случай закрытия | `final result = await showDialog<bool>(...); if (result == true)` |
| Вложенные `Scaffold`                        | Конфликт `ScaffoldMessenger` | Один `Scaffold` на экран                                          |
| `ListView` внутри `Column` без `shrinkWrap` | Unbounded height exception   | `shrinkWrap: true` или `Expanded(child: ListView(...))`           |

---

## 8. Практические рекомендации

1. **`MaterialApp` в корне** — без него тема, навигация и локализация не работают.
2. **`FilledButton` вместо `ElevatedButton`** в Material 3 проектах.
3. **`ScaffoldMessenger` вместо `Scaffold.of(context).showSnackBar`** — новый API с Flutter 2.
4. **`ListView.builder` для списков > 10 элементов** — ленивое создание виджетов.
5. **`const` для статических виджетов** — `const ListTile(...)`, `const Icon(Icons.home)`.
6. **Избегай вложенных Scaffold** — используй один `Scaffold` на экран.
