# 18.4: Безопасность — flutter_secure_storage и pinning

> Project: FitMenu | Глава 18 — Advanced

### 18.4: Безопасное хранение токенов и certificate pinning

🎯 **Цель шага:** Защитить FitMenu от перехвата данных: хранить JWT токены в `flutter_secure_storage` (Keychain/Keystore), настроить SSL certificate pinning через Dio и скрыть контент приложения при переключении задач.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  flutter_secure_storage: ^9.0.0
  dio: ^5.4.0
```

**1. SecureTokenStorage:**
```dart
class SecureTokenStorage {
  const SecureTokenStorage(this._storage);
  final FlutterSecureStorage _storage;

  static const _accessTokenKey  = 'access_token';
  static const _refreshTokenKey = 'refresh_token';

  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      _storage.write(key: _accessTokenKey,  value: accessToken),
      _storage.write(key: _refreshTokenKey, value: refreshToken),
    ]);
  }

  Future<String?> getAccessToken()  => _storage.read(key: _accessTokenKey);
  Future<String?> getRefreshToken() => _storage.read(key: _refreshTokenKey);
  Future<void>    clearTokens()     => _storage.deleteAll();
}
```

**2. Certificate Pinning с Dio:**
```dart
class PinningDioClient {
  static Dio create() {
    final adapter = IOHttpClientAdapter()
      ..createHttpClient = () {
        final client = HttpClient();
        client.badCertificateCallback = (cert, host, port) {
          // Проверяем fingerprint сертификата
          final sha256 = _calculateSHA256(cert.der);
          return _pinnedHashes.contains(sha256);
        };
        return client;
      };

    return Dio()..httpClientAdapter = adapter;
  }

  static const _pinnedHashes = {
    'AABBCC...', // SHA-256 fingerprint продакшн сертификата
  };
}
```

**3. Скрыть контент при переключении (Android):**
```kotlin
// android/app/src/main/kotlin/.../MainActivity.kt
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    window.addFlags(WindowManager.LayoutParams.FLAG_SECURE)
}
```

**4. Биометрическая защита (опционально):**
```dart
import 'package:local_auth/local_auth.dart';

final auth = LocalAuthentication();
final authenticated = await auth.authenticate(
  localizedReason: 'Подтвердите личность для входа в FitMenu',
  options: const AuthenticationOptions(biometricOnly: false),
);
```

---

✅ **Критерии приёмки:**
- [ ] JWT токены хранятся в `flutter_secure_storage`, не в `SharedPreferences`
- [ ] `clearTokens()` вызывается при logout
- [ ] Certificate pinning отклоняет запросы к серверам с неверным сертификатом
- [ ] `FLAG_SECURE` скрывает контент в recent tasks (Android)
- [ ] Нет токенов/паролей в логах (`debugPrint`, `print`)

---

💡 **Подсказка:** `flutter_secure_storage` использует Android Keystore API и iOS Keychain. На десктопе/вебе поведение отличается — проверь платформу. `iOptions: IOSOptions(accessibility: KeychainAccessibility.firstUnlock)` — токен доступен после первой разблокировки. Certificate pinning сложно отлаживать — сохрани backup сертификат!

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/security/secure_token_storage.dart

import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureTokenStorage {
  SecureTokenStorage() : _storage = const FlutterSecureStorage(
    aOptions: AndroidOptions(encryptedSharedPreferences: true),
    iOptions: IOSOptions(accessibility: KeychainAccessibility.firstUnlock),
  );

  final FlutterSecureStorage _storage;

  static const _kAccessToken  = 'fit_access_token';
  static const _kRefreshToken = 'fit_refresh_token';
  static const _kUserId       = 'fit_user_id';

  Future<void> saveSession({
    required String accessToken,
    required String refreshToken,
    required String userId,
  }) async {
    await Future.wait([
      _storage.write(key: _kAccessToken,  value: accessToken),
      _storage.write(key: _kRefreshToken, value: refreshToken),
      _storage.write(key: _kUserId,       value: userId),
    ]);
  }

  Future<({String? accessToken, String? refreshToken, String? userId})>
      loadSession() async {
    final results = await Future.wait([
      _storage.read(key: _kAccessToken),
      _storage.read(key: _kRefreshToken),
      _storage.read(key: _kUserId),
    ]);
    return (
      accessToken:  results[0],
      refreshToken: results[1],
      userId:       results[2],
    );
  }

  Future<void> clearSession() => _storage.deleteAll();
}
```

```dart
// lib/core/network/auth_interceptor.dart

class AuthInterceptor extends Interceptor {
  AuthInterceptor(this._storage);
  final SecureTokenStorage _storage;

  @override
  Future<void> onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final session = await _storage.loadSession();
    if (session.accessToken != null) {
      options.headers['Authorization'] = 'Bearer ${session.accessToken}';
    }
    handler.next(options);
  }
}
```

</details>
