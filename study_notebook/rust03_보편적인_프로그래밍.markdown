# 보편적인 프로그래밍 개념

## #변수와 가변성

- 기본변수는 **불변성**이다.

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

위와같이 변경이 불가하다.

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

하지만 `mut`키워드를 사용하면 값을 변경할 수 있다.

&nbsp;

### 변수와 상수간의 차이점들

- 상수는 `mut`사용이 허용되지 않는다. 불변성 그 자체

- 상수를 사용할때는 `const`를 사용해야한다.

## #Shadowing

- 이전에 선언한 변수와 같은 이름의 새 변수를 선언할 수 있다

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);
}
```

이는 mut로 선언하는것과는 차이가 있다. 우리가 몇번 값을 변경할 수는 있지만 그 이후에 변수는 불변성을 가지게 된다

---

---

## 3.2 데이터 타입들(스칼라/컴파운드)

### 스칼라 타입

- 정수형
- 부동 소수점 타입
- bool타입
- 문자타입
- 복합타입들

  - 튜플
  - 배열

  ***

# 3.3 함수 동작 원리

### 함수 매개변수

```rust
fn main() {
    another_function(5, 6);
}

// 각각 i32타입인 두개의 매개변수를 갖는 함수
fn another_function(x: i32, y: i32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```

&nbsp;

### 반환값을 갖는 함수

```rust
fn five() -> i32 {
    5 //반환값(; 이 없음)
}

fn main() {
    //구문
    let x = five();

    println!("The value of x is: {}", x);
}
```

- 반환값은 뒤에 `;`세미콜론이 붙지 않는다.

---

# 3.5 제어문

### if 표현식

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

&nbsp;

### let구문에서 if사용하기

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

- if에 속한 각 갈래의 결과는 반드시 같은 타입이어야한다

### 반복문과 반복

#### loop

#### while

```rust
fn main() {
    //값이 계속 변하니까 "mut"선언
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```

#### for

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

- for 반복문을 사용해 콜렉션의 각 요소를 순회하기
