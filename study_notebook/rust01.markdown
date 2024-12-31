# 01. Rust시작하기

### Cargo : 러스트의 빌드 시스템 및 패키지 매니저

- cargo를 사용하여 프로젝트 생성하기

```
$ cargo new hello_cargo --bin
$ cd hello_cargo
```

- --bin은 실행가능한 애플리케이션으로 만들어준다.
- TOML은 cargo의 환경설정 포맷.

&nbsp;

### Cargo 프로젝트를 빌드하고 실행하기

```
$ cargo build
$ ./target/debug/cargo_test
```

- `cargo build`나 `cargo check`를 활용하여 프로젝트를 빌드할 수 있다
- `cargo run`으로 프로젝트를 빌드하고 실행할 수 있다.
- 동일한 디렉토리에 빌드의 결과물이 저장되는 대신, cargo는 `target/debug` 디렉토리에 저장한다.

&nbsp;

### 릴리즈 빌드

- `cargo build --release`로 최적화와함께 이를 컴파일함
  - 이는 target/relese에 실행파일을 생성한다.
