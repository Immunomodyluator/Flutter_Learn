# 18.4 Безопасность Flutter-приложений

## 1. Суть

Мобильное приложение — постоянная цель атак: реверс-инжиниринг APK, перехват трафика, кража секретов из памяти. Защита строится на нескольких уровнях: обфускация кода, безопасное хранение, защита сети.

---

## 2. Обфускация и минификация

```bash
# Android release с обфускацией
flutter build apk --release --obfuscate --split-debug-info=./debug_symbols/

# iOS release с обфускацией
flutter build ipa --release --obfuscate --split-debug-info=./debug_symbols/

# --split-debug-info — сохраняет символы для crashlytics
# --obfuscate — переименовывает классы и методы
```

```bash
# Сохранить символы для декодирования стектрейсов в Crashlytics
flutter symbolize --input=flutter_01.stack --debug-info=./debug_symbols/app.android-arm64.symbols
```

---

## 3. ProGuard (Android)

```
# android/app/proguard-rules.pro

# Flutter
-keep class io.flutter.** { *; }
-keep class io.flutter.plugins.** { *; }

# Firebase
-keep class com.google.firebase.** { *; }

# Вашь пакет
-keep class com.example.myapp.** { *; }

# Не обфусцировать данные Gson
-keepattributes Signature
-keepattributes *Annotation*
```

```groovy
// android/app/build.gradle
buildTypes {
    release {
        minifyEnabled true
        shrinkResources true
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                      'proguard-rules.pro'
    }
}
```

---

## 4. Безопасное хранение секретов

### flutter_secure_storage

```yaml
# pubspec.yaml
dependencies:
  flutter_secure_storage: ^9.0.0
```

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureStorage {
  final _storage = const FlutterSecureStorage(
    aOptions: AndroidOptions(encryptedSharedPreferences: true),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock_this_device,
    ),
  );

  Future<void> saveToken(String token) async {
    await _storage.write(key: 'auth_token', value: token);
  }

  Future<String?> getToken() async {
    return _storage.read(key: 'auth_token');
  }

  Future<void> deleteToken() async {
    await _storage.delete(key: 'auth_token');
  }

  Future<void> clearAll() async {
    await _storage.deleteAll();
  }
}
```

> Использует Android Keystore / iOS Keychain — секреты не хранятся в SharedPreferences.

---

## 5. Certificate Pinning (Закрепление сертификата)

Защита от MITM-атак даже при скомпрометированном CA.

```dart
import 'dart:io';
import 'package:dio/dio.dart';

class SecureHttpClient {
  late final Dio _dio;

  SecureHttpClient() {
    _dio = Dio();
    _dio.httpClientAdapter = IOHttpClientAdapter(
      createHttpClient: () {
        final client = HttpClient();
        client.badCertificateCallback = (cert, host, port) => false;
        return client;
      },
      validateCertificate: (cert, host, port) {
        if (cert == null) return false;
        // Проверить fingerprint сертификата
        const pinnedSha256 = 'ABC123...'; // SHA-256 вашего сертификата
        return cert.sha256.map((b) => b.toRadixString(16).padLeft(2, '0')).join() == pinnedSha256;
      },
    );
  }

  Dio get client => _dio;
}
```

### Через dio_certificate_pinning

```dart
import 'package:http_certificate_pinning/http_certificate_pinning.dart';

final secureDio = Dio();

// Установить pinning interceptor
secureDio.interceptors.add(
  CertificatePinningInterceptor(
    allowedSHAFingerprints: ['AB:12:CD:34:...'],
  ),
);
```

---

## 6. Root/Jailbreak Detection

```yaml
dependencies:
  flutter_jailbreak_detection: ^1.10.0
```

```dart
import 'package:flutter_jailbreak_detection/flutter_jailbreak_detection.dart';

Future<void> checkDeviceSecurity() async {
  final isJailbroken = await FlutterJailbreakDetection.jailbroken;
  final isDeveloperMode = await FlutterJailbreakDetection.developerMode;

  if (isJailbroken) {
    // Показать предупреждение или заблокировать приложение
    showDialog(
      context: context,
      barrierDismissible: false,
      builder: (_) => AlertDialog(
        title: const Text('Угроза безопасности'),
        content: const Text('Ваше устройство рутировано. Приложение заблокировано.'),
      ),
    );
  }
}
```

---

## 7. Защита от скриншотов

```dart
// Запретить скриншоты (например, на экране ввода пинкода)
import 'package:flutter_windowmanager/flutter_windowmanager.dart';

class SecureScreen extends StatefulWidget {
  @override
  State<SecureScreen> createState() => _SecureScreenState();
}

class _SecureScreenState extends State<SecureScreen> {
  @override
  void initState() {
    super.initState();
    // Android: FLAG_SECURE
    FlutterWindowManager.addFlags(FlutterWindowManager.FLAG_SECURE);
  }

  @override
  void dispose() {
    FlutterWindowManager.clearFlags(FlutterWindowManager.FLAG_SECURE);
    super.dispose();
  }
  // ...
}
```

---

## 8. Безопасная работа с данными

```dart
// ПЛОХО — логирование чувствительных данных
print('User password: $password');
debugPrint('Token: $token');

// ХОРОШО — обнулять данные после использования
String? sensitiveData = 'secret';
// ... использование ...
sensitiveData = null; // явно освободить

// ХОРОШО — не хранить пароли в состоянии
class LoginState {
  // НЕ хранить пароль в поле класса надолго
  Future<void> login(String email, String password) async {
    await _authRepo.login(email, password);
    // password вышел из scope
  }
}
```

---

## 9. Скрытие секретов из pubspec / кода

```bash
# .gitignore — никогда не коммитить
.env
secrets.json
google-services.json   # для production
GoogleService-Info.plist
```

```dart
// Получать секреты через --dart-define из CI/CD
const apiKey = String.fromEnvironment('API_KEY');

// В CI: flutter build apk --dart-define=API_KEY=${{ secrets.API_KEY }}
```

---

## 10. Сетевая безопасность (Android)

```xml
<!-- android/app/src/main/res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
  <!-- Запретить cleartext HTTP в production -->
  <base-config cleartextTrafficPermitted="false">
    <trust-anchors>
      <certificates src="system"/>
    </trust-anchors>
  </base-config>

  <!-- Разрешить только для dev -->
  <domain-config cleartextTrafficPermitted="true">
    <domain includeSubdomains="true">localhost</domain>
  </domain-config>
</network-security-config>
```

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<application
  android:networkSecurityConfig="@xml/network_security_config">
```

---

## 11. Типичные угрозы и защиты

| Угроза                             | Защита                                       |
| ---------------------------------- | -------------------------------------------- |
| Реверс APK → открытый код          | `--obfuscate` + ProGuard                     |
| Кража токенов из SharedPreferences | `flutter_secure_storage` (Keystore/Keychain) |
| MITM перехват API                  | Certificate Pinning                          |
| Запуск на рутированном устройстве  | `flutter_jailbreak_detection`                |
| Скриншот секретного экрана         | `FLAG_SECURE` (Android)                      |
| API-ключи в коде                   | `--dart-define` + CI/CD secrets              |
| Логи с секретами                   | Отключать `debugPrint` в release             |

---

## 12. Рекомендации

1. **Минимум данных** — не хранить то, что не нужно. Удалять после использования.
2. **HTTPS везде** — `cleartextTrafficPermitted: false` в production.
3. **Rotate ключи** — API-ключи должны иметь ограниченный срок действия.
4. **Не доверять клиенту** — важная бизнес-логика и проверки на сервере.
5. **OWASP Mobile Top 10** — регулярно проверять на соответствие.
