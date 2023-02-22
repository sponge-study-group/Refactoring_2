## 11. API 리팩터링



### 11.1 질의 함수와 변경 함수 분리하기

<br />

```javascript
function getTotalOutstandingAndSendBill() {
  const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
  sendBill();

  return result;
}
```

↓

```javascript
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);  
}

function sendBill() {
  emailGateway.send(formatBill(customer));
}
```

<br />

**배경**  
외부에서 관찰할 수 있는 겉보기 부수효과가 전혀 없이 값을 반환해주는 함수를 추구.  

겉보기 부수효과가 있는 함수와 없는 함수르르 구분하는 방법  
→ **질의 함수는 모두 부수효과가 없어야 한다**라는 규칙을 따르는 것

값을 반환하면서 부수효과도 있는 함수를 발견  
→ 상태를 변경하는 부분과 질의하는 부분을 분리

**절차**  
① 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다.  
② 새 질의 함수에서 부수효과를 모두 제거한다.  
③ 정적 검사 수행  
④ 원래 함수를 호출하는 곳을 모두 찾는다.  
→ 호출하는 곳에서 반환 값을 사용한다면, 질의 함수를 호출하도록 변경  
→ 원래 함수를 호출하는 코드를 바로 아래 줄에 새로 추가  
→ 수정할 때마다 테스트  
⑤ 원래 함수에서 질의 관련 코드 제거  
⑥ 테스트

---

### 11.2 함수 매개변수화하기

<br />

```javascript
function tenPercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.1);
}

function fivePercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.05);
}
```

↓

```javascript
function raise(aPerson, factor) {
  aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

<br />

**배경**  
두 함수의 로직이 아주 비슷하고 리터럴 값만 다르다면  
→ 다른 값만 매개변수로 받아 처리하는 함수로 합쳐서 중복을 없앨 수 있다.

**절차**  
① 비슷한 함수 중 하나를 선택  
② 함수 선언 바꾸기로 리터럴들을 매개변수로 추가  
③ 이 함수를 호출하는 곳 모두에 적절한 리터럴 값 추가  
④ 테스트  
⑤ 매개변수로 받은 값을 사용하도록 함수 본문 수정  
⑥ 비슷한 다른 함수를 호출하는 곳을 찾아 매개변수화된 함수를 호출하도록 하나씩 수정  
→ 대체할 함수와 다르게 동작한다면, 그 비슷한 함수의 동작도 처리할 수 있도록 본문 코드를 적절히 수정한 후 진행한다. 

---

### 11.3 플래그 인수 제거하기

<br />

```javascript
function setDimension(name, value) {
  if (name === "height") {
    this._height = value;
    return;
  }

  if (name === "width") {
    this._width = value;
    return;
  }
}
```

↓

```javascript
function setHeight(value) {this._height = value;}
function setWidth (value) {this._width = value;}
```

<br />

**배경**  
플래그 인수  
→ 호출되는 함수가 실행할 로직을 선택하기 위해 전달되는 인수  
→ 호출할 수 있는 함수들이 무엇이고 어떻게 호출해야 하는지 이해하기가 어려워진다.  

특정한 기능 하나만 수행하는 명시적인 함수를 제공

함수 하나에서 플래그 이누를 두 개 이상 사용한다면  
→ 함수 하나가 많은 일을 처리하고 있다는 신호  
→ 같은 로직을 조합해내는 더 간단한 함수를 만들 방법을 고민

**절차**  
① 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성  
→ 함수에 조건문이 포함되어 있다면 **조건문 분해하기**로 명시적 함수들을 생성  
② 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정


---

### 11.4 객체 통째로 넘기기

<br />

```javascript
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;

if (aPlan.withinRange(low, high))
```

↓

```javascript
if (aPlan.withinRange(aRoom.daysTempRange)) 
```

<br />

**배경**  
레코드를 통째로 넘기면 변화에 대응하기 쉽다.  
→ 함수가 더 다양한 데이터를 사용하도록 바뀌어도 매개변수 목록은 수정할 필요 ×  
→ 매개변수 목록이 짧아져서 함수 사용법을 이해하기 쉬워진다.  

함수가 **레코드 자체에 의존하기를 원치 않을 때**는 이 리팩터링을 수행 ×  
→ 특히 레코드와 함수가 서로 다른 모듈에 속한 경우

어떤 객체로부터 값 몇 개를 얻은 후 그 값들만으로 무언가를 하는 로직  
→ 그 로직을 객체 안으로 집어넣어야 한다.  

한 객체가 제공하는 기능 중 항상 똑같은 일부만을 사용하는 코드  
→ 그 기능만 따로 묶어서 클래스로 추출  

**절차**  
① 매개변수들을 원하는 형태로 받는 빈 함수를 만든다.  
② 새 함수의 본문에서는 원래 함수를 호출하고, 새 매개변수와 원래 함수의 매개변수를 매핑한다.  
③ 정적 검사 수행  
④ 모든 호출자가 새 함수를 사용하도록 수정. 하나씩 수정하면서 테스트.  
⑤ 호출자를 모두 수정했다면 원래 함수 인라인  
⑥ 새 함수의 이름을 수정하고 모든 호출자에 반영

---

### 11.5 매개변수를 질의 함수로 바꾸기

<br />

```javascript
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
  // 연휴 계산...
}
```

↓

```javascript
availableVacation(anEmployee)

function availableVacation(anEmployee) {
  const grade = anEmployee.grade;
  // 연휴 계산...
}
```

<br />

**배경**  
매개변수 목록  
→ 함수의 변동 요인을 모아놓은 곳  
→ 함수의 동작에 변화를 줄 수 있는 일차적인 수단

피호출 함수가 **스스로 쉽게** 결정할 수 있는 값을 매개변수로 건네는 것  
→ 일종의 중복  
→ 해당 매개변수를 제거하서 피호출자에게 책임을 더 부여하고, 호출하는 쪽을 간소하게 만들자.

매개변수를 질의 함수로 바꾸지 말아야 할 상황  
→ 매개변수를 제거하면 피호출 함수에 원치 않는 의존성이 생길 때  

매개변수를 없에는 대신 가변 전역 변수를 이용하는 일 ×

**절차**  
① 필요하다면 대상 매개변수의 값을 계산하는 코드를 별도 함수로 추출  
② 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 변경.  
③ **함수 선언 바꾸기**로 대상 매개변수 삭제

---

### 11.6 질의 함수를 매개변수로 바꾸기

<br />

```javascript
targetTemperature(aPlan)

function targetTemperature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
  // 생략
}
```

↓

```javascript
targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
  // 생략
}
```

<br />

**배경**  
전역 변수를 참조한다거나 제거하길 원하는 원소를 참조하는 경우  
→ 해당 참조를 매개변수 바꿔서 해결  
→ 참조에 대한 책임을 호출자로 옮기는 것

질의 함수를 매개변수로 바꾸면 어떤 값을 제공할지 호출자가 알아내야 한다.  
→ 결국 호출자가 복잡해진다.  
→ 프로젝트를 진행하면서 균형점이 옮겨질 수 있다.

**절차**  
① 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리  
② 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 분리  
③ 방금 만든 변수를 인라인하여 제거  
④ 원래 함수도 인라인  
⑤ 새 함수의 이름을 원래 함수 이름으로 변경

---

### 11.7 세터 제거하기

<br />

```javascript
class Person {
  get name() { ... }
  set name(aString) { ... }
}
```

↓

```javascript
class Person {
  get name() { ... }
}
```

<br />

**배경**  
세터 제거하기 리팩터링이 필요한 상황  
1. 사람들이 무조건 접근자 메서드를 통해서만 필드를 다루려 할 때  
→ 생성자에서만 호출하는 세터가 생긴다면 오히려 세터를 제거하자.
2. 클라이언트에서 생성 스크립트를 사용해 객체를 생성할 때  
→ 스크립트가 완료된 후에는 해당 객체가 변경되지 않을거라고 생각.  
→ 세터를 제거하여 의도를 더 명확하게 하자.

**절차**  
① 설정해야 할 값을 생성자에게 받지 않는다면 그 값을 받을 매개변수를 생성자에게 추가.  
→ 생성자 안에서 적절한 세터 호출  
② 생성자 밖에서 세터를 호출하는 곳을 찾아서 제거하고 대신 새로운 생성자 사용.  
③ 세터 메서드를 인라인.  
→ 해당 필드를 불변으로 만들도록 하자.  
④ 테스트

---

### 11.8 생성자를 팩터리 함수로 바꾸기

<br />

```javascript
leadEngineer = new Employee(document.leadEngineer, 'E');
```

↓

```javascript
leadEngineer = createEngineer(document.leadEngineer);
```

<br />

**배경**  
자바 생성자는 반드시 그 생성자를 정의한 클래스의 인스턴스를 반환해야 한다.  
생성자의 이름도 고정되어, 더 적절한 이름이 있어도 사용할 수 ×.  
또한 해당 생성자를 호출하려면 특별한 연산자를 사용해야 해서 일반 함수가 있어야 할 부분에서는 사용하기 어렵다.

팩터리 함수는 이런 제약이 없다. 팩터리 함수를 구현하는 과정에서 생성자를 호출할 수는 있지만, 원한다면 다른 무언가로 대체할 수 있다.

**절차**  
① 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래의 생성자를 호출한다.  
② 생성자를 호출하던 코드를 팩터리 함수 호출로 바꾼다.  
③ 하나씩 수정할 때마다 테스트.  
④ 생성자의 가시 범위가 최소가 되도록 제한.  

---

### 11.9 함수를 명령으로 바꾸기

<br />

```javascript
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  // 생략
}
```

↓

```javascript
class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }

  execute() {
    this._result = 0;
    this._healthLevel = 0;
    // 생략
  }
}
```

<br />

**배경**  
함수를 그 함수만을 위한 객체 안으로 캡슐화  
→ '명령 객체' 혹은 '명령'  
→ 주로 메서드 하나로 구성  
→ 이 메서드를 요청해 실행하는 것이 해당 객체의 목표

명령 객체를 사용할 때 장점  
1. 평범한 함수 매커니즘보다 훨씬 유연하게 함수를 제어하고 표현 가능
2. 되돌리기 같은 보조 연산 제공
3. 수명주기를 더 정밀하게 제어하는 데 필요한 매개변수를 만들어주는 메서드도 제공할 수 ○
4. 상속과 훅을 통해 사용자 맞춤형으로 만들 수 있다.
5. 일급 함수 혹은 중첩 함수 지원하지 않는 언어에서 비슷하게 사용하도록 도울 수 있다.

일급 함수와 명령 중 선택해야 한다면, 일급 함수 선택.  
→ 명령을 선택할 때는 해당 방식보다 더 간단한 방식으로 얻을 수 없는 기능이 필요할 때뿐

**절차**  
① 대상 함수의 기능을 옮길 빈 클래스를 생성. 클래스 이름은 함수 이름을 바탕으로 짓는다.  
② 방금 생성한 빈 클래스로 함수를 옮긴다.  
③ 함수의 인수들은 각각 명령의 필드로 만들어 생성자를 통해 설정할지 고민.  

---

### 11.10 명령을 함수로 바꾸기

<br />

```javascript
class ChargeCalculator {
  constructor (customer, usage){
    this._customer = customer;
    this._usage = usage;
  }
  
  execute() {
    return this._customer.rate * this._usage;
  }
}
```

↓

```javascript
function charge(customer, usage) {
  return customer.rate * usage;
}
```

<br />

**배경**  
명령은 함수를 호출해 정해진 일을 수행하는 용도  
→ 로직이 크게 복잡하지 않다면, 명령 객체는 단점 ↑

**절차**  
① 명령을 생성하는 코드와 명령의 실행 메서드를 호출하는 코드를 함께 함수로 추출  
② 명령의 실행 함수가 호출하는 보조 메서드를 인라인  
③ **함수 선언 바꾸기**를 적용해 생성자의 매개변수 모드를 명령의 실행 메서드로 옮긴다.  
④ 명령의 실행 메서드에서 참조하는 필드들 대신 대응하는 매개변수를 사용하도록 수정.  
⑤ 생성자 호출과 명령의 실행 메서드 호출을 대체 함수 안으로 인라인  
⑥ 테스트  
⑦ **죽은 코드 제거하기**를 통해 명령 클래스를 없앤다.

---

### 11.11 수정된 값 반환하기

<br />

```javascript
let totalAscent = 0;

calculateAscent();

function calculateAscent() {
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i-1].elevation;
    totalAscent += (verticalChange > 0) ? verticalChange : 0;
  }
}
```

↓

```javascript
const totalAscent = calculateAscent();

function calculateAscent() {
  let result = 0;

  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i-1].elevation;
    result += (verticalChange > 0) ? verticalChange : 0;
  }

  return result;
}
```

<br />

**배경**  
같은 데이터 블록을 읽고 수정한느 코드가 여러 곳이라면  
→ 데이터가 수정되는 흐름과 코드의 흐름을 일치시키기 어렵다.  
→ 데이터가 수정된다면, 그 사실을 명확히 알려주고 어느 함수가 무슨 일을 하는지 쉽게 알려야 한다.

변수를 갱신하는 함수  
→ 수정된 값을 반환하여 호출자가 그 값을 변수에 담아두도록 하는 것.

값 하나를 계산한다는 목적이 있다면 → 매우 효과적  
값 여러 개를 갱신하는 함수 → 비효과적

**절차**  
① 함수가 수정된 값을 반환하게 해, 호출자가 그 값을 자신의 변수에 저장하게 한다.  
② 테스트  
③ 피호출 함수 안에 반환할 값을 가리키는 새로운 변수 생성.  
④ 테스트  
⑤ 계산이 선언 시점에 바로 대입.  
⑥ 테스트  
⑦ 피호출 함수의 변수 이름을 역할에 어울리도록 수정.  
⑧ 테스트

---

### 11.12 오류 코드를 예외로 바꾸기

<br />

```javascript
if (data)
  return new ShippingRules(data);
else
  return -23;
```

↓

```javascript
if (data)
  return new ShippingRules(data);
else
  throw new OrderProcessingError(-23);
```

<br />

**배경**  
예외는 프로그래밍 언어에서 제공하는 독립적인 오류 처리 매커니즘.  
→ 예외를 사용하면 오류 코드를 일일이 검사하거나 오류를 식별할 필요 ×  
→ 독자적인 흐름을 통해 오류 발생에 대한 복잡한 상황 코드를 작성하거나 읽을 일이 ×

예외는 정확히 예상 밖의 동작일 때만 쓰여야 한다.  
→ 예외를 던지는 코드를 프로그램 종료 코드로 바꿔도 프로그램이 여전히 정상 동작할지를 따져보는 것.  
→ 정상 동작하지 않을 것 같다면 예외를 사용하지 말라는 신호

**절차**  
① 콜스택 상위에 해당 예외를 처리할 예외 핸들러를 작성.  
② 테스트  
③ 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다.  
④ 정적 검사 수행  
⑤ catch절을 수정하여 직접 처리할 수 있는 예외는 적절히 대처하고 그렇지 않은 예외는 다시 던진다.  
⑥ 테스트  
⑦ 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정.  
⑧ 모두 수정했다면, 오류에 관련된 코드를 모두 제거.

---

### 11.13 예외를 사전확인으로 바꾸기

<br />

```javascript
double getValueForPeriod (int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}
```

↓

```javascript
double getValueForPeriod (int periodNumber) {
  return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```

<br />

**배경**  
함수 수행 시, 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면  
→ 예외를 던지는 대신 **호출하는 곳에서 조건을 검사**하도록 해야 한다.

**절차**  
① 예외 상황을 검사할 수 있는 조건문 추가. catch 블록의 코드를 조건문의 조건절 중 하나로 옮기고, 남은 try 블록의 코드를 다른 조건절로 옮긴다.  
② catch 블록에 어서션을 추가하고 테스트.  
③ try catch 블록을 제거.  
④ 테스트

---
