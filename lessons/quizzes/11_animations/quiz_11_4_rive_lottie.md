# Квиз: Rive и Lottie анимации

**Тема:** 11.4 — Rive / Lottie  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какое различие между Rive и Lottie анимациями?

- A) Они идентичны — оба для Flutter
- B) Lottie — After Effects экспорт (JSON); Rive — нативный редактор с интерактивными State Machines и реал-тайм управлением параметрами
- C) Rive — только для iOS; Lottie — для Android
- D) Lottie быстрее Rive

<details>
<summary>Ответ</summary>

**B) Lottie = After Effects JSON; Rive = интерактивная state machine с редактором**

|                 | Lottie          | Rive                        |
| --------------- | --------------- | --------------------------- |
| Источник        | After Effects   | Rive редактор               |
| Формат          | JSON (.lottie)  | .riv                        |
| Интерактивность | ограничена      | State Machine, Inputs       |
| Управление      | play/stop/speed | triggers, numbers, booleans |
| Сложность       | проще           | мощнее                      |

</details>

---

### Вопрос 2 🟢

Как воспроизвести Lottie анимацию из assets?

- A) `LottieAnimation.asset('assets/anim.json')`
- B) `Lottie.asset('assets/animations/loading.json')` — рендерит JSON анимацию
- C) `AnimatedLottie(path: 'assets/anim.json')`
- D) `LottieBuilder.asset()`

<details>
<summary>Ответ</summary>

**B) `Lottie.asset(path)` (пакет `lottie`)**

```dart
// В pubspec.yaml:
// assets:
//   - assets/animations/loading.json

// Простое воспроизведение:
Lottie.asset('assets/animations/loading.json')

// С настройками:
Lottie.asset(
  'assets/animations/success.json',
  width: 200,
  height: 200,
  fit: BoxFit.cover,
  repeat: false,  // один раз
  reverse: false,
  animate: true,
)
```

</details>

---

### Вопрос 3 🟢

Как воспроизвести Rive анимацию из assets?

- A) `RiveAnimation.asset('path', animations: ['idle'])`
- B) `RivePlayer(asset: 'path')`
- C) `Rive(file: riveFile, animation: 'name')`
- D) Нужно сначала вручную загрузить `.riv` файл

<details>
<summary>Ответ</summary>

**A) `RiveAnimation.asset(path, animations: ['animationName'])`**

```dart
// Простое воспроизведение именованной анимации:
RiveAnimation.asset(
  'assets/animations/character.riv',
  animations: const ['idle'], // имя анимации из Rive редактора
  fit: BoxFit.cover,
)

// Из сети:
RiveAnimation.network('https://example.com/anim.riv')
```

</details>

---

### Вопрос 4 🟡

Как управлять Lottie анимацией программно (старт, стоп, скорость)?

- A) `Lottie.asset(onLoaded: controller.play)`
- B) Создать `AnimationController` и передать его в `Lottie.asset(controller: ...)` — контроллировать как обычную анимацию Flutter
- C) `LottieController.instance.play()`
- D) `lottiePlayer.seekTo(0.5)`

<details>
<summary>Ответ</summary>

**B) Flutter `AnimationController` + `Lottie.asset(controller: ...)`**

```dart
class _LottieState extends State<LottieWidget> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
  }

  @override
  Widget build(ctx) => Lottie.asset(
    'assets/check.json',
    controller: _controller,
    onLoaded: (composition) {
      // duration из самого файла:
      _controller
        ..duration = composition.duration
        ..forward(); // запустить
    },
  );
}
```

</details>

---

### Вопрос 5 🟡

Как использовать Rive State Machine для интерактивного управления анимацией?

- A) Напрямую вызывать `animation.play('stateName')`
- B) Получить `StateMachineController`, добавить в артборд через `RiveAnimation` — использовать Input объекты для управления состоянием
- C) `RiveStateMachine.trigger('event')`
- D) State Machine управляется только в Rive редакторе

<details>
<summary>Ответ</summary>

**B) `StateMachineController` + Input объекты**

```dart
StateMachineController? _controller;
SMITrigger? _clickTrigger;
SMIBool? _isHovered;
SMINumber? _speed;

RiveAnimation.asset(
  'assets/button.riv',
  stateMachines: const ['ButtonMachine'],
  onInit: (artboard) {
    _controller = StateMachineController.fromArtboard(artboard, 'ButtonMachine');
    if (_controller != null) {
      artboard.addController(_controller!);
      _clickTrigger = _controller!.findInput<bool>('Click') as SMITrigger;
      _isHovered = _controller!.findInput<bool>('Hovered') as SMIBool;
      _speed = _controller!.findInput<double>('Speed') as SMINumber;
    }
  },
)

// Использование:
void onTap() => _clickTrigger?.fire();
void onHover(bool hovered) => _isHovered?.change(hovered);
void setSpeed(double speed) => _speed?.change(speed);
```

</details>

---

### Вопрос 6 🟡

Как сделать Lottie анимацию срабатывающей один раз при определённом событии?

- A) `repeat: false` + `animate: true`
- B) `AnimationController.addStatusListener()` + `forward()` при событии; `repeat: false` с `AnimationStatus.completed` для cleanup
- C) `Lottie.asset(onComplete: () {})`
- D) `Future.delayed` + `setState(() => animate = false)`

<details>
<summary>Ответ</summary>

**B) `AnimationController` + `forward()` при событии + `StatusListener` для завершения**

```dart
void _playSuccessAnimation() {
  _controller.reset();
  _controller.forward();
}

// Слушать завершение:
_controller.addStatusListener((status) {
  if (status == AnimationStatus.completed) {
    // Анимация завершена — показать следующий экран:
    Navigator.of(ctx).pushReplacement(...);
  }
});

// Кнопка с Lottie:
GestureDetector(
  onTap: _playSuccessAnimation,
  child: Lottie.asset('assets/success.json', controller: _controller, repeat: false),
)
```

</details>

---

### Вопрос 7 🟡

Как оптимизировать производительность Lottie на слабых устройствах?

- A) Уменьшить `frameRate`
- B) Предварительно кэшировать (`LottieComposition.fromAsset`), использовать `renderCache: RenderCache.raster` для статичных анимаций
- C) Конвертировать в GIF
- D) Использовать только `repeat: false`

<details>
<summary>Ответ</summary>

**B) `renderCache` + предзагрузка**

```dart
// renderCache = raster: сохранять каждый кадр как растровое изображение
// Хорошо для: анимации без изменения размера, медленные устройства
// Плохо для: анимации которые масштабируются, съедает больше памяти
Lottie.asset(
  'assets/background_loop.json',
  renderCache: RenderCache.raster, // кэшировать как растр
)

// Предзагрузить в initState:
LottieComposition? _composition;

@override
void initState() async {
  super.initState();
  _composition = await LottieComposition.fromAsset(
    DefaultAssetBundle.of(ctx),
    'assets/animation.json',
  );
}
```

</details>

---

### Вопрос 8 🔴

Как динамически изменить цвет в Lottie анимации?

- A) Нельзя — цвета фиксированы в JSON
- B) `LottieDelegates` с `valueCallback` для конкретных слоёв — программная замена цветов без изменения JSON
- C) Редактировать JSON файл
- D) `Lottie.asset(colorFilter: ColorFilter.mode(...))`

<details>
<summary>Ответ</summary>

**B) `LottieDelegates` + `ValueDelegate` для программного изменения**

```dart
Lottie.asset(
  'assets/icon.json',
  delegates: LottieDelegates(
    values: [
      // Изменить заливку конкретного слоя:
      ValueDelegate.color(
        const ['IconLayer', 'Shape', 'Fill'],
        value: Colors.red, // новый цвет
      ),
      ValueDelegate.opacity(
        const ['BackgroundLayer'],
        value: 0, // скрыть слой
      ),
      ValueDelegate.strokeColor(
        const ['OutlineLayer', '**'], // ** = все дочерние
        value: Colors.blue,
      ),
    ],
  ),
)
```

</details>

---

### Вопрос 9 🔴

Как использовать Rive Nested Artboards для составных анимаций?

- A) Nested Artboards недоступны в Flutter
- B) В Rive редакторе создать Nested Artboard — Flutter автоматически рендерит вложенные арт-доски; управление через SwapArtboard input
- C) Загрузить несколько `.riv` файлов и совместить
- D) `RiveNestedAnimation(children: [...])`

<details>
<summary>Ответ</summary>

**B) Nested Artboards создаются в Rive и рендерятся автоматически**

```dart
// Nested Artboard через State Machine:
RiveAnimation.asset(
  'assets/complex.riv',
  artboard: 'MainArtboard',
  stateMachines: const ['MainMachine'],
  onInit: (artboard) {
    final controller = StateMachineController.fromArtboard(
      artboard, 'MainMachine',
    );
    artboard.addController(controller!);

    // SwapArtboard позволяет менять вложенный артборд:
    final swapInput = controller.findInput<double>('ArtboardVariant') as SMINumber;
    swapInput?.change(2); // переключить на вариант 2
  },
)
```

</details>

---

### Вопрос 10 🔴

Как обработать ошибку загрузки Lottie/Rive и показать fallback?

- A) Ошибки невозможны в production
- B) `Lottie.network()` / `RiveAnimation.network()` — использовать `errorBuilder` или оборнуть в `FutureBuilder` с обработкой ошибок
- C) `try { Lottie.asset() } catch { fallback }`
- D) `Lottie.asset(onError: callback)`

<details>
<summary>Ответ</summary>

**B) `errorBuilder` для network, `try-catch` для asset загрузки**

```dart
// Lottie с fallback:
Lottie.network(
  'https://example.com/animation.json',
  errorBuilder: (ctx, error, stackTrace) {
    return const Icon(Icons.animation, size: 100); // fallback
  },
  frameBuilder: (ctx, child, composition) {
    if (composition == null) return const CircularProgressIndicator();
    return child;
  },
)

// Rive asset с обработкой:
RiveAnimation.asset(
  'assets/widget.riv',
  placeHolder: const CircularProgressIndicator(),
  onInit: (artboard) {
    // успешная инициализация
  },
)

// Предварительная загрузка с ошибкой:
Future<void> _loadRive() async {
  try {
    final data = await RiveFile.asset('assets/animation.riv');
    setState(() => _riveFile = data);
  } catch (e) {
    setState(() => _loadError = true);
  }
}
```

</details>
