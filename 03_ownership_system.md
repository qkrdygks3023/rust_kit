# 3. Rust 소유권 시스템 완벽 가이드

## 🎯 소유권이란?

Rust의 소유권 시스템은 메모리 안전성을 컴파일 타임에 보장하는 핵심 기능입니다. 가비지 컬렉터 없이도 메모리 누수나 댕글링 포인터 같은 문제를 방지합니다.

### 소유권의 3가지 규칙

1. **Rust의 각 값은 소유자(owner)가 정확히 하나씩 있다**
2. **소유자가 스코프 밖으로 벗어나면 값은 버려진다(dropped)**
3. **소유권은 이동(move)하거나 복사(copy)할 수 있다**

## 🔄 소유권 이동 (Move)

### 1.1 기본 이동

```rust
fn main() {
    let s1 = String::from("Hello");  // s1이 문자열 소유
    let s2 = s1;                      // s1의 소유권이 s2로 이동
    
    // println!("{}", s1);  // 오류! s1은 더 이상 유효하지 않음
    println!("{}", s2);      // 정상: s2가 소유권을 가짐
}  // s2가 스코프를 벗어나 메모리 해제됨
```

### 1.2 함수와 소유권

```rust
fn takes_ownership(some_string: String) {
    println!("{}", some_string);
}  // some_string이 스코프를 벗어나 메모리 해제됨

fn gives_ownership() -> String {
    let some_string = String::from("Yours");
    some_string  // 소유권을 반환
}  // some_string이 이동되므로 메모리 해제되지 않음

fn takes_and_gives_back(a_string: String) -> String {
    a_string  // 소유권을 반환
}

fn main() {
    let s1 = gives_ownership();         // gives_ownership이 반환한 소유권을 s1이 받음
    let s2 = String::from("Hello");
    let s3 = takes_and_gives_back(s2); // s2의 소유권이 함수로 이동했다가 다시 s3로 반환
    
    println!("s1 = {}", s1);
    println!("s3 = {}", s3);
    // println!("s2 = {}", s2);  // 오류! s2는 더 이상 유효하지 않음
}
```

### 1.3 복합 타입에서의 이동

```rust
fn main() {
    // 벡터의 소유권 이동
    let vec1 = vec![1, 2, 3];
    let vec2 = vec1;
    
    // println!("{:?}", vec1);  // 오류!
    println!("{:?}", vec2);      // 정상
    
    // 구조체의 소유권 이동
    struct User {
        username: String,
        email: String,
    }
    
    let user1 = User {
        username: String::from("alice"),
        email: String::from("alice@example.com"),
    };
    
    let user2 = user1;
    // println!("{}", user1.username);  // 오류!
    println!("{}", user2.username);     // 정상
}
```

## 📤 대여 (Borrowing)

### 2.1 불변 대여 (Immutable Borrow)

```rust
fn main() {
    let s1 = String::from("Hello");
    
    // 불변 대여: 소유권을 이동하지 않고 참조만 빌림
    let len = calculate_length(&s1);
    
    println!("'{}'의 길이는 {}입니다", s1, len);  // s1은 여전히 유효함
    
    // 여러 개의 불변 대여 가능
    let r1 = &s1;
    let r2 = &s1;
    let r3 = &s1;
    
    println!("참조들: {}, {}, {}", r1, r2, r3);
}

fn calculate_length(s: &String) -> usize {  // 참조를 매개변수로 받음
    s.len()
}  // s는 스코프를 벗어나지만 소유권이 없으므로 아무 일도 일어나지 않음
```

### 2.2 가변 대여 (Mutable Borrow)

```rust
fn main() {
    let mut s = String::from("Hello");
    
    change(&mut s);  // 가변 참조를 전달
    
    println!("변경된 문자열: {}", s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

### 2.3 대여 규칙

```rust
fn main() {
    let mut s = String::from("Hello");
    
    // ✅ 여러 개의 불변 대여는 가능
    let r1 = &s;
    let r2 = &s;
    println!("불변 참조: {} and {}", r1, r2);
    
    // ❌ 불변 대여가 있는 동안 가변 대여는 불가능
    // let r3 = &mut s;  // 오류!
    
    // ✅ 가변 대여는 하나만 가능
    let r3 = &mut s;
    r3.push_str(", world");
    println!("가변 참조: {}", r3);
    
    // ❌ 가변 대여가 있는 동안 다른 대여는 불가능
    // let r4 = &s;  // 오류!
    
    // ✅ 대여의 스코프가 끝나면 다시 대여 가능
    drop(r3);
    let r4 = &s;
    println!("다시 불변 참조: {}", r4);
}
```

### 2.4 댕글링 참조 방지

```rust
fn main() {
    // ❌ 댕글링 참조 (컴파일 오류)
    // let reference_to_nothing = dangle();
    
    // ✅ 올바른 참조 반환
    let reference = no_dangle();
    println!("참조: {}", reference);
}

// fn dangle() -> &String {  // 오류!
//     let s = String::from("Hello");
//     &s  // s가 스코프를 벗어나 메모리 해제되므로 댕글링 참조
// }

fn no_dangle() -> String {  // 소유권을 반환
    let s = String::from("Hello");
    s  // 소유권 이동
}
```

## ⏰ 수명 (Lifetime)

### 3.1 수명 기초

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
    
    let result = longest(string1.as_str(), string2);
    println!("더 긴 문자열: {}", result);
}

// 수명 주석: 반환값의 수명은 매개변수들의 수명 중 더 짧은 것
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### 3.2 구조체에서의 수명

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
    
    fn announce_and_return_part<'b>(&self, announcement: &'b str) -> &'b str
    where
        'a: 'b,  // 'a는 'b보다 길어야 함
    {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    
    let i = ImportantExcerpt {
        part: first_sentence,
    };
    
    println!("중요한 부분: {}", i.part);
    println!("레벨: {}", i.level());
}
```

### 3.3 수명 생략 (Lifetime Elision)

```rust
// 수명 생략 규칙이 적용되는 경우 (컴파일러가 자동으로 추론)
fn first_word(s: &str) -> &str {  // 실제로는 fn first_word<'a>(s: &'a str) -> &'a str
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    
    s
}

fn main() {
    let s = String::from("Hello world");
    let word = first_word(&s);
    println!("첫 단어: {}", word);
}
```

### 3.4 정적 수명

```rust
fn main() {
    let s: &'static str = "I have a static lifetime.";
    println!("{}", s);
    
    // 문자열 리터럴은 항상 정적 수명을 가짐
    let string_literal = "Hello";
    let reference: &'static str = string_literal;
    
    println!("{}", reference);
}
```

## 📋 복사 (Copy)

### 4.1 Copy 트레잇

```rust
fn main() {
    // 정수 타입은 Copy 트레잇을 구현
    let x = 5;
    let y = x;  // x가 복사됨 (이동 아님)
    
    println!("x = {}, y = {}", x, y);  // 둘 다 사용 가능
    
    // 불리언도 Copy
    let t = true;
    let f = t;
    println!("t = {}, f = {}", t, f);
    
    // 문자(char)도 Copy
    let c1 = 'a';
    let c2 = c1;
    println!("c1 = {}, c2 = {}", c1, c2);
    
    // 튜플도 모든 요소가 Copy이면 Copy
    let tuple1 = (1, 2.0, true);
    let tuple2 = tuple1;
    println!("tuple1 = {:?}, tuple2 = {:?}", tuple1, tuple2);
}
```

### 4.2 Copy vs Clone

```rust
#[derive(Debug, Clone)]  // Clone 트레잇 파생
struct Point {
    x: i32,
    y: i32,
}

// Copy 트레잇 수동 구현
impl Copy for Point {}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = p1;  // Copy (이동 아님)
    
    println!("p1 = {:?}, p2 = {:?}", p1, p2);  // 둘 다 사용 가능
    
    // 명시적 복제
    let p3 = p1.clone();
    println!("p3 = {:?}", p3);
    
    // String은 Copy가 아니라 Clone만 가능
    let s1 = String::from("Hello");
    let s2 = s1.clone();  // 명시적 복제 필요
    println!("s1 = {}, s2 = {}", s1, s2);
}
```

## 🔄 소유권 패턴

### 5.1 반복문에서의 소유권

```rust
fn main() {
    let vec = vec![1, 2, 3, 4, 5];
    
    // ❌ 소유권 이동 (오류)
    // for item in vec {
    //     println!("{}", item);
    // }
    // println!("{:?}", vec);  // 오류! vec이 더 이상 유효하지 않음
    
    // ✅ 참조를 사용한 순회
    for item in &vec {
        println!("{}", item);
    }
    println!("{:?}", vec);  // 정상: vec은 여전히 유효함
    
    // ✅ 가변 참조를 사용한 수정
    let mut vec2 = vec![1, 2, 3];
    for item in &mut vec2 {
        *item *= 2;
    }
    println!("{:?}", vec2);  // [2, 4, 6]
}
```

### 5.2 구조체 필드 접근

```rust
#[derive(Debug)]
struct User {
    username: String,
    email: String,
    age: u32,
    active: bool,
}

impl User {
    fn get_username(&self) -> &String {  // 참조 반환
        &self.username
    }
    
    fn set_age(&mut self, age: u32) {  // 가변 참조
        self.age = age;
    }
}

fn main() {
    let mut user = User {
        username: String::from("alice"),
        email: String::from("alice@example.com"),
        age: 30,
        active: true,
    };
    
    // 메서드 호출 시 자동으로 대여
    let username = user.get_username();
    println!("사용자 이름: {}", username);
    
    user.set_age(31);
    println!("변경된 나이: {}", user.age);
    
    // 필드 직접 접근
    println!("이메일: {}", user.email);
}
```

### 5.3 에러 처리와 소유권

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file_content(filename: &str) -> Result<String, io::Error> {
    let mut file = File::open(filename)?;  // ? 연산자가 소유권 이동 처리
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;   // file의 소유권이 여전히 유효
    Ok(contents)  // contents의 소유권 이동
}

fn main() {
    match read_file_content("test.txt") {
        Ok(content) => println!("파일 내용: {}", content),
        Err(error) => println!("오류: {}", error),
    }
}
```

## 🎯 실용적인 소유권 팁

### 6.1 소유권 최적화

```rust
fn main() {
    // ✅ 참조를 사용하여 불필요한 복제 방지
    let data = String::from("중요한 데이터");
    process_data(&data);  // 참조 전달
    println!("데이터 여전히 사용 가능: {}", data);
    
    // ✅ 필요할 때만 소유권 이동
    let result = transform_data(data);  // 소유권 이동 필요
    println!("변환된 데이터: {}", result);
}

fn process_data(data: &String) {
    println!("처리 중: {}", data);
}

fn transform_data(data: String) -> String {
    format!("변환: {}", data)
}
```

### 6.2 스마트 포인터와 소유권

```rust
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    // Rc: 참조 카운팅 스마트 포인터
    let s = Rc::new(String::from("Hello"));
    let s1 = Rc::clone(&s);  // 참조 카운트 증가
    let s2 = Rc::clone(&s);  // 참조 카운트 증가
    
    println!("참조 카운트: {}", Rc::strong_count(&s));  // 3
    
    // RefCell: 런타임 빌림 검사
    let data = RefCell::new(String::from("World"));
    {
        let mut borrowed = data.borrow_mut();
        borrowed.push_str("!");
    }  // 빌려온 참조가 여기서 해제됨
    
    println!("수정된 데이터: {}", data.borrow());
}
```

## 📝 연습 문제

### 문제 1: 소유권 이해
다음 코드의 오류를 수정하세요:

```rust
fn main() {
    let s1 = String::from("Hello");
    let s2 = s1;
    
    // TODO: 오류를 수정하고 두 문자열을 모두 출력하세요
    println!("s1: {}", s1);  // 오류!
    println!("s2: {}", s2);
}
```

### 문제 2: 함수와 소유권
다음 함수들을 구현하세요:

```rust
// TODO: 문자열의 소유권을 가져와서 길이를 반환하는 함수
fn string_length(s: String) -> usize {
    ___
}

// TODO: 문자열을 참조로 받아서 첫 단어를 반환하는 함수
fn first_word(s: &str) -> &str {
    ___
}

fn main() {
    let text = String::from("Hello world");
    let len = string_length(text);  // 소유권 이동
    println!("길이: {}", len);
    
    // TODO: first_word 함수를 호출하고 결과를 출력하세요
    // let word = first_word(___);  // 어떻게 해야 할까요?
    // println!("첫 단어: {}", word);
}
```

### 문제 3: 수명 주석
다음 함수에 적절한 수명 주석을 추가하세요:

```rust
// TODO: 수명 주석 추가
fn longest_with_an_announcement<'a, 'b>(
    x: &'a str,
    y: &'b str,
    ann: &'a str,
) -> &'a str {
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

---

**다음 단계**: [04_structs_and_enums.md](./04_structs_and_enums.md)에서 구조체와 열거형을 학습하세요! 🦀
