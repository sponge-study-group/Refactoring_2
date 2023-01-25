# Chapter09 데이터 조직화

>하나의 값이 여러 목적으로 사용된다면 변수 쪼개기를 적용해 용도별로 분리한다.

* 변수 쪼개기
* 변수 이름 바꾸기
* 파생 변수를 질의 함수로 바꾸기
* 참조를 값으로 바꾸기
* 값을 참조로 바꾸기

## 9.1 변수 쪼개기
Split Variable

```javascript
let temp = 2 * (height + width);
console.log(temp);
temp = heigth * width;
console.log(temp);
⬇️
const perimeter = 2 * (heigth + width);
console.log(perimeter);
const area = heigth * width;
console.log(area);
```

## 9.2 필드 이름 바꾸기
Rename Field

>프로그램 곳곳에서 쓰이는 레코드 구조체의 필드 이름들은 중요하다. 데이터 구조는 프로그램을 이해하는 데 큰 역할을 한다.
데이터 테이블이 명확하다면 흐름도가 없어도 이해하기 쉽다.

```javascript
class Organization {
    get name(){ ... }
}
⬇️
class Organnization {
    get title(){ ... }
}
```

## 9.3 파생 변수를 질의 함수로 바꾸기
Replace Derived Variable with Query

>가변 데이터는 원인을 찾기 어려운 오류를 발생시킬 수 있으므로 질의 함수로 바꾸는 것이 좋다.

```javascript
get discountedTotal(){ return this._discountedTotal; }
set discount(aNumber){
    const old = this._discount;
    this._discount = aNumber;
    this._discountedTotal += old - aNumber;
}
⬇️
get discountedTotal(){ return this._baseTotal - this._discount; }
set discount(aNumber){ this._discount = aNumber; }
```

## 9.4 참조를 값으로 바꾸기
Change Reference to Value

>객체(데이터 구조)를 다른 객체(데이터 구조)에 중첩하면 내부 객체를 참조 혹은 값으로 취급할 수 있다.
참조로 다루는 경우에는 내부 객체는 그대로 둔 채 그 객체의 속성만 갱신하며, 값으로 다루는 경우에는 
새로운 속성을 담은 객체로 기존 내부 객체를 통째로 대체한다.
값 객체는 대체로 자유롭게 활용하기 좋은데, 특히 불변이기 때문이다.
불변 데이터는 나중에 그 값이 나 몰래 바뀌어서 내부에 영향을 줄까 염려하지 않아도 된다.
값을 이곳 저곳에서 사용하더라도 서로 간의 참조를 관리하지 않아도 된다. 그래서 값 객체는 분산 시스템과 동시성 시스템에서 특히 유용하다.

```javascript
class Product {
    applyDiscount(arg){ this._price.amount -= arg; }
}
⬇️
class Product {
    applyDiscount(arg){
        this._price = new Money(this._price.amount - arg, this._price.currency);
    }
}
```

## 9.5 값을 참조로 바꾸기
Change Value to Reference

>논리적으로 같은 데이터를 물리적으로 복제해 사용할 떄 가장 크게 문제되는 상황은 그 데이터를 갱신해야 할 때다.
모든 복제본을 찾아서 빠짐없이 갱신해야 하며, 하나라도 놓치면 데이터 일관성이 깨져버린다. 이런 상황이라면
복제된 데이터들을 모두 참조로 바꿔주는 게 좋다.

```javascript
let customer = new Customer(customerData);
⬇️
let customer = customerRepository.get(customerData.id);
```

## 9.6 매직 리터럴 바꾸기
Replace Magic Literal

>매직 리터럴이란 소스 코드에 등장하는 일반적인 리터럴 값을 말한다. 예컨에 움직임을 계산하는 코드에서라면 9.80665라는
숫자가 산재해 있는 모습을 목격할 수 있다. 표준중력을 뜻하지만 코드를 읽는 사람이 값의 의미를 모른다면 숫자 자체로는
의미를 명확히 알려주지 못하므로 매직 리터럴이라 할 수 있다. 
이런 경우 상수를 정의하고 명확하게 코드로 알려주는 것이 좋다.

```javascript
function potentialEnergy(mass, heigth){
    return mass * 9.81 * heigth;
}
⬇️
const STANDARD_GRAVITY = 9.81;
function potentialEnergy(mass, heigth){
    return mass * STANDARD_GRAVITY * heigth;
}
```





