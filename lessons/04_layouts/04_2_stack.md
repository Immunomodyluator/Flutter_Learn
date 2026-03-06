# 4.2 Stack и Positioned

## 1. Суть концепции

`Stack` позволяет располагать виджеты **поверх друг друга** — как слои в Photoshop. `Positioned` задаёт точную позицию дочернего виджета внутри Stack.

Используется для: оверлеев, бейджей, кастомных карточек, splash-экранов, шапок с фото.

---

## 2. Базовый синтаксис

```dart
Stack(
  alignment: Alignment.center,       // выравнивание непозиционированных детей
  fit: StackFit.loose,               // loose (по умолчанию) или expand
  clipBehavior: Clip.hardEdge,       // обрезать выходящие за bounds
  children: [
    // Нижний слой — рисуется первым
    Container(width: 200, height: 200, color: Colors.blue),

    // Верхний слой без позиции — выровнен по alignment
    const Text('По центру', style: TextStyle(color: Colors.white)),

    // Верхний слой с точной позицией
    Positioned(
      top: 10,
      right: 10,
      child: Container(
        width: 20,
        height: 20,
        decoration: const BoxDecoration(
          color: Colors.red,
          shape: BoxShape.circle,
        ),
      ),
    ),
  ],
)
```

---

## 3. Positioned — параметры

```dart
Positioned(
  top: 16,      // отступ сверху
  left: 16,     // отступ слева
  right: 16,    // отступ справа (ширина = Stack.width - left - right)
  bottom: 16,   // отступ снизу
  width: 100,   // явная ширина
  height: 50,   // явная высота
  child: ...,
)

// Заполнить весь Stack
Positioned.fill(
  child: Container(color: Colors.black38),  // оверлей поверх всего
)

// Заполнить с отступами
Positioned.fill(
  left: 16, right: 16, top: 16, bottom: 16,
  child: ...,
)
```

---

## 4. Реальный пример: карточка с бейджем

```dart
class NotificationBadge extends StatelessWidget {
  final Widget child;
  final int count;

  const NotificationBadge({super.key, required this.child, required this.count});

  @override
  Widget build(BuildContext context) {
    if (count == 0) return child;

    return Stack(
      clipBehavior: Clip.none,   // бейдж выходит за границы иконки
      children: [
        child,
        Positioned(
          top: -6,
          right: -6,
          child: Container(
            padding: const EdgeInsets.all(4),
            constraints: const BoxConstraints(minWidth: 18, minHeight: 18),
            decoration: BoxDecoration(
              color: Colors.red,
              shape: count > 9 ? BoxShape.rectangle : BoxShape.circle,
              borderRadius: count > 9 ? BorderRadius.circular(10) : null,
            ),
            child: Text(
              count > 99 ? '99+' : '$count',
              style: const TextStyle(color: Colors.white, fontSize: 10, fontWeight: FontWeight.bold),
              textAlign: TextAlign.center,
            ),
          ),
        ),
      ],
    );
  }
}

// Использование
NotificationBadge(
  count: 5,
  child: IconButton(
    icon: const Icon(Icons.notifications),
    onPressed: () {},
  ),
)
```

---

## 5. Пример: шапка профиля с фото

```dart
class ProfileHeader extends StatelessWidget {
  const ProfileHeader({super.key});

  @override
  Widget build(BuildContext context) {
    return Stack(
      alignment: Alignment.bottomCenter,
      children: [
        // Фоновое изображение
        Image.network(
          'https://picsum.photos/400/200',
          width: double.infinity,
          height: 200,
          fit: BoxFit.cover,
        ),
        // Аватар поверх изображения
        Positioned(
          bottom: -40,
          child: Stack(
            alignment: Alignment.bottomRight,
            children: [
              CircleAvatar(
                radius: 50,
                backgroundColor: Theme.of(context).scaffoldBackgroundColor,
                child: const CircleAvatar(
                  radius: 46,
                  backgroundImage: NetworkImage('https://picsum.photos/100'),
                ),
              ),
              // Кнопка редактирования
              Container(
                decoration: BoxDecoration(
                  color: Theme.of(context).colorScheme.primary,
                  shape: BoxShape.circle,
                ),
                child: IconButton(
                  icon: const Icon(Icons.camera_alt, color: Colors.white, size: 16),
                  onPressed: () {},
                  constraints: const BoxConstraints(minWidth: 32, minHeight: 32),
                  padding: EdgeInsets.zero,
                ),
              ),
            ],
          ),
        ),
      ],
    );
  }
}
```

---

## 6. IndexedStack — показывает только один слой

```dart
// Все дети существуют, но видим только один
// Состояние сохраняется при переключении (в отличие от Visibility)
int _currentTab = 0;

IndexedStack(
  index: _currentTab,
  children: const [
    HomeTab(),
    SearchTab(),
    ProfileTab(),
  ],
)

// Используется в BottomNavigationBar для сохранения состояния вкладок
```

---

## 7. Типичные ошибки

| Ошибка                            | Проблема                          | Решение                                                  |
| --------------------------------- | --------------------------------- | -------------------------------------------------------- |
| `Stack` без явного размера        | Размер = размер первого дочернего | Задать `width`/`height` или `SizedBox`                   |
| `Clip.hardEdge` + выход за bounds | Дочерние виджеты обрезаются       | `Clip.none` если нужен выход за границы                  |
| Позиционирование по %             | Сложно рассчитать                 | `Positioned` + `LayoutBuilder` для динамических размеров |
| `Positioned` без Stack-родителя   | Ошибка рендеринга                 | `Positioned` работает только внутри `Stack`              |

---

## 8. Практические рекомендации

1. **`Stack` + `Positioned.fill`** для оверлеев (шторка загрузки, блюр-фон).
2. **`clipBehavior: Clip.none`** если дочерний элемент должен выходить за границы Stack.
3. **`IndexedStack`** для TabBar с сохранением состояния вкладок.
4. **Сначала пиши нижние слои** — порядок в `children` = порядок отрисовки снизу вверх.
5. **Избегай глубоких вложений Stack** — сложно читать и дебажить.
