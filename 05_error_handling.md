# 5. Rust 에러 처리 완벽 가이드

## 🚨 에러 처리의 중요성

Rust는 컴파일 타임에 에러 처리를 강제하여 런타임 오류를 최소화합니다. Rust의 에러 처리는 두 가지 주요 개념을 기반으로 합니다:

1. **Panic**: 복구 불가능한 에러
2. **Result**: 복구 가능한 에러

## 💥 Panic

### 1.1 Panic 발생

```rust
fn main() {
    // 명시적 panic!
    panic!("이것은 고의적인 panic입니다!");
    
    // 컴파일 타임 panic!
    // let v = vec![1, 2, 3];
    // v[99];  // 인덱스 범위 초과 panic
    
    // unwrap() panic
    // let nothing: Option<i32> = None;
    // nothing.unwrap();  // panic!
}
```

### 1.2 Panic에서 복구

```rust
use std::panic;

fn main() {
    let result = panic::catch_unwind(|| {
        panic!("이 panic은 잡힐 수 있습니다");
    });
    
    match result {
        Ok(_) => println!("정상 실행"),
        Err(payload) => {
            if let Some(message) = payload.downcast_ref::<String>() {
                println!("Panic 메시지: {}", message);
            } else if let Some(message) = payload.downcast_ref::<&str>() {
                println!("Panic 메시지: {}", message);
            }
        }
    }
    
    println!("프로그램 계속 실행");
}
```

### 1.3 Panic vs Assert

```rust
fn main() {
    let x = 5;
    
    // assert! - 디버그 모드에서만 panic
    assert!(x == 5, "x는 5여야 합니다");
    
    // debug_assert! - 디버그 빌드에서만 확인
    debug_assert!(x == 5);
    
    // assert_eq! - 값 비교
    assert_eq!(x, 5);
    
    // assert_ne! - 값이 다른지 확인
    assert_ne!(x, 10);
    
    println!("모든 assert 통과");
}
```

## 🎯 Result 타입

### 2.1 Result 기본 사용

```rust
// Result 타입 정의
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
    
    // Result 처리
    match result1 {
        Ok(value) => println!("결과: {}", value),
        Err(error) => println!("오류: {}", error),
    }
    
    match result2 {
        Ok(value) => println!("결과: {}", value),
        Err(error) => println!("오류: {}", error),
    }
}
```

### 2.2 Result와 ? 연산자

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file_content(filename: &str) -> Result<String, io::Error> {
    let mut file = File::open(filename)?;  // ? 연산자: Err이면 즉시 반환
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;   // ? 연산자: Err이면 즉시 반환
    Ok(contents)
}

fn main() {
    match read_file_content("test.txt") {
        Ok(content) => println!("파일 내용: {}", content),
        Err(error) => println!("파일 읽기 오류: {}", error),
    }
    
    // main 함수에서 ? 연산자 사용
    // fn main() -> Result<(), Box<dyn std::error::Error>> {
    //     let content = read_file_content("test.txt")?;
    //     println!("파일 내용: {}", content);
    //     Ok(())
    // }
}
```

### 2.3 Result 체이닝

```rust
fn process_number(n: i32) -> Result<String, String> {
    n.checked_mul(2)  // Option<i32> 반환
        .ok_or_else(|| "곱셈 오버플로우".to_string())  // Option -> Result
        .map(|result| result.to_string())  // Result<i32, String> -> Result<String, String>
        .and_then(|s| {  // Result<String, String> -> Result<String, String>
            if s.len() > 5 {
                Ok(format!("처리된 결과: {}", s))
            } else {
                Err("결과가 너무 짧습니다".to_string())
            }
        })
}

fn main() {
    let numbers = [10, 1000000000, 5];
    
    for &n in &numbers {
        match process_number(n) {
            Ok(result) => println!("성공: {}", result),
            Err(error) => println!("실패: {}", error),
        }
    }
}
```

## 🤔 Option 타입

### 3.1 Option 기본 사용

```rust
// Option 타입 정의
enum Option<T> {
    Some(T),
    None,
}

fn find_user_by_id(id: u32) -> Option<String> {
    let users = vec![
        (1, "Alice"),
        (2, "Bob"),
        (3, "Charlie"),
    ];
    
    for (user_id, name) in users {
        if user_id == id {
            return Some(name.to_string());
        }
    }
    
    None
}

fn main() {
    let user1 = find_user_by_id(1);
    let user2 = find_user_by_id(99);
    
    // Option 처리
    match user1 {
        Some(name) => println!("사용자: {}", name),
        None => println!("사용자를 찾을 수 없습니다"),
    }
    
    match user2 {
        Some(name) => println!("사용자: {}", name),
        None => println!("사용자를 찾을 수 없습니다"),
    }
}
```

### 3.2 Option 메서드

```rust
fn main() {
    let maybe_number = Some(5);
    let no_number: Option<i32> = None;
    
    // is_some(), is_none()
    println!("maybe_number.is_some(): {}", maybe_number.is_some());
    println!("no_number.is_none(): {}", no_number.is_none());
    
    // unwrap_or()
    let result1 = maybe_number.unwrap_or(0);
    let result2 = no_number.unwrap_or(0);
    println!("unwrap_or 결과: {}, {}", result1, result2);
    
    // unwrap_or_else()
    let result3 = no_number.unwrap_or_else(|| {
        println!("기본값 계산 중...");
        42
    });
    println!("unwrap_or_else 결과: {}", result3);
    
    // map()
    let doubled = maybe_number.map(|n| n * 2);
    println!("map 결과: {:?}", doubled);
    
    // and_then()
    let processed = maybe_number.and_then(|n| {
        if n > 3 {
            Some(n * 10)
        } else {
            None
        }
    });
    println!("and_then 결과: {:?}", processed);
    
    // filter()
    let filtered = maybe_number.filter(|&n| n > 3);
    println!("filter 결과: {:?}", filtered);
}
```

### 3.3 Option과 Result 변환

```rust
fn option_to_result(option: Option<i32>) -> Result<i32, String> {
    option.ok_or_else(|| "값이 없습니다".to_string())
}

fn result_to_option(result: Result<i32, String>) -> Option<i32> {
    result.ok()
}

fn main() {
    let some_value = Some(42);
    let no_value: Option<i32> = None;
    
    // Option -> Result
    let result1 = option_to_result(some_value);
    let result2 = option_to_result(no_value);
    
    println!("Option -> Result: {:?}, {:?}", result1, result2);
    
    // Result -> Option
    let ok_result = Ok(100);
    let err_result = Err("오류".to_string());
    
    let option1 = result_to_option(ok_result);
    let option2 = result_to_option(err_result);
    
    println!("Result -> Option: {:?}, {:?}", option1, option2);
}
```

## 🛠️ 커스텀 에러 타입

### 4.1 에러 타입 정의

```rust
use std::fmt;

#[derive(Debug)]
enum AppError {
    IoError(std::io::Error),
    ParseError(std::num::ParseIntError),
    ValidationError(String),
    DatabaseError(String),
}

// Display 트레잇 구현
impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::IoError(err) => write!(f, "IO 오류: {}", err),
            AppError::ParseError(err) => write!(f, "파싱 오류: {}", err),
            AppError::ValidationError(msg) => write!(f, "검증 오류: {}", msg),
            AppError::DatabaseError(msg) => write!(f, "데이터베이스 오류: {}", msg),
        }
    }
}

// std::error::Error 트레잇 구현
impl std::error::Error for AppError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            AppError::IoError(err) => Some(err),
            AppError::ParseError(err) => Some(err),
            _ => None,
        }
    }
}

// From 트레잇 구현 (에러 변환)
impl From<std::io::Error> for AppError {
    fn from(err: std::io::Error) -> Self {
        AppError::IoError(err)
    }
}

impl From<std::num::ParseIntError> for AppError {
    fn from(err: std::num::ParseIntError) -> Self {
        AppError::ParseError(err)
    }
}

fn main() {
    let error = AppError::ValidationError("잘못된 입력".to_string());
    println!("에러: {}", error);
    println!("디버그: {:?}", error);
}
```

### 4.2 커스텀 에러 사용

```rust
use std::fs;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    IoError(std::io::Error),
    ParseError(ParseIntError),
    ValidationError(String),
}

impl From<std::io::Error> for AppError {
    fn from(err: std::io::Error) -> Self {
        AppError::IoError(err)
    }
}

impl From<ParseIntError> for AppError {
    fn from(err: ParseIntError) -> Self {
        AppError::ParseError(err)
    }
}

fn read_and_parse_number(filename: &str) -> Result<i32, AppError> {
    let content = fs::read_to_string(filename)?;  // io::Error -> AppError
    let number: i32 = content.trim().parse()?;    // ParseIntError -> AppError
    
    if number < 0 {
        return Err(AppError::ValidationError("음수는 허용되지 않습니다".to_string()));
    }
    
    Ok(number)
}

fn main() {
    match read_and_parse_number("number.txt") {
        Ok(number) => println!("파싱된 숫자: {}", number),
        Err(error) => println!("오류: {:?}", error),
    }
}
```

## 🔄 에러 전파 패턴

### 5.1 에러 전파 기법

```rust
use std::fs;
use std::io;

fn read_config() -> Result<String, io::Error> {
    fs::read_to_string("config.toml")
}

fn parse_config(content: String) -> Result<Config, String> {
    // 설정 파싱 로직
    Ok(Config { /* ... */ })
}

struct Config {
    // 설정 필드
}

// 에러 전파 방법 1: 명시적 매칭
fn load_config_v1() -> Result<Config, String> {
    match read_config() {
        Ok(content) => parse_config(content),
        Err(e) => Err(format!("설정 파일 읽기 실패: {}", e)),
    }
}

// 에러 전파 방법 2: ? 연산자와 map_err
fn load_config_v2() -> Result<Config, String> {
    let content = read_config().map_err(|e| format!("설정 파일 읽기 실패: {}", e))?;
    parse_config(content)
}

// 에러 전파 방법 3: 커스텀 에러 타입
#[derive(Debug)]
enum ConfigError {
    Io(io::Error),
    Parse(String),
}

impl From<io::Error> for ConfigError {
    fn from(err: io::Error) -> Self {
        ConfigError::Io(err)
    }
}

fn load_config_v3() -> Result<Config, ConfigError> {
    let content = fs::read_to_string("config.toml")?;  // io::Error -> ConfigError
    parse_config(content).map_err(ConfigError::Parse)  // String -> ConfigError
}

fn main() {
    // 각 버전 테스트
}
```

### 5.2 복합 에러 처리

```rust
use std::fs;
use std::io;

#[derive(Debug)]
enum ProcessingError {
    IoError(io::Error),
    ParseError(String),
    ValidationError(String),
    NetworkError(String),
}

impl From<io::Error> for ProcessingError {
    fn from(err: io::Error) -> Self {
        ProcessingError::IoError(err)
    }
}

fn process_data(filename: &str) -> Result<String, ProcessingError> {
    // 1. 파일 읽기
    let content = fs::read_to_string(filename)?;
    
    // 2. 데이터 파싱
    let parsed = parse_data(&content)?;
    
    // 3. 데이터 검증
    validate_data(&parsed)?;
    
    // 4. 네트워크 전송
    send_data(&parsed)?;
    
    Ok("처리 완료".to_string())
}

fn parse_data(content: &str) -> Result<Vec<i32>, ProcessingError> {
    content
        .lines()
        .map(|line| line.parse::<i32>())
        .collect::<Result<Vec<_>, _>>()
        .map_err(|e| ProcessingError::ParseError(format!("파싱 오류: {}", e)))
}

fn validate_data(data: &[i32]) -> Result<(), ProcessingError> {
    if data.is_empty() {
        return Err(ProcessingError::ValidationError("데이터가 비어있습니다".to_string()));
    }
    
    if data.len() > 1000 {
        return Err(ProcessingError::ValidationError("데이터가 너무 많습니다".to_string()));
    }
    
    Ok(())
}

fn send_data(data: &[i32]) -> Result<(), ProcessingError> {
    // 네트워크 전송 시뮬레이션
    if data.len() % 2 == 0 {
        Ok(())
    } else {
        Err(ProcessingError::NetworkError("네트워크 오류".to_string()))
    }
}

fn main() {
    match process_data("data.txt") {
        Ok(result) => println!("성공: {}", result),
        Err(error) => println!("오류: {:?}", error),
    }
}
```

## 🎯 에러 처리 모범 사례

### 6.1 에러 처리 원칙

```rust
// 원칙 1: 명확한 에러 메시지
#[derive(Debug)]
enum ValidationError {
    EmptyField(String),
    InvalidFormat(String),
    OutOfRange { field: String, value: String, min: i64, max: i64 },
}

impl std::fmt::Display for ValidationError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            ValidationError::EmptyField(field) => {
                write!(f, "{} 필드는 비어있을 수 없습니다", field)
            }
            ValidationError::InvalidFormat(field) => {
                write!(f, "{} 필드의 형식이 올바르지 않습니다", field)
            }
            ValidationError::OutOfRange { field, value, min, max } => {
                write!(f, "{} 필드의 값({})은 {}에서 {} 사이여야 합니다", 
                       field, value, min, max)
            }
        }
    }
}

// 원칙 2: 적절한 에러 수준
fn process_user_input(input: &str) -> Result<i32, ValidationError> {
    if input.is_empty() {
        return Err(ValidationError::EmptyField("입력".to_string()));
    }
    
    let number = input.parse::<i32>()
        .map_err(|_| ValidationError::InvalidFormat("숫자".to_string()))?;
    
    if number < 1 || number > 100 {
        return Err(ValidationError::OutOfRange {
            field: "숫자".to_string(),
            value: number.to_string(),
            min: 1,
            max: 100,
        });
    }
    
    Ok(number)
}

// 원칙 3: 컨텍스트 정보 추가
use std::error::Error;
use std::fmt;

#[derive(Debug)]
struct ContextError {
    context: String,
    source: Box<dyn Error + Send + Sync>,
}

impl fmt::Display for ContextError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}: {}", self.context, self.source)
    }
}

impl Error for ContextError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        Some(&*self.source)
    }
}

fn with_context<T, E>(result: Result<T, E>, context: &str) -> Result<T, ContextError>
where
    E: Error + Send + Sync + 'static,
{
    result.map_err(|e| ContextError {
        context: context.to_string(),
        source: Box::new(e),
    })
}

fn main() {
    match process_user_input("150") {
        Ok(number) => println!("처리된 숫자: {}", number),
        Err(error) => println!("검증 오류: {}", error),
    }
}
```

### 6.2 에러 로깅

```rust
use std::fs;
use std::io;
use std::time::{SystemTime, UNIX_EPOCH};

fn log_error(error: &str) {
    let timestamp = SystemTime::now()
        .duration_since(UNIX_EPOCH)
        .unwrap()
        .as_secs();
    
    let log_entry = format!("[{}] ERROR: {}\n", timestamp, error);
    
    // 로그 파일에 쓰기 (실패해도 프로그램은 계속 실행)
    if let Err(e) = fs::write("error.log", log_entry) {
        eprintln!("로그 쓰기 실패: {}", e);
    }
    
    // 콘솔에도 출력
    eprintln!("에러: {}", error);
}

fn safe_file_operation(filename: &str) -> Result<String, Box<dyn Error>> {
    match fs::read_to_string(filename) {
        Ok(content) => Ok(content),
        Err(e) => {
            let error_msg = format!("파일 읽기 실패 ({}): {}", filename, e);
            log_error(&error_msg);
            Err(Box::new(e))
        }
    }
}

fn main() {
    match safe_file_operation("nonexistent.txt") {
        Ok(content) => println!("파일 내용: {}", content),
        Err(e) => println!("작업 실패: {}", e),
    }
}
```

## 📝 연습 문제

### 문제 1: Result와 Option
다음 함수들을 구현하세요:

```rust
// TODO: 문자열을 숫자로 변환하는 함수 (Result 반환)
fn string_to_number(s: &str) -> Result<i32, String> {
    ___
}

// TODO: 숫자가 짝수인지 확인하는 함수 (Option 반환)
fn is_even(n: i32) -> Option<bool> {
    ___
}

// TODO: 두 함수를 조합하여 문자열이 짝수인지 확인하는 함수
fn is_even_string(s: &str) -> Result<bool, String> {
    ___
}

fn main() {
    let test_cases = ["42", "hello", "100", "abc"];
    
    for case in test_cases {
        match is_even_string(case) {
            Ok(true) => println!("{}은 짝수입니다", case),
            Ok(false) => println!("{}은 홀수입니다", case),
            Err(e) => println!("{}: 오류 - {}", case, e),
        }
    }
}
```

### 문제 2: 커스텀 에러 타입
다음 요구사항을 만족하는 에러 타입을 구현하세요:

```rust
// TODO: BankAccount 에러 타입 정의
// - InsufficientFunds(잔액: i64, 시도금액: i64)
// - InvalidAmount(String)
// - AccountNotFound(String)

// TODO: Display 트레잇 구현

// TODO: BankAccount 구조체 정의
// - account_number: String
// - balance: i64

// TODO: BankAccount 메서드 구현
// - deposit(): 입금
// - withdraw(): 출금
// - get_balance(): 잔액 조회

fn main() {
    // TODO: 계좌 생성 및 다양한 상황 테스트
}
```

### 문제 3: 에러 처리 체이닝
다음 요구사항을 만족하는 함수를 구현하세요:

```rust
// TODO: 파일 처리 파이프라인 함수
// 1. 파일 읽기
// 2. 내용 파싱 (숫자 목록)
// 3. 숫자 필터링 (양수만)
// 4. 합계 계산

#[derive(Debug)]
enum ProcessingError {
    IoError(std::io::Error),
    ParseError(String),
    ValidationError(String),
}

// TODO: 필요한 From 구현

fn process_numbers_file(filename: &str) -> Result<i64, ProcessingError> {
    // TODO: 파이프라인 구현
    ___
}

fn main() {
    // TODO: 테스트용 파일 생성 및 함수 테스트
}
```

---

**다음 단계**: [06_generics_and_traits.md](./06_generics_and_traits.md)에서 제네릭과 트레잇을 학습하세요! 🦀
