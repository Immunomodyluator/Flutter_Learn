# Flutter Quiz — Plan & Progress

## Format

- Path: `lessons/quizzes/{NN_section}/quiz_NN_N_topic.md`
- 10 questions per file: Q1–3 🟢, Q4–7 🟡, Q8–10 🔴
- Answers hidden in `<details><summary>Ответ</summary>...</details>`

---

## Progress

| Section         | Files     | Status           |
| --------------- | --------- | ---------------- |
| 01_dart         | 7/7       | ✅               |
| 02_setup        | 3/3       | ✅               |
| 03_widgets      | 5/5       | ✅               |
| 04_layouts      | 5/5       | ✅               |
| 05_themes       | 3/3       | ✅               |
| 06_navigation   | 4/4       | ✅               |
| 07_state        | 6/6       | ✅               |
| 08_network      | 4/4       | ✅               |
| 09_storage      | 5/5       | ✅               |
| 10_firebase     | 5/5       | ✅               |
| 11_animations   | 4/4       | ✅               |
| 12_device       | 5/5       | ✅               |
| 13_architecture | 1/4       | 🔄               |
| 14_testing      | 0/4       | ⬜               |
| 15_performance  | 0/4       | ⬜               |
| 16_cicd         | 0/4       | ⬜               |
| 17_web_desktop  | 0/3       | ⬜               |
| 18_advanced     | 0/5       | ⬜               |
| **TOTAL**       | **75/98** | **23 remaining** |

---

## Remaining files (23)

### 13_architecture

- [x] `quiz_13_1_mvvm.md`
- [ ] `quiz_13_2_clean_architecture.md` — Layers: Entities/UseCases/Repositories/DataSources, dependency rule, interface segregation
- [ ] `quiz_13_3_di_get_it.md` — get_it ServiceLocator, injectable codegen, singleton/factory/lazySingleton, @injectable, @module
- [ ] `quiz_13_4_feature_first.md` — Feature-first vs layer-first, barrel exports (index.dart), package isolation, modular architecture

### 14_testing

- [ ] `quiz_14_1_unit_tests.md` — test(), expect(), mockito/mocktail when()/verify(), fake objects, coverage
- [ ] `quiz_14_2_widget_tests.md` — testWidgets, WidgetTester.pumpWidget, find.byType/text/key, tap/enterText/pump/pumpAndSettle
- [ ] `quiz_14_3_integration_tests.md` — integration_test package, IntegrationTestWidgetsFlutterBinding, real device/emulator
- [ ] `quiz_14_4_golden_tests.md` — matchesGoldenFile, --update-goldens, goldenFileComparator, platform pixel differences

### 15_performance

- [ ] `quiz_15_1_devtools.md` — Performance overlay, Widget Inspector, Memory profiler, Timeline, CPU profiler
- [ ] `quiz_15_2_rebuild_optimization.md` — const constructors, RepaintBoundary, shouldRebuild, select vs watch Riverpod, Keys
- [ ] `quiz_15_3_images.md` — cached_network_image, ResizeImage, cacheWidth/cacheHeight, WebP, precacheImage
- [ ] `quiz_15_4_isolates.md` — Isolate.run(), compute(), SendPort/ReceivePort, isolate groups (Flutter 3+), Worker pool

### 16_cicd

- [ ] `quiz_16_1_signing.md` — Android keystore, iOS provisioning profile + Xcode signing, upload key vs app signing key
- [ ] `quiz_16_2_github_actions.md` — flutter/actions workflow, matrix strategy, secrets, artifacts, pub cache
- [ ] `quiz_16_3_fastlane.md` — Fastfile, deliver (iOS), supply (Android), match, gym, scan
- [ ] `quiz_16_4_publishing.md` — Google Play Console tracks, App Store Connect review, metadata/screenshots, phased rollout

### 17_web_desktop

- [ ] `quiz_17_1_web.md` — flutter build web, CanvasKit vs HTML renderer, PWA manifest, SEO limitations, url_strategy
- [ ] `quiz_17_2_desktop.md` — flutter build windows/macos/linux, window_manager, menu bar, file system access, MSIX
- [ ] `quiz_17_3_adaptive_ui.md` — Pointer events, Shortcuts/Intent, MouseRegion hover, context menus, window size breakpoints

### 18_advanced

- [ ] `quiz_18_1_custom_painter.md` — CustomPainter paint(Canvas, Size), Paint, Path, shouldRepaint, CustomPaint + child
- [ ] `quiz_18_2_packages.md` — pub.dev publish checklist, pubspec versioning, CHANGELOG.md, platform interface pattern
- [ ] `quiz_18_3_flavors.md` — flutter_flavorizr, dart-define, flavor-specific assets, google-services.json per flavor
- [ ] `quiz_18_4_security.md` — Certificate pinning, root/jailbreak detection, ProGuard --obfuscate, flutter_secure_storage, biometric
- [ ] `quiz_18_5_accessibility.md` — Semantics, excludeSemantics, MergeSemantics, TalkBack/VoiceOver, textScaleFactor, WCAG

---

## Next action

➡️ Create `lessons/quizzes/13_architecture/quiz_13_2_clean_architecture.md`
