# 17.2 Flutter Desktop

## 1. Суть

Flutter Desktop позволяет запускать Flutter-приложения на macOS, Windows и Linux. Использует нативные окна системы, но рендерит UI через Impeller/Skia.

```bash
# Включить поддержку Desktop
flutter config --enable-macos-desktop
flutter config --enable-windows-desktop
flutter config --enable-linux-desktop

# Запустить
flutter run -d macos
flutter run -d windows
flutter run -d linux

# Сборка
flutter build macos
flutter build windows
flutter build linux
```

---

## 2. Размер окна и его настройка

```yaml
# pubspec.yaml
dependencies:
  window_manager: ^0.3.8
```

```dart
import 'package:window_manager/window_manager.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  if (!kIsWeb && (Platform.isMacOS || Platform.isWindows || Platform.isLinux)) {
    await windowManager.ensureInitialized();

    WindowOptions options = const WindowOptions(
      size: Size(1200, 800),
      minimumSize: Size(800, 600),
      center: true,
      title: 'My Desktop App',
      backgroundColor: Colors.transparent,
      skipTaskbar: false,
      titleBarStyle: TitleBarStyle.default_,
    );

    await windowManager.waitUntilReadyToShow(options, () async {
      await windowManager.show();
      await windowManager.focus();
    });
  }

  runApp(const MyApp());
}
```

---

## 3. Нативное меню приложения (macOS/Windows)

```yaml
dependencies:
  menubar: ^3.0.1
  # или встроенные platformMenuBar
```

```dart
// Flutter встроенный PlatformMenuBar (Flutter 3.0+)
PlatformMenuBar(
  menus: [
    PlatformMenu(
      label: 'Файл',
      menus: [
        PlatformMenuItem(
          label: 'Новый',
          shortcut: const SingleActivator(LogicalKeyboardKey.keyN, meta: true),
          onSelected: () => createNewFile(),
        ),
        PlatformMenuItem(
          label: 'Открыть...',
          shortcut: const SingleActivator(LogicalKeyboardKey.keyO, meta: true),
          onSelected: () => openFile(),
        ),
        const PlatformMenuDivider(),
        PlatformMenuItem(
          label: 'Закрыть',
          shortcut: const SingleActivator(LogicalKeyboardKey.keyW, meta: true),
          onSelected: () => closeCurrentTab(),
        ),
      ],
    ),
    PlatformMenu(
      label: 'Правка',
      menus: [
        PlatformMenuItem(
          label: 'Отменить',
          shortcut: const SingleActivator(LogicalKeyboardKey.keyZ, meta: true),
          onSelected: () => undo(),
        ),
      ],
    ),
  ],
  body: const MyMainContent(),
)
```

---

## 4. Drag & Drop

```yaml
dependencies:
  desktop_drop: ^0.4.4
```

```dart
import 'package:desktop_drop/desktop_drop.dart';
import 'package:cross_file/cross_file.dart';

class DropZone extends StatefulWidget {
  const DropZone({super.key});

  @override
  State<DropZone> createState() => _DropZoneState();
}

class _DropZoneState extends State<DropZone> {
  bool _isDragging = false;
  List<XFile> _droppedFiles = [];

  @override
  Widget build(BuildContext context) {
    return DropTarget(
      onDragDone: (detail) {
        setState(() {
          _droppedFiles = detail.files;
          _isDragging = false;
        });
        for (final file in detail.files) {
          print('Dropped: ${file.path}');
        }
      },
      onDragEntered: (_) => setState(() => _isDragging = true),
      onDragExited: (_) => setState(() => _isDragging = false),
      child: Container(
        decoration: BoxDecoration(
          border: Border.all(
            color: _isDragging ? Colors.blue : Colors.grey,
            width: 2,
          ),
          borderRadius: BorderRadius.circular(12),
        ),
        child: _droppedFiles.isEmpty
            ? const Center(child: Text('Перетащите файлы сюда'))
            : ListView(
                children: _droppedFiles
                    .map((f) => ListTile(title: Text(f.name)))
                    .toList(),
              ),
      ),
    );
  }
}
```

---

## 5. Системный трей

```yaml
dependencies:
  tray_manager: ^0.2.3
```

```dart
import 'package:tray_manager/tray_manager.dart';

class MyApp extends StatefulWidget with TrayListener {
  // ...

  Future<void> _initTray() async {
    await trayManager.setIcon('assets/tray_icon.png');
    await trayManager.setToolTip('My App');
    await trayManager.setContextMenu(Menu(
      items: [
        MenuItem(key: 'show', label: 'Показать'),
        MenuItem.separator(),
        MenuItem(key: 'quit', label: 'Выйти'),
      ],
    ));
    trayManager.addListener(this);
  }

  @override
  void onTrayIconMouseDown() {
    windowManager.show();
  }

  @override
  void onTrayMenuItemClick(MenuItem menuItem) {
    switch (menuItem.key) {
      case 'show':
        windowManager.show();
      case 'quit':
        windowManager.close();
    }
  }
}
```

---

## 6. Клавиатурные сокращения

```dart
// Глобальные горячие клавиши
Shortcuts(
  shortcuts: {
    LogicalKeySet(LogicalKeyboardKey.control, LogicalKeyboardKey.keyS):
        const SaveIntent(),
    LogicalKeySet(LogicalKeyboardKey.control, LogicalKeyboardKey.keyZ):
        const UndoIntent(),
  },
  child: Actions(
    actions: {
      SaveIntent: CallbackAction<SaveIntent>(
        onInvoke: (_) => saveDocument(),
      ),
      UndoIntent: CallbackAction<UndoIntent>(
        onInvoke: (_) => undoLastAction(),
      ),
    },
    child: const MyEditor(),
  ),
)

// macOS: meta (Command) вместо control
// Windows/Linux: control
final isMac = Platform.isMacOS;
final modifier = isMac ? LogicalKeyboardKey.meta : LogicalKeyboardKey.control;
```

---

## 7. Диалог выбора файла

```yaml
dependencies:
  file_picker: ^8.0.0
```

```dart
import 'package:file_picker/file_picker.dart';

// Выбор файла
final result = await FilePicker.platform.pickFiles(
  type: FileType.custom,
  allowedExtensions: ['pdf', 'docx', 'txt'],
  allowMultiple: true,
);

if (result != null) {
  for (final file in result.files) {
    print('${file.name}: ${file.path}');
  }
}

// Выбор директории
final directory = await FilePicker.platform.getDirectoryPath();
print('Выбрана папка: $directory');

// Сохранение файла
final outputPath = await FilePicker.platform.saveFile(
  dialogTitle: 'Сохранить как',
  fileName: 'document.pdf',
  allowedExtensions: ['pdf'],
);
```

---

## 8. Упаковка и дистрибуция

### Windows — MSIX пакет

```yaml
dev_dependencies:
  msix: ^3.8.0
```

```yaml
# pubspec.yaml
msix_config:
  display_name: My App
  publisher_display_name: My Company
  identity_name: MyCompany.MyApp
  publisher: CN=MyCompany
  logo_path: assets/logo.png
  start_menu_icon_path: assets/logo.png
  icons_background_color: transparent
```

```bash
flutter pub run msix:create
```

### macOS — DMG

```bash
flutter build macos --release
# Открыть Xcode → Product → Archive → Distribute App

# Через create-dmg
npm install -g create-dmg
create-dmg build/macos/Build/Products/Release/MyApp.app
```

### Linux — AppImage

```bash
flutter build linux --release
# Собрать AppImage через appimagetool
```

---

## 9. Типичные ошибки

| Ошибка                           | Причина                         | Решение                                    |
| -------------------------------- | ------------------------------- | ------------------------------------------ |
| Пакет не работает на Desktop     | Platform-specific зависимость   | Проверить pub.dev поддерживаемые платформы |
| Окно слишком маленькое           | Нет минимального размера        | `minimumSize` в `window_manager`           |
| Контекстное меню не показывается | macOS требует `PlatformMenuBar` | Добавить в корень виджет-дерева            |
| Приложение не подписано (macOS)  | Нет Developer Certificate       | Подписать или запустить без Gatekeeper     |

---

## 10. Рекомендации

1. **`window_manager`** — де-факто стандарт для управления окном.
2. **Keyboard shortcuts** обязательны для Desktop-UX — мышь + клавиатура норма.
3. **Drag & Drop** — ожидаемое поведение для Desktop-приложений (файловый менеджер, редактор).
4. **Адаптируй UI** — на Desktop больше места: sidebar вместо BottomNavigationBar.
5. **Тестируй на каждой ОС** — шрифты, отступы, менеджеры окон ведут себя по-разному.
