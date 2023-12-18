## 목표

- 암묵적 입력과 출력을 제거해서 재사용하기 좋은 코드 만들기
- 복잡한 코드를 더 좋은 구조로 만들기

### 단계

### 1. 비즈니스 요구사항과 설계를 맞추기

- **요구사항**
  - 장바구니에 담긴 제품을 주문할 때, 무료 배송인지 확인하는 것
- **기존 함수의 문제점**
  - 장바구니로 무료배송을 확인하지 않고, 제품의 합계와 가격으로 확인하고 있다.
    → 비즈니스 요구사항과 맞지 않다.
  - 중복된 코드가 있다.
    → 합계에 제품 가격을 더하는 코드가 두군데에 있다.
- as-is

```jsx
function gets_free_shipping(total, item_price) {
  return item_price + total >= 20;
}
```

- to-be

```jsx
function gets_free_shipping(cart) {
  return calc_total(cart) >= 20;
}
```

- 바꾼 점
  - 제품가격과 합계 대신 **장바구니 데이터를 사용**한다.

### 2. 비즈니스 요구 사항과 함수를 맞추기

- 함수의 동작을 바꾼건 Refactoring 이 아니다.
- 함수 시그니처가 바뀌었기 때문에 사용하는 부분을 고쳐야한다.

```jsx
function update_shipping_icons() {
  var buttons = get_buy_buttons_dom();

  for (var i = 0; i < buttons.length; i++) {
    var button = buttons[i];
    var item = button.item;

    // as-is
    if (gets_free_shipping(shopping_cart_total, item.price))
      // to-be
      var new_cart = add_item(shopping_cart, item.name, item.price);
    if (gets_free_shipping(new_cart)) button.show_free_shipping_icon();
    else button.hide_free_shipping_icon();
  }
}
```

### 3. 암묵적 입력과 출력 줄이기

- 먼저 암묵적 입력을 명시적 입력인 인자로 바꿔보자.

### 4. 코드 다시 살펴보기

### 5. 계산 분류하기

### 6. add_item() 을 분리해 더 좋은 설계 만들기

### 7. 카피-온-라이트 패턴을 빼내기

### 8. add_item() 사용하기

### 9. 계산을 분류하기

### 10. 작은 함수와 많은 계산
