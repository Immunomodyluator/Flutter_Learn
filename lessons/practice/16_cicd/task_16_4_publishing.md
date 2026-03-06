# 16.4: Публикация — Play Store и App Store

> Project: FitMenu | Глава 16 — CI/CD

### 16.4: Публикация FitMenu в Google Play Store и Apple App Store

🎯 **Цель шага:** Подготовить FitMenu к публикации в магазинах приложений: создать store listing, настроить скриншоты, metadata и автоматическую публикацию через `supply` (Android) и `deliver` (iOS).

---

📝 **Техническое задание:**

**1. Play Store — подготовка:**
```bash
# Создать Service Account в Google Play Console
# Скачать JSON ключ → сохранить как PLAY_STORE_JSON_KEY (secret)

# Структура metadata для supply
android/fastlane/metadata/android/
  ru-RU/
    title.txt                # "FitMenu — Трекер питания"
    short_description.txt    # до 80 символов
    full_description.txt     # до 4000 символов
    changelogs/
      1.0.0.txt              # "Первый релиз FitMenu"
  images/
    phoneScreenshots/
      01_home.png            # 320×568 – 3840×2160
      02_recipes.png
      03_meal_plan.png
    featureGraphic.png       # 1024×500
    icon.png                 # 512×512
```

**2. `pubspec.yaml` — финальная версия:**
```yaml
version: 1.0.0+1   # version: major.minor.patch+buildNumber
```

**3. Fastlane deploy lane (Play Store Internal):**
```ruby
lane :release do
  # Bumped версия
  version = "1.0.#{number_of_commits}"
  android_set_version_name(version_name: version)
  android_set_version_code(version_code: number_of_commits)

  gradle(task: "bundle", build_type: "Release", ...)

  supply(
    track: "internal",          # internal → alpha → beta → production
    aab:   "...",
    json_key_data: ENV["PLAY_STORE_JSON"],
    metadata_path: "fastlane/metadata/android",
  )
end
```

**4. App Store Connect — iOS:**
```ruby
lane :ios_release do
  build_app(
    scheme:        "Runner",
    configuration: "Release",
    export_options: { method: "app-store" },
  )
  deliver(
    submit_for_review:   true,
    automatic_release:   false,
    metadata_path:       "fastlane/metadata/ios",
    screenshots_path:    "fastlane/screenshots/ios",
  )
end
```

---

✅ **Критерии приёмки:**
- [ ] `title.txt`, `short_description.txt`, `full_description.txt` заполнены
- [ ] Скриншоты 4 штуки для `phoneScreenshots`
- [ ] `featureGraphic.png` 1024×500 подготовлен
- [ ] `versionCode` автоматически равен числу git-коммитов
- [ ] `supply(track: "internal")` успешно загружает AAB

---

💡 **Подсказка:** `supply init` скачивает текущие metadata из Play Console в правильную структуру папок. `deliver init` — аналогично для App Store. Переход internal→production делается вручную в консоли (требует проверки Google). Privacy Policy URL обязателен для приложений с авторизацией.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```
# Структура metadata для Play Store
android/fastlane/metadata/android/
├── ru-RU/
│   ├── title.txt
│   ├── short_description.txt
│   ├── full_description.txt
│   └── changelogs/
│       └── 1.txt
└── en-US/
    ├── title.txt
    ├── short_description.txt
    └── full_description.txt
```

```
# android/fastlane/metadata/android/ru-RU/title.txt
FitMenu — Трекер питания и рецептов
```

```
# android/fastlane/metadata/android/ru-RU/short_description.txt
Планируй питание, считай КБЖУ и открывай новые рецепты каждый день
```

```
# android/fastlane/metadata/android/ru-RU/full_description.txt
FitMenu — умный помощник для здорового питания и контроля веса.

✅ Что умеет FitMenu:
• Персональный план питания с расчётом КБЖУ
• База рецептов с фильтрами (калории, белки, время)
• Дневник питания с фото блюд
• Интеграция с Google Fit / Apple Health
• Синхронизация между устройствами через Firebase
• Тёмная тема и адаптивный дизайн

Создан Flutter-разработчиками для Flutter-сообщества как open-source проект.
```

```ruby
# android/fastlane/Fastfile

lane :promote_to_production do
  supply(
    track:               "internal",
    track_promote_to:    "production",
    rollout:             "0.1",   # 10% rollout
    json_key_data:       ENV["PLAY_STORE_JSON"],
    skip_upload_metadata: false,
    skip_upload_images:   false,
    skip_upload_screenshots: false,
  )
end
```

</details>
