# 4-1. 소유권(Ownership)

- 메모리는 컴파일 타임에 컴파일러가 체크할 규칙들로 구성된 소유권 시스템을 통해 관리된다

---

## 스택(stack)

- 새로운 데이터를 넣어두기 위한 공간, 혹은 데이터를 가져올 공간을 검색할 필요가 없다. --> 항상 공간이 top에 있기 때문에
- 스택에 담긴 모든 데이터가 결정되어 있는 고정된 크기를 갖고 있어야 한다.

## 힙(heap)

- 컴파일 타임에 크기가 결정되어 있지 않거나 크기가 변경될 수 있는 데이터를 위해서는 힙을 사용한다.

---

### 소유권 규칙

> 1. 러스트 각각의 값은 해당값의 오너(owner)라고 불리우는 변수를 가지고 있다.
>
> 2. 한번에 딱 하나의 오너만 존재할 수 있다.
>
> 3. 오너가 스코프 밖으로 벗어나는 때, 값은 버려진다(dropped).

### 변수의 스코프

- 스코프 : 프로그램 내에서 아이템이 유효함을 표시하기 위한 범위 -> `{}`

```rust
// example
{                      // s는 유효하지 않습니다. 아직 선언이 안됐거든요.
    let s = "hello";   // s는 이 지점부터 유효합니다.

    // s를 가지고 뭔가 합니다.
}                      // 이 스코프는 이제 끝이므로, s는 더이상 유효하지 않습니다.
```

- 스코프 안에서 `s`가 등장하면 유효하다
- 이 유효기간은 스코프 밖으로 벗어날때까지 지속된다

### String타입

힙에 저장되는 데이터를 관찰하고 러스트는 과연 어떻게 이 데이터를 비워낼까

```rust
let s = String::from("hello");
```

- `::`은 우리가 String타입 아래 from함수를 특정지을 수 있도록 해주는 네임스페이스 연산자.
- `String::from`을 호출하면, 구현부분에서 **필요한 만큼의 메모리를 요청**

```rust
{
    let s = String::from("hello"); // s는 여기서부터 유효합니다

    // s를 가지고 뭔가 합니다
}                                  // 이 스코프는 끝났고, s는 더 이상
                                   // 유효하지 않습니다
```

- 메모리는 변수가 소속되어있는 스코프 밖으로 벗어나는 순간 자동으로 반납된다. : `drop`
- 괄호가 닫힐때 자동적으로 `drop`을 호출하여 메모리를 반환한다.

---

### 변수와 데이터가 상호작용하는 방법 : 이동(move)

```rust
let s1 = String::from("hello");
let s2 = s1;
```

<img src="../img/04/04_1.png" width="50%" />

- `String`은 그림의 왼쪽과 같이 세개의 부분으로 이루어진다(메모리의 포인터, 길이, 용량)

- 데이터의 그룹은 스택에 저장되고, 내용물을 담을 오른쪽은 힙메모리에 있는다.

<img src="../img/04/04_2.png" width="50%" />

- S2에 S1을 대입했을때 이는 스택에 있는 포인터, 길이값, 용량값이 복사된다는 의미

- 포인터가 가리키고있는 힙메모리상의 데이터는 복사되지 않는다

### 두번해제 오류

- 두 데이터가 모두 같은 곳을 가리키고있기 때문에 `s1`과 `s2`이 스코프 밖으로 벗어나게 되면, 둘 다 같은 메모리를 해제하려할것임

- Rust : 할당된 메모리를 복사하는 것을 시도하는 대신, 러스트에서는 `S1`이 더이상 유효하지 않다고 간주(s1 무효화)

  -> `s1`이 스코프밖을 벗어났을때 아무것도 해제할 필요가 없어진다.

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("{}, world!", s1);
```

실제로 사용하려고하면 오류가 뜬다.

<img src="../img/04/04_3.png" width="50%" />

### 변수와 데이터가 상호작용하는 방법 : 클론

힙데이터까지 복사하기를 원할때

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

### 스택에만 있는 데이터 : 복사

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

- 정수형과 같이 컴파일 타임에 결정되어 있는 크기의 타입은 스택에 모두 저장 --> 실제 값의 복사본이 빠르게 만들어질 수 있다.

- Copy 트레잇이라 불리는 특별한 어노테이션 --> 어떤 타입이 Copy 트레잇을 가지고있다면 대입과정 후에도 예전변수를 사용할 수 있다.

- 만약 그 타입이 Drop트레인을 구현한 것이 있다면, Copy트레잇을 어노테이션 할 수 없다.

Copy가 가능한 타입

> 1. u32와 같은 모든 정수형 타입들
>
> 2. true와 false값을 갖는 부울린 타입 bool
>
> 3. f64와 같은 모든 부동 소수점 타입들
>
> 4. Copy가 가능한 타입만으로 구성된 튜플들. (i32, i32)는 Copy가 되지만, (i32, String)은 안된다

---

### 소유권과 함수

```rust
fn main() {
    let s = String::from("hello");  // s가 스코프 안으로 들어왔다

    takes_ownership(s);             // s의 값이 함수 안으로 이동...
                                    // ... 그리고 이제 더이상 유효하지 않다
    let x = 5;                      // x가 스코프 안으로 들어왔다

    makes_copy(x);                  // x가 함수 안으로 이동했습니다만,
                                    // i32는 Copy가 되므로, x를 이후에 계속
                                    // 사용해도 됩니다.

} // 여기서 x는 스코프 밖으로 나가고, s도 그 후 나갑니다. 하지만 s는 이미 이동되었으므로,
  // 별다른 일이 발생하지 않습니다.

fn takes_ownership(some_string: String) { // some_string이 스코프 안으로 들어왔습니다.
    println!("{}", some_string);
} // 여기서 some_string이 스코프 밖으로 벗어났고 `drop`이 호출됩니다. 메모리는
  // 해제되었습니다.

fn makes_copy(some_integer: i32) { // some_integer이 스코프 안으로 들어왔습니다.
    println!("{}", some_integer);
} // 여기서 some_integer가 스코프 밖으로 벗어났습니다. 별다른 일은 발생하지 않습니다.
```

만약 `s`를 `takes_ownership` 함수를 호출한 이후에 사용하려고 한다면, 러스트는 컴파일 타임오류를 낼것이다

- s를 함수로 전달하면, 소유권이 함수로 이동(move)된다. 원래 변수는 해당값을 사용할 수 없음
- 기본타입들은 `copy`트레잇을 사용하고있어, **값이 복사**되어 전달된다.

### 반환 값과 스코프

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership은 반환값을 s1에게
                                        // 이동시킵니다.

    let s2 = String::from("hello");     // s2가 스코프 안에 들어왔습니다.

    let s3 = takes_and_gives_back(s2);  // s2는 takes_and_gives_back 안으로
                                        // 이동되었고, 이 함수가 반환값을 s3으로도
                                        // 이동시켰습니다.

} // 여기서 s3는 스코프 밖으로 벗어났으며 drop이 호출됩니다. s2는 스코프 밖으로
  // 벗어났지만 이동되었으므로 아무 일도 일어나지 않습니다. s1은 스코프 밖으로
  // 벗어나서 drop이 호출됩니다.

fn gives_ownership() -> String {             // gives_ownership 함수가 반환 값을
                                             // 호출한 쪽으로 이동시킵니다.

    let some_string = String::from("hello"); // some_string이 스코프 안에 들어왔습니다.

    some_string                              // some_string이 반환되고, 호출한 쪽의
                                             // 함수로 이동됩니다.
}

// takes_and_gives_back 함수는 String을 하나 받아서 다른 하나를 반환합니다.
fn takes_and_gives_back(a_string: String) -> String { // a_string이 스코프
                                                      // 안으로 들어왔습니다.

    a_string  // a_string은 반환되고, 호출한 쪽의 함수로 이동됩니다.
}
```

- 변수의 소유권은 모든 순간 똑같은 패턴을 따른다 => **값을 다른 변수에 대입하면 값이 이동된다**

---

---

# 4-2. 참조자(reference)와 빌림(borrowing)

매번 소유권을 넘기고 싶지 않다. 값만 사용할 수 있게 하고싶다면?

> 함수에 `&변수`를 넘기고, 이는 어떤 값을 소유권을 넘기지않고 **참조**할 수 있게 해준다

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {// s는 String의 참조자입니다
    s.len()// 여기서 s는 스코프 밖으로 벗어났습니다. 하지만 가리키고 있는 값에 대한 소유권이 없기
  // 때문에, 아무런 일도 발생하지 않습니다.
}
```

함수에 `&s1`를 넘기고, 함수의 정의 부분에는 String이 아니라 `&String`을 이용했다

- usize : Rust에서 크기/길이를 나타내는 데 사용되는 특별한 정수 타입(음수가 될 수 없다)

<img src="../img/04/04_4.png" width="60%" />Stirng s1을 가리키고있는 &String s

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

```
error: cannot borrow immutable borrowed content `*some_string` as mutable
 --> error.rs:8:5
  |
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^
```

변수가 기본적으로 불변인것 처럼, 참조자도 마찬가지 우리가 빌린 무언가를 변경하는건 허용되지 않는다

### 가변 참조자(mutable References)

위의 코드를 살짝만 바꾸면 오류를 고칠 수 있다

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

- `s`를 `mut`로 바꾼다
- `&mut s`로 가변 참조자를 생성하고 `some_string: &mut String`으로 이 가변 참조자를 받아야 한다
- **제한 : 특정한 스코프 내에 특정한 데이터 조각에 대한 가변 참조자를 딱 하나만 만들 수 있다**

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;
```

```
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> borrow_twice.rs:5:19
  |
4 |     let r1 = &mut s;
  |                   - first mutable borrow occurs here
5 |     let r2 = &mut s;
  |                   ^ second mutable borrow occurs here
6 | }
  | - first borrow ends here
```

&nbsp;

아주 통제된 형식이다. 하지만 이러한 제한이 가지는 이점은 러스트가 컴파일 타임에 *데이터레이스(data race)*를 방지할 수 있도록 해준다는 것이다.

> #### data race
>
> 아래 정리된 세가지 동작이 발생했을때 나타나는 특정한 레이스 조건
>
> 1. 두개 이상의 포인터가 동시에 같은 데이터에 접근한다.
> 2. 그 중 적어도 하나의 포인터가 데이터를 쓴다.
> 3. 누가 먼저 처리되어야하는지에 대한 규칙(동기화)가 없다.
>
> ```rust
> let mut s = String::from("hello");
>
> let r1 = &mut s;  //ok, 첫번째 가변참조
> let r2 = &mut s;  //에러!, 이미 가변참조 있음
> ```

그래서 스코프가 쓰이는데 이는 동시에 만드는것이 아니게 해줌으로서, 여러개의 가변참조자를 만들 수 있도록 해준다

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // 여기서 r1은 스코프 밖으로 벗어났으므로, 우리는 아무 문제 없이 새로운 참조자를 만들 수 있습니다.

let r2 = &mut s;
```

가변참조자와 불변참조자를 혼용할 경우에도 오류가 발생

```rust
let mut s = String::from("hello");

let r1 = &s; // 문제 없음
let r2 = &s; // 문제 없음
let r3 = &mut s; // 큰 문제
```

```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as
immutable
 --> borrow_thrice.rs:6:19
  |
4 |     let r1 = &s; // 문제 없음
  |               - immutable borrow occurs here
5 |     let r2 = &s; // 문제 없음
6 |     let r3 = &mut s; // 큰 문제
  |                   ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

불변참조자는 사용중에 값이 갑자기 바뀔거라고 생각하지 않는다. 그래서 여러개의 불변참조자를 만들 수 있지만(그냥 읽기만하는거니까) 가변참조자를 다시 선언하면 안된다.

&nbsp;

### 댕글링 참조자(Dangling References)

> #### 댕글링 포인터(Dangling Pointer)
>
> 해제한 메모리를 참조하고있는 포인터

Rust는 컴파일러가 참조자들이 댕글링참조자가 되지 않도록 보장해준다
만약, 우리가 어떤 데이터의 참조자를 만들었다면, 컴파일러는 그 참조자가 스코프 밖으로 벗어나기 전에는 데이터가 스코프 밖으로 벗어나지 않을 것임을 확인해 줄 것.

```rust
//dangle코드

fn dangle() -> &String { // dangle은 String의 참조자를 반환합니다

    let s = String::from("hello"); // s는 새로운 String입니다

    &s // 우리는 String s의 참조자를 반환합니다.
} // 여기서 s는 스코프를 벗어나고 버려집니다. 이것의 메모리는 사라집니다.
  // 위험하군요!
```

dangle의 코드가 끝이나면 s는 할당해제가 된다. 하지만 우리는 s의 참조자를 반환하려고 했기때문에(소유권을 주는것이 아닌 값만 사용할 수 있도록 빌려주는것임) 댕글링참조자가 되는 것이다.

```rust
// 해법
fn no_dangle() -> String {
    let s = String::from("hello");

    s //소유권을 밖으로 이동시킨다
}
```

---

---

# 4-3. 슬라이스(slice)

소유권을 갖지 않는 또다른 데이터타입, 슬라이스는 컬렉션의 연속된 일련의 요소들을 참조할 수 있게 함

### 스트링 슬라이스

Stirng의 일부분에 대한 참조자.

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

[starting_index..ending_index] :

- starting_index : 슬라이스의 첫번째 위치 인덱스
- ending_index : 슬라이스의 포함된 마지막 위치보다 1을 더한 값

<img src="../img/04/04_5.png" width="50%" />

슬라이스 데이터 구조 : 포인터, 길이값을 갖고 있다

```rust
let s = String::from("hello");

let slice = &s[..2]; //처음부터
let slice = &s[3..]; //마지막까지
```

처음부터 마지막까지 참조하려면 위와같은 방식도 가능

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

- `&str` : 스트링 슬라이스를 나타내는 타입
- 공백을 발견하면 그 앞까지 리턴하는 함수

#### 스트링 리터럴은 슬라이스이다

```rust
let s = "Hello, world!";
```

여기서 `s`의 타입은 `&str` 바이너리의 특정지점을 가리키고 있는 슬라이스이다.

```rust
fn first_word(s: &str) -> &str {
```

`&String`과 `&str` 둘 모두에 대한 같은 함수를 사용할 수 있도록 해주기 때문
, 함수가 Stirng의 참조자 대신, 스트링슬라이스를 갖도록 정의하는것은 더 유용함.
