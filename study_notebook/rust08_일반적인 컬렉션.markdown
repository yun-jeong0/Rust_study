# 일반적인 컬렉션

러스트의 표준라이브러리에는 컬렉션이라 불리는 여러 개의 매우 유용한 데이터 구조들이 포함되어 있다. 이 컬렉션들이 가리키고 있는 데이터들은 힙에 저장되는데, 이는 즉 데이터량이 컴파일 타임에 결정되지 않아도 되며 프로그램이 실행될 때 늘어나거나 줄어들 수 있다는 의미, 각각의 컬렉션 종류는 서로 다른 용량과 비용을 가지고 있으며, 여러분의 현재 상황에 따라 적절한 컬렉션을 선택해야한다.

# 8-1. Vector

배열과 비슷한데 크기가 자유롭게 변할 수 있는 **동적배열**이다.

여러개의 데이터를 하나의 변수에 담을 수 있다. 이는 동적배열구조이고, 백터는 같은 타입의 값만을 저장할 수 있다.

### 새 벡터 만들기

```rust
// i32 타입의 값을 가질 수 있는 비어있는 새 백터
let v: Vec<i32> = Vec::new();
```

- 지금은 벡터에 어떠한 값도 집어넣지 않았기때문에 타입명시를 해줬다

초기화시 값을 집어넣으면 저장하고자하는 값의 타입을 대부분 유추할 수 있다

```rust
let v = vec![1,2,3];
```

&nbsp;

### 벡터 갱신하기

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
```

- 변수가 담고있는 값을 변경시킬 수 있게하려면 `mut`로 해당 변수를 가변으로 만들어줘야함
- 데이터로부터 타입을 추론하므로 Vec\<i32>를 붙일 필요없다

&nbsp;

### 벡터 드롭

struct와 마찬가지로 벡터도 스코프 밖으로 벗어났을대 해제된다

```rust
{
    let v = vec![1,2,3];
    // v를 가지고 뭔가한다
} // <- v가 스코프 밖으로 벗어났고, 여기서 해제됩니다(전부다)
```

&nbsp;

### 벡터의 요소들 읽기

두가지 방법이다. 하나는 인덱스문법, 하나는 get메소드

```rust
let v = vec![1,2,3,4];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

- 벡터 인덱스도 0부터 시작함
  - 1. &와 []를 이용하여 참조자를 얻은 것
  - 2. get 함수에 인덱스를 파라미터로 넘겨서 Option<&T>를 얻은 것

이 둘은 벡터가 가지고있지 않은 인덱스 값을 사용하려했을때 다르게 행동한다.

```rust
let v = vec![1,2,3,4];

let does_not_exist = &v[100];    // Panic!
let dose_not_exist = v.get(100); // None반환
```

&nbsp;

### 벡터 내의 값들에 대한 반복처리

만일 벡터 내의 각 요소들을 차례대로 접근하고 싶다면, 하나의 값에 접근하기 위해 인덱스를 사용하는 것 보다는, 모든 요소들에 대해 반복처리를 할 수 있다.

```rust
// for 루프를 이용한 반복
let v = vec![1,2,3,4];

for i in &v {
    println!("{}", i);
}

// 만약 요소들을 변형시키기 원한다면 각 요소에대한 가변참조자로 사용가능
let mut v = vec![1,2,3,4];

for i in &mut v {
    *i += 50;
}
```

- 가변참조자가 참고하고 있는 값을 바꾸기 위해서, i에 += 연산자를 이용하기 전에 `역참조 연산자 (*)`를 사용하여 값을 얻어야 한다.

&nbsp;

### 열거형을 사용하여 여러 타입을 저장하기

처음에 벡터는 같은 타입을 가진 값들만 저장할 수 있다고했는데, 열거형을 사용하면 여러타입을 지정할 수있다.

```rust
// 열거형을 정의하여 벡터내에 다른타입의 데이터 담기
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Float(10.12),
    SpreadsheetCell::Text(String::from("hello")),
];
```

> 얼만큼에 힙메모리가 필요한지 알아야하기 때문에, 벡터내에 저장될 타입을 알아야한다.

---

# 8-2. Stirng

Stirng타입은 언어 기능 내에 구현된 것이 아니고 러스트의 표준 라이브러리를 통해 제공된다.

가변적이며, 소유권을 갖고 있고, `UTF-8`로 인코딩된 스트링 타입

### 새로운 스트링 생성하기

```rust
let mut s = String::new();
```

위는 데이터를 담아둘 수 있는 **빈 스트링을** 생성한다.

하지만 때로는 스트링에 담아 둘 초기값이 있을텐데, 이런경우는 `to_string`메소드를 사용한다. `Display` 트레잇이 구현된 어느곳에던 사용가능하다.

```rust
let data = "initial contents";

let s = data.to_string();

//directly
let s = "initial contents".to_string();
//same work with to_string
let s = String::from("initial contents");
```

- `to_string()`과 `String::from()`은 정확히 같은일을 한다.

&nbsp;

### 스트링 갱신하기

String은 크기가 커질 수 있으며 이것이 담고 있는 내용물은 Vec의 내용물과 마찬가지로 더 많은 데이터를 집어넣음으로써 변경될 수 있습니다. 추가적으로, + 연산자나 format! 매크로를 사용하여 편리하게 String 값들을 서로 접합(concatenation)할 수 있습니다.

#### push_str, push를 이용하여 스트링 추가하기

```rust
let mut s1 = String::from("foo");

// method1
s1.push_str("bar");

// method2
let s2 = "bar";
s1.push_str(&s2);
println!("{}", s2);
```

- method1같은 경우는 "bar"를 재사용할 수 없다
- method2같은 경우는 s2를 다시사용가능, push_str이 소유권을 뺏는것도 아니다.

```rust
// push사용
let mut s1 = String::from("lo");
s1.push('l');
```

`push`매소드는 한개의 글자를 파라미터로 받아서 추가한다.

#### + 연산자나 format! 매크로를 이용한 접합

스트링을 조합하자

첫번째 방법 : `+ 연산자`

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");

let s3 = s1 + &s2;
```

> s1은 소유권을 잃어버려 더이사 쓸 수 없게된다.

왜그럴까? `+연산자`는 `add메소드`를 사용하는데 이를 살펴보자

`fn add(self, s: &str) -> String {`

이 메소드는 두번째 인자로 문자열의 참조(&str)을 받게 되어있다. 이는 의도적인데

만약 참조를 사용하지 않는다면 원본문자열을 둘다 잃어버리는 꼴이기 때문..아주 제한적이 되어버린다.

**비유:**

비유를 들어보자면

s1은 사용할 종이이고, s2의 내용을 배껴서 s1에 적은 다음 이를 s3라고 칭하는 것.

s2는 종이를 빌려준거고 실제로 사용한 종이는 s1것 이다. 이 새롭게 쓰여진 종이는 s3에게 넘어가서 **더이상 s1은 사용할 수 없게된다.**

두번째 방법 :`format! 매크로`

이는 **모든 문자열의 소유권이 유지**되고, 더 복잡한 문자열 조합에 적합하다

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");

let s3 = format!("{}-{}", s1, s2);
```

println!과 똑같은 방식으로 작동하지만, 출력하지않고 String을 반환한다.

&nbsp;

### 스트링 내부의 인덱싱

> rust String은 인덱싱을 지원하지 않는다.

왜?

일단 string은 Vec<u8>을 감싼것임.

`let len = String::from("Hola").len();`이것의 값은 4, 이는 Hola를 저장하고있는 Vec이 4바이트 길이라는 것.

하지만 이것은?

`let len = String::from("Здравствуйте").len();`

이 길이는 12가 아닌 24. 저 언어의 유니코드 스칼라 값이 저장소의 2바이트를 차지하기 때문

결론 : **스트링의 바이트들 안의 인덱스는 유효한 유니코드 스칼라 값과 항상 대응되지는 않을 것**

---

# 8-3. hash map

`HashMap<K, V>` 타입은 `K 타입의 "키"`에 `V 타입의 "값"`을 매핑한 것을 저장

이 매핑은 당연히 hashing function을 통해 동작한다, hashing function은 이 키와 값을 메모리 어디에 저장할지 결정한다.

벡터를 이용하듯 인덱스를 사용하는 것이 아닌, 임의의 타입으로 된 키를 이용하여 데이터를 찾는다

&nbsp;

### 새로운 HashMap 생성하기

`new`: 생성, `insert`: 요소추가

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"),10);
scores.insert(String::from("Yellow"),50);
```

튜플의 벡터에 대해 collect 메소드를 사용하는 방법

`collect` : 데이터를 모아서 HashMap을 포함한 여러 컬렉션 타입으로 만들어준다.

각각 분리된 벡터에 데이터를 가지고있다면 zip 메소드를 이용하여 둘이 한쌍이 되도록 할 수 있다.

```rust
use std::collections::HashMap;

let teams = vec![String::from("Blue"),String::from("Yellow")];
let initial_scores = vec![10,50];

let scores: HashMap<_,_> = teams.iter().zip(initial_scores.iter()).collect();
// 키와 값의 키와 값의 타입에 대한 타입 파라미터에 대해서는 밑줄을 쓸 수 있으며
// 러스트는 벡터에 담긴 데이터의 타입에 기초하여 해쉬에 담길 타입을 추론할 수 있다.
```

&nbsp;

### HashMap과 소유권

String과 같이 소유된 값들에 대해서는, 값들이 이동되어 해쉬맵이 그 값들에 대한 소유자가 될 것이다.

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map insert(field_name, field_value);
```

`insert`를 호출하여 `field_name`과 `field_value`를 해쉬맵으로 이동시킨 후에는 더 이상 이 둘을 사용할 수 없다.

만약 해쉬맵에 값들의 참조자들을 삽입한다면, 이 값들은 해쉬맵으로 이동되지 않음. 하지만 참조자가 가리키고 있는 값은 해쉬맵이 유효할 때 까지 계속 유효해야한다.

&nbsp;

### HashMap 내의 값 접근하기

`get` : 값 접근하기

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"),10);
scores.insert(String::from("Yellow"),50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

socre는 블루팀과 연관된 값을 가지고 있을거고, 결과값은 some(&10) 일 것이다. 결과값은 `Option<&T>`를 반환하기 때문에 Some으로 감싸져있다
만일 해쉬맵 내에 해당 키에 대한 값이 없다면 get은 None을 반환한다. 이때는 Option처리를 해야할것.

for루프를 이용하여 각각의 키/값 쌍에 대한 반복작업 가능

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"),10);
scores.insert(String::from("Yellow"),50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

&nbsp;

### HashMap 갱신하기

- #### 값을 덮어쓰기
  해쉬맵에 키와 값을 삽입하고, 그 후 똑같은 키에 다른값을 삽입하면 새값으로 대체된다.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"),10);
scores.insert(String::from("Blue"),25); // 10을 대체
```

- #### 키에 할당된 값이 없을 경우에만 삽입하기

`entry` : 해당키가 있는지 없는지 확인, `or_insert` : 해당키가 존재할경우 값반환, 그렇지 않을겨우 새 값 삽입.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"),10);

scores.entry(String::from("Yellow")).or_insert(50);
// entry().or_insert()메서드는 값에 대한 가변 참조(&mut V)를 반환
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
// {"Yellow": 50, "Blue": 10}
```

- #### 예전 값을 기초로 값을 갱신하기

어떤 텍스트 내에서 각 단어가 몇번나왔는지 세는 코드

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1; // 값을 할당하기 위해 *를 사용하여 count를 역참조해야한다.
}

println!("{:?}", map);
// {"hello": 1, "world": 2, "wonderful": 1}
```
