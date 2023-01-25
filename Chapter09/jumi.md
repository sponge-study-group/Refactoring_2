## 9. 데이터 조직화

### 9.1 변수 쪼개기

<br />

```javascript
let temp = 2 * (height + width)

console.log(temp)

temp = height * width

console.log(temp)
```

↓

```javascript
const perimeter = 2 * (height + width)

console.log(perimeter)

const area = height * width

console.log(area)
```

<br />

**배경**  
변수의 대입이 두 번 이상 이뤄진다면 여러 가지 역할을 수행한다는 신호  
→ 역할이 둘 이상인 변수가 있다면 쪼개야 한다.

**절차**  
① 변수를 선언한 곳과 값을 처음 대입하는 곳에서 변수 이름을 변경
② 가능하면 이때 불변으로 선언  
③ 이 변수에 두 번째로 대입하는 곳 앞까지 모든 참조를 새로운 변수 이름으로 변경  
④ 두 번째 대입 시 변수를 원래 이름으로 선언  
⑤ 테스트  
⑥ 해당 과정 반복  
⑦ 테스트  


---

### 9.2 필드 이름 바꾸기

<br />

```javascript
class Oranization {
  get name() { ... }
}
```

↓

```javascript
class Oranization {
  get title() { ... }
}
```

<br />

**배경**  
데이터 구조가 중요한 만큼 깔끔하게 관리  
→ 이 과정에서 레코드의 필드 이름을 바꾸고 싶을 수 있는데, 이는 클래스에서도 마찬가지  
→ 게터와 세터 메서드는 사용자 입장에서 필드와 다를 바 없다.

**절차**  
① 레코드의 유효 범위가 제한적이라면 필드에 접근하는 모든 코드를 수정한 후, 테스트(이후 단계는 필요 ×)  
② 레코드가 캡슐화 × → 레코드 캡슐화  
③ 캡슐화된 객체 안의 private 필드명을 변경. 그에 맞게 내부 메서드들 수정  
④ 테스트  
⑤ 생성자의 매개변수 중 필드와 이름이 겹치는 게 있다면 함수 선언 바꾸기로 변경    
⑥ 접근자들의 이름도 수정   


---

### 9.3 파생 변수를 질의 함수로 바꾸기

<br />

```javascript
get discountedTotal() { return this._discountedTotal }
set discount(aNumber) {
  const old = this._discount
  this._discount = aNumber
  this._discountedTotal += old = aNumber
}
```

↓

```javascript
get discountedTotal() { return this._baseTotal = this._discount }
set discount(aNumber) { this._discount = aNumber }
```

<br />

**배경**  
가변 데이터가 서로 다른 두 코드를 이상한 방식으로 결합  
→ 한 쪽 코드에서 수정한 값이 연쇄효과를 일으켜 다른 쪽 코드에서 어려운 문제를 야기시킬 수 있다.  
→ 가변 데이터의 유효 범위를 가능한 한 좁혀야 한다. 

새로운 데이터 구조를 생성하는 변형 연산  
→ 계산 코드로 대체할 수 있더라도 그대로 두는 것이 좋다.  

변형 연산의 종류  
1. 데이터 구조를 감싸며 그 데이터에 기초하여 계산한 결과를 속성으로 제공하는 객체  
2. 데이터 구조를 받아 다른 데이터 구조로 변환해 반환하는 함수  

**절차**  
① 변구 값이 갱신되는 지점을 모두 찾아, 필요에 따라 변수 쪼개기를 활용해 각 지점에서 변수를 분리   
② 해당 변수의 값을 계산해주는 함수를 만든다.  
③ 해당 변수가 사용되는 모든 곳에 어서션을 추가하여 함수의 계산 결과가 변수의 값과 같은지 확인  
④ 테스트  
⑤ 변수를 모두 함수 호출로 대체  
⑥ 테스트  
⑦ 변수를 선언하고 갱신하는 코드 → 죽은 코드 제거하기

---

### 9.4 참조를 값으로 바꾸기

<br />

```javascript
class Product {
  applyDiscount(arg) { this._price.amount -= arg }
}
```

↓

```javascript
class Product {
  applyDiscount(arg) { 
    this._price = new Money(this._price.amount - arg, this._price.currency)
  }
}
```

<br />

**배경**  
필드를 값으로 다룬다면 내부 객체의 클래스를 수정하여 값 객체로 만들 수 있다.  
→ 값 객체는 불변 데이터 구조로 자유롭게 활용하기 좋다.

_리팩터링 적용하면 안되는 상황_  
특정 객체를 여러 객체에서 공유하고자 할 때, 공유 객체의 값을 변경하고 이를 관련된 객체 모두에게 알려줘야 한다면  
→ 공유 객체를 참조로 다뤄야 한다.

**절차**  
① 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인  
② 각각의 세터를 하나씩 제거  
③ 이 값 객체의 필드들을 사용하는 동치성 비교 메서드를 만든다.  

---

### 9.5 값을 참조로 바꾸기

<br />

```javascript
let customer = new Customer(customerData)
```

↓

```javascript
let customer = customerRepository.get(customerData.id)
```

<br />

**배경**  
논리적으로 같은 데이터를 물리적으로 복제해 사용할 때 가장 크게 문제되는 상황은 그 데이터를 갱신해야 할 때  
→ 모든 복제본을 찾아서 빠짐없이 갱신  
→ 이런 상황이라면 복제된 데이터들을 모두 참조로 바꿔주는게 좋다.

값을 참조로 바꾸면 엔티티 하나당 객체도 단 하나만 존재  
→ 이런 객체들을 한데 모아놓고 클라이언트들의 접근을 관리해주는 저장소가 필요  
→ 각 엔티티를 표현하는 객체를 한 번만 만들고, 객체가 필요한 곳에서는 이 저장소를 통해 얻어 쓰는 방식  

**절차**  
① 같은 부류에 속하는 객체들을 보관할 저장소를 만든다.  
② 생성자에서 이 부류의 객체들 중 특정 객체를 정확히 찾아내는 방법이 있는지 확인  
③ 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾는다. 하나 수정할 때마다 테스트.  


---

### 9.6 매직 리터럴 바꾸기

<br />

```javascript
function potentialEnergy(mass, height) {
  return mass * 9.81 * height
}
```

↓

```javascript
const STANDARD_GRAVITY = 9.81

function potentialEnergy(mass, height) {
  return mass * STANDARD_GRAVITY * height
}
```

<br />

**배경**  
매직 리터럴  
→ 소스 코드에 등장하는 일반적인 리터럴 값  
→ 숫자 자체로는 어떤 의미인지 정확하게 모르는 어려운 값들  

의미 전달면에서 값을 바로 쓰는 것보다 나을게 없거나 리터럴이 함수 하나에서만 쓰이고 그 함수에서 충분히 의미 전달이 된다면 상수로 바꾸는 이득이 줄어든다.

**절차**  
① 상수를 선언하고 매직 리터럴을 대입  
② 해당 리터럴이 사용되는 곳을 모두 찾는다.  
③ 찾은 곳 각각에서 리터럴이 새 상수와 똑같은 의미로 쓰였는지 확인. 같은 의미라면 상수로 대체한 후 테스트.  


---