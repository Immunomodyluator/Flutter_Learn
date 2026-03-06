# Квиз: App Store Publishing

**Тема:** 16.4 — Publishing to App Stores  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какие треки (tracks) существуют в Google Play Console?

- A) Alpha, Beta, Production
- B) Internal testing, Closed testing (Alpha), Open testing (Beta), Production
- C) Dev, Staging, Release
- D) Preview, Public

<details>
<summary>Ответ</summary>

**B) Internal → Closed → Open → Production**

```
Google Play треки (в порядке тестирования → релиз):

Internal testing:
- До 100 тестировщиков (email)
- Доступен немедленно (без ревью)
- Идеально для команды и QA

Closed testing (Alpha):
- До 200 тестировщиков или группы Google
- Требует приглашения
- Быстрый ревью

Open testing (Beta):
- Любой пользователь может присоединиться
- Страница "Стать тестировщиком" в Play Store
- Быстрый ревью

Production:
- Все пользователи
- Полный ревью Google (обычно несколько часов - 3 дней)
- Поддерживает поэтапный выпуск (Staged Rollout)
```

</details>

---

### Вопрос 2 🟢

Что такое поэтапный выпуск (Staged Rollout / Phased Release) и зачем он нужен?

- A) Выпуск приложения поэтапно по странам
- B) Постепенное увеличение процента пользователей, получающих обновление — позволяет обнаружить проблемы до массового распространения
- C) Только для iOS
- D) Staged Rollout — только для premium приложений

<details>
<summary>Ответ</summary>

**B) Постепенное увеличение аудитории обновления**

```
Google Play Staged Rollout:
1% → 5% → 10% → 20% → 50% → 100%
Мониторинг ANR/crashes на каждом этапе

iOS Phased Release (App Store):
7 дней: 1% → 2% → 5% → 10% → 20% → 50% → 100%
Можно приостановить на любом этапе на 30 дней

Когда останавливать:
- Crash rate вырос на 0.5%+
- ANR rate вырос (Android)
- Резкое падение ratings
- Критические баг-репорты

Play Console: Release → Production → Managed Publishing
App Store Connect: Phased Release → Pause
```

</details>

---

### Вопрос 3 🟢

Какие метаданные нужны для публикации в App Store?

- A) Только название и скриншоты
- B) App name, subtitle, description, keywords, support URL, screenshots для каждого размера экрана, App Store icon (1024×1024)
- C) Только APK/IPA файл
- D) Описание на английском достаточно для всех локалей

<details>
<summary>Ответ</summary>

**B) Полный набор метаданных**

```
App Store Connect требования:

Обязательно:
- App Name (до 30 символов)
- Subtitle (до 30 символов)
- Description (до 4000 символов)
- Keywords (до 100 символов через запятую)
- Support URL
- App icon 1024×1024 px (PNG, без прозрачности)
- Screenshots для КАЖДОГО размера в использовании:
  • iPhone 6.7" (1290×2796 или 1320×2868)
  • iPhone 6.5" (1242×2688 или 1284×2778)
  • iPad Pro 12.9" (если поддерживается)

Опционально:
- Preview video (15-30 сек, .mov/.mp4)
- What's New (до 4000 символов) для обновлений
- Rating (для правильного ограничения возраста)
```

</details>

---

### Вопрос 4 🟡

Как работает ревью App Store и как ускорить процесс?

- A) Ревью проходит автоматически, без людей
- B) Ревью делают живые люди (Apple Review Team); обычно 24-48 часов; ускорить через Expedited Review при критических багах; не нарушать Guidelines
- C) Ревью занимает 7-14 дней всегда
- D) Ревью можно пропустить за $99/год

<details>
<summary>Ответ</summary>

**B) Живые ревьюеры Apple, 24-48 часов**

```
Типичные причины отклонения:
❌ Guideline 2.1 — приложение вылетает/не работает
❌ Guideline 4.2 — минимальная функциональность (шаблонные приложения)
❌ Guideline 3.1.1 — платежи мимо Apple IAP
❌ Guideline 1.5 — запрос избыточных разрешений
❌ Неактуальные скриншоты (не соответствуют реальному приложению)
❌ Ссылки на другие магазины в описании

Ускорение:
- App Review → Submit Appeal / Request Expedited Review
  (для critical bug fix или время-чувствительных событий)
- Обоснование в Review Notes
- Всегда заполнять "Notes for reviewers" с тест-аккаунтом

Предотвращение отказов:
✅ Тестировать свежую сборку перед отправкой
✅ Проверять App Store Review Guidelines
✅ Корректная Privacy Policy URL
```

</details>

---

### Вопрос 5 🟡

Как подготовить скриншоты для нескольких локалей?

- A) Скриншоты на английском достаточно для всех стран
- B) `fastlane snapshot` — автоматизированные скриншоты через UI тесты; отдельные скриншоты для каждой локали улучшают конверсию в разных странах
- C) Скриншоты можно не добавлять
- D) App Store не поддерживает локализованные скриншоты

<details>
<summary>Ответ</summary>

**B) `fastlane snapshot` для автоматизации**

```ruby
# Fastfile:
lane :screenshots do
  snapshot(
    workspace: "Runner.xcworkspace",
    scheme: "Runner",
    devices: [
      "iPhone 15 Pro Max",  # 6.7"
      "iPhone 14 Plus",     # 6.5"
      "iPad Pro (12.9-inch) (6th generation)",
    ],
    languages: ["en-US", "ru", "de-DE"],
    output_directory: "./fastlane/screenshots",
    clear_previous_screenshots: true,
    # UI тест файл со скриншотами:
    # UITest target с `snapshot("01_HomeScreen")`
  )

  # Загрузить скриншоты в App Store Connect:
  deliver(skip_binary_upload: true, skip_metadata: true)
end

# В UI тесте:
// RunnerUITests.swift:
func test_takeScreenshots() {
    let app = XCUIApplication()
    setupSnapshot(app)
    app.launch()
    snapshot("01_HomeScreen")
    app.buttons["Products"].tap()
    snapshot("02_ProductsScreen")
}
```

</details>

---

### Вопрос 6 🟡

Как правильно настроить App Privacy (Privacy Nutrition Labels) в App Store Connect?

- A) Оставить пустым — необязательно
- B) Указать точно какие данные собирает приложение и для каких целей; несоответствие с реальностью — причина отклонения
- C) Выбрать "We don't collect data" всегда
- D) Privacy labels заполняет Apple автоматически

<details>
<summary>Ответ</summary>

**B) Точное указание собираемых данных**

```
App Privacy Label категории данных:

Contact Info: имя, email, телефон, адрес
Health & Fitness: здоровье, фитнес
Financial Info: платёжная информация
Location: точное/приблизительное местоположение
Sensitive Info: этнос, религия, политика
Contacts: список контактов
User Content: фото, видео, геймплей, сообщения
Browsing History: история просмотра в приложении
Identifiers: UserID, DeviceID
Usage Data: взаимодействие с приложением, данные ошибок
Diagnostics: crash logs, performance, другая диагностика

Для каждого:
- Использование: Analytics, App Functionality, Product Personalization...
- Linked to Identity: да/нет
- Used for Tracking: да/нет

Типичные связки:
Flutter + Firebase Analytics → Usage Data (Analytics, не linked to identity)
Firebase Auth + Crashlytics → Diagnostics + Identifiers
```

</details>

---

### Вопрос 7 🟡

Как настроить In-App Purchases в Flutter приложении?

- A) `flutter_iap` пакет
- B) `in_app_purchase` официальный Flutter пакет; настройка в App Store Connect + Google Play Console; Apple требует IAP для цифровых товаров
- C) Можно принимать платежи через Stripe без IAP
- D) IAP только для игр

<details>
<summary>Ответ</summary>

**B) `in_app_purchase` + настройка в консолях**

```dart
import 'package:in_app_purchase/in_app_purchase.dart';

class PurchaseService {
  final InAppPurchase _iap = InAppPurchase.instance;

  Future<void> initialize() async {
    final available = await _iap.isAvailable();
    if (!available) return;

    // Подписаться на обновления покупок:
    _iap.purchaseStream.listen(_handlePurchase);
  }

  Future<void> buyProduct(String productId) async {
    final response = await _iap.queryProductDetails({productId});
    if (response.error != null) return;

    final product = response.productDetails.first;
    final param = PurchaseParam(productDetails: product);
    await _iap.buyNonConsumable(purchaseParam: param);
  }

  void _handlePurchase(List<PurchaseDetails> purchases) {
    for (final purchase in purchases) {
      if (purchase.status == PurchaseStatus.purchased) {
        // Верифицировать на сервере!
        _verifyWithServer(purchase);
        _iap.completePurchase(purchase);
      }
    }
  }
}
```

</details>

---

### Вопрос 8 🔴

Как организовать versioning Flutter приложения (semver + build number)?

- A) Только `version: 1.0.0` в pubspec.yaml без build number
- B) `version: MAJOR.MINOR.PATCH+BUILD_NUMBER` в pubspec.yaml; BUILD_NUMBER — автоинкремент в CI; semver для маркетинговой версии
- C) Build number не нужен для Flutter
- D) Version и build number — одно и то же

<details>
<summary>Ответ</summary>

**B) `version: MAJOR.MINOR.PATCH+BUILD_NUMBER`**

```yaml
# pubspec.yaml:
version: 2.5.1+134  # 2.5.1 = marketing version, 134 = build number
```

```bash
# В CI (GitHub Actions) — использовать номер run:
flutter build appbundle \
  --build-name=2.5.1 \
  --build-number=$GITHUB_RUN_NUMBER \
  --release

# Или через Fastlane + pubspec:
# increment_flutter_version(version_number: "2.5.1")
# increment_flutter_version_code

# Правила:
# versionName (Android) / CFBundleShortVersionString (iOS) = MAJOR.MINOR.PATCH
# versionCode (Android) / CFBundleVersion (iOS) = BUILD_NUMBER (монотонно растёт!)
# BUILD_NUMBER должен быть строго БОЛЬШЕ предыдущего в каждом магазине
```

</details>

---

### Вопрос 9 🔴

Как настроить Managed Publishing в Google Play для контролируемых релизов?

- A) Managed Publishing — автоматический деплой
- B) Play Console → Publishing Overview → Managed Publishing: изменения накапливаются, публикуются только после ручного "Send changes to review"
- C) Managed Publishing только для Enterprise аккаунтов
- D) `flutter deploy --managed`

<details>
<summary>Ответ</summary>

**B) Накопление изменений + ручная публикация**

```
Google Play Managed Publishing:

Обычный режим:
каждое изменение → сразу отправляется на ревью

Managed Publishing:
изменение 1 → накапливается
изменение 2 → накапливается
изменение 3 → накапливается
                ↓
"Send changes to review" → всё отправляется разом

Зачем нужен:
✅ Обновить метаданные + скриншоты + новую версию в одном ревью
✅ Подготовить релиз заранее, опубликовать в нужный момент
✅ Избежать нескольких отдельных ревью

В Play Console:
Release → Publishing overview → Managed publishing → Enable
```

</details>

---

### Вопрос 10 🔴

Как настроить App Store Connect API (ASC API key) для автоматизации без 2FA?

- A) ASC API не существует
- B) Создать API Key в App Store Connect → Users and Access → Keys; использовать в Fastlane через `app_store_connect_api_key` action
- C) Только личный Apple ID для автоматизации
- D) ASC API требует Team Agent доступ у каждого CI runner

<details>
<summary>Ответ</summary>

**B) ASC API Key + Fastlane**

```ruby
# Fastfile:
lane :deploy_ios do
  # Авторизация через API Key (без 2FA!):
  api_key = app_store_connect_api_key(
    key_id: ENV["ASC_KEY_ID"],           # из App Store Connect
    issuer_id: ENV["ASC_ISSUER_ID"],     # из App Store Connect
    key_content: ENV["ASC_KEY_CONTENT"], # .p8 файл в base64
    in_house: false,
  )

  match(
    type: "appstore",
    api_key: api_key,
  )

  gym(
    workspace: "Runner.xcworkspace",
    scheme: "Runner",
  )

  upload_to_app_store(
    api_key: api_key,
    skip_metadata: false,
    skip_screenshots: false,
    submit_for_review: true,
    automatic_release: false,
    force: true,
  )
end

# Создание API Key:
# App Store Connect → Users and Access → Integrations → App Store Connect API
# → Generate API Key → роль "Developer" или "App Manager"
# Скачать .p8 один раз (нельзя скачать повторно!)
```

</details>
