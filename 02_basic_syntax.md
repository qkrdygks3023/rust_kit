# 2. Rust 기본 문법 완벽 가이드

## 📝 변수와 상수

### 1.1 변수 선언

Rust에서 변수는 기본적으로 **불변(immutable)**입니다. 가변으로 만들려면 `mut` 키워드를 사용합니다.

```rust
fn main() {
    // 불변 변수 (기본)
    let x = 5;
    println!("x의 값: {}", x);
    // x = 6;  // 오류! 불변 변수는 재할당 불가
    
    // 가변 변수
    let mut y = 10;
    println!("y의 값: {}", y);
    y = 15;  // 가능
    println!("변경된 y의 값: {}", y);
    
    // 변수 섀도잉 (Shadowing)
    let z = 20;
    let z = z + 5;  // 새로운 변수 선언
    println!("섀도잉된 z: {}", z);
    
    // 타입 변경도 가능
    let spaces = "   ";
    let spaces = spaces.len();  // String에서 usize로
    println!("공백 개수: {}", spaces);
}
```

### 1.2 상수 선언

상수는 항상 불변이며, 타입을 명시해야 합니다.

```rust
// 상수는 반드시 타입 명시
const MAX_POINTS: u32 = 100_000;
const PI: f64 = 3.14159265359;
const SECONDS_IN_MINUTE: u64 = 60;

// 상수는 컴파일 타임에 값이 결정되어야 함
fn main() {
    println!("최대 점수: {}", MAX_POINTS);
    println!("π 값: {}", PI);
    
    // 상수는 전역 스코프에서도 선언 가능
    println!("분당 초: {}", SECONDS_IN_MINUTE);
}
```

### 1.3 변수 명명 규칙

```rust
// 카멜케이스 (camelCase) 사용
let userName = "Alice";
let userAge = 25;
let isLoggedIn = true;

// 상수는 스네이크케이스와 대문자
const DEFAULT_TIMEOUT: u64 = 30000;
const MAX_CONNECTIONS: usize = 1000;

// 함수와 변수도 스네이크케이스가 일반적
fn calculate_user_score(user_id: u32) -> u32 {
    let base_score = 100;
    let bonus_score = 50;
    base_score + bonus_score
}
```

## 🔢 데이터 타입

### 2.1 스칼라 타입

#### 정수 타입
```rust
fn main() {
    // 부호 있는 정수
    let small_int: i8 = 127;        // -128 ~ 127
    let medium_int: i16 = 32_767;   // -32,768 ~ 32,767
    let normal_int: i32 = 2_147_483_647;  // 기본 정수 타입
    let large_int: i64 = 9_223_372_036_854_775_807;
    let very_large_int: i128 = 170_141_183_460_469_231_731_687_303_715_884_105_727;
    
    // 부호 없는 정수
    let byte: u8 = 255;             // 0 ~ 255
    let unsigned_int: u32 = 4_294_967_295;
    let unsigned_large: u64 = 18_446_744_073_709_551_615;
    let unsigned_very_large: u128 = 340_282_366_920_938_463_463_374_607_431_768_211_455;
    
    // 아키텍처 의존적 타입
    let arch_int: isize = 100;      // 32비트: i32, 64비트: i64
    let arch_uint: usize = 100;     // 32비트: u32, 64비트: u64
    
    // 리터럴 표현
    let decimal = 98_222;           // 10진수
    let hex = 0xff;                 // 16진수
    let octal = 0o77;               // 8진수
    let binary = 0b1111_0000;       // 2진수
    let byte_literal = b'A';         // 바이트 리터럴 (u8만 가능)
    
    println!("정수 타입 예제: {} {} {} {}", decimal, hex, octal, binary);
}
```

#### 부동소수점 타입
```rust
fn main() {
    // f32 (단정밀도)
    let float_32: f32 = 3.14159265359;
    
    // f64 (배정밀도 - 기본)
    let float_64 = 2.71828182845904523536;
    
    // 과학적 표기법
    let scientific = 1.0e6;         // 1,000,000
    let small_scientific = 1.0e-6;  // 0.000001
    
    // 특수 값
    let infinity: f64 = f64::INFINITY;
    let neg_infinity: f64 = f64::NEG_INFINITY;
    let nan: f64 = f64::NAN;
    
    println!("부동소수점: {} {} {}", float_32, float_64, scientific);
    
    // 부동소수점 연산
    let result = 0.1 + 0.2;
    println!("0.1 + 0.2 = {}", result);  // 정확히 0.3이 아님!
}
```

#### 불리언 타입
```rust
fn main() {
    let is_rust_awesome: bool = true;
    let is_hard: bool = false;
    
    // 불리언 연산
    let and_result = is_rust_awesome && is_hard;
    let or_result = is_rust_awesome || is_hard;
    let not_result = !is_hard;
    
    println!("AND: {}, OR: {}, NOT: {}", and_result, or_result, not_result);
    
    // 조건문에서 사용
    if is_rust_awesome {
        println!("Rust는 정말 멋지다!");
    }
}
```

#### 문자 타입
```rust
fn main() {
    // char 타입은 유니코드 스칼라 값
    let letter: char = 'A';
    let emoji: char = '🦀';          // Rust 마스코트!
    let korean: char = '한';
    let japanese: char = 'あ';
    let unicode_char: char = '\u{1F600}';  // 😀
    
    // 이스케이프 시퀀스
    let newline = '\n';
    let tab = '\t';
    let backslash = '\\';
    let single_quote = '\'';
    
    println!("문자들: {} {} {} {}", letter, emoji, korean, unicode_char);
}
```

### 2.2 복합 타입

#### 튜플 (Tuple)
```rust
fn main() {
    // 다양한 타입의 튜플
    let person: (String, i32, bool) = ("Alice".to_string(), 30, true);
    let coordinates: (f64, f64) = (3.14, 2.71);
    let mixed_tuple = ("Hello", 100, 3.14, true);
    
    // 튜플 요소 접근
    let name = person.0;
    let age = person.1;
    let is_active = person.2;
    
    // 패턴 매칭으로 분해
    let (x, y) = coordinates;
    println!("좌표: ({}, {})", x, y);
    
    // 튜플 구조체
    struct Color(i32, i32, i32);
    struct Point(f64, f64);
    
    let black = Color(0, 0, 0);
    let origin = Point(0.0, 0.0);
    
    println!("이름: {}, 나이: {}, 활성: {}", name, age, is_active);
}
```

#### 배열 (Array)
```rust
fn main() {
    // 고정 크기 배열
    let numbers: [i32; 5] = [1, 2, 3, 4, 5];
    let months = ["January", "February", "March", "April", "May", "June"];
    
    // 같은 값으로 초기화
    let zeros: [i32; 10] = [0; 10];  // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    let fives = [5; 3];              // [5, 5, 5]
    
    // 배열 요소 접근
    let first = numbers[0];
    let second = numbers[1];
    
    // 배열 길이
    let length = numbers.len();
    
    // 배열 슬라이스
    let slice = &numbers[1..4];  // [2, 3, 4]
    
    println!("첫 번째: {}, 두 번째: {}, 길이: {}", first, second, length);
    
    // 배열 순회
    for (index, value) in numbers.iter().enumerate() {
        println!("인덱스 {}: {}", index, value);
    }
    
    // 런타임 범위 검사 (패닉 발생)
    // let out_of_bounds = numbers[10];  // 패닉!
}
```

## 🎯 함수

### 3.1 함수 정의와 호출

```rust
// 함수 정의 (snake_case)
fn greet(name: &str) {
    println!("안녕하세요, {}님!", name);
}

// 반환값이 있는 함수
fn add(a: i32, b: i32) -> i32 {
    a + b  // 세미콜론 없음 = 표현식 (반환값)
}

// 명시적 반환
fn multiply(a: i32, b: i32) -> i32 {
    return a * b;  // 세미콜론 있음 = 문장
}

// 여러 반환값 (튜플)
fn calculate_stats(numbers: &[i32]) -> (i32, f64, i32) {
    let sum: i32 = numbers.iter().sum();
    let avg: f64 = sum as f64 / numbers.len() as f64;
    let max = numbers.iter().max().unwrap_or(&0);
    
    (sum, avg, *max)
}

fn main() {
    // 함수 호출
    greet("Bob");
    
    let result = add(5, 3);
    println!("5 + 3 = {}", result);
    
    let product = multiply(4, 7);
    println!("4 * 7 = {}", product);
    
    let numbers = [1, 2, 3, 4, 5];
    let (sum, avg, max) = calculate_stats(&numbers);
    println!("합계: {}, 평균: {}, 최대: {}", sum, avg, max);
}
```

### 3.2 표현식 vs 문장

```rust
fn main() {
    // 문장 (Statement): 값을 반환하지 않음
    let y = 6;  // let 키워드는 문장
    
    // 표현식 (Expression): 값을 반환함
    let x = 5;  // 5는 표현식
    
    // 블록도 표현식
    let z = {
        let x = 3;
        x + 1  // 세미콜론 없음 = 표현식
    };
    
    println!("z의 값: {}", z);  // 4
    
    // 함수 호출도 표현식
    let result = add(2, 3);
    
    // 매크로 호출도 표현식
    let message = format!("결과: {}", result);
    
    println!("{}", message);
}

fn add(a: i32, b: i32) -> i32 {
    a + b  // 표현식으로 반환
}
```

### 3.3 매개변수

```rust
// 소유권을 넘기는 매개변수
fn take_ownership(s: String) {
    println!("소유권 넘겨받음: {}", s);
}  // s가 여기서 드롭됨

// 참조를 넘기는 매개변수
fn borrow_string(s: &String) {
    println!("문자열 빌려옴: {}", s);
}  // s의 소유권은 여전히 호출자에게 있음

// 가변 참조
fn modify_string(s: &mut String) {
    s.push_str(" (수정됨)");
}

fn main() {
    let s1 = String::from("Hello");
    take_ownership(s1);  // s1의 소유권이 넘어감
    // println!("{}", s1);  // 오류! s1은 더 이상 유효하지 않음
    
    let s2 = String::from("World");
    borrow_string(&s2);  // 참조만 넘김
    println!("s2는 여전히 사용 가능: {}", s2);
    
    let mut s3 = String::from("Rust");
    modify_string(&mut s3);
    println!("수정된 s3: {}", s3);
}
```

## 🔄 제어문

### 4.1 if 표현식

```rust
fn main() {
    let number = 6;
    
    // 기본 if 문
    if number % 4 == 0 {
        println!("4의 배수입니다");
    } else if number % 3 == 0 {
        println!("3의 배수입니다");
    } else {
        println!("4나 3의 배수가 아닙니다");
    }
    
    // if 표현식 (값을 반환)
    let condition = true;
    let result = if condition { 5 } else { 6 };
    println!("결과: {}", result);
    
    // let 바인딩에서 if 사용
    let score = 85;
    let grade = if score >= 90 {
        'A'
    } else if score >= 80 {
        'B'
    } else if score >= 70 {
        'C'
    } else if score >= 60 {
        'D'
    } else {
        'F'
    };
    
    println!("점수: {}, 등급: {}", score, grade);
    
    // 복잡한 if 표현식
    let message = if score >= 60 {
        if score >= 90 {
            "우수한 성적입니다!"
        } else {
            "합격입니다!"
        }
    } else {
        "불합격입니다. 다음에 다시 시도하세요."
    };
    
    println!("{}", message);
}
```

### 4.2 루프

#### loop 루프
```rust
fn main() {
    let mut counter = 0;
    
    // 무한 루프
    loop {
        counter += 1;
        
        if counter == 3 {
            println!("3번 반복 후 건너뛰기");
            continue;
        }
        
        println!("카운터: {}", counter);
        
        if counter == 5 {
            println!("5번 반복 후 종료");
            break;
        }
    }
    
    // loop에서 값 반환
    let result = loop {
        counter += 1;
        
        if counter == 10 {
            break counter * 2;  // 20 반환
        }
    };
    
    println!("루프 결과: {}", result);
    
    // 루프 라벨
    let mut count = 0;
    'outer: loop {
        println!("외부 루프: {}", count);
        
        let mut inner_count = 0;
        loop {
            println!("  내부 루프: {}", inner_count);
            inner_count += 1;
            
            if inner_count == 3 {
                break 'outer;  // 외부 루프 종료
            }
        }
        
        count += 1;
    }
    
    println!("최종 카운트: {}", count);
}
```

#### while 루프
```rust
fn main() {
    let mut number = 3;
    
    while number != 0 {
        println!("{}!", number);
        number -= 1;
    }
    
    println!("발사!");
    
    // 조건이 참인 동안 반복
    let mut temperature = 25;
    
    while temperature < 30 {
        println!("현재 온도: {}°C", temperature);
        temperature += 1;
    }
    
    println!("목표 온도 도달!");
    
    // while let 패턴
    let mut stack = Vec::new();
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(top) = stack.pop() {
        println!("팝: {}", top);
    }
}
```

#### for 루프
```rust
fn main() {
    // 범위를 이용한 for 루프
    for number in 1..5 {  // 1, 2, 3, 4
        println!("번호: {}", number);
    }
    
    // 포함 범위
    for number in 1..=5 {  // 1, 2, 3, 4, 5
        println!("번호: {}", number);
    }
    
    // 배열/벡터 순회
    let fruits = ["사과", "바나나", "오렌지"];
    
    for fruit in fruits.iter() {
        println!("과일: {}", fruit);
    }
    
    // 인덱스와 함께 순회
    for (index, fruit) in fruits.iter().enumerate() {
        println!("{}번 과일: {}", index + 1, fruit);
    }
    
    // 역순 순회
    for number in (1..=5).rev() {
        println!("카운트다운: {}", number);
    }
    
    // 특정 간격으로 순회
    for number in (0..10).step_by(2) {
        println!("짝수: {}", number);
    }
    
    // 문자열 순회
    let text = "Hello";
    for ch in text.chars() {
        println!("문자: {}", ch);
    }
    
    // 패턴 매칭과 for 루프
    let numbers = [Some(1), None, Some(3), Some(4), None];
    
    for maybe_number in numbers.iter() {
        match maybe_number {
            Some(n) => println!("숫자: {}", n),
            None => println!("값 없음"),
        }
    }
    
    // if let과 for 루프
    for maybe_number in numbers.iter() {
        if let Some(n) = maybe_number {
            println!("숫자: {}", n);
        }
    }
}
```

## 🎭 패턴 매칭

### 5.1 match 표현식

```rust
fn main() {
    let number = 13;
    
    match number {
        1 => println!("하나"),
        2 | 3 | 5 | 7 | 11 => println!("작은 소수"),
        13..=19 => println!("13에서 19 사이"),
        _ => println!("다른 숫자"),
    }
    
    // match로 값 반환
    let message = match number {
        0 => "영",
        1 => "하나",
        2 => "둘",
        _ => "많은 숫자",
    };
    
    println!("메시지: {}", message);
    
    // 구조체 매칭
    struct Point {
        x: i32,
        y: i32,
    }
    
    let origin = Point { x: 0, y: 0 };
    
    match origin {
        Point { x: 0, y: 0 } => println!("원점"),
        Point { x, y: 0 } => println!("x축 위: ({}, 0)", x),
        Point { x: 0, y } => println!("y축 위: (0, {})", y),
        Point { x, y } => println!("일반 점: ({}, {})", x, y),
    }
    
    // 열거형 매칭
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }
    
    let message = Message::Move { x: 10, y: 20 };
    
    match message {
        Message::Quit => println!("종료"),
        Message::Move { x, y } => println!("이동: ({}, {})", x, y),
        Message::Write(text) => println!("쓰기: {}", text),
        Message::ChangeColor(r, g, b) => println!("색상 변경: RGB({}, {}, {})", r, g, b),
    }
}
```

### 5.2 if let과 let else

```rust
fn main() {
    let some_value = Some(5);
    
    // if let - 하나의 패턴만 매칭
    if let Some(value) = some_value {
        println!("값: {}", value);
    } else {
        println!("값 없음");
    }
    
    // let else (Rust 1.65+)
    let Some(value) = some_value else {
        panic!("값이 없습니다!");
    };
    println!("값: {}", value);
    
    // 복잡한 패턴
    struct Point {
        x: i32,
        y: i32,
    }
    
    let point = Some(Point { x: 10, y: 20 });
    
    if let Some(Point { x, y: 0 }) = point {
        println!("x축 위의 점: {}", x);
    } else if let Some(Point { x: 0, y }) = point {
        println!("y축 위의 점: {}", y);
    } else if let Some(Point { x, y }) = point {
        println!("일반 점: ({}, {})", x, y);
    } else {
        println!("점 없음");
    }
}
```

## 📝 연습 문제

### 문제 1: 변수와 타입
다음 코드를 완성하세요:

```rust
fn main() {
    // TODO: 다음 변수들을 선언하고 초기값을 설정하세요
    let age: ___ = 25;
    let name: ___ = "Alice";
    let height: ___ = 170.5;
    let is_student: ___ = true;
    
    // TODO: 튜플을 사용하여 개인 정보를 저장하세요
    let person_info: (___, ___, ___) = (___, ___, ___);
    
    // TODO: 배열을 사용하여 주간 온도를 저장하세요
    let weekly_temps: [___; ___] = [___; 7];
    
    println!("이름: {}, 나이: {}, 키: {}, 학생: {}", name, age, height, is_student);
}
```

### 문제 2: 함수 작성
다음 함수들을 구현하세요:

```rust
// TODO: 두 숫자의 합을 반환하는 함수
fn add_numbers(a: ___, b: ___) -> ___ {
    ___
}

// TODO: 숫자가 짝수인지 확인하는 함수
fn is_even(n: ___) -> ___ {
    ___
}

// TODO: 구의 부피를 계산하는 함수 (4/3 * π * r³)
fn sphere_volume(radius: ___) -> ___ {
    ___
}

fn main() {
    let sum = add_numbers(5, 3);
    println!("5 + 3 = {}", sum);
    
    let even_check = is_even(4);
    println!("4는 짝수인가? {}", even_check);
    
    let volume = sphere_volume(2.0);
    println!("반지름 2인 구의 부피: {:.2}", volume);
}
```

### 문제 3: 제어문 활용
다음 기능들을 구현하세요:

```rust
fn main() {
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // TODO: for 루프를 사용하여 짝수만 출력
    println!("짝수:");
    for ___ in ___.iter() {
        if ___ {
            println!("{}", ___);
        }
    }
    
    // TODO: while 루프를 사용하여 100까지의 피보나치 수열 생성
    println!("\n피보나치 수열:");
    let mut a = 0;
    let mut b = 1;
    while ___ {
        println!("{}", a);
        let temp = a + b;
        a = b;
        b = temp;
    }
    
    // TODO: match를 사용하여 학점 계산
    let score = 85;
    let grade = match score {
        ___ => 'A',
        ___ => 'B',
        ___ => 'C',
        ___ => 'D',
        ___ => 'F',
    };
    println!("\n점수: {}, 학점: {}", score, grade);
}
```

---

**다음 단계**: [03_ownership_system.md](./03_ownership_system.md)에서 Rust의 가장 중요한 개념인 소유권 시스템을 학습하세요! 🦀
