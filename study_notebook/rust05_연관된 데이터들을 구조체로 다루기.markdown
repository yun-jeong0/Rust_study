# 5-1. 구조체를 정의하고 생성하기

- 구조체의 구성요소들은 각자 다른 타입을 지닐 수 있다.
- 튜플과는 다르게 각 구성요소들은 명명할 수 있어 값이 의미하는 바를 명확하게 인지할 수 있다.

<구조체 정의>

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

<구조체 인스턴스 생성>

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

<구조체에 인스턴스에 들어있는 값을 바꿀때>

```rust
 let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
```

점(.)으로 특정필드에 새 값을 할당할 수 있다

<구조체의 필드 변경하기>

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email: email,  //email
        username: username,  //username
        active: true,
        sign_in_count: 1,
    }
}
```

- 변수와 필드명이 같다면 (`emai: email`) --> `email`로 축약가능

&nbsp;

### 구조체 갱신법을 이용하여 기존 구조체 인스턴스로 새 구조체 인스턴스 생성하기

존재하는 인스턴스에서 기존 값의 대부분은 재사용하고, 몇몇 값만 바꿔 새로운 인스턴스 생성

```rust
let user2 = User {
    email: String::from("anotheremail@example.com"),
    username: String::from("anotherusername567"),
    active: user1.active,
    sign_in_count: user1.sign_in_count,
};
```

--> email, username은 새로 할당하고, 나머지 필드는 user1의 값을 그대로 사용

```rust
let user2 = User {
    email: String::from("anotheremail@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

--> email, username은 새로 할당하고, 나머지 필드는 user1의 값을 그대로 사용

&nbsp;

### 튜플 구조체 : 이름이 없고 필드마다 타입은 다르게 정의 가능

> #### 튜플 구조체
>
> 구조체 명을 통해 의미를 부여할 수 있으나, 필드의 타입만 정의할 수 있고 명명은 할 수 없다.

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

&nbsp;

# 5-2. 구조체를 이용한 예제 프로그램

### 튜플을 이용한 리팩터링

```rust
fn main() {
    let length1 = 50;
    let width1 = 30;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(length1, width1)
    );
}

fn area(length: u32, width: u32) -> u32 {
    length * width
}
```

길이와 너비를 함께 묶으면 더 읽기 쉽고 관리하기도 좋을것

```rust
fn main() {
    let rect1 = (50, 30);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

하지만 이렇게하면 length가 인덱스 0이고, width가 인덱스 1인걸 기억해야함 --> 의미부여

```rust
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1) //빌려줌
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.length * rectangle.width
}
```

- 함수 호출시에 `&`를 사용하여 main은 소유권을 유지

&nbsp;

### 파생 트레잇(derived traits)으로 유용한 기능 추가하기

구조체 내에 모든값을 보기위해 인스터스를 출력하기

```rust
#[derive(Debug)]
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!("rect1 is {:?}", rect1);
    // {:#?}을 사용하게 되면 구조체 내용을 정렬해준다
}
```

&nbsp;

# 5-3. 메소드 문법

fn(함수)와 비슷하지만 함수와는 달리 **구조체의 내용안에 정의되며(혹은 열거형이나 트레잇 객체)** 첫번째 파라미터가 언제나 self인데, 이는 메소드가 호출되고있는 구조체의 인스턴스를 나타냄

### 메소드 정의하기

rectangle인스턴스를 파라미터로 가지고있는 `area`함수대신 rectangle 구조체위에서 정의된 **area메소드사용**

```rust
#[derive(Debug)] //출력으로 사용할 수 있게
struct Rectangle { //구조체
    length: u32,
    width: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {// 구조체 위에 정의된 area메소드
        self.length * self.width
    }
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

만일 메소드가 동작하는 과정에서 메소드 호출에 사용된 인스턴스가 변하기를 원한다면 첫번째 파라미터로 `&mut self`를 썼을것이다. 하지만 그냥 `self`를 첫번째 파라미터로 사용하여 **소유권을가져오는 경우는 드물다**

> 하나의 impl 블록 내에 해당 타입의 인스턴스로 할 수 있는 모든 것을 모아두는것을 지향하는것 같다.

> 러스트는 자동 참조 및 역참조라는 기능이 있다.
> bject.something()이라고 메소드를 호출했을 때, 러스트는 자동적으로 &나 &mut, 혹은 \*을 붙여서 object가 해당 메소드의 시그니처와 맞도록 한다.
>
> ```rust
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 둘은 동일한 표현.

&nbsp;

### 더 많은 파리미터를 가진 메소드

rectangle의 인스턴스가 다른 rectangle인스턴스를 가져와 두번째 rectagle이 self내에 완전히 들어갈 수 있다면 ture

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.length * self.width
    }

    fn can_hold(&self, other: &Rectangle) -> bool { //다른 rectangle인스턴스도 가져옴
        self.length > other.length && self.width > other.width
    }
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };
    let rect2 = Rectangle { length: 40, width: 10 };
    let rect3 = Rectangle { length: 45, width: 60 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

```
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

&nbsp;

### 연관함수

`self`파라미터를 갖지 않는 함수도 `impl`내에 정의하는것이 허용된다.

- 새로운 구조체의 인스턴스를 반환해주는 생성자로서 자주 사용된다.
- 예 : 하나의 차원값 파라미터를 받아서 이를 길이와 너비 양쪽에 사용할 때 같은 값을 두번 명시해주는것보다 쉽다.

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { length: size, width: size }
    }
}
```

이 연관함수를 호출하기 위해서는 `Rectangle::square(3)`로 사용해야함
