# 🎭 Макросы в Rust

Макросы позволяют писать код, который пишет другой код (метапрограммирование).

## 📋 Содержание

### Декларативные макросы
- [[Macro-Rules]] - macro_rules! макросы
- [[Pattern-Matching-Macros]] - Сопоставление паттернов в макросах
- [[Repetition-Patterns]] - Повторяющиеся паттерны

### Процедурные макросы
- [[Custom-Derive]] - Пользовательские #[derive] макросы
- [[Attribute-Like]] - Атрибут-подобные макросы
- [[Function-Like]] - Функция-подобные макросы

## 🎯 Декларативные макросы

### Базовый синтаксис
```rust
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

fn main() {
    say_hello!();
}
```

### Макросы с параметрами
```rust
macro_rules! create_function {
    ($func_name:ident) => {
        fn $func_name() {
            println!("You called {:?}()", stringify!($func_name));
        }
    };
}

create_function!(foo);
create_function!(bar);

fn main() {
    foo();
    bar();
}
```

### Повторяющиеся паттерны
```rust
macro_rules! find_min {
    ($x:expr) => ($x);
    ($x:expr, $($y:expr),+) => {
        std::cmp::min($x, find_min!($($y),+))
    };
}

fn main() {
    println!("{}", find_min!(1));
    println!("{}", find_min!(1 + 2, 2));
    println!("{}", find_min!(5, 2 * 3, 4));
}
```

## 🔧 Продвинутые техники

### Многие паттерны
```rust
macro_rules! calculate {
    (eval $e:expr) => {{
        {
            let val: usize = $e;
            println!("{} = {}", stringify!{$e}, val);
        }
    }};
    
    (eval $e:expr, $(eval $es:expr),+) => {{
        calculate! { eval $e }
        calculate! { $(eval $es),+ }
    }};
}

fn main() {
    calculate! {
        eval 1 + 2,
        eval 3 + 4,
        eval (2 * 3) + 1
    }
}
```

### DSL создание
```rust
macro_rules! hashmap {
    ($($key:expr => $val:expr),* $(,)?) => {{
        let mut map = ::std::collections::HashMap::new();
        $( map.insert($key, $val); )*
        map
    }};
}

fn main() {
    let map = hashmap!{
        "one" => 1,
        "two" => 2,
        "three" => 3,
    };
    println!("{:?}", map);
}
```

## 🏗️ Процедурные макросы

### Custom Derive
```rust
// В отдельном crate с proc-macro = true
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    let ast = syn::parse(input).unwrap();
    impl_hello_macro(&ast)
}

fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```

## 💡 Полезные встроенные макросы

### Отладочные макросы
```rust
fn main() {
    // Вывод с информацией о файле и строке
    println!("Debug info at {}:{}", file!(), line!());
    
    // Условная компиляция
    if cfg!(target_os = "windows") {
        println!("Running on Windows");
    } else {
        println!("Not running on Windows");
    }
    
    // Включение файлов во время компиляции
    let content = include_str!("../README.md");
    println!("README length: {}", content.len());
}
```

### Форматирование
```rust
fn main() {
    let name = "Rust";
    let version = 1.70;
    
    // format! семейство макросов
    let msg = format!("Hello, {} {}!", name, version);
    println!("{}", msg);
    
    // Отладочный вывод
    dbg!(&msg);
    
    // В stderr
    eprintln!("Error: something went wrong");
}
```

## ✅ Чек-лист изучения
- [ ] Создаю простые macro_rules! макросы
- [ ] Использую паттерны и повторения
- [ ] Понимаю разницу между декларативными и процедурными макросами
- [ ] Знаю популярные встроенные макросы

#macros #metaprogramming #code-generation #advanced