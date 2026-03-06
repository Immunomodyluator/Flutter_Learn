# 7.6 GetX

## 1. Суть концепции

GetX — пакет "всё в одном": управление состоянием, навигация, внедрение зависимостей, утилиты. Минимум кода, максимум функциональности. Спорный выбор в сообществе: удобен для быстрого старта, но может привести к архитектурным проблемам.

```yaml
dependencies:
  get: ^4.6.6
```

---

## 2. Три вида состояния в GetX

### Reactive State (Rx — реактивное)

```dart
class CounterController extends GetxController {
  final count = 0.obs;  // .obs делает переменную наблюдаемой (RxInt)
  final name = ''.obs;  // RxString
  final items = <Product>[].obs;  // RxList

  void increment() => count++;
  void setName(String n) => name.value = n;
  void addItem(Product p) => items.add(p);
}

// В виджете — Obx автоматически rebuild при изменении .obs переменных
class CounterView extends StatelessWidget {
  final controller = Get.put(CounterController());

  @override
  Widget build(BuildContext context) {
    return Obx(() => Text('${controller.count}'));
    // Rebuild только когда count изменился
  }
}
```

### Simple State (GetBuilder — менее реактивное, лучше производительность)

```dart
class CartController extends GetxController {
  int _count = 0;
  int get count => _count;

  void addItem() {
    _count++;
    update();  // аналог setState / notifyListeners
  }
}

// GetBuilder — rebuild только при update()
GetBuilder<CartController>(
  builder: (controller) => Text('${controller.count}'),
)
```

---

## 3. Навигация с GetX

```dart
// Без BuildContext!
Get.to(() => DetailsScreen());
Get.toNamed('/details');
Get.back();
Get.off(() => HomeScreen());       // replace
Get.offAll(() => LoginScreen());   // clear stack
Get.offAllNamed('/login');

// С аргументами
Get.to(() => DetailsScreen(), arguments: {'id': 42});
final args = Get.arguments as Map;

// Настройка маршрутов
GetMaterialApp(
  initialRoute: '/',
  getPages: [
    GetPage(name: '/', page: () => const HomeScreen()),
    GetPage(name: '/details', page: () => const DetailsScreen()),
    GetPage(
      name: '/product/:id',
      page: () => ProductDetailScreen(),
      binding: ProductBinding(),
    ),
  ],
)
```

---

## 4. Dependency Injection

```dart
// put — немедленное создание
Get.put(ApiService());
Get.put(UserRepository());

// lazyPut — создание при первом обращении
Get.lazyPut<AuthService>(() => AuthService());

// Получение
final api = Get.find<ApiService>();

// Bindings — связывание зависимостей с маршрутом
class HomeBinding extends Bindings {
  @override
  void dependencies() {
    Get.lazyPut<HomeController>(() => HomeController());
  }
}

GetPage(
  name: '/home',
  page: () => const HomeScreen(),
  binding: HomeBinding(),  // контроллер создаётся при входе, удаляется при выходе
)
```

---

## 5. GetxController — жизненный цикл

```dart
class ProductController extends GetxController {
  final _products = <Product>[].obs;
  List<Product> get products => _products;

  @override
  void onInit() {
    super.onInit();
    loadProducts();  // вызывается при создании контроллера
  }

  @override
  void onReady() {
    super.onReady();
    // вызывается после первого build
  }

  @override
  void onClose() {
    // вызывается при удалении контроллера (аналог dispose)
    super.onClose();
  }

  Future<void> loadProducts() async {
    final data = await Get.find<ApiService>().getProducts();
    _products.assignAll(data);
  }
}

// В виджете
class ProductScreen extends GetView<ProductController> {
  // GetView автоматически предоставляет controller через Get.find
  const ProductScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Obx(() =>
      controller.products.isEmpty
        ? const CircularProgressIndicator()
        : ListView.builder(
            itemCount: controller.products.length,
            itemBuilder: (_, i) => ProductCard(product: controller.products[i]),
          ),
    );
  }
}
```

---

## 6. Утилиты GetX

```dart
// Снекбар
Get.snackbar('Успех', 'Товар добавлен в корзину');

// Диалог
Get.defaultDialog(
  title: 'Удалить?',
  middleText: 'Это нельзя отменить',
  onConfirm: () { /* ... */ Get.back(); },
);

// BottomSheet
Get.bottomSheet(
  Container(height: 200, child: const Text('Контент')),
  backgroundColor: Colors.white,
);

// Локализация
class Translations extends Translations {
  @override
  Map<String, Map<String, String>> get keys => {
    'ru_RU': {'hello': 'Привет'},
    'en_US': {'hello': 'Hello'},
  };
}
// Использование:
Text('hello'.tr)

// Размеры без context
GetX предоставляет Get.width, Get.height, Get.context
```

---

## 7. GetMaterialApp — замена MaterialApp

```dart
void main() => runApp(
  GetMaterialApp(
    title: 'My App',
    initialRoute: '/',
    getPages: appPages,
    translations: AppTranslations(),
    locale: const Locale('ru', 'RU'),
    theme: AppTheme.light,
    darkTheme: AppTheme.dark,
    themeMode: ThemeMode.system,
  ),
);
```

---

## 8. Спорные аспекты GetX

| Плюсы                 | Минусы                                    |
| --------------------- | ----------------------------------------- |
| Минимум кода          | Нарушает Flutter-паттерны                 |
| Не нужен BuildContext | Глобальные зависимости сложно тестировать |
| Всё в одном пакете    | Сильная связанность                       |
| Быстрый старт         | Трудно масштабировать                     |
|                       | Не поддерживает null safety хорошо        |

---

## 9. Когда использовать GetX

✅ **Подходит**:

- Небольшие проекты
- Прототипы и MVP
- Команда знает GetX

❌ **Не рекомендуется**:

- Крупные enterprise-приложения
- Командная разработка с разными уровнями
- Проекты с серьёзным тестированием

---

## 10. Практические рекомендации

1. **GetX или Riverpod** — выберите один паттерн для проекта, не смешивай.
2. **`Obx` точечно** — оборачивай только то, что должно обновляться.
3. **`GetBuilder` быстрее `Obx`** для списков с частыми обновлениями.
4. **Bindings для каждого маршрута** — автоматическое создание и удаление контроллеров.
5. **Для серьёзных проектов рассмотри Riverpod** — лучше тестируемость и архитектура.
