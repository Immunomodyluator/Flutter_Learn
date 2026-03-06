# Квиз: Стратегии кэширования

**Тема:** 09.5 — Caching Strategies  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое `cached_network_image` и зачем его использовать?

- A) Встроенный виджет Flutter для загрузки изображений
- B) Пакет для загрузки и кэширования сетевых изображений с placeholder и error builder
- C) Инструмент для оптимизации PNG файлов
- D) Менеджер памяти приложения

<details>
<summary>Ответ</summary>

**B) Пакет для загрузки и кэширования сетевых изображений**

```dart
CachedNetworkImage(
  imageUrl: 'https://example.com/photo.jpg',
  placeholder: (ctx, url) => CircularProgressIndicator(),
  errorWidget: (ctx, url, err) => Icon(Icons.error),
  width: 200,
  height: 200,
  fit: BoxFit.cover,
)
```

Автоматически кэширует на диск и в памяти. Использует `flutter_cache_manager` под капотом.

</details>

---

### Вопрос 2 🟢

Какие два уровня кэширования обычно используются в Flutter приложениях?

- A) L1 и L2 кэш процессора
- B) In-memory кэш (быстрый, ограниченный) + disk кэш (медленнее, персистентный)
- C) HTTP кэш + SharedPreferences кэш
- D) Widget кэш + State кэш

<details>
<summary>Ответ</summary>

**B) Память (in-memory) + диск (disk cache)**

| Уровень | Скорость | Размер        | Персистентность |
| ------- | -------- | ------------- | --------------- |
| Memory  | ~1ms     | ограничен RAM | Только сессия   |
| Disk    | ~10ms    | ГБ доступно   | Между запусками |

Flutter `PaintingBinding.instance.imageCache` — in-memory для изображений.
`flutter_cache_manager` — disk cache для любых данных.

</details>

---

### Вопрос 3 🟢

Как очистить кэш изображений Flutter?

- A) `Image.clearCache()`
- B) `PaintingBinding.instance.imageCache.clear()` — сбросить memory cache
- C) `CacheManager.clearAll()`
- D) Перезапуск приложения

<details>
<summary>Ответ</summary>

**B) `PaintingBinding.instance.imageCache.clear()`**

```dart
// Очистить весь in-memory кэш изображений:
PaintingBinding.instance.imageCache.clear();
PaintingBinding.instance.imageCache.clearLive(); // только живые (used)

// Настроить лимиты:
PaintingBinding.instance.imageCache.maximumSize = 100; // макс. изображений
PaintingBinding.instance.imageCache.maximumSizeBytes = 50 << 20; // 50 MB

// Disk кэш (cached_network_image):
await DefaultCacheManager().emptyCache();
```

</details>

---

### Вопрос 4 🟡

Что такое Stale-While-Revalidate (SWR) стратегия кэширования?

- A) Кэш запрещён — всегда запрос к серверу
- B) Вернуть кэшированные данные немедленно и параллельно обновить кэш из сети — пользователь видит данные без ожидания
- C) Данные показываются только после обновления с сервера
- D) Кэш хранит данные 1 секунду (stale = 1s)

<details>
<summary>Ответ</summary>

**B) Показать кэш сразу + тихо обновить в фоне**

```dart
// Реализация SWR:
Stream<List<Product>> getProducts() async* {
  // 1. Вернуть кэш немедленно:
  final cached = await localCache.getProducts();
  if (cached.isNotEmpty) yield cached;

  // 2. Обновить с сервера в фоне:
  try {
    final fresh = await api.getProducts();
    await localCache.saveProducts(fresh);
    yield fresh; // обновить UI
  } catch (e) {
    if (cached.isEmpty) rethrow; // ошибка только если нет кэша
  }
}
```

</details>

---

### Вопрос 5 🟡

Как настроить кастомную `CacheManager` в `flutter_cache_manager`?

- A) `DefaultCacheManager(config: ...)`
- B) Наследовать `BaseCacheManager` с кастомным `Config(key, stalePeriod, maxNrOfCacheObjects)`
- C) `CacheManager.create(maxAge: Duration(days: 7))`
- D) Настройка недоступна в публичном API

<details>
<summary>Ответ</summary>

**B) `BaseCacheManager` + `Config`**

```dart
class MyCacheManager extends CacheManager with ImageCacheManager {
  static const key = 'myCacheManager';

  MyCacheManager._() : super(Config(
    key,
    stalePeriod: const Duration(days: 3),   // время до устаревания
    maxNrOfCacheObjects: 200,               // макс. файлов
    repo: JsonCacheInfoRepository(databaseName: key),
    fileService: HttpFileService(),
  ));

  static final instance = MyCacheManager._();
}
```

</details>

---

### Вопрос 6 🟡

Как реализовать TTL (Time to Live) кэш для API данных?

- A) `CachePolicy.withTTL(60)` в http пакете
- B) Сохранять вместе с данными timestamp создания и при чтении проверять `DateTime.now().difference(createdAt) > ttl`
- C) `Hive.box.setExpiry(key, ttl)`
- D) HTTP заголовки автоматически

<details>
<summary>Ответ</summary>

**B) Сохранять timestamp + проверять при чтении**

```dart
class CacheEntry<T> {
  final T data;
  final DateTime createdAt;
  final Duration ttl;

  CacheEntry(this.data, this.ttl) : createdAt = DateTime.now();

  bool get isExpired => DateTime.now().difference(createdAt) > ttl;
}

// Использование:
Future<List<User>> getUsers() async {
  final cached = _cache['users'];
  if (cached != null && !cached.isExpired) {
    return cached.data as List<User>;
  }
  final users = await api.getUsers();
  _cache['users'] = CacheEntry(users, Duration(minutes: 5));
  return users;
}
```

</details>

---

### Вопрос 7 🟡

Что такое LRU (Least Recently Used) кэш и как реализовать его в Dart?

- A) Last Read Updated — кэш с автообновлением
- B) Вытеснять наименее недавно использованные элементы при переполнении; в Dart можно использовать `LinkedHashMap` или `lru_map` пакет
- C) Flutter `ImageCache` не использует LRU
- D) Алгоритм сортировки данных

<details>
<summary>Ответ</summary>

**B) Вытесняет наименее недавно использованные элементы**

```dart
import 'package:collection/collection.dart';

class LruCache<K, V> {
  final int maxSize;
  final _map = LinkedHashMap<K, V>();

  LruCache(this.maxSize);

  V? get(K key) {
    if (!_map.containsKey(key)) return null;
    final value = _map.remove(key)!;
    _map[key] = value; // переместить в конец (most recently used)
    return value;
  }

  void put(K key, V value) {
    _map.remove(key);
    _map[key] = value;
    if (_map.length > maxSize) {
      _map.remove(_map.keys.first); // вытеснить LRU (начало)
    }
  }
}
```

Flutter `ImageCache` также использует LRU стратегию.

</details>

---

### Вопрос 8 🔴

Как правильно реализовать offline-first архитектуру с синхронизацией?

- A) Хранить данные только локально, без синхронизации
- B) Read from cache → show UI → fetch from network → update cache → update UI; для записи: save locally → queue for sync → sync when online
- C) Блокировать UI до получения ответа от сервера
- D) Использовать только HTTP кэш заголовки

<details>
<summary>Ответ</summary>

**B) Cache-first чтение + очередь операций для синхронизации**

```dart
class OfflineFirstRepository {
  Future<void> createTodo(Todo todo) async {
    // 1. Сохранить локально сразу:
    await localDb.insert(todo.copyWith(syncStatus: SyncStatus.pending));

    // 2. Попробовать синхронизировать:
    if (await connectivity.isConnected) {
      await _sync(todo);
    }
    // Если offline — SyncWorker отправит при восстановлении сети
  }

  // Запускается при connectivity change:
  Future<void> syncPending() async {
    final pending = await localDb.getPendingItems();
    for (final item in pending) {
      try {
        await api.create(item);
        await localDb.markSynced(item.id);
      } catch(_) { break; } // retry later
    }
  }
}
```

</details>

---

### Вопрос 9 🔴

Как избежать "thundering herd" проблемы при инвалидации кэша?

- A) Эта проблема не существует в Flutter
- B) Использовать mutex/lock, дедупликацию одновременных запросов (`singleFlight` паттерн) или случайный jitter при retry
- C) Удалить кэш и перезагрузить приложение
- D) Увеличить TTL до 24 часов

<details>
<summary>Ответ</summary>

**B) `singleFlight` паттерн — дедупликация одновременных запросов**

Thundering herd: при инвалидации кэша множество клиентов одновременно идут к бэкенду.

```dart
class SingleFlightCache<K, V> {
  final _inFlight = <K, Future<V>>{};
  final _cache = <K, V>{};

  Future<V> get(K key, Future<V> Function() fetcher) async {
    if (_cache.containsKey(key)) return _cache[key]!;

    // singleFlight: если запрос уже в полёте — ждём его
    return _inFlight[key] ??= () async {
      try {
        final value = await fetcher();
        _cache[key] = value;
        return value;
      } finally {
        _inFlight.remove(key);
      }
    }();
  }
}
```

</details>

---

### Вопрос 10 🔴

Как правильно кэшировать данные чувствительных к безопасности (токены, платёжные данные)?

- A) Использовать SharedPreferences
- B) Использовать `flutter_secure_storage` — данные шифруются через Keychain (iOS) / EncryptedSharedPreferences/Keystore (Android)
- C) Хранить в зашифрованном файле с ключом в коде приложения
- D) Не кэшировать — всегда запрашивать заново

<details>
<summary>Ответ</summary>

**B) `flutter_secure_storage` — OS-level шифрование**

```dart
const storage = FlutterSecureStorage(
  aOptions: AndroidOptions(encryptedSharedPreferences: true),
  iOptions: IOSOptions(accessibility: KeychainAccessibility.first_unlock),
);

// Сохранить токен:
await storage.write(key: 'access_token', value: token);

// Прочитать:
final token = await storage.read(key: 'access_token');

// Удалить при logout:
await storage.delete(key: 'access_token');
await storage.deleteAll(); // все записи

// НЕ делать:
// SharedPreferences.setString('token', token); // не шифрует!
// File('token.txt').writeAsString(token); // небезопасно!
```

Никогда не хранить secrets в коде (`const apiKey = '...'`) — деобфускация тривиальна.

</details>
