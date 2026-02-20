# Rust 학습 완벽 가이드

## 🦀 Rust란 무엇인가?

Rust는 Mozilla에서 개발한 시스템 프로그래밍 언어로, **안전성**, **속도**, **동시성**을 중점으로 설계되었습니다. 메모리 안전성을 컴파일 타임에 보장하면서도 C/C++ 수준의 성능을 제공합니다.

## 📚 학습 로드맵

### 1단계: 기초 다지기 (1-2주)

#### 필수 개념
- **소유권(Ownership) 시스템**
  - 소유권, 대여(Borrowing), 수명(Lifetime)
  - Rust의 핵심 개념으로 반드시 이해해야 함
  
- **기본 문법**
  ```rust
  fn main() {
      let mut x = 5;
      println!("Hello, {}", x);
  }
  ```

#### 추천 학습 자료
- [The Rust Programming Language](https://doc.rust-lang.org/book/) (공식 문서)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rustlings](https://github.com/rust-lang/rustlings/) (실습 문제)

### 2단계: 중급 개념 (2-3주)

#### 핵심 주제
- **구조체와 열거형**
  - struct, enum, impl 블록
  - 패턴 매칭
  
- **에러 처리**
  ```rust
  fn divide(a: i32, b: i32) -> Result<i32, String> {
      if b == 0 {
          Err("Cannot divide by zero".to_string())
      } else {
          Ok(a / b)
      }
  }
  ```

- **제네릭과 트레잇**
  - 제네릭 프로그래밍
  - 트레잇 시스템 이해

#### 실습 프로젝트
- 간단한 계산기
- To-Do 애플리케이션
- 파일 관리 도구

### 3단계: 고급 기능 (3-4주)

#### 심화 개념
- **동시성 프로그래밍**
  - 스레드, 채널, 공유 상태
  
- **매크로**
  - 선언적 매크로
  - 절차적 매크로

- **unsafe Rust**
  - 안전하지 않은 코드 블록
  - FFI(Foreign Function Interface)

#### 추천 프로젝트
- 멀티스레드 웹 서버
- 데이터베이스 클라이언트
- 간단한 운영체제 커널

### 4단계: 생태계 탐색 (2-3주)

#### 주요 라이브러리
- **웹 개발**: Actix-web, Axum, Rocket
- **CLI 도구**: Clap, Structopt
- **비동기**: Tokio, async-std
- **데이터베이스**: Diesel, SQLx

#### 도구 체인
- **Cargo**: 패키지 매니저 및 빌드 도구
- **rustfmt**: 코드 포매터
- **clippy**: 린트 도구

## 🛠️ 실용적인 학습 방법

### 1. 프로젝트 기반 학습

#### 초급 프로젝트
```rust
// 간단한 숫자 추측 게임
use std::io;
use rand::Rng;

fn main() {
    let secret = rand::thread_rng().gen_range(1..=100);
    
    loop {
        println!("숫자를 추측해보세요:");
        let mut guess = String::new();
        io::stdin().read_line(&mut guess).expect("입력 실패");
        
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
        
        match guess.cmp(&secret) {
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

#### 중급 프로젝트
- REST API 서버
- 파일 동기화 도구
- 간단한 블록체인

#### 고급 프로젝트
- 운영체제 커널
- 컴파일러
- 데이터베이스 엔진

### 2. 코드 리뷰 및 커뮤니티 참여

#### 커뮤니티
- [Rust 사용자 포럼](https://users.rust-lang.org/)
- [Reddit r/rust](https://www.reddit.com/r/rust/)
- [Discord Rust 서버](https://discord.gg/rust-lang)

#### 기여 방법
- 오픈소스 프로젝트에 기여
- 버그 리포트 및 수정
- 문서 번역

### 3. 효과적인 학습 습관

#### 일일 루틴
- 매일 1-2시간 코드 작성
- 새로운 개념 정리
- 작은 프로젝트 완성

#### 학습 팁
- **소유권 개념**은 반복 학습 필수
- **컴파일러 오류 메시지**를 친구처럼 여기기
- **클론 코딩**으로 시작하여 점진적으로 독창적인 코드로

## 📖 추천 학습 자료

### 공식 문서
- [The Rust Book](https://doc.rust-lang.org/book/) - 가장 중요한 자료
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/) - 예제 중심 학습
- [Rust Reference](https://doc.rust-lang.org/reference/) - 상세 언어 명세

### 비디오 강좌
- [Rust Crash Course](https://www.youtube.com/watch?v=vOMJNfj2y3c)
- [Let's Get Rusty](https://www.youtube.com/c/LetsGetRusty)

### 책
- "The Rust Programming Language" (공식 서적)
- "Programming Rust" by Jim Blandy
- "Rust in Action" by Tim McNamara

### 실습 플랫폼
- [Exercism Rust](https://exercism.org/tracks/rust)
- [Codewars Rust](https://www.codewars.com/?language=rust)
- [LeetCode Rust](https://leetcode.com/)

## 🎯 학습 체크리스트

### 기초 단계 (1-2주)

#### 환경 설정
- [ ] **Rust 설치**: `rustup`을 통한 최신 버전 설치
  ```bash
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```
- [ ] **IDE 설정**: VS Code + rust-analyzer 확장 프로그램
- [ ] **Cargo 기본 명령어 숙지**
  ```bash
  cargo new project_name    # 새 프로젝트 생성
  cargo build              # 빌드
  cargo run                # 실행
  cargo test               # 테스트
  cargo check              # 빠른 검사
  ```

#### 기본 문법
- [ ] **변수와 상수**: `let`, `const`, `mut` 키워드 이해
  ```rust
  let x = 5;              // 불변 변수
  let mut y = 10;         // 가변 변수
  const MAX_POINTS: u32 = 100_000;  // 상수
  ```
- [ ] **데이터 타입**: 스칼라 타입, 복합 타입
  ```rust
  // 스칼라 타입
  let integer: i32 = 42;
  let float: f64 = 3.14;
  let boolean: bool = true;
  let character: char = '🦀';
  
  // 복합 타입
  let tuple: (i32, f64, bool) = (42, 3.14, true);
  let array: [i32; 5] = [1, 2, 3, 4, 5];
  ```
- [ ] **함수**: 매개변수, 반환값, 표현식 vs 문장
  ```rust
  fn add(a: i32, b: i32) -> i32 {
      a + b  // 세미콜론 없으면 표현식 (반환값)
  }
  
  fn print_sum(a: i32, b: i32) {
      println!("합계: {}", a + b);  // 세미콜론 있으면 문장
  }
  ```
- [ ] **제어문**: `if/else`, `loop`, `while`, `for`
  ```rust
  // if 표현식
  let number = if condition { 5 } else { 6 };
  
  // 패턴 매칭과 for 루프
  for (index, value) in array.iter().enumerate() {
      println!("{}: {}", index, value);
  }
  ```

#### 소유권 시스템 (가장 중요!)
- [ ] **소유권 규칙 이해**
  1. Rust의 각 값은 소유자(owner)가 정확히 하나씩 있다
  2. 소유자가 스코프 밖으로 벗어나면 값은 버려진다(dropped)
  3. 소유권은 이동(move)하거나 복사(copy)할 수 있다
  
- [ ] **대여(Borrowing) 개념**
  ```rust
  fn calculate_length(s: &String) -> usize {  // 불변 대여
      s.len()
  }  // s는 스코프를 벗어나지만 소유권이 없으므로 아무 일도 일어나지 않음
  
  fn change(some_string: &mut String) {  // 가변 대여
      some_string.push_str(", world");
  }
  ```

- [ ] **수명(Lifetime) 기초**
  ```rust
  fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
      if x.len() > y.len() {
          x
      } else {
          y
      }
  }
  ```

#### 구조체와 열거형
- [ ] **구조체 정의와 사용**
  ```rust
  #[derive(Debug)]
  struct User {
      username: String,
      email: String,
      age: u32,
      active: bool,
  }
  
  impl User {
      fn new(username: String, email: String) -> User {
          User {
              username,
              email,
              age: 0,
              active: true,
          }
      }
  }
  ```

- [ ] **열거형과 패턴 매칭**
  ```rust
  enum IpAddr {
      V4(u8, u8, u8, u8),
      V6(String),
  }
  
  enum Message {
      Quit,
      Move { x: i32, y: i32 },
      Write(String),
      ChangeColor(i32, i32, i32),
  }
  
  impl Message {
      fn process(&self) {
          match self {
              Message::Quit => println!("종료"),
              Message::Move { x, y } => println!("이동: ({}, {})", x, y),
              Message::Write(text) => println!("쓰기: {}", text),
              Message::ChangeColor(r, g, b) => println!("색상 변경: RGB({}, {}, {})", r, g, b),
          }
      }
  }
  ```

### 중급 단계 (2-3주)

#### 에러 처리
- [ ] **Result 타입 완전 정복**
  ```rust
  use std::fs::File;
  use std::io::{self, Read};
  
  fn read_file_content(path: &str) -> Result<String, io::Error> {
      let mut file = File::open(path)?;
      let mut contents = String::new();
      file.read_to_string(&mut contents)?;
      Ok(contents)
  }
  
  // 여러 타입의 에러 처리
  fn process_data() -> Result<(), Box<dyn std::error::Error>> {
      let content = read_file_content("data.txt")?;
      println!("내용: {}", content);
      Ok(())
  }
  ```

- [ ] **Option 타입 활용**
  ```rust
  fn find_user_by_id(id: u32) -> Option<User> {
      if id == 1 {
          Some(User::new("admin".to_string(), "admin@example.com".to_string()))
      } else {
          None
      }
  }
  
  // Option 체이닝
  let user_email = find_user_by_id(1)
      .map(|user| user.email)
      .unwrap_or_else(|| "사용자를 찾을 수 없습니다".to_string());
  ```

- [ ] **에러 전파 연산자 `?` 마스터하기**
  ```rust
  fn parse_config() -> Result<Config, std::env::VarError> {
      let db_url = std::env::var("DATABASE_URL")?;
      let port: u16 = std::env::var("PORT")?.parse()
          .map_err(|_| std::env::VarError::NotPresent)?;
      
      Ok(Config { db_url, port })
  }
  ```

#### 제네릭과 트레잇
- [ ] **제네릭 함수와 구조체**
  ```rust
  fn largest<T: PartialOrd>(list: &[T]) -> &T {
      let mut largest = &list[0];
      
      for item in list {
          if item > largest {
              largest = item;
          }
      }
      
      largest
  }
  
  #[derive(Debug)]
  struct Point<T> {
      x: T,
      y: T,
  }
  
  impl<T> Point<T> {
      fn x(&self) -> &T {
          &self.x
      }
  }
  ```

- [ ] **핵심 트레잇 이해**
  ```rust
  // Clone 트레잇
  #[derive(Clone, Debug)]
  struct MyStruct {
      value: i32,
  }
  
  // Copy 트레잇 (Clone이 먼저 필요)
  #[derive(Copy, Clone, Debug)]
  struct SimpleStruct {
      value: i32,
  }
  
  // Display 트레잇
  use std::fmt;
  
  impl fmt::Display for MyStruct {
      fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
          write!(f, "MyStruct {{ value: {} }}", self.value)
      }
  }
  
  // 커스텀 트레잇
  trait Summary {
      fn summarize(&self) -> String;
      
      fn summarize_verbose(&self) -> String {
          format!("(더 자세한 내용) {}", self.summarize())
      }
  }
  
  struct NewsArticle {
      headline: String,
      location: String,
      author: String,
      content: String,
  }
  
  impl Summary for NewsArticle {
      fn summarize(&self) -> String {
          format!("{}, by {} ({})", self.headline, self.author, self.location)
      }
  }
  ```

- [ ] **트레잇 바운드와 수명**
  ```rust
  fn notify<T: Summary>(item: &T) {
      println!("속보! {}", item.summarize());
  }
  
  // where 절 사용
  fn some_function<T, U>(_t: T, _u: U) -> i32
  where
      T: Display + Clone,
      U: Clone + Debug,
  {
      42
  }
  ```

#### 모듈 시스템
- [ ] **모듈 구조 이해**
  ```rust
  // lib.rs
  mod front_of_house {
      pub mod hosting {
          pub fn add_to_waitlist() {}
          
          fn seat_at_table() {}
      }
      
      mod serving {
          fn take_order() {}
          fn serve_order() {}
          fn take_payment() {}
      }
  }
  
  pub use crate::front_of_house::hosting::add_to_waitlist;
  
  pub fn eat_at_restaurant() {
      add_to_waitlist();
  }
  ```

- [ ] **패키지와 크레이트**
  - 바이너리 크레이트 vs 라이브러리 크레이트
  - Cargo.toml 설정
  - 외부 의존성 관리

#### 테스트
- [ ] **단위 테스트 작성**
  ```rust
  #[cfg(test)]
  mod tests {
      use super::*;
      
      #[test]
      fn exploration() {
          let result = add(2, 2);
          assert_eq!(result, 4);
      }
      
      #[test]
      #[should_panic(expected = "값은 100보다 작아야 합니다")]
      fn greater_than_100() {
          guess(200);
      }
      
      #[test]
      fn it_works() -> Result<(), String> {
          if 2 + 2 != 4 {
              Err!("두 수를 더한 결과가 4가 아닙니다".to_string())
          } else {
              Ok(())
          }
      }
  }
  ```

- [ ] **통합 테스트**
  ```rust
  // tests/integration_test.rs
  use adder;
  
  #[test]
  fn it_adds_two() {
      assert_eq!(4, adder::add_two(2));
  }
  ```

### 고급 단계 (3-4주)

#### 동시성 프로그래밍
- [ ] **스레드 기초**
  ```rust
  use std::thread;
  use std::time::Duration;
  
  fn main() {
      let handle = thread::spawn(|| {
          for i in 1..10 {
              println!("hi number {} from the spawned thread!", i);
              thread::sleep(Duration::from_millis(1));
          }
      });
      
      for i in 1..5 {
          println!("hi number {} from the main thread!", i);
          thread::sleep(Duration::from_millis(1));
      }
      
      handle.join().unwrap();
  }
  ```

- [ ] **채널을 통한 메시지 전달**
  ```rust
  use std::sync::mpsc;
  use std::thread;
  
  fn main() {
      let (tx, rx) = mpsc::channel();
      
      thread::spawn(move || {
          let vals = vec![
              String::from("hi"),
              String::from("from"),
              String::from("the"),
              String::from("thread"),
          ];
          
          for val in vals {
              tx.send(val).unwrap();
              thread::sleep(Duration::from_millis(1));
          }
      });
      
      for received in rx {
          println!("Got: {}", received);
      }
  }
  ```

- [ ] **공유 상태 동시성**
  ```rust
  use std::sync::{Arc, Mutex};
  use std::thread;
  
  fn main() {
      let counter = Arc::new(Mutex::new(0));
      let mut handles = vec![];
      
      for _ in 0..10 {
          let counter = Arc::clone(&counter);
          let handle = thread::spawn(move || {
              let mut num = counter.lock().unwrap();
              *num += 1;
          });
          handles.push(handle);
      }
      
      for handle in handles {
          handle.join().unwrap();
      }
      
      println!("Result: {}", *counter.lock().unwrap());
  }
  ```

- [ ] **Async/Await 기초**
  ```rust
  use tokio;
  
  #[tokio::main]
  async fn main() {
      let result1 = async_operation().await;
      let result2 = another_async_operation().await;
      
      println!("결과: {}, {}", result1, result2);
  }
  
  async fn async_operation() -> String {
      tokio::time::sleep(Duration::from_secs(1)).await;
      "첫 번째 작업 완료".to_string()
  }
  ```

#### 매크로
- [ ] **선언적 매크로**
  ```rust
  macro_rules! vec {
      ( $( $x:expr ),* ) => {
          {
              let mut temp_vec = Vec::new();
              $(
                  temp_vec.push($x);
              )*
              temp_vec
          }
      };
  }
  
  // 사용
  let v: Vec<i32> = vec!(1, 2, 3, 4);
  ```

- [ ] **절차적 매크로 기초**
  ```rust
  use proc_macro::TokenStream;
  
  #[proc_macro_derive(MyTrait)]
  pub fn my_trait_derive(input: TokenStream) -> TokenStream {
      // 매크로 구현 로직
      TokenStream::new()
  }
  ```

#### unsafe Rust
- [ ] **unsafe 블록 이해**
  ```rust
  unsafe fn dangerous() {}
  
  fn main() {
      unsafe {
          dangerous();
      }
  }
  ```

- [ ] **원시 포인터**
  ```rust
  fn main() {
      let mut num = 5;
      
      let r1 = &num as *const i32;
      let r2 = &mut num as *mut i32;
      
      unsafe {
          println!("r1 is: {}", *r1);
          println!("r2 is: {}", *r2);
      }
  }
  ```

- [ ] **FFI (Foreign Function Interface)**
  ```rust
  extern "C" {
      fn abs(input: i32) -> i32;
  }
  
  fn main() {
      unsafe {
          println!("Absolute value of -3 according to C: {}", abs(-3));
      }
  }
  ```

### 실용 단계 (2-3주)

#### 웹 서버 개발
- [ ] **기본 HTTP 서버**
  ```rust
  use std::io::prelude::*;
  use std::net::TcpListener;
  use std::net::TcpStream;
  
  fn main() {
      let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
      
      for stream in listener.incoming() {
          let stream = stream.unwrap();
          
          handle_connection(stream);
      }
  }
  
  fn handle_connection(mut stream: TcpStream) {
      let mut buffer = [0; 1024];
      
      stream.read(&mut buffer).unwrap();
      
      let response = "HTTP/1.1 200 OK\r\n\r\n";
      
      stream.write(response.as_bytes()).unwrap();
      stream.flush().unwrap();
  }
  ```

- [ ] **웹 프레임워크 사용 (Axum/Actix)**
  ```rust
  use axum::{response::Html, routing::get, Router};
  
  #[tokio::main]
  async fn main() {
      let app = Router::new()
          .route("/", get(handler))
          .route("/hello/:name", get(hello_handler));
      
      let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
          .await
          .unwrap();
      
      println!("listening on {}", listener.local_addr().unwrap());
      axum::serve(listener, app).await.unwrap();
  }
  
  async fn handler() -> Html<&'static str> {
      Html("<h1>Hello, World!</h1>")
  }
  
  async fn hello_handler(axum::extract::Path(name): axum::extract::Path<String>) -> String {
      format!("Hello, {}!", name)
  }
  ```

#### CLI 도구 제작
- [ ] **인자 파싱 (Clap)**
  ```rust
  use clap::{Parser, Subcommand};
  
  #[derive(Parser)]
  #[command(name = "my-cli")]
  #[command(about = "A simple CLI tool")]
  struct Cli {
      #[command(subcommand)]
      command: Commands,
  }
  
  #[derive(Subcommand)]
  enum Commands {
      Add {
          name: String,
      },
      List,
      Remove {
          id: usize,
      },
  }
  
  fn main() {
      let cli = Cli::parse();
      
      match cli.command {
          Commands::Add { name } => {
              println!("Adding: {}", name);
          }
          Commands::List => {
              println!("Listing items");
          }
          Commands::Remove { id } => {
              println!("Removing item with id: {}", id);
          }
      }
  }
  ```

#### 데이터베이스 연동
- [ ] **SQLx 사용**
  ```rust
  use sqlx::{PgPool, Row};
  
  #[tokio::main]
  async fn main() -> Result<(), sqlx::Error> {
      let pool = PgPool::connect("postgres://user:password@localhost/db").await?;
      
      let rows = sqlx::query("SELECT id, name FROM users")
          .fetch_all(&pool)
          .await?;
      
      for row in rows {
          let id: i32 = row.get("id");
          let name: String = row.get("name");
          println!("User {}: {}", id, name);
      }
      
      Ok(())
  }
  ```

#### 오픈소스 기여
- [ ] **프로젝트 선택 및 포크**
- [ ] **이슈 분석 및 해결**
- [ ] **풀 리퀘스트 작성**
- [ ] **코드 리뷰 참여**

## 💡 학습 조언

### 성공 전략
1. **꾸준함이 핵심**: 매일 1-2시간씩 코드 작성
   - 주말에는 더 긴 시간 투자
   - 작은 목표 설정 및 달성 체크
   - GitHub 커밋 기록으로 꾸준함 증명

2. **컴파일러와 친해지기**: 오류 메시지를 학습 기회로 활용
   ```rust
   // 흔한 컴파일 오류와 해결법
   // error[E0382]: borrow of moved value
   let s1 = String::from("hello");
   let s2 = s1;  // s1의 소유권이 s2로 이동
   println!("{}", s1);  // 오류! s1은 더 이상 유효하지 않음
   
   // 해결책: 복사(clone) 또는 대여(borrow)
   let s1 = String::from("hello");
   let s2 = s1.clone();  // 명시적 복사
   println!("{}, {}", s1, s2);  // 정상 작동
   ```

3. **작은 성취 쌓기**: 작은 프로젝트부터 시작
   - 1일: Hello World 출력
   - 1주: 간단한 계산기
   - 2주: To-Do 리스트
   - 1달: 간단한 웹 서버

4. **커뮤니티 활용**: 다른 개발자들과 교류
   - Rust Discord 서버 참여
   - Weekly Rust Newsletter 구독
   - Rust Meetup 참석 (온라인/오프라인)

### 흔한 실수와 해결책

#### 소유권 관련 실수
```rust
// 실수 1: 반복문에서 소유권 이동
let vec = vec![1, 2, 3];
for item in vec {  // vec의 소유권이 이동
    println!("{}", item);
}
// println!("{:?}", vec);  // 오류! vec은 더 이상 유효하지 않음

// 해결책: 참조자 사용
for item in &vec {
    println!("{}", item);
}
println!("{:?}", vec);  // 정상 작동
```

#### 수명 관련 실수
```rust
// 실수: 수명 충돌
fn longest<'a>(x: &'a str, y: &str) -> &'a str {  // y의 수명 지정 누락
    if x.len() > y.len() {
        x
    } else {
        y  // 컴파일 오류
    }
}

// 해결책: 모든 참조에 수명 지정
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

#### 에러 처리 실수
```rust
// 실수: panic! 남용
fn get_user(id: u32) -> User {
    let user = find_user(id).unwrap();  // panic 가능성
    user
}

// 해결책: 적절한 에러 처리
fn get_user(id: u32) -> Result<User, Error> {
    find_user(id).ok_or(Error::UserNotFound(id))
}
```

### 문제 해결 방법론

#### 1. 컴파일 오류 분석 단계
1. **오류 메시지 주의 깊게 읽기**
   - 오류 코드(E0382, E0277 등) 이해
   - 제안된 해결책 검토
   - 관련 문서 링크 확인

2. **코드 분리**
   ```rust
   // 복잡한 함수를 작은 단위로 분리
   fn complex_function(data: &str) -> Result<Output, Error> {
       let step1 = parse_data(data)?;      // 1단계
       let step2 = validate(step1)?;      // 2단계
       let step3 = process(step2)?;       // 3단계
       Ok(step4)
   }
   ```

3. **테스트 케이스 작성**
   ```rust
   #[test]
   fn test_edge_cases() {
       assert_eq!(add(0, 0), 0);
       assert_eq!(add(-1, 1), 0);
       assert_eq!(add(i32::MAX, 0), i32::MAX);
   }
   ```

#### 2. 런타임 문제 해결
- **디버그 출력**: `println!`, `dbg!` 매크로 활용
- **단위 테스트**: 문제 재현 및 검증
- **메모리 프로파일링**: `valgrind`, `heaptrack` 사용

### 학습 자료 활용 전략

#### 공식 문서 활용법
1. **Rust Book**: 체계적 학습 (1회독 필수)
2. **Rust by Example**: 실습 중심 학습
3. **API 문서**: 특정 함수/트레잇 조회
4. **Rust Reference**: 깊이 있는 이해

#### 비디오 강좌 활용법
- **이론 학습**: 개념 설명 영상 시청
- **코딩 따라하기**: 실습 영상과 함께 코딩
- **코드 리뷰**: 다른 사람의 코드 분석

### 실용적인 학습 팁

#### 1. 코드 작성 습관
```rust
// 좋은 코드 스타일
fn calculate_area(width: f64, height: f64) -> f64 {
    // 입력 검증
    if width <= 0.0 || height <= 0.0 {
        return 0.0;
    }
    
    width * height
}

// 테스트 코드 함께 작성
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_calculate_area() {
        assert_eq!(calculate_area(10.0, 5.0), 50.0);
        assert_eq!(calculate_area(0.0, 10.0), 0.0);
    }
}
```

#### 2. 효율적인 Cargo 사용
```bash
# 개발 시 유용한 명령어
cargo check          # 빠른 문법 검사
cargo watch          # 파일 변경 시 자동 빌드
cargo clippy          # 린트 검사
cargo fmt             # 코드 포맷팅
cargo doc --open      # 문서 생성 및 열기
```

#### 3. 의존성 관리
```toml
# Cargo.toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }  # 필요한 기능만
tokio = { version = "1.0", features = ["full"] }    # 전체 기능
log = "0.4"                                        # 로깅
env_logger = "0.9"                                 # 환경 변수 로깅

[dev-dependencies]  # 테스트용 의존성
tokio-test = "0.4"
mockall = "0.10"
```

## 🚀 다음 단계와 전문성 개발

### 전문 분야 선택 가이드

#### 1. 웹 백엔드 개발
**필수 기술 스택:**
- **웹 프레임워크**: Axum, Actix-web, Rocket
- **데이터베이스**: SQLx, Diesel ORM
- **인증**: JWT, OAuth2 라이브러리
- **배포**: Docker, Kubernetes

**학습 로드맵:**
```rust
// 1단계: 기본 웹 서버
use axum::{routing::get, Router};

#[tokio::main]
async fn main() {
    let app = Router::new().route("/", get(handler));
    
    axum::Server::bind(&"0.0.0.0:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}

// 2단계: REST API
async fn get_user(
    axum::extract::Path(id): axum::extract::Path<u32>
) -> Result<Json<User>, StatusCode> {
    match find_user(id).await {
        Some(user) => Ok(Json(user)),
        None => Err(StatusCode::NOT_FOUND),
    }
}

// 3단계: 미들웨어 및 에러 처리
async fn auth_middleware(
    req: Request<Body>,
    next: Next<Body>,
) -> Result<Response, StatusCode> {
    // JWT 검증 로직
    next.run(req).await
}
```

**추천 프로젝트:**
- 블로그 API 서버
- 채팅 서버 (WebSocket)
- 파일 업로드 서비스
- 인증/인가 시스템

#### 2. 임베디드 시스템
**필수 기술 스택:**
- **임베디드 Rust**: `embedded-hal`, `cortex-m`
- **하드웨어**: STM32, ESP32, Raspberry Pi Pico
- **통신**: I2C, SPI, UART
- **RTOS**: RTIC, Embassy

**학습 로드맵:**
```rust
// 1단계: LED 제어
use embedded_hal::digital::v2::OutputPin;

fn blink_led<LED: OutputPin>(mut led: LED) -> ! {
    loop {
        led.set_high().unwrap();
        delay_ms(1000);
        led.set_low().unwrap();
        delay_ms(1000);
    }
}

// 2단계: 센서 데이터 읽기
use embedded_hal::blocking::i2c::{Read, Write};

fn read_temperature<I2C: Read + Write>(i2c: &mut I2C, addr: u8) -> Result<f32, Error> {
    let mut buffer = [0u8; 2];
    i2c.write(addr, &[0x00])?;  // 온도 레지스터 선택
    i2c.read(addr, &mut buffer)?;  // 데이터 읽기
    
    let raw = u16::from_be_bytes(buffer);
    Ok(raw as f32 * 0.02)  // 센서 데이터 변환
}
```

**추천 프로젝트:**
- 온습도 모니터
- 스마트 홈 컨트롤러
- 로봇 제어 시스템
- 데이터 로거

#### 3. 블록체인 개발
**필수 기술 스택:**
- **블록체인 프레임워크**: Substrate, ink!
- **암호화**: `sha2`, `ed25519-dalek`
- **P2P 네트워크**: `libp2p`
- **스마트 컨트랙트**: `solana-program`

**학습 로드맵:**
```rust
// 1단계: 기본 블록 구조
#[derive(Debug, Clone)]
struct Block {
    index: u64,
    timestamp: u64,
    data: String,
    previous_hash: String,
    hash: String,
}

impl Block {
    fn new(index: u64, data: String, previous_hash: String) -> Self {
        let timestamp = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_secs();
            
        let hash = Self::calculate_hash(index, timestamp, &data, &previous_hash);
        
        Block {
            index,
            timestamp,
            data,
            previous_hash,
            hash,
        }
    }
}

// 2단계: 간단한 블록체인
struct Blockchain {
    chain: Vec<Block>,
    difficulty: usize,
}

impl Blockchain {
    fn new() -> Self {
        let genesis_block = Block::new(0, "Genesis Block".to_string(), "0".to_string());
        Blockchain {
            chain: vec![genesis_block],
            difficulty: 4,
        }
    }
    
    fn add_block(&mut self, data: String) {
        let previous_hash = self.chain.last().unwrap().hash.clone();
        let mut new_block = Block::new(
            self.chain.len() as u64,
            data,
            previous_hash,
        );
        
        // Proof of Work
        new_block.mine_block(self.difficulty);
        self.chain.push(new_block);
    }
}
```

**추천 프로젝트:**
- 간단한 암호화폐
- NFT 마켓플레이스
- 투표 시스템
- 공급망 추적

#### 4. 게임 개발
**필수 기술 스택:**
- **게임 엔진**: Bevy, Fyrox
- **그래픽**: wgpu, vulkano
- **물리 엔진**: rapier3d, nphysics
- **오디오**: rodio

**학습 로드맵:**
```rust
// Bevy 게임 엔진 예제
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_startup_system(setup)
        .add_system(move_player)
        .run();
}

#[derive(Component)]
struct Player {
    speed: f32,
}

fn setup(mut commands: Commands) {
    commands
        .spawn_bundle(SpriteBundle {
            sprite: Sprite {
                color: Color::RED,
                custom_size: Some(Vec2::new(50.0, 50.0)),
                ..default()
            },
            transform: Transform::from_xyz(0.0, 0.0, 0.0),
            ..default()
        })
        .insert(Player { speed: 200.0 });
}

fn move_player(
    time: Res<Time>,
    keyboard_input: Res<Input<KeyCode>>,
    mut query: Query<&mut Transform, With<Player>>,
) {
    for mut transform in query.iter_mut() {
        let mut direction = Vec3::ZERO;
        
        if keyboard_input.pressed(KeyCode::Left) {
            direction.x -= 1.0;
        }
        if keyboard_input.pressed(KeyCode::Right) {
            direction.x += 1.0;
        }
        if keyboard_input.pressed(KeyCode::Up) {
            direction.y += 1.0;
        }
        if keyboard_input.pressed(KeyCode::Down) {
            direction.y -= 1.0;
        }
        
        transform.translation += direction * 200.0 * time.delta_seconds();
    }
}
```

**추천 프로젝트:**
- 간단한 플랫폼 게임
- 퍼즐 게임
- 2D 슈팅 게임
- 간단한 RPG

### 커리어 개발 전략

#### 1. 포트폴리오 구축
**GitHub 프로필 최적화:**
- 프로필 README 작성
- 프로젝트 상세 설명
- 기술 스택 뱃지 추가
- 커밋 그래프 꾸준함 유지

**프로젝트 선정 기준:**
- 실용성 있는 문제 해결
- 코드 품질 중시
- 문서화 완성
- 테스트 커버리지 확보

#### 2. 커뮤니티 참여
**오픈소스 기여 방법:**
- 문서 번역 및 개선
- 버그 리포트 및 수정
- 기능 구현 PR 제출
- 코드 리뷰 참여

**네트워킹:**
- Rust 컨퍼런스 참석
- 온라인 커뮤니티 활동
- 기술 블로그 운영
- 발표 및 튜토리얼 제작

#### 3. 채용 시장 준비
**필수 기술:**
- Rust 핵심 개념 완전 이해
- 선택 분야 전문성
- 시스템 프로그래밍 지식
- 협업 및 커뮤니케이션 능력

**면접 준비:**
- 알고리즘 문제 풀이 (LeetCode)
- 시스템 디자인 기본
- Rust 특유의 개념 설명 연습
- 프로젝트 경험 정리

---

**마지막 조언**: Rust는 단순한 언어가 아니라 안전한 시스템 프로그래밍에 대한 철학입니다. 꾸준한 학습과 실천을 통해 Rust의 강력함을 마음껏 활용할 수 있을 것입니다.

**여정을 응원합니다! 🦀✨**
