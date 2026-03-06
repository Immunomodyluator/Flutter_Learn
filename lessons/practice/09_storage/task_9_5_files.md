# 9.5: Работа с файлами — экспорт дневника питания

> Project: FitMenu | Глава 9 — Storage

### 9.5: Экспорт дневника в CSV/JSON через path_provider и dart:io

🎯 **Цель шага:** Реализовать экспорт дневника питания в CSV-файл — пользователь нажимает "Экспорт" и получает файл `fitmenu_log_2024-01-15.csv` в директории документов устройства.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  path_provider: ^2.1.0
  share_plus: ^7.2.0   # для шаринга файла
```

Реализуй `lib/features/meal_plan/services/export_service.dart`.

**ExportService:**
```dart
class ExportService {
  Future<File> exportToCsv(List<MealLogEntry> entries, DateTime date) async { ... }
  Future<File> exportToJson(List<MealLogEntry> entries, DateTime date) async { ... }
  Future<void> shareFile(File file) async { ... }
}
```

**CSV формат:**
```
Date,MealType,Title,Calories,Protein,Fat,Carbs
2024-01-15,breakfast,Овсянка с ягодами,350,12.5,6.0,65.0
2024-01-15,lunch,Куриный суп,280,25.0,8.0,22.0
```

**Алгоритм exportToCsv:**
1. Получить директорию: `getApplicationDocumentsDirectory()`
2. Сформировать имя файла: `fitmenu_log_${dateStr}.csv`
3. Создать `File(path)` и записать через `file.writeAsString(csvContent)`
4. Вернуть `File`

**UI — кнопка экспорта:**
```dart
ElevatedButton.icon(
  icon: const Icon(Icons.download),
  label: const Text('Экспорт в CSV'),
  onPressed: () async {
    final file = await exportService.exportToCsv(entries, selectedDate);
    await exportService.shareFile(file);
  },
)
```

**share_plus:** `Share.shareXFiles([XFile(file.path)], text: 'Мой дневник питания')`.

---

✅ **Критерии приёмки:**
- [ ] CSV-файл создаётся в `getApplicationDocumentsDirectory()`
- [ ] Заголовок CSV на русском: `Дата,Тип,Блюдо,Калории,Белки,Жиры,Углеводы`
- [ ] Имя файла содержит дату: `fitmenu_log_2024-01-15.csv`
- [ ] `share_plus` открывает системный шаринг-лист
- [ ] Числа с плавающей точкой корректно форматируются (не экспоненциальная запись)
- [ ] Обработаны ошибки записи файла (нет места, нет прав)

---

💡 **Подсказка:** `getApplicationDocumentsDirectory()` возвращает директорию, доступную только приложению. `getExternalStorageDirectory()` (Android) — для файлов доступных пользователю. `file.writeAsString()` — асинхронная запись; `file.writeAsStringSync()` — синхронная (не рекомендуется в UI потоке). Запятые в данных нужно экранировать кавычками.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/meal_plan/services/export_service.dart

import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:share_plus/share_plus.dart';

class ExportService {
  Future<File> exportToCsv(
    List<MealLogEntry> entries,
    DateTime date,
  ) async {
    final dateStr = '${date.year}-'
        '${date.month.toString().padLeft(2, '0')}-'
        '${date.day.toString().padLeft(2, '0')}';

    final buffer = StringBuffer();
    // Заголовок
    buffer.writeln('Дата,Тип,Блюдо,Калории,Белки,Жиры,Углеводы');
    // Данные
    for (final entry in entries) {
      final rowDate = entry.eatenAt.toIso8601String().substring(0, 10);
      final title = entry.title.contains(',')
          ? '"${entry.title}"' // экранируем запятые
          : entry.title;
      buffer.writeln([
        rowDate,
        entry.mealType,
        title,
        entry.calories.toStringAsFixed(1),
        entry.protein.toStringAsFixed(1),
        entry.fat.toStringAsFixed(1),
        entry.carbs.toStringAsFixed(1),
      ].join(','));
    }

    final dir  = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/fitmenu_log_$dateStr.csv');

    try {
      await file.writeAsString(buffer.toString(), flush: true);
    } on FileSystemException catch (e) {
      throw ExportException('Ошибка записи файла: ${e.message}');
    }

    return file;
  }

  Future<File> exportToJson(
    List<MealLogEntry> entries,
    DateTime date,
  ) async {
    final dateStr = date.toIso8601String().substring(0, 10);
    final data = {
      'exportDate': DateTime.now().toIso8601String(),
      'period': dateStr,
      'totalCalories': entries.fold(0.0, (s, e) => s + e.calories),
      'entries': entries.map((e) => e.toMap()).toList(),
    };

    final dir  = await getApplicationDocumentsDirectory();
    final file = File('${dir.path}/fitmenu_log_$dateStr.json');
    await file.writeAsString(
      const JsonEncoder.withIndent('  ').convert(data),
    );
    return file;
  }

  Future<void> shareFile(File file) async {
    await Share.shareXFiles(
      [XFile(file.path)],
      text: 'Мой дневник питания FitMenu',
      subject: 'Экспорт дневника питания',
    );
  }
}

class ExportException implements Exception {
  const ExportException(this.message);
  final String message;
  @override
  String toString() => 'ExportException: $message';
}
```

</details>
