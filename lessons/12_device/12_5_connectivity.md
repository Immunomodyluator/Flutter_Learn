# 12.5 Connectivity — состояние сети

## 1. Суть

`connectivity_plus` позволяет определить тип подключения (Wi-Fi, мобильный интернет, отсутствие сети) и реагировать на его изменения.

```yaml
dependencies:
  connectivity_plus: ^6.0.0
  internet_connection_checker_plus: ^2.0.0 # проверка реального доступа
```

> `connectivity_plus` определяет тип подключения, но **не гарантирует доступ к интернету** (например, Wi-Fi может быть без выхода в сеть). Для реальной проверки используй `internet_connection_checker_plus`.

---

## 2. Основной синтаксис

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

// Разовая проверка
final results = await Connectivity().checkConnectivity();
// results — List<ConnectivityResult>
final isConnected = results.any((r) => r != ConnectivityResult.none);

// Тип подключения
for (final result in results) {
  switch (result) {
    case ConnectivityResult.wifi:
      print('Wi-Fi');
    case ConnectivityResult.mobile:
      print('Мобильный интернет');
    case ConnectivityResult.ethernet:
      print('Ethernet');
    case ConnectivityResult.vpn:
      print('VPN');
    case ConnectivityResult.bluetooth:
      print('Bluetooth');
    case ConnectivityResult.none:
      print('Нет подключения');
    default:
      print('Неизвестно');
  }
}

// Поток изменений
final subscription = Connectivity().onConnectivityChanged.listen((results) {
  final hasNetwork = results.any((r) => r != ConnectivityResult.none);
  print('Сеть: ${hasNetwork ? "есть" : "нет"}');
});

// Не забудь отменить подписку!
subscription.cancel();
```

---

## 3. Проверка реального доступа к интернету

```dart
import 'package:internet_connection_checker_plus/internet_connection_checker_plus.dart';

final checker = InternetConnection();

// Разовая проверка
final hasInternet = await checker.hasInternetAccess;

// Поток
checker.onStatusChange.listen((status) {
  switch (status) {
    case InternetStatus.connected:
      print('Интернет есть');
    case InternetStatus.disconnected:
      print('Интернет недоступен');
  }
});
```

---

## 4. Реальный пример — сервис и offline UI

```dart
// --- connectivity_service.dart ---
import 'dart:async';
import 'package:connectivity_plus/connectivity_plus.dart';
import 'package:internet_connection_checker_plus/internet_connection_checker_plus.dart';

class ConnectivityService {
  final _connectivity = Connectivity();
  final _checker = InternetConnection();

  final _controller = StreamController<bool>.broadcast();

  Stream<bool> get onConnectivityChanged => _controller.stream;
  bool _isConnected = true;
  bool get isConnected => _isConnected;

  late StreamSubscription _sub;

  ConnectivityService() {
    _init();
  }

  Future<void> _init() async {
    _isConnected = await _checker.hasInternetAccess;

    _sub = _checker.onStatusChange.listen((status) {
      final connected = status == InternetStatus.connected;
      if (connected != _isConnected) {
        _isConnected = connected;
        _controller.add(_isConnected);
      }
    });
  }

  void dispose() {
    _sub.cancel();
    _controller.close();
  }
}

// --- offline_banner.dart ---
class OfflineBanner extends StatelessWidget {
  final ConnectivityService service;
  final Widget child;

  const OfflineBanner({
    super.key,
    required this.service,
    required this.child,
  });

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<bool>(
      stream: service.onConnectivityChanged,
      initialData: service.isConnected,
      builder: (context, snapshot) {
        final isConnected = snapshot.data ?? true;
        return Column(
          children: [
            if (!isConnected)
              Container(
                width: double.infinity,
                color: Colors.red,
                padding: const EdgeInsets.symmetric(vertical: 6),
                child: const Text(
                  'Нет подключения к интернету',
                  textAlign: TextAlign.center,
                  style: TextStyle(color: Colors.white),
                ),
              ),
            Expanded(child: child),
          ],
        );
      },
    );
  }
}

// --- main.dart ---
void main() {
  final connectivityService = ConnectivityService();

  runApp(
    OfflineBanner(
      service: connectivityService,
      child: const MyApp(),
    ),
  );
}
```

---

## 5. Паттерн Riverpod + Connectivity

```dart
// providers.dart
final connectivityProvider = StreamProvider<bool>((ref) {
  final checker = InternetConnection();
  return checker.onStatusChange.map(
    (s) => s == InternetStatus.connected,
  );
});

// В виджете:
class NetworkAwareButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final isConnected = ref.watch(connectivityProvider).valueOrNull ?? true;

    return ElevatedButton(
      onPressed: isConnected ? _doRequest : null,
      child: Text(isConnected ? 'Загрузить' : 'Нет сети'),
    );
  }
}
```

---

## 6. Под капотом

- `connectivity_plus` использует `NetworkCallback` (Android) / `NWPathMonitor` (iOS).
- `internet_connection_checker_plus` периодически делает HEAD-запросы к надёжным адресам (Google, Cloudflare).
- Событие смены типа подключения **не** означает появление интернета — это только смена интерфейса.

---

## 7. Типичные ошибки

| Ошибка                       | Причина                                   | Решение                                         |
| ---------------------------- | ----------------------------------------- | ----------------------------------------------- |
| Wi-Fi подключён, но сети нет | `connectivity_plus` не проверяет интернет | Использовать `internet_connection_checker_plus` |
| Утечка памяти                | `StreamSubscription` не отменён           | Вызывать `cancel()` в `dispose()`               |
| Событие приходит дважды      | Несколько подписчиков                     | Использовать `broadcast()` или один синглтон    |
| Не работает на iOS simulator | Симулятор не поддерживает все типы        | Тестировать на реальном устройстве              |

---

## 8. Рекомендации

1. **Используй `internet_connection_checker_plus`** вместо одного `connectivity_plus` для надёжной проверки.
2. **Singleton-сервис** — один экземпляр `ConnectivityService` на всё приложение.
3. **Offline-first подход**: кешируй данные (Hive/Isar), показывай кешированное при отсутствии сети.
4. **Не блокируй UI** — показывай баннер/снекбар, но не отключай весь экран.
5. **Повторная загрузка**: при восстановлении сети автоматически retry последнего запроса.
