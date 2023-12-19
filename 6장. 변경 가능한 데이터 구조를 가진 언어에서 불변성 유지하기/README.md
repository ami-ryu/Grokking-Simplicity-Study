# 목표

- **데이터가 바뀌지 않도록** 하기 위해 copy-on-write 를 적용한다.
- **배열과 객체를 데이터에 쓸 수 있는** copy-on-write 동작을 만든다.
- **깊이 중첩된 데이터**도 copy-on-write 가 동작하게 만든다.

# 5장에서…

- 배열을 복사하고 값을 바꾼 다음 리턴함으로써, 장바구니 동작 일부 (“제품 추가하기” 동작) 에 copy-on-write 원칙을 적용해보았다.
- 더 많은 함수에 copy-on-write 를 적용해보자.

### 구현해야할 동작들

- **장바구니에 대한 동작**

  1. 제품 개수 가져오기
  2. 제품 이름으로 제품 가져오기
  3. 제품 추가하기 (→ 5장에서 완료)
  4. 제품 이름으로 제품 빼기
  5. 제품 이름으로 제품 구매 수량 바꾸기
     (→ 중첩된 데이터에 대한 동작)

- **제품에 대한 동작**
  1. 가격 설정하기
  2. 가격 가져오기
  3. 이름 가져오기

### 중첩

- 정의: 데이터 구조 안에 데이터 구조가 있는 경우
- 예) 배열 안에 객체가 있는 경우 → 객체가 배열 안에 ‘중첩’ 되었다.

### 어떻게 중첩된 데이터에 대해 불변 동작을 구현할 수 있을까?

---

## 1. 동작을 읽기, 쓰기 로 분류하기

- 읽기
  - 데이터에서 정보를 가져온다.
  - 데이터를 바꾸지 않는다.
  - 인자에만 의존해 정보를 가져오는 읽기 동작은 ‘계산’ 이다.
- 쓰기

  - 데이터를 바꾼다.
  - 쓰기 동작은 **‘불변성’ 원칙에 따라** 구현해야 한다.
    (= copy-on-write)

- **장바구니에 대한 동작**

  1. 제품 개수 가져오기 → 읽기
  2. 제품 이름으로 제품 가져오기 → 읽기
  3. 제품 추가하기 (→ 5장에서 완료) → 쓰기
  4. 제품 이름으로 제품 빼기 → 쓰기
  5. 제품 이름으로 제품 구매 수량 바꾸기
     → 쓰기

- **제품에 대한 동작**
  1. 가격 설정하기 → 쓰기
  2. 가격 가져오기 → 읽기
  3. 이름 가져오기 → 읽기

### 불변성 원칙

- copy-on-write
- JavaScript 는 기본적으로 변경가능한 데이터 구조를 사용하기 때문에, 불변성 원칙은 직접 구현해야한다.
  - 불변형 데이터 구조는 함수형 프로그래밍 언어의 일반적인 기능이지만, 언어에서 지원하지 않는 경우가 많다.
- 불변성 데이터 구조를 기본으로 지원하는 언어들
  - Haskell, Clojure, Elm, PureScript, Erlang, Elixir

## 2. copy-on-write 원칙 3단계

1. 복사본 만들기
2. 복사본 변경하기 (원하는만큼)
3. 복사본 리턴하기

### 지난 장에서.. 우리는 쓰기를 “읽기” 로 바꾸었다.

```jsx
function add_element_last(array, elem) {
  const new_array = array.slice();
  new_array.push(elem);
  return new_array;
}
```

copy-on-write 는 쓰기를 “읽기” 로 바꿉니다.

### “제품 이름으로 제품 빼기” 함수에 copy-on-write 적용해보기

as-is

```jsx
function remove_item_by_name(cart, name) {
  let idx = null;
  for (let i = 0; i < cart.length; i++) {
    if (cart[i].name == name) idx = i;
  }

  if (idx !== null) cart.splice(idx, 1); // idx 위치에 있는 항목을 1개 삭제하는 구문
}
```

to-be

```jsx
function remove_item_by_name(cart, name) {
  let new_cart = cart.slice(); // 1. 복사본 만들기 (cart 를 복사해 지역변수에 저장)
  let idx = null;

  for (let i = 0; i < new_cart.length; i++) {
    if (new_cart[i].name == name) idx = i;
  }

  if (idx !== null) new_cart.splice(idx, 1); // 2. 복사본 변경하기

  return new_cart; // 3. 복사본 리턴하기
}
```

### splice 함수를 copy-on-write 함수로 일반화 하기

```jsx
function removeItems(array, idx, count) {
	let copy = array.slice();
	copy.splice(idx, count);
	return copy;
}

function remove_item_by_name(cart, name) {
	...
	if (idx !== null) return removeItems(cart, idx, 1);
	else return cart; // 이렇게 구현하면 값을 바꾸지 않ㅇ면, 복사하지 않아도 된다는 추가 이점이 있다.
}
```

### JavaScript 배열

- `arr[idx]`: 인덱스로 값 찾기
- `unshift(el)`: 배열 맨 앞에 el 을 추가하고 (원본을 수정함) 새로운 길이의 배열을 리턴한다.
- `shift()`: 배열 맨 앞 값을 지우고 (원본을 수정함) 지워진 배열을 리턴한다.
- `slice()`: 배열을 얕게 복사해서 새로운 배열을 리턴한다.
  - 얕은 복사:
    객체의 **얕은 복사**는 복사본의 속성이 복사본이 만들어진 원본 객체와 같은 참조 (메모리 내의 같은 값을 가리킴)를 공유하는 복사.
- `splice(idx, num)`: idx 위치에서 num개 항목을 지우고 (원본을 수정함) 지워진 배열을 리턴한다.

## 3. 읽기 & 쓰기를 동시에 하는 함수를 copy-on-write 로 바꾸기

두가지 방법이 있다.

1. 함수를 분리하기
2. 값을 두 개 리턴하기

### 1. “함수를 분리하기” 방법

1. 먼저, 쓰기에서 읽기를 분리하기
2. 쓰기에 copy-on-write 적용해서 읽기로 바꾸기

```jsx
// 단순히 첫번째 항목을 읽어 리턴하는 함수 = "계산"
function first_element(array) {
  return array[0];
}

// 쓰기 동작 shift 를 copy-on-write 함수로 바꾸기
function drop_first(array) {
  const array_copy = array.slice();
  array_copy.shift();
  return array_copy;
}

// 위의 두 함수 조합하기
function shift(array) {
  return {
    first: first_element(array),
    array: drop_first(array),
  };
}
```

### 2. “값을 두개 리턴하기” 방법

1. 동작을 감싸기
2. 읽으면서 쓰기도 하는 함수를 → 읽기 함수로 바꾸기

```jsx
function shift(array) {
  const array_copy = array.slice();
  let first = array_copy.shift();
  return {
    first,
    array: array_copy,
  };
}
```

### 액션, 계산, 데이터

- 변경 가능한 데이터를 읽는 것은 “액션” 이다.
- 쓰기는 데이터를 바꾸기 때문에, 데이터를 변경 가능한 구조로 만든다.
- 어떤 데이터에 쓰기를 모두 없앴다면, 그 데이터는 ‘불변’ 데이터이다.
- 불변 데이터 구조를 읽는 것은 “계산” 이다.
- **데이터 구조를 불변형으로 만들수록, 코드에 더 많은 계산이 생기고 액션을 줄어든다.**

---

# 시간에 따라 바꾸는 값 다루기

데이터가 모두 불변형이더라도, **시간에 따라 변하는 상태를 다룰 수 있어야 한다.**

ex) 장바구니에 제품 넣기

우리가 다룬 `shopping_cart` 전역변수가 이에 해당한다.

<교체>

1. 읽기
2. 바꾸기
3. 쓰기

```jsx
shopping_cart = add_item(shopping_cart, shoes);
3. 쓰기          2. 바꾸기  1. 읽기
```

shopping_cart 전역변수는 필요할 때, 새로운 값으로 교체되어 항상 최신값을 나타낸다.

“교체” 는 함수형 프로그래밍에서 일반적으로 사용하는 방법이다.

---

# 불변 데이터 구조의 장단점

### 바뀔 때마다 복사하는 것은 비효율적이지 않을까?

### 단점

- 불변데이터 구조는 변경가능한 데이터 구조보다 메모리를 더 많이 쓰고 느리다.

### 하지만,

- 가비지 콜렉터는 충분히 빠르다.
- “얕은 복사” 를 사용하면, 같은 메모리를 가리키는 참조에 대한 복사본을 만들기 때문에,
  생각보다 그리 많은 것을 복사하진 않는다.
- 몇몇 함수형 프로그래밍 언어에서는 기본으로 불변 데이터 구조를 지원하므로,
  데이터 구조를 복사할 때 최대한 많은 구조를 공유하게해,
  더 적은 메모리를 사용하고 가비지 콜렉터의 부담을 줄여준다.
  ex) Clojure

---

# “객체” 에 대한 copy-on-write

위에서는 “배열” 에 대한 copy-on-write 를 다루었다. 이제 객체에 대해 다루어보자!

원칙은 동일하다.

**<copy-on-write 원칙 3단계>**

1. 복사본 만들기
2. 복사본 변경하기 (원하는만큼)
3. 복사본 리턴하기

### JavaScript 객체 연산

- `Object.assign(a, b)`: 객체 복사
  - b 객체의 모든 key를 a 객체로 복사한다.
  - 빈 객체에 모든 key/value 쌍을 복사해서 b 의 복사본을 만든다.
- `Object.keys()`: key 목록 가져오기
- `delete`: key/value 쌍 지우기

```jsx
// 원래 코드
function setPrice(item, new_price) {
  item.price = new_price;
}

// copy-on-write 적용
function setPrice(item, new_price) {
  const item_copy = Object.assign({}, item); // 복사
  item_copy.price = new_price; // 변경
  return item_copy; // 리턴
}
```

### 중첩된 데이터 구조에서 쓰기 → 읽기로 바꾸기

```jsx
function setPriceByName(cart, name, price) {
  const cartCopy = cart.slice();

  for (let i = 0; i < cartCopy.length; i++) {
    if (cartCopy[i].name === name) {
      cartCopy[i] = setPrice(cartCopy[i], price);
      // 중첩된 항목을 바꾸기 위해 copy-on-write 동작을 부른다.
    }
  }

  return cartCopy;
}
```

위 코드에서 4개 데이터 (배열 1, 객체 3) 에서 두개의 데이터만 복사되었다. (배열 1, 객체 1)

나머지 객체는 변경하지도 복사하지도 않았고,

원래 배열과 복사한 배열 모두 바뀌지 않은 객체를 가리킨다.

**→ “구조적 공유”**

# 결론

- JavaScript 에서는 copy-on-write 원칙을 직접 구현해야하고, 유틸리티 함수로 만들어 나중에 편리하게 쓸 수 있게 해야한다.
- 함수형 프로그래밍에는 “불변 데이터” 가 필요하다.
  copy-on-write 는 데이터를 불변형으로 유지하는 원칙이다.
- 복사하고, 변경하고, 리턴한다.

### 다음 장에서는…

- 방어적 복사 (Defensive Copy) 원칙
