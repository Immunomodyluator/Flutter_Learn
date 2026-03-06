# 12.5: Connectivity и Battery — офлайн-режим

> Project: FitMenu | Глава 12 — Device Features

### 12.5: ConnectivityService — индикатор офлайн-режима

🎯 **Цель шага:** Отслеживать состояние интернет-соединения в FitMenu — показывать баннер "Офлайн режим" когда нет сети и использовать кешированные данные.

---

📝 **Техническое задание:**

**Зависимость:** `connectivity_plus: ^5.0.0`

Реализуй `lib/core/network/connectivity_service.dart`.

**ConnectivityService:**
```dart
class ConnectivityService {
  Stream<bool> get isOnlineStream => Connectivity()
      .onConnectivityChanged
      .map((result) => result != ConnectivityResult.none);

  Future<bool> get isOnline async {
    final result = await Connectivity().checkConnectivity();
    return result != ConnectivityResult.none;
  }
}
```

**OfflineBanner виджет:**
```dart
class OfflineBanner extends StatelessWidget {
  // StreamBuilder на isOnlineStream
  // Когда офлайн: AnimatedContainer с жёлтым баннером вверху экрана
  // Когда онлайн: невидимый (height: 0)
}
```

**Интеграция в AppShell:**
- `OfflineBanner` над основным контентом через `Column`
- При переходе в офлайн: кешировать последние данные
- При восстановлении: автоматически синхронизировать

**Стратегия данных:**
```dart
Future<List<Recipe>> getRecipes() async {
  if (!await _connectivity.isOnline) {
    return _cache.getCachedRecipes(); // из Hive
  }
  final recipes = await _api.getPopularRecipes();
  await _cache.cacheRecipes(recipes);
  return recipes;
}
```

---

✅ **Критерии приёмки:**
- [ ] `onConnectivityChanged` — стрим изменений соединения
- [ ] Баннер появляется/скрывается анимированно (`AnimatedContainer`)
- [ ] Офлайн-стратегия: данные из кеша при отсутствии сети
- [ ] `ConnectivityResult.none` — нет сети; `.wifi`/`.mobile` — есть
- [ ] `StreamBuilder` в `OfflineBanner` подписан на стрим

---

💡 **Подсказка:** `connectivity_plus` определяет тип соединения, но не гарантирует реальный интернет (можно быть подключённым к Wi-Fi без интернета). Для точной проверки — ping известного хоста. `AnimatedSize` или `AnimatedContainer(height: ...)` — плавное появление баннера.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/network/connectivity_service.dart

import 'package:connectivity_plus/connectivity_plus.dart';

class ConnectivityService {
  final _connectivity = Connectivity();

  Stream<bool> get isOnlineStream =>
      _connectivity.onConnectivityChanged
          .map((result) => result != ConnectivityResult.none);

  Future<bool> get isOnline async {
    final result = await _connectivity.checkConnectivity();
    return result != ConnectivityResult.none;
  }
}
```

```dart
// lib/core/widgets/offline_banner.dart

import 'package:flutter/material.dart';

class OfflineBanner extends StatelessWidget {
  const OfflineBanner({super.key, required this.child});
  final Widget child;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        StreamBuilder<bool>(
          stream: ConnectivityService().isOnlineStream,
          initialData: true,
          builder: (context, snapshot) {
            final isOnline = snapshot.data ?? true;
            return AnimatedContainer(
              duration: const Duration(milliseconds: 300),
              height: isOnline ? 0 : 32,
              color: Colors.orange,
              child: isOnline
                  ? const SizedBox.shrink()
                  : const Center(
                      child: Text(
                        '📡 Офлайн режим — показываются кешированные данные',
                        style: TextStyle(color: Colors.white, fontSize: 12),
                      ),
                    ),
            );
          },
        ),
        Expanded(child: child),
      ],
    );
  }
}
```

</details>
