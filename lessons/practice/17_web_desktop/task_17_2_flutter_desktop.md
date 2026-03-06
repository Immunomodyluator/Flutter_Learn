# 17.2: Flutter Desktop — оконное приложение FitMenu

> Project: FitMenu | Глава 17 — Web & Desktop

### 17.2: Desktop версия FitMenu — размер окна и нативное меню

🎯 **Цель шага:** Настроить desktop-версию FitMenu для macOS/Windows: задать минимальный размер окна, добавить нативное меню (File, Edit, Help) и горячие клавиши.

---

📝 **Техническое задание:**

**Включить desktop:**
```bash
flutter config --enable-macos-desktop
flutter config --enable-windows-desktop
flutter create . --platforms macos,windows
flutter run -d macos
```

**Зависимость:**
```yaml
dependencies:
  window_manager: ^0.3.7
```

**Управление окном:**
```dart
// lib/main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  if (isDesktop) {
    await windowManager.ensureInitialized();
    await windowManager.waitUntilReadyToShow(
      const WindowOptions(
        size:        Size(1200, 800),
        minimumSize: Size(900, 600),
        center:      true,
        title:       'FitMenu',
        titleBarStyle: TitleBarStyle.normal,
      ),
      () async {
        await windowManager.show();
        await windowManager.focus();
      },
    );
  }

  runApp(const FitMenuApp());
}
```

**Нативное меню (macOS PlatformMenuBar):**
```dart
class FitMenuApp extends StatelessWidget {
  const FitMenuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return PlatformMenuBar(
      menus: [
        PlatformMenu(label: 'FitMenu', menus: [
          PlatformMenuItem(
            label: 'О программе',
            onSelected: () => showAboutDialog(context: context),
          ),
          PlatformMenuItemGroup(members: [
            PlatformMenuItem(
              label: 'Настройки...',
              shortcut: const SingleActivator(LogicalKeyboardKey.comma, meta: true),
              onSelected: () => context.go('/settings'),
            ),
          ]),
          const PlatformProvidedMenuItem(type: PlatformProvidedMenuItemType.quit),
        ]),
        PlatformMenu(label: 'Файл', menus: [
          PlatformMenuItem(
            label: 'Экспорт данных...',
            shortcut: const SingleActivator(LogicalKeyboardKey.keyE, meta: true),
            onSelected: () => _exportData(context),
          ),
        ]),
      ],
      child: MaterialApp.router(...),
    );
  }
}
```

---

✅ **Критерии приёмки:**
- [ ] `windowManager` задаёт начальный размер 1200×800
- [ ] Минимальный размер окна 900×600 (нельзя сжать меньше)
- [ ] Нативное меню отображается в macOS menu bar
- [ ] `PlatformMenuItem` с `SingleActivator` работает по горячей клавише
- [ ] Заголовок окна отображает текущий экран

---

💡 **Подсказка:** `kIsWeb` и `Platform.isDesktop` — проверка платформы. `isDesktop = !kIsWeb && (Platform.isMacOS || Platform.isWindows || Platform.isLinux)`. `WindowListener` mixin позволяет реагировать на изменение размера/фокуса окна.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/platform/window_utils.dart

import 'dart:io';
import 'package:flutter/foundation.dart';
import 'package:window_manager/window_manager.dart';

bool get isDesktopPlatform =>
    !kIsWeb && (Platform.isMacOS || Platform.isWindows || Platform.isLinux);

Future<void> setupDesktopWindow() async {
  if (!isDesktopPlatform) return;
  await windowManager.ensureInitialized();
  await windowManager.waitUntilReadyToShow(
    const WindowOptions(
      size:           Size(1280, 800),
      minimumSize:    Size(960, 640),
      center:         true,
      title:          'FitMenu',
      backgroundColor: Colors.transparent,
      titleBarStyle:  TitleBarStyle.normal,
    ),
    () async {
      await windowManager.show();
      await windowManager.focus();
    },
  );
}
```

```dart
// lib/main.dart

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await setupDesktopWindow();
  runApp(const FitMenuApp());
}
```

```dart
// lib/app.dart (с нативным меню)

class FitMenuApp extends StatelessWidget {
  const FitMenuApp({super.key});

  @override
  Widget build(BuildContext context) {
    final app = MaterialApp.router(
      title:        'FitMenu',
      theme:        AppTheme.lightTheme,
      darkTheme:    AppTheme.darkTheme,
      themeMode:    ThemeMode.system,
      routerConfig: AppRouter.router,
    );

    // PlatformMenuBar работает только на macOS
    if (Platform.isMacOS) {
      return PlatformMenuBar(
        menus: _buildMenus(),
        child: app,
      );
    }
    return app;
  }

  List<PlatformMenuDelegate> _buildMenus() => [
    PlatformMenu(
      label: 'FitMenu',
      menus: [
        const PlatformProvidedMenuItem(type: PlatformProvidedMenuItemType.about),
        const PlatformMenuDivider(),
        const PlatformProvidedMenuItem(type: PlatformProvidedMenuItemType.quit),
      ],
    ),
    PlatformMenu(label: 'Файл', menus: [
      PlatformMenuItem(
        label: 'Новый план питания',
        shortcut: const SingleActivator(LogicalKeyboardKey.keyN, meta: true),
        onSelected: () {},
      ),
    ]),
  ];
}
```

</details>
