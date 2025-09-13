# 📋 Code Snippets Library

## 🔧 Utility Functions

### Read file to string
```rust
use std::fs;
use std::io::Result;

fn read_file(path: &str) -> Result<String> {
    fs::read_to_string(path)
}

// With error handling
fn read_file_safe(path: &str) -> Option<String> {
    fs::read_to_string(path).ok()
}
```

### Parse JSON
```rust
use serde::{Deserialize, Serialize};
use serde_json::Result;

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
}

fn parse_json(json_str: &str) -> Result<Person> {
    serde_json::from_str(json_str)
}

fn to_json(person: &Person) -> Result<String> {
    serde_json::to_string_pretty(person)
}
```

### Command line arguments
```rust
use std::env;

fn get_args() -> Vec<String> {
    env::args().collect()
}

// With clap
use clap::Parser;

#[derive(Parser, Debug)]
#[command(author, version, about, long_about = None)]
struct Args {
    /// Name of the person to greet
    #[arg(short, long)]
    name: String,

    /// Number of times to greet
    #[arg(short, long, default_value_t = 1)]
    count: u8,
}
```

## 🌐 Web & Networking

### HTTP Request with reqwest
```rust
use reqwest;
use tokio;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // GET request
    let response = reqwest::get("https://api.github.com/users/rust-lang")
        .await?
        .text()
        .await?;
    
    // POST request with JSON
    let client = reqwest::Client::new();
    let response = client
        .post("https://httpbin.org/post")
        .json(&serde_json::json!({
            "name": "Rust",
            "type": "programming language"
        }))
        .send()
        .await?;
    
    Ok(())
}
```

### Simple TCP Server
```rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};

fn handle_client(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();
    
    let response = "HTTP/1.1 200 OK\r\n\r\nHello, World!";
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    
    for stream in listener.incoming() {
        let stream = stream.unwrap();
        handle_client(stream);
    }
}
```

## 🔄 Concurrency Patterns

### Thread Pool
```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;

struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
        
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        
        let mut workers = Vec::with_capacity(size);
        
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
        
        ThreadPool { workers, sender }
    }
    
    fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.send(job).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
            println!("Worker {} got a job; executing.", id);
            job();
        });
        
        Worker { id, thread }
    }
}
```

### Channel communication
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn channel_example() {
    let (tx, rx) = mpsc::channel();
    
    // Multiple producers
    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec!["hi", "from", "the", "thread"];
        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_millis(100));
        }
    });
    
    thread::spawn(move || {
        let vals = vec!["more", "messages"];
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_millis(100));
        }
    });
    
    // Single consumer
    for received in rx {
        println!("Got: {}", received);
    }
}
```

## 📊 Data Structures

### Binary Tree
```rust
#[derive(Debug)]
struct TreeNode<T> {
    value: T,
    left: Option<Box<TreeNode<T>>>,
    right: Option<Box<TreeNode<T>>>,
}

impl<T> TreeNode<T> {
    fn new(value: T) -> Self {
        TreeNode {
            value,
            left: None,
            right: None,
        }
    }
    
    fn insert_left(&mut self, value: T) {
        self.left = Some(Box::new(TreeNode::new(value)));
    }
    
    fn insert_right(&mut self, value: T) {
        self.right = Some(Box::new(TreeNode::new(value)));
    }
}

// In-order traversal
fn inorder<T: std::fmt::Display>(node: &Option<Box<TreeNode<T>>>) {
    if let Some(n) = node {
        inorder(&n.left);
        print!("{} ", n.value);
        inorder(&n.right);
    }
}
```

### Custom Iterator
```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}

// Usage
let counter = Counter::new();
let sum: u32 = counter.sum();
```

## 🎨 Macros

### Debug print macro
```rust
#[macro_export]
macro_rules! debug_print {
    ($($arg:tt)*) => {
        #[cfg(debug_assertions)]
        {
            println!("[DEBUG] {}", format!($($arg)*));
        }
    };
}

// Usage
debug_print!("Value: {}", 42);
```

### HashMap literal
```rust
#[macro_export]
macro_rules! hashmap {
    ($($key:expr => $val:expr),* $(,)?) => {{
        let mut map = std::collections::HashMap::new();
        $(map.insert($key, $val);)*
        map
    }};
}

// Usage
let map = hashmap! {
    "one" => 1,
    "two" => 2,
    "three" => 3,
};
```

### Measure time
```rust
#[macro_export]
macro_rules! measure_time {
    ($expr:expr) => {{
        let start = std::time::Instant::now();
        let result = $expr;
        let duration = start.elapsed();
        println!("Time elapsed: {:?}", duration);
        result
    }};
}

// Usage
let result = measure_time!({
    // Some expensive computation
    std::thread::sleep(std::time::Duration::from_secs(1));
    42
});
```

## ⚡ Async Patterns

### Retry with exponential backoff
```rust
use std::time::Duration;
use tokio::time::sleep;

async fn retry_async<F, Fut, T, E>(
    mut f: F,
    max_retries: u32,
) -> Result<T, E>
where
    F: FnMut() -> Fut,
    Fut: std::future::Future<Output = Result<T, E>>,
    E: std::fmt::Debug,
{
    let mut retries = 0;
    let mut delay = Duration::from_millis(100);
    
    loop {
        match f().await {
            Ok(val) => return Ok(val),
            Err(e) if retries >= max_retries => {
                eprintln!("Max retries reached: {:?}", e);
                return Err(e);
            }
            Err(e) => {
                eprintln!("Attempt {} failed: {:?}", retries + 1, e);
                sleep(delay).await;
                delay = delay.saturating_mul(2);
                retries += 1;
            }
        }
    }
}
```

### Concurrent futures
```rust
use futures::future::join_all;

async fn process_batch<T>(items: Vec<T>) -> Vec<Result<String, Error>>
where
    T: AsRef<str> + Send + 'static,
{
    let futures = items
        .into_iter()
        .map(|item| async move {
            // Process each item
            process_single(item).await
        });
    
    join_all(futures).await
}
```

## 🔒 Error Handling

### Custom Error Type
```rust
use std::fmt;

#[derive(Debug)]
enum MyError {
    Io(std::io::Error),
    Parse(std::num::ParseIntError),
    Custom(String),
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            MyError::Io(e) => write!(f, "IO error: {}", e),
            MyError::Parse(e) => write!(f, "Parse error: {}", e),
            MyError::Custom(s) => write!(f, "Custom error: {}", s),
        }
    }
}

impl std::error::Error for MyError {}

// Conversions
impl From<std::io::Error> for MyError {
    fn from(e: std::io::Error) -> Self {
        MyError::Io(e)
    }
}

impl From<std::num::ParseIntError> for MyError {
    fn from(e: std::num::ParseIntError) -> Self {
        MyError::Parse(e)
    }
}
```

### Result chain
```rust
fn process_data(input: &str) -> Result<i32, Box<dyn std::error::Error>> {
    input
        .trim()
        .parse::<i32>()
        .map(|n| n * 2)
        .map_err(|e| e.into())
}

// With anyhow
use anyhow::{Context, Result};

fn process_with_context(path: &str) -> Result<String> {
    std::fs::read_to_string(path)
        .context("Failed to read file")?
        .lines()
        .next()
        .context("File is empty")
        .map(|s| s.to_string())
}
```

## 🧪 Testing Utilities

### Test with temp files
```rust
#[cfg(test)]
mod tests {
    use tempfile::NamedTempFile;
    use std::io::Write;
    
    #[test]
    fn test_with_temp_file() {
        let mut file = NamedTempFile::new().unwrap();
        writeln!(file, "test content").unwrap();
        
        let path = file.path();
        // Test with the temp file
        // File is automatically deleted when dropped
    }
}
```

### Parameterized tests
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn test_multiple_cases() {
        let test_cases = vec![
            (2, 2, 4),
            (3, 3, 9),
            (4, 4, 16),
        ];
        
        for (a, b, expected) in test_cases {
            assert_eq!(a * b, expected, 
                      "Failed for {} * {}", a, b);
        }
    }
}
```

## 🔗 Useful Links

- [[01_Core/00_Index|Core Concepts]]
- [[02_Advanced/00_Index|Advanced Topics]]
- [[03_Projects/00_Index|Projects]]
- [[Rust Cheatsheet|Cheatsheet]]

---
#rust #snippets #code #examples #patterns
