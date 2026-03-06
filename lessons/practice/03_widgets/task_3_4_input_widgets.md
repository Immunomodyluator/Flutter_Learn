# 3.4: Виджеты ввода

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 3 — Основы виджетов

---

### 3.4: Виджеты ввода

🎯 **Цель шага:** Создать `AddRecipeScreen` — форму добавления рецепта с валидацией, числовым вводом, выпадающим списком категорий и корректной обработкой фокуса между полями.

📝 **Техническое задание:**

Реализуй форму с полями:

1. `TextFormField` «Название» — обязательное, минимум 3 символа, `textCapitalization.sentences`
2. `TextFormField` «Калории» — `keyboardType: TextInputType.number`, `FilteringTextInputFormatter.digitsOnly`, диапазон 1–9999
3. `DropdownButtonFormField<String>` «Категория» — 4 варианта
4. `TextFormField` «Описание» — опциональное, `maxLines: 3`
5. Кнопка «Сохранить» — при нажатии: `form.validate()`, симуляция сохранения через `Future.delayed`, затем `form.reset()`
6. `TextInputAction.next` на всех полях кроме последнего (`done`)

✅ **Критерии приёмки:**
- [ ] `GlobalKey<FormState>` используется для `validate()` и `reset()`
- [ ] `FilteringTextInputFormatter.digitsOnly` — поле калорий принимает только цифры
- [ ] При нажатии «Сохранить» с пустыми полями — показываются сообщения об ошибках
- [ ] После успешного сохранения поля сбрасываются
- [ ] `TextEditingController` утилизируется в `dispose()`

💡 **Подсказка:** `FocusScope.of(context).nextFocus()` в `onFieldSubmitted` позволяет переходить к следующему полю по кнопке «Next» на клавиатуре — лучший UX для форм.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class AddRecipeScreen extends StatefulWidget {
  const AddRecipeScreen({super.key});
  @override State<AddRecipeScreen> createState() => _State();
}

class _State extends State<AddRecipeScreen> {
  final _formKey   = GlobalKey<FormState>();
  final _titleCtrl = TextEditingController();
  final _calCtrl   = TextEditingController();
  final _descCtrl  = TextEditingController();
  String _category = 'завтрак';
  bool _saving     = false;

  @override
  void dispose() {
    _titleCtrl.dispose();
    _calCtrl.dispose();
    _descCtrl.dispose();
    super.dispose();
  }

  Future<void> _save() async {
    if (!_formKey.currentState!.validate()) return;
    setState(() => _saving = true);
    await Future.delayed(const Duration(milliseconds: 600));
    if (!mounted) return;
    setState(() => _saving = false);
    _formKey.currentState!.reset();
    _titleCtrl.clear();
    _calCtrl.clear();
    _descCtrl.clear();
    if (mounted) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Рецепт сохранён ✅')));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Новый рецепт')),
      body: Form(
        key: _formKey,
        child: ListView(padding: const EdgeInsets.all(16), children: [
          TextFormField(
            controller: _titleCtrl,
            decoration: const InputDecoration(
              labelText: 'Название *',
              prefixIcon: Icon(Icons.restaurant),
              border: OutlineInputBorder(),
            ),
            textCapitalization: TextCapitalization.sentences,
            textInputAction: TextInputAction.next,
            onFieldSubmitted: (_) => FocusScope.of(context).nextFocus(),
            validator: (v) {
              if (v == null || v.trim().isEmpty) return 'Введите название';
              if (v.trim().length < 3) return 'Минимум 3 символа';
              return null;
            },
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _calCtrl,
            decoration: const InputDecoration(
              labelText: 'Калории (ккал) *',
              prefixIcon: Icon(Icons.local_fire_department),
              suffixText: 'ккал',
              border: OutlineInputBorder(),
            ),
            keyboardType: TextInputType.number,
            textInputAction: TextInputAction.next,
            inputFormatters: [FilteringTextInputFormatter.digitsOnly],
            onFieldSubmitted: (_) => FocusScope.of(context).nextFocus(),
            validator: (v) {
              if (v == null || v.isEmpty) return 'Укажите калорийность';
              final n = int.tryParse(v);
              if (n == null || n < 1 || n > 9999) return 'Введите число 1–9999';
              return null;
            },
          ),
          const SizedBox(height: 16),
          DropdownButtonFormField<String>(
            value: _category,
            decoration: const InputDecoration(
              labelText: 'Категория',
              prefixIcon: Icon(Icons.category),
              border: OutlineInputBorder(),
            ),
            items: const [
              DropdownMenuItem(value: 'завтрак', child: Text('🥞 Завтрак')),
              DropdownMenuItem(value: 'обед',    child: Text('🥗 Обед')),
              DropdownMenuItem(value: 'ужин',    child: Text('🌙 Ужин')),
              DropdownMenuItem(value: 'перекус', child: Text('🍎 Перекус')),
            ],
            onChanged: (v) => setState(() => _category = v!),
            validator: (v) => v == null ? 'Выберите категорию' : null,
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _descCtrl,
            decoration: const InputDecoration(
              labelText: 'Описание',
              prefixIcon: Icon(Icons.description),
              border: OutlineInputBorder(),
              alignLabelWithHint: true,
            ),
            maxLines: 3,
            textInputAction: TextInputAction.done,
          ),
          const SizedBox(height: 24),
          FilledButton(
            onPressed: _saving ? null : _save,
            style: FilledButton.styleFrom(
                padding: const EdgeInsets.symmetric(vertical: 16)),
            child: _saving
                ? const SizedBox(width: 20, height: 20,
                    child: CircularProgressIndicator(strokeWidth: 2,
                        color: Colors.white))
                : const Text('Сохранить', style: TextStyle(fontSize: 16)),
          ),
        ]),
      ),
    );
  }
}
```

</details>
