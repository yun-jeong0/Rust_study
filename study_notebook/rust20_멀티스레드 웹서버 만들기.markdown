# 싱글스레드 기반 웹서버 만들기

### TCP 연결에 대한처리

일단 새 프로젝트를 만든다

```
$ cargo new hello --bin
     Created binary (application) `hello` project
$ cd hello
```

다음의 코드는 `127.0.0.1:7878`주소로 TCP연결 요청에 대해 수신대기할거다. 요청이 들어온다면 `connection established!`를 출력한다.

```rust
use std::net::TcpListener;

fn main() {
    //TcpListener의 새 인스턴스를 반환한다.
    // bind함수는 바인딩의 성공여부를 나타내는 Result<T, E> 타입의 인스턴스를 반환한다.
    // 따라서 unwrap을 이용하여 에러가 생길경우 프로그램을 멈출것이다.
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    // incoming은 스트림의 차례에 대한 반복을 반환한다.
    for stream in listener.incoming() {
        // unwrap을 호출하여 오류가 있을경우 프로그램을 종료시키는 방식으로 스트림을 처리한다.
        let stream = stream.unwrap();

        println!("Connection established!");
    }
}
```

`cargo run`를 실행하면

접속하면 여러번 요청으로 Connection established!가 여러번 뜰것이다.

중요한건 연결이 되었다는 것.

&nbsp;

### 요청 데이터 읽기

```rust

use std::io::prelude::*;  // 스트림으로부터 읽고 쓰는 것을 허용할 수 있다.
use std::net::TcpStream;
use std::net::TcpListener;

fn main() {
    //TcpListener의 새 인스턴스를 반환한다.
    // bind함수는 바인딩의 성공여부를 나타내는 Result<T, E> 타입의 인스턴스를 반환한다.
    // 따라서 unwrap을 이용하여 에러가 생길경우 프로그램을 멈출것이다.
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    // incoming은 스트림의 차례에 대한 반복을 반환한다.
    for stream in listener.incoming() {
        // unwrap을 호출하여 오류가 있을경우 프로그램을 종료시키는 방식으로 스트림을 처리한다.
        let stream = stream.unwrap();

        println!("Connection established!");
    }
}

// TcpStream 인스턴스가 내부에서 어떤 데이터가 우리에게 반환되는지 추적하기 때문에
// 내부의 상태가 변경될 수 있어 가변으로 해야한다.
fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];

    // stream.read()는 TcpStream으로부터 읽어들인 바이트를 버퍼로 집어넣는 역할을 한다
    stream.read(&mut buffer).unwrap();
    // buffer를 문자열로 변환
    println!("Request: {}", String::from_utf8_lossy(&buffer));
}
```

&nbsp;

### 응답 작성하기

```rust
use std::io::prelude::*;  // 스트림으로부터 읽고 쓰는 것을 허용할 수 있다.
use std::net::TcpStream;
use std::net::TcpListener;

fn main() {
    //TcpListener의 새 인스턴스를 반환한다.
    // bind함수는 바인딩의 성공여부를 나타내는 Result<T, E> 타입의 인스턴스를 반환한다.
    // 따라서 unwrap을 이용하여 에러가 생길경우 프로그램을 멈출것이다.
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    // incoming은 스트림의 차례에 대한 반복을 반환한다.
    for stream in listener.incoming() {
        // unwrap을 호출하여 오류가 있을경우 프로그램을 종료시키는 방식으로 스트림을 처리한다.
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

// TcpStream 인스턴스가 내부에서 어떤 데이터가 우리에게 반환되는지 추적하기 때문에
// 내부의 상태가 변경될 수 있어 가변으로 해야한다.
fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];

    // stream.read()는 TcpStream으로부터 읽어들인 바이트를 버퍼로 집어넣는 역할을 한다
    // 상대방이 보낸 데이터를 우리 버퍼에 받아들이는 것.
    stream.read(&mut buffer).unwrap();

    // 응답메세지 저장
    let response = "HTTP/1.1 200 OK\r\n\r\n";

    // write() 메소드는 &[u8]을 전달받고 커넥션에 바이트배열을 전송한다.
    // 우리가 만든 데이터를 상대방에게 보내는것.
    // write작업이 실패할 수 있기 때문에 unwrap을 사용하지만, 실제는 에러처리를 해야한다.
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

&nbsp;

### 실제 HTML로 응답하기

```rust
use std::fs::File;
// --생략--

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];

    // stream.read()는 TcpStream으로부터 읽어들인 바이트를 버퍼로 집어넣는 역할을 한다
    // 상대방이 보낸 데이터를 우리 버퍼에 받아들이는 것.
    stream.read(&mut buffer).unwrap();

    let mut file = File::open("hello.html").unwrap();

    let mut contents = String::new();
    file.read_to_string(&mut contents).unwrap();

    // 응답메세지 저장
    let response = format!(
        "HTTP/1.1 200 OK\r\nContent-Length: {}\r\n\r\n{}",
        contents.len(),
        contents
    );

    // write() 메소드는 &[u8]을 전달받고 커넥션에 바이트배열을 전송한다.
    // 우리가 만든 데이터를 상대방에게 보내는것.
    // write작업이 실패할 수 있기 때문에 unwrap을 사용하지만, 실제는 에러처리를 해야한다.
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

&nbsp;

### 요청을 확인하고 선택적으로 응답하기

```rust
fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

    let get =b"GET / HTTP/1.1\r\n";

    if buffer.starts_with(get){
        let mut file = File::open("hello.html").unwrap();

        let mut contents = String::new();
        file.read_to_string(&mut contents).unwrap();

        let response = format!(
            "HTTP/1.1 200 OK\r\nContent-Length: {}\r\n\r\n{}",
            contents.len(),
            contents
        );


        stream.write(response.as_bytes()).unwrap();
        stream.flush().unwrap();

    }else{
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let mut file = File::open("404.html").unwrap();
        let mut contents = String::new();

        file.read_to_string(&mut contents).unwrap();
        let response = format!(
            "{} \r\nContent-Length: {}\r\n\r\n{}",
            status_line,
            contents.len(),
            contents
        );

        stream.write(response.as_bytes()).unwrap();
        stream.flush().unwrap();
    }
}
```

이제 127.0.0.1:7878 로의 요청은 hello.html 의 내용을 반환할 것이고, 그 외의 요청 (127.0.0.1:7878/foo 등)은 404.html 의 내용을 반환할 것.

&nbsp;

### 리팩토링

if와 else블록에 중복되는 부분이 많다. 둘다 파일을 읽고 파일의 내용을 스트림에 작성,

이 차이점만 if와 else줄로 분리하고 status line과 파일명을 변수로 맡기도록

```rust
fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

    let get =b"GET / HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get){
        ("HTTP/1.1 200 OK", "hello.html")
    }else{
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let mut file = File::open(filename).unwrap();
    let mut contents = String::new();

    file.read_to_string(&mut contents).unwrap();

    let response = format!(
        "{}\r\nContent-Length: {}\r\n\r\n{}",
        status_line,
        contents.len(),
        contents
    );

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

&nbsp;

# 싱글스레드 서버를 멀티스레드 서버로 바꾸기

현재 서버는 한번에 하나의 요청만 처리하고있다 즉 첫번째 요청에 대한 작업이 끝나기 전에 두번째 요청이 들어온다면 앞선 작업이 끝날때까지 대기하게 된다

### 스레드 풀을 이용한 처리량 증진

스레드 풀 : 대기중이거나 작업을 처리할 준비가 되어 있는 스레드들의 그룹

풀 안에서 대기할 고정된 개수의 스레드를 가질것.

요청이 들어오면 요청들은 처리르 위해 풀로 보내지고, 풀에선 들어오는 요청들에 대한 큐를 유지한다. 이 형태를 이용해 동시에 N개의 요청을 처리할 수 있다.

&nbsp;

### 요청마다 스레드를 생성할 수 있는 코드 구조

스레드를 무한정생산하는건 결국 시스템의 과부하를 일으킴

&nbsp;

### 유한 스레드 수를 위한 인터페이스 만들기
