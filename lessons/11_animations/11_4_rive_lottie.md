# 11.4 Rive и Lottie — дизайнерские анимации

## 1. Суть

**Lottie** — воспроизводит JSON-анимации, созданные в Adobe After Effects. Векторные, легковесные, кросс-платформенные.

**Rive** — интерактивные анимации с State Machine. Дизайнер создаёт анимацию в Rive-редакторе → разработчик управляет ею из кода (вход, выход, нажатие и т.д.).

```yaml
dependencies:
  lottie: ^3.1.0
  rive: ^0.12.3
```

---

## 2. Lottie — использование

```dart
import 'package:lottie/lottie.dart';

// Из assets (добавить в pubspec.yaml flutter: assets: [assets/animations/])
Lottie.asset(
  'assets/animations/loading.json',
  width: 200,
  height: 200,
  fit: BoxFit.contain,
)

// Из сети
Lottie.network('https://assets.lottiefiles.com/packages/lf20_xyadoh9h.json')

// Контроль через AnimationController
class LoadingAnimation extends StatefulWidget {
  const LoadingAnimation({super.key});

  @override
  State<LoadingAnimation> createState() => _LoadingAnimationState();
}

class _LoadingAnimationState extends State<LoadingAnimation>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Lottie.asset(
      'assets/animations/success.json',
      controller: _controller,
      onLoaded: (composition) {
        // Задаём длительность из файла анимации
        _controller.duration = composition.duration;
        _controller.forward().whenComplete(() {
          // Анимация завершена
        });
      },
    );
  }
}
```

---

## 3. Rive — базовое воспроизведение

```dart
import 'package:rive/rive.dart';

// Простое воспроизведение из assets
// pubspec.yaml: assets: [assets/animations/car.riv]
RiveAnimation.asset(
  'assets/animations/car.riv',
  animations: const ['idle'], // имя анимации из Rive редактора
  fit: BoxFit.contain,
)

// Из сети
RiveAnimation.network(
  'https://cdn.rive.app/animations/vehicles.riv',
)
```

---

## 4. Rive — State Machine (интерактивность)

```dart
// Rive State Machine позволяет управлять состоянием анимации из кода
class RiveButtonExample extends StatefulWidget {
  const RiveButtonExample({super.key});

  @override
  State<RiveButtonExample> createState() => _RiveButtonExampleState();
}

class _RiveButtonExampleState extends State<RiveButtonExample> {
  StateMachineController? _controller;
  SMITrigger? _pressTrigger;     // триггер (одноразовое событие)
  SMIBool? _isHovered;            // булевый инпут
  SMINumber? _progressValue;      // числовой инпут

  void _onRiveInit(Artboard artboard) {
    // Получаем контроллер State Machine
    final ctrl = StateMachineController.fromArtboard(
      artboard,
      'ButtonMachine', // имя State Machine из редактора
    );
    if (ctrl == null) return;

    artboard.addController(ctrl);
    _controller = ctrl;

    // Получаем инпуты по имени
    _pressTrigger = ctrl.findSMI('Press') as SMITrigger?;
    _isHovered = ctrl.findSMI('isHovered') as SMIBool?;
    _progressValue = ctrl.findSMI('progress') as SMINumber?;
  }

  @override
  void dispose() {
    _controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => _pressTrigger?.fire(),           // запустить триггер
      onTapDown: (_) => _isHovered?.change(true),   // изменить bool
      onTapUp: (_) => _isHovered?.change(false),
      child: SizedBox(
        width: 200,
        height: 80,
        child: RiveAnimation.asset(
          'assets/animations/button.riv',
          onInit: _onRiveInit,
        ),
      ),
    );
  }
}

// Обновление числового значения (например, прогресс загрузки)
void updateProgress(double value) {
  _progressValue?.change(value * 100); // 0-100
}
```

---

## 5. Реальный пример — Loading-состояние с Lottie

```dart
class DataScreen extends StatelessWidget {
  final AsyncValue<List<Item>> items;

  const DataScreen({super.key, required this.items});

  @override
  Widget build(BuildContext context) {
    return switch (items) {
      AsyncLoading() => Center(
          child: Lottie.asset(
            'assets/animations/loading_dots.json',
            width: 120,
            height: 120,
            repeat: true,
          ),
        ),
      AsyncError(:final error) => Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Lottie.asset('assets/animations/error.json', repeat: false),
              Text('Ошибка: $error'),
            ],
          ),
        ),
      AsyncData(:final value) => ListView.builder(
          itemCount: value.length,
          itemBuilder: (_, i) => ListTile(title: Text(value[i].name)),
        ),
      _ => const SizedBox(),
    };
  }
}
```

---

## 6. Под капотом

- **Lottie** десериализует JSON → `LottieComposition` → рендерит через `Canvas` каждый кадр.
- **Rive** использует собственный рантайм на C++ (через FFI) — очень производительный.
- Rive State Machine — конечный автомат: состояния + переходы по инпутам.
- Оба поддерживают `AnimationController` Flutter для синхронизации с другими анимациями.

---

## 7. Типичные ошибки

| Ошибка                         | Причина                     | Решение                                         |
| ------------------------------ | --------------------------- | ----------------------------------------------- |
| Lottie не загружается          | Файл не в assets            | Добавь путь в pubspec.yaml `flutter: assets:`   |
| Rive State Machine не работает | Неверное имя machine/инпута | Проверь точные имена в Rive-редакторе           |
| Rive чёрный экран              | Artboard не найден          | Укажи `artboard: 'Artboard Name'` явно          |
| Lottie большой размер файла    | Много кадров/слоёв          | Оптимизируй в LottieFiles или уменьши framerate |
| `controller` зависает          | Не вызван `dispose`         | `_riveController?.dispose()` в `dispose()`      |

---

## 8. Рекомендации

1. **Lottie** — для декоративных анимаций (loading, success, empty state).
2. **Rive** — для интерактивных анимаций (кнопки, персонажи, онбординг).
3. **Оптимизируй Lottie-файлы** перед использованием на lottiefiles.com.
4. **`repeat: false`** для однократных анимаций (success, error).
5. **Кешируй Rive файлы** — тяжёлые файлы загружай заранее через `RiveFile.asset()`.
