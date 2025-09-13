# 📝 Шаблоны для создания заметок

Готовые шаблоны для быстрого создания заметок различных типов в едином стиле.

## 🎨 Доступные шаблоны

### 📚 Обучающие материалы
- [[Concept-Template]] - Для изучения новых концепций
- [[Topic-Overview-Template]] - Для обзора крупных тем
- [[Exercise-Template]] - Для практических упражнений

### 🏗️ Проекты
- [[Project-Template]] - Описание проекта
- [[Task-Template]] - Отдельная задача в проекте
- [[Code-Review-Template]] - Анализ кода

### 🎴 Обучение
- [[Flashcard-Template]] - Флеш-карточки  
- [[Error-Analysis-Template]] - Анализ ошибок компилятора
- [[Snippet-Template]] - Сниппеты кода

### 📊 Отслеживание
- [[Daily-Note-Template]] - Ежедневные записи
- [[Weekly-Review-Template]] - Еженедельные обзоры
- [[Progress-Update-Template]] - Обновления прогресса

## 🔧 Как использовать шаблоны

### В Obsidian с плагином Templater:
1. Установите плагин **Templater**
2. Настройте папку шаблонов: `Rust-Learning/Templates/`
3. Используйте горячие клавиши или команды для вставки

### Ручное копирование:
1. Откройте нужный шаблон
2. Скопируйте содержимое
3. Замените {{переменные}} на ваши значения

---

## 📚 Шаблон концепции

```markdown
# {{title}}

## 📖 Определение
{{definition}}

## 🔧 Синтаксис
```rust
{{syntax_example}}
```

## 💡 Практический пример
```rust
{{practical_example}}
```

## 🎯 Ключевые моменты
- {{key_point_1}}
- {{key_point_2}}
- {{key_point_3}}

## 🚨 Частые ошибки
### {{error_title_1}}
```rust
// ❌ Неправильно:
{{wrong_example}}

// ✅ Правильно:
{{correct_example}}
```

## 🔗 Связанные темы
- [[{{related_topic_1}}]]
- [[{{related_topic_2}}]]

## 📚 Дополнительное чтение
- [{{resource_title}}]({{resource_url}})

#{{tags}}
```

---

## 🏗️ Шаблон проекта

```markdown
# {{project_name}}

## 🎯 Цели проекта

### Изучаемые концепции
- [[{{concept_1}}]] - {{description_1}}
- [[{{concept_2}}]] - {{description_2}}

### Используемые crates
- `{{crate_1}}` - {{crate_description_1}}
- `{{crate_2}}` - {{crate_description_2}}

## 📋 Функциональность

### MVP (Minimum Viable Product)
- [ ] {{feature_1}}
- [ ] {{feature_2}}
- [ ] {{feature_3}}

### Дополнительные функции
- [ ] {{additional_feature_1}}
- [ ] {{additional_feature_2}}

## 🏗️ План разработки

### Этап 1: {{stage_1_name}}
- [[{{task_1_file}}]] - {{task_1_description}}
- [[{{task_2_file}}]] - {{task_2_description}}

### Этап 2: {{stage_2_name}}  
- [[{{task_3_file}}]] - {{task_3_description}}

## 💻 Основная структура кода

```rust
{{main_code_structure}}
```

## 🧪 Примеры использования

```bash
{{usage_example}}
```

## 🎯 Практические упражнения

### {{difficulty_1}}
1. {{exercise_1}}
2. {{exercise_2}}

## 🔍 Code Review Points
- [ ] {{review_point_1}}
- [ ] {{review_point_2}}

## 🔗 Связанные материалы
- [[{{related_material_1}}]]
- [[{{related_material_2}}]]

#project #{{difficulty}} #{{domain}}
```

---

## 🎴 Шаблон флеш-карточки

```markdown
# {{card_title}}

## {{question_type}}

{{question}}
?
{{answer}}

{{code_example_if_needed}}

#flashcard #{{category}} #{{difficulty}}
```

---

## 📊 Шаблон ежедневной записи

```markdown
# 📅 {{date}}

## 🎯 Планы на сегодня
- [ ] {{plan_1}}
- [ ] {{plan_2}}
- [ ] {{plan_3}}

## 📚 Что изучил
- **Тема**: [[{{topic}}]]
- **Время**: {{time}} часов
- **Источник**: {{source}}
- **Новые концепции**: 
  - {{concept_1}}
  - {{concept_2}}

## 💻 Практика
- **Проект**: [[{{project}}]]
- **Строк кода**: {{lines_of_code}}
- **Новые техники**: 
  - {{technique_1}}

## 🎴 Повторения
- **Флеш-карточки**: {{cards_count}} шт
- **Успешность**: {{success_rate}}%
- **Новые карточки**: {{new_cards}} шт

## 🤔 Рефлексия
**Что прошло хорошо**:
- {{good_point_1}}
- {{good_point_2}}

**Что было сложно**:
- {{challenge_1}}
- {{challenge_2}}

**Вопросы для изучения**:
- {{question_1}}

## 💡 Инсайты и заметки
- {{insight_1}}

## 📋 Планы на завтра
- [ ] {{tomorrow_plan_1}}
- [ ] {{tomorrow_plan_2}}

#journal #date-{{date_tag}} #{{main_topic}}
```

---

## 🚨 Шаблон анализа ошибки

```markdown
# {{error_code}}: {{error_title}}

**Категория**: {{category}}  
**Частота**: {{frequency_stars}} {{frequency_description}}  
**Сложность**: {{complexity_icon}} {{complexity_level}}

## 🚫 Описание ошибки
{{error_description}}

## 💥 Примеры ошибок

### {{example_title}}
```rust
{{error_example}}
```

**Сообщение компилятора:**
```
{{compiler_message}}
```

## ✅ Решения

### {{solution_number}}. {{solution_title}}
```rust
{{solution_code}}
```

{{solution_explanation}}

## 🤔 Когда происходит {{error_type}}?

### {{error_type}} происходит:
- {{condition_1}}
- {{condition_2}}

### {{error_type}} НЕ происходит:
- {{non_condition_1}}

## 🔍 Диагностика
{{diagnostic_tips}}

## 💡 Лучшие практики
{{best_practices}}

## 🎯 Практические упражнения
{{exercises}}

## 🔗 Связанные ошибки
- [[{{related_error_1}}]]

#error-{{error_code}} #{{category_tag}} #{{difficulty}}
```

---

## 🛠️ Шаблон сниппета

```markdown
# {{snippet_title}}

## 📝 Описание
{{description}}

## 💻 Код
```rust
{{code}}
```

## 🎯 Использование
{{usage_description}}

```rust
{{usage_example}}
```

## ⚡ Варианты
### {{variant_title}}
```rust
{{variant_code}}
```

## 🔗 Связанные паттерны
- [[{{related_pattern_1}}]]

#snippet #{{category}} #{{use_case}}
```

---

## 🔧 Настройка шаблонов в Obsidian

### Установка Templater
1. Откройте настройки Obsidian
2. Перейдите в Community Plugins  
3. Найдите и установите **Templater**
4. Активируйте плагин

### Настройка папки шаблонов
1. Создайте папку `Templates/` в хранилище
2. В настройках Templater укажите эту папку
3. Скопируйте шаблоны в папку

### Использование
- **Горячая клавиша**: Ctrl/Cmd + P → "Templater: Insert Template"
- **Кнопка**: Добавьте кнопку на панель инструментов
- **Автоматически**: При создании файлов в определённых папках

#templates #productivity #obsidian-setup #workflow