# 10.1 Firebase Authentication

## 1. Суть

`firebase_auth` — готовая система аутентификации. Поддерживает Email/Password, Google, Apple, GitHub, Phone. Хранит сессию автоматически — пользователь остаётся залогиненным после перезапуска.

```yaml
dependencies:
  firebase_auth: ^4.17.9
  google_sign_in: ^6.2.1 # для Google-входа
```

---

## 2. Основной синтаксис

```dart
import 'package:firebase_auth/firebase_auth.dart';

final auth = FirebaseAuth.instance;

// Текущий пользователь (null если не залогинен)
final user = auth.currentUser;
print(user?.uid);         // уникальный ID
print(user?.email);       // email
print(user?.displayName); // имя
print(user?.photoURL);    // фото URL

// Поток состояния авторизации
auth.authStateChanges().listen((User? user) {
  if (user == null) {
    // не залогинен
  } else {
    // залогинен
  }
});

// Email / Password
final credential = await auth.signInWithEmailAndPassword(
  email: 'ivan@example.com',
  password: 'securePassword123',
);

// Регистрация
final credential = await auth.createUserWithEmailAndPassword(
  email: 'ivan@example.com',
  password: 'securePassword123',
);

// Выход
await auth.signOut();

// Сброс пароля
await auth.sendPasswordResetEmail(email: 'ivan@example.com');
```

---

## 3. Google Sign In

```dart
import 'package:google_sign_in/google_sign_in.dart';
import 'package:firebase_auth/firebase_auth.dart';

Future<UserCredential?> signInWithGoogle() async {
  // 1. Показать диалог выбора Google-аккаунта
  final googleUser = await GoogleSignIn().signIn();
  if (googleUser == null) return null; // пользователь отменил

  // 2. Получить токены
  final googleAuth = await googleUser.authentication;

  // 3. Создать credential для Firebase
  final credential = GoogleAuthProvider.credential(
    accessToken: googleAuth.accessToken,
    idToken: googleAuth.idToken,
  );

  // 4. Войти в Firebase
  return FirebaseAuth.instance.signInWithCredential(credential);
}
```

---

## 4. Реальный пример — AuthRepository

```dart
import 'package:firebase_auth/firebase_auth.dart';

class AppUser {
  final String uid;
  final String? email;
  final String? displayName;
  final String? photoUrl;

  const AppUser({
    required this.uid,
    this.email,
    this.displayName,
    this.photoUrl,
  });

  factory AppUser.fromFirebase(User user) => AppUser(
        uid: user.uid,
        email: user.email,
        displayName: user.displayName,
        photoUrl: user.photoURL,
      );
}

class AuthRepository {
  final FirebaseAuth _auth;

  AuthRepository([FirebaseAuth? auth]) : _auth = auth ?? FirebaseAuth.instance;

  // Поток текущего пользователя
  Stream<AppUser?> get userStream => _auth.authStateChanges().map(
        (user) => user == null ? null : AppUser.fromFirebase(user),
      );

  // Текущий пользователь (синхронно)
  AppUser? get currentUser {
    final user = _auth.currentUser;
    return user == null ? null : AppUser.fromFirebase(user);
  }

  Future<AppUser> signIn(String email, String password) async {
    try {
      final credential = await _auth.signInWithEmailAndPassword(
        email: email.trim(),
        password: password,
      );
      return AppUser.fromFirebase(credential.user!);
    } on FirebaseAuthException catch (e) {
      throw _mapAuthError(e);
    }
  }

  Future<AppUser> register(String email, String password, String name) async {
    try {
      final credential = await _auth.createUserWithEmailAndPassword(
        email: email.trim(),
        password: password,
      );
      // Обновить имя
      await credential.user!.updateDisplayName(name);
      await credential.user!.reload();
      return AppUser.fromFirebase(_auth.currentUser!);
    } on FirebaseAuthException catch (e) {
      throw _mapAuthError(e);
    }
  }

  Future<void> signOut() => _auth.signOut();

  Future<void> resetPassword(String email) async {
    try {
      await _auth.sendPasswordResetEmail(email: email.trim());
    } on FirebaseAuthException catch (e) {
      throw _mapAuthError(e);
    }
  }

  Exception _mapAuthError(FirebaseAuthException e) {
    return switch (e.code) {
      'user-not-found' => Exception('Пользователь не найден'),
      'wrong-password' => Exception('Неверный пароль'),
      'email-already-in-use' => Exception('Email уже используется'),
      'weak-password' => Exception('Пароль слишком слабый'),
      'invalid-email' => Exception('Неверный формат email'),
      'user-disabled' => Exception('Аккаунт заблокирован'),
      'too-many-requests' => Exception('Слишком много попыток, подождите'),
      _ => Exception('Ошибка авторизации: ${e.message}'),
    };
  }
}

// --- Auth Guard в GoRouter ---
final router = GoRouter(
  redirect: (context, state) {
    final isLoggedIn = FirebaseAuth.instance.currentUser != null;
    final isAuthRoute = state.matchedLocation.startsWith('/auth');

    if (!isLoggedIn && !isAuthRoute) return '/auth/login';
    if (isLoggedIn && isAuthRoute) return '/home';
    return null;
  },
  refreshListenable: GoRouterRefreshStream(
    FirebaseAuth.instance.authStateChanges(),
  ),
  routes: [...],
);
```

---

## 5. Под капотом

- Firebase Auth хранит токен в защищённом хранилище платформы (Keychain iOS / Keystore Android).
- `authStateChanges()` — `Stream<User?>`, срабатывает при: входе, выходе, обновлении токена.
- Токен автоматически обновляется каждый час (Firebase ID Token живёт 1 час).
- `currentUser?.getIdToken()` — получить свежий JWT для отправки на бэкенд.

---

## 6. Типичные ошибки

| Ошибка                                              | Причина                                          | Решение                                                          |
| --------------------------------------------------- | ------------------------------------------------ | ---------------------------------------------------------------- |
| `FirebaseAuthException: user-not-found`             | Email не зарегистрирован                         | Показывай общее сообщение "неверные данные"                      |
| `PlatformException: sign_in_failed` (Google)        | SHA-1 не добавлен в Firebase                     | Добавь SHA-1 fingerprint в консоли                               |
| Пользователь залогинен, но `currentUser == null`    | `initializeApp()` вызван не перед использованием | `await Firebase.initializeApp()` в `main()`                      |
| `wrong-password` раскрывает информацию об аккаунтах | Enum атака                                       | Используй одно сообщение для `user-not-found` и `wrong-password` |

---

## 7. Рекомендации

1. **Оберни в `AuthRepository`** — не вызывай `FirebaseAuth.instance` напрямую из виджетов.
2. **`authStateChanges()` как источник истины** — следи за потоком, не за переменными.
3. **Не раскрывай конкретные ошибки** (`user-not-found` vs `wrong-password`) — это уязвимость перечисления аккаунтов.
4. **Верифицируй email** — `user.sendEmailVerification()` после регистрации.
5. **На бэкенде верифицируй ID Token** — `user.getIdToken()` → `FirebaseAdmin.verifyIdToken()`.
