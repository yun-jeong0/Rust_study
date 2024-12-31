# 제네릭 타입, 트레잇, 라이프타임

모든 프로그래밍언어는 컨셉의 복제를 효율적으로 다루기 위한 도구를 가지고 있다.

제네릭(generic)은 구체화된 타입이나 다른 속성들에 대하여 추상화된 대리인

# 10-1. 제네릭 데이터 타입

구체적인 타입 대신, 일반화된 타입 매개변수를 사용할 수 있게 해준다.

### 함수 정의 내에서 제네릭 데이터 타입을 이용하기

슬라이스 내에서 가장 큰 값을 찾는 함수를 작성

```rust
fn largest_i32(list: &[i32]) --> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest

}

fn largest_char(list: &[char]) --> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
     }
     largest
}

fn main() {
    let number_lest = vec![34,23,445,67,34]

    let result = largest_i32(&number_list);
    println!("largest number is {}", result);

    let chars = vec!['y', 'm', 'a', 'q'];
    let result= largest_char(&chars);
    println!("largest char is {}", result);
}
```

여기서 함수 `largest_i32`와 `largest_char` 는 정확히 똑같은 본체를 가지고있으므로 만일 우리가 이 두 함수를 하나로 바꿔서 중복을 제거할 수 있다면 좋을 것이다.

--> 이걸 제네릭 타입 파라미터를 도입해서 한다.

타입들을 파라미터화 하기 위해서 타입파라미터를 위한 이름이 필요함

어떤 식별자든지 타입파라미터의 이름으로 사용될 수 있지만, 러스트 타입 이름에 대한 관례가 낙타 표기법(CamelCase)이기 때문에 T를 사용한다

```rust
fn largest<T>(list: &[T]) --> T{

}
```

함수의 본체에 파라미터를 이요할 때는, 스그니처 내에서 그 파라미터를 선언하여 해당 이름이 함수 본체 내에서 무엇을 의미하는지 컴파일러가 알 수 있다.

`largest`는 어떤 타입 `T`을 이용한 제네릭입니다. 이것은 `list`라는 이름을 가진 하나의 파라미터를 가지고 있고, `list`의 타입은 `T` 타입 값들의 슬라이스입니다. `largest` 함수는 동일한 타입 `T`를 반환

i32 값들의 슬라이스 혹은 char 값들의 슬라이스를 가지고 어떻게 largest를 호출할 수 있을까?

```rust
fn largest<T>(list:&[T]) ->T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let numbers = vec![34,23,445,67,34];
    let result = largest(&numbesrs);
    println!("largest number is {}", result);

    let chars = vec!['y', 'm', 'a', 'q'];
    let result = largest(&chars);
    println!("largest char is {}", result);
}
```

```
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

이 코드를 실행하려고 하면 위와같이 에러를 얻게된다

에러 : T가 될 수 있는 모든 가능한 타입에 대해서 동작하지 않으리라

### 구조체 정의 내에서 제네릭 데이터 타입 사용하기

구조체 필드 내에서도 제네릭 타입 파라미터를 사용하여 구조체를 정의할 수 있다.

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y:10};
    let float = Point { x:1.0, y:4.3};
}
```

- `Point`의 정의 내에서 단 하나의 제네릭 타입을 사용
- x와 y 필드는 둘 모두 동일한 제네릭 데이터 타입 T를 가지고 있기 때문에 동일한 타입이어야 함

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point {x: 5, y:10};
    let both_float = Point {x:1.0, y:4.3};
    let integer_and_float = Point {x: 5, y: 4.3};
}
```

- 두 타입을 이용한 제네릭이어서 x와 y가 다른 타입의 값일 수도 있는 Point

> 만약 제네릭 타입의 수가 많아야하는 지점에 다달았다면, 그건 코드가 더 작은조각으로 나뉘는게 필요할지도 모른다는 신호일 수 있음

&nbsp;

### 열거형 정의 내에서 제네릭 데이터 타입 사용하기

```rust
enum Option<T> {
    Some(T),
    None,
}
```

- `Option<T>`는 T 타입에 제네릭인 열거형
- 타입 T 값 하나를 들고 있는 Some, 그리고 어떠한 값도 들고 있지 않는 None variant, 총 2가지의 variant를 가지고 있음

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

- T와 E, 두 개의 타입을 이용한 제네릭
- 타입 T의 값을 들고 있는 Ok, 그리고 타입 E의 값을 들고 있는 Err, 총 2가지의 variant를 가지고 있음

&nbsp;

### 메소드 정의 내에서 제네릭 데이터 타입 사용하기

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.x());
}
```

- `Point<T>` 구조체 상에 x라는 이름의 메소드 정의(x값이 뭔지 보여줌)
- `impl` 바로 뒤에 T를 정의해야만 타입 `Point<T>` 메소드를 구현하는 중에 이를 사용할 수 있음

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
    // p3.x = 5, p3.y = c
}
```

- 제네릭 파라미터 T와 U는 impl 뒤에 선언되었는데, 이는 구조체 정의와 함께 사용되기 때문
- V와 W는 fn mixup 뒤에 선언되었는데, 이는 이들이 오직 해당 메소드에 대해서만 관련이 있기 때문

&nbsp;

---

# 10-2. trait, 공유 동작을 정의하기

만약 우리가 서로 다른 타입에 대해 모두 동일한 메소드를 호출할 수 있다면 이 타입들은 동일한 동작을 공유하는 것이다.

trait : 여러 타입들이 공통적으로 가질 수 있는 기능들을 하나로 묶어놓은 것.

예시) '운전 가능한 것'이라는 trait가 있다면, 자동차, 오토바이, 자전거 등 여러 타입들이 이 trait을 구현할 수 있는 것.

쉽게말해 trait은 "이런 기능이 있어야 해"라고 정의만 해두고, 실제로 각 타입에서 "그 기능을 이렇게 구현할거야"라고 구체적으로 작성하는 것이다.

<예시>
소셜미디어 컨텐츠를 요약해주는 라이브러리를 만든다고 하자.
이 라이브러리는 두가지 종류의 컨텐츠를 요약해준다.

1. `NewArticle` : 뉴스기사
2. `트위터 게시물`
   - 최대 140자의글
   - 리트윗인지 아닌지
   - 다른트윗에 대한 답글인지 아닌지 등의 정보

```rust
pub trait summarizable {
    fn summary(&self) -> String;
}
```

&nbsp;

# 특정 타입에 대한 트레잇 구현하기

- 이제 이 동작을 갖길 원했던 타입들 상에 이 트레잇을 구현할 수 있다..

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl summarizable for NewsArticle {
    fn summary(&self) -> String {
        format!("{} by {} in {}", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl summarizable for Tweet {
    fn summary(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

- trait가 정의한 메소드의 함수틀(summary)을 가져온다.
- 중괄호 {}안에 실제로 그 메소드가 어떻게 동작할지를 작성한다.
- 각 타입에 맞게 다르게 구현할 수 있다.

```rust
let tweet = Tweet {
    username: String::from("harry"),
    content: String::from("I'm learning Rust!"),
    reply: false,
    retweet: false,
};

println!("1 new teet: {}", tweet.summary());
```

- trait을 구현한 후에는 일반 메소드처럼 사용할 수 있다.

만약 trait이 aggregator라는 크레이트에 있다고 하고, 다른 개발자가 이 trait을 자신의 코드에서 사용하고싶어 한다면

```rust
// lib.rs
extern crate aggregator;

use aggregator::summarizable;

struct WeatherForecast {
    high_temp: f64,
    low_temp: f64,
    chance_of_rain: f64,
}

impl summarizable for WeatherForecast {
    fn summary(&self) -> String {
        format!("High: {}, Low: {}, Rain: {}", self.high_temp, self.low_temp, self.chance_of_rain)
    }
}
```

- 마치 우리가 다른 사람의 라이브러리를 사용할 때 use로 가져오는 것처럼, 우리의 trait도 같은 방식으로 가져와서 사용할 수 있습니다.
- 다른 크레이트에서 우리의 trait을 사용하려면, 그 trait이 pub(공개)로 선언되어 있어야 합니다.

> **trait 구현시 제한사항**
>
> 허용되는 경우:
>
> 1.  내가 만든 타입 + 남의 trait
>
> - 예: 내가 만든 Tweet 구조체에 표준 라이브러리의 Display trait 구현 가능
>
> 2. 남의 타입 + 내가 만든 trait
>
> - 예: 표준 라이브러리의 Vec에 내가 만든 Summarizable trait 구현 가능
>
> 허용되지 않는 경우:
>
> 1. 남의 타입 + 남의 trait
>
> - 예: 표준 라이브러리의 Vec에 표준 라이브러리의 Display trait 구현 불가

&nbsp;

### 기본구현

기본구현 : trait에서 메소드를 정의할 때 "기본적으로 이렇게 동작하게 할게"라고 미리 구현해두는 것

```rust
pub trait Summarizable {
    // 그냥 정의만 하는 경우 (기본 구현 없음)
    fn summary(&self) -> String;

    // 기본 구현을 포함하는 경우
    fn summary(&self) -> String {
        String::from("(Read more...)")  // 기본 동작
    }
}
```

- 장점
  - 모든 타입에서 같은 동작을 반복해서 구현할 필요가 없음
  - 필요한 경우에만 다르게 구현하면 됨

기본구현을 그대로 사용하고싶을때.

```rust
impl Summarizable for NewsArticle { }  // 비어있는 impl 블록

// 사용 예시
let article = NewsArticle {
    headline: String::from("펭귄즈 우승!"),
    location: String::from("피츠버그"),
    author: String::from("Ice"),
    content: String::from("피츠버그 펭귄즈가 NHL 최고팀이 되었습니다."),
};

println!("새 기사: {}", article.summary());
// 출력: "새 기사: (Read more...)"  <- 기본 구현이 사용됨
```

동일한 trait내에 다른 메소드들을 호출할 수 있고 어떤건 기본구현을 갖거나 어떤건 안가지고 있어도 된다. 그 trait을 사용하기 위해선 기본구현이 없는 메소드만 정의하면된다.

```rust
pub trait Summarizable {
    // 반드시 구현해야 하는 메소드
    fn author_summary(&self) -> String;

    // 기본 구현이 있는 메소드
    fn summary(&self) -> String {
        format!("작성자 보기... {}", self.author_summary())
    }
}

----
impl Summarizable for Tweet {
    fn author_summary(&self) -> String { //기본구현이 없는것만 정의하면돼
        format!("@{}", self.username)
    }
}
```

오버라이딩된 구현으로부터 기본 구현을 호출하는 것은 불가능하다

```rust
pub trait Animal {
    // 기본 구현이 있는 메소드
    fn make_sound(&self) -> String {
        String::from("Some generic animal sound")
    }
}

struct Dog;

impl Animal for Dog {
    fn make_sound(&self) -> String {
        // 이렇게 하면 안됨! (기본 구현을 호출하려고 시도)
        let basic_sound = Animal::make_sound(self);  // 이건 불가능!
        format!("{} and woof!", basic_sound)

    // 대신 새로운 구현을 완전히 따로 해야 함
        String::from("Woof!")
    }
}
```

- 일단 메소드를 오버라이드(재정의)하기로 했다면, 기존의 기본 구현은 완전히 잊어버려야 함
- 기본 구현을 "부분적으로" 사용하는 것은 불가능함

&nbsp;

### 트레잇 바운드

제네릭 타입에 특정 제약을 거는 방법

"이 타입은 반드시 이러이러한 기능을 가지고있어야해!"

```rust
pub fn notify<T: Summarizable>(item: T) {
    println!("Breaking news! {}", item.summary());
}
```

- `item`은 그냥 함수의 매개변수 이름. 이 매개변수 타입은 `T`이다.
- 우리는 먼저 `Summarizable trait`을 만들고, `Tweet`과 `NewsArticle` 같은 타입들에 대해 이를 구현했다.
- 그 다운에 우리는 이 트레잇을 사용하는 함수를 만들고 싶은것.
- T는 "아무 타입이나 될 수 있다"가 아니라 **"Summarizable" 트레잇을 구현한 타입만 될 수 있다**는 의미이다.

```rust
// 실제사용예시
// Tweet 인스턴스 생성
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("물론이죠!"),
    reply: false,
    retweet: false,
};

// notify 함수 호출
notify(tweet);  // 여기서 tweet이 item이 되는 거
```

\+ 를 사용해서 여러 트레잇을 동시에 요구할 수도 있다.

`pub fn notify<T: Summarizable + Display>(item: T)`

또한 여러개의 제네릭 타입과 여러개의 트레잇을 사용할 수 도 있다.

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

하지만 위의 코드는 너무 복잡하니 where절을 사용할 수 있다.

```rust
fn some_function<T, U>(t: T, u: U) -> i32
where T: Display + Clone,
      U: Clone + Debug
```

- Clone은 복사해서 사용하는거. Display를 복사해서도 쓰고 화면에도 나오게 하는것임
- `Clone + Debug`같은 경우는 Clone과 Debug가 구현된 타입만 받을 수 있다

&nbsp;

### 트레잇 바운드를 사용하여 largest 함수 고치기

아까 largest에서 T타입의 두 값을 > 연산자로 비교하려했다

```rust
fn largest<T>(list:&[T]) ->T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let numbers = vec![34,23,445,67,34];
    let result = largest(&numbesrs);
    println!("largest number is {}", result);

    let chars = vec!['y', 'm', 'a', 'q'];
    let result = largest(&chars);
    println!("largest char is {}", result);
}
```

rust는 여기서 "이 T라는 타입이 비교가능한지 어떻게 아냐"라고 묻는 것임

예를들면

```rust
// 이건 불가능합니다
struct Point { x: i32, y: i32 }
if Point{x:1, y:2} > Point{x:3, y:4} { ... }  // 두 점을 어떻게 비교하죠?
```

그래서 `PartialOrd` 라는 trait을 추가해본다

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {  // 이제 가능!
            largest = item;
        }
    }
    largest
}
```

- "T는 반드시 비교 가능한 타입이어야 해!"라고 Rust에게 알려주는 것임
- i32, char 같은 기본 타입들은 이미 PartialOrd가 구현되어 있어서 사용 가능

하지만 다시 아래와 같은 오류가 발생한다

```
error[E0508]: cannot move out of type `[T]`, a non-copy array
 --> src/main.rs:4:23
  |
4 |     let mut largest = list[0];
  |         -----------   ^^^^^^^ cannot move out of here
  |         |
  |         hint: to prevent move, use `ref largest` or `ref mut largest`

error[E0507]: cannot move out of borrowed content
 --> src/main.rs:6:9
  |
6 |     for &item in list.iter() {
  |         ^----
  |         ||
  |         |hint: to prevent move, use `ref item` or `ref mut item`
  |         cannot move out of borrowed content
```

이 문제점은 `list[0]`의 값을 `largest`로 **"이동(move)"**하려고 한다는 거

list는 빌린건데(&[T]), 그 책의 첫페이지를 가져가려고 하는것과 같은것이다.

그래서 저 타입들은 Copy trait을 구현하고 있는 타입이다 그래서 T의 trait bound에 Copy를 추가할 수 있다.

```rust
use std::cmp::PartialOrd;

fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];

    let result = largest(&numbers);
    println!("The largest number is {}", result);

    let chars = vec!['y', 'm', 'a', 'q'];

    let result = largest(&chars);
    println!("The largest char is {}", result);
}
```

- copy trait이 있으므로 list[0]이 복사가 된다.

&nbsp;

---

# 10-3. 라이프타임을 이용한 참조자 유효화

```rust
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

이건 컴파일이 안될거다. r의 값을 변경한 x가 스코프 밖으로 벗어나서

그럼 러스트는 이 코드가 허용되어서는 안된다는 것을 어떻게 결정할까

&nbsp;

### 함수에서의 제네릭 라이프타임

우선 두개의 스트링 슬라이스 중에서 긴 쪽을 반환하는 함수작성

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

그리고 저 longest함수 구현을

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

이렇게 하면 에러가 뜬다.

함수는 참조자(&str)을 반환하는데

rust는 이 반환되는 참조자가 x의 참조잔지, y의 참조잔지 알 수 없음

> rust가 걱정하는것
>
> x와 y중 어느것이 반환될지는 실행시점에 결정되는데 반환된 참조자가 가리키는 데이터가 이미 없어졌을 수도 있다고 생각

그래서 해야하는것이 참조자들의 유효기간을 명시적으로 표시하는 것이다.

&nbsp;

### 라이프타임 명시 문법

참조자들이 얼마나 오래 유효한지를 Rust에게 알려주는 방법

참조자의 실제수명을 바꾸는게 아니라, 여러 참조자들의 수명이 서로 어떻게 연관되어 있는지 알려주는 것이다.

```rust
&i32        // 일반 참조자
&'a i32     // 라이프타임 'a를 가진 참조자
&'a mut i32 // 라이프타임 'a를 가진 가변 참조자
```

> **규칙**
>
> 1.  항상 `'`로 시작
> 2.  보통 짧은 소문자 사용('a가 가장 일반적)
> 3.  &뒤에 위치

&nbsp;

### 함수 시그니처 내의 라이프타임 명시

라이프타임을 도입한 longest 함수

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len(){
        x
    } else {
        y
    }
}
```

- 함수는 정확한 라이프타임을 모른다
- 단지 둘이 같은기간동안 유효하다는 것만 알면 된다
- 라이프타임 명시는 에러를 막기위한다기 보다, 참조자들 간의 관계를 명확하게 표현하기 위한것이다.

&nbsp;

### 라이프타임의 측면에서 생각하기

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

문제 : `result`가 `longest`함수가 끝나는 지점에서 스코프 밖으로 벗어나게 되어 메모리 해제가 일어나게 되는데, 이 함수로부터 `result`의 참조자를 반환하려는 시도를 한다는것.

&nbsp;

### 구조체 정의 상에서의 라이프타임 명시

구조체가 참조자를 들고 있도록 할 수 있지만, 구조체 정의 내의 모든 참조자들에 대하여 라이프타임을 표시할 필요가 있다.

```rust
struct ImportantExcerpt<'a>{
    part: &'a str,
}

fn main() {
    let novel = String::from("call me ishmael. some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence};
    // 이미 split가 &str을 반환하고 있기 때문에, first_sentence에 참조자(&)를 붙일 필요가 없다.
}
```

&nbsp;

### 라이프타임 생략

rust가 뻔한 패턴이니까 직접 라이프타임을 써주지 않아도 될 것 같아 라고 판단하는 규칙들

> **규칙1** : 입력 파라미터는 각자의 라이프타임을 받음

```rust
fn foo(x: &str)          // 컴파일러: "음, x는 'a를 가져야겠네"
fn foo<'a>(x: &'a str)   // 이렇게 변환됨

fn foo(x: &str, y: &str) // 컴파일러: "x는 'a, y는 'b를 가져야겠네"
fn foo<'a, 'b>(x: &'a str, y: &'b str) // 이렇게 변환됨
```

> **규칙2** : 입력 라이프타임이 하나면, 그게 출력 라이프타임이 됨

```rust
fn foo(x: &str) -> &str  // 입력이 하나면
fn foo<'a>(x: &'a str) -> &'a str  // 출력도 같은 'a를 씀
```

> **규칙3** : 메소드에서 &self가 있으면, self의 라이프타임이 출력 라이프타임이 됨

```rust
impl Something {
    fn do_something(&self, x: &str) -> &str {  // self가 있으니
        // self의 라이프타임이 반환값의 라이프타임이 됨
    }
}
```

규칙을 순서대로 적용해보고 모든 참조자의 라이프타임이 명확하게 결정이 되었으면 생략해도 된다.

&nbsp;

### 메소드 정의 내에서의 라이프타임 명시

구조체의 메소드에서 라이프타임을 다루는 방법

```rust
//기본적인 메소드 예제
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

- 구조체가 라이프타임 'a를 가지므로 impl 뒤에도 <'a>를 써줘야함
- level 메소드는 참조자를 반황하지 않으므로 라이브타임 신경쓸 필요 없음

```rust
// 좀 더 복잡한 메소드 예제
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

이 메소드는 생략규칙이 적용된다.

- 규칙1적용 :
  &self는 자동으로 'a받음, announcement는 자동으로 'b받음
- 규칙 3적용 : &self가 있으므로, 반환값은 self의 라이프타임이어야해.

따라서 이렇게도 쓸 수 있다

```rust
// 실제 컴파일러가 이해하는 버전
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &'a str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

&nbsp;

### 정적 라이프타임(Static lifetime)

프로그램이 시작될 때부터 끝날 때까지 계속 유효한 데이터

라이프타임 에러를 해결하기 위한 마법의 해결책으로 사용하면 안 됨

`let s: &'static str = "I have a static lifetime.";`
