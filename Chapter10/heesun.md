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

### 10.5 특이 케이스 추가하기
Introduce Special Case

>데이터 구조의 특정 값을 확인할 후 똑같은 동작을 수행하는 코드가 곳곳에 등장하는 경우,
>코드베이스에서 특정 값에 대해 똑같이 반응하는 코드가 여러 곳이라면 그 반응을 한 데로 
>모으는 게 효율적이다.
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

### 10.7 제어 플래그를 탈출문으로 바꾸기
Replace Control Flag with Break

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

### 


<br/><br/>

### REFERENCE
📖 [Slowly velog.io](https://velog.io/@billion109/%EB%A6%AC%ED%8C%A9%ED%84%B0%EB%A7%81-10.-%EC%A1%B0%EA%B1%B4%EB%B6%80-%EB%A1%9C%EC%A7%81-%EA%B0%84%EC%86%8C%ED%99%94)
