# 7-1. mod와 파일시스템

라이브러리 크레이트(crate) : 다른사람들이 자신들의 프로젝트에 dependency로 추가할 수 있는 프로젝트

ex. 2장의 rand 크레이트

우선 communicator라고 부르는 라이브러리를 생성

```
$ cargo new communicator --lib
$ cd communicator
```

- 카고가 src/main.rs 대신 src/lib.rs를 생성

```rust
//Filename: src/lib.rs
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
    }
}
```

- --bin 옵션을 사용했을때 "hello world" 바이너리를 만들어준것과 사뭇 다르다
- src/main.rs 파일이 없기 때문에, cargo run 커맨드를 실행할 것이 없다. 단지 lib.rs를 컴파일하기 위해 `cargo build`를 사용한다

&nbsp;

## 모듈정의

connect라는 이름의 함수가 정의되어있는 network 모듈을 정의

```rust
mod network {
    fn connect() {
    }
}
```

- src/lib.rs의 위쪽에 추가한다.
- 이 모듈을 쓸때는 `network::connect();`로 표현한다,

모듈안에 모듈도 만들 수 있다.

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

모듈을 계층구조를 이루는데 위코드의 계층구조는 아래와 같이 구성될 것이다

```
communicator
 └── network
     └── client
```

만약 이 모듈들이 여러 개의 함수를 갖기 사작하면 보기가 불편해질것이다.
그래서 client, network, server모듈을 src/lib.rs로부터 떼어네어 각자를 위한 파일에 위치시키면 좋을것 같다.

```rust
// src/lib.rs
mod client;
// 원래는
// mod client {
//     fn connect() {
//     }
// }

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

우선 client모듈을 선언하고있지만, 코드블록을 세미콜론으로 대체함으로서, 우리는 **client모듈의 스코프내에 정의된 코드를 다른위치에서 찾아라** 라고 전달하는것

다음은 모듈의 이름과 같은 이름을 가진 외부파일을 만들필요가 있다.(ex. client.rs) `src/client.rs`와 같은 형태로 만들어질것 그 안에 앞서 제거했던 `connect` 함수를 입력

```rust
// src/client.rs
fn connect() {
}
```

이 파일은 단지 client 모듈의 내용물만 제공할 뿐이다.

> 러스트는 기본적으로 src/lib.rs만 찾아볼줄 안다. 이는 `mod client`가 왜 `src/lib.rs`에 있는지, `src/client.rs`내에는 정의될 수 없는지에 대한 이유이다.

이어서 network도 위의 과정을 진행한다.

```rust
// src/lib.rs
mod client;

mod network;
```

그리고 새로운 `src/network.rs` 파일을 만들어준다.

```rust
// src/network.rs
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

하지만 여전히 `mod 선언`이 있다. server가 서브모듈로 필요하기 때문이다.

하지만 src/network.rs에 `mod server;`를 생성할 수 없다.

아래와 같은 방법을 만든다

> 1. 부모 모듈의 이름에 해당하는, network라는 이름의 새로운 디렉토리 형성
> 2. src/network.rs 파일을 새로운 network 디렉토리 안으로 옮기고 파일이름을 **src/network/mod.rs로 변환**
> 3. 서브모듈파일 src/server.rs를 network 안으로 옮긴다

이에 대응하는 파일 레이아웃을 아래와 같다

```
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

&nbsp;

# 7-2. pub으로 가시성 제어하기

build는 했지만 사용하지 않고있는 `client::connect`, `network::connect`, 그리고 `network::server::connect` 함수에 대한 경고 메세지를 보게 된다.

경고들을 해제하기위해 connect 라이브러리를 다른프로젝트에서 사용해보자

```rust
// src/main.rs
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

하지만 `cargo build`를 실행하면 에러가 뜰것임

```
error: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

이 에러는 client모듈이 비공개(private)임을 알려준다.

러스트의 모든 코드의 기본상태는 비공개이다.

&nbsp;

## 함수를 공개로 만들기

> pub : 러스트에게 어떤것을 공개하도록 말하기 위함

```rust
// src/lib.rs
pub mod client;

mod network;

// src/client.rs
pub fn connect() {
}
```

&nbsp;

# 7-3. use로 이름 가져오기

완전하게 지정된 경로를 참조하는건 너무 길어질 수 있다

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules(); // 너무 길다
}
```

&nbsp;

## use를 이용한 간결한 가져오기

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of;

fn main() {
    of::nested_modules();
}
```

- `of` 모듈을 참조하고 싶은 곳 마다 `a::series::of` 전부를 사용하기 보다는 of를 사용할 수 있다

하지만 모듈의 모든 자식을 스코프 내로 가져오진 않는다.

그래서 use구문안에서 `use a::series::of::nested_modules;`이런식으로 함수를 명시하여 스코프 내에서 함수를 가져올 수도 있다.

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of::nested_modules;

fn main() {
    nested_modules();
}
```

열거형 또한 모듈과 비슷한 구조를 형성하고 있기 때문에, 열거형의 variant도 use를 이용하여 가져올 수 있다.

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green; //use에 Green을 포함하지 않아서
}
```

&nbsp;

## \*을 이용한 글롭(glob)가져오기(모든 아이템)

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::*;
//편하지만 많은 아이템을 가져와서 이름간의 충돌이 생길 수 있다.

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = Green;
}
```

&nbsp;

## super를 사용하여 부모 모듈에 접근하기
