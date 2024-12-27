# 6-1. 열거형 정의하기

열거형은 언제 구조체보다 유용하고 적절하게 사용될까?

> #### 음료수 주문 예시
>
> 구조체 : 한사람이 동시에 콜라,스프레이드,환타 다 마시는 이상한 상황이 생김
>
> 열거형 : 한번에 하나의 음료만 선택할 수 있다.
>
> 구조체는 "여러 속성이 함께 있어야하는 경우에"유리하다
>
> ex. IP주소는 버전 4나 6중에 하나이며 동시에 두버전이 될 수 없다 --> 열거형 자료구조가 적합

<열거형 정의>

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

<열거형 인스턴스 생성>

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

<ipaddrkind타입을 인자로 받는 함수정의>

```rust
fn route(ip_type: IpAddrKind) { }
```

<함수호출>

```rust
route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

> 지금은 정의만 할뿐 아무런 값도 들어가있지 않다

<데이터 붙이기>

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

V4와 v6 variant는 연관된 String타입의 값을 갖게 된다.

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

V4와 같은 구성요소도 가능하다.

<메소드 정의>

```rust
impl Message {
    fn call(&self) {
        // 메소드 내용은 여기 정의할 수 있습니다.
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

&nbsp;

### Option 열거형과 Null 값 보다 좋은 점들

null값을 null이 아닌값처럼 사용하려할때 여러 종류의 오류가 생기기때문에

**rust에서는 null이 없지만, 값의 존재 혹은 부재의 개념을 표현할 수 있는 열거형이 있다**

```rust
enum Option<T> {
    Some(T),  // T : 어떤타입의 데이터라도 가질 수 있다.
    None,
}
```

컴파일러는 `option<T>`의 값을 명확하게 유효한 값처럼 사용하지 못하도록 한다

예를들면

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```

위의 코드는 컴파일 되지않는다.

x는그냥 테이블위에 놓여있는 사과라고하고, y는 상자안에 들어있는 사과라고하자(상자가 비었을 수도 있다.) 그래서 `let sum = x + y;`처럼 할 수가 없는것, 대신

```rust
let sum = match y {
    Some(number) => x + number,  // 상자에 사과가 있으면 꺼내서 더하기
    None => x,                   // 상자가 비어있으면 그냥 x 사용
};
```

또는 더 간단하게

```rust
let sum = x + y.unwrap_or(0);
```

이렇게 사용할 수 있다.

---

---

# 6-2. match 흐름 제어 연산자

일련의 패턴에 어떤 값을 비교한뒤 어떤 패턴에 매치되었는지를 바탕으로 코드를 수행하도록 해준다.

```rust
//동전이 어떤것이고 센트로 해당값을 반환하는
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

> 어떠한 갈래와 매칭이 되면 나온다.

패턴과 연관된 코드도 실행시킬 수 있다

```rust
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
            // 문자열도 출력하지만 해당블록 마지막값인 1도 반환한다
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

&nbsp;

### 값을 바인딩하는 패턴

미국은 각 50개 주마다 한쪽면의 디자인이 다른 쿼터동전을 주조했다. 오직 쿼터 동전만 특별한 값을 갖게 되는데...이를 나타낼 수 있다.

```rust
#[derive(Debug)] // 출력으로 사용할 수 있게
enum UsState {
    Alabama,
    Alaska,
    // ... etc
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

> #### 표현식
>
> `Coin::Quarter`의 값과 매치되는 패턴에 `state`라는 이름의 `변수`를 추가
>
> state 변수는 그 쿼터 동전의 주에 대한 값에 바인드

```rust
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

만약 `value_in_cents(Coin::Quarter(UsState::Alaska))`를 호출했다면, coin은
`Coin::Quarter(UsState::Alaska)`가 된다.

이 시점에서 state에 대한 바인딩 값은 `UsState::Alaska`이고,

이 바인딩을 println!표현식에 사용할 수 있다.

&nbsp;

### Opthon\<T>를 이용하는 매칭

Option\<i32>를 파라미터로 받아서 내부에 값이 있으면 그 값에 1을 더하는 함수를 작성
,만약 내부에 값이 없으면 함수는 None을 반환하고 어떤 연산도 수행하는 시도를 하면 안된다.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None, // None케이스를 다루지 않으면 오류가 난다
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

&nbsp;

### \_변경자(placeholder)

모든 가능한 값을 나열하기는 쉽지 않다.

ex. u8은 0부터 255까지 유요한 값을 가지는데 모든 값을 매치할 필요가 없기때문에 사용

```rust
let some_u8_value = 0u8; //u8형식의 0
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

- `_`는 그 전에 명시하지 않은 모든 가능한 경우에 대해 매칭됨
- `()`는 단위값이라 아무일도 일어나지 않음

---

---

# 6-3. if let을 사용한 간결한 흐름제어

match표현식은 한가지 경우에 대해 고려하는 상황에서는 장황할 수 있다

if와 let을 조합하여 하나의 패턴만 매칭시키고, 나머지경우는 무시하는 값을 다룬다.

<정식>

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

<iflet 이용>

```rust
if let Some(3) = some_u8_value {
    println!("three");
}
```

또한 iflet과 함께 `else`를 포함시킬 수 있다. 이는 match표현식에서 \_케이스 위에 나오는 코드블록과 동일하다.

```rust
// 만일 쿼터가 아닌 모든 동전을 세고싶은 동시에 쿼터동전일 경우 또한 알려주고싶었다면,

// match
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}

// iflet
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```
