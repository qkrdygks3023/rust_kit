# 13. 실전 프로젝트 예제

## 🎯 프로젝트 기반 학습의 중요성

이론 학습만으로는 프로그래밍을 마스터할 수 없습니다. 실제 프로젝트를 통해 다음을 배울 수 있습니다:

- **실제 문제 해결 능력**
- **코드 구성 및 아키텍처 설계**
- **디버깅 및 테스트 기술**
- **프로젝트 관리 및 배포 경험**

## 📚 프로젝트 목록

### 1단계: 입문 프로젝트 (1-2주)
- 숫자 추측 게임
- 간단한 계산기
- To-Do 리스트 CLI

### 2단계: 중급 프로젝트 (2-4주)
- 파일 동기화 도구
- 간단한 웹 서버
- 데이터베이스 클라이언트

### 3단계: 고급 프로젝트 (4-8주)
- 블로그 웹 애플리케이션
- 채팅 서버
- REST API 서비스

---

## 🎮 프로젝트 1: 숫자 추측 게임

### 개요
컴퓨터가 생각한 숫자를 맞추는 간단한 CLI 게임입니다.

### 학습 목표
- 기본 Rust 문법
- 변수와 제어문
- 사용자 입력 처리
- 에러 처리

### 코드 구조
```rust
// src/main.rs
use std::io;
use rand::Rng;

fn main() {
    println!("숫자 추측 게임!");
    
    let secret_number = rand::thread_rng().gen_range(1..=100);
    
    loop {
        println!("숫자를 추측해보세요:");
        
        let mut guess = String::new();
        io::stdin()
            .read_line(&mut guess)
            .expect("입력을 읽을 수 없습니다");
        
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("숫자를 입력해주세요!");
                continue;
            }
        };
        
        match guess.cmp(&secret_number) {
            std::cmp::Ordering::Less => println!("너무 작습니다!"),
            std::cmp::Ordering::Greater => println!("너무 큽니다!"),
            std::cmp::Ordering::Equal => {
                println!("정답입니다!");
                break;
            }
        }
    }
}
```

### 확장 아이디어
- 여러 난이도 레벨
- 추측 횟수 제한
- 점수 시스템
- 재시작 기능

---

## 🧮 프로젝트 2: 간단한 계산기

### 개요
기본 사칙연산을 수행하는 CLI 계산기입니다.

### 학습 목표
- 함수 정의 및 호출
- 패턴 매칭
- 에러 처리
- 구조체 사용

### 코드 구조
```rust
// src/main.rs
use std::io;

#[derive(Debug)]
enum Operation {
    Add,
    Subtract,
    Multiply,
    Divide,
}

impl Operation {
    fn from_str(s: &str) -> Option<Self> {
        match s {
            "+" => Some(Operation::Add),
            "-" => Some(Operation::Subtract),
            "*" => Some(Operation::Multiply),
            "/" => Some(Operation::Divide),
            _ => None,
        }
    }
}

fn calculate(a: f64, b: f64, op: Operation) -> Result<f64, String> {
    match op {
        Operation::Add => Ok(a + b),
        Operation::Subtract => Ok(a - b),
        Operation::Multiply => Ok(a * b),
        Operation::Divide => {
            if b == 0.0 {
                Err("0으로 나눌 수 없습니다".to_string())
            } else {
                Ok(a / b)
            }
        }
    }
}

fn main() {
    println!("간단한 계산기");
    
    loop {
        println!("첫 번째 숫자 (끝내려면 'q' 입력):");
        let mut input = String::new();
        io::stdin().read_line(&mut input).unwrap();
        
        if input.trim() == "q" {
            break;
        }
        
        let a: f64 = match input.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("유효한 숫자가 아닙니다");
                continue;
            }
        };
        
        println!("연산자 (+, -, *, /):");
        input.clear();
        io::stdin().read_line(&mut input).unwrap();
        
        let op = match Operation::from_str(input.trim()) {
            Some(op) => op,
            None => {
                println!("유효한 연산자가 아닙니다");
                continue;
            }
        };
        
        println!("두 번째 숫자:");
        input.clear();
        io::stdin().read_line(&mut input).unwrap();
        
        let b: f64 = match input.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("유효한 숫자가 아닙니다");
                continue;
            }
        };
        
        match calculate(a, b, op) {
            Ok(result) => println!("결과: {}", result),
            Err(error) => println!("오류: {}", error),
        }
        
        println!();
    }
}
```

### 확장 아이디어
- 괄호 지원
- 변수 저장 기능
- 히스토리 기능
- 과학적 계산 기능

---

## 📝 프로젝트 3: To-Do 리스트 CLI

### 개요
명령줄에서 작업을 관리하는 To-Do 리스트 애플리케이션입니다.

### 학습 목표
- 구조체와 열거형
- 파일 I/O
- JSON 직렬화
- CLI 인자 처리

### 코드 구조
```rust
// Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
clap = { version = "4.0", features = ["derive"] }
chrono = { version = "0.4", features = ["serde"] }

// src/main.rs
use clap::{Parser, Subcommand};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::fs;
use chrono::{DateTime, Utc};

#[derive(Parser)]
#[command(name = "todo")]
#[command(about = "A simple todo list CLI")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    Add {
        title: String,
        #[arg(short, long)]
        priority: Option<String>,
    },
    List {
        #[arg(short, long)]
        all: bool,
    },
    Complete {
        id: usize,
    },
    Delete {
        id: usize,
    },
}

#[derive(Debug, Clone, Serialize, Deserialize)]
struct TodoItem {
    id: usize,
    title: String,
    completed: bool,
    priority: String,
    created_at: DateTime<Utc>,
    completed_at: Option<DateTime<Utc>>,
}

struct TodoList {
    items: HashMap<usize, TodoItem>,
    next_id: usize,
}

impl TodoList {
    fn new() -> Self {
        TodoList {
            items: HashMap::new(),
            next_id: 1,
        }
    }
    
    fn load_from_file(&mut self, filename: &str) -> Result<(), Box<dyn std::error::Error>> {
        if let Ok(content) = fs::read_to_string(filename) {
            let loaded_items: HashMap<usize, TodoItem> = serde_json::from_str(&content)?;
            self.items = loaded_items;
            self.next_id = self.items.keys().max().unwrap_or(&0) + 1;
        }
        Ok(())
    }
    
    fn save_to_file(&self, filename: &str) -> Result<(), Box<dyn std::error::Error>> {
        let content = serde_json::to_string_pretty(&self.items)?;
        fs::write(filename, content)?;
        Ok(())
    }
    
    fn add_item(&mut self, title: String, priority: Option<String>) -> usize {
        let id = self.next_id;
        let item = TodoItem {
            id,
            title,
            completed: false,
            priority: priority.unwrap_or_else(|| "normal".to_string()),
            created_at: Utc::now(),
            completed_at: None,
        };
        
        self.items.insert(id, item);
        self.next_id += 1;
        id
    }
    
    fn complete_item(&mut self, id: usize) -> Result<(), String> {
        match self.items.get_mut(&id) {
            Some(item) => {
                item.completed = true;
                item.completed_at = Some(Utc::now());
                Ok(())
            }
            None => Err(format!("ID {}를 찾을 수 없습니다", id)),
        }
    }
    
    fn delete_item(&mut self, id: usize) -> Result<(), String> {
        match self.items.remove(&id) {
            Some(_) => Ok(()),
            None => Err(format!("ID {}를 찾을 수 없습니다", id)),
        }
    }
    
    fn list_items(&self, show_all: bool) -> Vec<&TodoItem> {
        let mut items: Vec<&TodoItem> = self.items.values().collect();
        
        if !show_all {
            items.retain(|item| !item.completed);
        }
        
        items.sort_by(|a, b| {
            match (a.completed, b.completed) {
                (false, true) => std::cmp::Ordering::Less,
                (true, false) => std::cmp::Ordering::Greater,
                _ => a.created_at.cmp(&b.created_at),
            }
        });
        
        items
    }
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cli = Cli::parse();
    let mut todo_list = TodoList::new();
    
    // 기존 데이터 로드
    todo_list.load_from_file("todos.json")?;
    
    match cli.command {
        Commands::Add { title, priority } => {
            let id = todo_list.add_item(title, priority);
            println!("추가된 작업 (ID: {})", id);
        }
        Commands::List { all } => {
            let items = todo_list.list_items(all);
            
            if items.is_empty() {
                println!("작업이 없습니다");
            } else {
                println!("작업 목록:");
                for item in items {
                    let status = if item.completed { "✓" } else { "○" };
                    let completed_at = item.completed_at
                        .map(|dt| format!(" (완료: {})", dt.format("%Y-%m-%d %H:%M")))
                        .unwrap_or_default();
                    
                    println!("{} [{}] {} - {}{}",
                        status,
                        item.id,
                        item.title,
                        item.priority,
                        completed_at
                    );
                }
            }
        }
        Commands::Complete { id } => {
            match todo_list.complete_item(id) {
                Ok(_) => println!("작업 {}를 완료했습니다", id),
                Err(error) => println!("오류: {}", error),
            }
        }
        Commands::Delete { id } => {
            match todo_list.delete_item(id) {
                Ok(_) => println!("작업 {}를 삭제했습니다", id),
                Err(error) => println!("오류: {}", error),
            }
        }
    }
    
    // 변경사항 저장
    todo_list.save_to_file("todos.json")?;
    
    Ok(())
}
```

### 확장 아이디어
- 태그 시스템
- 마감일 기능
- 검색 기능
- 통계 정보

---

## 🔄 프로젝트 4: 파일 동기화 도구

### 개요
두 디렉토리 간의 파일을 동기화하는 도구입니다.

### 학습 목표
- 파일 시스템 작업
- 해시 계산
- 동시성 프로그래밍
- 진행률 표시

### 코드 구조
```rust
// Cargo.toml
[dependencies]
clap = { version = "4.0", features = ["derive"] }
sha2 = "0.10"
indicatif = "0.17"
tokio = { version = "1.0", features = ["full"] }
walkdir = "2.3"

// src/main.rs
use clap::Parser;
use sha2::{Sha256, Digest};
use std::path::{Path, PathBuf};
use std::fs;
use std::collections::HashMap;
use indicatif::{ProgressBar, ProgressStyle};
use tokio::task::JoinSet;
use walkdir::WalkDir;

#[derive(Parser)]
#[command(name = "sync")]
#[command(about = "File synchronization tool")]
struct Cli {
    /// Source directory
    source: String,
    
    /// Destination directory
    destination: String,
    
    /// Perform dry run
    #[arg(short, long)]
    dry_run: bool,
    
    /// Number of concurrent operations
    #[arg(short, long, default_value_t = 4)]
    threads: usize,
}

#[derive(Debug, Clone)]
struct FileInfo {
    path: PathBuf,
    hash: String,
    modified: std::time::SystemTime,
}

impl FileInfo {
    fn new(path: PathBuf) -> Result<Self, Box<dyn std::error::Error>> {
        let content = fs::read(&path)?;
        let hash = format!("{:x}", Sha256::digest(&content));
        let metadata = fs::metadata(&path)?;
        let modified = metadata.modified()?;
        
        Ok(FileInfo {
            path,
            hash,
            modified,
        })
    }
}

async fn scan_directory(dir: &Path) -> Result<HashMap<PathBuf, FileInfo>, Box<dyn std::error::Error>> {
    let mut files = HashMap::new();
    let mut set = JoinSet::new();
    
    for entry in WalkDir::new(dir) {
        let entry = entry?;
        let path = entry.path();
        
        if path.is_file() {
            let path = path.to_path_buf();
            set.spawn(async move {
                FileInfo::new(path)
            });
        }
    }
    
    while let Some(result) = set.join_next().await {
        let file_info = result??;
        files.insert(file_info.path.clone(), file_info);
    }
    
    Ok(files)
}

async fn sync_files(
    source_files: HashMap<PathBuf, FileInfo>,
    dest_files: HashMap<PathBuf, FileInfo>,
    dest_dir: &Path,
    dry_run: bool,
    threads: usize,
) -> Result<(), Box<dyn std::error::Error>> {
    let mut set = JoinSet::new();
    let mut total_files = 0;
    let mut copied_files = 0;
    
    // 파일 복사 작업 생성
    for (relative_path, source_info) in source_files {
        let dest_path = dest_dir.join(&relative_path);
        total_files += 1;
        
        let should_copy = match dest_files.get(&relative_path) {
            Some(dest_info) => {
                source_info.hash != dest_info.hash || source_info.modified > dest_info.modified
            }
            None => true,
        };
        
        if should_copy {
            copied_files += 1;
            
            let source_path = source_info.path.clone();
            let dest_path = dest_path.clone();
            let dry_run = dry_run;
            
            set.spawn(async move {
                if dry_run {
                    println!("복사할 파일: {} -> {}", source_path.display(), dest_path.display());
                } else {
                    // 대상 디렉토리 생성
                    if let Some(parent) = dest_path.parent() {
                        fs::create_dir_all(parent)?;
                    }
                    
                    fs::copy(&source_path, &dest_path)?;
                    println!("복사 완료: {} -> {}", source_path.display(), dest_path.display());
                }
                
                Ok::<(), Box<dyn std::error::Error>>(())
            });
        }
        
        // 동시성 제한
        if set.len() >= threads {
            set.join_next().await.unwrap()??;
        }
    }
    
    // 남은 작업 완료
    while let Some(result) = set.join_next().await {
        result??;
    }
    
    println!("동기화 완료: {}/{} 파일 복사됨", copied_files, total_files);
    
    Ok(())
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cli = Cli::parse();
    
    let source_path = Path::new(&cli.source);
    let dest_path = Path::new(&cli.destination);
    
    if !source_path.exists() {
        return Err(format!("소스 디렉토리가 존재하지 않습니다: {}", cli.source).into());
    }
    
    if !dest_path.exists() {
        fs::create_dir_all(dest_path)?;
    }
    
    println!("소스 디렉토리 스캔 중...");
    let source_files = scan_directory(source_path).await?;
    println!("대상 디렉토리 스캔 중...");
    let dest_files = scan_directory(dest_path).await?;
    
    if cli.dry_run {
        println!("드라이런 모드 - 실제 파일은 복사되지 않습니다");
    }
    
    sync_files(source_files, dest_files, dest_path, cli.dry_run, cli.threads).await?;
    
    Ok(())
}
```

### 확장 아이디어
- 양방향 동기화
- 제외 파일 패턴
- 실시간 모니터링
- 백업 기능

---

## 🌐 프로젝트 5: 간단한 웹 서버

### 개요
정적 파일을 제공하고 간단한 API 엔드포인트를 가진 웹 서버입니다.

### 학습 목표
- TCP 소켓 프로그래밍
- HTTP 프로토콜 이해
- 동시성 처리
- 라우팅 시스템

### 코드 구조
```rust
// Cargo.toml
[dependencies]
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

// src/main.rs
use std::collections::HashMap;
use std::fs;
use std::io::prelude::*;
use std::net::{TcpListener, TcpStream};
use std::path::Path;
use std::sync::{Arc, Mutex};

#[derive(Debug, Clone, Serialize, Deserialize)]
struct User {
    id: u32,
    name: String,
    email: String,
}

type UsersDb = Arc<Mutex<HashMap<u32, User>>>;

#[derive(Debug)]
enum HttpMethod {
    Get,
    Post,
    Put,
    Delete,
}

impl HttpMethod {
    fn from_str(s: &str) -> Option<Self> {
        match s {
            "GET" => Some(HttpMethod::Get),
            "POST" => Some(HttpMethod::Post),
            "PUT" => Some(HttpMethod::Put),
            "DELETE" => Some(HttpMethod::Delete),
            _ => None,
        }
    }
}

#[derive(Debug)]
struct HttpRequest {
    method: HttpMethod,
    path: String,
    headers: HashMap<String, String>,
    body: String,
}

impl HttpRequest {
    fn parse(request: &str) -> Option<Self> {
        let mut lines = request.lines();
        
        let request_line = lines.next()?;
        let mut parts = request_line.split_whitespace();
        
        let method = HttpMethod::from_str(parts.next()?)?;
        let path = parts.next()?.to_string();
        
        let mut headers = HashMap::new();
        let mut body_lines = Vec::new();
        let mut header_section = true;
        
        for line in lines {
            if line.is_empty() {
                header_section = false;
                continue;
            }
            
            if header_section {
                if let Some((key, value)) = line.split_once(':') {
                    headers.insert(key.trim().to_lowercase(), value.trim().to_string());
                }
            } else {
                body_lines.push(line);
            }
        }
        
        let body = body_lines.join("\n");
        
        Some(HttpRequest {
            method,
            path,
            headers,
            body,
        })
    }
}

fn handle_static_file(request: &HttpRequest, public_dir: &Path) -> Option<Vec<u8>> {
    let file_path = if request.path == "/" {
        public_dir.join("index.html")
    } else {
        public_dir.strip_prefix("/").ok()
            .and_then(|p| Path::new(&request.path[1..]).strip_prefix(p).ok())
            .map(|p| public_dir.join(p))
            .or_else(|| public_dir.join(&request.path[1..]))
    };
    
    if let Some(path) = file_path {
        if path.exists() && path.is_file() {
            return fs::read(&path).ok();
        }
    }
    
    None
}

fn handle_api_users_get(users_db: &UsersDb) -> Vec<u8> {
    let users = users_db.lock().unwrap();
    let users_vec: Vec<&User> = users.values().collect();
    serde_json::to_string(&users_vec).unwrap().into_bytes()
}

fn handle_api_users_post(users_db: &UsersDb, body: &str) -> Vec<u8> {
    let new_user: User = serde_json::from_str(body).unwrap();
    
    let mut users = users_db.lock().unwrap();
    let new_id = users.keys().max().unwrap_or(&0) + 1;
    
    let mut user = new_user;
    user.id = new_id;
    
    users.insert(new_id, user.clone());
    
    serde_json::to_string(&user).unwrap().into_bytes()
}

fn handle_connection(mut stream: TcpStream, users_db: UsersDb) {
    let mut buffer = [0; 1024];
    
    stream.read(&mut buffer).unwrap();
    let request = String::from_utf8_lossy(&buffer);
    
    if let Some(http_request) = HttpRequest::parse(&request) {
        let (status, content_type, content) = if http_request.path.starts_with("/api/") {
            match http_request.path.as_str() {
                "/api/users" => match http_request.method {
                    HttpMethod::Get => {
                        let content = handle_api_users_get(&users_db);
                        (200, "application/json", content)
                    }
                    HttpMethod::Post => {
                        let content = handle_api_users_post(&users_db, &http_request.body);
                        (201, "application/json", content)
                    }
                    _ => (405, "text/plain", b"Method Not Allowed".to_vec()),
                },
                _ => (404, "text/plain", b"Not Found".to_vec()),
            }
        } else {
            // 정적 파일 처리
            let public_dir = Path::new("public");
            
            if let Some(content) = handle_static_file(&http_request, public_dir) {
                let content_type = if http_request.path.ends_with(".html") {
                    "text/html"
                } else if http_request.path.ends_with(".css") {
                    "text/css"
                } else if http_request.path.ends_with(".js") {
                    "application/javascript"
                } else {
                    "text/plain"
                };
                
                (200, content_type, content)
            } else {
                (404, "text/html", include_bytes!("404.html").to_vec())
            }
        };
        
        let response = format!(
            "HTTP/1.1 {} OK\r\nContent-Type: {}\r\nContent-Length: {}\r\n\r\n",
            status, content_type, content.len()
        );
        
        stream.write(response.as_bytes()).unwrap();
        stream.write(&content).unwrap();
    }
}

#[tokio::main]
async fn main() -> std::io::Result<()> {
    let users_db = UsersDb::new(Mutex::new(HashMap::new()));
    
    // 초기 데이터
    {
        let mut users = users_db.lock().unwrap();
        users.insert(1, User {
            id: 1,
            name: "Alice".to_string(),
            email: "alice@example.com".to_string(),
        });
        users.insert(2, User {
            id: 2,
            name: "Bob".to_string(),
            email: "bob@example.com".to_string(),
        });
    }
    
    let listener = TcpListener::bind("127.0.0.1:8080")?;
    println!("서버가 http://127.0.0.1:8080 에서 실행 중");
    
    for stream in listener.incoming() {
        let stream = stream.unwrap();
        let users_db = users_db.clone();
        
        tokio::spawn(async move {
            handle_connection(stream, users_db);
        });
    }
    
    Ok(())
}
```

### 확장 아이디어
- HTTPS 지원
- 웹소켓
- 미들웨어 시스템
- 템플릿 엔진

---

## 📝 프로젝트 6: 블로그 웹 애플리케이션

### 개요
Axum을 사용한 전체 블로그 웹 애플리케이션입니다.

### 학습 목표
- 웹 프레임워크 사용
- 데이터베이스 연동
- 템플릿 렌더링
- 인증 시스템

### 코드 구조
```rust
// Cargo.toml
[dependencies]
axum = "0.6"
tokio = { version = "1.0", features = ["full"] }
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "chrono", "uuid"] }
serde = { version = "1.0", features = ["derive"] }
chrono = { version = "0.4", features = ["serde"] }
uuid = { version = "1.0", features = ["v4", "serde"] }
askama = "0.12"
tower = "0.4"
tower-http = { version = "0.4", features = ["fs", "trace"] }
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }

// templates/base.html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Rust 블로그{% endblock %}</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
        .post { border-bottom: 1px solid #eee; padding: 20px 0; }
        .post h2 { margin: 0 0 10px 0; }
        .post .meta { color: #666; font-size: 0.9em; }
        .nav { margin-bottom: 20px; }
        .nav a { margin-right: 10px; }
    </style>
</head>
<body>
    <nav class="nav">
        <a href="/">홈</a>
        <a href="/posts/new">글쓰기</a>
    </nav>
    
    {% block content %}{% endblock %}
</body>
</html>

// templates/index.html
{% extends "base.html" %}

{% block title %}Rust 블로그 - 홈{% endblock %}

{% block content %}
    <h1>Rust 블로그</h1>
    
    {% for post in posts %}
    <div class="post">
        <h2><a href="/posts/{{ post.id }}">{{ post.title }}</a></h2>
        <div class="meta">
            {{ post.created_at.format("%Y-%m-%d %H:%M") }} | 
            {{ post.comments_count }} 댓글
        </div>
        <div>{{ post.content | safe }}</div>
    </div>
    {% endfor %}
{% endblock %}

// src/main.rs
use axum::{
    extract::{Path, State},
    http::StatusCode,
    response::{Html, Redirect},
    routing::{get, post},
    Router,
};
use askama::Template;
use chrono::{DateTime, Utc};
use serde::{Deserialize, Serialize};
use sqlx::postgres::PgPool;
use std::sync::Arc;
use uuid::Uuid;

#[derive(Debug, Serialize, sqlx::FromRow)]
struct Post {
    id: Uuid,
    title: String,
    content: String,
    created_at: DateTime<Utc>,
    updated_at: DateTime<Utc>,
    comments_count: i64,
}

#[derive(Template)]
#[template(path = "index.html")]
struct IndexTemplate {
    posts: Vec<Post>,
}

#[derive(Template)]
#[template(path = "post_detail.html")]
struct PostDetailTemplate {
    post: Post,
}

#[derive(Deserialize)]
struct CreatePostForm {
    title: String,
    content: String,
}

type AppState = Arc<PgPool>;

// 홈페이지
async fn index(State(pool): State<AppState>) -> Result<IndexTemplate, StatusCode> {
    let posts = sqlx::query_as::<_, Post>(
        r#"
        SELECT p.*, COUNT(c.id) as comments_count
        FROM posts p
        LEFT JOIN comments c ON p.id = c.post_id
        GROUP BY p.id
        ORDER BY p.created_at DESC
        "#
    )
    .fetch_all(pool.as_ref())
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok(IndexTemplate { posts })
}

// 게시물 상세
async fn post_detail(
    State(pool): State<AppState>,
    Path(id): Path<Uuid>,
) -> Result<PostDetailTemplate, StatusCode> {
    let post = sqlx::query_as::<_, Post>(
        r#"
        SELECT p.*, COUNT(c.id) as comments_count
        FROM posts p
        LEFT JOIN comments c ON p.id = c.post_id
        WHERE p.id = $1
        GROUP BY p.id
        "#
    )
    .bind(id)
    .fetch_one(pool.as_ref())
    .await
    .map_err(|_| StatusCode::NOT_FOUND)?;
    
    Ok(PostDetailTemplate { post })
}

// 게시물 생성 폼
async fn new_post_form() -> Html<&'static str> {
    Html(include_str!("../templates/new_post.html"))
}

// 게시물 생성
async fn create_post(
    State(pool): State<AppState>,
    form: axum::extract::Form<CreatePostForm>,
) -> Result<Redirect, StatusCode> {
    let id = Uuid::new_v4();
    let now = Utc::now();
    
    sqlx::query(
        r#"
        INSERT INTO posts (id, title, content, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5)
        "#
    )
    .bind(id)
    .bind(&form.title)
    .bind(&form.content)
    .bind(now)
    .bind(now)
    .execute(pool.as_ref())
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok(Redirect::to("/"))
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 데이터베이스 연결
    let database_url = std::env::var("DATABASE_URL")
        .unwrap_or_else(|_| "postgres://user:password@localhost/blog".to_string());
    
    let pool = PgPool::connect(&database_url).await?;
    
    // 마이그레이션
    sqlx::migrate!("./migrations").run(&pool).await?;
    
    let app = Router::new()
        .route("/", get(index))
        .route("/posts/new", get(new_post_form))
        .route("/posts/new", post(create_post))
        .route("/posts/:id", get(post_detail))
        .nest_service("/static", tower_http::services::ServeDir::new("static"))
        .with_state(Arc::new(pool));
    
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await?;
    
    println!("블로그 서버가 http://127.0.0.1:3000 에서 실행 중");
    
    axum::serve(listener, app.into_make_service())
    .await?;
    
    Ok(())
}
```

### 확장 아이디어
- 댓글 시스템
- 검색 기능
- 태그 시스템
- 사용자 인증

---

## 🎯 프로젝트 선택 가이드

### 초급자에게 추천
1. **숫자 추측 게임** - 가장 기본적인 개념 학습
2. **간단한 계산기** - 함수와 에러 처리 연습
3. **To-Do 리스트 CLI** - 구조화된 프로그래밍

### 중급자에게 추천
1. **파일 동기화 도구** - 동시성과 파일 I/O
2. **간단한 웹 서버** - 네트워크 프로그래밍
3. **데이터베이스 클라이언트** - 외부 라이브러리 사용

### 고급자에게 추천
1. **블로그 웹 애플리케이션** - 전체 스택 개발
2. **채팅 서버** - 실시간 통신
3. **REST API 서비스** - 프로덕션 수준 애플리케이션

## 📈 프로젝트 진행 팁

### 1. 계획 단계
- 요구사항 명확히 정의
- 기능 목록 작성
- 개발 범위 설정

### 2. 개발 단계
- 작은 단위로 나누기
- 테스트 주도 개발
- 정기적인 커밋

### 3. 완성 단계
- 코드 리뷰
- 문서 작성
- 배포 준비

### 4. 학습 단계
- 코드 리팩토링
- 새로운 기능 추가
- 다른 프로젝트에 적용

## 🔗 추가 학습 자료

### 프로젝트 아이디어
- [Rust Project Ideas](https://github.com/jasonwilliams/awesome-rust#projects)
- [Build Your Own X](https://github.com/pomber/code-overview#build-your-own-x)
- [Rustlings](https://github.com/rust-lang/rustlings)

### 실전 예제
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rust Cookbook](https://rust-lang-nursery.github.io/rust-cookbook/)
- [Awesome Rust](https://github.com/rust-unofficial/awesome-rust)

---

**축하합니다!** 이제 실제 프로젝트를 통해 Rust를 마스터할 준비가 되었습니다. 🦀

**마지막 조언**: 가장 중요한 것은 꾸준함입니다. 작은 프로젝트라도 완성하는 경험이 실력을 만듭니다. 행운을 빕니다! ✨
