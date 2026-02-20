# 6. 제네릭과 트레잇 완벽 가이드

## 🧬 제네릭 (Generics)

### 1.1 제네릭 함수

```rust
// 제네릭 함수 정의
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("가장 큰 숫자: {}", result);
    
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
    println!("가장 큰 문자: {}", result);
}
```

### 1.2 제네릭 구조체

```rust
#[derive(Debug, PartialEq)]
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn new(x: T, y: T) -> Self {
        Point { x, y }
    }
    
    fn x(&self) -> &T {
        &self.x
    }
    
    fn y(&self) -> &T {
        &self.y
    }
}

// 특정 타입에 대한 메서드 구현
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

fn main() {
    let integer_point = Point::new(5, 10);
    let float_point = Point::new(1.0, 4.0);
    
    println!("정수점: {:?}", integer_point);
    println!("실수점: {:?}", float_point);
    
    println!("정수점 x: {}", integer_point.x());
    println!("실수점 원점으로부터의 거리: {}", float_point.distance_from_origin());
}
```

### 1.3 제네릭 열거형

```rust
#[derive(Debug)]
enum Result<T, E> {
    Ok(T),
    Err(E),
}

#[derive(Debug)]
enum Option<T> {
    Some(T),
    None,
}

#[derive(Debug)]
enum MyOption<T> {
    Some(T),
    None,
}

impl<T> MyOption<T> {
    fn unwrap(self) -> T {
        match self {
            MyOption::Some(value) => value,
            MyOption::None => panic!("unwrap called on None"),
        }
    }
    
    fn map<U, F>(self, f: F) -> MyOption<U>
    where
        F: FnOnce(T) -> U,
    {
        match self {
            MyOption::Some(value) => MyOption::Some(f(value)),
            MyOption::None => MyOption::None,
        }
    }
}

fn main() {
    let some_value = MyOption::Some(5);
    let no_value: MyOption<i32> = MyOption::None;
    
    println!("some_value: {:?}", some_value);
    println!("no_value: {:?}", no_value);
    
    let doubled = some_value.map(|x| x * 2);
    println!("doubled: {:?}", doubled);
    
    // println!("unwrap: {}", no_value.unwrap());  // panic!
}
```

### 1.4 제네릭 메서드

```rust
#[derive(Debug)]
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };
    
    let p3 = p1.mixup(p2);
    println!("p3: {:?}", p3);
}
```

## 🎭 트레잇 (Traits)

### 2.1 트레잇 정의

```rust
// 트레잇 정의
trait Summary {
    fn summarize(&self) -> String;
    
    // 기본 구현 제공
    fn summarize_verbose(&self) -> String {
        format!("(자세한 내용) {}", self.summarize())
    }
}

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
    
    // 기본 구현을 오버라이드
    fn summarize_verbose(&self) -> String {
        format!("트윗: {} | 답글: {} | 리트윗: {}", 
                self.summarize(), self.reply, self.retweet)
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

### 2.2 트레잇을 매개변수로 사용

```rust
trait Summary {
    fn summarize(&self) -> String;
}

// 방법 1: impl 문법
fn notify(item: &impl Summary) {
    println!("속보! {}", item.summarize());
}

// 방법 2: 트레잇 바운드 문법
fn notify_generic<T: Summary>(item: &T) {
    println!("속보! {}", item.summarize());
}

// 방법 3: where 절
fn some_function<T, U>(_t: T, _u: U) -> i32
where
    T: Summary + Display,
    U: Clone,
{
    42
}

// 여러 트레잇 바운드
fn notify_multiple(item: &(impl Summary + Display)) {
    println!("속보! {}", item.summarize());
}

fn main() {
    // notify 함수 사용 예제
}
```

### 2.3 트레잇을 반환값으로 사용

```rust
trait Summary {
    fn summarize(&self) -> String;
}

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

// impl 문법으로 트레잇 반환
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}

// 동적 디스패치 (트레잇 객체)
fn returns_summarizable_dynamic(switch: bool) -> Box<dyn Summary> {
    if switch {
        Box::new(NewsArticle {
            headline: String::from("Penguins win the Stanley Cup Championship!"),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from("The Pittsburgh Penguins once again are the best hockey team in the NHL."),
        })
    } else {
        Box::new(Tweet {
            username: String::from("horse_ebooks"),
            content: String::from("of course, as you probably already know, people"),
            reply: false,
            retweet: false,
        })
    }
}

fn main() {
    let tweet = returns_summarizable();
    println!("트윗 요약: {}", tweet.summarize());
    
    let article = returns_summarizable_dynamic(true);
    println!("기사 요약: {}", article.summarize());
}
```

## 🔧 표준 라이브러리 트레잇

### 3.1 Clone과 Copy

```rust
#[derive(Debug, Clone, Copy)]
struct Point {
    x: i32,
    y: i32,
}

#[derive(Debug, Clone)]
struct StringHolder {
    data: String,
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = p1;  // Copy (소유권 이동 아님)
    
    println!("p1: {:?}, p2: {:?}", p1, p2);  // 둘 다 사용 가능
    
    let s1 = StringHolder {
        data: String::from("Hello"),
    };
    let s2 = s1.clone();  // 명시적 복제 필요
    
    println!("s1: {:?}, s2: {:?}", s1, s2);
}
```

### 3.2 Display와 Debug

```rust
use std::fmt;

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

fn main() {
    let point = Point { x: 3, y: 4 };
    
    println!("Debug: {:?}", point);      // Debug 트레잇
    println!("Display: {}", point);     // Display 트레잇
}
```

### 3.3 PartialEq와 Eq

```rust
#[derive(Debug, PartialEq, Eq)]
struct Point {
    x: i32,
    y: i32,
}

#[derive(Debug, PartialEq)]
struct FloatPoint {
    x: f64,
    y: f64,
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 1, y: 2 };
    let p3 = Point { x: 3, y: 4 };
    
    println!("p1 == p2: {}", p1 == p2);  // true
    println!("p1 == p3: {}", p1 == p3);  // false
    
    let fp1 = FloatPoint { x: 0.1 + 0.2, y: 0.3 };
    let fp2 = FloatPoint { x: 0.3, y: 0.3 };
    
    println!("fp1 == fp2: {}", fp1 == fp2);  // false (부동소수점 비교)
}
```

### 3.4 PartialOrd와 Ord

```rust
#[derive(Debug, PartialOrd, Ord, PartialEq, Eq)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let p1 = Person {
        name: String::from("Alice"),
        age: 30,
    };
    
    let p2 = Person {
        name: String::from("Bob"),
        age: 25,
    };
    
    println!("p1 > p2: {}", p1 > p2);  // true (나이로 비교)
    
    let mut people = vec![p2, p1];
    people.sort();  // Ord 트레잇으로 정렬
    
    println!("정렬된 사람들: {:?}", people);
}
```

### 3.5 Hash

```rust
use std::collections::HashMap;
use std::hash::{Hash, Hasher};

#[derive(Debug, Hash, Eq, PartialEq)]
struct Student {
    id: u32,
    name: String,
}

fn main() {
    let student1 = Student {
        id: 1,
        name: String::from("Alice"),
    };
    
    let student2 = Student {
        id: 2,
        name: String::from("Bob"),
    };
    
    let mut grades = HashMap::new();
    grades.insert(student1, 95);
    grades.insert(student2, 87);
    
    for (student, grade) in &grades {
        println!("학생 {} (ID: {}): 점수 {}", student.name, student.id, grade);
    }
}
```

## 🎯 트레잇 객체와 동적 디스패치

### 4.1 트레잇 객체 기본

```rust
trait Draw {
    fn draw(&self);
}

struct Circle {
    radius: f64,
}

impl Draw for Circle {
    fn draw(&self) {
        println!("원 그리기 (반지름: {})", self.radius);
    }
}

struct Rectangle {
    width: f64,
    height: f64,
}

impl Draw for Rectangle {
    fn draw(&self) {
        println!("사각형 그리기 (가로: {}, 세로: {})", self.width, self.height);
    }
}

fn main() {
    let circle = Circle { radius: 5.0 };
    let rectangle = Rectangle { width: 10.0, height: 20.0 };
    
    // 트레잇 객체 벡터
    let shapes: Vec<Box<dyn Draw>> = vec![
        Box::new(circle),
        Box::new(rectangle),
    ];
    
    for shape in &shapes {
        shape.draw();  // 동적 디스패치
    }
}
```

### 4.2 트레잇 객체와 수명

```rust
trait Animal {
    fn make_sound(&self);
}

struct Dog {
    name: String,
}

impl Animal for Dog {
    fn make_sound(&self) {
        println!("{}가 멍멍!", self.name);
    }
}

struct Cat {
    name: String,
}

impl Animal for Cat {
    fn make_sound(&self) {
        println!("{}가 야옹!", self.name);
    }
}

// 트레잇 객체 반환
fn create_animal(sound: &str) -> Box<dyn Animal> {
    match sound {
        "dog" => Box::new(Dog { name: "바둑이".to_string() }),
        "cat" => Box::new(Cat { name: "나비".to_string() }),
        _ => panic!("알 수 없는 동물"),
    }
}

fn main() {
    let dog = create_animal("dog");
    let cat = create_animal("cat");
    
    dog.make_sound();
    cat.make_sound();
}
```

### 4.3 객체 안전성

```rust
// 객체 안전한 트레잇
trait Printable {
    fn print(&self);
}

// 객체 안전하지 않음 (제네릭 메서드)
// trait NotObjectSafe {
//     fn generic_method<T>(&self, item: T);
// }

// 객체 안전하지 않음 (Self 반환)
// trait NotObjectSafe {
//     fn clone_self(&self) -> Self;
// }

// 객체 안전하지 않음 (연관 타입)
// trait NotObjectSafe {
//     type Output;
//     fn get_output(&self) -> Self::Output;
// }

struct Document {
    content: String,
}

impl Printable for Document {
    fn print(&self) {
        println!("문서 내용: {}", self.content);
    }
}

fn print_item(item: &dyn Printable) {
    item.print();
}

fn main() {
    let doc = Document {
        content: "Rust는 멋진 언어입니다".to_string(),
    };
    
    print_item(&doc);
}
```

## 🏗️ 고급 트레잇 패턴

### 5.1 트레잇 바운드와 수명

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
    let announcement = "Comparing strings";
    
    let result = longest_with_an_announcement(&string1, string2, announcement);
    println!("Longest: {}", result);
}
```

### 5.2 블랭킷 구현 (Blanket Implementations)

```rust
use std::fmt::Display;

// 모든 Display를 구현하는 타입에 대한 Summary 구현
trait Summary {
    fn summarize(&self) -> String;
}

impl<T> Summary for T
where
    T: Display,
{
    fn summarize(&self) -> String {
        format!("요약: {}", self)
    }
}

fn main() {
    let number = 42;
    let text = "Hello";
    
    println!("숫자 요약: {}", number.summarize());
    println!("텍스트 요약: {}", text.summarize());
}
```

### 5.3 연관 타입 (Associated Types)

```rust
trait Iterator {
    type Item;  // 연관 타입
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {
    current: usize,
    max: usize,
}

impl Iterator for Counter {
    type Item = usize;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max {
            let current = self.current;
            self.current += 1;
            Some(current)
        } else {
            None
        }
    }
}

fn main() {
    let mut counter = Counter { current: 0, max: 5 };
    
    while let Some(value) = counter.next() {
        println!("카운터: {}", value);
    }
}
```

### 5.4 제네릭 타입 vs 연관 타입

```rust
// 제네릭 타입 사용
trait GenericIterator<T> {
    fn next(&mut self) -> Option<T>;
}

// 연관 타입 사용
trait AssociatedIterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter;

impl GenericIterator<usize> for Counter {
    fn next(&mut self) -> Option<usize> {
        Some(42)  // 단순화된 예제
    }
}

impl AssociatedIterator for Counter {
    type Item = usize;
    
    fn next(&mut self) -> Option<Self::Item> {
        Some(42)  // 단순화된 예제
    }
}

fn main() {
    // 제네릭 타입은 각 구현마다 다른 타입을 지정할 수 있음
    // 연관 타입은 타입당 하나의 구현만 가짐
}
```

## 🎯 실용적인 예제

### 6.1 벡용 컬렉션

```rust
use std::fmt::Display;

trait Container<T> {
    fn add(&mut self, item: T);
    fn get(&self, index: usize) -> Option<&T>;
    fn len(&self) -> usize;
    fn is_empty(&self) -> bool;
}

#[derive(Debug)]
struct MyVec<T> {
    items: Vec<T>,
}

impl<T> MyVec<T> {
    fn new() -> Self {
        MyVec { items: Vec::new() }
    }
}

impl<T> Container<T> for MyVec<T> {
    fn add(&mut self, item: T) {
        self.items.push(item);
    }
    
    fn get(&self, index: usize) -> Option<&T> {
        self.items.get(index)
    }
    
    fn len(&self) -> usize {
        self.items.len()
    }
    
    fn is_empty(&self) -> bool {
        self.items.is_empty()
    }
}

impl<T: Display> MyVec<T> {
    fn print_all(&self) {
        for (i, item) in self.items.iter().enumerate() {
            println!("{}: {}", i, item);
        }
    }
}

fn main() {
    let mut numbers = MyVec::new();
    numbers.add(1);
    numbers.add(2);
    numbers.add(3);
    
    println!("길이: {}", numbers.len());
    println!("비어있음: {}", numbers.is_empty());
    numbers.print_all();
    
    let mut strings = MyVec::new();
    strings.add("Hello".to_string());
    strings.add("World".to_string());
    
    strings.print_all();
}
```

### 6.2 플러그인 시스템

```rust
trait Plugin {
    fn name(&self) -> &str;
    fn execute(&self);
    fn version(&self) -> &str {
        "1.0.0"
    }
}

struct LoggerPlugin;

impl Plugin for LoggerPlugin {
    fn name(&self) -> &str {
        "Logger"
    }
    
    fn execute(&self) {
        println!("로깅 플러그인 실행 중...");
    }
    
    fn version(&self) -> &str {
        "2.1.0"
    }
}

struct DatabasePlugin;

impl Plugin for DatabasePlugin {
    fn name(&self) -> &str {
        "Database"
    }
    
    fn execute(&self) {
        println!("데이터베이스 플러그인 실행 중...");
    }
}

struct PluginManager {
    plugins: Vec<Box<dyn Plugin>>,
}

impl PluginManager {
    fn new() -> Self {
        PluginManager { plugins: Vec::new() }
    }
    
    fn add_plugin<P: Plugin + 'static>(&mut self, plugin: P) {
        self.plugins.push(Box::new(plugin));
    }
    
    fn execute_all(&self) {
        println!("모든 플러그인 실행:");
        for plugin in &self.plugins {
            println!("실행: {} (v{})", plugin.name(), plugin.version());
            plugin.execute();
        }
    }
    
    fn find_plugin(&self, name: &str) -> Option<&dyn Plugin> {
        self.plugins.iter().find(|p| p.name() == name).map(|p| p.as_ref())
    }
}

fn main() {
    let mut manager = PluginManager::new();
    
    manager.add_plugin(LoggerPlugin);
    manager.add_plugin(DatabasePlugin);
    
    manager.execute_all();
    
    if let Some(logger) = manager.find_plugin("Logger") {
        println!("로거 플러그인 찾음: {}", logger.name());
    }
}
```

## 📝 연습 문제

### 문제 1: 제네릭 구조체
다음 요구사항을 만족하는 제네릭 구조체를 구현하세요:

```rust
// TODO: Stack<T> 제네릭 구조체 정의
// - items: Vec<T>
// - push(): 아이템 추가
// - pop(): 아이템 제거 및 반환
// - peek(): 최상단 아이템 참조
// - is_empty(): 비어있는지 확인

// TODO: 특정 타입에 대한 추가 메서드 구현
// - Stack<i32>에 sum() 메서드 추가

fn main() {
    // TODO: 다양한 타입의 스택 테스트
}
```

### 문제 2: 트레잇 설계
다음 요구사항을 만족하는 트레잇을 설계하세요:

```rust
// TODO: Drawable 트레잇 정의
// - draw(): 도형 그리기
// - area(): 넓이 계산

// TODO: 여러 도형 구조체 정의 및 트레잇 구현
// - Circle, Rectangle, Triangle

// TODO: 도형 관리자 구조체 정의
// - add_shape(): 도형 추가
// - draw_all(): 모든 도형 그리기
// - total_area(): 전체 넓이 계산

fn main() {
    // TODO: 다양한 도형 생성 및 테스트
}
```

### 문제 3: 트레잇 객체
다음 요구사항을 만족하는 트레잇 객체 시스템을 구현하세요:

```rust
// TODO: MessageHandler 트레잇 정의
// - handle(): 메시지 처리

// TODO: 다양한 핸들러 구현
// - EmailHandler, SmsHandler, PushHandler

// TODO: MessageRouter 구조체 정의
// - add_handler(): 핸들러 추가
// - route_message(): 메시지 라우팅

fn main() {
    // TODO: 라우터 생성 및 다양한 핸들러 테스트
}
```

---

**다음 단계**: [07_collections.md](./07_collections.md)에서 Rust의 컬렉션 타입을 학습하세요! 🦀
