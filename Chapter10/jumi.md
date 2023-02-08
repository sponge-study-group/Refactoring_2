## 10. 조건부 로직 간소화

### 10.1 조건문 분해하기

<br />

```javascript
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
```

↓

```javascript
if (summer())
  charge = summerCharge();
else
  charge = regularCharge();
```

<br />

**배경**  
긴 함수에 있는 조건문은 어려움을 한층 가중시킨다.  
→ 해당 조건문의 의도를 파악하기 어렵다.  

거대한 코드 블록이 있다면  
→ 코드를 부위별로 분해  
→ 해체된 코드 덩어리들을 각 덩어리의 의도를 살린 이름의 함수 호출로 변경  
→ 전체적인 의도가 더 확실히 드러난다. 


**절차**  
① 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출

---

### 10.2 조건식 통합하기

<br />

```javascript
  if (anEmployee.seniority < 2) return 0;
  if (anEmployee.monthsDisabled > 12) return 0;
  if (anEmployee.isPartTime) return 0;
```

↓

```javascript
  if (isNotEligableForDisability()) return 0;

  function isNotEligableForDisability() {
    return ((anEmployee.seniority < 2)
            || (anEmployee.monthsDisabled > 12)
            || (anEmployee.isPartTime));
  }
```

<br />

**배경**  
비교하는 조건은 다르지만 결과로 수행하는 동작은 똑같은 코드들  
→ 조건 검사를 하나도 통합 

조건부 코드를 통합 시키는 중요한 이유  
1. 여러 조각으로 나뉜 조건들을 하나로 통합 → 하려는 일이 더 명확해진다.  
2. 이 작업을 통해 함수 추출하기로 이어질 가능성이 높다.

**절차**  
① 해당 조건식들 모두에 부수효과가 없는지 확인  
② 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합  
③ 테스트  
④ 조건이 하나만 남을 때까지 해당 과정 반복  
⑤ 하나로 합쳐진 조건식을 함수로 추출할지 고려  


---

### 10.3 중첩 조건문을 보호 구문으로 바꾸기

<br />

```javascript
function getPayAmount() {
  let result;
  if (isDead)
    result = deadAmount();
  else {
    if (isSeparated)
      result = separatedAmount();
    else {
      if (isRetired)
        result = retiredAmount();
      else
        result = normalPayAmount();
    }
  }
  return result;
}
```

↓

```javascript
function getPayAmount() {
  if (isDead) return deadAmount();
  if (isSeparated) return separatedAmount();
  if (isRetired) return retiredAmount();
  return normalPayAmount();
}
```

<br />

**배경**  
조건문
- 참인 경로와 거짓인 경로 모두 정상 동작으로 이어지는 형태  
- 한쪽만 정상인 형태 → 보호 구문

함수의 진입점과 반환점이 하나일 때 함수의 로직이 더 명백하지 않다면 하지 ×

**절차**  
① 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 변경  
② 테스트  
③ 해당 과정을 필요한 만큼 반복  
④ 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합


---

### 10.4 조건부 로직을 다형성으로 바꾸기

<br />

```javascript
switch (bird.type) {
  case '유럽 제비':
    return "보통";
  case '아프리카 제비':
    return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통";
  case '노르웨이 파랑 앵무':
    return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
  default:
    return "알 수 없다";
```

↓

```javascript
class EuropeanSwallow {
  get plumage() {
    return "보통";
  }

  ...
}
class AfricanSwallow {
  get plumage() {
     return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통";
  }
  
  ...
}
class NorwegianBlueParrot {
  get plumage() {
     return (this.voltage > 100) ? "그을렸다" : "예쁘다";
  }
  
  ...
}
```

<br />

**배경**  
복잡한 조건 로직은 프로그래밍에서 해석하기 가장 난해한 대상  
→ 클래스와 다형성을 이용하면 조건부 로직을 더 확실하게 분리할 수 있다.

1. 타입을 여러 개 만들고 각 타입이 조건부 로직을 자신만의 방식으로 처리하도록 구성
2. 기본 동작을 위한 case문과 그 변형 동작으로 구성된 로직  

다형성은 객체 지향 프로그래밍의 핵심  
→ 모든 조건부 로직을 다형성으로 대체해야 하는건 ×

**절차**  
① 다형적 동작을 표현하는 클래스가 없다면 팩터리 함수와 함께 만들어준다.  
② 호출하는 코드에서 팩터리 함수를 사용  
③ 조건부 로직 함수를 슈퍼클래스로 옮긴다.  
④ 서브클래스 중 하나를 선택. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다. 
조건부 문장 중 선택된 서브클래스의 해당하는 조건절을 서브클래스 메서드를 복사한 다음 적절히 수정.  
⑤ 같은 방식으로 각 조건절을 해당 서브클래스에서 메서드로 구현  
⑥ 슈퍼클래스 메서드에는 기본 동작 부분만 남긴다.  
→ 슈퍼클래스가 추상 클래스여야 한다면, 이 메서드를 추상으로 선언 or 서브클래스에서 처리해야 함을 알리는 에러를 던진다. 


---

### 10.5 특이 케이스 추가하기

<br />

```javascript
if (aCustomer === "미확인 고객") customerName = "거주자";
```

↓

```javascript
class UnknownCustomer {
  get name() { return "거주자"; }
}
```

<br />

**배경**  
데이터 구조의 특정 값을 확인한 후 똑같은 동작을 수행하는 코드가 곳곳에 등장하는 경우  
→ 중복 코드 중 하나  
→ 이를 한 데로 모으는 게 효율적

특이 케이스 패턴  
→ 공통 동작을 요소 하나에 모아서 사용하는 매커니즘  
→ 이 패턴을 활용하면 특이 케이스를 확인하는 코드 대부분을 단순함 함수 호출로 변경 가능  
→ null과 같은 특이 케이스

**절차**  
① 컨테이너에 특이 케이스인지를 검사하는 속성을 추가 → false를 반환하게 만든다.  
② 특이 케이스 객체를 생성. 이 객체는 특이 케이스인지를 검사하는 속성만 포함 → true를 반환하게 만든다.  
③ 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출. 해당 함수를 사용하도록 수정.  
④ 코드에 새로운 특이 케이스 대상을 추가. 함수의 반환 값으로 받거나 변환 함수를 적용.  
⑤ 특이 케이스를 검사하는 함수 본문을 수정 → 특이 케이스 객체의 속성 사용.  
⑥ 테스트  
⑦ **여러 함수를 클래스로 묶기**나 **여러 함수를 변환 함수로 묶기**를 적용 → 특이 케이스를 처리하는 공통 동작을 새로운 요소로 옮긴다.  
⑧ 특이 케이스 검사 함수를 이용하는 곳이 남아 있다면 검사 함수를 인라인

---

### 10.6 어서션 추가하기

<br />

```javascript
if (this.discountRate)
  base = base - (this.discountRate * base);
```

↓

```javascript
assert(this.discountRate >= 0);
if (this.discountRate)
  base = base - (this.discountRate * base);
```

<br />

**배경**  
특정 조건이 참일 때만 동작하는 코드  
→ 조건들이 항상 코드에 명시적으로 기술 ×  
→ **어서션을 이용**해서 코드 자체에 삽입하여 명시적으로 보여주는 것.

어서션  
→ 항상 참이라고 가정하는 조건부 문장  
→ 어서션 실패는 시스템의 다른 부분에서 절대 검사 ×  
→ 어서션 여부가 프로그램 기능 정상 동작에 영향을 주어선 ×  

어서션은 프로그램이 어떤 상태임을 가정한 채 실행되는지를 다른 개발에게 알려주는 소통 도구

**절차**  
① 참이라고 가정하는 조건이 보이면 그 조건을 명시하는 어서션을 추가.


---

### 10.7 제어 플래그를 탈출문으로 바꾸기

<br />

```javascript
for (const p of people) {
  if (!found) {
    if ( p === "조커") {
      sendAlert();
      found = true;
    }

    ...
  }
}
```

↓

```javascript
for (const p of people) {
  if ( p === "조커" ) {
    sendAlert();
    break;
  }

  ...
}
```

<br />

**배경**  
제어 플래그  
→ 코드의 동작을 변경하는 데 사용하는 변수   

어딘가에서 값을 계산해 제어 플래그에 설정한 후 다른 어딘가의 조건문에서 검사하는 행태  
→ 악취나는 코드  
→ 충분히 리팩터링으로 간소화할 수 ○

**절차**  
① 제어 플래그를 사용하는 코드를 함수로 추출할 지 고려  
② 제어 플래그를 갱신하는 코드 각각을 적절한 제어문(return, break, continue)으로 변경 → 변경할 때마다 테스트
③ 모두 수정했다면 제어 플래그 제거


---