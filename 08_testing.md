# 8. Rust 테스트 완벽 가이드

## 🧪 테스트의 중요성

Rust는 내장된 테스트 프레임워크를 제공하여 코드의 정확성을 보장하고 리팩토링을 안전하게 만듭니다. Rust의 테스트 시스템은 다음과 같은 특징을 가집니다:

- **내장 테스트 프레임워크**: 별도의 의존성 없이 테스트 작성 가능
- **단위 테스트**: 개별 함수/모듈 테스트
- **통합 테스트**: 여러 모듈이 함께 작동하는지 테스트
- **문서 테스트**: 코드 예제가 실제로 동작하는지 테스트

## ✅ 단위 테스트

### 1.1 기본 테스트 작성

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
    
    #[test]
    fn another() {
        panic!("이 테스트는 실패할 것입니다");
    }
}

// 테스트할 함수
fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }
    
    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }
    
    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

### 1.2 테스트 실행

```bash
# 모든 테스트 실행
cargo test

# 특정 테스트 실행
cargo test test_name

# 특정 모듈의 테스트 실행
cargo test module_name

# 테스트 출력 표시
cargo test -- --nocapture

# 단일 스레드로 테스트 실행
cargo test -- --test-threads=1

# 특정 테스트 무시
cargo test --ignore test_name

# 테스트 필터링
cargo test add
```

### 1.3 Assert 매크로

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert!(true);  // 조건이 참이면 통과
        
        let x = 5;
        assert_eq!(x, 5);  // 값이 같으면 통과
        assert_ne!(x, 6);  // 값이 다르면 통과
    }
    
    #[test]
    fn custom_message() {
        let result = 2 + 2;
        assert_eq!(
            result, 
            4, 
            "2 더하기 2는 4여야 합니다. 실제 결과: {}", result
        );
    }
    
    #[test]
    #[should_panic]  // panic이 발생해야 통과
    fn it_panics() {
        panic!("이 테스트는 panic을 발생시킵니다");
    }
    
    #[test]
    #[should_panic(expected = "Index out of bounds")]
    fn it_panics_with_message() {
        let v = vec![1, 2, 3];
        v[99];  // panic 발생
    }
}
```

### 1.4 Result 타입을 반환하는 테스트

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("두 수를 더한 결과가 4가 아닙니다"))
        }
    }
    
    #[test]
    fn result_with_operations() -> Result<(), String> {
        let result = std::fs::read_to_string("test.txt")
            .map_err(|e| format!("파일 읽기 실패: {}", e))?;
        
        if result.is_empty() {
            return Err("파일이 비어있습니다".to_string());
        }
        
        Ok(())
    }
}
```

## 🔧 테스트 구성과 조직

### 2.1 테스트 모듈 구성

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
    
    #[test]
    fn external() {
        assert_eq!(4, add_two(2));
    }
}
```

### 2.2 테스트 헬퍼 함수

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;
    
    // 테스트 헬퍼 함수
    fn setup_test_data() -> Vec<i32> {
        vec![1, 2, 3, 4, 5]
    }
    
    #[test]
    fn test_with_helper() {
        let data = setup_test_data();
        assert_eq!(data.len(), 5);
        
        for item in &data {
            let result = add_two(*item);
            assert!(result > *item);
        }
    }
}
```

### 2.3 테스트 공통 설정

```rust
// 테스트 공통 모듈
#[cfg(test)]
mod common {
    pub struct TestData {
        pub input: Vec<i32>,
        pub expected: i32,
    }
    
    impl TestData {
        pub fn new(input: Vec<i32>, expected: i32) -> Self {
            TestData { input, expected }
        }
    }
    
    pub fn setup() -> TestData {
        TestData::new(vec![1, 2, 3], 6)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use super::common::*;
    
    #[test]
    fn test_with_common_setup() {
        let data = setup();
        let sum: i32 = data.input.iter().sum();
        assert_eq!(sum, data.expected);
    }
}
```

## 🌐 통합 테스트

### 3.1 통합 테스트 구조

```
my_project/
├── src/
│   ├── lib.rs
│   └── main.rs
└── tests/
    ├── integration_test.rs
    └── common/
        └── mod.rs
```

### 3.2 통합 테스트 작성

```rust
// src/lib.rs
pub fn add_two(a: i32) -> i32 {
    a + 2
}

pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

pub fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

// tests/integration_test.rs
use adder;

#[test]
fn test_add_two() {
    assert_eq!(4, adder::add_two(2));
}

#[test]
fn test_greeting() {
    let result = adder::greeting("World");
    assert!(result.contains("World"));
    assert!(result.contains("Hello"));
}

#[test]
fn test_prints_and_returns_10() {
    let result = adder::prints_and_returns_10(5);
    assert_eq!(result, 10);
}
```

### 3.3 통합 테스트 서브모듈

```rust
// tests/common/mod.rs
pub fn setup() {
    // 테스트 환경 설정
    println!("테스트 환경 설정");
}

pub fn teardown() {
    // 테스트 환경 정리
    println!("테스트 환경 정리");
}

// tests/integration_test.rs
use adder;
mod common;

#[test]
fn test_with_setup() {
    common::setup();
    
    let result = adder::add_two(2);
    assert_eq!(result, 4);
    
    common::teardown();
}
```

## 📚 문서 테스트

### 4.1 기본 문서 테스트

```rust
/// 두 수를 더합니다.
/// 
/// # 예제
/// 
/// ```
/// let result = adder::add_two(2);
/// assert_eq!(result, 4);
/// ```
pub fn add_two(a: i32) -> i32 {
    a + 2
}

/// 인사말을 생성합니다.
/// 
/// # 예제
/// 
/// ```
/// use adder::greeting;
/// let result = greeting("World");
/// assert!(result.contains("World"));
/// ```
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}
```

### 4.2 문서 테스트 속성

```rust
/// # 예제
/// 
/// ```
/// # use adder::add_two;
/// # let input = 2;
/// let result = add_two(input);
/// assert_eq!(result, 4);
/// ```
/// 
/// # 패닉 예제
/// 
/// ```should_panic
/// let result = adder::add_two(999999999999999999);
/// ```
pub fn add_two(a: i32) -> i32 {
    a + 2
}
```

### 4.3 문서 테스트 실행

```bash
# 문서 테스트만 실행
cargo test --doc

# 특정 모듈의 문서 테스트
cargo test --doc adder

# 모든 테스트 (단위, 통합, 문서) 실행
cargo test --all
```

## 🎯 고급 테스트 기법

### 5.1 모의 객체 (Mock Objects)

```rust
pub trait NewsArticle {
    fn fetch_news(&self) -> Result<Vec<String>, String>;
}

pub struct NewsAggregator {
    source: Box<dyn NewsArticle>,
}

impl NewsAggregator {
    pub fn new(source: Box<dyn NewsArticle>) -> Self {
        NewsAggregator { source }
    }
    
    pub fn get_headlines(&self) -> Vec<String> {
        match self.source.fetch_news() {
            Ok(articles) => articles,
            Err(_) => vec!["뉴스를 가져올 수 없습니다".to_string()],
        }
    }
}

// 테스트용 Mock 구현
#[cfg(test)]
mod tests {
    use super::*;
    
    struct MockNewsSource {
        articles: Result<Vec<String>, String>,
    }
    
    impl MockNewsSource {
        fn new_success() -> Self {
            MockNewsSource {
                articles: Ok(vec![
                    "Rust 1.0 출시".to_string(),
                    "새로운 기능 발표".to_string(),
                ]),
            }
        }
        
        fn new_failure() -> Self {
            MockNewsSource {
                articles: Err("네트워크 오류".to_string()),
            }
        }
    }
    
    impl NewsArticle for MockNewsSource {
        fn fetch_news(&self) -> Result<Vec<String>, String> {
            self.articles.clone()
        }
    }
    
    #[test]
    fn test_successful_news_fetch() {
        let mock_source = MockNewsSource::new_success();
        let aggregator = NewsAggregator::new(Box::new(mock_source));
        
        let headlines = aggregator.get_headlines();
        assert_eq!(headlines.len(), 2);
        assert!(headlines[0].contains("Rust"));
    }
    
    #[test]
    fn test_failed_news_fetch() {
        let mock_source = MockNewsSource::new_failure();
        let aggregator = NewsAggregator::new(Box::new(mock_source));
        
        let headlines = aggregator.get_headlines();
        assert_eq!(headlines.len(), 1);
        assert!(headlines[0].contains("가능 없습니다"));
    }
}
```

### 5.2 테스트 더블과 스파이

```rust
use std::collections::HashMap;

pub trait Database {
    fn get(&self, key: &str) -> Option<String>;
    fn set(&mut self, key: &str, value: String);
}

pub struct UserService {
    db: Box<dyn Database>,
}

impl UserService {
    pub fn new(db: Box<dyn Database>) -> Self {
        UserService { db }
    }
    
    pub fn get_user_name(&self, user_id: &str) -> Option<String> {
        self.db.get(user_id)
    }
    
    pub fn update_user_name(&mut self, user_id: &str, name: String) {
        self.db.set(user_id, name);
    }
}

// 테스트용 스파이
#[cfg(test)]
mod tests {
    use super::*;
    
    struct SpyDatabase {
        data: HashMap<String, String>,
        get_calls: Vec<String>,
        set_calls: Vec<(String, String)>,
    }
    
    impl SpyDatabase {
        fn new() -> Self {
            SpyDatabase {
                data: HashMap::new(),
                get_calls: Vec::new(),
                set_calls: Vec::new(),
            }
        }
        
        fn get_call_count(&self, key: &str) -> usize {
            self.get_calls.iter().filter(|k| k == &key).count()
        }
        
        fn was_set(&self, key: &str, value: &str) -> bool {
            self.set_calls.iter()
                .any(|(k, v)| k == key && v == value)
        }
    }
    
    impl Database for SpyDatabase {
        fn get(&self, key: &str) -> Option<String> {
            // 호출 기록
            // Note: 실제 구현에서는 내부 가변성이 필요
            self.data.get(key).cloned()
        }
        
        fn set(&mut self, key: &str, value: String) {
            self.set_calls.push((key.to_string(), value.clone()));
            self.data.insert(key.to_string(), value);
        }
    }
    
    #[test]
    fn test_user_service_interaction() {
        let mut spy_db = SpyDatabase::new();
        spy_db.set("123".to_string(), "Alice".to_string());
        
        let mut service = UserService::new(Box::new(spy_db));
        
        // 사용자 이름 가져오기
        let name = service.get_user_name("123");
        assert_eq!(name, Some("Alice".to_string()));
        
        // 사용자 이름 업데이트
        service.update_user_name("123", "Bob".to_string());
        
        // 상호작용 검증
        // Note: 실제 구현에서는 더 정교한 스파이가 필요
    }
}
```

### 5.3 매개변수화 테스트

```rust
#[cfg(test)]
mod tests {
    // 테스트 케이스 구조체
    struct TestCase {
        input: i32,
        expected: i32,
        description: &'static str,
    }
    
    fn add_two(a: i32) -> i32 {
        a + 2
    }
    
    // 테스트 케이스 벡터
    fn test_cases() -> Vec<TestCase> {
        vec![
            TestCase {
                input: 0,
                expected: 2,
                description: "0 + 2 = 2",
            },
            TestCase {
                input: 1,
                expected: 3,
                description: "1 + 2 = 3",
            },
            TestCase {
                input: -1,
                expected: 1,
                description: "-1 + 2 = 1",
            },
        ]
    }
    
    #[test]
    fn test_add_two_parameterized() {
        for test_case in test_cases() {
            let result = add_two(test_case.input);
            assert_eq!(
                result, 
                test_case.expected,
                "{}: expected {}, got {}",
                test_case.description,
                test_case.expected,
                result
            );
        }
    }
}
```

## 🛠️ 테스트 유틸리티

### 6.1 테스트 환경 설정

```rust
// 테스트 전역 설정
use std::sync::Once;

static INIT: Once = Once::new();

fn setup() {
    INIT.call_once(|| {
        // 한 번만 실행되는 설정
        println!("테스트 환경 초기화");
        std::env::set_var("TEST_MODE", "true");
    });
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_one() {
        setup();
        // 테스트 로직
    }
    
    #[test]
    fn test_two() {
        setup();
        // 테스트 로직
    }
}
```

### 6.2 테스트 헬퍼 매크로

```rust
// 커스텀 테스트 매크로
macro_rules! assert_approx_eq {
    ($left:expr, $right:expr, $tolerance:expr) => {
        {
            let diff = ($left - $right).abs();
            assert!(
                diff <= $tolerance,
                "값이 너무 다릅니다: {} != {} (차이: {} > 허용오차: {})",
                $left, $right, diff, $tolerance
            );
        }
    };
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_approximate_equality() {
        assert_approx_eq!(3.14159, 3.14, 0.01);
        assert_approx_eq!(1.0, 1.0001, 0.001);
    }
}
```

### 6.3 벤치마크 테스트

```rust
// Cargo.toml에 추가
// [dev-dependencies]
// criterion = "0.3"

use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 1,
        1 => 1,
        n => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn bench_fibonacci(c: &mut Criterion) {
    c.bench_function("fib 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, bench_fibonacci);
criterion_main!(benches);

// 실행: cargo bench
```

## 📊 테스트 커버리지

### 7.1 커버리지 도구 설치

```bash
# Linux/macOS
cargo install cargo-tarpaulin

# 또는 grcov 사용
cargo install grcov
```

### 7.2 커버리지 실행

```bash
# tarpaulin 사용
cargo tarpaulin --out Html

# grcov 사용
export RUSTFLAGS="-C instrument-coverage"
export LLVM_PROFILE_FILE="target/coverage/%p-%m.profraw"

cargo test

grcov . --binary-path ./target/debug/ \
       --source-dir . \
       --output-type html \
       --branch \
       --ignore-not-existing \
       --ignore "/*" \
       --ignore "target/*" \
       --output-path ./target/coverage/
```

### 7.3 커버리지 기반 테스트

```rust
#[cfg(test)]
mod tests {
    // 커버리지를 고려한 테스트 케이스
    #[test]
    fn test_edge_cases() {
        // 경계값 테스트
        assert_eq!(add_two(i32::MAX - 2), i32::MAX);
        
        // 오버플로우 테스트
        // Note: 실제로는 panic이 발생할 수 있음
        // assert_eq!(add_two(i32::MAX - 1), i32::MAX);
    }
    
    #[test]
    fn test_error_conditions() {
        // 에러 조건 테스트
        let result = process_data("");
        assert!(result.is_err());
    }
}
```

## 🎯 테스트 모범 사례

### 8.1 테스트 명명 규칙

```rust
#[cfg(test)]
mod tests {
    // 좋은 테스트 이름: [테스트 대상]_[조건]_[기대 결과]
    
    #[test]
    fn add_two_with_positive_numbers_returns_sum() {
        // 양수 입력 시 합계 반환
    }
    
    #[test]
    fn add_two_with_zero_returns_original() {
        // 0 입력 시 원래 값 반환
    }
    
    #[test]
    fn add_two_with_negative_numbers_returns_correct_sum() {
        // 음수 입력 시 올바른 합계 반환
    }
    
    #[test]
    fn process_data_with_empty_input_returns_error() {
        // 빈 입력 시 에러 반환
    }
    
    #[test]
    fn process_data_with_valid_input_returns_success() {
        // 유효한 입력 시 성공 반환
    }
}
```

### 8.2 테스트 구성 원칙

```rust
#[cfg(test)]
mod tests {
    // AAA 패턴: Arrange, Act, Assert
    
    #[test]
    fn user_service_creates_user_successfully() {
        // Arrange: 테스트 데이터 준비
        let user_name = "Alice";
        let user_email = "alice@example.com";
        let mut service = UserService::new();
        
        // Act: 테스트 대상 실행
        let result = service.create_user(user_name, user_email);
        
        // Assert: 결과 검증
        assert!(result.is_ok());
        
        let user = result.unwrap();
        assert_eq!(user.name, user_name);
        assert_eq!(user.email, user_email);
        assert!(user.id > 0);
    }
}
```

### 8.3 테스트 데이터 관리

```rust
#[cfg(test)]
mod tests {
    // 테스트 데이터 빌더 패턴
    struct UserBuilder {
        name: String,
        email: String,
        age: u32,
    }
    
    impl UserBuilder {
        fn new() -> Self {
            UserBuilder {
                name: "Test User".to_string(),
                email: "test@example.com".to_string(),
                age: 25,
            }
        }
        
        fn with_name(mut self, name: &str) -> Self {
            self.name = name.to_string();
            self
        }
        
        fn with_email(mut self, email: &str) -> Self {
            self.email = email.to_string();
            self
        }
        
        fn with_age(mut self, age: u32) -> Self {
            self.age = age;
            self
        }
        
        fn build(self) -> User {
            User {
                name: self.name,
                email: self.email,
                age: self.age,
                id: 0,  // 테스트용 기본값
            }
        }
    }
    
    #[test]
    fn test_with_user_builder() {
        let user = UserBuilder::new()
            .with_name("Alice")
            .with_age(30)
            .build();
        
        assert_eq!(user.name, "Alice");
        assert_eq!(user.age, 30);
    }
}
```

## 📝 연습 문제

### 문제 1: 단위 테스트 작성
다음 함수에 대한 테스트를 작성하세요:

```rust
// TODO: 다음 함수에 대한 단위 테스트 작성
pub fn calculate_discount(price: f64, discount_rate: f64) -> f64 {
    if discount_rate < 0.0 || discount_rate > 1.0 {
        price  // 잘못된 할인율은 무시
    } else {
        price * (1.0 - discount_rate)
    }
}

pub fn is_prime(n: u32) -> bool {
    if n < 2 {
        return false;
    }
    
    for i in 2..=(n as f64).sqrt() as u32 {
        if n % i == 0 {
            return false;
        }
    }
    
    true
}

#[cfg(test)]
mod tests {
    use super::*;
    
    // TODO: calculate_discount 함수 테스트
    
    // TODO: is_prime 함수 테스트
}
```

### 문제 2: 통합 테스트 작성
다음 모듈에 대한 통합 테스트를 작성하세요:

```rust
// src/lib.rs
pub mod calculator;
pub mod validator;

// src/calculator.rs
pub struct Calculator {
    result: f64,
}

impl Calculator {
    pub fn new() -> Self {
        Calculator { result: 0.0 }
    }
    
    pub fn add(&mut self, value: f64) -> &mut Self {
        self.result += value;
        self
    }
    
    pub fn subtract(&mut self, value: f64) -> &mut Self {
        self.result -= value;
        self
    }
    
    pub fn get_result(&self) -> f64 {
        self.result
    }
}

// src/validator.rs
pub fn validate_number(value: &str) -> Result<f64, String> {
    value.parse::<f64>()
        .map_err(|_| "유효한 숫자가 아닙니다".to_string())
}

// TODO: tests/integration_tests.rs 파일 생성
```

### 문제 3: 모의 객체 테스트
다음 인터페이스에 대한 모의 객체 테스트를 작성하세요:

```rust
// TODO: PaymentProcessor 트레잇 정의
pub trait PaymentProcessor {
    fn process_payment(&self, amount: f64, card_number: &str) -> Result<String, String>;
}

// TODO: OrderService 구조체 정의
pub struct OrderService {
    payment_processor: Box<dyn PaymentProcessor>,
}

impl OrderService {
    pub fn new(payment_processor: Box<dyn PaymentProcessor>) -> Self {
        // TODO: 구현
    }
    
    pub fn process_order(&mut self, amount: f64, card_number: &str) -> Result<String, String> {
        // TODO: 구현
    }
}

// TODO: MockPaymentProcessor 구현 및 테스트 작성
```

---

**다음 단계**: [09_concurrency.md](./09_concurrency.md)에서 Rust의 동시성 프로그래밍을 학습하세요! 🦀
