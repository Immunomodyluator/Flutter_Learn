# 11.1 Implicit-анимации (неявные)

## 1. Суть

Implicit-анимации — самый простой способ анимировать UI. Виджет автоматически анимирует переход от старого значения к новому при `setState`. Не нужен `AnimationController`.

---

## 2. Готовые Animated-виджеты

```dart
// AnimatedContainer — анимирует любые свойства Container
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: _isExpanded ? 200 : 100,
  height: _isExpanded ? 200 : 100,
  color: _isExpanded ? Colors.blue : Colors.red,
  decoration: BoxDecoration(
    borderRadius: BorderRadius.circular(_isExpanded ? 16 : 50),
  ),
)

// AnimatedOpacity — плавное появление/исчезновение
AnimatedOpacity(
  duration: const Duration(milliseconds: 500),
  opacity: _isVisible ? 1.0 : 0.0,
  child: const Text('Привет!'),
)

// AnimatedAlign — анимация выравнивания
AnimatedAlign(
  duration: const Duration(milliseconds: 400),
  alignment: _isTop ? Alignment.topCenter : Alignment.bottomCenter,
  child: const FlutterLogo(size: 50),
)

// AnimatedPadding
AnimatedPadding(
  duration: const Duration(milliseconds: 300),
  padding: EdgeInsets.all(_padding),
  child: const Card(child: Text('Card')),
)

// AnimatedPositioned (внутри Stack)
Stack(children: [
  AnimatedPositioned(
    duration: const Duration(milliseconds: 400),
    left: _left,
    top: _top,
    child: const FlutterLogo(size: 60),
  ),
])

// AnimatedCrossFade — переключение между двумя виджетами
AnimatedCrossFade(
  duration: const Duration(milliseconds: 300),
  firstChild: const Text('Состояние A'),
  secondChild: const Text('Состояние B'),
  crossFadeState: _showFirst ? CrossFadeState.showFirst : CrossFadeState.showSecond,
)

// AnimatedSwitcher — анимированная замена дочернего виджета
AnimatedSwitcher(
  duration: const Duration(milliseconds: 300),
  transitionBuilder: (child, animation) => ScaleTransition(scale: animation, child: child),
  child: Text('$_count', key: ValueKey(_count)), // key обязателен!
)
```

---

## 3. TweenAnimationBuilder — кастомная implicit-анимация

```dart
// TweenAnimationBuilder — когда нет готового Animated-виджета
TweenAnimationBuilder<double>(
  duration: const Duration(milliseconds: 600),
  tween: Tween<double>(begin: 0, end: _targetValue),
  curve: Curves.elasticOut,
  builder: (context, value, child) {
    return Transform.scale(
      scale: value,
      child: child, // child не перестраивается на каждом кадре
    );
  },
  child: const FlutterLogo(size: 100), // статичная часть
)

// Анимация цвета
TweenAnimationBuilder<Color?>(
  duration: const Duration(milliseconds: 400),
  tween: ColorTween(
    begin: Colors.blue,
    end: _isDark ? Colors.grey.shade900 : Colors.white,
  ),
  builder: (context, color, _) {
    return Container(
      color: color,
      child: const Placeholder(),
    );
  },
)
```

---

## 4. Реальный пример — анимированная кнопка лайка

```dart
class LikeButton extends StatefulWidget {
  final bool isLiked;
  final int likeCount;
  final VoidCallback onTap;

  const LikeButton({
    super.key,
    required this.isLiked,
    required this.likeCount,
    required this.onTap,
  });

  @override
  State<LikeButton> createState() => _LikeButtonState();
}

class _LikeButtonState extends State<LikeButton> {
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: widget.onTap,
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          // Анимация иконки — ScaleTransition через AnimatedSwitcher
          AnimatedSwitcher(
            duration: const Duration(milliseconds: 200),
            transitionBuilder: (child, animation) {
              return ScaleTransition(
                scale: Tween<double>(begin: 0.5, end: 1.0).animate(animation),
                child: child,
              );
            },
            child: Icon(
              widget.isLiked ? Icons.favorite : Icons.favorite_border,
              key: ValueKey(widget.isLiked),
              color: widget.isLiked ? Colors.red : Colors.grey,
              size: 24,
            ),
          ),
          const SizedBox(width: 4),
          // Анимация счётчика
          AnimatedSwitcher(
            duration: const Duration(milliseconds: 200),
            transitionBuilder: (child, animation) {
              return SlideTransition(
                position: Tween<Offset>(
                  begin: const Offset(0, 0.5),
                  end: Offset.zero,
                ).animate(animation),
                child: FadeTransition(opacity: animation, child: child),
              );
            },
            child: Text(
              '${widget.likeCount}',
              key: ValueKey(widget.likeCount),
              style: const TextStyle(fontSize: 14),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

## 5. Кривые анимации (Curves)

```dart
// Часто используемые кривые
Curves.linear          // равномерно
Curves.easeIn          // медленно → быстро
Curves.easeOut         // быстро → медленно
Curves.easeInOut       // медленно → быстро → медленно (самый частый)
Curves.bounceOut       // отскок в конце
Curves.elasticOut      // пружина в конце
Curves.fastOutSlowIn   // Material Design стандарт
```

---

## 6. Под капотом

- `AnimatedContainer` — внутри использует `ImplicitlyAnimatedWidget` + `AnimationController`.
- При изменении значения запускается анимация от текущего к новому.
- `TweenAnimationBuilder` каждый раз создаёт новый контроллер при изменении `tween.end`.
- `child` в `TweenAnimationBuilder` не перестраивается — оптимизация производительности.

---

## 7. Типичные ошибки

| Ошибка                          | Причина                               | Решение                                       |
| ------------------------------- | ------------------------------------- | --------------------------------------------- |
| `AnimatedSwitcher` не анимирует | Нет уникального `key`                 | Добавь `key: ValueKey(uniqueValue)`           |
| Анимация не запускается         | `setState` не вызван                  | Убедись что изменяешь переменную состояния    |
| Мерцание при AnimatedCrossFade  | Разные размеры виджетов               | Используй `SizedBox` одинакового размера      |
| `TweenAnimationBuilder` мигает  | `tween` пересоздаётся на каждый build | Вынеси `Tween` за `build()` или в поле класса |

---

## 8. Рекомендации

1. **Начинай с `AnimatedContainer`** — покрывает большинство случаев.
2. **`AnimatedSwitcher` + `ValueKey`** — лучший способ анимировать замену виджета.
3. **Кривая `easeInOut`** — универсальный выбор для большинства UI-анимаций.
4. **`TweenAnimationBuilder`** — когда нужна анимация нестандартных свойств без `AnimationController`.
5. **Оборачивай `child`** в `TweenAnimationBuilder` — он не перестраивается на каждом кадре.
