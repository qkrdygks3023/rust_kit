# 7. Rust 컬렉션 타입 완벽 가이드

## 📦 컬렉션 개요

Rust의 표준 라이브러리는 다양한 컬렉션 타입을 제공합니다. 각 컬렉션은 특정 사용 사례에 최적화되어 있습니다:

- **Vector**: 가변 크기의 리스트
- **String**: UTF-8 인코딩된 텍스트
- **HashMap**: 키-값 쌍 저장

## 📋 Vector (Vec<T>)

### 1.1 Vector 기본 사용

```rust
fn main() {
    // Vector 생성
    let mut v: Vec<i32> = Vec::new();
    
    // 매크로로 Vector 생성
    let v2 = vec![1, 2, 3, 4, 5];
    
    // 요소 추가
    v.push(5);
    v.push(6);
    v.push(7);
    
    println!("v: {:?}", v);
    println!("v2: {:?}", v2);
    
    // 요소 접근
    let third: &i32 = &v2[2];
    println!("세 번째 요소: {}", third);
    
    // get 메서드로 안전한 접근
    let third_option: Option<&i32> = v2.get(2);
    match third_option {
        Some(third) => println!("세 번째 요소: {}", third),
        None => println!("세 번째 요소 없음"),
    }
    
    // 범위를 벗어난 접근
    // let does_not_exist = &v[100];  // panic!
    let does_not_exist = v.get(100);  // Option<i32> 반환
    println!("100번째 요소: {:?}", does_not_exist);
}
```

### 1.2 Vector 소유권과 대여

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    
    // 불변 대여
    let first = &v[0];
    println!("첫 번째 요소: {}", first);
    
    // 가변 대여 (불변 대여가 끝난 후 가능)
    v.push(6);
    
    // 반복문에서의 대여
    for i in &v {
        println!("요소: {}", i);
    }
    
    // 가변 반복
    for i in &mut v {
        *i *= 2;
    }
    
    println!("수정된 Vector: {:?}", v);
}
```

### 1.3 Vector와 소유권

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let mut points = vec![
        Point { x: 0, y: 0 },
        Point { x: 1, y: 1 },
        Point { x: 2, y: 2 },
    ];
    
    // 참조로 접근
    for point in &points {
        println!("점: ({}, {})", point.x, point.y);
    }
    
    // 가변 참조로 수정
    for point in &mut points {
        point.x += 10;
        point.y += 10;
    }
    
    println!("수정된 점들: {:?}", points);
    
    // 소유권 이동
    let first_point = points.remove(0);
    println!("제거된 점: {:?}", first_point);
    println!("남은 점들: {:?}", points);
}
```

### 1.4 Vector 유틸리티 메서드

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    
    // 길이와 용량
    println!("길이: {}", v.len());
    println!("용량: {}", v.capacity());
    
    // 용량 예약
    v.reserve(10);
    println!("예약 후 용량: {}", v.capacity());
    
    // 정렬
    v.sort();
    println!("정렬: {:?}", v);
    
    // 역순 정렬
    v.sort_by(|a, b| b.cmp(a));
    println!("역순 정렬: {:?}", v);
    
    // 필터링
    let even_numbers: Vec<i32> = v.iter().filter(|&&x| x % 2 == 0).cloned().collect();
    println!("짝수: {:?}", even_numbers);
    
    // 맵핑
    let doubled: Vec<i32> = v.iter().map(|x| x * 2).collect();
    println!("두 배: {:?}", doubled);
    
    // 제거
    v.retain(|&x| x > 2);
    println!("2보다 큰 수만: {:?}", v);
    
    // 검색
    let position = v.iter().position(|&x| x == 4);
    println!("4의 위치: {:?}", position);
    
    // 포함 여부 확인
    println!("3 포함: {}", v.contains(&3));
    println!("10 포함: {}", v.contains(&10));
}
```

## 🧵 String

### 2.1 String 생성

```rust
fn main() {
    // 빈 String 생성
    let mut s = String::new();
    
    // 문자열 리터럴로부터 생성
    let s1 = String::from("Hello");
    let s2 = "World".to_string();
    
    // 문자열 연결
    s.push_str("Hello");
    s.push(' ');
    s.push_str("World");
    
    println!("s: {}", s);
    println!("s1: {}", s1);
    println!("s2: {}", s2);
    
    // + 연산자로 연결 (소유권 이동 주의)
    let s3 = s1 + &s2;  // s1은 더 이상 유효하지 않음
    println!("s3: {}", s3);
    // println!("s1: {}", s1);  // 오류!
    
    // format! 매크로 (소유권 이동 없음)
    let s4 = format!("{}-{}-{}", s2, s3, "Rust");
    println!("s4: {}", s4);
}
```

### 2.2 String 슬라이싱

```rust
fn main() {
    let s = String::from("Hello, Rust!");
    
    // 슬라이스
    let hello = &s[0..5];
    let rust = &s[7..11];
    let all = &s[..];
    
    println!("Hello: {}", hello);
    println!("Rust: {}", rust);
    println!("All: {}", all);
    
    // 문자 단위 슬라이싱
    let hello_chars = s.chars().take(5).collect::<String>();
    println!("Hello (문자): {}", hello_chars);
    
    // 단어 분리
    let words: Vec<&str> = s.split(", ").collect();
    println!("단어들: {:?}", words);
    
    // 라인 분리
    let multiline = String::from("첫 번째 줄\n두 번째 줄\n세 번째 줄");
    let lines: Vec<&str> = multiline.lines().collect();
    println!("라인들: {:?}", lines);
}
```

### 2.3 String 수정

```rust
fn main() {
    let mut s = String::from("Hello World");
    
    // 대소문자 변환
    println!("대문자: {}", s.to_uppercase());
    println!("소문자: {}", s.to_lowercase());
    
    // 공백 제거
    let s_with_spaces = String::from("  Hello World  ");
    println!("앞뒤 공백 제거: '{}'", s_with_spaces.trim());
    println!("모든 공백 제거: '{}'", s_with_spaces.replace(" ", ""));
    
    // 문자열 교체
    let replaced = s.replace("World", "Rust");
    println!("교체: {}", replaced);
    
    // 문자열 반복
    let repeated = "Rust ".repeat(3);
    println!("반복: {}", repeated);
    
    // 문자 삽입
    s.insert(5, ',');
    println!("삽입: {}", s);
    
    // 문자열 범위 교체
    s.replace_range(7..12, "Rust");
    println!("범위 교체: {}", s);
    
    // 문자 삭제
    s.remove(5);  // 쉼표 삭제
    println!("삭제: {}", s);
    
    // 문자열 비우기
    s.clear();
    println!("비운 후 길이: {}", s.len());
}
```

### 2.4 String과 UTF-8

```rust
fn main() {
    let s = String::from("안녕하세요 Rust! 🦀");
    
    // 바이트 길이 vs 문자 길이
    println!("바이트 길이: {}", s.len());
    println!("문자 길이: {}", s.chars().count());
    
    // 문자 순회
    for (i, c) in s.chars().enumerate() {
        println!("{}번째 문자: {}", i, c);
    }
    
    // 바이트 순회
    println!("바이트들:");
    for (i, b) in s.bytes().enumerate() {
        print!("{} ", b);
    }
    println!();
    
    // UTF-8 클러스터 (글자 그래프)
    for grapheme in s.graphemes(true) {
        println!("글자 그래프: {}", grapheme);
    }
    
    // 특정 문자 검색
    if s.contains("Rust") {
        println!("'Rust' 포함");
    }
    
    if s.starts_with("안녕") {
        println!("'안녕'으로 시작");
    }
    
    if s.ends_with("🦀") {
        println!("'🦀'로 끝남");
    }
}
```

## 🗺️ HashMap

### 3.1 HashMap 기본 사용

```rust
use std::collections::HashMap;

fn main() {
    // HashMap 생성
    let mut scores = HashMap::new();
    
    // 키-값 쌍 추가
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    
    println!("점수: {:?}", scores);
    
    // 벡터에서 HashMap 생성
    let teams = vec![String::from("Blue"), String::from("Yellow")];
    let initial_scores = vec![10, 50];
    
    let scores2: HashMap<_, _> = teams.into_iter().zip(initial_scores.into_iter()).collect();
    println!("점수2: {:?}", scores2);
}
```

### 3.2 HashMap 소유권

```rust
use std::collections::HashMap;

fn main() {
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");
    
    let mut map = HashMap::new();
    map.insert(field_name, field_value);  // 소유권 이동
    
    // field_name과 field_value은 더 이상 유효하지 않음
    // println!("{}", field_name);  // 오류!
    
    // 값 접근
    let color = map.get("Favorite color");
    match color {
        Some(c) => println!("좋아하는 색: {}", c),
        None => println!("색상 정보 없음"),
    }
    
    // for 루프로 순회
    for (key, value) in &map {
        println!("{}: {}", key, value);
    }
}
```

### 3.3 HashMap 업데이트

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    
    // 기존 값에 기반한 업데이트
    scores.entry(String::from("Blue")).or_insert(25);
    println!("업데이트 후: {:?}", scores);
    
    // 기존 값 수정
    let text = String::from("Hello world hello world");
    let mut map = HashMap::new();
    
    for word in text.split_whitespace() {
        let count = map.entry(word.to_string()).or_insert(0);
        *count += 1;
    }
    
    println!("단어 빈도: {:?}", map);
    
    // 조건부 업데이트
    scores.entry(String::from("Red")).or_insert(30);
    println!("Red 추가: {:?}", scores);
    
    // 값 업데이트
    let old_score = scores.entry(String::from("Blue")).insert(20);
    println!("Blue의 이전 점수: {:?}", old_score);
    println!("업데이트된 점수: {:?}", scores);
}
```

### 3.4 HashMap 유틸리티

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    
    map.insert("A", 10);
    map.insert("B", 20);
    map.insert("C", 30);
    
    // 길이
    println!("맵 길이: {}", map.len());
    println!("맵 비어있음: {}", map.is_empty());
    
    // 키와 값 분리
    let keys: Vec<_> = map.keys().cloned().collect();
    let values: Vec<_> = map.values().cloned().collect();
    
    println!("키들: {:?}", keys);
    println!("값들: {:?}", values);
    
    // 키 존재 확인
    println!("A 키 존재: {}", map.contains_key("A"));
    println!("D 키 존재: {}", map.contains_key("D"));
    
    // 값 제거
    let removed = map.remove("B");
    println!("제거된 값: {:?}", removed);
    println!("제거 후 맵: {:?}", map);
    
    // 모든 값 제거
    map.clear();
    println!("클리어 후 길이: {}", map.len());
}
```

## 🔄 다른 컬렉션 타입들

### 4.1 VecDeque

```rust
use std::collections::VecDeque;

fn main() {
    let mut deque = VecDeque::new();
    
    // 뒤에 추가
    deque.push_back(1);
    deque.push_back(2);
    deque.push_back(3);
    
    // 앞에 추가
    deque.push_front(0);
    deque.push_front(-1);
    
    println!("Deque: {:?}", deque);
    
    // 앞에서 제거
    let front = deque.pop_front();
    println!("앞에서 제거: {:?}", front);
    
    // 뒤에서 제거
    let back = deque.pop_back();
    println!("뒤에서 제거: {:?}", back);
    
    println!("남은 Deque: {:?}", deque);
}
```

### 4.2 LinkedList

```rust
use std::collections::LinkedList;

fn main() {
    let mut list = LinkedList::new();
    
    // 앞에 추가
    list.push_front(1);
    list.push_front(2);
    
    // 뒤에 추가
    list.push_back(3);
    list.push_back(4);
    
    println!("LinkedList: {:?}", list);
    
    // 앞에서 제거
    let front = list.pop_front();
    println!("앞에서 제거: {:?}", front);
    
    // 뒤에서 제거
    let back = list.pop_back();
    println!("뒤에서 제거: {:?}", back);
    
    println!("남은 LinkedList: {:?}", list);
}
```

### 4.3 HashSet

```rust
use std::collections::HashSet;

fn main() {
    let mut set = HashSet::new();
    
    // 값 추가
    set.insert(1);
    set.insert(2);
    set.insert(3);
    set.insert(2);  // 중복은 무시됨
    
    println!("HashSet: {:?}", set);
    
    // 포함 확인
    println!("1 포함: {}", set.contains(&1));
    println!("4 포함: {}", set.contains(&4));
    
    // 제거
    set.remove(&2);
    println!("2 제거 후: {:?}", set);
    
    // 집합 연산
    let set1: HashSet<_> = [1, 2, 3, 4].iter().cloned().collect();
    let set2: HashSet<_> = [3, 4, 5, 6].iter().cloned().collect();
    
    // 교집합
    let intersection: HashSet<_> = set1.intersection(&set2).collect();
    println!("교집합: {:?}", intersection);
    
    // 합집합
    let union: HashSet<_> = set1.union(&set2).collect();
    println!("합집합: {:?}", union);
    
    // 차집합
    let difference: HashSet<_> = set1.difference(&set2).collect();
    println!("차집합: {:?}", difference);
}
```

### 4.4 BinaryHeap

```rust
use std::collections::BinaryHeap;

fn main() {
    let mut heap = BinaryHeap::new();
    
    // 값 추가
    heap.push(1);
    heap.push(5);
    heap.push(3);
    heap.push(2);
    heap.push(4);
    
    println!("Heap: {:?}", heap);
    
    // 최대값 확인 (제거하지 않음)
    println!("최대값: {:?}", heap.peek());
    
    // 최대값 제거
    let max = heap.pop();
    println!("제거된 최대값: {:?}", max);
    println!("남은 Heap: {:?}", heap);
    
    // 모든 값 제거 (정렬된 순서)
    let mut sorted = Vec::new();
    while let Some(value) = heap.pop() {
        sorted.push(value);
    }
    println!("정렬된 값: {:?}", sorted);
}
```

## 🎯 실용적인 컬렉션 패턴

### 5.1 데이터 그룹화

```rust
use std::collections::HashMap;

fn main() {
    let words = vec![
        "apple", "banana", "apple", "orange", "banana", 
        "grape", "apple", "orange", "grape", "grape"
    ];
    
    // 단어 빈도 계산
    let mut frequency = HashMap::new();
    for word in words {
        let count = frequency.entry(word.to_string()).or_insert(0);
        *count += 1;
    }
    
    println!("단어 빈도: {:?}", frequency);
    
    // 길이별 단어 그룹화
    let mut by_length: HashMap<usize, Vec<&str>> = HashMap::new();
    for word in ["apple", "banana", "cat", "dog", "elephant"].iter() {
        by_length.entry(word.len()).or_insert(Vec::new()).push(word);
    }
    
    println!("길이별 단어: {:?}", by_length);
}
```

### 5.2 데이터 필터링과 변환

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // 짝수 필터링
    let even_numbers: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)
        .cloned()
        .collect();
    
    println!("짝수: {:?}", even_numbers);
    
    // 제곱 계산
    let squares: Vec<i32> = numbers
        .iter()
        .map(|x| x * x)
        .collect();
    
    println!("제곱: {:?}", squares);
    
    // 체이닝: 짝수의 제곱
    let even_squares: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)
        .map(|x| x * x)
        .collect();
    
    println!("짝수의 제곱: {:?}", even_squares);
    
    // fold/reduce
    let sum: i32 = numbers.iter().sum();
    let product: i32 = numbers.iter().product();
    
    println!("합계: {}", sum);
    println!("곱: {}", product);
}
```

### 5.3 문자열 처리 패턴

```rust
fn main() {
    let text = "  Hello, World! Rust is awesome.  ";
    
    // 정규화 (공백 제거, 소문자 변환)
    let normalized = text.trim().to_lowercase();
    println!("정규화: '{}'", normalized);
    
    // 단어 추출
    let words: Vec<&str> = normalized.split_whitespace().collect();
    println!("단어들: {:?}", words);
    
    // 문장 분리
    let sentences: Vec<&str> = text.split('.').collect();
    println!("문장들: {:?}", sentences);
    
    // 문자 통계
    let char_count = normalized.chars().count();
    let word_count = words.len();
    let sentence_count = sentences.len() - 1;  // 마지막 빈 문자열 제외
    
    println!("문자 수: {}", char_count);
    println!("단어 수: {}", word_count);
    println!("문장 수: {}", sentence_count);
    
    // 패턴 매칭
    let rust_related = normalized.contains("rust");
    println!("Rust 관련: {}", rust_related);
}
```

## 📝 연습 문제

### 문제 1: Vector 연산
다음 기능들을 구현하세요:

```rust
// TODO: 평균 계산 함수
fn calculate_average(numbers: &[i32]) -> f64 {
    ___
}

// TODO: 중앙값 계산 함수
fn calculate_median(numbers: &mut Vec<i32>) -> f64 {
    ___
}

// TODO: 최빈값 계산 함수
fn calculate_mode(numbers: &[i32]) -> i32 {
    ___
}

fn main() {
    let mut numbers = vec![3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5];
    
    println!("평균: {}", calculate_average(&numbers));
    println!("중앙값: {}", calculate_median(&mut numbers));
    println!("최빈값: {}", calculate_mode(&numbers));
}
```

### 문제 2: String 처리
다음 기능들을 구현하세요:

```rust
// TODO: 단어 뒤집기 함수
fn reverse_words(text: &str) -> String {
    ___
}

// TODO: 팰린드롬 확인 함수
fn is_palindrome(text: &str) -> bool {
    ___
}

// TODO: 애너그램 확인 함수
fn are_anagrams(word1: &str, word2: &str) -> bool {
    ___
}

fn main() {
    let text = "Hello world";
    println!("뒤집힌 단어: {}", reverse_words(text));
    
    let palindrome = "A man a plan a canal Panama";
    println!("팰린드롬: {}", is_palindrome(palindrome));
    
    let word1 = "listen";
    let word2 = "silent";
    println!("애너그램: {}", are_anagrams(word1, word2));
}
```

### 문제 3: HashMap 응용
다음 기능들을 구현하세요:

```rust
use std::collections::HashMap;

// TODO: 학생 성적 관리 구조체
struct GradeManager {
    grades: HashMap<String, Vec<i32>>,
}

impl GradeManager {
    fn new() -> Self {
        ___
    }
    
    fn add_student(&mut self, name: String) {
        ___
    }
    
    fn add_grade(&mut self, name: &str, grade: i32) {
        ___
    }
    
    fn get_average(&self, name: &str) -> Option<f64> {
        ___
    }
    
    fn get_class_average(&self) -> f64 {
        ___
    }
}

fn main() {
    let mut manager = GradeManager::new();
    
    // TODO: 학생 추가 및 성적 입력
    // TODO: 평균 계산 테스트
}
```

---

**다음 단계**: [08_testing.md](./08_testing.md)에서 Rust의 테스트 시스템을 학습하세요! 🦀
