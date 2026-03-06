# 16. CI/CD для Flutter-приложений

## 1. Суть

**CI** (Continuous Integration) — автоматическая проверка кода при каждом PR/push: тесты, линтер, сборка.
**CD** (Continuous Delivery/Deployment) — автоматическая доставка сборки в TestFlight/Internal Track/Production.

---

## 2. Типичный пайплайн

```
Разработчик → git push → CI запускается автоматически
  ↓
1. Установка зависимостей (flutter pub get)
2. Статический анализ (flutter analyze)
3. Тесты (flutter test)
4. Сборка (flutter build apk / ipa)
5. Подпись (keystore / certificates)
6. Публикация (Google Play / TestFlight)
```

---

## 3. Популярные CI/CD платформы для Flutter

| Платформа          | Бесплатный план | OS                    | Особенности                                     |
| ------------------ | --------------- | --------------------- | ----------------------------------------------- |
| **GitHub Actions** | 2000 мин/мес    | Linux, macOS, Windows | Интеграция с GitHub, большая экосистема actions |
| **Codemagic**      | 500 мин/мес     | все                   | Специализирован для Flutter                     |
| **Bitrise**        | 90 мин/мес      | все                   | Visual workflow                                 |
| **GitLab CI**      | 400 мин/мес     | все                   | Self-hosted вариант                             |
| **Fastlane**       | Бесплатен       | macOS (iOS)           | Скриптовый подход, локально и в CI              |

---

## 4. Секреты и переменные окружения

Ключи подписи, пароли, API-ключи **никогда не коммитятся** в репозиторий.

```bash
# GitHub Secrets: Settings → Secrets and variables → Actions
KEYSTORE_BASE64       # base64 от keystore-файла
KEYSTORE_PASSWORD
KEY_ALIAS
KEY_PASSWORD
PLAY_STORE_JSON_KEY   # сервисный аккаунт Google Play
APP_STORE_CONNECT_KEY # Apple Connect API Key
```

---

## 5. Обзор тем раздела

| Файл                     | Тема                                                   |
| ------------------------ | ------------------------------------------------------ |
| `16_1_signing.md`        | Подпись приложения: Android Keystore, iOS Certificates |
| `16_2_github_actions.md` | GitHub Actions: CI/CD workflow для Flutter             |
| `16_3_fastlane.md`       | Fastlane: автоматизация публикации                     |
| `16_4_publishing.md`     | Публикация в Google Play и App Store                   |

---

## 6. Рекомендации

1. **CI с первого дня** — даже `flutter analyze && flutter test` в GitHub Actions.
2. **Secrets** для всех ключей — никогда в исходном коде.
3. **Codemagic** — самый быстрый старт для Flutter CI/CD.
4. **Fastlane** — хорош для сложных пайплайнов и локального запуска.
5. **Версии приложения** автоматически из git tag: `1.2.3+456`.
