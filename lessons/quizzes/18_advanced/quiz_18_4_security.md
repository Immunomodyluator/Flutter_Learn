# Квиз: Security

**Тема:** 18.4 — Mobile App Security  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как безопасно хранить токены и пароли в Flutter?

- A) `SharedPreferences` достаточно
- B) `flutter_secure_storage` — использует Keychain (iOS) и EncryptedSharedPreferences (Android) для защищённого хранения
- C) Хранить в `dart-define` константах
- D) `SecureStorage` встроен во Flutter

<details>
<summary>Ответ</summary>

**B) `flutter_secure_storage` — нативное защищённое хранилище**

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureTokenStorage {
  static const _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
    ),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock,
    ),
  );

  static Future<void> saveToken(String token) async {
    await _storage.write(key: 'auth_token', value: token);
  }

  static Future<String?> getToken() async {
    return _storage.read(key: 'auth_token');
  }

  static Future<void> deleteToken() async {
    await _storage.delete(key: 'auth_token');
  }

  // При logout — очистить всё:
  static Future<void> clearAll() async {
    await _storage.deleteAll();
  }
}
```

</details>

---

### Вопрос 2 🟢

Что такое обфускация кода и как её включить во Flutter?

- A) Обфускация — шифрование данных пользователя
- B) `--obfuscate --split-debug-info=<dir>` при сборке — переименовывает классы/методы, затрудняя реверс-инжиниринг
- C) `flutter build apk --minify`
- D) Обфускация включена по умолчанию

<details>
<summary>Ответ</summary>

**B) `--obfuscate --split-debug-info`**

```bash
# Android:
flutter build apk --release \
  --obfuscate \
  --split-debug-info=debug_info/android/

# iOS:
flutter build ios --release \
  --obfuscate \
  --split-debug-info=debug_info/ios/

# AAB:
flutter build appbundle --release \
  --obfuscate \
  --split-debug-info=debug_info/

# ВАЖНО: сохранить файлы из debug_info/
# Они нужны для symbolication crash reports!
# Хранить в надёжном месте (не в Git, но и не потерять)

# Deobfuscate stack trace:
flutter symbolize \
  -i crash_stack_trace.txt \
  -d debug_info/android/app.android-arm64.symbols
```

</details>

---

### Вопрос 3 🟢

Как реализовать биометрическую аутентификацию в Flutter?

- A) `flutter_biometrics` пакет
- B) `local_auth` пакет — `authenticate()` метод с поддержкой Face ID/Touch ID/Fingerprint
- C) Только через platform channel
- D) Биометрия доступна только в iOS

<details>
<summary>Ответ</summary>

**B) `local_auth` пакет**

```dart
import 'package:local_auth/local_auth.dart';

class BiometricService {
  final _auth = LocalAuthentication();

  Future<bool> isBiometricAvailable() async {
    final canCheck = await _auth.canCheckBiometrics;
    final isDeviceSupported = await _auth.isDeviceSupported();
    return canCheck && isDeviceSupported;
  }

  Future<List<BiometricType>> getAvailableBiometrics() async {
    return _auth.getAvailableBiometrics();
    // [BiometricType.face, BiometricType.fingerprint, ...]
  }

  Future<bool> authenticate() async {
    try {
      return await _auth.authenticate(
        localizedReason: 'Подтвердите вашу личность',
        options: const AuthenticationOptions(
          biometricOnly: false,  // разрешить PIN как fallback
          stickyAuth: true,       // не отменять при уходе из приложения
        ),
      );
    } on PlatformException catch (e) {
      return false;
    }
  }
}
```

</details>

---

### Вопрос 4 🟡

Что такое Certificate Pinning и как его реализовать в Flutter?

- A) Хранение сертификатов в SharedPreferences
- B) Проверка сертификата сервера в приложении — защита от MITM атак; через кастомный `SecurityContext` или `http_certificate_pinning` пакет
- C) Certificate Pinning только для HTTPS
- D) `CertificatePinning.pin(url)` встроенный метод

<details>
<summary>Ответ</summary>

**B) Проверка fingerprint сертификата сервера**

```dart
import 'dart:io';
import 'package:dio/dio.dart';

// Вариант 1: кастомный HttpClient с pinning:
HttpClient createPinnedHttpClient() {
  final context = SecurityContext();
  // Добавить сертификат сервера:
  context.setTrustedCertificatesBytes(
    (await rootBundle.load('assets/certs/server.cer')).buffer.asUint8List(),
  );
  return HttpClient(context: context);
}

// Вариант 2: Dio с кастомным адаптером:
class PinnedHttpAdapter extends HttpClientAdapter {
  @override
  Future<ResponseBody> fetch(RequestOptions options,
      Stream<Uint8List>? requestStream, Future<void>? cancelFuture) async {
    final client = createPinnedHttpClient();
    // ...
  }
}

// Вариант 3: http_certificate_pinning пакет:
// await HttpCertificatePinning.check(
//   serverURL: 'https://api.example.com',
//   headerHttp: {},
//   sha: SHA.SHA256,
//   allowedSHAFingerprints: ['expected:fingerprint:here'],
//   timeout: 50,
// );
```

</details>

---

### Вопрос 5 🟡

Как обнаружить root/jailbreak на устройстве пользователя?

- A) `Platform.isRooted` — встроенная проверка
- B) `flutter_jailbreak_detection` пакет — `JailbreakDetection.jailbroken` (iOS) / `JailbreakDetection.developerMode` (Android)
- C) Проверить наличие файла `/system/xbin/su`
- D) Root detection только на Android

<details>
<summary>Ответ</summary>

**B) `flutter_jailbreak_detection` пакет**

```dart
import 'package:flutter_jailbreak_detection/flutter_jailbreak_detection.dart';

class SecurityCheck {
  static Future<bool> isDeviceCompromised() async {
    try {
      final isJailbroken = await FlutterJailbreakDetection.jailbroken;
      final isDeveloperMode = await FlutterJailbreakDetection.developerMode;

      return isJailbroken || isDeveloperMode;
    } catch (e) {
      // При ошибке — считать устройство безопасным
      return false;
    }
  }

  static Future<void> checkAndExit() async {
    if (await isDeviceCompromised()) {
      // Показать предупреждение и выйти:
      showDialog(
        context: navigatorKey.currentContext!,
        barrierDismissible: false,
        builder: (_) => AlertDialog(
          title: const Text('Безопасность'),
          content: const Text('Ваше устройство скомпрометировано. '
              'Использование приложения невозможно.'),
          actions: [TextButton(
            onPressed: () => exit(0),
            child: const Text('Понятно'),
          )],
        ),
      );
    }
  }
}
```

</details>

---

### Вопрос 6 🟡

Как защититься от перехвата трафика (Man-in-the-Middle атаки)?

- A) Использовать только HTTP (HTTPS — ложная безопасность)
- B) HTTPS + Certificate Pinning + Network Security Config (Android) + App Transport Security (iOS)
- C) VPN автоматически защищает от MITM
- D) `flutter run --secure` включает защиту

<details>
<summary>Ответ</summary>

**B) HTTPS + Pinning + нативные политики безопасности**

```xml
<!-- Android: res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2026-01-01">
            <pin digest="SHA-256">BASE64_SHA256_OF_PUBLIC_KEY=</pin>
            <!-- Резервный pin: -->
            <pin digest="SHA-256">BACKUP_BASE64_SHA256=</pin>
        </pin-set>
    </domain-config>
    <!-- Запретить cleartext (HTTP): -->
    <base-config cleartextTrafficPermitted="false"/>
</network-security-config>
```

```xml
<!-- AndroidManifest.xml: -->
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ...>
```

```xml
<!-- iOS Info.plist (App Transport Security): -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
</dict>
```

</details>

---

### Вопрос 7 🟡

Как предотвратить скриншоты конфиденциального контента в Flutter?

- A) Flutter автоматически защищает конфиденциальный контент
- B) Android: `FLAG_SECURE` через platform channel; iOS: скрыть окно при переходе в background; `prevent_screen_capture` пакет
- C) Скриншоты нельзя предотвратить
- D) `SecureScreen()` виджет

<details>
<summary>Ответ</summary>

**B) `FLAG_SECURE` (Android) + iOS background blur**

```dart
// prevent_screen_capture пакет:
import 'package:prevent_screen_capture/prevent_screen_capture.dart';

class SecureScreen extends StatefulWidget {
  @override
  State<SecureScreen> createState() => _SecureScreenState();
}

class _SecureScreenState extends State<SecureScreen> {
  @override
  void initState() {
    super.initState();
    PreventScreenCapture.preventScreenCapture();
  }

  @override
  void dispose() {
    PreventScreenCapture.allowScreenCapture();
    super.dispose();
  }
}

// iOS — скрыть контент при переходе в фон:
class _AppState extends State<App> with WidgetsBindingObserver {
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.inactive) {
      // Показать blur overlay
    } else if (state == AppLifecycleState.resumed) {
      // Убрать blur overlay
    }
  }
}
```

</details>

---

### Вопрос 8 🔴

Как реализовать защиту от реверс-инжиниринга API ключей?

- A) Хранить API ключи в dart-define — безопасно
- B) Не хранить секреты в клиентском коде; все секретные операции делать на сервере; использовать токены с ограниченными правами
- C) Шифровать ключи алгоритмом AES в коде
- D) API ключи в `flutter_secure_storage` нельзя извлечь

<details>
<summary>Ответ</summary>

**B) Секреты — только на сервере**

```
НЕПРАВИЛЬНО — секрет в клиентском коде:
dart-define=STRIPE_SECRET_KEY=sk_live_...  ← можно достать из APK!
flutter_secure_storage: можно прочитать на rooted устройстве

ПРАВИЛЬНО:
1. Серверный прокси:
   Flutter → POST /api/payment → Сервер → Stripe API (с секретным ключом)
   Flutter никогда не видит секретный ключ

2. Временные токены:
   Flutter → GET /api/token → Server → { publishable_key: pk_live_... }
   Публичный ключ (publishable) можно хранить в приложении

3. Certificate-based auth:
   Клиентский сертификат для доступа к API
   (сложнее, но защищает от несанкционированных клиентов)

Что МОЖНО хранить в клиенте:
✅ Google Maps publishable key
✅ Firebase config (не admin key!)
✅ Stripe publishable key (pk_*)
✅ Public API endpoints URL

Что НЕЛЬЗЯ хранить:
❌ Stripe secret key (sk_*)
❌ Database credentials
❌ Admin API keys
❌ Private keys
```

</details>

---

### Вопрос 9 🔴

Как защититься от SQL injection и XSS в Flutter приложении?

- A) Flutter автоматически защищает от SQL injection
- B) Параметризованные запросы в SQLite; экранировать HTML перед отображением в WebView; валидация входных данных
- C) Только серверная защита от SQL injection
- D) SQL injection невозможен в мобильных приложениях

<details>
<summary>Ответ</summary>

**B) Параметризованные запросы + экранирование HTML**

```dart
// SQLite — НИКОГДА не строить запросы через конкатенацию:
// ПЛОХО — SQL injection:
final result = await db.rawQuery(
  "SELECT * FROM users WHERE name = '$userInput'",
);

// ХОРОШО — параметризованный запрос:
final result = await db.query(
  'users',
  where: 'name = ?',
  whereArgs: [userInput],
);

// Или:
final result = await db.rawQuery(
  'SELECT * FROM users WHERE name = ?',
  [userInput],
);

// WebView — экранировать HTML:
import 'package:html_escape/html_escape.dart';

final escaped = const HtmlEscape().convert(userInput);
webViewController.loadHtmlString('<p>$escaped</p>');

// Валидация входных данных:
bool isValidEmail(String email) {
  return RegExp(r'^[\w.-]+@[\w.-]+\.\w+$').hasMatch(email);
}

// Sanitize перед сохранением:
String sanitizeInput(String input) {
  return input.trim().replaceAll(RegExp(r'[<>&"\'\\]'), '');
}
```

</details>

---

### Вопрос 10 🔴

Как реализовать антитамперинг (защиту от модификации APK)?

- A) Android автоматически блокирует модифицированные APK
- B) Проверка signature hash приложения; SafetyNet/Play Integrity API; runtime проверка целостности
- C) Антитамперинг только для iOS
- D) `flutter build apk --anti-tamper`

<details>
<summary>Ответ</summary>

**B) Play Integrity API + signature проверка**

```dart
// Play Integrity API (Google):
// Заменяет устаревший SafetyNet Attestation

// 1. Получить nonce от сервера
// 2. Отправить в Play Integrity API
// 3. Получить токен → отправить на сервер для верификации

// android/app/src/main/kotlin/MainActivity.kt:
// val integrityManager = IntegrityManagerFactory.create(context)
// val request = StandardIntegrityManager.StandardIntegrityTokenRequest
//   .builder().setRequestHash(nonce).build()

// Проверка подписи приложения через platform channel:
// PackageManager.getPackageInfo() → signatures → compare hash

// Dart сторона:
class IntegrityCheck {
  static const _channel = MethodChannel('app.integrity');

  static Future<bool> verifyAppIntegrity() async {
    try {
      final isValid = await _channel.invokeMethod<bool>('checkIntegrity');
      return isValid ?? false;
    } on PlatformException {
      return false;
    }
  }
}

// ВАЖНО:
// - Все проверки должны происходить на сервере
// - Клиентские проверки можно обойти на rooted устройствах
// - Defense in depth: несколько уровней защиты
```

</details>
