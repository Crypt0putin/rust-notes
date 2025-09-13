# 🕷️ Web Scraper - Парсер веб-страниц

Проект для изучения HTTP-клиентов, парсинга HTML, асинхронного программирования и обработки ошибок.

## 🎯 Цели проекта

### Изучаемые концепции  
- [[HTTP-Requests]] - HTTP запросы с reqwest
- [[HTML-Parsing]] - Парсинг HTML с scraper
- [[Async-Programming]] - Асинхронное программирование
- [[07-Error-Handling/Custom-Errors]] - Пользовательские типы ошибок
- [[File-IO]] - Сохранение результатов в файлы

### Используемые crates
- `reqwest` - HTTP клиент с async/await поддержкой
- `scraper` - парсинг HTML и CSS селекторы  
- `tokio` - асинхронная среда выполнения
- `serde` - сериализация данных
- `clap` - парсинг аргументов командной строки

## 📋 Функциональность

### MVP (Minimum Viable Product)
- ✅ Скачивание HTML страницы
- ✅ Извлечение данных по CSS селекторам
- ✅ Сохранение в JSON/CSV файл
- ✅ Базовая обработка ошибок

### Дополнительные функции
- 🎯 Множественные URL одновременно
- 📊 Статистика скрапинга
- 🚫 Respect robots.txt  
- ⏰ Задержки между запросами
- 🔄 Повторные попытки при ошибках
- 📱 Разные User-Agent

## 🏗️ План разработки

### Этап 1: Базовый скрапер
- [[Task-1-HTTP-Client]] - Настройка HTTP клиента
- [[Task-2-HTML-Parser]] - Парсинг HTML 
- [[Task-3-Data-Extraction]] - Извлечение данных

### Этап 2: Продвинутые функции
- [[Task-4-Async-Multiple]] - Множественные запросы
- [[Task-5-Error-Handling]] - Улучшенная обработка ошибок
- [[Task-6-CLI-Interface]] - Командный интерфейс

### Этап 3: Оптимизация
- [[Task-7-Rate-Limiting]] - Ограничение скорости
- [[Task-8-Caching]] - Кэширование результатов
- [[Task-9-Configuration]] - Конфигурационные файлы

## 💻 Основная структура кода

### Cargo.toml
```toml
[package]
name = "web_scraper"
version = "0.1.0"
edition = "2021"

[dependencies]
reqwest = { version = "0.11", features = ["json"] }
scraper = "0.17"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
csv = "1.2"
clap = { version = "4.0", features = ["derive"] }
anyhow = "1.0"
url = "2.4"
```

### Базовая структура
```rust
use reqwest::Client;
use scraper::{Html, Selector};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use anyhow::{Result, anyhow};

#[derive(Debug, Serialize, Deserialize)]
struct ScrapedData {
    url: String,
    title: Option<String>,
    data: HashMap<String, Vec<String>>,
    scraped_at: chrono::DateTime<chrono::Utc>,
}

#[derive(Debug)]
struct WebScraper {
    client: Client,
    selectors: HashMap<String, Selector>,
}

impl WebScraper {
    fn new() -> Self {
        let client = Client::builder()
            .user_agent("Web Scraper 1.0")
            .timeout(std::time::Duration::from_secs(30))
            .build()
            .expect("Failed to create HTTP client");
            
        Self {
            client,
            selectors: HashMap::new(),
        }
    }
    
    fn add_selector(&mut self, name: &str, selector: &str) -> Result<()> {
        let parsed_selector = Selector::parse(selector)
            .map_err(|e| anyhow!("Invalid CSS selector '{}': {:?}", selector, e))?;
        self.selectors.insert(name.to_string(), parsed_selector);
        Ok(())
    }
    
    async fn scrape(&self, url: &str) -> Result<ScrapedData> {
        // Скачиваем HTML
        let response = self.client.get(url).send().await?;
        let html_content = response.text().await?;
        let document = Html::parse_document(&html_content);
        
        // Извлекаем title
        let title_selector = Selector::parse("title").unwrap();
        let title = document
            .select(&title_selector)
            .next()
            .map(|element| element.text().collect::<Vec<_>>().join(" "));
        
        // Извлекаем данные по селекторам
        let mut data = HashMap::new();
        for (name, selector) in &self.selectors {
            let elements: Vec<String> = document
                .select(selector)
                .map(|element| element.text().collect::<Vec<_>>().join(" "))
                .collect();
            data.insert(name.clone(), elements);
        }
        
        Ok(ScrapedData {
            url: url.to_string(),
            title,
            data,
            scraped_at: chrono::Utc::now(),
        })
    }
}
```

## 🧪 Примеры использования

### Простой пример
```rust
#[tokio::main]
async fn main() -> Result<()> {
    let mut scraper = WebScraper::new();
    
    // Добавляем селекторы для извлечения данных
    scraper.add_selector("headings", "h1, h2, h3")?;
    scraper.add_selector("links", "a[href]")?;
    scraper.add_selector("paragraphs", "p")?;
    
    // Скрапим страницу
    let data = scraper.scrape("https://example.com").await?;
    
    // Выводим результат
    println!("Title: {:?}", data.title);
    println!("Found {} headings", data.data.get("headings").unwrap_or(&vec![]).len());
    
    // Сохраняем в JSON
    let json = serde_json::to_string_pretty(&data)?;
    std::fs::write("scraped_data.json", json)?;
    
    Ok(())
}
```

### Множественные URL
```rust
async fn scrape_multiple_urls(urls: Vec<&str>) -> Result<Vec<ScrapedData>> {
    let mut scraper = WebScraper::new();
    scraper.add_selector("title", "h1")?;
    scraper.add_selector("description", "meta[name='description']")?;
    
    // Создаём задачи для параллельного выполнения
    let tasks: Vec<_> = urls
        .into_iter()
        .map(|url| {
            let scraper = &scraper;
            async move {
                scraper.scrape(url).await
            }
        })
        .collect();
    
    // Ждём завершения всех задач
    let results = futures::future::join_all(tasks).await;
    
    // Собираем успешные результаты
    let mut scraped_data = Vec::new();
    for result in results {
        match result {
            Ok(data) => scraped_data.push(data),
            Err(e) => eprintln!("Scraping error: {}", e),
        }
    }
    
    Ok(scraped_data)
}
```

## 🚀 Расширенные функции

### 1. Настраиваемые селекторы из файла
```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Deserialize, Serialize)]
struct ScrapingConfig {
    selectors: HashMap<String, String>,
    delay_ms: Option<u64>,
    max_retries: Option<u32>,
    user_agent: Option<String>,
}

impl ScrapingConfig {
    fn load_from_file(path: &str) -> Result<Self> {
        let content = std::fs::read_to_string(path)?;
        let config: ScrapingConfig = serde_yaml::from_str(&content)?;
        Ok(config)
    }
}

// config.yaml
/*
selectors:
  title: "h1"
  price: ".price"
  description: ".description"
delay_ms: 1000
max_retries: 3
user_agent: "Custom Scraper 1.0"
*/
```

### 2. Respect robots.txt
```rust
use reqwest::Url;

struct RobotsChecker {
    client: Client,
    cache: HashMap<String, bool>,
}

impl RobotsChecker {
    async fn can_fetch(&mut self, url: &str, user_agent: &str) -> Result<bool> {
        let parsed_url = Url::parse(url)?;
        let base_url = format!("{}://{}", parsed_url.scheme(), parsed_url.host_str().unwrap());
        
        if let Some(&allowed) = self.cache.get(&base_url) {
            return Ok(allowed);
        }
        
        let robots_url = format!("{}/robots.txt", base_url);
        match self.client.get(&robots_url).send().await {
            Ok(response) => {
                if response.status().is_success() {
                    let robots_content = response.text().await?;
                    let allowed = self.parse_robots_txt(&robots_content, user_agent);
                    self.cache.insert(base_url, allowed);
                    Ok(allowed)
                } else {
                    Ok(true) // Если robots.txt недоступен, разрешаем
                }
            }
            Err(_) => Ok(true),
        }
    }
    
    fn parse_robots_txt(&self, content: &str, user_agent: &str) -> bool {
        // Упрощённый парсер robots.txt
        let lines: Vec<&str> = content.lines().collect();
        let mut current_user_agent = "";
        let mut disallowed_paths = Vec::new();
        
        for line in lines {
            let line = line.trim();
            if line.starts_with("User-agent:") {
                current_user_agent = line.split(":").nth(1).unwrap_or("").trim();
            } else if line.starts_with("Disallow:") && 
                     (current_user_agent == "*" || current_user_agent == user_agent) {
                let path = line.split(":").nth(1).unwrap_or("").trim();
                disallowed_paths.push(path);
            }
        }
        
        // Проверяем, не запрещён ли путь
        !disallowed_paths.iter().any(|&path| path == "/" || path == "/*")
    }
}
```

### 3. Rate limiting и retry логика
```rust
use std::time::{Duration, Instant};
use tokio::time::sleep;

struct RateLimitedScraper {
    scraper: WebScraper,
    last_request: Option<Instant>,
    delay: Duration,
    max_retries: u32,
}

impl RateLimitedScraper {
    fn new(delay_ms: u64, max_retries: u32) -> Self {
        Self {
            scraper: WebScraper::new(),
            last_request: None,
            delay: Duration::from_millis(delay_ms),
            max_retries,
        }
    }
    
    async fn scrape_with_retry(&mut self, url: &str) -> Result<ScrapedData> {
        // Rate limiting
        if let Some(last) = self.last_request {
            let elapsed = last.elapsed();
            if elapsed < self.delay {
                sleep(self.delay - elapsed).await;
            }
        }
        
        // Retry logic
        let mut attempts = 0;
        loop {
            attempts += 1;
            self.last_request = Some(Instant::now());
            
            match self.scraper.scrape(url).await {
                Ok(data) => return Ok(data),
                Err(e) => {
                    if attempts >= self.max_retries {
                        return Err(e);
                    }
                    eprintln!("Attempt {} failed, retrying in {:?}...", attempts, self.delay);
                    sleep(self.delay * attempts).await; // Exponential backoff
                }
            }
        }
    }
}
```

## 🎯 CLI интерфейс

```rust
use clap::{Args, Parser, Subcommand};

#[derive(Parser)]
#[command(name = "web_scraper")]
#[command(about = "A web scraping tool built with Rust")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// Scrape a single URL
    Single(SingleArgs),
    /// Scrape multiple URLs from a file
    Batch(BatchArgs),
    /// Test CSS selectors on a page
    Test(TestArgs),
}

#[derive(Args)]
struct SingleArgs {
    /// URL to scrape
    url: String,
    /// CSS selector to extract data
    #[arg(short, long)]
    selector: String,
    /// Output file path
    #[arg(short, long, default_value = "output.json")]
    output: String,
}

#[derive(Args)]
struct BatchArgs {
    /// File with URLs (one per line)
    #[arg(short, long)]
    urls_file: String,
    /// Configuration file
    #[arg(short, long)]
    config: String,
    /// Output directory
    #[arg(short, long, default_value = "output")]
    output_dir: String,
}

#[tokio::main]
async fn main() -> Result<()> {
    let cli = Cli::parse();
    
    match cli.command {
        Commands::Single(args) => {
            let mut scraper = WebScraper::new();
            scraper.add_selector("data", &args.selector)?;
            
            let result = scraper.scrape(&args.url).await?;
            let json = serde_json::to_string_pretty(&result)?;
            std::fs::write(&args.output, json)?;
            
            println!("Scraped data saved to {}", args.output);
        }
        Commands::Batch(args) => {
            // Реализация батч-режима
            println!("Batch scraping not implemented yet");
        }
        Commands::Test(args) => {
            // Реализация тестового режима
            println!("Test mode not implemented yet");
        }
    }
    
    Ok(())
}
```

## 🧪 Тестирование

### Модульные тесты
```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tokio_test;

    #[tokio::test]
    async fn test_scraper_creation() {
        let scraper = WebScraper::new();
        assert_eq!(scraper.selectors.len(), 0);
    }

    #[tokio::test]
    async fn test_add_selector() {
        let mut scraper = WebScraper::new();
        assert!(scraper.add_selector("test", "h1").is_ok());
        assert!(scraper.add_selector("invalid", ">>invalid<<").is_err());
    }

    #[test]
    fn test_scraped_data_serialization() {
        let data = ScrapedData {
            url: "https://example.com".to_string(),
            title: Some("Test Title".to_string()),
            data: HashMap::new(),
            scraped_at: chrono::Utc::now(),
        };
        
        let json = serde_json::to_string(&data).unwrap();
        let _deserialized: ScrapedData = serde_json::from_str(&json).unwrap();
    }
}
```

## 🎯 Практические упражнения

### Начальный уровень
1. **Скрапер новостей**: Извлекайте заголовки новостей с новостного сайта
2. **Проверка цен**: Мониторьте цены товаров в интернет-магазине  
3. **Сбор ссылок**: Извлекайте все ссылки со страницы

### Средний уровень
1. **Многостраничный скрапинг**: Переходите по пагинации
2. **Обработка форм**: Заполняйте и отправляйте формы
3. **Извлечение изображений**: Скачивайте изображения со страницы

### Продвинутый уровень
1. **JavaScript рендеринг**: Используйте headless браузер для SPA
2. **Обход защиты**: Работа с CAPTCHA и анти-бот системами
3. **Распределённый скрапинг**: Координация между несколькими процессами

## ⚠️ Этические соображения

### Лучшие практики
- **Уважайте robots.txt**
- **Не перегружайте сервера** - используйте задержки
- **Соблюдайте Terms of Service**
- **Кэшируйте результаты** для минимизации запросов

### Правовые аспекты
- Проверяйте законность скрапинга в вашей юрисдикции
- Не скрапьте авторские материалы без разрешения
- Учитывайте GDPR при работе с персональными данными

## 🔗 Связанные материалы
- [[Async-Programming]] - Асинхронное программирование
- [[HTTP-Client-Patterns]] - Паттерны HTTP клиентов
- [[Code-Snippets/Error-Handling-Patterns]] - Обработка ошибок

#project #web-scraping #async #http #intermediate #automation