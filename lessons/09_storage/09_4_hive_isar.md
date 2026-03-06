# 09.4 Hive и Isar — NoSQL хранилища

## 1. Суть

**Hive** и **Isar** — быстрые NoSQL базы данных для Flutter. Хранят Dart-объекты без SQL. Isar — более современная замена Hive с лучшей производительностью и реактивными запросами.

| | Hive | Isar |
|--|------|------|
| Тип | Key-Value (Box) | Документная БД |
| Запросы | Ручной фильтр (where в Dart) | DSL-запросы (типобезопасный) |
| Производительность | Высокая | Очень высокая (Rust-движок) |
| Реактивность | Нет (только вручную) | Stream из коробки |
| Codegen | Опционально | Обязателен |
| Шифрование | HiveCipher | Нет встроенного |

---

## 2. Hive — установка и использование

```yaml
dependencies:
  hive_flutter: ^1.1.0

dev_dependencies:
  hive_generator: ^2.0.1  # для TypeAdapter
  build_runner: ^2.4.8
```

```dart
import 'package:hive_flutter/hive_flutter.dart';

// --- Инициализация (main.dart) ---
void main() async {
  await Hive.initFlutter();
  Hive.registerAdapter(SettingsAdapter()); // регистрируем адаптер
  await Hive.openBox<Settings>('settings'); // открываем box
  runApp(const MyApp());
}

// --- Модель с TypeAdapter ---
part 'settings.g.dart'; // сгенерируется

@HiveType(typeId: 0)    // уникальный typeId для каждого класса
class Settings extends HiveObject {
  @HiveField(0)
  String languageCode;

  @HiveField(1)
  bool isDarkMode;

  @HiveField(2)
  int fontSize;

  Settings({
    required this.languageCode,
    required this.isDarkMode,
    required this.fontSize,
  });
}

// --- Использование Box ---
final box = Hive.box<Settings>('settings');

// Запись
await box.put('user_settings', Settings(
  languageCode: 'ru',
  isDarkMode: false,
  fontSize: 16,
));

// Чтение
final settings = box.get('user_settings');
final isDark = settings?.isDarkMode ?? false;

// Список всех значений
final allSettings = box.values.toList();

// Слушать изменения
box.listenable().addListener(() {
  final updated = box.get('user_settings');
  // обновить UI
});

// Удаление
await box.delete('user_settings');
await box.clear(); // очистить всё
```

```bash
dart run build_runner build --delete-conflicting-outputs
```

---

## 3. Isar — установка и использование

```yaml
dependencies:
  isar: ^3.1.0
  isar_flutter_libs: ^3.1.0
  path_provider: ^2.1.2

dev_dependencies:
  isar_generator: ^3.1.0
  build_runner: ^2.4.8
```

```dart
import 'package:isar/isar.dart';
import 'package:path_provider/path_provider.dart';

part 'product.g.dart';

// --- Модель ---
@collection
class Product {
  Id id = Isar.autoIncrement;  // автоинкремент

  @Index(type: IndexType.value)
  late String name;

  late double price;
  late bool inStock;

  @Index()
  late String category;
}

// --- Инициализация ---
late final Isar isar;

Future<void> initIsar() async {
  final dir = await getApplicationDocumentsDirectory();
  isar = await Isar.open(
    [ProductSchema],
    directory: dir.path,
  );
}

// --- CRUD ---
// INSERT / UPDATE (upsert по id)
await isar.writeTxn(() async {
  await isar.products.put(product);
});

// SELECT все
final all = await isar.products.where().findAll();

// SELECT с условием (типобезопасный DSL)
final cheap = await isar.products
    .where()
    .filter()
    .priceLessThan(100)
    .and()
    .inStockEqualTo(true)
    .sortByPrice()
    .findAll();

// Реактивный Stream
Stream<List<Product>> watchProducts() =>
    isar.products.where().watch(fireImmediately: true);

// DELETE
await isar.writeTxn(() async {
  await isar.products.delete(productId);
});

// Транзакция
await isar.writeTxn(() async {
  await isar.products.putAll([product1, product2]);
});
```

---

## 4. Реальный пример — кеш данных с API + Isar

```dart
class ProductCacheRepository {
  final Isar _isar;
  final ProductApiService _api;

  const ProductCacheRepository(this._isar, this._api);

  // Сначала возвращаем из кеша, потом обновляем из сети
  Stream<List<Product>> getProducts() async* {
    // 1. Сразу отдаём кешированные данные
    final cached = await _isar.products.where().findAll();
    if (cached.isNotEmpty) yield cached;

    // 2. Запрашиваем свежие данные
    try {
      final fresh = await _api.fetchProducts();
      // 3. Сохраняем в Isar
      await _isar.writeTxn(() => _isar.products.putAll(fresh));
      yield fresh;
    } catch (_) {
      // Сеть недоступна — ничего не делаем, кеш уже у пользователя
    }
  }

  // Подписка на реактивные обновления
  Stream<List<Product>> watchProducts() =>
      _isar.products.where().watch(fireImmediately: true);
}
```

---

## 5. Под капотом

- **Hive**: данные в `.hive` файлах, TypeAdapter сериализует в бинарный формат.
- **Isar**: движок написан на Rust (через FFI), что даёт высокую скорость. Индексы — B-Tree.
- Оба работают в основном изолейте, тяжёлые операции можно перенести в compute.

---

## 6. Типичные ошибки

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `HiveError: Box not found` | Box не открыт до использования | `await Hive.openBox(...)` до `runApp` |
| `typeId` конфликт | Два класса с одинаковым `typeId` | Каждый `typeId` должен быть уникальным |
| Isar: данные не обновляются в UI | Используешь `findAll()` вместо `watch()` | Замени на `.watch()` |
| `Cannot write outside transaction` (Isar) | INSERT без `writeTxn` | Оборачивай в `isar.writeTxn(...)` |
| Hive поле добавлено, но не сохранено | `@HiveField` без `put` | Не забывай `save()` у `HiveObject` |

---

## 7. Рекомендации

1. **Isar вместо Hive** для новых проектов — лучше скорость, реактивность, запросы.
2. **Hive** хорош для простого key-value кеша без сложных запросов.
3. **`watch(fireImmediately: true)`** в Isar — сразу отдаёт данные + подписка на изменения.
4. **Всегда оборачивай изменения Isar в `writeTxn`** — иначе runtime ошибка.
5. **Уникальные `typeId` в Hive** — записывай в комментарии к классу, чтобы не перепутать.
6. **Для шифрования данных** используй Hive с `encryptionCipher` (Isar шифрование не поддерживает).
