# 10.1: Firebase Auth — авторизация в FitMenu

> Project: FitMenu | Глава 10 — Firebase

### 10.1: AuthService с Firebase Authentication (Email/Google)

🎯 **Цель шага:** Подключить Firebase Authentication к FitMenu — реализовать регистрацию, вход через Email/Password и Google Sign-In, с экраном авторизации и редиректом в зависимости от состояния пользователя.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  firebase_core: ^2.24.0
  firebase_auth: ^4.16.0
  google_sign_in: ^6.2.0
```

Реализуй `lib/core/auth/auth_service.dart`.

**AuthService:**
```dart
class AuthService {
  final _auth = FirebaseAuth.instance;

  Stream<User?> get authStateChanges => _auth.authStateChanges();
  User? get currentUser => _auth.currentUser;
  bool get isAuthenticated => _auth.currentUser != null;

  Future<UserCredential> signInWithEmail(String email, String password) async { ... }
  Future<UserCredential> registerWithEmail(String email, String password) async { ... }
  Future<UserCredential?> signInWithGoogle() async { ... }
  Future<void> signOut() async { ... }
  Future<void> sendPasswordReset(String email) async { ... }
}
```

**AuthGate — роутинг по состоянию:**
```dart
class AuthGate extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const SplashScreen();
        }
        return snapshot.hasData ? const AppShell() : const LoginScreen();
      },
    );
  }
}
```

**LoginScreen:**
- Поля Email и Password с валидацией
- Кнопки: "Войти", "Зарегистрироваться", "Войти через Google"
- Обработка ошибок: `FirebaseAuthException` → понятные сообщения на русском

**Инициализация в main:**
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(const FitMenuApp());
}
```

---

✅ **Критерии приёмки:**
- [ ] `Firebase.initializeApp()` вызывается до `runApp()`
- [ ] `AuthGate` использует `StreamBuilder` на `authStateChanges()`
- [ ] `FirebaseAuthException.code` обрабатывается: `'user-not-found'`, `'wrong-password'`, `'email-already-in-use'`
- [ ] Индикатор загрузки отображается во время запроса
- [ ] Google Sign-In работает через `GoogleSignIn().signIn()` + `signInWithCredential()`
- [ ] `signOut()` вызывает и `FirebaseAuth.signOut()` и `GoogleSignIn.signOut()`

---

💡 **Подсказка:** `FirebaseAuthException` имеет поле `.code` со строковым кодом ошибки. Используй `switch (e.code)` для пользовательских сообщений. `authStateChanges()` — hot stream, нет нужды вручную проверять `currentUser` при каждом билде.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/auth/auth_service.dart

import 'package:firebase_auth/firebase_auth.dart';
import 'package:google_sign_in/google_sign_in.dart';

class AuthException implements Exception {
  const AuthException(this.message);
  final String message;
}

class AuthService {
  final _auth    = FirebaseAuth.instance;
  final _google  = GoogleSignIn();

  Stream<User?> get authStateChanges => _auth.authStateChanges();
  User? get currentUser => _auth.currentUser;
  bool get isAuthenticated => _auth.currentUser != null;

  Future<UserCredential> signInWithEmail(String email, String password) async {
    try {
      return await _auth.signInWithEmailAndPassword(
          email: email.trim(), password: password);
    } on FirebaseAuthException catch (e) {
      throw AuthException(_mapAuthError(e.code));
    }
  }

  Future<UserCredential> registerWithEmail(String email, String password) async {
    try {
      return await _auth.createUserWithEmailAndPassword(
          email: email.trim(), password: password);
    } on FirebaseAuthException catch (e) {
      throw AuthException(_mapAuthError(e.code));
    }
  }

  Future<UserCredential?> signInWithGoogle() async {
    final googleUser = await _google.signIn();
    if (googleUser == null) return null; // отменено пользователем

    final googleAuth = await googleUser.authentication;
    final credential = GoogleAuthProvider.credential(
      accessToken: googleAuth.accessToken,
      idToken:     googleAuth.idToken,
    );
    return _auth.signInWithCredential(credential);
  }

  Future<void> signOut() async {
    await Future.wait([_auth.signOut(), _google.signOut()]);
  }

  Future<void> sendPasswordReset(String email) async {
    await _auth.sendPasswordResetEmail(email: email.trim());
  }

  String _mapAuthError(String code) => switch (code) {
        'user-not-found'        => 'Пользователь не найден',
        'wrong-password'        => 'Неверный пароль',
        'email-already-in-use'  => 'Email уже используется',
        'invalid-email'         => 'Некорректный email',
        'weak-password'         => 'Пароль слишком простой (минимум 6 символов)',
        'too-many-requests'     => 'Слишком много попыток. Попробуйте позже',
        _                       => 'Ошибка авторизации: $code',
      };
}
```

```dart
// lib/core/auth/auth_gate.dart

import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/material.dart';

class AuthGate extends StatelessWidget {
  const AuthGate({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }
        if (snapshot.hasData) return const AppShell();
        return const LoginScreen();
      },
    );
  }
}
```

</details>
