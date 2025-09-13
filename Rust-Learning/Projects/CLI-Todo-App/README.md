# 📋 CLI Todo Application

Консольное приложение для управления списком задач - отличный первый проект для изучения Rust!

## 🎯 Цели проекта

### Изучаемые концепции
- [[02-Ownership/Ownership-Rules]] - Управление владением данными
- [[05-Structs-Enums/Struct-Definition]] - Создание пользовательских типов
- [[07-Error-Handling/Result-Type]] - Обработка ошибок
- [[File-IO]] - Работа с файлами
- [[Command-Line-Parsing]] - Парсинг аргументов командной строки

### Используемые crates
- `clap` - парсинг аргументов командной строки
- `serde` - сериализация в JSON
- `chrono` - работа с датами

## 📋 Функциональность

### MVP (Minimum Viable Product)
- ✅ Добавление новых задач
- ✅ Просмотр списка задач
- ✅ Отметка задач как выполненных
- ✅ Удаление задач
- ✅ Сохранение в файл

### Дополнительные функции
- 🎯 Приоритеты задач
- 📅 Дедлайны
- 🏷️ Теги и категории
- 🔍 Поиск и фильтрация
- 📊 Статистика

## 🏗️ План разработки

### Этап 1: Базовая структура
- [[Task-1-Define-Structs]] - Определить структуры Task и TodoList
- [[Task-2-Basic-Operations]] - Реализовать CRUD операции
- [[Task-3-CLI-Interface]] - Создать командный интерфейс

### Этап 2: Персистентность
- [[Task-4-File-Storage]] - Сохранение и загрузка из файла
- [[Task-5-Error-Handling]] - Обработка ошибок файловых операций
- [[Task-6-Data-Validation]] - Валидация данных

### Этап 3: Улучшения
- [[Task-7-Pretty-Output]] - Красивый вывод
- [[Task-8-Advanced-Features]] - Дополнительные возможности
- [[Task-9-Tests]] - Написание тестов

## 💻 Основная структура кода

### Структуры данных
```rust
use chrono::{DateTime, Utc};
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct Task {
    pub id: u32,
    pub title: String,
    pub description: Option<String>,
    pub completed: bool,
    pub created_at: DateTime<Utc>,
    pub priority: Priority,
}

#[derive(Debug, Serialize, Deserialize, Clone)]
pub enum Priority {
    Low,
    Medium,
    High,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct TodoList {
    tasks: Vec<Task>,
    next_id: u32,
}
```

### CLI интерфейс
```rust
use clap::{Arg, Command, SubCommand};

pub fn build_cli() -> Command<'static> {
    Command::new("todo")
        .about("A simple CLI todo application")
        .version("1.0")
        .subcommand(
            Command::new("add")
                .about("Add a new task")
                .arg(Arg::new("title").required(true))
        )
        .subcommand(
            Command::new("list")
                .about("List all tasks")
        )
        // ... другие команды
}
```

## 🧪 Примеры использования

```bash
# Добавить новую задачу
todo add "Изучить Rust ownership"

# Показать все задачи
todo list

# Отметить задачу как выполненную
todo complete 1

# Удалить задачу
todo remove 1

# Показать только невыполненные
todo list --status pending
```

## 🎯 Практические упражнения

### Начальный уровень
1. **Создайте структуру Task** с базовыми полями
2. **Реализуйте методы** для создания и отображения задач
3. **Добавьте простой CLI** с одной командой

### Средний уровень
1. **Добавьте сериализацию** с помощью serde
2. **Реализуйте сохранение** в JSON файл
3. **Обработайте ошибки** с помощью Result

### Продвинутый уровень
1. **Добавьте фильтрацию** по различным критериям
2. **Создайте систему тегов** для категоризации
3. **Реализуйте экспорт** в разные форматы

## 🔍 Code Review Points

### Что проверять
- [ ] Правильное использование владения и заимствования
- [ ] Обработка всех возможных ошибок
- [ ] Валидация пользовательского ввода
- [ ] Читаемость и структурированность кода
- [ ] Наличие документации

### Типичные ошибки
- Неправильное управление владением
- Отсутствие обработки ошибок
- Небезопасный unwrap() вместо pattern matching
- Дублирование кода

## 🚀 Возможные улучшения

### Функциональные
- Подзадачи (subtasks)
- Повторяющиеся задачи
- Напоминания
- Интеграция с календарём

### Технические  
- Конфигурационный файл
- Цветной вывод
- Автодополнение в shell
- Веб-интерфейс

## 🔗 Связанные материалы
- [[Code-Snippets/CLI-Patterns]] - Паттерны для CLI приложений
- [[Code-Snippets/Error-Handling-Patterns]] - Обработка ошибок
- [[Resources/Crates-CLI]] - Полезные crates для CLI

## 📚 Дополнительное чтение
- [Clap Documentation](https://docs.rs/clap/)
- [Serde Guide](https://serde.rs/)
- [Command Line Apps in Rust](https://rust-cli.github.io/book/)

#project #cli #beginner #todo-app #practical