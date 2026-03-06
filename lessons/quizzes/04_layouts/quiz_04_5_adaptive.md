# Квиз: Adaptive Layout

**Тема:** 04.5 — Adaptive Layout  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как получить ширину экрана в Flutter?

- A) `Screen.width`
- B) `MediaQuery.of(context).size.width`
- C) `window.physicalSize.width`
- D) `context.screenWidth`

<details>
<summary>Ответ</summary>

**B) `MediaQuery.of(context).size.width`**

`MediaQuery.of(context)` возвращает `MediaQueryData` с `size`, `padding` (safe area), `viewInsets` (клавиатура), `devicePixelRatio`, `platformBrightness` и др. В Flutter 3.7+ есть `MediaQuery.sizeOf(context)` — подписывается только на изменения `size`, без лишних перестроек.

</details>

---

### Вопрос 2 🟢

Для чего используется `LayoutBuilder`?

- A) Создаёт layout виджетов автоматически
- B) Строит дерево виджетов в зависимости от `BoxConstraints` родителя
- C) Оптимизирует перестройку layout
- D) Заменяет `MediaQuery`

<details>
<summary>Ответ</summary>

**B) Строит дерево виджетов в зависимости от `BoxConstraints` родителя**

```dart
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth > 600) {
      return WideLayout();
    }
    return NarrowLayout();
  },
)
```

Отличие от `MediaQuery`: работает с реальным доступным пространством виджета, а не всего экрана.

</details>

---

### Вопрос 3 🟢

Как определить текущую ориентацию устройства?

- A) `device.orientation`
- B) `MediaQuery.of(context).orientation` → `Orientation.portrait` / `landscape`
- C) `OrientationBuilder` — единственный способ
- D) `Platform.isLandscape`

<details>
<summary>Ответ</summary>

**B) `MediaQuery.of(context).orientation`**

Возвращает `Orientation.portrait` или `Orientation.landscape`. `OrientationBuilder` — удобная обёртка для реагирования на смену ориентации в build методе. Оба подхода корректны.

</details>

---

### Вопрос 4 🟡

Что означает паттерн breakpoints в responsive дизайне?

- A) Точки, где приложение вылетает
- B) Пороговые значения ширины экрана, при которых меняется **layout** (mobile/tablet/desktop)
- C) Точки оптимизации производительности
- D) Размеры шрифтов для разных экранов

<details>
<summary>Ответ</summary>

**B) Пороговые значения ширины экрана, при которых меняется layout**

Общепринятые точки:

- < 600px → Mobile (1 колонка)
- 600–1200px → Tablet (2 колонки)
- > 1200px → Desktop (3+ колонки)

Реализуется через `LayoutBuilder` или `MediaQuery`. Пакет `flutter_adaptive_scaffold` от Google предоставляет готовые breakpoints.

</details>

---

### Вопрос 5 🟡

Как адаптировать отступы под нотч/вырез экрана?

- A) Добавить фиксированный `padding: EdgeInsets.only(top: 40)`
- B) `SafeArea` виджет или `MediaQuery.of(context).padding`
- C) `NotchAware` виджет
- D) Это делается автоматически

<details>
<summary>Ответ</summary>

**B) `SafeArea` виджет или `MediaQuery.of(context).padding`**

`SafeArea` добавляет отступы на основе `viewPadding` (нотач, закруглённые углы, StatusBar). `MediaQuery.of(context).padding` содержит конкретные значения: `padding.top`, `padding.bottom`. `SafeArea(top: false)` — отключает отступ сверху, но оставляет снизу.

</details>

---

### Вопрос 6 🟡

Какой виджет использовать для адаптивного NavigationBar (нижний на mobile, боковой на tablet)?

- A) `NavigationBar` с условием
- B) `AdaptiveNavigationScaffold` из `adaptive_navigation` или реализовать через `LayoutBuilder` + `NavigationRail` / `NavigationBar`
- C) `PlatformScaffold`
- D) Flutter автоматически переключает тип навигации

<details>
<summary>Ответ</summary>

**B) Реализовать через `LayoutBuilder` + `NavigationRail` / `NavigationBar`**

```dart
LayoutBuilder(builder: (context, constraints) {
  if (constraints.maxWidth >= 600) {
    return Row(children: [NavigationRail(...), Expanded(child: body)]);
  }
  return Scaffold(bottomNavigationBar: NavigationBar(...), body: body);
})
```

`flutter_adaptive_scaffold` от Google автоматизирует этот паттерн.

</details>

---

### Вопрос 7 🟡

Как сделать текст адаптивным к размеру экрана?

- A) `Text(style: TextStyle(fontSize: MediaQuery.of(context).size.width * 0.04))`
- B) `FittedBox(child: Text(...))`
- C) Оба подхода корректны в разных ситуациях
- D) `AutoSizeText` из пакета — единственный правильный вариант

<details>
<summary>Ответ</summary>

**C) Оба подхода корректны в разных ситуациях**

- `MediaQuery.size.width * k` — масштабируется пропорционально ширине экрана
- `FittedBox` — вписывает текст в доступное пространство (масштабирует)
- `auto_size_text` пакет — уменьшает шрифт чтобы текст вписался, с поддержкой `minFontSize`
- `TextScaler` (Flutter 3.12+) — учитывает системные настройки размера шрифта

</details>

---

### Вопрос 8 🔴

Как реализовать разный UI для Web и Mobile в Flutter?

- A) `if (Platform.isAndroid)`
- B) `kIsWeb` константа + `Platform.isIOS/isAndroid` для мобильных
- C) Только через сборку разных flavors
- D) `dart:html` для веба автоматически

<details>
<summary>Ответ</summary>

**B) `kIsWeb` константа + `Platform.isIOS/isAndroid` для мобильных**

```dart
import 'package:flutter/foundation.dart';

if (kIsWeb) {
  return WebLayout();
} else if (Platform.isAndroid || Platform.isIOS) {
  return MobileLayout();
} else {
  return DesktopLayout();
}
```

`Platform` из `dart:io` недоступен на Web — всегда проверяйте `kIsWeb` первым.

</details>

---

### Вопрос 9 🔴

Что такое `flutter_adaptive_scaffold` и какую проблему он решает?

- A) Пакет для анимированных переходов между layouts
- B) Google-пакет реализующий Material 3 canonical layouts с адаптацией под size classes (compact/medium/expanded)
- C) Аналог ResponsiveBuilder для веба
- D) Пакет для тестирования адаптивных layouts

<details>
<summary>Ответ</summary>

**B) Google-пакет реализующий Material 3 canonical layouts с адаптацией под size classes**

`AdaptiveScaffold` автоматически переключает `BottomNavigationBar` → `NavigationRail` → `NavigationDrawer` на основе ширины экрана по Material 3 breakpoints (compact < 600, medium 600–840, expanded > 840). Поддерживает вторичные body (two-pane layouts).

</details>

---

### Вопрос 10 🔴

Как оптимизировать `MediaQuery.of(context)` чтобы избежать лишних перестроек?

- A) Кэшировать в `initState`
- B) Использовать специализированные extensions: `MediaQuery.sizeOf(context)`, `MediaQuery.paddingOf(context)` вместо `MediaQuery.of(context).size`
- C) Обернуть в `RepaintBoundary`
- D) `MediaQuery` не вызывает лишних перестроек — оптимизация не нужна

<details>
<summary>Ответ</summary>

**B) Использовать специализированные extensions в Flutter 3.7+**

`MediaQuery.of(context)` подписывает виджет на **все** изменения `MediaQueryData` (ориентация, клавиатура, brightness...). Если нужна только ширина — `MediaQuery.sizeOf(context)` перестраивает только при изменении `size`. Аналогично: `paddingOf`, `viewInsetsOf`, `textScalerOf`, `platformBrightnessOf`.

</details>
