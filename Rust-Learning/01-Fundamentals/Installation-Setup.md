# 🛠️ Установка и настройка Rust

## 📦 Установка Rust

### Windows
```bash
# Скачайте rustup-init.exe с официального сайта
# Или используйте PowerShell:
Invoke-WebRequest -Uri "https://win.rustup.rs/" -OutFile "rustup-init.exe"
.\rustup-init.exe
```

### macOS и Linux
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### После установки
```bash
# Перезапустите терминал и проверьте установку
rustc --version
cargo --version

# Обновление Rust
rustup update
```

## 🔧 Настройка окружения

### VS Code
Рекомендуемые расширения:
- **rust-analyzer** - основное расширение для Rust
- **CodeLLDB** - отладчик
- **crates** - помощь с зависимостями

### Другие редакторы
- **IntelliJ IDEA** - плагин Rust
- **Vim/Neovim** - rust.vim, coc-rust-analyzer
- **Emacs** - rustic-mode

## 🎯 Проверка установки

Создайте тестовый проект:
```bash
cargo new hello_rust
cd hello_rust
cargo run
```

Должен появиться вывод: `Hello, world!`

## 🔗 Связанные темы
- [[Hello-World]] - Создание первой программы
- [[Cargo-Basics]] - Изучение Cargo

## 📚 Полезные ссылки
- [Официальная документация](https://doc.rust-lang.org/)
- [Rustup документация](https://rust-lang.github.io/rustup/)

#rust-setup #tools #beginner