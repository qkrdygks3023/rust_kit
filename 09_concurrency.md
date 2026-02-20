# 9. Rust 동시성 프로그래밍 완벽 가이드

## 🚀 동시성이란?

동시성(Concurrency)은 여러 작업이 동시에 진행되는 것처럼 보이게 하는 프로그래밍 패러다임입니다. Rust는 **안전한 동시성**을 컴파일 타임에 보장하여 데이터 경합과 같은 문제를 방지합니다.

Rust의 동시성 모델은 다음과 같은 특징을 가집니다:

- **소유권 시스템**: 컴파일 타임에 데이터 경합 방지
- **Fearless Concurrency**: 동시성 코드를 안전하게 작성
- **Zero-cost Abstractions**: 런타임 오버헤드 없는 추상화

## 🧵 스레드 (Threads)

### 1.1 기본 스레드 생성

```rust
use std::thread;
use std::time::Duration;

fn main() {
    // 메인 스레드
    let handle = thread::spawn(|| {
        println!("안녕하세요, 새로운 스레드에서!");
        
        // 시간 지연
        thread::sleep(Duration::from_millis(100));
        
        println!("스레드 작업 완료!");
    });
    
    println!("메인 스레드에서 메시지");
    
    // 스레드가 끝날 때까지 대기
    handle.join().unwrap();
    
    println!("메인 스레드 계속 실행");
}
```

### 1.2 스레드와 소유권

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    
    // move 클로저: 소유권 이동
    let handle = thread::spawn(move || {
        println!("벡터: {:?}", v);
        // v은 여기서 소유권을 가짐
    });
    
    // println!("{:?}", v);  // 오류! v의 소유권이 이동됨
    
    handle.join().unwrap();
}
```

### 1.3 여러 스레드 생성

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let mut handles = vec![];
    
    for i in 0..5 {
        let handle = thread::spawn(move || {
            println!("스레드 {} 시작", i);
            
            // 각 스레드에서 다른 작업 수행
            thread::sleep(Duration::from_millis(100 * (i + 1)));
            
            println!("스레드 {} 완료", i);
            
            i  // 스레드 ID 반환
        });
        
        handles.push(handle);
    }
    
    // 모든 스레드 결과 수집
    for handle in handles {
        let thread_id = handle.join().unwrap();
        println!("스레드 {} 결과 수집", thread_id);
    }
    
    println!("모든 스레드 완료");
}
```

### 1.4 스레드 이름 설정

```rust
use std::thread;

fn main() {
    let handle = thread::Builder::new()
        .name("worker_thread".to_string())
        .spawn(|| {
            println!("현재 스레드 이름: {:?}", thread::current().name());
            println!("스레드 ID: {:?}", thread::current().id());
        })
        .unwrap();
    
    handle.join().unwrap();
}
```

## 📡 채널 (Channels)

### 2.1 기본 채널 사용

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    // 채널 생성 (송신자, 수신자)
    let (tx, rx) = mpsc::channel();
    
    // 송신자를 새 스레드로 이동
    thread::spawn(move || {
        let val = String::from("안녕");
        tx.send(val).unwrap();
        println!("메시지 전송 완료");
        // val은 여기서 소유권을 잃음
    });
    
    // 메시지 수신
    let received = rx.recv().unwrap();
    println!("수신된 메시지: {}", received);
}
```

### 2.2 여러 값 전송

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        let vals = vec![
            String::from("첫 번째 메시지"),
            String::from("두 번째 메시지"),
            String::from("세 번째 메시지"),
        ];
        
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_millis(500));
        }
    });
    
    // 수신된 메시지 처리
    for received in rx {
        println!("수신: {}", received);
    }
}
```

### 2.3 다중 송신자

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    // 송신자 복제
    let tx1 = tx.clone();
    
    // 첫 번째 송신자 스레드
    thread::spawn(move || {
        let vals = vec![
            String::from("송신자 1: 메시지 1"),
            String::from("송신자 1: 메시지 2"),
        ];
        
        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_millis(200));
        }
    });
    
    // 두 번째 송신자 스레드
    thread::spawn(move || {
        let vals = vec![
            String::from("송신자 2: 메시지 A"),
            String::from("송신자 2: 메시지 B"),
        ];
        
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_millis(300));
        }
    });
    
    // 모든 메시지 수신
    for received in rx {
        println!("수신: {}", received);
    }
}
```

### 2.4 동기화 채널

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    // 동기화 채널 (rendezvous channel)
    let (tx, rx) = mpsc::sync_channel(0);
    
    thread::spawn(move || {
        println!("송신자: 메시지 보내는 중...");
        tx.send("안녕하세요!").unwrap();
        println!("송신자: 메시지 전송 완료");
    });
    
    thread::sleep(std::time::Duration::from_millis(1000));
    println!("수신자: 메시지 기다리는 중...");
    
    let msg = rx.recv().unwrap();
    println!("수신자: '{}' 수신 완료", msg);
}
```

## 🔒 공유 상태 동시성

### 3.1 Mutex 기본 사용

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];
    
    for _ in 0..10 {
        let handle = thread::spawn(|| {
            // 오류! counter를 여러 스레드에서 사용할 수 없음
            // let data = counter.lock().unwrap();
            // *data += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("결과: {}", *counter.lock().unwrap());
}
```

### 3.2 Arc와 Mutex 함께 사용

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut data = counter_clone.lock().unwrap();
            *data += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("결과: {}", *counter.lock().unwrap());
}
```

### 3.3 복잡한 공유 상태

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

#[derive(Debug)]
struct BankAccount {
    balance: f64,
}

impl BankAccount {
    fn new(initial_balance: f64) -> Self {
        BankAccount {
            balance: initial_balance,
        }
    }
    
    fn deposit(&mut self, amount: f64) {
        self.balance += amount;
    }
    
    fn withdraw(&mut self, amount: f64) -> bool {
        if self.balance >= amount {
            self.balance -= amount;
            true
        } else {
            false
        }
    }
    
    fn get_balance(&self) -> f64 {
        self.balance
    }
}

fn main() {
    let account = Arc::new(Mutex::new(BankAccount::new(1000.0)));
    let mut handles = vec![];
    
    // 입금 스레드들
    for i in 0..5 {
        let account_clone = Arc::clone(&account);
        let handle = thread::spawn(move || {
            for j in 0..10 {
                let mut acc = account_clone.lock().unwrap();
                acc.deposit(100.0);
                println!("입금 스레드 {}: 입금 100.0, 잔액: {:.2}", i, acc.get_balance());
                thread::sleep(Duration::from_millis(10));
            }
        });
        handles.push(handle);
    }
    
    // 출금 스레드들
    for i in 0..3 {
        let account_clone = Arc::clone(&account);
        let handle = thread::spawn(move || {
            for j in 0..10 {
                let mut acc = account_clone.lock().unwrap();
                let success = acc.withdraw(50.0);
                if success {
                    println!("출금 스레드 {}: 출금 50.0, 잔액: {:.2}", i, acc.get_balance());
                } else {
                    println!("출금 스레드 {}: 출금 실패, 잔액: {:.2}", i, acc.get_balance());
                }
                thread::sleep(Duration::from_millis(15));
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    let final_balance = account.lock().unwrap().get_balance();
    println!("최종 잔액: {:.2}", final_balance);
}
```

## 🔄 Async/Await

### 4.1 기본 Async/Await

```rust
// Cargo.toml에 추가
// [dependencies]
// tokio = { version = "1.0", features = ["full"] }

use tokio::time::{sleep, Duration};

async fn hello_world() {
    println!("Hello, world!");
}

async fn delayed_message() {
    sleep(Duration::from_secs(2)).await;
    println!("2초 후 메시지");
}

#[tokio::main]
async fn main() {
    hello_world().await;
    delayed_message().await;
}
```

### 4.2 여러 Async 작업

```rust
use tokio::time::{sleep, Duration};

async fn task_one() -> String {
    sleep(Duration::from_secs(1)).await;
    "작업 1 완료".to_string()
}

async fn task_two() -> String {
    sleep(Duration::from_secs(2)).await;
    "작업 2 완료".to_string()
}

async fn task_three() -> String {
    sleep(Duration::from_millis(500)).await;
    "작업 3 완료".to_string()
}

#[tokio::main]
async fn main() {
    // 순차 실행
    let result1 = task_one().await;
    let result2 = task_two().await;
    let result3 = task_three().await;
    
    println!("순차 실행:");
    println!("{}", result1);
    println!("{}", result2);
    println!("{}", result3);
    
    // 동시 실행
    let (result1, result2, result3) = tokio::join!(
        task_one(),
        task_two(),
        task_three()
    );
    
    println!("동시 실행:");
    println!("{}", result1);
    println!("{}", result2);
    println!("{}", result3);
}
```

### 4.3 Async 채널

```rust
use tokio::sync::mpsc;
use tokio::time::{sleep, Duration};

async fn producer(tx: mpsc::Sender<String>) {
    for i in 1..=5 {
        let message = format!("메시지 {}", i);
        tx.send(message).await.unwrap();
        println!("전송: 메시지 {}", i);
        sleep(Duration::from_millis(500)).await;
    }
}

async fn consumer(mut rx: mpsc::Receiver<String>) {
    while let Some(message) = rx.recv().await {
        println!("수신: {}", message);
        sleep(Duration::from_millis(300)).await;
    }
}

#[tokio::main]
async fn main() {
    let (tx, rx) = mpsc::channel(32);
    
    let producer_handle = tokio::spawn(producer(tx));
    let consumer_handle = tokio::spawn(consumer(rx));
    
    // 두 작업이 완료될 때까지 대기
    let _ = tokio::join!(producer_handle, consumer_handle);
    
    println!("모든 작업 완료");
}
```

## 🎯 실용적인 동시성 패턴

### 5.1 워커 풀 패턴

```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;

type Job = Box<dyn FnOnce() + Send + 'static>;

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            loop {
                let job = receiver.lock().unwrap().recv().unwrap();
                
                println!("워커 {} 작업 실행", id);
                job();
            }
        });
        
        Worker { id, thread }
    }
}

struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

impl ThreadPool {
    fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
        
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        
        let mut workers = Vec::with_capacity(size);
        
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
        
        ThreadPool { workers, sender }
    }
    
    fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.send(job).unwrap();
    }
}

fn main() {
    let pool = ThreadPool::new(4);
    
    for i in 0..8 {
        pool.execute(move || {
            println!("작업 {} 처리 중", i);
            thread::sleep(std::time::Duration::from_millis(1000));
            println!("작업 {} 완료", i);
        });
    }
    
    thread::sleep(std::time::Duration::from_secs(5));
}
```

### 5.2 생산자-소비자 패턴

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

struct Queue<T> {
    items: Arc<Mutex<Vec<T>>>,
}

impl<T> Queue<T> {
    fn new() -> Self {
        Queue {
            items: Arc::new(Mutex::new(Vec::new())),
        }
    }
    
    fn push(&self, item: T) {
        let mut items = self.items.lock().unwrap();
        items.push(item);
        println!("생산: 아이템 추가 (총 {}개)", items.len());
    }
    
    fn pop(&self) -> Option<T> {
        let mut items = self.items.lock().unwrap();
        if let Some(item) = items.pop() {
            println!("소비: 아이템 제거 (남은 {}개)", items.len());
            Some(item)
        } else {
            println!("소비: 큐가 비어있음");
            None
        }
    }
}

fn main() {
    let queue = Arc::new(Queue::new());
    let mut handles = vec![];
    
    // 생산자 스레드
    for i in 0..5 {
        let queue_clone = Arc::clone(&queue);
        let handle = thread::spawn(move || {
            for j in 0..3 {
                queue_clone.push(i * 10 + j);
                thread::sleep(Duration::from_millis(200));
            }
        });
        handles.push(handle);
    }
    
    // 소비자 스레드
    for i in 0..3 {
        let queue_clone = Arc::clone(&queue);
        let handle = thread::spawn(move || {
            for _ in 0..5 {
                if let Some(item) = queue_clone.pop() {
                    println!("소비자 {}: 아이템 {} 처리", i, item);
                    thread::sleep(Duration::from_millis(300));
                } else {
                    thread::sleep(Duration::from_millis(100));
                }
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

### 5.3 팬아웃/팬인 패턴

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn process_data(data: i32) -> i32 {
    thread::sleep(Duration::from_millis(100));
    data * 2
}

fn main() {
    // 팬아웃: 데이터를 여러 워커에게 분배
    let (data_sender, data_receiver) = mpsc::channel();
    let (result_sender, result_receiver) = mpsc::channel();
    
    // 워커 스레드들
    let mut handles = vec![];
    for worker_id in 0..3 {
        let data_receiver = data_receiver.clone();
        let result_sender = result_sender.clone();
        
        let handle = thread::spawn(move || {
            while let Ok(data) = data_receiver.recv() {
                let result = process_data(data);
                result_sender.send((worker_id, result)).unwrap();
            }
        });
        
        handles.push(handle);
    }
    
    // 데이터 전송 (팬아웃)
    for i in 1..=10 {
        data_sender.send(i).unwrap();
    }
    
    // 송신자 닫기
    drop(data_sender);
    drop(result_sender);
    
    // 결과 수집 (팬인)
    let mut results = vec![];
    while let Ok((worker_id, result)) = result_receiver.recv() {
        results.push((worker_id, result));
    }
    
    // 모든 워커 대기
    for handle in handles {
        handle.join().unwrap();
    }
    
    // 결과 정렬 및 출력
    results.sort_by_key(|&(_, result)| result);
    println!("처리된 결과:");
    for (worker_id, result) in results {
        println!("워커 {}: 결과 {}", worker_id, result);
    }
}
```

## 🛡️ 동시성 안전성

### 6.1 데이터 경합 방지

```rust
use std::sync::{Arc, Mutex};
use std::thread;

// 안전하지 않은 코드 (데이터 경합 가능)
fn unsafe_counter() {
    let mut counter = 0;
    let mut handles = vec![];
    
    for _ in 0..10 {
        let handle = thread::spawn(|| {
            // counter에 대한 동시 접근 - 데이터 경합!
            // counter += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    // println!("결과: {}", counter);
}

// 안전한 코드 (Mutex 사용)
fn safe_counter() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter_clone.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("안전한 결과: {}", *counter.lock().unwrap());
}

fn main() {
    safe_counter();
}
```

### 6.2 데드락 방지

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let mutex1 = Arc::new(Mutex::new(0));
    let mutex2 = Arc::new(Mutex::new(0));
    
    let mutex1_clone = Arc::clone(&mutex1);
    let mutex2_clone = Arc::clone(&mutex2);
    
    // 스레드 1
    let handle1 = thread::spawn(move || {
        let _lock1 = mutex1_clone.lock().unwrap();
        println!("스레드 1: mutex1 획득");
        
        thread::sleep(std::time::Duration::from_millis(100));
        
        let _lock2 = mutex2_clone.lock().unwrap();
        println!("스레드 1: mutex2 획득");
    });
    
    // 스레드 2
    let handle2 = thread::spawn(move || {
        let _lock2 = mutex2.lock().unwrap();
        println!("스레드 2: mutex2 획득");
        
        thread::sleep(std::time::Duration::from_millis(100));
        
        let _lock1 = mutex1.lock().unwrap();
        println!("스레드 2: mutex1 획득");
    });
    
    // 데드락 발생 가능성!
    handle1.join().unwrap();
    handle2.join().unwrap();
}
```

### 6.3 원자적 연산

```rust
use std::sync::atomic::{AtomicI32, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicI32::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                // 원자적 증가
                counter_clone.fetch_add(1, Ordering::SeqCst);
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("원자적 결과: {}", counter.load(Ordering::SeqCst));
}
```

## 📝 연습 문제

### 문제 1: 스레드 동기화
다음 요구사항을 만족하는 프로그램을 작성하세요:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

// TODO: SharedCounter 구조체 정의
// - counter: Arc<Mutex<i32>>

// TODO: SharedCounter 메서드 구현
// - new(): 초기화
// - increment(): 카운터 증가
// - get_value(): 현재 값 반환

fn main() {
    // TODO: 여러 스레드에서 카운터 증가 테스트
    // 10개 스레드, 각 스레드에서 1000번 증가
}
```

### 문제 2: 채널 통신
다음 요구사항을 만족하는 프로그램을 작성하세요:

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

// TODO: Task 구조체 정의
// - id: u32
// - data: String

// TODO: Worker 스레드 구현
// - 채널에서 Task 수신
// - 처리 시간 시뮬레이션
// - 결과 전송

fn main() {
    // TODO: 작업 분배 및 결과 수집 시스템 구현
    // 5개의 작업을 3개의 워커 스레드에 분배
}
```

### 문제 3: Async/Await
다음 요구사항을 만족하는 비동기 프로그램을 작성하세요:

```rust
// Cargo.toml에 tokio 의존성 추가 필요

// TODO: 비동기 함수들 구현
// - fetch_data(): 데이터 가져오기 (시간 지연)
// - process_data(): 데이터 처리
// - save_data(): 데이터 저장

// TODO: 여러 비동기 작업을 동시에 실행
// - 여러 데이터 소스에서 가져오기
// - 모든 작업 완료 대기

#[tokio::main]
async fn main() {
    // TODO: 비동기 파이프라인 테스트
}
```

---

**다음 단계**: [10_file_io.md](./10_file_io.md)에서 Rust의 파일 입출력을 학습하세요! 🦀
