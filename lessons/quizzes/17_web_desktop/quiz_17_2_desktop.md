# Квиз: Flutter Desktop

**Тема:** 17.2 — Flutter for Desktop  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какие десктопные платформы поддерживает Flutter?

- A) Только Windows
- B) Windows, macOS, Linux — все в stable канале
- C) macOS и Linux только в beta
- D) Desktop поддержка — только для Flutter 4+

<details>
<summary>Ответ</summary>

**B) Windows, macOS, Linux — stable**

```bash
# Проверить доступные устройства:
flutter devices

# Запустить на конкретной десктопной платформе:
flutter run -d windows
flutter run -d macos
flutter run -d linux

# Собрать для production:
flutter build windows --release
flutter build macos --release
flutter build linux --release

# Результаты:
# Windows: build/windows/x64/runner/Release/
# macOS: build/macos/Build/Products/Release/MyApp.app
# Linux: build/linux/x64/release/bundle/

# Добавить desktop поддержку к существующему проекту:
flutter create --platforms=windows,macos,linux .
```

</details>

---

### Вопрос 2 🟢

Как использовать `window_manager` пакет для управления окном?

- A) `Window.resize()` встроенный метод Flutter
- B) `window_manager` пакет — управление размером, позицией, заголовком, режимом окна
- C) Только через нативный platform channel
- D) `desktop_window` — единственный пакет для окон

<details>
<summary>Ответ</summary>

**B) `window_manager` пакет**

```dart
// pubspec.yaml: window_manager: ^0.3.0
import 'package:window_manager/window_manager.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await windowManager.ensureInitialized();

  WindowOptions windowOptions = const WindowOptions(
    size: Size(1200, 800),
    minimumSize: Size(800, 600),
    center: true,
    backgroundColor: Colors.transparent,
    skipTaskbar: false,
    titleBarStyle: TitleBarStyle.normal,
    title: 'My Flutter App',
  );

  windowManager.waitUntilReadyToShow(windowOptions, () async {
    await windowManager.show();
    await windowManager.focus();
  });

  runApp(const MyApp());
}

// В коде:
await windowManager.setFullScreen(true);
await windowManager.maximize();
await windowManager.setTitle('Новый заголовок');
final size = await windowManager.getSize();
```

</details>

---

### Вопрос 3 🟢

Как собрать Windows приложение в MSIX формат для Microsoft Store?

- A) `flutter build windows --msix`
- B) Пакет `msix` для Flutter; `flutter pub run msix:create` — создаёт .msix пакет с настройками из pubspec.yaml
- C) Только через Visual Studio
- D) MSIX не поддерживает Flutter

<details>
<summary>Ответ</summary>

**B) Пакет `msix`**

```yaml
# pubspec.yaml:
dev_dependencies:
  msix: ^3.16.0

msix_config:
  display_name: My Flutter App
  publisher_display_name: Ivan Developer
  identity_name: com.example.myapp
  msix_version: 1.0.0.0
  logo_path: assets/icons/app_icon.png
  # Для Microsoft Store:
  publisher: CN=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
  # Для sideloading (без Store):
  capabilities: 'internetClient,privateNetworkClientServer'
```

```bash
# Собрать MSIX:
flutter pub run msix:create

# Или за один шаг:
flutter build windows && flutter pub run msix:create

# Результат: build/windows/x64/runner/Release/MyApp.msix

# Установить для тестирования:
# ПКМ на .msix → Install
# Или Add-AppxPackage в PowerShell
```

</details>

---

### Вопрос 4 🟡

Как реализовать меню приложения (menu bar) в Flutter Desktop?

- A) AppBar работает как меню
- B) `PlatformMenuBar` виджет (Flutter 3+) — нативное меню macOS; `MenuBar` — кроссплатформенный виджет
- C) `MenuBar` только для Windows
- D) Нативное меню недоступно из Flutter

<details>
<summary>Ответ</summary>

**B) `PlatformMenuBar` для нативного + `MenuBar` кроссплатформенный**

```dart
// Нативное меню (macOS menu bar, Windows system menu):
PlatformMenuBar(
  menus: [
    PlatformMenu(
      label: 'File',
      menus: [
        PlatformMenuItem(
          label: 'New',
          shortcut: const SingleActivator(LogicalKeyboardKey.keyN,
              meta: true),
          onSelected: () => createNewFile(),
        ),
        const PlatformMenuDivider(),
        PlatformMenuItem(
          label: 'Exit',
          onSelected: () => windowManager.close(),
        ),
      ],
    ),
  ],
  child: const Scaffold(body: MyApp()),
)

// Кроссплатформенный MenuBar (Material 3):
MenuBar(
  children: [
    SubmenuButton(
      menuChildren: [
        MenuItemButton(
          onPressed: () => createNewFile(),
          child: const Text('New'),
        ),
      ],
      child: const Text('File'),
    ),
  ],
)
```

</details>

---

### Вопрос 5 🟡

Как получить доступ к файловой системе в Flutter Desktop?

- A) `File` из `dart:io` — только для мобильных
- B) `dart:io` полностью доступен; диалоги файлов через `file_picker` пакет; `path_provider` для стандартных директорий
- C) Только через platform channel
- D) Flutter Desktop не имеет доступа к файловой системе

<details>
<summary>Ответ</summary>

**B) `dart:io` + `file_picker` + `path_provider`**

```dart
import 'dart:io';
import 'package:file_picker/file_picker.dart';
import 'package:path_provider/path_provider.dart';

// Чтение/запись файлов:
final file = File('/Users/ivan/documents/data.json');
final content = await file.readAsString();
await file.writeAsString('{"key": "value"}');

// Диалог выбора файла:
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.custom,
  allowedExtensions: ['json', 'csv'],
  allowMultiple: false,
);

if (result != null) {
  final file = File(result.files.single.path!);
  final data = await file.readAsString();
}

// Диалог сохранения:
String? outputPath = await FilePicker.platform.saveFile(
  dialogTitle: 'Сохранить как',
  fileName: 'export.csv',
  type: FileType.custom,
  allowedExtensions: ['csv'],
);

// Стандартные директории:
final docs = await getApplicationDocumentsDirectory();
final temp = await getTemporaryDirectory();
final support = await getApplicationSupportDirectory(); // Desktop
```

</details>

---

### Вопрос 6 🟡

Как настроить поддержку drag & drop файлов в Flutter Desktop?

- A) Drag & drop недоступен в Flutter
- B) `desktop_drop` пакет — `DropTarget` виджет с `onDragDone` callback; или `DragTarget` для внутреннего DnD
- C) Встроенный `DragTarget` поддерживает файлы
- D) Только через web iframe

<details>
<summary>Ответ</summary>

**B) `desktop_drop` пакет**

```dart
// pubspec.yaml: desktop_drop: ^0.4.0
import 'package:desktop_drop/desktop_drop.dart';

class DropZone extends StatefulWidget {
  @override
  State<DropZone> createState() => _DropZoneState();
}

class _DropZoneState extends State<DropZone> {
  bool _dragging = false;
  List<String> _droppedFiles = [];

  @override
  Widget build(BuildContext context) {
    return DropTarget(
      onDragEntered: (_) => setState(() => _dragging = true),
      onDragExited: (_) => setState(() => _dragging = false),
      onDragDone: (detail) {
        setState(() {
          _dragging = false;
          _droppedFiles = detail.files.map((f) => f.path).toList();
        });
        // Обработать файлы:
        for (final file in detail.files) {
          processFile(File(file.path));
        }
      },
      child: Container(
        color: _dragging ? Colors.blue.withOpacity(0.2) : Colors.grey[100],
        child: const Center(child: Text('Перетащите файлы сюда')),
      ),
    );
  }
}
```

</details>

---

### Вопрос 7 🟡

Как реализовать system tray (значок в трее) для Flutter Desktop?

- A) System tray недоступен в Flutter
- B) `tray_manager` пакет — инициализация, иконка, контекстное меню, клик-события
- C) Только для Windows через native plugin
- D) `SystemTray` встроенный класс Flutter

<details>
<summary>Ответ</summary>

**B) `tray_manager` пакет**

```dart
// pubspec.yaml: tray_manager: ^0.2.0
import 'package:tray_manager/tray_manager.dart';

class MyApp extends StatefulWidget {
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> with TrayListener {
  @override
  void initState() {
    super.initState();
    trayManager.addListener(this);
    _initTray();
  }

  Future<void> _initTray() async {
    await trayManager.setIcon(
      Platform.isWindows ? 'assets/icons/tray.ico' : 'assets/icons/tray.png',
    );
    await trayManager.setContextMenu(Menu(
      items: [
        MenuItem(label: 'Показать', onClick: (_) => windowManager.show()),
        MenuItem.separator(),
        MenuItem(label: 'Выйти', onClick: (_) => windowManager.destroy()),
      ],
    ));
  }

  @override
  void onTrayIconMouseDown() => windowManager.show();

  @override
  void dispose() {
    trayManager.removeListener(this);
    super.dispose();
  }
}
```

</details>

---

### Вопрос 8 🔴

Как подписать macOS приложение для распространения?

- A) macOS не требует подписи
- B) Требуются Developer ID Application certificate + Notarization через `xcrun notarytool`; или App Store Distribution через Xcode
- C) `flutter build macos --sign certificate.p12`
- D) Только через Apple Developer Program Enterprise

<details>
<summary>Ответ</summary>

**B) Developer ID + Notarization для распространения вне App Store**

```bash
# 1. Получить Developer ID Application certificate в Xcode
# 2. Собрать подписанное приложение:
flutter build macos --release

# 3. Нотаризация через notarytool (Xcode 13+):
xcrun notarytool submit \
  build/macos/Build/Products/Release/MyApp.app \
  --apple-id "developer@example.com" \
  --team-id "XXXXXXXXXX" \
  --password "@keychain:notarytool-password" \
  --wait

# 4. Staple (встроить нотаризацию):
xcrun stapler staple build/macos/Build/Products/Release/MyApp.app

# 5. Создать DMG для распространения:
create-dmg \
  --volname "MyApp Installer" \
  --window-size 800 400 \
  --icon-size 100 \
  "MyApp.dmg" \
  "build/macos/Build/Products/Release/MyApp.app"
```

</details>

---

### Вопрос 9 🔴

Как реализовать автоматическое обновление приложения в Flutter Desktop?

- A) Desktop приложения не поддерживают обновления
- B) `upgrader` пакет или `auto_updater` (Squirrel/Sparkle); проверка версии через API; скачать и установить новый инсталлятор
- C) Только через магазины приложений
- D) `FlutterAutoUpdater.check()`

<details>
<summary>Ответ</summary>

**B) `auto_updater` или `upgrader` пакет**

```dart
// auto_updater пакет (использует Squirrel.Windows + Sparkle для macOS):
import 'package:auto_updater/auto_updater.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await windowManager.ensureInitialized();
  await autoUpdater.ensureInitialized();

  // appcast URL (XML с информацией о версиях):
  String feedURL = 'https://myserver.com/appcast.xml';
  await autoUpdater.setFeedURL(feedURL);
  await autoUpdater.checkForUpdates();

  runApp(const MyApp());
}

// appcast.xml:
// <?xml version="1.0" encoding="utf-8"?>
// <rss version="2.0">
//   <channel>
//     <item>
//       <title>Version 2.0.0</title>
//       <enclosure url="https://myserver.com/MyApp-2.0.0.dmg"
//                  sparkle:version="2.0.0"
//                  type="application/octet-stream"/>
//     </item>
//   </channel>
// </rss>
```

</details>

---

### Вопрос 10 🔴

Как оптимизировать размер Flutter Desktop приложения?

- A) Размер Desktop приложений нельзя уменьшить
- B) `--tree-shake-icons`, деферированная загрузка, MSIX сжатие, AppX bundling, исключение ненужных assets
- C) Использовать debug сборку — она меньше
- D) `flutter build windows --compress`

<details>
<summary>Ответ</summary>

**B) Tree-shaking + оптимизация assets**

```bash
# Windows release с оптимизациями:
flutter build windows --release \
  --tree-shake-icons \
  --obfuscate \
  --split-debug-info=debug_info/

# Типичные размеры (Windows):
# Без оптимизаций: ~50-100 MB (с flutter engine)
# После MSIX упаковки: ~15-30 MB
# С UPX сжатием exe (осторожно — может вызвать антивирус!): -30%

# macOS App Bundle:
# Без оптимизаций: ~80-120 MB
# После сжатия в DMG (zlib): ~25-40 MB

# Что занимает место:
# - flutter_engine.dll / FlutterEmbedderGlfw.so: ~20 MB
# - dart runtime: ~5 MB
# - app assets: зависит от приложения

# Практика: исключать неиспользуемые fonts, images:
# Проверить через flutter build windows --analyze-size
```

</details>
