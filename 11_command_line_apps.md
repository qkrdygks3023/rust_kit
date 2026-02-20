# 11. Rust CLI 애플리케이션 완벽 가이드

## 💻 CLI 애플리케이션 개요

Rust는 CLI(명령줄 인터페이스) 애플리케이션 개발에 매우 적합한 언어입니다. Rust의 장점은 다음과 같습니다:

- **고성능**: C/C++ 수준의 실행 속도
- **메모리 안전**: 메모리 누수나 버퍼 오버플로우 없음
- **단일 바이너리**: 의존성 포함된 실행 파일
- **크로스 플랫폼**: 여러 운영체제에서 컴파일 가능

## 📦 기본 CLI 애플리케이션

### 1.1 기본 인자 처리

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    
    println!("인자 개수: {}", args.len());
    println!("프로그램 이름: {}", args[0]);
    
    if args.len() > 1 {
        println!("추가 인자:");
        for (i, arg) in args.iter().enumerate().skip(1) {
            println!("  {}: {}", i, arg);
        }
    }
}
```

### 1.2 간단한 계산기

```rust
use std::env;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();
    
    if args.len() != 4 {
        eprintln!("사용법: calculator <숫자1> <연산자> <숫자2>");
        eprintln!("연산자: +, -, *, /");
        process::exit(1);
    }
    
    let num1: f64 = match args[1].parse() {
        Ok(n) => n,
        Err(_) => {
            eprintln!("오류: 첫 번째 인자가 유효한 숫자가 아닙니다");
            process::exit(1);
        }
    };
    
    let operator = &args[2];
    let num2: f64 = match args[3].parse() {
        Ok(n) => n,
        Err(_) => {
            eprintln!("오류: 두 번째 인자가 유효한 숫자가 아닙니다");
            process::exit(1);
        }
    };
    
    let result = match operator.as_str() {
        "+" => num1 + num2,
        "-" => num1 - num2,
        "*" => num1 * num2,
        "/" => {
            if num2 == 0.0 {
                eprintln!("오류: 0으로 나눌 수 없습니다");
                process::exit(1);
            }
            num1 / num2
        }
        _ => {
            eprintln!("오류: 지원되지 않는 연산자 '{}'", operator);
            process::exit(1);
        }
    };
    
    println!("결과: {} {} {} = {}", num1, operator, num2, result);
}
```

## 🛠️ Clap 라이브러리

### 2.1 Clap 기본 사용

```toml
# Cargo.toml
[dependencies]
clap = { version = "4.0", features = ["derive"] }
```

```rust
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "my-cli")]
#[command(about = "A simple CLI tool")]
#[command(version = "1.0")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// Add a new item
    Add {
        /// The item to add
        name: String,
        /// The quantity (default: 1)
        #[arg(short, long, default_value_t = 1)]
        quantity: u32,
    },
    /// List all items
    List {
        /// Show detailed information
        #[arg(short, long)]
        verbose: bool,
    },
    /// Remove an item
    Remove {
        /// The item to remove
        name: String,
        /// Force removal without confirmation
        #[arg(short, long)]
        force: bool,
    },
}

fn main() {
    let cli = Cli::parse();
    
    match cli.command {
        Commands::Add { name, quantity } => {
            println!("추가: {} (수량: {})", name, quantity);
        }
        Commands::List { verbose } => {
            if verbose {
                println!("상세 목록:");
                println!("  - 아이템 1 (설명: 첫 번째 아이템)");
                println!("  - 아이템 2 (설명: 두 번째 아이템)");
            } else {
                println!("간단 목록:");
                println!("  - 아이템 1");
                println!("  - 아이템 2");
            }
        }
        Commands::Remove { name, force } => {
            if force {
                println!("강제 제거: {}", name);
            } else {
                println!("제거 확인: {} (y/n)", name);
                // 실제로는 사용자 입력 받기
            }
        }
    }
}
```

### 2.2 복잡한 CLI 구조

```rust
use clap::{Parser, Subcommand, ValueEnum};

#[derive(Parser)]
#[command(name = "file-manager")]
#[command(about = "A file management tool")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
    
    /// Verbose output
    #[arg(short, long, global = true)]
    verbose: bool,
    
    /// Configuration file
    #[arg(short, long, global = true)]
    config: Option<String>,
}

#[derive(Subcommand)]
enum Commands {
    /// File operations
    File {
        #[command(subcommand)]
        operation: FileOperations,
    },
    /// Directory operations
    Directory {
        #[command(subcommand)]
        operation: DirectoryOperations,
    },
    /// Search operations
    Search {
        /// Search pattern
        pattern: String,
        /// Search directory
        #[arg(short, long, default_value = ".")]
        directory: String,
        /// Search type
        #[arg(short, long, value_enum, default_value_t = SearchType::Name)]
        search_type: SearchType,
    },
}

#[derive(Subcommand)]
enum FileOperations {
    /// Copy a file
    Copy {
        /// Source file
        source: String,
        /// Destination file
        destination: String,
        /// Preserve permissions
        #[arg(short, long)]
        preserve: bool,
    },
    /// Move a file
    Move {
        /// Source file
        source: String,
        /// Destination file
        destination: String,
    },
    /// Delete a file
    Delete {
        /// File to delete
        file: String,
        /// Force deletion
        #[arg(short, long)]
        force: bool,
    },
}

#[derive(Subcommand)]
enum DirectoryOperations {
    /// Create a directory
    Create {
        /// Directory path
        path: String,
        /// Create parent directories
        #[arg(short, long)]
        parents: bool,
    },
    /// List directory contents
    List {
        /// Directory path
        #[arg(default_value = ".")]
        path: String,
        /// Show hidden files
        #[arg(short, long)]
        all: bool,
        /// Long format
        #[arg(short, long)]
        long: bool,
    },
    /// Remove a directory
    Remove {
        /// Directory path
        path: String,
        /// Remove recursively
        #[arg(short, long)]
        recursive: bool,
    },
}

#[derive(Clone, ValueEnum)]
enum SearchType {
    Name,
    Content,
    Extension,
}

fn main() {
    let cli = Cli::parse();
    
    if cli.verbose {
        println!("상세 모드 활성화");
    }
    
    if let Some(config) = cli.config {
        println!("설정 파일: {}", config);
    }
    
    match cli.command {
        Commands::File { operation } => {
            handle_file_operation(operation, cli.verbose);
        }
        Commands::Directory { operation } => {
            handle_directory_operation(operation, cli.verbose);
        }
        Commands::Search { pattern, directory, search_type } => {
            handle_search(pattern, directory, search_type, cli.verbose);
        }
    }
}

fn handle_file_operation(operation: FileOperations, verbose: bool) {
    match operation {
        FileOperations::Copy { source, destination, preserve } => {
            if verbose {
                println!("파일 복사: {} -> {}", source, destination);
                if preserve {
                    println!("권한 보존 활성화");
                }
            } else {
                println!("복사 완료");
            }
        }
        FileOperations::Move { source, destination } => {
            println!("파일 이동: {} -> {}", source, destination);
        }
        FileOperations::Delete { file, force } => {
            if force {
                println!("강제 삭제: {}", file);
            } else {
                println!("삭제: {}", file);
            }
        }
    }
}

fn handle_directory_operation(operation: DirectoryOperations, verbose: bool) {
    match operation {
        DirectoryOperations::Create { path, parents } => {
            if verbose {
                println!("디렉토리 생성: {} (부모 생성: {})", path, parents);
            } else {
                println!("디렉토리 생성 완료");
            }
        }
        DirectoryOperations::List { path, all, long } => {
            if verbose {
                println!("디렉토리 목록: {} (숨김 파일: {}, 상세: {})", path, all, long);
            }
        }
        DirectoryOperations::Remove { path, recursive } => {
            if recursive {
                println!("재귀적 디렉토리 삭제: {}", path);
            } else {
                println!("디렉토리 삭제: {}", path);
            }
        }
    }
}

fn handle_search(pattern: String, directory: String, search_type: SearchType, verbose: bool) {
    if verbose {
        println!("검색: 패턴='{}', 디렉토리='{}', 타입={:?}", pattern, directory, search_type);
    } else {
        println!("검색 완료");
    }
}
```

## 🎨 터미널 출력

### 3.1 색상 출력

```toml
# Cargo.toml
[dependencies]
colored = "2"
```

```rust
use colored::*;

fn main() {
    println!("{}일반 텍스트", "일반".clear());
    println!("{}빨간색 텍스트", "빨강".red());
    println!("{}초록색 텍스트", "초록".green());
    println!("{}파란색 텍스트", "파랑".blue());
    println!("{}노란색 텍스트", "노랑".yellow());
    println!("{}자홍색 텍스트", "자홍".magenta());
    println!("{}청록색 텍스트", "청록".cyan());
    println!("{}흰색 텍스트", "흰색".white());
    
    // 배경색
    println!("{}빨간 배경", "텍스트".on_red());
    println!("{}초록 배경", "텍스트".on_green());
    println!("{}파란 배경", "텍스트".on_blue());
    
    // 스타일
    println!("{}굵게", "굵게".bold());
    println!("{}밑줄", "밑줄".underline());
    println!("{}깜빡임", "깜빡임".blink());
    println!("{}반전", "반전".reversed());
    
    // 조합
    println!("{}조합 스타일", "텍스트".red().bold().underline());
}
```

### 3.2 진행률 표시

```toml
# Cargo.toml
[dependencies]
indicatif = "0.17"
```

```rust
use indicatif::{ProgressBar, ProgressStyle};
use std::thread;
use std::time::Duration;

fn main() {
    let total = 100;
    let pb = ProgressBar::new(total);
    
    pb.set_style(
        ProgressStyle::default_bar()
            .template("{spinner:.green} [{elapsed_precise}] [{bar:40.cyan/blue}] {pos}/{len} ({eta})")
            .progress_chars("#>-")
    );
    
    for i in 0..total {
        thread::sleep(Duration::from_millis(25));
        pb.inc(1);
        
        if i % 10 == 0 {
            pb.set_message(format!("처리 중: {}%", i));
        }
    }
    
    pb.finish_with_message("처리 완료!");
    
    // 다중 진행률 표시
    let pb1 = ProgressBar::new(50);
    let pb2 = ProgressBar::new(30);
    
    pb1.set_style(ProgressStyle::default_bar().template("{spinner:.green} [{bar:40}] {pos}/{len}"));
    pb2.set_style(ProgressStyle::default_bar().template("{spinner:.blue} [{bar:40}] {pos}/{len}"));
    
    for i in 0..50 {
        thread::sleep(Duration::from_millis(50));
        pb1.inc(1);
        
        if i < 30 {
            pb2.inc(1);
        }
    }
    
    pb1.finish();
    pb2.finish();
}
```

### 3.3 대화형 입력

```toml
# Cargo.toml
[dependencies]
dialoguer = "0.10"
```

```rust
use dialoguer::{Confirm, Input, Select, theme::ColorfulTheme};
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let theme = ColorfulTheme::default();
    
    // 확인 질문
    let confirm = Confirm::with_theme(&theme)
        .with_prompt("계속하시겠습니까?")
        .default(true)
        .interact()?;
    
    if confirm {
        println!("사용자가 '예'를 선택했습니다");
    } else {
        println!("사용자가 '아니오'를 선택했습니다");
        return Ok(());
    }
    
    // 텍스트 입력
    let name: String = Input::with_theme(&theme)
        .with_prompt("이름을 입력하세요")
        .default("게스트".to_string())
        .interact_text()?;
    
    println!("안녕하세요, {}!", name);
    
    // 비밀번호 입력
    let password: String = Input::with_theme(&theme)
        .with_prompt("비밀번호를 입력하세요")
        .password()
        .interact()?;
    
    println!("비밀번호가 입력되었습니다 (길이: {})", password.len());
    
    // 선택 메뉴
    let selections = vec![
        "옵션 1: 기본 설정",
        "옵션 2: 고급 설정",
        "옵션 3: 종료",
    ];
    
    let selection = Select::with_theme(&theme)
        .with_prompt("옵션을 선택하세요")
        .items(&selections)
        .interact()?;
    
    println!("선택된 옵션: {}", selections[selection]);
    
    Ok(())
}
```

## 📊 테이블 출력

```toml
# Cargo.toml
[dependencies]
tabled = "0.12"
```

```rust
use tabled::{Table, Tabled, settings::Style};
use serde::{Deserialize, Serialize};

#[derive(Tabled, Serialize, Deserialize)]
struct User {
    #[tabled(rename = "ID")]
    id: u32,
    #[tabled(rename = "이름")]
    name: String,
    #[tabled(rename = "이메일")]
    email: String,
    #[tabled(rename = "나이")]
    age: u32,
}

fn main() {
    let users = vec![
        User {
            id: 1,
            name: "김철수".to_string(),
            email: "kim@example.com".to_string(),
            age: 30,
        },
        User {
            id: 2,
            name: "이영희".to_string(),
            email: "lee@example.com".to_string(),
            age: 25,
        },
        User {
            id: 3,
            name: "박민준".to_string(),
            email: "park@example.com".to_string(),
            age: 35,
        },
    ];
    
    // 기본 테이블
    let table = Table::new(&users)
        .with(Style::modern())
        .to_string();
    
    println!("사용자 목록:");
    println!("{}", table);
    
    // 사용자 정의 스타일
    let custom_table = Table::new(&users)
        .with(Style::ascii())
        .to_string();
    
    println!("\nASCII 스타일:");
    println!("{}", custom_table);
}
```

## 🚀 실용적인 CLI 예제

### 4.1 파일 검색 도구

```rust
use clap::Parser;
use std::fs;
use std::io;
use std::path::Path;
use colored::*;

#[derive(Parser)]
#[command(name = "find")]
#[command(about = "A file search tool")]
struct Cli {
    /// Search pattern
    pattern: String,
    
    /// Search directory
    #[arg(short, long, default_value = ".")]
    directory: String,
    
    /// Case insensitive search
    #[arg(short, long)]
    ignore_case: bool,
    
    /// Search in file contents
    #[arg(short, long)]
    content: bool,
    
    /// Show line numbers
    #[arg(short, long)]
    line_numbers: bool,
}

fn find_files_by_name(dir: &Path, pattern: &str, ignore_case: bool) -> io::Result<Vec<String>> {
    let mut found_files = Vec::new();
    
    for entry in fs::read_dir(dir)? {
        let entry = entry?;
        let path = entry.path();
        
        if path.is_dir() {
            let mut sub_files = find_files_by_name(&path, pattern, ignore_case)?;
            found_files.append(&mut sub_files);
        } else if let Some(filename) = path.file_name() {
            let filename_str = filename.to_string_lossy();
            
            let matches = if ignore_case {
                filename_str.to_lowercase().contains(&pattern.to_lowercase())
            } else {
                filename_str.contains(pattern)
            };
            
            if matches {
                if let Some(path_str) = path.to_str() {
                    found_files.push(path_str.to_string());
                }
            }
        }
    }
    
    Ok(found_files)
}

fn search_in_file(file_path: &Path, pattern: &str, ignore_case: bool, show_line_numbers: bool) -> io::Result<Vec<(usize, String)>> {
    let content = fs::read_to_string(file_path)?;
    let mut matches = Vec::new();
    
    for (line_num, line) in content.lines().enumerate() {
        let search_line = if ignore_case {
            line.to_lowercase()
        } else {
            line.to_string()
        };
        
        let search_pattern = if ignore_case {
            pattern.to_lowercase()
        } else {
            pattern.to_string()
        };
        
        if search_line.contains(&search_pattern) {
            matches.push((line_num + 1, line.to_string()));
        }
    }
    
    Ok(matches)
}

fn main() -> io::Result<()> {
    let cli = Cli::parse();
    
    let search_dir = Path::new(&cli.directory);
    
    if cli.content {
        // 파일 내용 검색
        for entry in fs::read_dir(search_dir)? {
            let entry = entry?;
            let path = entry.path();
            
            if path.is_file() {
                if let Ok(matches) = search_in_file(&path, &cli.pattern, cli.ignore_case, cli.line_numbers) {
                    if !matches.is_empty() {
                        println!("{}", path.display().to_string().green());
                        for (line_num, line) in matches {
                            if cli.line_numbers {
                                println!("  {}: {}", line_num.to_string().blue(), line);
                            } else {
                                println!("  {}", line);
                            }
                        }
                        println!();
                    }
                }
            }
        }
    } else {
        // 파일 이름 검색
        let found_files = find_files_by_name(search_dir, &cli.pattern, cli.ignore_case)?;
        
        if found_files.is_empty() {
            println!("{}: '{}' 패턴으로 파일을 찾을 수 없습니다", "오류".red(), cli.pattern);
        } else {
            println!("{}개 파일을 찾았습니다:", found_files.len());
            for file in found_files {
                println!("{}", file.green());
            }
        }
    }
    
    Ok(())
}
```

### 4.2 로그 분석기

```rust
use clap::Parser;
use std::collections::HashMap;
use std::fs::File;
use std::io::{self, BufRead};
use std::path::Path;
use tabled::{Table, Tabled, settings::Style};

#[derive(Parser)]
#[command(name = "loganalyzer")]
#[command(about = "A log file analyzer")]
struct Cli {
    /// Log file path
    file: String,
    
    /// Show statistics
    #[arg(short, long)]
    stats: bool,
    
    /// Filter by log level
    #[arg(short, long)]
    level: Option<String>,
    
    /// Search for specific pattern
    #[arg(short, long)]
    search: Option<String>,
    
    /// Show last N lines
    #[arg(short, long, default_value_t = 10)]
    tail: usize,
}

#[derive(Tabled)]
struct LogEntry {
    #[tabled(rename = "시간")]
    timestamp: String,
    #[tabled(rename = "레벨")]
    level: String,
    #[tabled(rename = "메시지")]
    message: String,
}

#[derive(Tabled)]
struct LogStats {
    #[tabled(rename = "레벨")]
    level: String,
    #[tabled(rename = "개수")]
    count: usize,
    #[tabled(rename = "비율")]
    percentage: f64,
}

fn parse_log_line(line: &str) -> Option<LogEntry> {
    // 간단한 로그 포맷 파싱: [TIMESTAMP] [LEVEL] MESSAGE
    if let (Some(start), Some(level_end)) = (line.find('['), line.find(']')) {
        if let (Some(level_start), Some(msg_start)) = (line.find('[', start + 1), line.find(']', level_start + 1)) {
            let timestamp = line[start + 1..level_end].to_string();
            let level = line[level_start + 1..msg_start].to_string();
            let message = line[msg_start + 1..].trim().to_string();
            
            return Some(LogEntry {
                timestamp,
                level,
                message,
            });
        }
    }
    None
}

fn analyze_log(file_path: &str, cli: &Cli) -> io::Result<()> {
    let file = File::open(file_path)?;
    let reader = io::BufReader::new(file);
    
    let mut entries = Vec::new();
    let mut level_counts = HashMap::new();
    let mut total_count = 0;
    
    for line in reader.lines() {
        let line = line?;
        total_count += 1;
        
        if let Some(entry) = parse_log_line(&line) {
            // 레벨 필터링
            if let Some(ref filter_level) = cli.level {
                if entry.level.to_uppercase() != filter_level.to_uppercase() {
                    continue;
                }
            }
            
            // 검색 필터링
            if let Some(ref search_pattern) = cli.search {
                if !entry.message.to_lowercase().contains(&search_pattern.to_lowercase()) {
                    continue;
                }
            }
            
            // 레벨 통계
            *level_counts.entry(entry.level.clone()).or_insert(0) += 1;
            
            entries.push(entry);
        }
    }
    
    // 마지막 N개만 표시
    if entries.len() > cli.tail {
        entries = entries.into_iter().skip(entries.len() - cli.tail).collect();
    }
    
    // 로그 엔트리 표시
    if !entries.is_empty() {
        let table = Table::new(&entries)
            .with(Style::modern())
            .to_string();
        println!("로그 엔트리 (마지막 {}개):", entries.len());
        println!("{}", table);
    }
    
    // 통계 정보 표시
    if cli.stats {
        let mut stats = Vec::new();
        for (level, count) in level_counts {
            let percentage = (count as f64 / total_count as f64) * 100.0;
            stats.push(LogStats {
                level,
                count,
                percentage,
            });
        }
        
        stats.sort_by(|a, b| b.count.cmp(&a.count));
        
        let table = Table::new(&stats)
            .with(Style::modern())
            .to_string();
        println!("\n로그 통계 (총 {}개):", total_count);
        println!("{}", table);
    }
    
    Ok(())
}

fn main() -> io::Result<()> {
    let cli = Cli::parse();
    
    if !Path::new(&cli.file).exists() {
        eprintln!("오류: 파일 '{}'을 찾을 수 없습니다", cli.file);
        std::process::exit(1);
    }
    
    analyze_log(&cli.file, &cli)?;
    
    Ok(())
}
```

### 4.3 시스템 정보 도구

```rust
use clap::{Parser, Subcommand};
use std::process::Command;
use tabled::{Table, Tabled, settings::Style};
use sysinfo::{System, SystemExt, ProcessExt, CpuExt};
use colored::*;

#[derive(Parser)]
#[command(name = "sysinfo")]
#[command(about = "System information tool")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// Show system information
    System,
    /// Show CPU information
    Cpu,
    /// Show memory information
    Memory,
    /// Show process information
    Process {
        /// Show top N processes
        #[arg(short, long, default_value_t = 10)]
        top: usize,
    },
    /// Show disk information
    Disk,
}

#[derive(Tabled)]
struct ProcessInfo {
    #[tabled(rename = "PID")]
    pid: u32,
    #[tabled(rename = "이름")]
    name: String,
    #[tabled(rename = "CPU 사용률")]
    cpu_usage: f32,
    #[tabled(rename = "메모리 사용률")]
    memory_usage: f64,
}

#[derive(Tabled)]
struct CpuInfo {
    #[tabled(rename = "코어")]
    core: usize,
    #[tabled(rename = "사용률")]
    usage: f32,
    #[tabled(rename = "주파수")]
    frequency: u64,
}

fn get_system_info() -> System {
    let mut sys = System::new_all();
    sys.refresh_all();
    sys
}

fn show_system_info(sys: &System) {
    println!("{}\n", "시스템 정보".bold().cyan());
    println!("운영체제: {}", sys.name().unwrap_or("알 수 없음"));
    println!("커널 버전: {}", sys.kernel_version().unwrap_or("알 수 없음"));
    println!("호스트 이름: {}", sys.host_name().unwrap_or("알 수 없음"));
    println!("부팅 시간: {}초", sys.uptime());
    println!("총 프로세스: {}", sys.processes().len());
}

fn show_cpu_info(sys: &System) {
    println!("{}\n", "CPU 정보".bold().cyan());
    
    let mut cpu_info = Vec::new();
    for (i, cpu) in sys.cpus().iter().enumerate() {
        cpu_info.push(CpuInfo {
            core: i,
            usage: cpu.cpu_usage(),
            frequency: cpu.frequency(),
        });
    }
    
    let table = Table::new(&cpu_info)
        .with(Style::modern())
        .to_string();
    println!("{}", table);
    
    let total_usage = sys.global_cpu_info().cpu_usage();
    println!("전체 CPU 사용률: {:.1}%", total_usage);
}

fn show_memory_info(sys: &System) {
    println!("{}\n", "메모리 정보".bold().cyan());
    
    let total_memory = sys.total_memory();
    let used_memory = sys.used_memory();
    let free_memory = sys.free_memory();
    
    let total_swap = sys.total_swap();
    let used_swap = sys.used_swap();
    let free_swap = sys.free_swap();
    
    println!("메모리:");
    println!("  전체: {} MB", total_memory / 1024);
    println!("  사용: {} MB ({:.1}%)", used_memory / 1024, 
             (used_memory as f64 / total_memory as f64) * 100.0);
    println!("  여유: {} MB", free_memory / 1024);
    
    if total_swap > 0 {
        println!("스왑:");
        println!("  전체: {} MB", total_swap / 1024);
        println!("  사용: {} MB ({:.1}%)", used_swap / 1024,
                 (used_swap as f64 / total_swap as f64) * 100.0);
        println!("  여유: {} MB", free_swap / 1024);
    }
}

fn show_process_info(sys: &System, top: usize) {
    println!("{}\n", "프로세스 정보".bold().cyan());
    
    let mut processes: Vec<_> = sys.processes()
        .values()
        .collect();
    
    // CPU 사용률로 정렬
    processes.sort_by(|a, b| b.cpu_usage().partial_cmp(&a.cpu_usage()).unwrap());
    
    let mut process_info = Vec::new();
    for process in processes.iter().take(top) {
        process_info.push(ProcessInfo {
            pid: process.pid(),
            name: process.name().to_string(),
            cpu_usage: process.cpu_usage(),
            memory_usage: (process.memory() as f64 / sys.total_memory() as f64) * 100.0,
        });
    }
    
    let table = Table::new(&process_info)
        .with(Style::modern())
        .to_string();
    println!("상위 {}개 프로세스:", top);
    println!("{}", table);
}

fn show_disk_info() {
    println!("{}\n", "디스크 정보".bold().cyan());
    
    // Linux/Unix 시스템에서 df 명령어 사용
    if cfg!(unix) {
        if let Ok(output) = Command::new("df").arg("-h").output() {
            let output_str = String::from_utf8_lossy(&output.stdout);
            println!("{}", output_str);
        }
    } else if cfg!(windows) {
        // Windows에서는 다른 방식으로 디스크 정보 가져오기
        println!("Windows 디스크 정보는 추가 구현이 필요합니다");
    }
}

fn main() {
    let cli = Cli::parse();
    
    let sys = get_system_info();
    
    match cli.command {
        Commands::System => {
            show_system_info(&sys);
        }
        Commands::Cpu => {
            show_cpu_info(&sys);
        }
        Commands::Memory => {
            show_memory_info(&sys);
        }
        Commands::Process { top } => {
            show_process_info(&sys, top);
        }
        Commands::Disk => {
            show_disk_info();
        }
    }
}
```

## 📝 연습 문제

### 문제 1: TODO 관리 CLI
다음 기능들을 가진 TODO 관리 CLI를 구현하세요:

```rust
use clap::{Parser, Subcommand};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::fs;

// TODO: TodoItem 구조체 정의
// - id: u32
// - title: String
// - completed: bool
// - created_at: String

// TODO: TodoManager 구조체 정의
// - todos: HashMap<u32, TodoItem>
// - next_id: u32

// TODO: CLI 명령어 정의
// - add: 새 TODO 추가
// - list: TODO 목록 표시
// - complete: TODO 완료
// - delete: TODO 삭제

// TODO: 파일 저장/로드 기능 구현

#[derive(Parser)]
#[command(name = "todo")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    Add {
        title: String,
    },
    List {
        #[arg(short, long)]
        all: bool,
    },
    Complete {
        id: u32,
    },
    Delete {
        id: u32,
    },
}

fn main() {
    // TODO: TODO 관리기능 구현
}
```

### 문제 2: 파일 비교 도구
다음 기능들을 가진 파일 비교 CLI를 구현하세요:

```rust
use clap::Parser;
use std::fs;
use std::io;

// TODO: 파일 비교 기능 구현
// - 내용 비교
// - 라인별 비교
// - 바이너리 비교
// - 차이점 보고서 생성

#[derive(Parser)]
#[command(name = "diff")]
struct Cli {
    file1: String,
    file2: String,
    
    #[arg(short, long)]
    ignore_whitespace: bool,
    
    #[arg(short, long)]
    ignore_case: bool,
    
    #[arg(short, long)]
    show_context: bool,
}

fn main() -> io::Result<()> {
    // TODO: 파일 비교 구현
}
```

### 문제 3: 시스템 모니터
다음 기능들을 가진 시스템 모니터 CLI를 구현하세요:

```rust
use clap::{Parser, Subcommand};
use std::thread;
use std::time::Duration;

// TODO: 실시간 시스템 모니터링 기능 구현
// - CPU 사용률 모니터링
// - 메모리 사용량 모니터링
// - 프로세스 모니터링
// - 알림 기능

#[derive(Parser)]
#[command(name = "monitor")]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    Cpu {
        #[arg(short, long, default_value_t = 1)]
        interval: u64,
    },
    Memory {
        #[arg(short, long, default_value_t = 1)]
        interval: u64,
    },
    Process {
        #[arg(short, long)]
        name: Option<String>,
        #[arg(short, long, default_value_t = 1)]
        interval: u64,
    },
}

fn main() {
    // TODO: 시스템 모니터링 구현
}
```

---

**다음 단계**: [12_web_development.md](./12_web_development.md)에서 Rust 웹 개발을 학습하세요! 🦀
