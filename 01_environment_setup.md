# 1. Rust 환경 설정 완벽 가이드

## 🦀 Rust 설치

### 1.1 rustup을 통한 설치 (권장)

Rustup은 Rust의 공식 버전 관리 도구입니다. 여러 버전의 Rust를 쉽게 관리할 수 있습니다.

```bash
# 설치 스크립트 실행
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 설치 과정에서 선택 옵션:
# 1) Default installation (권장)
# 2) Custom installation
# 3) Cancel installation
```

### 1.2 설치 확인

```bash
# Rust 버전 확인
rustc --version

# Cargo 버전 확인  
cargo --version

# rustup 버전 확인
rustup --version

# 설치된 컴포넌트 목록
rustup component list
```

### 1.3 추가 컴포넌트 설치

```bash
# Rust 소스 코드 (문서 생성에 필요)
rustup component add rust-src

# Rustfmt (코드 포매터)
rustup component add rustfmt

# Clippy (린트 도구)
rustup component add clippy

# RLLS (Rust Language Server - rust-analyzer로 대체됨)
rustup component add rls
```

## 🛠️ IDE 설정

### 2.1 VS Code 설정 (강력 추천)

#### 확장 프로그램 설치
1. **rust-analyzer** - Rust 언어 서버
2. **CodeLLDB** - 디버거
3. **Better TOML** - Cargo.toml 편집 지원
4. **Crates** - 의존성 버전 관리

#### VS Code 설정
```json
// .vscode/settings.json
{
    "rust-analyzer.checkOnSave.command": "clippy",
    "rust-analyzer.cargo.loadOutDirsFromCheck": true,
    "rust-analyzer.procMacro.enable": true,
    "rust-analyzer.inlayHints.typeHints.enable": true,
    "rust-analyzer.inlayHints.parameterHints.enable": true,
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "rust-lang.rust"
}
```

### 2.2 다른 IDE 옵션

#### IntelliJ IDEA / CLion
- Rust 플러그인 설치
- IntelliJ의 강력한 리팩토링 기능 활용

#### Vim/Neovim
```vim
" .vimrc 또는 init.vim
Plug 'rust-lang/rust.vim'
Plug 'fatih/vim-rustfmt'
Plug 'racer-rust/vim-racer'
```

#### Emacs
```elisp
;; .emacs 또는 init.el
(use-package rust-mode
  :ensure t
  :config
  (setq rust-format-on-save t))

(use-package flycheck-rust
  :ensure t
  :config
  (add-hook 'flycheck-mode-hook #'flycheck-rust-setup))
```

## 📦 Cargo 기본 명령어 마스터하기

### 3.1 프로젝트 생성

```bash
# 새 바이너리 프로젝트 생성
cargo new my_project

# 새 라이브러리 프로젝트 생성
cargo new --lib my_library

# 기존 디렉토리에 프로젝트 초기화
cargo init

# Git 저장소와 함께 프로젝트 생성
cargo new my_project --vcs git
```

### 3.2 빌드 명령어

```bash
# 디버그 빌드 (기본)
cargo build

# 릴리즈 빌드 (최적화)
cargo build --release

# 특정 타겟으로 빌드
cargo build --target x86_64-unknown-linux-musl

# 의존성만 다운로드
cargo fetch
```

### 3.3 실행 명령어

```bash
# 프로젝트 실행
cargo run

# 특정 바이너리 실행
cargo run --bin binary_name

# 인자 전달하여 실행
cargo run -- --arg1 value1 --arg2 value2

# 릴리즈 모드로 실행
cargo run --release
```

### 3.4 테스트 명령어

```bash
# 모든 테스트 실행
cargo test

# 특정 테스트만 실행
cargo test test_name

# 특정 모듈의 테스트 실행
cargo test module_name

# 테스트 출력 자세히 보기
cargo test -- --nocapture

# 단일 스레드로 테스트 실행
cargo test -- --test-threads=1
```

### 3.5 검사 및 분석

```bash
# 빠른 문법 검사 (컴파일 없이)
cargo check

# 린트 검사
cargo clippy

# 코드 포맷팅
cargo fmt

# 포맷팅 확인만 하기
cargo fmt -- --check

# 문서 생성
cargo doc

# 문서 생성 후 브라우저에서 열기
cargo doc --open
```

### 3.6 의존성 관리

```bash
# 의존성 추가
cargo add serde

# 특정 버전의 의존성 추가
cargo add serde@1.0.150

# 개발 의존성 추가
cargo add --dev tokio-test

# 빌드 의존성 추가
cargo add --build cc

# 의존성 제거
cargo remove serde

# 의존성 업데이트
cargo update

# 최신 버전으로 의존성 업데이트
cargo update --package package_name
```

## 🔧 고급 설정

### 4.1 Cargo 설정 파일

#### ~/.cargo/config.toml
```toml
# 빌드 타겟 설정
[build]
target = "x86_64-unknown-linux-gnu"

# 네트워크 설정
[net]
git-fetch-with-cli = true

# 레지스트리 설정
[registry]
default = "crates-io"

# 소스 설정
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"

# 토큰 설정 (crates.io 게시용)
[registry]
token = "api_token_here"
```

### 4.2 프로젝트별 설정

#### .cargo/config.toml
```toml
# 프로젝트별 빌드 설정
[build]
rustflags = ["-C", "target-cpu=native"]

# 타겟별 설정
[target.x86_64-unknown-linux-gnu]
rustflags = ["-C", "link-arg=-Wl,--as-needed"]

# 환경 변수 설정
[env]
CUSTOM_VAR = "custom_value"
```

### 4.3 Cargo.toml 상세 설정

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <your.email@example.com>"]
description = "A brief description of my project"
license = "MIT OR Apache-2.0"
repository = "https://github.com/username/my_project"
homepage = "https://example.com"
documentation = "https://docs.rs/my_project"
keywords = ["keyword1", "keyword2"]
categories = ["category1", "category2"]
readme = "README.md"

[dependencies]
# 기본 의존성
serde = "1.0"
serde_json = "1.0"

# 특정 기능만 포함
tokio = { version = "1.0", features = ["full"] }

# 조건부 의존성
[target.'cfg(unix)'.dependencies]
libc = "0.2"

[dev-dependencies]
# 개발용 의존성
tokio-test = "0.4"
mockall = "0.11"

[build-dependencies]
# 빌드 스크립트용 의존성
cc = "1.0"

[[bin]]
name = "main_binary"
path = "src/main.rs"

[[bin]]
name = "utility_binary"
path = "src/util.rs"

[lib]
name = "my_library"
path = "src/lib.rs"

[profile.dev]
opt-level = 0
debug = true
overflow-checks = true

[profile.release]
opt-level = 3
debug = false
lto = true
codegen-units = 1
panic = "abort"

[profile.test]
opt-level = 1
debug = true
```

## 🚀 유용한 도구들

### 5.1 cargo-watch (자동 빌드)

```bash
# 설치
cargo install cargo-watch

# 파일 변경 시 자동으로 빌드
cargo watch

# 파일 변경 시 자동으로 테스트
cargo watch -x test

# 파일 변경 시 자동으로 클리피 실행
cargo watch -x clippy

# 특정 파일만 감시
cargo watch --watch src --watch tests
```

### 5.2 cargo-audit (보안 취약점 검사)

```bash
# 설치
cargo install cargo-audit

# 보안 감사 실행
cargo audit

# 특정 ID의 취약점 무시
cargo audit --ignore RUSTSEC-2021-0001

# 데이터베이스 업데이트
cargo audit --fetch
```

### 5.3 cargo-deny (의존성 검사)

```bash
# 설치
cargo install cargo-deny

# 설정 파일 생성
cargo deny init

# 검사 실행
cargo deny check

# 라이선스 검사
cargo deny check licenses

# 보안 검사
cargo deny check bans
```

### 5.4 cargo-expand (매크로 확장)

```bash
# 설치
cargo install cargo-expand

# 매크로 확장 결과 보기
cargo expand

# 특정 함수만 확장
cargo expand --lib function_name
```

## 🐛 문제 해결

### 6.1 흔한 설치 문제

#### SSL/TLS 오류
```bash
# OpenSSL 설치 (macOS)
brew install openssl

# 환경 변수 설정
export OPENSSL_DIR=/usr/local/opt/openssl
```

#### 프록시 환경
```bash
# HTTP 프록시 설정
export http_proxy=http://proxy.example.com:8080
export https_proxy=http://proxy.example.com:8080

# Cargo 설정
echo '[net]
git-fetch-with-cli = true' >> ~/.cargo/config
```

### 6.2 빌드 문제

#### 링커 오류 (Windows)
```bash
# MSVC 빌드 도구 설치
# Visual Studio Installer에서 C++ build tools 설치

# 또는 GNU 툴체인 사용
rustup toolchain install stable-x86_64-pc-windows-gnu
rustup default stable-x86_64-pc-windows-gnu
```

#### 메모리 부족
```bash
# 빌드 시 메모리 제한
export CARGO_BUILD_JOBS=1

# 또는 설정 파일에 추가
echo '[build]
jobs = 1' >> ~/.cargo/config
```

## 📚 다음 단계

환경 설정이 완료되었다면 다음으로 진행하세요:

1. **[02_basic_syntax.md](./02_basic_syntax.md)** - Rust 기본 문법 학습
2. **[03_ownership_system.md](./03_ownership_system.md)** - 소유권 시스템 이해
3. **[04_project_structure.md](./04_project_structure.md)** - 프로젝트 구조 이해

---

**팁**: 환경 설정은 한 번만 하면 되지만, 제대로 하는 것이 중요합니다. 시간을 들여 꼼꼼하게 설정하세요! 🦀
