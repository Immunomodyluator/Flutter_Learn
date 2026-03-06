# 4.4 Прокручиваемые контейнеры

## 1. Суть концепции

Flutter предоставляет несколько виджетов для прокрутки: `ListView`, `GridView`, `CustomScrollView` + `Sliver`. Выбор зависит от количества элементов, типа лейаута и необходимых эффектов прокрутки.

---

## 2. ListView — список элементов

```dart
// ListView — для небольших статических списков (все дети в памяти)
ListView(
  padding: const EdgeInsets.all(16),
  children: const [
    Text('Элемент 1'),
    Text('Элемент 2'),
    Text('Элемент 3'),
  ],
)

// ListView.builder — ленивый (только видимые элементы в памяти)
ListView.builder(
  itemCount: items.length,
  itemExtent: 72,          // фиксированная высота — ускоряет прокрутку
  itemBuilder: (context, index) {
    return ListTile(
      title: Text(items[index].title),
      subtitle: Text(items[index].subtitle),
    );
  },
)

// ListView.separated — с разделителями
ListView.separated(
  itemCount: items.length,
  separatorBuilder: (_, __) => const Divider(height: 1),
  itemBuilder: (context, index) => ListTile(title: Text(items[index])),
)

// ListView.custom — с произвольным SliverChildDelegate
ListView.custom(
  childrenDelegate: SliverChildBuilderDelegate(
    (context, index) => ListTile(title: Text('$index')),
    childCount: 100,
  ),
)
```

---

## 3. GridView — сетка

```dart
// GridView.count — фиксированное количество столбцов
GridView.count(
  crossAxisCount: 2,
  crossAxisSpacing: 8,
  mainAxisSpacing: 8,
  childAspectRatio: 3 / 4,  // соотношение сторон ячейки
  padding: const EdgeInsets.all(16),
  children: items.map((item) => ProductCard(product: item)).toList(),
)

// GridView.builder — ленивая сетка
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 3,
    crossAxisSpacing: 4,
    mainAxisSpacing: 4,
  ),
  itemCount: photos.length,
  itemBuilder: (context, index) => Image.network(
    photos[index].url,
    fit: BoxFit.cover,
  ),
)

// GridView с адаптивной шириной ячейки
GridView.builder(
  gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
    maxCrossAxisExtent: 200,  // максимальная ширина ячейки
    mainAxisSpacing: 8,
    crossAxisSpacing: 8,
    childAspectRatio: 1,
  ),
  itemCount: items.length,
  itemBuilder: (_, i) => Card(child: Center(child: Text('$i'))),
)
```

---

## 4. SingleChildScrollView

```dart
// Для форм и небольшого контента, который нужно прокрутить
SingleChildScrollView(
  padding: const EdgeInsets.all(16),
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.stretch,
    children: [
      const TextField(decoration: InputDecoration(labelText: 'Имя')),
      const SizedBox(height: 16),
      const TextField(decoration: InputDecoration(labelText: 'Email')),
      const SizedBox(height: 16),
      // ... много полей
      FilledButton(onPressed: () {}, child: const Text('Сохранить')),
    ],
  ),
)

// С клавиатурой — добавляем padding снизу
SingleChildScrollView(
  padding: EdgeInsets.only(
    left: 16, right: 16,
    bottom: MediaQuery.of(context).viewInsets.bottom + 16,
  ),
  child: Column(children: [...]),
)
```

---

## 5. CustomScrollView + Slivers

```dart
// Мощный инструмент для сложных прокрутных UI
CustomScrollView(
  slivers: [
    // Коллапсирующий AppBar
    const SliverAppBar(
      expandedHeight: 200,
      pinned: true,                    // остаётся видимым при прокрутке
      floating: false,
      flexibleSpace: FlexibleSpaceBar(
        title: Text('Заголовок'),
        background: Image(image: NetworkImage('https://...'), fit: BoxFit.cover),
      ),
    ),

    // Секция с заголовком
    const SliverToBoxAdapter(
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Text('Рекомендуемые', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
      ),
    ),

    // Горизонтальный список
    SliverToBoxAdapter(
      child: SizedBox(
        height: 120,
        child: ListView.builder(
          scrollDirection: Axis.horizontal,
          itemCount: 10,
          itemBuilder: (_, i) => Card(child: SizedBox(width: 100, child: Center(child: Text('$i')))),
        ),
      ),
    ),

    // Вертикальный список
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (_, i) => ListTile(title: Text('Элемент $i')),
        childCount: 20,
      ),
    ),
  ],
)
```

---

## 6. Пагинация — подгрузка данных при прокрутке

```dart
class _InfiniteListState extends State<InfiniteList> {
  final ScrollController _scrollController = ScrollController();
  final List<String> _items = [];
  bool _isLoading = false;
  int _page = 1;

  @override
  void initState() {
    super.initState();
    _loadMore();
    _scrollController.addListener(() {
      if (_scrollController.position.pixels >=
          _scrollController.position.maxScrollExtent - 200) {
        _loadMore();
      }
    });
  }

  Future<void> _loadMore() async {
    if (_isLoading) return;
    setState(() => _isLoading = true);
    final newItems = await api.fetchPage(_page++);
    if (mounted) setState(() {
      _items.addAll(newItems);
      _isLoading = false;
    });
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      controller: _scrollController,
      itemCount: _items.length + (_isLoading ? 1 : 0),
      itemBuilder: (_, i) {
        if (i == _items.length) {
          return const Center(child: CircularProgressIndicator());
        }
        return ListTile(title: Text(_items[i]));
      },
    );
  }
}
```

---

## 7. Типичные ошибки

| Ошибка                                   | Проблема                    | Решение                             |
| ---------------------------------------- | --------------------------- | ----------------------------------- |
| `ListView` в `Column` без обёртки        | Unbounded height            | `Expanded(child: ListView(...))`    |
| `shrinkWrap: true` для длинных списков   | Ухудшает производительность | Использовать `Expanded`             |
| `ListView` для 3-5 элементов с `builder` | Оверинжиниринг              | Обычный `ListView(children: [...])` |
| Нет `itemExtent` при известной высоте    | Лишние вычисления           | Добавить `itemExtent`               |

---

## 8. Практические рекомендации

1. **`ListView.builder` для списков > 10 элементов** — ленивая загрузка.
2. **`itemExtent` если высота ячейки одинакова** — значительное ускорение.
3. **`CustomScrollView` + Slivers** для сложных страниц с AppBar + список.
4. **`ScrollController`** для пагинации и управления позицией прокрутки.
5. **`shrinkWrap: true` только для небольших списков** — не используй для длинных.
