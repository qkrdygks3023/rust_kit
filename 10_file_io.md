# 10. Rust 파일 입출력 완벽 가이드

## 📁 파일 I/O 개요

Rust는 파일 입출력을 위한 강력하고 안전한 API를 제공합니다. Rust의 파일 I/O 시스템은 다음과 같은 특징을 가집니다:

- **타입 안전성**: 컴파일 타임에 에러 검출
- **에러 처리**: Result 타입을 통한 명시적 에러 처리
- **메모리 안전**: 버퍼 오버플로우 방지
- **성능**: 제로-비용 추상화

## 📖 파일 읽기

### 1.1 기본 파일 읽기

```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    // 파일 내용을 문자열로 읽기
    let content = fs::read_to_string("hello.txt")?;
    println!("파일 내용:\n{}", content);
    
    // 파일을 바이트 벡터로 읽기
    let bytes = fs::read("hello.txt")?;
    println!("바이트 수: {}", bytes.len());
    
    Ok(())
}
```

### 1.2 File 구조체 사용

```rust
use std::fs::File;
use std::io::{self, Read};

fn main() -> io::Result<()> {
    // 파일 열기
    let mut file = File::open("hello.txt")?;
    
    // 버퍼 생성
    let mut buffer = [0; 1024];
    
    // 파일 내용 읽기
    let bytes_read = file.read(&mut buffer)?;
    println!("읽은 바이트 수: {}", bytes_read);
    
    // 읽은 내용을 문자열로 변환
    let content = String::from_utf8_lossy(&buffer[..bytes_read]);
    println!("파일 내용:\n{}", content);
    
    Ok(())
}
```

### 1.3 BufReader 사용

```rust
use std::fs::File;
use std::io::{self, BufReader, BufRead};

fn main() -> io::Result<()> {
    let file = File::open("hello.txt")?;
    let reader = BufReader::new(file);
    
    // 한 줄씩 읽기
    for line in reader.lines() {
        let line = line?;
        println!("라인: {}", line);
    }
    
    Ok(())
}
```

### 1.4 파일 읽기 옵션

```rust
use std::fs::File;
use std::io::{self, Read, Seek, SeekFrom};

fn main() -> io::Result<()> {
    let mut file = File::open("hello.txt")?;
    
    // 파일 크기 확인
    let file_size = file.metadata()?.len();
    println!("파일 크기: {} 바이트", file_size);
    
    // 특정 위치로 이동
    file.seek(SeekFrom::Start(10))?;
    
    // 특정 크기만 읽기
    let mut buffer = vec![0u8; 100];
    let bytes_read = file.read(&mut buffer)?;
    println!("읽은 바이트: {}", bytes_read);
    
    // 파일 끝으로 이동
    file.seek(SeekFrom::End(0))?;
    
    // 마지막 10바이트 읽기
    file.seek(SeekFrom::End(-10))?;
    let mut last_bytes = [0u8; 10];
    let bytes_read = file.read(&mut last_bytes)?;
    
    let content = String::from_utf8_lossy(&last_bytes[..bytes_read]);
    println!("마지막 10바이트: {}", content);
    
    Ok(())
}
```

## ✏️ 파일 쓰기

### 2.1 기본 파일 쓰기

```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    // 문자열로 파일 쓰기
    fs::write("output.txt", "Hello, Rust!\nThis is a test file.")?;
    println!("파일 쓰기 완료");
    
    // 바이트 벡터로 파일 쓰기
    let bytes = b"Binary data";
    fs::write("binary.dat", bytes)?;
    println!("바이너리 파일 쓰기 완료");
    
    Ok(())
}
```

### 2.2 File 구조체로 쓰기

```rust
use std::fs::File;
use std::io::{self, Write};

fn main() -> io::Result<()> {
    // 파일 생성 (기존 파일이 있으면 덮어쓰기)
    let mut file = File::create("output.txt")?;
    
    // 문자열 쓰기
    file.write_all(b"Hello, World!\n")?;
    file.write_all("This is Rust file I/O.\n".as_bytes())?;
    
    // 포맷된 쓰기
    use std::fmt::Write;
    writeln!(&mut file, "Number: {}", 42)?;
    writeln!(&mut file, "Boolean: {}", true)?;
    
    println!("파일 쓰기 완료");
    
    Ok(())
}
```

### 2.3 BufWriter 사용

```rust
use std::fs::File;
use std::io::{self, BufWriter, Write};

fn main() -> io::Result<()> {
    let file = File::create("large_output.txt")?;
    let mut writer = BufWriter::new(file);
    
    // 많은 양의 데이터 쓰기
    for i in 0..10000 {
        writeln!(writer, "Line {}: This is a test line", i)?;
    }
    
    // 버퍼 플러시 (필요한 경우)
    writer.flush()?;
    
    println!("대용량 파일 쓰기 완료");
    
    Ok(())
}
```

### 2.4 파일 추가 모드

```rust
use std::fs::OpenOptions;
use std::io::{self, Write};

fn main() -> io::Result<()> {
    // 파일을 추가 모드로 열기
    let mut file = OpenOptions::new()
        .append(true)
        .create(true)
        .open("log.txt")?;
    
    // 로그 메시지 추가
    writeln!(file, "[{}] 새로운 로그 메시지", chrono::Utc::now())?;
    
    println!("로그 추가 완료");
    
    Ok(())
}
```

## 📁 디렉토리 작업

### 3.1 디렉토리 생성

```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    // 단일 디렉토리 생성
    fs::create_dir("new_directory")?;
    println!("디렉토리 생성 완료");
    
    // 중첩 디렉토리 생성
    fs::create_dir_all("path/to/nested/directory")?;
    println!("중첩 디렉토리 생성 완료");
    
    Ok(())
}
```

### 3.2 디렉토리 읽기

```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    // 디렉토리 내용 읽기
    let entries = fs::read_dir(".")?;
    
    println!("현재 디렉토리 내용:");
    for entry in entries {
        let entry = entry?;
        let path = entry.path();
        
        if path.is_dir() {
            println!("DIR:  {:?}", path);
        } else {
            println!("FILE: {:?}", path);
        }
    }
    
    Ok(())
}
```

### 3.3 디렉토리 순회

```rust
use std::fs;
use std::io;
use std::path::Path;

fn visit_dirs(dir: &Path, depth: usize) -> io::Result<()> {
    if dir.is_dir() {
        let indent = "  ".repeat(depth);
        println!("{}DIR:  {:?}", indent, dir.file_name().unwrap());
        
        for entry in fs::read_dir(dir)? {
            let entry = entry?;
            let path = entry.path();
            
            if path.is_dir() {
                visit_dirs(&path, depth + 1)?;
            } else {
                println!("{}FILE: {:?}", indent, path.file_name().unwrap());
            }
        }
    }
    Ok(())
}

fn main() -> io::Result<()> {
    println!("디렉토리 트리:");
    visit_dirs(Path::new("."), 0)?;
    Ok(())
}
```

### 3.4 디렉토리 삭제

```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    // 빈 디렉토리 삭제
    // fs::remove_dir("empty_directory")?;
    
    // 디렉토리와 내용 모두 삭제
    // fs::remove_dir_all("directory_with_contents")?;
    
    println!("디렉토리 삭제 완료");
    Ok(())
}
```

## 📋 파일 메타데이터

### 4.1 파일 정보 조회

```rust
use std::fs;
use std::io;
use std::time::SystemTime;

fn main() -> io::Result<()> {
    let metadata = fs::metadata("hello.txt")?;
    
    // 파일 타입 확인
    println!("파일인가: {}", metadata.is_file());
    println!("디렉토리인가: {}", metadata.is_dir());
    println!("심볼릭 링크인가: {}", metadata.is_symlink());
    
    // 파일 크기
    println!("파일 크기: {} 바이트", metadata.len());
    
    // 권한 정보
    println!("읽기 가능: {}", metadata.permissions().readonly());
    
    // 시간 정보
    if let Ok(modified) = metadata.modified() {
        println!("수정 시간: {:?}", modified);
    }
    
    if let Ok(accessed) = metadata.accessed() {
        println!("접근 시간: {:?}", accessed);
    }
    
    if let Ok(created) = metadata.created() {
        println!("생성 시간: {:?}", created);
    }
    
    Ok(())
}
```

### 4.2 파일 경로 작업

```rust
use std::path::{Path, PathBuf};

fn main() {
    // Path 사용
    let path = Path::new("/home/user/documents/file.txt");
    
    println!("경로: {:?}", path);
    println!("부모: {:?}", path.parent());
    println!("파일 이름: {:?}", path.file_name());
    println!("확장자: {:?}", path.extension());
    println!("스템: {:?}", path.file_stem());
    
    // PathBuf 사용 (가변 경로)
    let mut path_buf = PathBuf::from("/home/user");
    path_buf.push("documents");
    path_buf.push("file.txt");
    
    println!("PathBuf: {:?}", path_buf);
    
    // 경로 조작
    let absolute_path = path_buf.canonicalize().unwrap_or_else(|_| path_buf.clone());
    println!("절대 경로: {:?}", absolute_path);
}
```

## 🔄 파일 복사 및 이동

### 5.1 파일 복사

```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    // 간단한 파일 복사
    fs::copy("source.txt", "destination.txt")?;
    println!("파일 복사 완료");
    
    // 수동 파일 복사 (더 많은 제어)
    use std::io::{Read, Write};
    
    let mut source = File::open("source.txt")?;
    let mut destination = File::create("destination_manual.txt")?;
    
    let mut buffer = [0; 1024];
    loop {
        let bytes_read = source.read(&mut buffer)?;
        if bytes_read == 0 {
            break;
        }
        destination.write_all(&buffer[..bytes_read])?;
    }
    
    println!("수동 파일 복사 완료");
    
    Ok(())
}
```

### 5.2 파일 이동 및 이름 변경

```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    // 파일 이름 변경
    fs::rename("old_name.txt", "new_name.txt")?;
    println!("파일 이름 변경 완료");
    
    // 파일 이동
    fs::rename("source.txt", "destination/new_name.txt")?;
    println!("파일 이동 완료");
    
    Ok(())
}
```

## 🔍 파일 검색

### 6.1 패턴으로 파일 찾기

```rust
use std::fs;
use std::io;
use std::path::Path;

fn find_files_by_extension(dir: &Path, extension: &str) -> io::Result<Vec<String>> {
    let mut found_files = Vec::new();
    
    for entry in fs::read_dir(dir)? {
        let entry = entry?;
        let path = entry.path();
        
        if path.is_dir() {
            // 재귀적으로 하위 디렉토리 검색
            let mut sub_files = find_files_by_extension(&path, extension)?;
            found_files.append(&mut sub_files);
        } else if let Some(ext) = path.extension() {
            if ext == extension {
                if let Some(path_str) = path.to_str() {
                    found_files.push(path_str.to_string());
                }
            }
        }
    }
    
    Ok(found_files)
}

fn main() -> io::Result<()> {
    let rust_files = find_files_by_extension(Path::new("."), "rs")?;
    
    println!("Rust 파일들:");
    for file in rust_files {
        println!("{}", file);
    }
    
    Ok(())
}
```

### 6.2 파일 내용 검색

```rust
use std::fs;
use std::io::{self, BufRead};
use std::path::Path;

fn search_in_file(file_path: &Path, search_term: &str) -> io::Result<Vec<(usize, String)>> {
    let file = fs::File::open(file_path)?;
    let reader = io::BufReader::new(file);
    
    let mut matches = Vec::new();
    
    for (line_num, line) in reader.lines().enumerate() {
        let line = line?;
        if line.contains(search_term) {
            matches.push((line_num + 1, line));
        }
    }
    
    Ok(matches)
}

fn search_in_directory(dir: &Path, search_term: &str) -> io::Result<Vec<(String, Vec<(usize, String) )>> {
    let mut results = Vec::new();
    
    for entry in fs::read_dir(dir)? {
        let entry = entry?;
        let path = entry.path();
        
        if path.is_file() {
            if let Ok(matches) = search_in_file(&path, search_term) {
                if !matches.is_empty() {
                    if let Some(path_str) = path.to_str() {
                        results.push((path_str.to_string(), matches));
                    }
                }
            }
        }
    }
    
    Ok(results)
}

fn main() -> io::Result<()> {
    let search_term = "TODO";
    let results = search_in_directory(Path::new("."), search_term)?;
    
    println!("'{}' 검색 결과:", search_term);
    for (file, matches) in results {
        println!("\n파일: {}", file);
        for (line_num, line) in matches {
            println!("  라인 {}: {}", line_num, line);
        }
    }
    
    Ok(())
}
```

## 📝 실용적인 파일 처리 예제

### 7.1 로그 파일 처리

```rust
use std::fs::{OpenOptions, File};
use std::io::{self, Write, BufRead, BufReader};
use std::time::SystemTime;

struct Logger {
    file: File,
}

impl Logger {
    fn new(filename: &str) -> io::Result<Self> {
        let file = OpenOptions::new()
            .create(true)
            .append(true)
            .open(filename)?;
        
        Ok(Logger { file })
    }
    
    fn log(&mut self, message: &str) -> io::Result<()> {
        let timestamp = SystemTime::now()
            .duration_since(SystemTime::UNIX_EPOCH)
            .unwrap()
            .as_secs();
        
        writeln!(&mut self.file, "[{}] {}", timestamp, message)?;
        self.file.flush()?;
        Ok(())
    }
    
    fn read_logs(filename: &str) -> io::Result<Vec<String>> {
        let file = File::open(filename)?;
        let reader = BufReader::new(file);
        
        let mut logs = Vec::new();
        for line in reader.lines() {
            logs.push(line?);
        }
        
        Ok(logs)
    }
}

fn main() -> io::Result<()> {
    // 로그 쓰기
    let mut logger = Logger::new("app.log")?;
    logger.log("애플리케이션 시작")?;
    logger.log("사용자 로그인")?;
    logger.log("데이터 처리 완료")?;
    
    // 로그 읽기
    let logs = Logger::read_logs("app.log")?;
    println!("로그 내용:");
    for log in logs {
        println!("{}", log);
    }
    
    Ok(())
}
```

### 7.2 설정 파일 처리

```rust
use std::collections::HashMap;
use std::fs;
use std::io;

struct Config {
    settings: HashMap<String, String>,
}

impl Config {
    fn new() -> Self {
        Config {
            settings: HashMap::new(),
        }
    }
    
    fn load_from_file(&mut self, filename: &str) -> io::Result<()> {
        let content = fs::read_to_string(filename)?;
        
        for line in content.lines() {
            // 주석과 빈 줄 무시
            if line.trim().is_empty() || line.trim().starts_with('#') {
                continue;
            }
            
            // key=value 형식 파싱
            if let Some((key, value)) = line.split_once('=') {
                self.settings.insert(key.trim().to_string(), value.trim().to_string());
            }
        }
        
        Ok(())
    }
    
    fn save_to_file(&self, filename: &str) -> io::Result<()> {
        let mut content = String::new();
        
        for (key, value) in &self.settings {
            content.push_str(&format!("{}={}\n", key, value));
        }
        
        fs::write(filename, content)?;
        Ok(())
    }
    
    fn get(&self, key: &str) -> Option<&String> {
        self.settings.get(key)
    }
    
    fn set(&mut self, key: String, value: String) {
        self.settings.insert(key, value);
    }
}

fn main() -> io::Result<()> {
    let mut config = Config::new();
    
    // 설정 값 추가
    config.set("database_url".to_string(), "sqlite://app.db".to_string());
    config.set("debug".to_string(), "true".to_string());
    config.set("port".to_string(), "8080".to_string());
    
    // 파일에 저장
    config.save_to_file("config.ini")?;
    println!("설정 파일 저장 완료");
    
    // 파일에서 로드
    let mut loaded_config = Config::new();
    loaded_config.load_from_file("config.ini")?;
    
    // 설정 값 읽기
    if let Some(db_url) = loaded_config.get("database_url") {
        println!("데이터베이스 URL: {}", db_url);
    }
    
    if let Some(debug) = loaded_config.get("debug") {
        println!("디버그 모드: {}", debug);
    }
    
    Ok(())
}
```

### 7.3 CSV 파일 처리

```rust
use std::fs::File;
use std::io::{self, BufRead, BufReader, Write};

struct CSVRow {
    fields: Vec<String>,
}

impl CSVRow {
    fn new() -> Self {
        CSVRow { fields: Vec::new() }
    }
    
    fn from_line(line: &str) -> Self {
        let fields = line.split(',').map(|s| s.trim().to_string()).collect();
        CSVRow { fields }
    }
    
    fn to_string(&self) -> String {
        self.fields.join(",")
    }
}

struct CSVReader {
    file: BufReader<File>,
}

impl CSVReader {
    fn new(filename: &str) -> io::Result<Self> {
        let file = File::open(filename)?;
        Ok(CSVReader {
            file: BufReader::new(file),
        })
    }
    
    fn read_row(&mut self) -> io::Result<Option<CSVRow>> {
        let mut line = String::new();
        
        match self.file.read_line(&mut line)? {
            0 => Ok(None),  // 파일 끝
            _ => {
                let line = line.trim_end_matches('\n');
                Ok(Some(CSVRow::from_line(line)))
            }
        }
    }
}

struct CSVWriter {
    file: File,
}

impl CSVWriter {
    fn new(filename: &str) -> io::Result<Self> {
        let file = File::create(filename)?;
        Ok(CSVWriter { file })
    }
    
    fn write_row(&mut self, row: &CSVRow) -> io::Result<()> {
        writeln!(&mut self.file, "{}", row.to_string())?;
        Ok(())
    }
}

fn main() -> io::Result<()> {
    // CSV 파일 쓰기
    let mut writer = CSVWriter::new("output.csv")?;
    
    // 헤더 쓰기
    let mut header = CSVRow::new();
    header.fields.push("Name".to_string());
    header.fields.push("Age".to_string());
    header.fields.push("City".to_string());
    writer.write_row(&header)?;
    
    // 데이터 쓰기
    let mut row1 = CSVRow::new();
    row1.fields.push("Alice".to_string());
    row1.fields.push("30".to_string());
    row1.fields.push("Seoul".to_string());
    writer.write_row(&row1)?;
    
    let mut row2 = CSVRow::new();
    row2.fields.push("Bob".to_string());
    row2.fields.push("25".to_string());
    row2.fields.push("Busan".to_string());
    writer.write_row(&row2)?;
    
    // CSV 파일 읽기
    let mut reader = CSVReader::new("output.csv")?;
    
    println!("CSV 내용:");
    while let Some(row) = reader.read_row()? {
        println!("{:?}", row.fields);
    }
    
    Ok(())
}
```

## 📝 연습 문제

### 문제 1: 파일 복사 도구
다음 기능들을 가진 파일 복사 도구를 구현하세요:

```rust
use std::fs;
use std::io;

// TODO: FileCopier 구조체 정의
// - buffer_size: usize

// TODO: FileCopier 메서드 구현
// - new(): 초기화
// - copy_file(): 파일 복사 (진행률 표시)
// - copy_directory(): 디렉토리 복사 (재귀적)

fn main() -> io::Result<()> {
    // TODO: 파일 및 디렉토리 복사 테스트
}
```

### 문제 2: 로그 분석기
다음 기능들을 가진 로그 분석기를 구현하세요:

```rust
use std::collections::HashMap;
use std::fs;
use std::io;

// TODO: LogEntry 구조체 정의
// - timestamp: String
// - level: String
// - message: String

// TODO: LogAnalyzer 구조체 정의
// - entries: Vec<LogEntry>

// TODO: LogAnalyzer 메서드 구현
// - parse_file(): 로그 파일 파싱
// - count_by_level(): 로그 레벨별 개수
// - filter_by_time(): 시간 범위 필터링
// - search_messages(): 메시지 검색

fn main() -> io::Result<()> {
    // TODO: 로그 파일 분석 테스트
}
```

### 문제 3: 간단한 데이터베이스
다음 기능들을 가진 간단한 파일 기반 데이터베이스를 구현하세요:

```rust
use std::collections::HashMap;
use std::fs;
use std::io;

// TODO: Record 구조체 정의
// - id: u32
// - key: String
// - value: String

// TODO: SimpleDB 구조체 정의
// - file_path: String
// - records: HashMap<String, String>

// TODO: SimpleDB 메서드 구현
// - new(): 초기화
// - load(): 파일에서 데이터 로드
// - save(): 파일에 데이터 저장
// - put(): 키-값 저장
// - get(): 값 조회
// - delete(): 키 삭제
// - list(): 모든 키 목록

fn main() -> io::Result<()> {
    // TODO: 데이터베이스 조작 테스트
}
```

---

**다음 단계**: [11_command_line_apps.md](./11_command_line_apps.md)에서 Rust로 CLI 애플리케이션을 만드는 방법을 학습하세요! 🦀
