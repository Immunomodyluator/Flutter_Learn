# Квиз: Firebase Authentication

**Тема:** 10.1 — Firebase Auth  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как инициализировать Firebase в Flutter приложении?

- A) `Firebase.start()` в `main()`
- B) `await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform)` + `WidgetsFlutterBinding.ensureInitialized()`
- C) `FirebaseApp.init(config)`
- D) Автоматически при первом использовании

<details>
<summary>Ответ</summary>

**B) `Firebase.initializeApp()` после `WidgetsFlutterBinding.ensureInitialized()`**

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform, // из firebase_options.dart
  );
  runApp(const MyApp());
}
```

`firebase_options.dart` генерируется через `flutterfire configure` (FlutterFire CLI).

</details>

---

### Вопрос 2 🟢

Как зарегистрировать нового пользователя по email/password?

- A) `FirebaseAuth.register(email, password)`
- B) `await FirebaseAuth.instance.createUserWithEmailAndPassword(email: email, password: password)`
- C) `await Auth.signUp(email, password)`
- D) `FirebaseAuth.newUser(email, password)`

<details>
<summary>Ответ</summary>

**B) `createUserWithEmailAndPassword()`**

```dart
try {
  final credential = await FirebaseAuth.instance
      .createUserWithEmailAndPassword(
        email: 'user@example.com',
        password: 'SecurePass123!',
      );
  final user = credential.user!;
  print('Created: ${user.uid}');
} on FirebaseAuthException catch (e) {
  if (e.code == 'weak-password') print('Слабый пароль');
  if (e.code == 'email-already-in-use') print('Email занят');
}
```

</details>

---

### Вопрос 3 🟢

Как подписаться на изменения состояния аутентификации (auth state)?

- A) `FirebaseAuth.instance.currentUser` (polling)
- B) `FirebaseAuth.instance.authStateChanges()` — возвращает `Stream<User?>`
- C) `FirebaseAuth.onAuthChange.listen(callback)`
- D) `StreamBuilder` автоматически получает auth state

<details>
<summary>Ответ</summary>

**B) `authStateChanges()` — Stream<User?>**

```dart
StreamBuilder<User?>(
  stream: FirebaseAuth.instance.authStateChanges(),
  builder: (ctx, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return SplashScreen();
    }
    if (snapshot.hasData) {
      return HomeScreen(); // вошёл
    }
    return LoginScreen(); // не вошёл
  },
)
```

`idTokenChanges()` — более широкий: срабатывает при обновлении токена.

</details>

---

### Вопрос 4 🟡

Как реализовать вход через Google?

- A) `GoogleAuth.signIn()`
- B) `google_sign_in` пакет → `GoogleSignIn().signIn()` → получить `GoogleAuthCredential` → `signInWithCredential()`
- C) `FirebaseAuth.instance.signInWithGoogle()`
- D) `OAuth.google(clientId)`

<details>
<summary>Ответ</summary>

**B) `google_sign_in` → `GoogleAuthProvider.credential()` → `signInWithCredential()`**

```dart
Future<UserCredential> signInWithGoogle() async {
  final googleUser = await GoogleSignIn().signIn();
  final googleAuth = await googleUser!.authentication;

  final credential = GoogleAuthProvider.credential(
    accessToken: googleAuth.accessToken,
    idToken: googleAuth.idToken,
  );

  return await FirebaseAuth.instance.signInWithCredential(credential);
}
```

</details>

---

### Вопрос 5 🟡

Как верифицировать email после регистрации?

- A) Автоматически — Firebase верифицирует при регистрации
- B) `await user.sendEmailVerification()` — отправить письмо; `user.reload()` + `user.emailVerified` — проверить
- C) `FirebaseAuth.verify(user.email)`
- D) `user.requestVerification(callback)`

<details>
<summary>Ответ</summary>

**B) `sendEmailVerification()` + проверка `emailVerified` после `reload()`**

```dart
final user = FirebaseAuth.instance.currentUser!;

// Отправить письмо верификации:
await user.sendEmailVerification(
  ActionCodeSettings(url: 'https://myapp.page.link/verify'),
);

// Проверить (нужно reload, т.к. кэшируется):
await user.reload();
if (FirebaseAuth.instance.currentUser!.emailVerified) {
  print('Email подтверждён!');
}
```

Ограничение: 1 письмо в 60 секунд.

</details>

---

### Вопрос 6 🟡

Как обновить профиль пользователя (displayName, photoURL)?

- A) `user.name = 'Alice'`
- B) `await user.updateDisplayName('Alice')` и `await user.updatePhotoURL(url)`; или `user.updateProfile(displayName: 'Alice', photoURL: url)`
- C) `FirebaseAuth.setProfile(uid, data)`
- D) Только через Firebase Console

<details>
<summary>Ответ</summary>

**B) `updateDisplayName()` / `updatePhotoURL()` / `updateProfile()`**

```dart
final user = FirebaseAuth.instance.currentUser!;

// Обновить displayName:
await user.updateDisplayName('Alice Smith');

// Обновить photoURL:
await user.updatePhotoURL('https://example.com/photo.jpg');

// Или вместе (устаревший API, но работает):
await user.updateProfile(
  displayName: 'Alice Smith',
  photoURL: 'https://example.com/photo.jpg',
);

// Обновить email (требует reauthentication):
await user.verifyBeforeUpdateEmail('new@example.com');
```

</details>

---

### Вопрос 7 🟡

Как реализовать анонимную аутентификацию и потом связать с permanent аккаунтом?

- A) Нельзя конвертировать анонимный аккаунт
- B) `signInAnonymously()` → аноним; затем `linkWithCredential()` для привязки email/Google — данные сохраняются
- C) `AnonymousUser.upgrade(email, password)`
- D) Создать новый аккаунт и перенести данные вручную

<details>
<summary>Ответ</summary>

**B) `signInAnonymously()` → `linkWithCredential()`**

```dart
// Анонимный вход:
final anonCredential = await FirebaseAuth.instance.signInAnonymously();
print('Anon UID: ${anonCredential.user!.uid}');

// Позже привязать email:
final emailCredential = EmailAuthProvider.credential(
  email: 'user@example.com',
  password: 'password',
);
try {
  final linked = await FirebaseAuth.instance.currentUser!
      .linkWithCredential(emailCredential);
  print('Linked! UID: ${linked.user!.uid}'); // тот же UID!
} on FirebaseAuthException catch (e) {
  if (e.code == 'credential-already-in-use') { ... }
}
```

</details>

---

### Вопрос 8 🔴

Как реализовать reauthentication перед выполнением чувствительных операций?

- A) Не нужно — Firebase разрешает всё аутентифицированным пользователям
- B) `user.reauthenticateWithCredential(credential)` — Firebase требует повторной аутентификации для удаления аккаунта, смены email/password
- C) `FirebaseAuth.relogin(user)`
- D) Проверить `token.expiresAt < DateTime.now()`

<details>
<summary>Ответ</summary>

**B) `reauthenticateWithCredential()` перед sensitive операциями**

```dart
Future<void> deleteAccount(String password) async {
  final user = FirebaseAuth.instance.currentUser!;

  // Обязательно reauthenticate:
  final credential = EmailAuthProvider.credential(
    email: user.email!,
    password: password,
  );

  try {
    await user.reauthenticateWithCredential(credential);
    await user.delete(); // теперь можно удалить
  } on FirebaseAuthException catch (e) {
    if (e.code == 'wrong-password') throw WrongPasswordException();
    if (e.code == 'requires-recent-login') { ... }
  }
}
```

</details>

---

### Вопрос 9 🔴

Как настроить Custom Claims и использовать их для ролевого доступа?

- A) `user.setClaim('role', 'admin')`
- B) Custom Claims устанавливаются только через Firebase Admin SDK (сервер/Cloud Functions); в клиенте доступны через `user.getIdTokenResult()`
- C) В Firestore document `/users/{uid}.claims`
- D) Через Firebase Auth console для каждого пользователя

<details>
<summary>Ответ</summary>

**B) Admin SDK (сервер) устанавливает; клиент читает через `getIdTokenResult()`**

```javascript
// Cloud Function (Node.js):
await admin.auth().setCustomUserClaims(uid, { role: "admin" });
```

```dart
// Flutter (клиент):
final idTokenResult = await FirebaseAuth.instance.currentUser!
    .getIdTokenResult(forceRefresh: true);
final role = idTokenResult.claims?['role'];
if (role == 'admin') showAdminPanel();
```

Firestore Security Rules:

```javascript
allow write: if request.auth.token.role == 'admin';
```

</details>

---

### Вопрос 10 🔴

Как безопасно реализовать выход из всех устройств (revoke tokens)?

- A) `FirebaseAuth.instance.signOut()` — автоматически выходит со всех устройств
- B) `signOut()` только на текущем устройстве; для всех — Admin SDK `revokeRefreshTokens(uid)` + обработка в клиенте
- C) `user.invalidateAllSessions()`
- D) Смена пароля автоматически отзывает все токены

<details>
<summary>Ответ</summary>

**B) `signOut()` — только текущее; отзыв всех — Admin SDK**

```javascript
// Cloud Function:
exports.revokeAllTokens = onCall(async (request) => {
  const uid = request.auth.uid;
  await admin.auth().revokeRefreshTokens(uid);
  const user = await admin.auth().getUser(uid);
  console.log("Tokens revoked at:", user.tokensValidAfterTime);
});
```

```dart
// Клиент должен обрабатывать revoked token:
FirebaseAuth.instance.idTokenChanges().listen((user) async {
  if (user != null) {
    final result = await user.getIdTokenResult();
    // Если токен отозван — сервер вернёт 401
  }
});
```

Revoke влияет только на refresh tokens — access токены живут ещё ~1 час.

</details>
