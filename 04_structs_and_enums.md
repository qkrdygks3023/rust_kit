# 4. 구조체와 열거형 완벽 가이드

## 🏗️ 구조체 (Structs)

### 1.1 기본 구조체 정의

```rust
// 기본 구조체 정의
struct User {
    username: String,
    email: String,
    age: u32,
    active: bool,
}

fn main() {
    // 구조체 인스턴스 생성
    let user1 = User {
        username: String::from("alice"),
        email: String::from("alice@example.com"),
        age: 30,
        active: true,
    };
    
    // 필드 접근
    println!("사용자: {}", user1.username);
    println!("이메일: {}", user1.email);
    println!("나이: {}", user1.age);
    println!("활성: {}", user1.active);
}
```

### 1.2 구조체 업데이트 문법

```rust
#[derive(Debug)]
struct User {
    username: String,
    email: String,
    age: u32,
    active: bool,
}

fn main() {
    let user1 = User {
        username: String::from("alice"),
        email: String::from("alice@example.com"),
        age: 30,
        active: true,
    };
    
    // 구조체 업데이트 문법
    let user2 = User {
        username: String::from("bob"),
        email: String::from("bob@example.com"),
        ..user1  // 나머지 필드는 user1에서 복사
    };
    
    println!("user1: {:?}", user1);
    println!("user2: {:?}", user2);
    
    // user1.username은 더 이상 접근 불가 (String이 이동했기 때문)
    // println!("{}", user1.username);  // 오류!
}
```

### 1.3 튜플 구조체

```rust
// 튜플 구조체 (필드에 이름이 없음)
struct Color(i32, i32, i32);
struct Point(f64, f64);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0.0, 0.0);
    
    // 필드 접근
    let Color(r, g, b) = black;
    println!("RGB: ({}, {}, {})", r, g, b);
    
    let Point(x, y) = origin;
    println!("좌표: ({}, {})", x, y);
}
```

### 1.4 유닛 구조체

```rust
// 필드가 없는 구조체
struct AlwaysEqual;
struct AlwaysTrue;

fn main() {
    let _subject = AlwaysEqual;
    let _condition = AlwaysTrue;
    
    // 주로 트레잇 마커로 사용됨
}
```

### 1.5 구조체 메서드

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // 연관 함수 (정적 메서드)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
    
    // 인스턴스 메서드 (불변)
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    // 인스턴스 메서드 (가변)
    fn set_width(&mut self, width: u32) {
        self.width = width;
    }
    
    // 소유권을 가져가는 메서드
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
    
    // 소유권을 이동시키는 메서드
    fn into_square(self) -> Rectangle {
        let size = self.width.min(self.height);
        Rectangle {
            width: size,
            height: size,
        }
    }
}

fn main() {
    let mut rect1 = Rectangle::new(30, 50);
    
    println!("사각형: {:?}", rect1);
    println!("넓이: {}", rect1.area());
    
    rect1.set_width(40);
    println!("수정된 너비: {}", rect1.width);
    
    let rect2 = Rectangle::new(20, 30);
    println!("rect1이 rect2를 포함할 수 있는가? {}", rect1.can_hold(&rect2));
    
    let square = rect1.into_square();
    println!("정사각형: {:?}", square);
}
```

### 1.6 여러 impl 블록

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

// 첫 번째 impl 블록
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

// 두 번째 impl 블록 (가능)
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect = Rectangle { width: 30, height: 50 };
    println!("넓이: {}", rect.area());
}
```

## 🎭 열거형 (Enums)

### 2.1 기본 열거형

```rust
// 기본 열거형 정의
enum IpAddr {
    V4,
    V6,
}

fn main() {
    let ip = IpAddr::V4;
    
    match ip {
        IpAddr::V4 => println!("IPv4 주소"),
        IpAddr::V6 => println!("IPv6 주소"),
    }
}
```

### 2.2 데이터를 저장하는 열거형

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),                    // 튜플 형태
    V6(String),                            // 문자열
}

fn main() {
    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
    
    match home {
        IpAddr::V4(a, b, c, d) => {
            println!("IPv4: {}.{}.{}.{}", a, b, c, d);
        }
        IpAddr::V6(address) => {
            println!("IPv6: {}", address);
        }
    }
}
```

### 2.3 구조체와 유사한 열거형

```rust
enum Message {
    Quit,                                   // 유닛
    Move { x: i32, y: i32 },               // 익명 구조체
    Write(String),                          // 튜플
    ChangeColor(i32, i32, i32),             // 튜플
}

impl Message {
    fn process(&self) {
        match self {
            Message::Quit => println!("종료 메시지"),
            Message::Move { x, y } => println!("이동: ({}, {})", x, y),
            Message::Write(text) => println!("쓰기: {}", text),
            Message::ChangeColor(r, g, b) => {
                println!("색상 변경: RGB({}, {}, {})", r, g, b);
            }
        }
    }
}

fn main() {
    let messages = vec![
        Message::Move { x: 10, y: 20 },
        Message::Write(String::from("Hello")),
        Message::ChangeColor(255, 0, 0),
        Message::Quit,
    ];
    
    for message in messages {
        message.process();
    }
}
```

### 2.4 Option 열거형

```rust
// Option은 표준 라이브러리에 내장된 열거형
enum Option<T> {
    Some(T),
    None,
}

fn main() {
    let some_number = Some(5);
    let some_string = Some("문자열");
    let absent_number: Option<i32> = None;
    
    // Option 사용
    let x: i32 = 5;
    let y: Option<i32> = Some(10);
    
    match y {
        Some(i) => println!("y의 값: {}", i),
        None => println!("y에 값 없음"),
    }
    
    // if let으로 Option 처리
    if let Some(value) = y {
        println!("y의 값: {}", value);
    }
    
    // Option 체이닝
    let result = some_number.and_then(|n| {
        if n > 0 {
            Some(n * 2)
        } else {
            None
        }
    });
    
    println!("결과: {:?}", result);
}
```

### 2.5 Result 열거형

```rust
// Result는 표준 라이브러리에 내장된 열거형
enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("0으로 나눌 수 없습니다".to_string())
    } else {
        Ok(a / b)
    }
}

fn main() {
    let result1 = divide(10.0, 2.0);
    let result2 = divide(10.0, 0.0);
    
    match result1 {
        Ok(value) => println!("결과: {}", value),
        Err(error) => println!("오류: {}", error),
    }
    
    match result2 {
        Ok(value) => println!("결과: {}", value),
        Err(error) => println!("오류: {}", error),
    }
    
    // ? 연산자와 Result
    match divide(20.0, 4.0) {
        Ok(result) => println!("20 / 4 = {}", result),
        Err(e) => println!("오류: {}", e),
    }
}
```

## 🎯 패턴 매칭

### 3.1 구조체 패턴 매칭

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };
    
    match p {
        Point { x, y: 0 } => println!("x축 위: {}", x),
        Point { x: 0, y } => println!("y축 위: {}", y),
        Point { x, y } => println!("일반 점: ({}, {})", x, y),
    }
    
    // 구조체 필드 무시
    match p {
        Point { x: _, y: 0 } => println!("x축 위"),
        Point { x: 0, .. } => println!("y축 위"),
        Point { .. } => println!("다른 점"),
    }
}
```

### 3.2 열거형 패턴 매칭

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let messages = vec![
        Message::Move { x: 10, y: 20 },
        Message::Write(String::from("Hello")),
        Message::ChangeColor(255, 0, 0),
        Message::Quit,
    ];
    
    for message in messages {
        match message {
            Message::Quit => println!("종료"),
            Message::Move { x, y } => println!("이동: ({}, {})", x, y),
            Message::Write(text) => println!("쓰기: {}", text),
            Message::ChangeColor(r, g, b) => {
                println!("색상: RGB({}, {}, {})", r, g, b);
            }
        }
    }
}
```

### 3.3 복잡한 패턴

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let point = Point { x: 3, y: 5 };
    
    // 중첩 패턴
    match point {
        Point { x, y: 0 } => println!("x축 위: {}", x),
        Point { x: 0, y } => println!("y축 위: {}", y),
        Point { x, y } if x == y => println!("대각선 위: ({}, {})", x, y),
        Point { x, y } => println!("일반 점: ({}, {})", x, y),
    }
    
    let message = Message::ChangeColor(Color::Rgb(255, 0, 0));
    
    match message {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("RGB 색상 변경: ({}, {}, {})", r, g, b);
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("HSV 색상 변경: ({}, {}, {})", h, s, v);
        }
        _ => println!("다른 메시지"),
    }
}
```

## 🔧 트레잇 구현

### 4.1 기본 트레잇 구현

```rust
#[derive(Debug, Clone, Copy)]  // 자동 트레잇 구현
struct Point {
    x: i32,
    y: i32,
}

// 수동 트레잇 구현
impl PartialEq for Point {
    fn eq(&self, other: &Point) -> bool {
        self.x == other.x && self.y == other.y
    }
}

impl Eq for Point {}

impl std::fmt::Display for Point {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 1, y: 2 };
    let p3 = Point { x: 3, y: 4 };
    
    println!("p1: {}", p1);
    println!("p1 == p2: {}", p1 == p2);
    println!("p1 == p3: {}", p1 == p3);
}
```

### 4.2 커스텀 트레잇

```rust
// 커스텀 트레잇 정의
trait Summary {
    fn summarize(&self) -> String;
    
    // 기본 구현 제공
    fn summarize_verbose(&self) -> String {
        format!("(자세한 내용) {}", self.summarize())
    }
}

// 구조체에 트레잇 구현
#[derive(Debug)]
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

#[derive(Debug)]
struct Tweet {
    username: String,
    content: String,
    reply: bool,
    retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

fn main() {
    let article = NewsArticle {
        headline: String::from("Rust 1.0 출시!"),
        location: String::from("서울, 한국"),
        author: String::from("개발자"),
        content: String::from("Rust 1.0이 공식 출시되었습니다..."),
    };
    
    let tweet = Tweet {
        username: String::from("rustacean"),
        content: String::from("Rust는 정말 멋지다!"),
        reply: false,
        retweet: true,
    };
    
    println!("기사 요약: {}", article.summarize());
    println!("기사 상세: {}", article.summarize_verbose());
    
    println!("트윗 요약: {}", tweet.summarize());
    println!("트윗 상세: {}", tweet.summarize_verbose());
}
```

### 4.3 트레잇 바운드

```rust
trait Summary {
    fn summarize(&self) -> String;
}

fn notify(item: &impl Summary) {  // 트레잇 바운드 (impl 문법)
    println!("속보! {}", item.summarize());
}

fn notify_generic<T: Summary>(item: &T) {  // 제네릭 트레잇 바운드
    println!("속보! {}", item.summarize());
}

// 여러 트레잇 바운드
fn notify_multiple(item: &(impl Summary + std::fmt::Display)) {
    println!("속보! {}", item.summarize());
    println!("표시: {}", item);
}

// where 절 사용
fn some_function<T, U>(_t: T, _u: U) -> i32
where
    T: Summary + std::fmt::Display,
    U: Clone,
{
    42
}

fn main() {
    // notify 함수 사용 예제
}
```

## 🎯 실용적인 예제

### 5.1 도메인 모델링

```rust
// 사용자 상태 열거형
#[derive(Debug, Clone, PartialEq)]
enum UserStatus {
    Active,
    Inactive,
    Suspended,
    Deleted,
}

// 사용자 구조체
#[derive(Debug, Clone)]
struct User {
    id: u64,
    username: String,
    email: String,
    status: UserStatus,
    created_at: std::time::SystemTime,
}

impl User {
    fn new(id: u64, username: String, email: String) -> Self {
        User {
            id,
            username,
            email,
            status: UserStatus::Active,
            created_at: std::time::SystemTime::now(),
        }
    }
    
    fn activate(&mut self) {
        self.status = UserStatus::Active;
    }
    
    fn deactivate(&mut self) {
        self.status = UserStatus::Inactive;
    }
    
    fn is_active(&self) -> bool {
        self.status == UserStatus::Active
    }
}

// 사용자 관리 구조체
#[derive(Debug)]
struct UserManager {
    users: Vec<User>,
}

impl UserManager {
    fn new() -> Self {
        UserManager { users: Vec::new() }
    }
    
    fn add_user(&mut self, user: User) {
        self.users.push(user);
    }
    
    fn find_user_by_id(&self, id: u64) -> Option<&User> {
        self.users.iter().find(|user| user.id == id)
    }
    
    fn find_user_by_id_mut(&mut self, id: u64) -> Option<&mut User> {
        self.users.iter_mut().find(|user| user.id == id)
    }
    
    fn active_users(&self) -> Vec<&User> {
        self.users
            .iter()
            .filter(|user| user.is_active())
            .collect()
    }
}

fn main() {
    let mut manager = UserManager::new();
    
    let user1 = User::new(1, "alice".to_string(), "alice@example.com".to_string());
    let user2 = User::new(2, "bob".to_string(), "bob@example.com".to_string());
    
    manager.add_user(user1.clone());
    manager.add_user(user2.clone());
    
    println!("모든 사용자: {:?}", manager.users);
    
    if let Some(user) = manager.find_user_by_id_mut(1) {
        user.deactivate();
    }
    
    let active_users = manager.active_users();
    println!("활성 사용자: {:?}", active_users);
}
```

### 5.2 상태 머신

```rust
#[derive(Debug, Clone, PartialEq)]
enum ConnectionState {
    Disconnected,
    Connecting,
    Connected,
    Disconnecting,
}

#[derive(Debug)]
struct Connection {
    state: ConnectionState,
    address: String,
}

impl Connection {
    fn new(address: String) -> Self {
        Connection {
            state: ConnectionState::Disconnected,
            address,
        }
    }
    
    fn connect(&mut self) -> Result<(), String> {
        match self.state {
            ConnectionState::Disconnected => {
                self.state = ConnectionState::Connecting;
                println!("{}에 연결 중...", self.address);
                self.state = ConnectionState::Connected;
                Ok(())
            }
            ConnectionState::Connecting => {
                Err("이미 연결 중입니다".to_string())
            }
            ConnectionState::Connected => {
                Err("이미 연결되어 있습니다".to_string())
            }
            ConnectionState::Disconnecting => {
                Err("연결 해제 중입니다".to_string())
            }
        }
    }
    
    fn disconnect(&mut self) -> Result<(), String> {
        match self.state {
            ConnectionState::Connected => {
                self.state = ConnectionState::Disconnecting;
                println!("{}에서 연결 해제 중...", self.address);
                self.state = ConnectionState::Disconnected;
                Ok(())
            }
            ConnectionState::Disconnected => {
                Err("이미 연결 해제되어 있습니다".to_string())
            }
            _ => Err("연결 해제할 수 없는 상태입니다".to_string()),
        }
    }
    
    fn send_data(&self, data: &str) -> Result<(), String> {
        match self.state {
            ConnectionState::Connected => {
                println!("{}로 데이터 전송: {}", self.address, data);
                Ok(())
            }
            _ => Err("연결되어 있지 않습니다".to_string()),
        }
    }
}

fn main() {
    let mut conn = Connection::new("192.168.1.1:8080".to_string());
    
    // 연결 시도
    match conn.connect() {
        Ok(_) => println!("연결 성공"),
        Err(e) => println!("연결 실패: {}", e),
    }
    
    // 데이터 전송
    match conn.send_data("Hello, Server!") {
        Ok(_) => println!("데이터 전송 성공"),
        Err(e) => println!("전송 실패: {}", e),
    }
    
    // 연결 해제
    match conn.disconnect() {
        Ok(_) => println!("연결 해제 성공"),
        Err(e) => println!("연결 해제 실패: {}", e),
    }
    
    println!("최종 상태: {:?}", conn.state);
}
```

## 📝 연습 문제

### 문제 1: 구조체 설계
다음 요구사항을 만족하는 구조체를 설계하세요:

```rust
// TODO: 책을 나타내는 Book 구조체 정의
// 필드: 제목(String), 저자(String), ISBN(String), 출판년도(u32), 가격(f64)

// TODO: Book에 대한 메서드 구현
// - new(): 새 책 생성
// - discount(): 가격 할인
// - display_info(): 책 정보 출력

fn main() {
    // TODO: Book 인스턴스 생성하고 메서드 테스트
}
```

### 문제 2: 열거형 설계
다음 요구사항을 만족하는 열거형을 설계하세요:

```rust
// TODO: 다양한 도형을 나타내는 Shape 열거형 정의
// - Circle(반지름: f64)
// - Rectangle(가로: f64, 세로: f64)
// - Triangle(밑변: f64, 높이: f64)

// TODO: Shape에 대한 메서드 구현
// - area(): 넓이 계산
// - perimeter(): 둘레 계산

fn main() {
    // TODO: 다양한 도형 생성하고 메서드 테스트
}
```

### 문제 3: 트레잇 구현
다음 요구사항을 만족하는 트레잇을 구현하세요:

```rust
// TODO: Drawable 트레잇 정의
// - draw(): 도형 그리기

// TODO: Shape 열거형에 Drawable 트레잇 구현

// TODO: 도형 목록을 관리하는 Canvas 구조체 정의
// - add_shape(): 도형 추가
// - draw_all(): 모든 도형 그리기

fn main() {
    // TODO: Canvas 생성하고 여러 도형 추가한 후 그리기
}
```

---

**다음 단계**: [05_error_handling.md](./05_error_handling.md)에서 Rust의 에러 처리 시스템을 학습하세요! 🦀
