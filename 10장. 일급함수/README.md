<aside>
📖 **목차**

</aside>

# 알아볼 내용

- 일급값의 장점
- 일급 함수로 만드는 방법
- 고차 함수로 문법을 감싸는 방법
- 일급 함수와 고차 함수를 사용한 리팩토링

---

### 일급값

- 변수에 할당될 수 있고, 자료구조에 저장될 수 있으며, 함수의 인자나 반환값이 될 수 있는 값이나 객체

### 일급 함수

- 변수나 자료구조에 저장할 수 있고, 함수의 인자나 반환값이 될 수 있는 함수

### 고차 함수

- 함수를 인자로 받거나 반환하는 함수

### 일급함수와 고차함수를 사용한 “추상화를 잘할 수 있는 리팩토링”

리팩토링 1: 암묵적 인자를 드러내기

- 목표: 함수 이름에 있는 암묵적 인자 → 명시적인 함수 인자로 바꾸기
- 방법: 암묵적 인자가 일급 값이 되도록 함수에 인자를 추가한다.
- 장점: 잠재적 중복을 없애고 코드의 목적을 잘 표현할 수 있다.

리팩토링 2: 함수 본문을 콜백으로 바꾸기

- 목표: 원래 있던 코드를 고차 함수로 만들기
- 방법: 함수 본문에 어떤 부분을 콜백으로 바꾼다.
- 장점: 일급 함수로 어떤 함수에 동작을 전달할 수 있다.

위 내용을 실제 요구사항에 적용해봅시다.

---

# 실제 요구사항에 적용해보기

### 신규 요구사항

- 장바구니 제품 **값** 설정 기능
- 장바구니 제품 **개수** 설정 기능
- 장바구니 제품 **배송** 설정 기능
- 장바구니 제품 **세금** 설정 기능

### 위 요구사항을 구현하기 위한 기존 코드

- setPriceByName(cart, name, price)
- setQuantityByName(cart, name, quant)
- setShippingByName(cart, name, ship)
- setTaxByName(cart, name, tax)

위 코드들은 냄새로 가득하다 !

### 문제점

1. 비슷한 기능을 하는 함수들의 “중복” 냄새
2. 필드를 결정하는 문자열이 함수 이름에 있다는 점
   → “함수 이름에 있는 암묵적 인자” 냄새
   값을 명시적으로 전달하지 않고, 함수 이름의 일부로 전달하고 있다는 문제점

---

# 리팩토링 시~작

## 리팩토링 1. 암묵적 인자를 드러내기

### 리팩토링 전

```jsx
function setPriceByName(cart, name, price) {
  var item = cart[name];
  var newItem = objectSet(item, 'price', price);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}
```

**문제점**

1. 함수 이름에 있는 “Price” 는 암묵적 인자
2. 역할이 중복되는 비슷한 함수들을 여러개 생성해야함

### 리팩토링 후

```jsx
function setFieldByName(cart, name, field, value) {
	vart item = cart[name];
	var newItem = objectSet(item, field, value);
	var newCart = objectSet(cart, name, newItem);
	return newCart;
}
```

**해결**

- 함수명 “setFieldByName” 으로 교체, `field` `value` **명시적인 인자**를 추가
- 호출부에서 **하나의 함수를 재사용**
  ```jsx
  cart = setFieldByName(cart, 'shoe', 'price', 13);
  cart = setFieldByName(cart, 'shoe', 'quantity', 3);
  cart = setFieldByName(cart, 'shoe', 'shipping', 0);
  cart = setFieldByName(cart, 'shoe', 'tax', 2.34);
  ```

## 일급인 것과 아닌 것 구별하기

### JavaScript 에서 일급인 것 vs 아닌것

- 일급값: 숫자, 문자열, Boolean, 배열, 객체, 함수
- 일급값이 아닌것: 수식 연산자 (+, -), 반복문, 조건문, try/catch 블록

**일급값이 아닌것을 일급값으로 바꾸는 것이 중요하다.**

ex) JavaScript 에서 함수명 일부를 값처럼 쓸 수 있는 방법은 없다.
함수명은 일급이 아니므로, 함수명의 일부를 인자로 바꿔 일급으로 만든다.

### field 명을 문자열로 사용하면 버그가 생기지 않을까?

- 해결법: field 명이 일급이기 때문에, 객체와 배열에 담기

```jsx
var validItemFields = ['price', 'quantity', 'shipping', 'tax', 'number'];
var traslations = { 'quantity': 'number' };

function setFieldByName(cart, name, field, value) {
	if(!validItemFields.includes(field))
		throw "Not a valid item field: '" + field + "'.";
	if(translations.hasOwnProperty(field))
		field = translations[field];
	...
}
```

- 장점
  - 필드명 문자열에 오타가 있는 경우 방지
  - 일부 필드명이 변경된 경우, 추상화 벽 위에서 기존 필드명을 유지한 채 코드 내부에서 간단히 바꿀 수 있다.

### 데이터 지향

- 데이터 지향: 이벤트와 엔티티에 대한 사실을 표현하기 위해, 일반 데이터 구조를 사용하는 프로그래밍 형식
  - ‘일반 데이터 구조’: 일반적이고 재사용할 수 있는 객체 or 배열

![스크린샷 2024-02-02 오전 11.40.42.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/016f3885-02a2-4cff-ab02-0eab67b7347c/545b36de-a5f5-4d6c-a55f-6f150f7019ad/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-02-02_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_11.40.42.png)

### 정적 타입 vs 동적 타입

- 정적 타입: 컴파일 할 때 타입을 검사
- 동적 타입: 런타임에 타입을 검사

정적 타입, 동적 타입 모두 타당한 이유와 장점이 있다. 팀에서 선택한 언어를 믿고 사용하자!

### 어떤 문법이든 일급 함수로 바꿀 수 있다 !

JavaScript 에서 일급이 아닌 것들을 일급 함수로 바꿀 수 있다!

ex) + 연산자는 일급이 아니기에, plus 라는 함수를 만든다.

```jsx
function plus(a, b) {
  return a + b;
}
```

## 반복문 리팩토링 해보기

### 예시: 배열을 순회하는 두가지 반복문

1. 준비하고 먹기

```jsx
for (var i = 0; i < foods.length; i++) {
  var food = foods[i];
  cook(food);
  eat(food);
}
```

1. 설거지하기

```jsx
for (var i = 0; i < dishes.length; i++) {
  var dish = dishes[i];
  wash(dish);
  dry(dish);
  putAway(dish);
}
```

코드는 비슷하나 반복문이 하는 일은 다르다.

하지만 두 반복문에서 최대한 문법적으로 비슷한 부분을 찾아 하나로 만들어보자.

- 두 코드의 비슷한 점: 받은 배열 인자를 순회하며, 반복문 안에서 인덱스로 접근해 하나씩 값을 가져오는 부분이 같다.

### 다음 리팩토링에 앞서.. 반복문에서 암묵적 인자 제거하기

1. 각 반복문을 함수로 감싸고, 직관적인 함수명으로 이름짓기
2. 재사용할 함수로 추출해낼 부분에서 쓰이는 변수명을 더 일반적인 이름으로 바꾸기

   ```jsx
   // as-is
   var food = foods[i];
   var dish = dishes[i];

   // to-be
   var item = array[i];
   ```

3. 함수명을 일반적인 이름으로 바꾸고, 명시적인 배열을 인자로 추가하기

   ```jsx
   // as-is
   function cookAndEatFoods() { ... }
   function cleanDishes() { ... }

   // to-be
   function cookAndEatArray(array) { ... }
   cookAndEatArray(foods);

   function cleanArray(array) { ... }
   cleanArray(dishes);
   ```

4. 반복문 안에 있는 본문을 분리하기

   ```jsx
   function cookAndEatArray(array) {
     for (var i = 0; i < array.length; i++) {
       var item = array[i];
       cookAndEat(item);
     }
   }

   function cookAndEat(food) {
     cook(food);
     eat(food);
   }

   cookAndEatArray(foodS);
   ```

   ```jsx
   function cleanArray(array) {
     for (var i = 0; i < array.length; i++) {
       var item = array[i];
       clean(item);
     }
   }

   function clean(dish) {
     wash(dish);
     dry(dish);
     putAway(dish);
   }

   cleanArray(dishes);
   ```

### 리팩토링 시~작

1. 함수 이름 일반적인 이름으로 바꾸기

   ```jsx
   // as-is
   function cookAndEatArray(array) { ... }
   function cleanArray(array) { ... }

   // to-be
   function operateOnArray(array, f) {
   	for (var i = 0; i < array.length; i++) {
   		var item = array[i];
   		f(item);
   	}
   }
   ```

2. 위 함수를 공통으로 사용하기

   - 일반적으로 이런 기능을 하는 함수를 forEach 라 부르므로 이름도 바꾸어본다.
   - forEach 함수는 인자로 함수를 받으므로 '고차함수' 이다.

   ```jsx
   function forEach(array, f) {
   	for (var i = 0; i < array.length; i++) {
   		var item = array[i];
   		f(item);
   	}
   }

   function cookAndEat(food) {
   	...
   }

   function clean(dish) {
   	...
   }

   forEach(foods, cookAndEat);
   forEach(dishes, clean);
   ```

## 리팩토링 단계 요약

1. 코드를 함수로 감싸기
2. 더 일반적인 이름으로 바꾸기
3. 암묵적 인자를 드러내기
4. 함수 추출하기
5. 암묵적 인자를 드러내기

너무 많은 단계들이 있다.

하나로 합치면 좋을것 같은데, 이런 리팩토링을 “함수 본문을 콜백으로 바꾸기” 라고 한다.

## 리팩토링 2. 함수 본문을 콜백으로 바꾸기

코드 본문의 앞부분, 뒷부분의 패턴을 찾으면 함수 본문을 콜백으로 바꾸기에 다가갈 수 있습니다.

```jsx
코드 1.
try { // 앞부분
	saveUserData(user); // 본문
} catch (error) { // 뒷부분
	logToSnapErrors(error);
}

코드 2.
try { // 앞부분
	fetchProduct(productId); // 본문
} catch (error) { // 뒷부분
	logToSnapErrors(error);
}
```

앞부분, 뒷부분은 두 코드가 중복이다. 다른 부분에 ‘구멍’ 을 만들어 코드를 넣자!

일단 함수로 빼내기

```jsx
function withLogging() {
  try {
    saveUserData(user);
  } catch (error) {
    logToSnapErrors(error);
  }
}

withLogging();
```

콜백으로 빼내기

```jsx
function withLogging(f) {
  try {
    f();
  } catch (error) {
    logToSnapErrors(error);
  }
}

withLogging(function () {
  saveUserData(user);
});
```

### 요약: 함수 본문을 콜백으로 바꾸기 단계

1. 본문과 본문의 앞부분, 뒷부분을 구분한다.
2. 전체를 함수로 빼낸다.
3. 본문 부분을, 빼낸 함수의 인자로 전달할 함수로 바꾼다.

---

# 요약

- 일급 값은 변수에 저장할 수 있고, 인자로 전달하거나 함수의 리턴값으로 사용할 수 있다.
- 일급이 아닌 기능은? 함수로 감싸서 일급으로 만들 수 있다.
- 함수이름에 있는 암묵적 인자는 일급 값인 인자로 바꾸자 = “암묵적 인자를 드러내기 리팩토링”
- 동작을 추상화하기 위해 본문을 콜백으로 바꾸기 리팩토링을 하자!

### 다음 장에서..

- 개선과 액션에서 고차함수가 얼마나 도움이 되는지 살펴보자.
