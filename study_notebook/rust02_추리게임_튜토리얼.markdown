## 추리게임

---

### 1~100사이의 임의의 정수를 생성하고 플레이어가 추리한 정수를 입력하면 프로그램은 입력받은 추리값이 정답보다 높거나 낮은지 알려준다.

## #입력값 받기

```rust
// 사용자 입력을 받고 결과값을 표시하기 위해서는 io(input/output)라이브러리를 스코프로 가져와야한다
use std::io;

fn main() {
    println!("Guess the number!");
    println!("Please input your guess.");

    // rust에서 변수는 기본적으로 불변이다. 변수앞에 "mut"를 이용하여 가변변수를 만든다.
    // 새로운 빈 String인스턴스와 연결된 가변변수를 생성.
    let mut guess = String::new();

    //io의 연관함수인 stdin을 호출. read_line함수를 이용하여 문자열을 입력받는다.
    io::stdin().read_line(&mut guess)
    .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

- `read_line`은 사용자가 표준 입력에 입력할 때마다 입력된 문자들을 하나의 문자열에 저장하므로 인자로 값을 저장할 문자열이 필요하다. 그 문자열 인자는 사용자 입력을 추가하면서 변경되므로 가변이어야 함

- `io::Result` 인스턴스는 `expect` 메소드를 가지고 있습니다. 만약 io::Result 인스턴스가 `Err`일 경우 `expect` 메소드는 프로그램이 작동을 멈추게 하고 expect에 인자로 넘겼던 메세지를 출력하도록 함

- `expect`를 호출하지 않는다면, 컴파일은 되지만 경고가 나타난다.

&nbsp;

## #비밀번호 생성하기

- rand 크리에이트를 의존리스트에 추가해야한다.(랜덤값을 불러와야하기 때문)
- `Cargo.toml`에 추가

```
[dependencies]

rand = "0.3.14"
```

- `cargo build`실행

  > Compiling libc v0.2.169
  >
  > Compiling rand v0.4.6
  >
  > Compiling rand v0.3.23
  >
  > Compiling guessing_game v0.1.0 (/Users/sds/Documents/Rust_ex/guessing_game)
  >
  > Finished \`dev` profile [unoptimized + debuginfo] target(s) in 0.53s

- `cargo.lock`파일이 처음 빌드시 버전을 저장해두기때문에 다시 빌드할때 cargo.lock을 확인하고 없는것만 추가하는 형식

&nbsp;

### 임의의 숫자 생성하기

```rust
extern crate rand;

use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1,101);

    println!("the secret number is: {}", secret_number);

    println!("please input your guess");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("failed to read line");

    println!()("you guessed: {}", guess);
}
```

- 외부에 의존하는 crate가 있음을 알림

- `Rng`은 정수 생성기가 구현한 메소드들을 정의한 trait이다

- `rand::thread_rng`은 OS가 시드(seed)를 정하고 현재 스레드에서만 사용되는 특별한 정수생성기를 돌려 준다.

- 다음으로 `get_range`를 호출하여 임의의 숫자를 생성한다.

&nbsp;

### 비밀번호와 추리값을 비교하기

```rust
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number= rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {

        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        // 위에서 guess를 문자열로 선언했기때문에 정수로 비교하려면 정수로 바꿔줘야다.(shadow) --> guess를 재사용
        // trim()은 처음과 끝 부분의 빈칸을 제거, readline때문에 개행이 추가되었을 것임
        // parse()은 문자열을 정수로 변환
        let guess: u32 = guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

     // 두 숫자를 비교한 결과처리
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("you win!");
                break;
            },
        }
    }
}
```

- `loop`는 무한루프를 제공

- guess는 문자열로 선언했기때문에 비교를 위해 정수로 바꿔줘야한다.

- loop문을 빠져나와야하기 때문에 둘의 값이 같을때는 break로 loop를 빠져나온다.
