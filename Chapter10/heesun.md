# 조건부 로직 간소화

📚 목차
* 조건문 분해하기
  - 로직을 이해하기 쉽게 바꾸기
* 중복 조건식 통합하기
  - 논리적 조합을 명확하기 다듬기
* 중첩 조건문을 보호 구문으로 바꾸기
  - 핵심 로직에 들어가기 전 검사하기
* 조건부 로직을 다형성으로 바꾸기
  - 똑같은 분기 로직 (주로 swich문)이 등장하면 다형성 적용하기
* 특이 케이스 추가하기 (널 객체 추가하기)
  - 널과 같은 특이 케이스를 처리하는 데 조건부 로직이 흔히 쓰인다. 
    이 처리 로직이 거의 똑같다면 특이 케이스 추가하기로 코드 중복을 줄인다.
* 어서션 추가하기
  - 프로그램 결과에 따라 다르게 동작해야 하면 어서션 추가하기를 적용한다.


### 10.1 조건문 분해하기
Decompose Conditional


>복잡한 조건부 로직은 함수로 추출해서 간단하게 만든다.



```javascript
// before
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge

// after
if (summer())
  charge = summerCharge();
else
  charge = regularCharge();
```

### 10.2 조건식 통합하기
Consolidate Conditional Expression


>비교하는 조건은 다르지만 결과가 똑같은 코드는 하나로 통합한다.
>여러 조각으로 나뉜 조각을들 하나로 통합함으로써 내가 하려는 일이 더 명확해진다.


* 제어플래그란 코드의 동작을 변경하는데 사용되는 변수로, 이는 항상 악취이다.


```javascript
// before
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;

// after
function disabilityAmount(anEmployee) {
  if (isNotEligableForDisability()) return 0;

function isNotEligableForDisability() {
  return ((anEmployee.seniority < 2)
    || (anEmployee.monthsDisabled > 12)
    || (anEmployee.isPartTime));
}
```

### 10.3 중첩 조건문을 보호 구문으로 바꾸기
Replace Nested Conditional with Guard Clauses

>

```javascript
// before
function payAmount(employee) {
  let result;
  if(isDead) {
    result = deadAmount();
  }
  else {
    if (isSeparated) {
      result = separatedAmount();
    }
    else {
      if (isRetired)
        result = retiredAmount();
      else
        result = normalPayAmount();
      }
  }
  return result;
}

// after
function getpayAmount(employee) {
  if(isDead) return deadAmount();
  if (isSeparated) return separatedAmount();
  if(isRetired) return retiredAmount();
  return normalPayAmount();
}

- 중첩 조건문을 보호구문으로 바꾸는 핵심은 의도부각
- if-then-else의 경우 양쪽이 모두 중요, 보호구문의 경우 핵심이아니라 무언가 조치를취하고 빠져나온다는 느낌.

// before
function payAmount(employee) {
  let result;
  if(isDead) {
    result = deadAmount();
  }
  else {
    if (isSeparated) {
      result = separatedAmount();
    }
    else {
      if (isRetired)
        result = retiredAmount();
      else
        result = normalPayAmount();
      }
  }
  return result;
}

// 1. 가장 바깥것을 선택하여 보호구문으로 변경후 테스트
function payAmount(employee) {
  let result;
  if(isDead) return deadAmount();
  if (isSeparated) {
    result = separatedAmount();
  }
  else {
    if (isRetired)
      result = retiredAmount();
    else
      result = normalPayAmount();
    }
 return result;
}

// 2. 위 과정을 반복하기
```

### 10.4 조건부 로직을 다형성으로 바꾸기
Replace Conditional with Polymorphism

>복잡한 조건부 로직은 클래스와 다형성을 이용하면 확실하게 분리할 수 있다.

```javascript
swich (bird.type) {
  case '유럽 제비':
  return "보통이다";
  case '아프리카 제비';
  return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
  case '노르웨이 파랑 앵무';
  return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
  default:
  return "알 수 없다";
}
⬇
class EuropeanSwallow {
  get plumage(){
    return "보통이다";
  }
}

class AfricanSwallow {
  get plumage(){
    return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
  }
}

class NorwegianBlueParrot {
  get plumage(){
    return (this.voltage > 100) ? "그을렸다" : "예쁘다";
  }
}
```

* 절차
1. 다형적 동작을 표현하는 클래스들이 아직 없다면 만들어준다. 이왕이면 적합한 인스턴스를 알아서 만들어 반환하는
팩터리 함수도 함께 만든다.
2. 호출하는 코드에서 팩터리 함수를 사용하게 한다.
3. 조건부 로직 함수를 슈퍼 클래스로 옮긴다. -> 조건부 로직이 온전한 함수로 분리되어 있지 않다면 먼저 함수로 추출한다.
4. 서브클래스 중 하나를 선택한다. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다. 조건부 문장 중 선택된 서브클래스에 해당하는 조건절을 서브클래스 메서드로 복사한 다음 적절히 수정한다.

>새 종류에 따라 다르게 동작하는 함수가 보인다.
```javascript
function plumages(birds){
  return new Map(birds.map(b => [b.nname, plumage(b)]));
}

function speeds(birds){
  return new Map(birds.map(b => [b.name, airSpeedVelocity(b)]));
}

function plumage(bird){ // 깃털상태
  switch(bird.type){
    case '유럽 제비':
      return "보통이다";
    case '아프리카 제비':
      return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
    case '노르웨이 파랑 앵무':
      return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
    default:
      return "알 수 없다";
  }
}

function airSpeedVelocity(bird){ // 비행속도
  swich(bird.type){
    case '유럽 제비':
      return 35;
    case '아프리카 제비':
      return 40 - 2 * bird.numberOfCoconuts;
    case '노르웨이 파랑 앵무':
      return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
    default:
      return null;
  }
}

function plumage(bird){
  return new Bird(bird).plumage;
}

function airSpeedVelocity(bird){
  return new Bird(bird).airSpeedVelocity;
}
```

airSpeedVelocity()와 plumage()를 Bird라는 클래스로 묶는다.(여러 함수를 클래스로 묶기)
⬇️
```javascript
class Bird {
  constructor(birdObject){
    Object.assign(this, birdObject);
  }
  
  get plumage(){
    swich(this.type){
      case '유럽 제비':
        return "보통이다";
      case '아프리카 제비':
        return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
      case '노르웨이 파랑 앵무':
        return (this.voltage > 100) ? "그을렸다" : "예쁘다";
      default:
        return "알 수 없다";
    }
  }
  
  get sirSpeedVelocity(){
    switch(this.type){
      case '유럽 제비':
        return 35;
      case '아프리카 제비':
        return 40 -2 * this.numberOfCoconuts;
      case '노르웨이 파랑 앵무':
        return (this.isNailed) ? 0 : 10 + this.voltage / 10;
      default:
        return null;
    }
  } 
}
```

⬇️ 최종 코드
```javascript
function speeds(birds){
  return new Map(birds
      .map(b => createBird(b))
      .map(bird => [bird.name, bird.airSpeedVelocity]));
}

function createBird(bird){
  switch(bird.type){
    case '유럽 제비':
      return new EuropeanSwallow(bird);
    case '아프리카 제비':
      return new AfricanSwallow(bird);
    case '노르웨이 파랑 앵무':
      return new NorwegianBlueParrot(bird);
    default:
      return new Bird(bird);
  }
}

class Bird {
  constructor(birdObject){
    Object.assign(this, birdObject);
  }
  get plumage(){
    return "알 수 없다";
  }
  get airSpeedVelocity(){
    return null;
  }
}

class EuropeanSwallow extends Bird {
  get plumage(){
    return "보통이다";
  }
  get airSpeedVelocity(){
    return 35;
  }
}

class AfricanSwallow extends Bird {
  get plumage(){
    return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
  }
  get airSpeedVelocity(){
    return 40 - 2 * this.numberOfCoconuts;
  }
}

class NorwegianBlueParrot extends Bird {
  get plumage(){
    return (this.voltage > 100) ? "그을렸다" : "예쁘다";
  }
  get airSpeedVelocity(){
    return (this.isNailed) ? 0 : 10 + this.voltage / 10;
  }
}

```
>자바스크립트에서는 타입 계층 구조 없이도 다형성을 표현할 수 있다. 객체가 적절한 이름의 메서드만 구현하고 있다면 
>아무 문제없이 같은 타입으로 취급하기 때문이다. (이를 덕 타이핑이라 한다.)
>최종 코드에서는 슈퍼클래스가 클래스들의 관계를 잘 설명해주기 때문에 그대로 둔다.


### 10.5 특이 케이스 추가하기
Introduce Special Case

>데이터 구조의 특정 값을 확인한 후 똑같은 동작을 수행하는 코드가 곳곳에 등장하는 경우가 더러 있는데,
>흔히 볼 수 있는 중복 코드 중 하나다.
>코드베이스에서 특정 값에 대해 똑같이 반응하는 코드가 여러 곳이라면 그 반응을 한 데로 
>모으는 게 효율적이다.
>특수한 경우의 공통 동작을 요소 하나에 모아서 사용하는 특이 케이스 패턴이라는 것이 있는데,
바로 이럴 때 적용하면 좋은 메커니즘이다.
이 패턴을 활용하면 특이 케이스를 확인하는 코드 대부분을 단순한 함수 호출로 바꿀 수 있다.
특이 케이스 객체에서 단순히 데이터를 읽기만 한다면 반환할 값들을 담은 리터럴 객체 형태로 준비하면 된다.
그 이상의 어떤 동작을 수해해야 한다면 필요한 메서드를 담은 객체를 생성하면 된다. 
특이 케이스 객체는 이를 캡슐화한 클래스가 반환하도록 만들 수도 있고, 변환을 거쳐 데이터 구조에 추가시키는 
형태로 될 수 있다.
>널은 특이 케이스로 처리해야 할 때가 많다. 그래서 이 패턴을 널 객체 패턴이라고 한다.


```javascript
// before
if (aCustomer === "미확인 고객") customerName = "거주자";

// after
class UnknownCustomer {
  get name() { return "거주자";}
```


### 10.6 어서션 추가하기
Introduce Assertion

>어서션(assertion:단언, 확언)은 항상 참이라고 가정하는 조건부 문장으로, 어서션이 실패했다는 건
>프로그래머가 잘 못했다는 뜻이다.
>어서션 실패는 시스템의 다른 부분에서는 절대 검사하지 않아야 하며, 어서션이 있고 없고가 
>프로그램 기능의 정상 동작에 아무런 영향을 주지 않도록 작성돼야 한다.
>

```javascript
// before
if (this.discountRate)
  base = base - (this.discountRate * base);

// after
assert(this.discountRate >= 0);
if (this.discountRate)
  base = base - (this.discountRate * base);
```

```javascript
// 할인율이 항상 양수라는 가정이 깔려 있다. 
applyDiscount(aNumber){
  return (this.discountRate)
    ? aNumber - (this.discountRate * aNumber)
    : aNumber;
}


⬇️
applyDiscount(aNumber){
  if(!this.discountRate) return aNumber;
  else {
    assert(this.discountRate >= 0);
    return aNumber - (this.discountRate * aNumber);
  }
}

⬇
set discountRate(aNumber){
  assert(null === aNumber || aNumber >= 0);
  this._discountRate = aNumber;
}
```


### 10.7 제어 플래그를 탈출문으로 바꾸기
Replace Control Flag with Break

>제어 플래그란 코드의 동작을 변경하는 데 사용되는 변수를 말하며, 어딘가에서 값을 계산해
>제어 플래그에 설정한 후 다른 어딘가의 조건문에서 검사하는 형태로 쓰인다.
>이런 코드도 악취가 난다.
>제어 플래그의 주 서식지는 반복문 안이다. break문이나 continue문 활용에 익숙하지 않은
>사람이 심어놓기도 하고, 함수의 return문을 하나로 유지하고자 노력하는 사람이 심기도 한다.

```javascript
// before
for (const p of people) {
  if (!found) {
    if (p === "조커") {
      sendAlert();
      found = true;
    }
...

// after
for (const p of people) {
  if (p === "조커") {
    sendAlert();
    break;
    }
...
```

* 절차
1. 제어 플래그를 사용하는 코드를 함수로 추출할지 고려한다.
2. 제어 플래그를 갱신하는 코드 각가을 적절한 제어문으로 바꾼다. 하나 바꿀 때마다 테스트한다.
3. 모두 수정했다면 제어 플래그를 제거한다.

```javascript
let found = false;
for (const p of people) {
  if (!fount) {
    if ( p === "조커" ) {
      sendAlert();
      fount = true;
    }
    if ( p === "사루만" ) {
      sendAlert();
      found = true;
    }
  }
}

⬇️
function checkForMiscreants(people) {
  for (const p of people) {
    if ( p === "조커 ) {
      sendAlert();
      return;
    }
    if ( p === "사루만 ) {
      sendAlert();
      return;
    }
  }
}
```
<br/><br/>


\[더 가다듬기\]

```javascript
function checkForMiscreants(people) {
  if (people.some(p => ["조커", "사루만"].includes(p))) sendAlert();
}
```


<br/><br/>

### REFERENCE
📖 [Slowly velog.io](https://velog.io/@billion109/%EB%A6%AC%ED%8C%A9%ED%84%B0%EB%A7%81-10.-%EC%A1%B0%EA%B1%B4%EB%B6%80-%EB%A1%9C%EC%A7%81-%EA%B0%84%EC%86%8C%ED%99%94)
