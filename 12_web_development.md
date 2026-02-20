# 12. Rust 웹 개발 완벽 가이드

## 🌐 웹 개발 개요

Rust은 웹 개발에서 점점 더 인기를 얻고 있으며, 다음과 같은 장점을 제공합니다:

- **고성능**: C++ 수준의 실행 속도
- **메모리 안전**: 메모리 누수나 버퍼 오버플로우 없음
- **동시성**: 안전한 동시성 프로그래밍
- **타입 안전**: 컴파일 타임에 많은 오류 방지
- **생태계**: 성장하는 웹 프레임워크 생태계

## 🚀 기본 HTTP 서버

### 1.1 표준 라이브러리로 서버 구현

```rust
use std::io::prelude::*;
use std::net::{TcpListener, TcpStream};
use std::thread;

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    
    // 요청 읽기
    stream.read(&mut buffer).unwrap();
    
    // HTTP 응답 생성
    let response = "HTTP/1.1 200 OK\r\n\r\nHello, World!";
    
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    println!("서버가 http://127.0.0.1:7878 에서 실행 중");
    
    for stream in listener.incoming() {
        let stream = stream.unwrap();
        
        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
```

### 1.2 라우팅 기능 추가

```rust
use std::io::prelude::*;
use std::net::{TcpListener, TcpStream};
use std::collections::HashMap;

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();
    
    let request = String::from_utf8_lossy(&buffer[..]);
    let request_line = request.lines().next().unwrap_or("");
    
    let (status, content) = match request_line {
        "GET / HTTP/1.1" => ("200 OK", "홈페이지"),
        "GET /about HTTP/1.1" => ("200 OK", "소개 페이지"),
        "GET /contact HTTP/1.1" => ("200 OK", "연락처 페이지"),
        _ => ("404 NOT FOUND", "페이지를 찾을 수 없습니다"),
    };
    
    let response = format!(
        "HTTP/1.1 {}\r\nContent-Type: text/html; charset=utf-8\r\n\r\n\
        <!DOCTYPE html>\
        <html>\
        <head><title>Rust 웹 서버</title></head>\
        <body>\
        <h1>{}</h1>\
        </body>\
        </html>",
        status, content
    );
    
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    println!("서버가 http://127.0.0.1:7878 에서 실행 중");
    
    for stream in listener.incoming() {
        let stream = stream.unwrap();
        thread::spawn(|| handle_connection(stream));
    }
}
```

## 🦀 Axum 프레임워크

### 2.1 Axum 기본 설정

```toml
# Cargo.toml
[dependencies]
axum = "0.6"
tokio = { version = "1.0", features = ["full"] }
tower = "0.4"
tower-http = { version = "0.4", features = ["fs", "trace"] }
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

```rust
use axum::{
    response::Html,
    routing::get,
    Router,
    extract::{Path, Query},
    http::StatusCode,
};
use serde::Deserialize;
use std::collections::HashMap;

#[derive(Deserialize)]
struct HelloParams {
    name: Option<String>,
    age: Option<u32>,
}

async fn hello() -> Html<&'static str> {
    Html("<h1>Hello, Axum!</h1>")
}

async fn hello_params(Query(params): Query<HelloParams>) -> String {
    let name = params.name.as_deref().unwrap_or("World");
    let age = params.age.map(|a| format!(" (나이: {})", a)).unwrap_or_default();
    format!("Hello, {}{}!", name, age)
}

async fn hello_name(Path(name): Path<String>) -> String {
    format!("Hello, {}!", name)
}

async fn user_info(Path(id): Path<u32>) -> String {
    match id {
        1 => "사용자: 관리자".to_string(),
        2 => "사용자: 일반 사용자".to_string(),
        _ => "사용자를 찾을 수 없습니다".to_string(),
    }
}

#[tokio::main]
async fn main() {
    // 로깅 설정
    tracing_subscriber::fmt::init();
    
    // 앱 생성
    let app = Router::new()
        .route("/", get(hello))
        .route("/hello", get(hello_params))
        .route("/hello/:name", get(hello_name))
        .route("/user/:id", get(user_info));
    
    // 서버 실행
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();
    
    println!("서버가 http://127.0.0.1:3000 에서 실행 중");
    
    axum::serve(listener, app.into_make_service())
        .await
        .unwrap();
}
```

### 2.2 JSON API 구현

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    response::Json,
    routing::{get, post, put, delete},
    Router,
};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::sync::{Arc, Mutex};

#[derive(Debug, Clone, Serialize, Deserialize)]
struct User {
    id: u32,
    name: String,
    email: String,
}

#[derive(Debug, Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

#[derive(Debug, Deserialize)]
struct UpdateUser {
    name: Option<String>,
    email: Option<String>,
}

type AppState = Arc<Mutex<HashMap<u32, User>>>;

// 모든 사용자 조회
async fn get_users(State(state): State<AppState>) -> Json<Vec<User>> {
    let users = state.lock().unwrap();
    let user_list: Vec<User> = users.values().cloned().collect();
    Json(user_list)
}

// 특정 사용자 조회
async fn get_user(
    State(state): State<AppState>,
    Path(id): Path<u32>,
) -> Result<Json<User>, StatusCode> {
    let users = state.lock().unwrap();
    
    match users.get(&id) {
        Some(user) => Ok(Json(user.clone())),
        None => Err(StatusCode::NOT_FOUND),
    }
}

// 사용자 생성
async fn create_user(
    State(state): State<AppState>,
    Json(new_user): Json<CreateUser>,
) -> Result<(StatusCode, Json<User>), StatusCode> {
    let mut users = state.lock().unwrap();
    
    // 새 ID 생성
    let new_id = users.keys().max().unwrap_or(&0) + 1;
    
    let user = User {
        id: new_id,
        name: new_user.name,
        email: new_user.email,
    };
    
    users.insert(new_id, user.clone());
    
    Ok((StatusCode::CREATED, Json(user)))
}

// 사용자 업데이트
async fn update_user(
    State(state): State<AppState>,
    Path(id): Path<u32>,
    Json(update_user): Json<UpdateUser>,
) -> Result<Json<User>, StatusCode> {
    let mut users = state.lock().unwrap();
    
    match users.get_mut(&id) {
        Some(user) => {
            if let Some(name) = update_user.name {
                user.name = name;
            }
            if let Some(email) = update_user.email {
                user.email = email;
            }
            Ok(Json(user.clone()))
        }
        None => Err(StatusCode::NOT_FOUND),
    }
}

// 사용자 삭제
async fn delete_user(
    State(state): State<AppState>,
    Path(id): Path<u32>,
) -> StatusCode {
    let mut users = state.lock().unwrap();
    
    match users.remove(&id) {
        Some(_) => StatusCode::NO_CONTENT,
        None => StatusCode::NOT_FOUND,
    }
}

#[tokio::main]
async fn main() {
    // 초기 데이터
    let initial_users = HashMap::from([
        (1, User {
            id: 1,
            name: "Alice".to_string(),
            email: "alice@example.com".to_string(),
        }),
        (2, User {
            id: 2,
            name: "Bob".to_string(),
            email: "bob@example.com".to_string(),
        }),
    ]);
    
    let state = Arc::new(Mutex::new(initial_users));
    
    let app = Router::new()
        .route("/users", get(get_users).post(create_user))
        .route("/users/:id", get(get_user).put(update_user).delete(delete_user))
        .with_state(state);
    
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();
    
    println!("API 서버가 http://127.0.0.1:3000 에서 실행 중");
    println!("엔드포인트:");
    println!("  GET    /users        - 모든 사용자 조회");
    println!("  POST   /users        - 사용자 생성");
    println!("  GET    /users/:id    - 특정 사용자 조회");
    println!("  PUT    /users/:id    - 사용자 업데이트");
    println!("  DELETE /users/:id    - 사용자 삭제");
    
    axum::serve(listener, app.into_make_service())
        .await
        .unwrap();
}
```

### 2.3 미들웨어 구현

```rust
use axum::{
    extract::{Request, State},
    http::{StatusCode, HeaderMap},
    middleware::{self, Next},
    response::Response,
    Router,
    routing::get,
};
use std::time::{Duration, Instant};
use std::sync::Arc;
use tokio::sync::RwLock;

#[derive(Clone)]
struct AppState {
    request_count: Arc<RwLock<usize>>,
}

// 로깅 미들웨어
async fn logging_middleware(
    request: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    let start = Instant::now();
    let method = request.method().clone();
    let uri = request.uri().clone();
    
    let response = next.run(request).await;
    
    let duration = start.elapsed();
    println!(
        "{} {} {} - {}ms",
        response.status(),
        method,
        uri,
        duration.as_millis()
    );
    
    Ok(response)
}

// 요청 카운터 미들웨어
async fn request_counter_middleware(
    State(state): State<AppState>,
    request: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    // 카운터 증가
    {
        let mut count = state.request_count.write().await;
        *count += 1;
    }
    
    // 현재 카운터를 헤더에 추가
    let mut response = next.run(request).await;
    
    let count = state.request_count.read().await;
    response.headers_mut().insert(
        "X-Request-Count",
        count.to_string().parse().unwrap(),
    );
    
    Ok(response)
}

// 인증 미들웨어
async fn auth_middleware(
    headers: HeaderMap,
    request: Request,
    next: Next,
) -> Result<Response, StatusCode> {
    // 간단한 API 키 인증
    if let Some(api_key) = headers.get("X-API-Key") {
        if api_key == "secret-key" {
            Ok(next.run(request).await)
        } else {
            Err(StatusCode::UNAUTHORIZED)
        }
    } else {
        Err(StatusCode::UNAUTHORIZED)
    }
}

async fn public_handler() -> &'static str {
    "공용 엔드포인트"
}

async fn protected_handler() -> &'static str {
    "보호된 엔드포인트"
}

async fn stats_handler(State(state): State<AppState>) -> String {
    let count = state.request_count.read().await;
    format!("총 요청 수: {}", *count)
}

#[tokio::main]
async fn main() {
    let state = AppState {
        request_count: Arc::new(RwLock::new(0)),
    };
    
    let app = Router::new()
        .route("/public", get(public_handler))
        .route("/protected", get(protected_handler))
        .route("/stats", get(stats_handler))
        .layer(middleware::from_fn_with_state(
            state.clone(),
            request_counter_middleware,
        ))
        .layer(middleware::from_fn(logging_middleware))
        .route("/protected", get(protected_handler))
        .layer(middleware::from_fn(auth_middleware))
        .with_state(state);
    
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();
    
    println!("서버가 http://127.0.0.1:3000 에서 실행 중");
    println!("엔드포인트:");
    println!("  GET /public      - 공용 엔드포인트");
    println!("  GET /protected   - 보호된 엔드포인트 (API 키 필요)");
    println!("  GET /stats        - 통계 정보");
    
    axum::serve(listener, app.into_make_service())
        .await
        .unwrap();
}
```

## 🔥 Actix-web 프레임워크

### 3.1 Actix-web 기본 설정

```toml
# Cargo.toml
[dependencies]
actix-web = "4.0"
actix-cors = "0.6"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

```rust
use actix_web::{get, post, web, App, HttpResponse, HttpServer, Responder};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct Greeting {
    message: String,
}

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().json(Greeting {
        message: "Hello, Actix-web!".to_string(),
    })
}

#[get("/hello/{name}")]
async fn hello_name(name: web::Path<String>) -> impl Responder {
    HttpResponse::Ok().json(Greeting {
        message: format!("Hello, {}!", name),
    })
}

#[post("/echo")]
async fn echo(body: web::Json<Greeting>) -> impl Responder {
    HttpResponse::Ok().json(body.into_inner())
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(hello)
            .service(hello_name)
            .service(echo)
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 3.2 Actix-web 데이터베이스 연동

```rust
use actix_web::{get, post, web, App, HttpResponse, HttpServer, Responder};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::sync::Mutex;

#[derive(Serialize, Deserialize, Clone)]
struct User {
    id: u32,
    name: String,
    email: String,
}

type UsersDb = Mutex<HashMap<u32, User>>;

#[get("/users")]
async fn get_users(db: web::Data<UsersDb>) -> impl Responder {
    let users = db.lock().unwrap();
    let user_list: Vec<User> = users.values().cloned().collect();
    HttpResponse::Ok().json(user_list)
}

#[get("/users/{id}")]
async fn get_user(
    db: web::Data<UsersDb>,
    path: web::Path<u32>,
) -> impl Responder {
    let users = db.lock().unwrap();
    
    match users.get(&*path) {
        Some(user) => HttpResponse::Ok().json(user),
        None => HttpResponse::NotFound().json("사용자를 찾을 수 없습니다"),
    }
}

#[post("/users")]
async fn create_user(
    db: web::Data<UsersDb>,
    user: web::Json<User>,
) -> impl Responder {
    let mut users = db.lock().unwrap();
    let mut new_user = user.into_inner();
    
    // 새 ID 생성
    let new_id = users.keys().max().unwrap_or(&0) + 1;
    new_user.id = new_id;
    
    users.insert(new_id, new_user.clone());
    
    HttpResponse::Created().json(new_user)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // 초기 데이터
    let initial_users = HashMap::from([
        (1, User {
            id: 1,
            name: "Alice".to_string(),
            email: "alice@example.com".to_string(),
        }),
        (2, User {
            id: 2,
            name: "Bob".to_string(),
            email: "bob@example.com".to_string(),
        }),
    ]);
    
    let db = web::Data::new(Mutex::new(initial_users));
    
    HttpServer::new(move || {
        App::new()
            .app_data(db.clone())
            .service(get_users)
            .service(get_user)
            .service(create_user)
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 🌊 Rocket 프레임워크

### 4.1 Rocket 기본 설정

```toml
# Cargo.toml
[dependencies]
rocket = { version = "0.5.0-rc.2", features = ["json"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

```rust
#[macro_use] extern crate rocket;
use rocket::serde::{json::Json, Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct Message {
    message: String,
}

#[get("/")]
fn index() -> Json<Message> {
    Json(Message {
        message: "Hello, Rocket!".to_string(),
    })
}

#[get("/hello/<name>")]
fn hello(name: &str) -> Json<Message> {
    Json(Message {
        message: format!("Hello, {}!", name),
    })
}

#[post("/echo", data = "<message>")]
fn echo(message: Json<Message>) -> Json<Message> {
    message
}

#[launch]
fn rocket() -> _ {
    rocket::build()
        .mount("/", routes![index, hello, echo])
}
```

## 🗄️ 데이터베이스 연동

### 5.1 SQLx 사용

```toml
# Cargo.toml
[dependencies]
axum = "0.6"
tokio = { version = "1.0", features = ["full"] }
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "chrono", "uuid"] }
serde = { version = "1.0", features = ["derive"] }
chrono = { version = "0.4", features = ["serde"] }
uuid = { version = "1.0", features = ["v4", "serde"] }
```

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    response::Json,
    routing::{get, post},
    Router,
};
use sqlx::postgres::PgPool;
use serde::{Deserialize, Serialize};
use chrono::{DateTime, Utc};
use uuid::Uuid;

#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
struct Post {
    id: Uuid,
    title: String,
    content: String,
    created_at: DateTime<Utc>,
    updated_at: DateTime<Utc>,
}

#[derive(Deserialize)]
struct CreatePost {
    title: String,
    content: String,
}

#[derive(Deserialize)]
struct UpdatePost {
    title: Option<String>,
    content: Option<String>,
}

// 모든 게시물 조회
async fn get_posts(
    State(pool): State<PgPool>,
) -> Result<Json<Vec<Post>>, StatusCode> {
    let posts = sqlx::query_as::<_, Post>("SELECT * FROM posts ORDER BY created_at DESC")
        .fetch_all(&pool)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok(Json(posts))
}

// 특정 게시물 조회
async fn get_post(
    State(pool): State<PgPool>,
    Path(id): Path<Uuid>,
) -> Result<Json<Post>, StatusCode> {
    let post = sqlx::query_as::<_, Post>("SELECT * FROM posts WHERE id = $1")
        .bind(id)
        .fetch_one(&pool)
        .await
        .map_err(|_| StatusCode::NOT_FOUND)?;
    
    Ok(Json(post))
}

// 게시물 생성
async fn create_post(
    State(pool): State<PgPool>,
    Json(new_post): Json<CreatePost>,
) -> Result<(StatusCode, Json<Post>), StatusCode> {
    let id = Uuid::new_v4();
    let now = Utc::now();
    
    let post = sqlx::query_as::<_, Post>(
        r#"
        INSERT INTO posts (id, title, content, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5)
        RETURNING *
        "#
    )
    .bind(id)
    .bind(&new_post.title)
    .bind(&new_post.content)
    .bind(now)
    .bind(now)
    .fetch_one(&pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok((StatusCode::CREATED, Json(post)))
}

// 게시물 업데이트
async fn update_post(
    State(pool): State<PgPool>,
    Path(id): Path<Uuid>,
    Json(update_post): Json<UpdatePost>,
) -> Result<Json<Post>, StatusCode> {
    let now = Utc::now();
    
    let post = sqlx::query_as::<_, Post>(
        r#"
        UPDATE posts 
        SET title = COALESCE($1, title),
            content = COALESCE($2, content),
            updated_at = $3
        WHERE id = $4
        RETURNING *
        "#
    )
    .bind(update_post.title)
    .bind(update_post.content)
    .bind(now)
    .bind(id)
    .fetch_one(&pool)
    .await
    .map_err(|_| StatusCode::NOT_FOUND)?;
    
    Ok(Json(post))
}

// 게시물 삭제
async fn delete_post(
    State(pool): State<PgPool>,
    Path(id): Path<Uuid>,
) -> StatusCode {
    let result = sqlx::query("DELETE FROM posts WHERE id = $1")
        .bind(id)
        .execute(&pool)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR);
    
    match result {
        Ok(rows_affected) if rows_affected > 0 => StatusCode::NO_CONTENT,
        _ => StatusCode::NOT_FOUND,
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 데이터베이스 연결
    let database_url = std::env::var("DATABASE_URL")
        .unwrap_or_else(|_| "postgres://user:password@localhost/database".to_string());
    
    let pool = PgPool::connect(&database_url).await?;
    
    // 마이그레이션 실행 (간단한 버전)
    sqlx::query(
        r#"
        CREATE TABLE IF NOT EXISTS posts (
            id UUID PRIMARY KEY,
            title VARCHAR(255) NOT NULL,
            content TEXT NOT NULL,
            created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
            updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
        )
        "#
    )
    .execute(&pool)
    .await?;
    
    let app = Router::new()
        .route("/posts", get(get_posts).post(create_post))
        .route("/posts/:id", get(get_post).put(update_post).delete(delete_post))
        .with_state(pool);
    
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await?;
    
    println!("서버가 http://127.0.0.1:3000 에서 실행 중");
    
    axum::serve(listener, app.into_make_service())
        .await?;
    
    Ok(())
}
```

### 5.2 Diesel ORM 사용

```toml
# Cargo.toml
[dependencies]
diesel = { version = "2.0", features = ["postgres", "uuid", "chrono"] }
uuid = { version = "1.0", features = ["v4"] }
chrono = { version = "0.4", features = ["serde"] }
serde = { version = "1.0", features = ["derive"] }
axum = "0.6"
tokio = { version = "1.0", features = ["full"] }
```

```rust
// diesel.toml
[print_schema]
file = "src/schema.rs"
```

```rust
// src/schema.rs (diesel print-schema로 생성)
diesel::table! {
    posts (id) {
        id -> Uuid,
        title -> Varchar,
        content -> Text,
        created_at -> Timestamp,
        updated_at -> Timestamp,
    }
}

use diesel::prelude::*;
use axum::{
    extract::{Path, State},
    http::StatusCode,
    response::Json,
    routing::{get, post},
    Router,
};
use serde::{Deserialize, Serialize};
use uuid::Uuid;
use chrono::{DateTime, Utc};

// 모델 정의
#[derive(Debug, Serialize, Deserialize, Queryable)]
pub struct Post {
    pub id: Uuid,
    pub title: String,
    pub content: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Insertable)]
#[diesel(table_name = posts)]
pub struct NewPost {
    pub id: Uuid,
    pub title: String,
    pub content: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

// 앱 상태
pub struct AppState {
    pub pool: deadpool_diesel::postgres::Pool,
}

// 핸들러들
async fn get_posts(
    State(state): State<AppState>,
) -> Result<Json<Vec<Post>>, StatusCode> {
    use self::schema::posts::dsl::*;
    
    let mut conn = state.pool
        .get()
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    let results = posts
        .order(created_at.desc())
        .load::<Post>(&mut conn)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok(Json(results))
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 데이터베이스 설정
    let database_url = std::env::var("DATABASE_URL")
        .unwrap_or_else(|_| "postgres://user:password@localhost/database".to_string());
    
    let manager = deadpool_diesel::postgres::Config::new();
    let pool = manager.create_pool(Some(database_url), deadpool::Runtime::Tokio1)?;
    
    let state = AppState { pool };
    
    let app = Router::new()
        .route("/posts", get(get_posts))
        .with_state(state);
    
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await?;
    
    println!("서버가 http://127.0.0.1:3000 에서 실행 중");
    
    axum::serve(listener, app.into_make_service())
        .await?;
    
    Ok(())
}
```

## 🧪 테스트

### 6.1 단위 테스트

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use axum::{
        body::Body,
        http::{Request, StatusCode},
        response::Response,
    };
    use tower::ServiceExt;

    // 테스트용 앱 생성
    async fn create_app() -> Router {
        // 테스트용 데이터베이스 설정
        let state = AppState {
            // 인메모리 데이터베이스 또는 테스트 데이터베이스
        };
        
        Router::new()
            .route("/users", get(get_users).post(create_user))
            .with_state(state)
    }

    #[tokio::test]
    async fn test_get_users() {
        let app = create_app().await;
        
        let request = Request::builder()
            .uri("/users")
            .body(Body::empty())
            .unwrap();
        
        let response = app.oneshot(request).await.unwrap();
        
        assert_eq!(response.status(), StatusCode::OK);
    }

    #[tokio::test]
    async fn test_create_user() {
        let app = create_app().await;
        
        let new_user = CreateUser {
            name: "Test User".to_string(),
            email: "test@example.com".to_string(),
        };
        
        let request = Request::builder()
            .uri("/users")
            .method("POST")
            .header("content-type", "application/json")
            .body(Body::from(serde_json::to_string(&new_user).unwrap()))
            .unwrap();
        
        let response = app.oneshot(request).await.unwrap();
        
        assert_eq!(response.status(), StatusCode::CREATED);
    }
}
```

### 6.2 통합 테스트

```rust
// tests/integration_test.rs
use axum::{body::Body, http::Request, response::Response};
use serde_json::json;
use tower::ServiceExt;

#[tokio::test]
async fn test_user_crud() {
    let app = create_app().await;
    
    // 사용자 생성
    let create_user = json!({
        "name": "Integration Test User",
        "email": "integration@example.com"
    });
    
    let request = Request::builder()
        .uri("/users")
        .method("POST")
        .header("content-type", "application/json")
        .body(Body::from(create_user.to_string()))
        .unwrap();
    
    let response = app.clone().oneshot(request).await.unwrap();
    assert_eq!(response.status(), 201);
    
    // 사용자 목록 조회
    let request = Request::builder()
        .uri("/users")
        .body(Body::empty())
        .unwrap();
    
    let response = app.clone().oneshot(request).await.unwrap();
    assert_eq!(response.status(), 200);
}
```

## 🚀 배포

### 7.1 Docker 설정

```dockerfile
# Dockerfile
FROM rust:1.70 as builder

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src ./src

# 빌드
RUN cargo build --release

# 실행 환경
FROM debian:bookworm-slim

# 런타임 의존성
RUN apt-get update && apt-get install -y \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY --from=builder /app/target/release/my-app /app/my-app

EXPOSE 3000

CMD ["./my-app"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://user:password@db:5432/database
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=database
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 7.2 systemd 서비스

```ini
# /etc/systemd/system/rust-web-app.service
[Unit]
Description=Rust Web Application
After=network.target

[Service]
Type=simple
User=rust-app
WorkingDirectory=/opt/rust-app
ExecStart=/opt/rust-app/my-app
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## 📝 연습 문제

### 문제 1: 블로그 API
다음 기능들을 가진 블로그 API를 구현하세요:

```rust
// TODO: Post 모델 정의
// - id: Uuid
// - title: String
// - content: String
// - author: String
// - published: bool
// - created_at: DateTime<Utc>
// - updated_at: DateTime<Utc>

// TODO: CRUD 엔드포인트 구현
// - GET /posts - 모든 게시물 조회
// - GET /posts/:id - 특정 게시물 조회
// - POST /posts - 게시물 생성
// - PUT /posts/:id - 게시물 업데이트
// - DELETE /posts/:id - 게시물 삭제

// TODO: 추가 기능
// - 게시물 검색
// - 작성자별 게시물 조회
// - 발행된 게시물만 조회
```

### 문제 2: 파일 업로드 API
다음 기능들을 가진 파일 업로드 API를 구현하세요:

```rust
// TODO: 파일 업로드 엔드포인트 구현
// - POST /upload - 파일 업로드
// - GET /files/:id - 파일 다운로드
// - GET /files - 파일 목록 조회
// - DELETE /files/:id - 파일 삭제

// TODO: 파일 메타데이터 관리
// - 파일 크기 제한
// - 파일 타입 검증
// - 고유 파일명 생성
```

### 문제 3: 인증 시스템
다음 기능들을 가진 인증 시스템을 구현하세요:

```rust
// TODO: User 모델 정의
// - id: Uuid
// - username: String
// - email: String
// - password_hash: String
// - created_at: DateTime<Utc>

// TODO: 인증 엔드포인트 구현
// - POST /auth/register - 사용자 등록
// - POST /auth/login - 로그인
// - POST /auth/logout - 로그아웃
// - GET /auth/me - 현재 사용자 정보

// TODO: JWT 토큰 기반 인증 미들웨어
```

---

**축하합니다!** 이제 Rust 웹 개발의 기초를 마쳤습니다. 🦀

**다음 단계**: [13_project_examples.md](./13_project_examples.md)에서 실제 프로젝트 예제를 살펴보세요!
