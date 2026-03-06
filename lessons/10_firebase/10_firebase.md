# 10. Firebase во Flutter

## 1. Что такое Firebase

Firebase — платформа от Google: BaaS (Backend as a Service). Предоставляет готовую серверную инфраструктуру без написания бэкенда.

```yaml
# Базовый пакет (обязателен для всех Firebase-сервисов)
dependencies:
  firebase_core: ^2.27.0
```

---

## 2. Ключевые сервисы Firebase для Flutter

| Сервис | Пакет | Назначение |
|--------|-------|-----------|
| Authentication | `firebase_auth` | Email/Google/Apple/Phone вход |
| Firestore | `cloud_firestore` | NoSQL база в реальном времени |
| Storage | `firebase_storage` | Хранение файлов (фото, видео) |
| FCM | `firebase_messaging` | Push-уведомления |
| Analytics | `firebase_analytics` | Аналитика событий |
| Crashlytics | `firebase_crashlytics` | Отчёты об ошибках |
| Remote Config | `firebase_remote_config` | A/B тесты, feature flags |

---

## 3. Инициализация (обязательно!)

```dart
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart'; // генерируется через flutterfire configure

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const MyApp());
}
```

```bash
# Установка FlutterFire CLI
dart pub global activate flutterfire_cli

# Настройка проекта (создаёт firebase_options.dart)
flutterfire configure
```

---

## 4. Настройка проекта

1. Создать проект в [console.firebase.google.com](https://console.firebase.google.com)
2. Запустить `flutterfire configure` — автоматически добавит `google-services.json` (Android) и `GoogleService-Info.plist` (iOS)
3. Файл `firebase_options.dart` генерируется автоматически — не добавляй в `.gitignore`

---

## 5. Рекомендации по архитектуре

1. **Инициализируй Firebase один раз** в `main()` до `runApp`.
2. **Каждый Firebase-сервис** получай через его синглтон: `FirebaseAuth.instance`, `FirebaseFirestore.instance`.
3. **Не обращайся к Firebase напрямую из виджетов** — оборачивай в Repository.
4. **Crashlytics как FlutterError handler** — ловит все ошибки Flutter автоматически.
5. **Храни `firebase_options.dart` в git** — это не секрет, там только публичные App ID.
