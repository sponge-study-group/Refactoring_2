## 6. 기본적인 리팩터링

### 6.1 함수 추출하기

<br />

반대 리팩터링: 함수 인라인하기

```javascript
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();

  //세부 사항 출력
  console.log(`고객명: ${invoice.customer}`);
  console.log(`채무액: ${outstanding}`);
}
```

↓

```javascript
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();

  printDetails(outstanding);

  function printDetails(outstanding) {
    console.log(`고객명: ${invoice.customer}`);
    console.log(`채무액: ${outstanding}`);
  }
}
```

<br />

**배경**

'목적과 구현을 분리'하는 방식  
→ 코드를 보고 무슨 일을 하는지 파악하는 데 오래 걸린다면, 그 부분을 함수로 추출한 뒤, '무슨 일'에 걸맞는 이름을 짓는다.  
→ 나중에 코드를 다시 읽을 때 목적이 한눈에 들어오고, 본문 코드에 대해서는 더 이상 신경 쓸 일이 ×

함수를 짧게 만들면 함수 호출이 많아져서 성능이 느려질 일  
→ 함수가 짧으면 캐싱하기가 더 쉽기 때문에 컴파일러가 최적화하는 데 유리할 때가 많다.

짧은 함수의 이점은 이름을 잘 지어야만 발휘

**절차**

1. 함수는 새로 만들고 목적을 잘 드러내는 이름을 붙인다.  
   → 무엇을 하는지가 드러나야 한다.  
   → 목적이 더 잘 드러나는 이름이 생각나지 않는다면 함수로 추출하면 안 된다는 신호

2. 추출할 코드를 원본 함수에서 복사하여 새 함수에 붙여넣는다.
3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는 지 검사한다. 있다면 매개변수로 전달한다.  
   → 추출한 코드 안에서 값이 바뀌는 변수 중에서 값으로 전달되는 것들은 주의해서 처리.  
   → 추출한 코드에서 지역 변수가 너무 많다면, 다른 리팩터링 기법을 적용해서 변수를 단순하게 바꾼 후, 함수 추출을 다시 시도한다.

4. 컴파일
   → 제대로 처리하지 못한 변수를 찾는 데 도움이 된다.

5. 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꾼다.
6. 테스트
7. 다른 코드에 방금 추출한 것과 비슷하거나 동일한 코드가 없는지 살펴본다. 있다면 방금 추출한 새 함수를 호출할건지 검토한다.

**예시: 유효범위를 벗어나는 함수가 없을 때**

```javascript
function printOwing(invoice) {
  let outstanding = 0;

  console.log('******************');
  console.log('**** 고객 채무 ****');
  console.log('******************');

  // 미해결 채무 계산
  for (const o of invoice.orders) {
    outstanding += o.amount;
  }

  // 마감일 기록
  const today = Clock.today;
  invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);

  // 세부 사항 출력
  console.log(`고객명: ${invoice.customer}`);
  console.log(`채무액: ${outstanding}`);
  console.log(`마감일: ${invoice.dueDate.toLocaleDateString()}`);
}
```

```javascript
function printOwing(invoice) {
  let outstanding = 0;

  printBanner();

  // 미해결 채무 계산
  for (const o of invoice.orders) {
    outstanding += o.amount;
  }

  // 마감일 기록
  const today = Clock.today;
  invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);

  // 세부 사항 출력
  printDetails();

  function printDetails() {
    console.log(`고객명: ${invoice.customer}`);
    console.log(`채무액: ${outstanding}`);
    console.log(`마감일: ${invoice.dueDate.toLocaleDateString()}`);
  }
}

function printBanner() {
  console.log('******************');
  console.log('**** 고객 채무 ****');
  console.log('******************');
}
```

**예시: 지역 변수를 사용할 때**

```javascript
function printOwing(invoice) {
  let outstanding = 0;

  printBanner();

  // 미해결 채무 계산
  for (const o of invoice.orders) {
    outstanding += o.amount;
  }

  recordDueDate(invoice); // 마감일 기록
  printDetails(invoice, outstanding); // 세부 사항 출력
}

function printDetails(invoice, outstanding) {
  console.log(`고객명: ${invoice.customer}`);
  console.log(`채무액: ${outstanding}`);
  console.log(`마감일: ${invoice.dueDate.toLocaleDateString()}`);
}

function recordDueDate(invoice) {
  const today = Clock.today;
  invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 30);
}
```

**예시: 지역 변수의 값을 변경할 때**

1.  코드 근처로 슬라이드

```javascript
function printOwing(invoice) {
  printBanner();

  // 미해결 채무 계산
  for (const o of invoice.orders) {
    let outstanding = 0; // 위치 이동
    outstanding += o.amount;
  }

  recordDueDate(invoice); // 마감일 기록
  printDetails(invoice, outstanding); // 세부 사항 출력
}
```

2. 추출할 부분을 새로운 함수로 복사

```javascript
...

function calculateOutstanding(invoice) {
  let outstanding = 0;
  for (const o of invoice.orders) {
    outstanding += o.amount;
  }
  return outstanding;
}
```

3. outstanding 값을 반환
4. 컴파일해도 아무런 값 출력 ×
5. 추출한 코드의 원래 자리를 새로 뽑아낸 함수로 호출하는 문장으로 교체

```javascript
function printOwing(invoice) {
  printBanner();

  const outstanding = calculateOutstanding(invoice); // 추출한 함수가 반환한 값을 원래 변수에 저장

  recordDueDate(invoice);
  printDetails(invoice, outstanding);
}

function calculateOutstanding(invoice) {
  let result = 0;
  for (const o of invoice.orders) {
    result += o.amount;
  }
  return result;
}
```

---

### 6.2 함수 인라인하기

```javascript
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}
```

↓

```javascript
function getRating(driver) {
  return driver.numberOfLateDeliveries > 5 ? 2 : 1;
}
```

<br />

**배경**

리팩터링 과정에서 잘못 추출된 함수들도 다시 인라인  
→ 잘못 추출된 함수들을 원래 함수로 합친 다음, 필요하면 원하는 형태로 다시 추출

간접 호출을 너무 과하게 쓰는 코드도 인라인  
→ 위임 관계가 복잡하게 얽혀 있으면 인라인

**절차**

1. 다형 메서드인지 확인
2. 인라인할 함수를 호출하는 곳을 모두 찾는다.
3. 각 호출문을 함수 본문으로 교체
4. 하나씩 교체할 때마다 테스트
5. 원래 함수 삭제

**예시**

일이 더 많은 경우

```javascript
function reportLines(aCustomer) {
  const lines = [];
  gatherCustomerData(lines, aCustomer);
  return lines;
}

function gatherCustomerData(out, aCustomer) {
  out.push(['name', aCustomer.name]);
  out.push(['location', aCustomer.location]);
}
```

```javascript
function reportLines(aCustomer) {
  const lines = [];

  lines.push(['name', aCustomer.name]);
  lines.push(['location', aCustomer.location]);

  return lines;
}
```

---

### 6.3 변수 추출하기

<br />

```javascript
return (
  order.quantity * order.itemPrice -
  Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
  Math.min(order.quantity * order.itemPrice * 0.1, 100)
);
```

↓

```javascript
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);

return basePrice - quantityDiscount + shipping;
```

<br />

**배경**

현재 함수 안에서만 의미가 있다면 변수로 추출하는 것이 좋다.  
→ 함수를 벗어난 넓은 문맥의 의미가 된다면, 변수가 아닌 함수로 추출

**절차**

1. 추출하려는 표현식에 부작용은 없는지 확인
2. 불변 변수를 하나 선언하고 이름을 붙일 표현식의 복제본을 대입
3. 원본 표현식을 새로 만든 변수로 교체
4. 테스트
5. 표현식을 여러 곳에서 사용한다면 각각을 새로 만든 변수로 교체. 교체할 때마다 테스트

**예시**

```javascript
function price(order) {
  // 가격(price) = 기본 가격 - 수량 할인 + 배송비
  return (
    order.quantity * order.itemPrice -
    Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
    Math.min(order.quantity * order.itemPrice * 0.1, 100)
  );
}
```

```javascript
function price(order) {
  const basePrice = order.quantity * order.itemPrice; // 기본 가격
  const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05; // 수량 할인
  const shipping = Math.min(basePrice * 0.1, 100); // 배송비

  return basePrice - quantityDiscount + shipping;
}
```

**예시: 클래스 안에서**

```javascript
class Order {
  constructor(aRecord) {
    this._data = aRecord;
  }

  get quantity() {
    return this._data.quantity;
  }

  get itemPrice() {
    return this._data.itemPrice;
  }

  get price() {
    return (
      this.quantity * this.itemPrice - Math.max(0, this.quantity - 500) * this.itemPrice * 0.05 + Math.min(this.quantity * this.itemPrice * 0.1, 100)
    );
  }
}
```

클래스 전체에 영향을 줄 때는 변수가 아닌 메서드로 추출

```javascript
class Order {
  constructor(aRecord) {
    this._data = aRecord;
  }

  get quantity() {
    return this._data.quantity;
  }

  get itemPrice() {
    return this._data.itemPrice;
  }

  get price() {
    return this.basePrice - this.quantityDiscount + this.shipping;
  }

  get basePrice() {
    return this.quantity * this.itemPrice;
  }

  get quantityDiscount() {
    return Math.max(0, this.quantity - 500) * this.itemPrice * 0.05;
  }

  get shipping() {
    return Math.min(this.basePrice * 0.1, 100);
  }
}
```

---

### 6.4 변수 인라인하기

<br />

```javascript
let basePrice = anOrder.basePrice;
return basePrice > 1000;
```

↓

```javascript
return anOrder.basePrice > 1000;
```

<br />

**배경**

변수  
→ 함수 안에서 표현식을 가리키는 이름  
→ 그 이름이 원래 표현식과 다를 바가 없다면, 변수를 인라인 한다.

**절차**

1. 대입문의 우변에서 부작용이 생기는지 확인
2. 변수가 불변으로 선언 ×, 불변으로 변경 후, 테스트
3. 이 변수를 가장 처음 사용하는 코드를 찾아서 우변의 코드로 변경
4. 테스트
5. 변수를 사용하는 부분을 모두 교체할 때까지 반복
6. 변수 선언문과 대입문을 지운다.
7. 테스트

---

### 6.5 함수 선언 바꾸기

<br />

```javascript
function circum(radius) {
    ...
}
```

↓

```javascript
function circumference(radius) {
    ...
}
```

<br />

**배경**

함수 선언  
→ 소프트웨어 시스템의 구성 요소를 조립하는 연결부 역할  
→ 잘못 정의하면 지속적인 방해 요인  
→ **연결부에서 가장 중요한 요소는 함수의 이름**

어떻게 연결하는 것이 더 나은지 이해하게 될 때마다 그에 맞게 코드를 개선할 수 있도록 익숙해져야 한다.

**절차**

간단한 절차

1. 매개변수를 제거하려면, 함수 본문에서 해당 매개변수를 참조하는 곳이 있는지 확인
2. 메서드 선언을 원하는 형태로 변경
3. 기본 메서드 선언을 참조하는 부분을 찾아서 바뀐 형태로 수정
4. 테스트

마이그레이션 절차

1. 이어지는 추출 단계를 수월하게 만들어야 한다면, 함수 본문을 적절히 리팩터링
2. 함수 본문을 새로운 함수로 추출
3. 추출한 함수에 매개변수를 추가해야 한다면, **간단한 절차**를 따라 수정
4. 테스트
5. 기존 함수 인라인
6. 이름을 임시로 붙였다면, 함수 선언 바꾸기 적용
7. 테스트

상속 구조 속에 있는 클래스의 메서드를 변경  
→ 다른 클래스들에도 변경이 반영  
→ 원하는 형태의 메서드를 새로 만들어서 원래 함수를 호출하는 전달 메서드로 활용  
→ 단일 상속 구조라면, 전달 메서드를 슈퍼클래스에 정의

**예시: 함수 이름 바꾸기(간단한 절차)**

```javascript
function circum(radius) {
  return 2 * Math.PI * radius;
}
```

1. 함수 선언부터 수정
2. circum()을 호출한 곳을 모두 찾아 circumfernece()로 변경

```javascript
function circumfernece(radius) {
  return 2 * Math.PI * radius;
}
```

함수 이름 바꾸기와 매개변수 추가를 모두 할 때  
→ 이름 변경 후, 테스트  
→ 매개변수 추가 후, 테스트

함수가 두 개의 클래스의 모두 정의되어 있을 때, 하나의 클래스에서만 이름을 변경하는 경우  
혹은 간단한 절차를 따르다가 문제가 생겼을 경우  
→ 마이그레이션 절차로 수행

**예시: 함수 이름 바꾸기(마이그레이션 절차)**

```javascript
function circumfernece(radius) {
  return 2 * Math.PI * radius;
}

function circumfernece(radius) {
  return 2 * Math.PI * radius;
}
```

1. 함수 본문 전체를 새로운 함수로 추출
2. 수정한 코드 테스트
3. 예전 함수 인라인 → 예전 함수를 호출하는 부분이 모두 새 함수 호출로 변경
4. 하나를 변경할 때마다 테스트 → 모두 변경 시, 기존 함수 삭제

> 함수 선언 바꾸기만큼은 공개된 API, 다시 말해 직접 고칠 수 없는 외부 코드가 사용하는 부분을 리팩터링하기에 좋다.

**예시: 매개변수 추가하기**

```javascript
addReservation(customer) {
  this._reservations.push(customer);
}
```

예약 시 우선순위 큐를 지원하라는 새로운 요구

1. addReservation()의 본문을 새로운 함수로 추출

```javascript
addReservation(customer) {
  this.zz_addReservation(customer);
}

zz_addReservation(customer) {
  this._reservations.push(customer);
}
```

2. 새 함수의 선언문과 호출문에 원하는 매개변수 추가

자바스크립트로 프로그래밍 한다면, 호출문 변경 전에 **어서션을 추가**하여 호출 하는 곳에서 **새로 추가한 매개변수를 실제로 사용하는지 확인**

3. 기존 함수를 인라인하여 호출 코드들이 새 함수를 이용하도록 수정 → 호출문은 한번에 하나씩 변경
4. 다 고친 후, 새 함수의 이름을 기존 함수의 이름으로 수정

**예시: 매개변수를 속성으로 바꾸기**

1. 매개변수로 사용할 코드를 변수로 추출
2. 함수 추출하기로 새 함수 생성
3. 기존 함수 안에 변수로 추출해둔 입력 매개변수 인라인
4. 함수 인라인하기로 기존 함수의 본문을 호출문에 넣는다. → 한번에 하나씩 처리
5. 함수 선언 바꾸기를 다시 한번 더 적용 → 새 함수의 이름을 기존 함수 이름으로 변경

---

### 6.6 변수 캡슐화하기

<br />

```javascript
let defaultOwner = { firstName: '마틴', lastName: '파울러' };
```

↓

```javascript
let defaultOwnerData = { firstName: '마틴', lastName: '파울러' };

export function defaultOwner() {
  return defaultOwnerData;
}

export function setDefaultOwner(arg) {
  defaultOwnerData = arg;
}
```

<br />

**배경**

데이터는 참조하는 모든 부분을 바꿔야 코드가 제대로 작동  
→ 유효범위가 넓어질수록 다루기가 어렵다.

접근할 수 있는 범위가 넓은 데이터를 옮길 때  
→ 데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화  
→ 데이터 재구성이라는 어려운 작업을 함수 재구성이라는 단순한 작업으로 변환!

**절차**

1. 변수로의 접근과 갱신을 전담하는 캡슐화 함수들을 생성
2. 정적 검사 수행
3. 변수를 직접 참조하던 부분을 모두 캡슐화 함수 호출로 변경 → 바꿀 때마다 테스트
4. 변수의 접근 범위 제한
5. 테스트
6. 변수 값이 레코드라면 **레코드 캡슐화하기** 적용을 고려

**예시**

```javascript
// 전역 변수
let defaultOwner = { firstName: '마틴', lastName: '파울러' };

// 참조하는 코드
spaceship.owner = defaultOwner;

// 갱신하는 코드
defaultOwner = { firstName: '레베카', lastName: '파슨스' };
```

1. 데이터를 읽고 쓰는 함수 정의

```javascript
function getDefaultOwner() {
  return defaultOwner;
}

function setDefaultOwner(arg) {
  defaultOwner = arg;
}
```

2. defaultOwner를 참조하는 코드를 찾아서 게터 함수를 호출하도록 수정

```javascript
spaceship.owner = getDefaultOwner();
setDefaultOwner({ firstName: '레베카', lastName: '파슨스' });
```

3. 변수의 접근 범위 제한  
   → 미처 발견하지 못한 참조를 확인할 수 있고, 나중에 수정하는 코드에서도 이 변수에 직접 접근 못하게 만들 수 있다.  
   → 자바스크립트에서는 변수와 접근자 메서드를 같은 파일로 옮기고 접근자만 노출 시킨다.

```javascript
let defaultOwner = { firstName: '마틴', lastName: '파울러' };

export function getDefaultOwner() {
  return defaultOwner;
}

export function setDefaultOwner(arg) {
  defaultOwner = arg;
}
```

변수로의 접근을 제한할 수 없다면, 변수 이름을 바꿔서 다시 테스트

**값 캡슐화하기**

```javascript
const owner1 = defaultOwner();
assert.equal('파울러', owner1.lastName, '처음 값 확인');

const owner2 = defaultOwner();
owner2.lastName = '파슨스';

assert.equal('파슨스', owner1.lastName, 'owner2를 변경한 후');
```

변수뿐만 아니라 변수에 담긴 내용을 변경하는 행위까지 제어할 수 있게 캡슐화하는 방법

1. 그 값을 바꿀 수 없게 만드는 것  
   → 게터가 데이터의 복제본을 반환하도록 수정

```javascript
let defaultOwnerData = { firstName: '마틴', lastName: '파울러' };

export function defaultOwner() {
  return Object.assign({}, defaultOwnerData);
}

export function setDefaultOwner(arg) {
  defaultOwnerData = arg;
}
```

공유 데이터를 변경하기를 원한다면 레코드 캡슐화하기 적용

2. 레코드 캡슐화하기

```javascript
let defaultOwnerData = { firstName: '마틴', lastName: '파울러' };

export function defaultOwner() {
  return new Person(defaultOwnerData);
}

export function setDefaultOwner(arg) {
  defaultOwnerData = arg;
}

class Person {
  constructor(data) {
    this._lastName = data.lastName;
    this._firstName = data.firstName;
  }

  get lastName() {
    return this._lastName;
  }

  get firstName() {
    return this._firstName;
  }
}
```

defaultOwnerData의 속성을 다시 대입하는 연산은 모두 무시

변경을 감지하여 막는 기법을 임시로 활용  
→ 변경하는 부분을 제거  
→ 적절한 변경 함수 제공

링크가 필요 없다면, 세터에서도 복제본을 만드는 편이 좋다.  
→ 데이터를 복제해 저장하여 나중에 원본이 변경돼서 발생하는 사고를 방지

복제가 성능에 주는 영향은 미미하고, 원본을 그대로 사용하면 나중에 디버딩하기 어렵고 시간도 오래 걸릴 위험이 있다.

복제본 만들기와 클래스로 감싸는 방식은 레코드 구조에서 깊이가 1인 속성들까지만 효과가 있다.  
→ 더 깊이 들어가면 복제본과 객체 래핑 단계가 늘어난다.

---

### 6.7 변수 이름 바꾸기

<br />

```javascript
let a = height * width;
```

↓

```javascript
let area = height * width;
```

<br />

**배경**

명확한 프로그래밍의 핵심은 이름짓기  
→ 특히 이름의 중요성은 사용 범위에 영향을 많이 받는다.

**절차**

1. 폭넓게 쓰이는 변수라면 변수 캡슐화하기 고려
2. 이름을 바꿀 변수를 참조하는 곳을 모두 찾아, 하나씩 변경
3. 테스트

**예시**

```javascript
let tpHd = 'untitled';

result += `<h1>${tpHd}</h1>`;

tpHd = obj['articleTitle'];
```

```javascript
result += `<h1>${title()}</h1>`;
setTitle(obj['articleTitle']);

// tpHd 변수의 getter
function title() {
  return tpHd;
}

// tpHd 변수의 setter
function setTitle(arg) {
  tpHd = arg;
}
```

**예시: 상수 이름 바꾸기**

상수의 이름은 캡슐화하기 않고도 복제 방식으로 점진적으로 변경 가능

```javascript
const cpyNm = '애크미 구스베리';
```

```javascript
const companyName = '애크미 구스베리';
const cpyNm = companyName;
```

---

### 6.8 매개변수 객체 만들기

<br />

```javascript
function amountInvoiced(startDate, endDate) {...}
function amountReceived(startDate, endDate) {...}
function amountOverdue(startDate, endDate) {...}
```

↓

```javascript
function amountInvoiced(aDataRange) {...}
function amountReceived(aDataRange) {...}
function amountOverdue(aDataRange) {...}
```

<br />

**배경**

데이터 뭉치들을 데이터 구조로 묶으면, 데이터 사이의 관계가 명확해진다.  
→ 함수가 이 데이터 구조를 받게 하면 매개변수의 수 ↓  
→ 같은 데이터 구조를 사용하는 모든 함수가 원소를 참조할 때 똑같은 이름을 사용하기 때문에, 일관성 ↑

리팩터링을 통해 코드를 더 근본적으로 변경 가능  
→ 데이터 구조에 담길 데이터에 공통으로 적용되는 동작을 추출해서 함수로 만드는 것.

**절차**

1. 적당한 데이터 구조가 없다면, 새로 생성
2. 테스트
3. 함수 선언 바꾸기로 새 데이터 구조를 매개변수로 추가
4. 테스트
5. 함수 호출 시, 새로운 데이터 구조 인스턴스를 넘기도록 수정 → 하나씩 수정할 때마다 테스트
6. 기존 매개변수를 사용하던 코드 → 새 데이터 구조의 원소를 사용하도록 변경
7. 기존 매개변수를 제거하고 테스트

---

### 6.9 여러 함수를 클래스로 묶기

<br />

```javascript
function base(aReading) {...}
function taxableCharge(aReading) {...}
function calculateBaseCharge(aReading) {...}
```

↓

```javascript
class Reading {
  base() {...}
  taxableCharge() {...}
  calculateBaseCharge() {...}
}
```

<br />

**배경**

공통 데이터를 중심으로 긴밀하게 엮여 작동하는 함수들을 밝견하면 클래스로 묶기  
→ 함수들이 공유하는 공통 환경을 더 명확하게 표현 ○  
→ 각 함수에 전달되는 인수 ↓, 함수 호출을 간결하게 만들 수 있다.  
→ 이런 객체를 시스템의 다른 부분에 전달하기 위한 참조 제공

클래스를 지원하지 않는 언어  
→ **함수를 객체처럼 패턴**을 이용해 구현 가능

**절차**

1. 함수들이 공유하는 공통 데이터 레코드를 캡슐화
2. 공통 레코드를 사용하는 함수 각각을 새 클래스로 옮긴다.
3. 데이터를 조작하는 로직들은 함수로 추출하여 새 클래스로 옮긴다.

---

### 6.10 여러 함수를 변환 함수로 묶기

<br />

```javascript
function base(aReading) {...}
function taxableCharge(aReading) {...}
```

↓

```javascript
function enrichReading(argReading) {
  const aReading = _.cloneDeep(argReading);
  aReading.baseCharge = base(aReading);
  aReading.taxableCharge = taxableCharge(aReading);
  return aReading;
}
```

<br />

**배경**

데이터를 입력 받아서 여러 가지 정보를 도출  
→ 도출된 정보를 여러곳에서 사용  
→ 도출 로직이 반복될 수도 있다.
→ 해당 로직을 모아두면 검색과 갱신을 일관된 장소에서 처리할 수 있도록 중복도 방지

도출 작업을 한곳으로 모아두기 위한 방법

1. 변환 함수를 사용
2. 여러 함수를 클래스로 묶기로 처리

**절차**

1. 변환할 레코드를 입력받아서 값을 그대로 반환하는 변환 함수 생성
2. 묶을 함수 중 함수 하나를 골라서 본문 코드를 변환 함수로 옮기고, 처리 결과를 레코드에 새 필드로 기록. 그 다음 클라이언트 코드가 이 필드를 사용하도록 수정
3. 테스트
4. 나머지 관련 함수도 해당 절차에 따라 처리

**예시**

---

### 6.11 단계 쪼개기

<br />

```javascript
const orderData = orderString.split(/\s+/);
const productPrice = priceList[orderData[0].split('-')[1]];
const orderPrice = parseInt(orderData[1]) * productPrice;
```

↓

```javascript
const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString) {
  const values = aString.split(/\s+/);
  return { productID: values[0].split('-')[1], quantity: parseInt(values[1]) };
}

function price(order, priceList) {
  return order.quantity * priceList[order.productID];
}
```

<br />

**배경**

서로 다른 두 대상을 한꺼번에 다루는 코드
→ 각각 별개의 모듈로 분리  
→ 코드를 수정할 때, 동시에 생각할 필요 ✖︎

모듈을 분리하는 가장 간편한 방법  
→ 동작을 연이은 두 단계로 쪼개는 것

규모에 상관없이 여러 단계로 분리하면 좋을만한 코드를 발견할 때마다 기본적인 단계 쪼개기 리팩터링 적용  
→ 다른 단계로 볼 수 있는 코드 영역들이 서로 다른 데이터와 함수를 사용한다면 단게 쪼개기에 적합

**절차**

1. 두 번째 단계에 해당하는 코드를 독립 함수로 추출
2. 테스트
3. 중간 데이터 구조를 만들어서 앞에서 추출한 함수의 인수로 추가
4. 테스트
5. 추출한 두 번째 단계 함수의 매개변수 검토. 첫 번째 단게에 사용되는 매개변수는 중간 데이터 구조로 옮긴다. → 하나씩 옮길 때마다 테스트
6. 첫 번째 단게 코드르르 함수로 추출 + 중간 데이터 구조를 반환하도록 수정

```javascript
function priceOrder(product, quantity, shippingMethod) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
  const shippingPerCase = basePrice > shippingMethod.discountThreshold ? shippingMethod.discountedFee : shippingMethod.feePerCase;
  const shippingCost = quantity * shippingPerCase;
  const price = basePrice - discount + shippingCost;
  return price;
}
```

- 상품 정보를 이용해서 결제 금액 중 **상품 가격 계산**
- 배송 정보를 이용하여 결제 금액 중 **배송비 계산**

1. 배송비 계산 부분을 함수로 추출
2. 중간 데이터 구조 생성

```javascript
function priceOrder(product, quantity, shippingMethod) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
  const priceData = {}; // 중간 데이터 구조
  const price = applyShipping(priceData, basePrice, shippingMethod, quantity, discount);
  return price;
}

// 배송비 계산 함수
function applyShipping(priceData, basePrice, shippingMethod, quantity, discount) {
  const shippingPerCase = basePrice > shippingMethod.discountThreshold ? shippingMethod.discountedFee : shippingMethod.feePerCase;
  const shippingCost = quantity * shippingPerCase;
  const price = basePrice - discount + shippingCost;
  return price;
}
```

3. applyShipping()에 전달되는 매개변수 중  
   → basePrice는 첫 번째 수행하는 코드에서 생성  
   → 중간 데이터 구조로 옮기고 매개변수 목록에서 제거

quantity와 discount도 해당 방식으로 변경

```javascript
function priceOrder(product, quantity, shippingMethod) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
  const priceData = { basePrice: basePrice, quantity: quantity, discount: discount }; // 중간 데이터 구조
  const price = applyShipping(priceData, shippingMethod);
  return price;
}

function applyShipping(priceData, shippingMethod) {
  const shippingPerCase = priceData.basePrice > shippingMethod.discountThreshold ? shippingMethod.discountedFee : shippingMethod.feePerCase;
  const shippingCost = priceData.quantity * shippingPerCase;
  const price = priceData.basePrice - priceData.discount + shippingCost;
  return price;
}
```

4. 첫 번째 단계 코드를 함수로 추출하고 데이터 구조 반환

```javascript
function priceOrder(product, quantity, shippingMethod) {
  const priceData = calculatePricingData(product, quantity);
  const price = applyShipping(priceData, shippingMethod);
  return price;
}

// 첫 번째 단계를 처리하는 함수
function calculatePricingData(product, quantity) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
  return { basePrice: basePrice, quantity: quantity, discount: discount };
}

// 두 번째 단계를 처리하는 함수
function applyShipping(priceData, shippingMethod) {
  const shippingPerCase = priceData.basePrice > shippingMethod.discountThreshold ? shippingMethod.discountedFee : shippingMethod.feePerCase;
  const shippingCost = priceData.quantity * shippingPerCase;
  const price = priceData.basePrice - priceData.discount + shippingCost;
  return price;
}
```
