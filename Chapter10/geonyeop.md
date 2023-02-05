# 조건부 로직 간소화

조건부 로직은 프로그램의 힘을 강화하는 데 크게 기여하지만, 안타깝게도 프로그램을 복잡하게 만드는 주요 원흉이기도 하다.

그래서 나는 조건부 로직을 이해하기 쉽게 바꾸는 리팩터링을 자주 한다.

복잡한 조건문에는 **조건문 분해하기**를, 논리적 조합을 명확하게 다듬는 데는 **중복 조건식 통합하기**를 적용한다.

함수의 핵심 로직에 본격적으로 들어가기 앞서 무언가를 검사해야 할 때는 **중첩 조건문을 보호 구문으로 바꾸기**를,

똑같은 분기 로직(주로 switch 문)이 여러 곳에 등장했다면 **조건부 로직을 다형성으로 바꾸기**를 적용한다.

null과 같은 특이 케이스를 처리하는 데도 조건부 로직이 흔히 쓰인다.

이 처리 로직이 거의 똑같다면 **특이 케이스 추가하기(null 객체 추가하기** 라고도 한다)를 적용해 코드 중복을 상당히 줄일 수 있다.

한편, (내가 조건절 없애기를 매우 좋아하는 건 사실이지만) 프로그램의 상태를 확인하고 그 결과에 따라 다르게 동작해야 하는 상황이면

어서션 추가하기가 도움이 된다.

제어 플래그를 이용해 코드 동작 흐름을 변경하는 코드는 대부분 **제어 플래그를 탈출문으로 바꾸기**를 적용해 더 간소화할 수 있다.

## 조건문 분해하기(Decompose Conditional)

```jsx
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else charge = quantity * plan.regularRate + plan.regularServiceCharge;
```

```jsx
if (summber()) charge = summerCharge();
else charge = regularCharge();
```

### 배경

복잡한 조건부 로직은 프로그램을 복잡하게 만드는 가장 흔한 원흉에 속한다.

다양한 조건, 그에 따라 동작도 다양한 코드를 작성하면 순식간에 꽤 긴 함수가 탄생한다.

긴 함수는 그 자체로 읽기가 어렵지만, 조건문은 그 어려움을 한층 가중시킨다.

조건을 검사하고 그 결과에 따른 동작을 표현한 코드는 무슨 일이 일어나는지 이야기해주지만

‘왜’ 일어나는지는 제대로 말해주지 않을 때가 많은 것이 문제다.

거대한 코드 블록이 주어지면 코드를 부위별로 분해한 다음 해체된 코드 덩어리들을 각 덩어리의 의도를 살린 이름의 함수 호출로 바꿔주자.

그러면 전체적인 의도가 더 확실히 드러난다. 조건문이 보이면 나는 조건식과 각 조건절에 이 작업을 해주길 좋아한다.

이렇게 하면 해당 조건이 무엇인지 강조하고, 그래서 무엇을 분기했는지가 명확해진다.

분기한 이유 역시 더 명확해진다.

이 리팩터링은 자신의 코드에 함수 추출하기를 적용하는 한 사례로 볼 수도 있다.

하지만 연습용으로 아주 훌륭하기 때문에 독립된 리팩터링으로 추가했다.

---

### 절차

1. 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출한다.

---

### 예시

여름철이면 할인율이 달라지는 어떤 서비스의 요금을 계산한다고 해보자.

```jsx
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else charge = quantity * plan.regularRate + plan.regularServiceCharge;
```

[1] 우선 조건 부분(조건식)을 별도 함수로 추출하자.

```jsx
if (summer()) charge = quantity * plan.summerRate;
else charge = quantity * plan.regularRate + plan.regularServiceCharge;

function summer() {
  return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
}
```

그런 다음 조건이 만족했을 때의 로직도 또 다른 함수로 추출한다.

```jsx
if (summer()) charge = summerCharge();
else charge = quantity * plan.regularRate + plan.regularServiceCharge;

function summer() {
  return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
}

function summerCharge() {
  return quantity * plan.summerRate;
}
```

마지막으로 else절도 별도 함수로 추출한다.

```jsx
if (summer()) charge = summerCharge();
else charge = regularCharge();

function summer() {
  return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
}

function summerCharge() {
  return quantity * plan.summerRate;
}

function regularCharge() {
  return quantity * plan.regularRate + plan.regularServiceCharge;
}
```

모두 끝났다면 취향에 따라 전체 조건문을 3항 연산자로 바꿔줄 수도 있다.

```jsx
charge = summer() ? summerCharge() : regularCharge();

function summer() {
  return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
}

function summerCharge() {
  return quantity * plan.summerRate;
}

function regularCharge() {
  return quantity * plan.regularRate + plan.regularServiceCharge;
}
```

---

## 조건식 통합하기(Consolidate Conditional Expression)

```jsx
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;
```

```jsx
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisability() {
  return (
    anEmployee.seniority < 2 ||
    anEmployee.monthsDisabled > 12 ||
    anEmployee.isPartTime
  );
}
```

### 배경

비교하는 조건은 다르지만 그 결과로 수행하는 동작은 똑같은 코드들이 더러 있는데,

어차피 같은 일을 할 거라면 조건 검사도 하나로 통합하는 게 낫다.

이럴 때 ‘and’연산자와 ‘or’연산자를 사용하면 여러 개의 비교 로직을 하나로 합칠 수 있다.

조건부 코드를 통합하는게 중요한 이유는 두 가지다.

1. 여러 조각으로 나뉜 조건들을 하나로 통합함으로써 내가 하려는 일이 더 명확해진다.

   나눠서 순서대로 비교해도 결과는 같지만, 읽는 사람은 독립된 검사들이 우연히 함께 나열된 것으로 오해할 수 있다.

2. 이 작업이 함수 추출하기까지 이어질 가능성이 높기 때문이다.

   복잡한 조건식을 함수로 추출하면 코드의 의도가 훨씬 분명하게 드러나는 경우가 많다.

   함수 추출하기는 ‘무엇’을 하는지를 기술하던 코드를 ‘왜’ 하는지를 말해주는 코드로 바꿔주는 효과적인 도구다.

조건식을 통합해야 하는 이유는 이 리팩터링을 하지 말아야 하는 이유도 설명해준다.

하나의 검사라고 생각할 수 없는, 다시 말해 진짜로 독립된 검사들이라고 판단되면 이 리팩터링을 해서는 안 된다.

---

### 절차

1. 해당 조건식들 모두에 부수효과가 없는지 확인한다.

   → 부수효과가 있는 조건식들에는 질의 함수와 변경 함수 분리하기를 먼저 적용한다.

2. 조건문 두 개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다.

   → 순차적으로 이뤄지는(레벨이 같은) 조건문은 or로 결합하고, 중첩된 조건문은 and로 결합한다.

3. 테스트한다.
4. 조건이 하나만 남을 때까지 2~3 과정을 반복한다.
5. 하나로 합쳐진 조건식을 함수로 추출할지 고려해본다.

---

### 예시

- or 사용하기
  코드를 훑다가 다음 코드를 발견했다고 하자.

  ```jsx
  function disabilityAmount(anEmployee) {
    if (anEmployee.seniority < 2) return 0;
    if (anEmployee.monthsDisabled > 12) return 0;
    if (anEmployee.isPartTime) return 0;
    // 장애 수당 계산
  }
  ```

  똑같은 결과로 이어지는 조건 검사가 순차적으로 진행되고 있다.
  [2] 결과로 행하는 동작이 같으므로 이 조건들을 하나의 식으로 통합해보자.
  이처럼 순차적인 경우엔 or 연산자를 이용하면 된다.

  ```jsx
  function disabilityAmount(anEmployee) {
    if (
      anEmployee.seniority < 2 ||
      anEmployee.monthsDisabled > 12 ||
      anEmployee.isPartTime
    )
      return 0;
    // 장애 수당 계산
  }
  ```

  [5] 모든 조건을 통합했다면 최종 조건식을 함수로 추출해볼 수 있다.

  ```jsx
  function disabilityAmount(anEmployee) {
  	if(isNotEligibleForDisability()) return 0;
  	// 장애 수당 계산
  }

  function isNotEligibleForDisability() { // 장애 수당 적용 여부 확인
  	return 	if((anEmployee.seniority < 2)
  						 || (anEmployee.monthsDisabled > 12)
  						 || (anEmployee.isPartTime)) return 0;
  }
  ```

- and 사용하기
  앞에서는 식들을 or로 결합하는 예를 보여줬는데, if문이 중첩되어 나오면 and를 사용해야 한다.
  ```jsx
  if (anEmployee.onVacation) if (anEmployee.seniority > 10) return 1;
  return 0.5;
  ```
  이 조건들을 and 연산자로 결합해보자.
  ```jsx
  if (anEmployee.onVacation && anEmployee.seniority > 10) return 1;
  return 0.5;
  ```
  두 경우가 복합된 상황에서는 and와 or 연산자를 적절히 섞어 결합하자. 이처럼 복잡한 상황이라면 대체로 코드가 지저분하다.
  따라서 함수 추출하기를 적절히 활용하여 전체를 더 이해하기 쉽게 만들어주면 좋다.

---

## 중첩 조건문을 보호 구문으로 만들기(Reploace Nested Conditional with Guard Clauses)

```jsx
function getPayAmount() {
  let result;
  if (isDead) result = deadAmount();
  else {
    if (isSeparated) result = separatedAmount();
    else {
      if (isRetired) result = retiredAmount();
      else result = normalPayAmount();
    }
  }
  return result;
}
```

```jsx
function getPayAmount() {
  if (isDead) return deadAmount();
  if (isSeparated) return separatedAmount();
  if (isRetired) return retiredAmount();
  return normalPayAmount();
}
```

### 배경

조건문은 주로 두 가지 형태로 쓰인다. 참인 경로와 거짓인 경로 모두 정상 동작으로 이어지는 형태와, 한쪽만 정상인 형태다.

두 형태는 의도하는 바가 서로 다르므로 그 의도가 코드에 드러나야 한다.

나는 두 경로 모두 정상 동작이라면 if와 else절을 사용한다.

한쪽만 정상이라면 비정상 조건을 if에서 검사한 다음, 조건이 참이면(비정상이면) 함수에서 빠져나온다.

두 번째 검사 형태를 흔히 보고 구문(guard clauses)이라고 한다.

중첩 조건문을 보호 구문으로 바꾸기 리팩터링의 핵심은 의도를 부각하는 데 있다.

나는 if-then-else 구조를 사용할 때 if절과 else절에 똑같은 무게를 두어, 코드를 읽는 이에게 양 갈래가 똑같이 중요하다는 뜻을 전달한다.

이와 달리, 보호 구문은 “이건 이 함수의 핵심이 아니다. 이 일이 일어나면 무언가 조치를 취한 후 함수에서 빠져나온다”라고 이야기한다.

함수의 진입점과 반환점이 하나여야 한다고 배운 프로그래머와 함께 일하다 보면 이 리팩터링을 자주 사용하게 된다.

진입점이 하나라는 조건은 최신 프로그래밍 언어에서는 강제된다.

그런데 반환점이 하나여야 한다는 규칙은, 정말이지 유용하지 않다.

코드에서는 명확함이 핵심이다. 반환점이 하나일 때 함수의 로직이 더 명백하다면 그렇게 하자.

그렇지 않다면 하지 말자.

---

### 절차

1. 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 바꾼다.
2. 테스트한다.
3. 1~2 과정을 필요한 만큼 반복한다.
4. 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합한다.

---

### 예시

- 기본 예시

  ```jsx
  function payAmount(employee) {
    let result;
    if (employee.isSeparated) {
      // 퇴사한 직원인가 ?
      result = { amount: 0, reasonCode: "SEP" };
    } else {
      if (employee.isRetired) {
        result = { amount: 0, reasonCode: "RET" };
      } else {
        // 급여 계산 로직
        lorem.ipsum(dolor.sitAmet);
        consectetur(adipiscing).elit();
        sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
        ut.enim.ad(minim.veniam);
        result = someFinalComputation();
      }
    }
    return result;
  }
  ```

  이 코드는 실제로 벌어지는 중요한 일들이 중첩된 조건들에 가려서 잘 보이지 않는다.
  이 코드가 진짜 의도한 일은 모든 조건이 거짓일 때만 실행되기 때문이다.
  이 상황에서는 보호 구문을 사용하면 코드의 의도가 더 잘 드러난다.
  다른 리팩터링과 마찬가지로 나는 단계를 잘게 나눠 하나씩 밟아가길 좋아한다.
  [1] 그러니 최상위 조건부터 보호 구문으로 바꿔보자.

  ```jsx
  function payAmount(employee) {
    let result;
    if (employee.isSeparated) return { amount: 0, reasonCode: "SEP" };
    if (employee.isRetired) {
      result = { amount: 0, reasonCode: "RET" };
    } else {
      // 급여 계산 로직
      lorem.ipsum(dolor.sitAmet);
      consectetur(adipiscing).elit();
      sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
      ut.enim.ad(minim.veniam);
      result = someFinalComputation();
    }
    return result;
  }
  ```

  [2] 변경 후 테스트하고 [3] 다음 조건으로 넘어간다.

  ```jsx
  function payAmount(employee) {
    let result;
    if (employee.isSeparated) return { amount: 0, reasonCode: "SEP" };
    if (employee.isRetired) return { amount: 0, reasonCode: "RET" };

    // 급여 계산 로직
    lorem.ipsum(dolor.sitAmet);
    consectetur(adipiscing).elit();
    sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
    ut.enim.ad(minim.veniam);
    result = someFinalComputation();

    return result;
  }
  ```

  여기까지 왔다면 result 변수는 아무 일도 하지 않으므로 제거하자.

  ```jsx
  function payAmount(employee) {
    if (employee.isSeparated) return { amount: 0, reasonCode: "SEP" };
    if (employee.isRetired) return { amount: 0, reasonCode: "RET" };

    // 급여 계산 로직
    lorem.ipsum(dolor.sitAmet);
    consectetur(adipiscing).elit();
    sed.do.eiusmod = tempor.incididunt.ut(labore) && dolore(magna.aliqua);
    ut.enim.ad(minim.veniam);

    return someFinalComputation();
  }
  ```

  가변 변수를 제거하면 자다가도 떡이 생긴다는 사실을 항상 기억하자 !

- 조건 반대로 만들기
  이 책 초판의 원고를 검토하던 조슈아 케리에프스키가 이 리팩터링을 수행할 때는 조건식을 반대로 만들어 적용하는 경우도 많다고 알려왔다.
  그리고 고맙게도 예시까지 만들어 보내주었다.
  ```jsx
  function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital > 0) {
      if (anInstrument.interestRate > 0 && anInstrument.duration > 0) {
        result =
          (anInstrument.income / anInstrument.duration) *
          anInstrument.adjustmentFactor;
      }
    }
    return result;
  }
  ```
  역시 한 번에 하나씩 수정한다. 다만 이번에는 보호 구문을 추가하면서 조건을 역으로 바꿀 것이다.
  ```jsx
  function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital <= 0) return result;
    if (anInstrument.interestRate > 0 && anInstrument.duration > 0) {
      result =
        (anInstrument.income / anInstrument.duration) *
        anInstrument.adjustmentFactor;
    }
    return result;
  }
  ```
  다음 조건은 살짝 더 복잡하므로 두 단계로 나눠 진행하겠다.
  먼저 간단히 not 연산자를 추가한다.
  ```jsx
  function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital <= 0) return result;
    if (!(anInstrument.interestRate > 0 && anInstrument.duration > 0))
      return result;
    result =
      (anInstrument.income / anInstrument.duration) *
      anInstrument.adjustmentFactor;
    return result;
  }
  ```
  이처럼 조건식 안에 not이 있으면 영 개운치 않으니 다음처럼 간소화한다.
  ```jsx
  function adjustedCapital(anInstrument) {
    let result = 0;
    if (anInstrument.capital <= 0) return result;
    if (anInstrument.interestRate <= 0 || anInstrument.duration <= 0)
      return result;
    result =
      (anInstrument.income / anInstrument.duration) *
      anInstrument.adjustmentFactor;
    return result;
  }
  ```
  두 if문 모두 같은 결과를 내는 조건을 포함하니 조건식을 통합한다.
  ```jsx
  function adjustedCapital(anInstrument) {
    let result = 0;
    if (
      anInstrument.capital <= 0 ||
      anInstrument.interestRate <= 0 ||
      anInstrument.duration <= 0
    )
      return result;
    result =
      (anInstrument.income / anInstrument.duration) *
      anInstrument.adjustmentFactor;
    return result;
  }
  ```
  여기서 result 변수는 두 가지 일을 한다. 처음 설정한 값 0은 보호 구문이 발동했을 때 반환할 값이다.
  두 번째로 설정한 값은 계산의 최종 결과다. 이 변수를 제거하면 변수 하나가 두 가지 용도로 쓰이는 상황이 사라진다.
  ```jsx
  function adjustedCapital(anInstrument) {
    if (
      anInstrument.capital <= 0 ||
      anInstrument.interestRate <= 0 ||
      anInstrument.duration <= 0
    )
      return 0;
    return (
      (anInstrument.income / anInstrument.duration) *
      anInstrument.adjustmentFactor
    );
  }
  ```

---

## 조건부 로직을 다형성으로 바꾸기(Replace Conditional with Polymorphism)

```jsx
switch (bird.type) {
  case "유럽 제비":
    return "보통이다";
  case "아프리카 제비":
    return bird.numberOfCoconuts > 2 ? "지쳤다" : "보통이다";
  case "노르웨이 파랑 앵무":
    return bird.voltage > 100 ? "그을렸다" : "예쁘다";
  default:
    return "알 수 없다";
}
```

```jsx
class EuropeanSwallow {
	get plumage() {
		return "보통이다";
	}
	...
}

class AfricanSwallow {
	get plumage() {
		return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
	}
	...
}

class NorwegianBlueParrot {
	get plumage() {
		return (this.voltage > 100) ? "그을렸다" : "예쁘다";
	}
}
```

### 배경

복잡한 조건부 로직은 프로그래밍에서 해석하기 가장 난해한 대상에 속한다.

그래서 나는 조건부 로직을 직관적으로 구조화할 방법을 항상 고민한다.

종종 더 높은 수준의 개념을 도입해 이 조건들을 분리해낼 수 있다.

조건문 구조를 그대로 둔 채 해결될 때도 있지만, 클래스와 다형성을 이용하면 더 확실하게 분리할 수도 있다.

흔한 예로, 타입을 여러 개 만들고 각 타입이 조건부 로직을 자신만의 방식으로 처리하도록 구성하는 방법이 있다.

책, 음악, 음식은 다르게 처리해야 한다. 왜 ? 타입이 다르니까!

타입을 기준으로 분기하는 switch문이 포함된 함수가 여러 개 보인다면 분명 이러한 상황이다.

이런 경우 case별로 클래스를 하나씩 만들어 공통 switch 로직의 중복을 없앨 수 있다.

다형성을 활용하여 어떻게 동작할지를 각 타입이 알아서 처리하도록 하면 된다.

또 다른 예로, 기본 동작을 위한 case문과 그 변형 동작으로 구성된 로직을 떠올릴 수 있다.

기본 동작은 가장 일반적이거나 가장 직관적인 동작일 것이다.

먼저 이 로직을 슈퍼클래스로 넣어서 변형 동작에 신경쓰지 않고 기본에 집중하게 한다.

그런 다음 변형 동작을 뜻하는 case들을 각각의 서브클래스로 만든다.

이 서브클래스들은 기본 동작과의 차이를 표현하는 코드로 채워질 것이다.

다형성은 객체 지향 프로그래밍의 핵심이다. 하지만 (유용한 기능들이 늘 그렇듯) 남용하기 쉽다.

실제로 모든 조건부 로직을 다형성으로 대체해야 한다고 주장하는 사람도 만난 적이 있다.

나는 그 견해에는 동의하지 않는다. 조건부 로직 대부분은 기본 조건문인 if/else와 switch/case로 이뤄지기 떄문이다.

하지만 앞서 이야기한 방법들로 개선할 수 있는 복잡한 조건부 로직을 발견하면 다형성이 막강한 도구임을 깨닫게 된다.

---

### 절차

1. 다형적 동작을 표현하는 클래스들이 아직 없다면 만들어준다. 이왕이면 적합한 인스턴스를 알아서 만들어 반환하는 팩터리 함수도 함께 만든다.
2. 호출하는 코드에서 팩터리 함수를 사용하게 한다.
3. 조건부 로직 함수를 슈퍼클래스로 옮긴다.

   → 조건부 로직이 온전한 함수로 분리되어 있지 않다면 먼저 함수로 추출한다.

4. 서브 클래스 중 하나를 선택한다. 서브클래스에서 슈퍼클래스의 조건부 로직 메서드를 오버라이드한다.

   조건부 문장 중 선택된 서브클래스에 해당하는 조건절을 서브클래스 메서드로 복사한 다음 적절히 수정한다.

5. 같은 방식으로 각 조건절을 해당 서브클레스에서 메서드로 구현한다.
6. 슈퍼클래스 메서드에는 기본 동작 부분만 남긴다.

   혹은 슈퍼클래스가 추상 클래스여야 한다면, 이 메서드를 추상으로 선언하거나 서브클래스에서 처리해야 함을 알리는 에러를 던진다.

---

### 예시

- 기본 예시
  다양한 새를 키우는 친구가 있는데, 새의 종에 따른 비행 속도와 깃털 상태를 알고 싶어 한다.
  그래서 이 정보를 알려주는 작은 프로그램을 짜봤다.

  ```jsx
  function plumages(birds){
    return new Map(birds.map(b => [b.name, plumage(b)]));
  }

  function speeds(birds) {
    return new Map(birds.map(b -> [b.name, airSpeedVelocity(b)]));
  }

  function plumage(bird) { // 깃털 상태
    switch(bird.type){
      case '유럽 제비' :
        return "보통이다";
      case '아프리카 제비' :
        return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
      case '노르웨이 파랑 앵무' :
        return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
      default :
        return "알 수 없다";
    }
  }

  function airSpeedVelocity(bird) { // 비행 속도
    switch(bird.type) {
      case '유럽 제비' :
        return 35;
      case '아프리카 제비' :
        return 40 - 2 * bird.numberOfCoconuts;
      case '노르웨이 파랑 앵무' :
        return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
      default :
        return null;
    }
  }
  ```

  - 코드 배경
    배경 이야기를 모른 채 이 코드를 보면 무슨 의미인지 도통 이해되지 않을 것이다.
    ‘코코넛 수나 전압이 깃털과 무슨 관계가 ? 못(nail) 이야기는 또 뭔가 ?’ 의문을 풀려면 두 가지 이야기를 알아야 한다.
    ```
    **<몬티 파이튼과 성배>**
    영국의 전설적인 코미디 그룹 몬티 파이튼이 제작한 유럽 중세 배경의 영화다.
    당시 특수효과가 미비하던 시절이라 라디오에서 말발굽 소리가 필요할 때 빈 코코넛 껍데기 두 개를 부딪혀 흉내 냈다고 하는데,
    이 영화는 어차피 코미디 장르기도 하고 제작비를 줄일 겸 하여 등장인물들이 실제로 코코넛 껍데기를 흔들고 다니며 말을 타는 척한다.
    이 점을 작중에서 재미 요소로 활용하여, 성을 찾은 아서왕에게 경비병이 "열대작물인 코코넛이 어디서 났나"고 묻고,
    "제비가 물어왔다", "코코넛을 물고 비행 속도를 유지하는게 가능하겠나" , "유럽 제비는 안 되겠지만 아프리카 제비라면 되지 않을까"
    등의 엉뚱한 이야기가 나온다.
    ```
    ```
    **<죽은 앵무새>**
    몬티 파이튼이 등장한 영국 TV쇼(몬티 파이튼의 비행 서커스)에서 가장 유명한 에피소드로, 애완동물 가게에서 병든 앵무를 팔아서
    금세 죽어버렸다며 항의하는 고객과 온갖 핑계를 대며 앵무가 살아 있다고 우기는 점원의 이야기다.
    예시 코드와 관련한 대사만 추려보면 "앵무가 움직이지 않는다", "쉬는 중이다. 깃털도 예쁘고",
    "처음에는 어떻게 서 있었나 봤더니 못으로 고정되어 있더라", "당연하다. 그러지 않으면 난리를 치다가 펑 터져버린다",
    "4백만 볼트를 걸지 않는 이상 터질 일 없다. 그냥 죽은 거다" 등의 이야기를 주고 받는다.
    ```
    새 종류에 따라 다르게 동작하는 함수가 몇 개 보이니 종류별 클래스를 만들어서 각각에 맞는 동작을 표현하면 좋을 것 같다.
    [3] 가장 먼저 airSpeedVelocity()와 plumage()를 Bird라는 클래스로 묶어보자(여러 함수를 클래스로 묶기)

  ```jsx
  function plumages(bird) {
    return new Bird(bird).plumage;
  }

  function speeds(bird) {
    return new Bird(bird).airSpeedVelocity;
  }

  class Bird {
    constructor(birdObject) {
      Object.assign(this, birdObject);
    }

    get plumage() {
      switch (this.type) {
        case "유럽 제비":
          return "보통이다";
        case "아프리카 제비":
          return this.numberOfCoconuts > 2 ? "지쳤다" : "보통이다";
        case "노르웨이 파랑 앵무":
          return this.voltage > 100 ? "그을렸다" : "예쁘다";
        default:
          return "알 수 없다";
      }
    }

    get airSpeedVelocity() {
      switch (this.type) {
        case "유럽 제비":
          return 35;
        case "아프리카 제비":
          return 40 - 2 * this.numberOfCoconuts;
        case "노르웨이 파랑 앵무":
          return this.isNailed ? 0 : 10 + this.voltage / 10;
        default:
          return null;
      }
    }
  }
  ```

  [1] 이제 종별 서브클래스를 만든다. 적합한 서브클래스의 인스턴스를 만들어줄 팩터리 함수도 잊지말자.
  [2] 그러고 나서 객체를 얻을 때 팩터리 함수를 사용하도록 수정한다.

  ```jsx
  function plumages(bird) {
    return createBird(bird).plumage;
  }

  function speeds(bird) {
    return cratedBird(bird).airSpeedVelocity;
  }

  function createBird(bird) {
    switch (bird.type) {
      case "유럽 제비":
        return new EuropeanSwallow(bird);
      case "아프리카 제비":
        return new AfricanSwallow(bird);
      case "노르웨이 파랑 앵무":
        return new NorwegianBlueParrot(bird);
      default:
        return new Bird(bird);
    }
  }

  class EuropeanSwallow extends Bird {}

  class AfricanSwallow extends Bird {}

  class NorwegianBlueParrot extends Bird {}
  ```

  필요한 클래스 구조가 준비되었으니 두 조건부 메서드를 처리할 차례다. plumage() 부터 시작하자.
  [4] switch문의 절 하나를 선택해 해당 서브클래스에서 오버라이드한다. 첫 번째 절의 유럽 제비를 선택해봤다.

  ```jsx
  ...

  class EuropeanSwallow extends Bird {
    get plumage() {
      return "보통이다";
    }
  }

  ...

  class Bird {
    constructor(birdObject) {
      Object.assign(this, birdObject);
    }

    get plumage() {
      switch (this.type) {
        case "유럽 제비":
          throw "오류 발생";
  			...
      }
    }
  }
  ```

  나는 완벽주의자이니 슈퍼클래스의 조건문에 throw문을 추가했다.
  이 시점에서 컴파일하고 테스트해보자.[5] 잘 동작한다면 다음 조건절을 처리한다.

  ```jsx
  class AfricanSwallow extends Bird {
    get plumage() {
      return this.numberOfCoconuts > 2 ? "지쳤다" : "보통이다";
    }
  }

  class NorwegianBlueParrot extends Bird {
    get plumage() {
      return this.voltage > 100 ? "그을렸다" : "예쁘다";
    }
  }
  ```

  [6] 슈퍼클래스의 메서드는 기본 동작용으로 남겨놓는다.

  ```jsx
  class Bird {
  	get plumage() {
  		return "알 수 없다.";
  	}
  	...
  }
  ```

  똑같은 과정을 airSpeedVelocity()에도 수행한다. 다 끝내면 코드의 모습이 다음처럼 변해 있을 것이다.
  (참고로 최상위 함수인 airSpeedVelocity()와 plumage()는 인라인시켰다).

  ```jsx
  function plumages(birds) {
    return new Map(
      birds.map((b) => createBird(b)).map((bird) => [bird.name, bird.plumage])
    );
  }

  function speeds(birds) {
    return new Map(
      birds
        .map((b) => createBird(b))
        .map((bird) => [bird.name, bird.airSpeedVelocity])
    );
  }

  function createBird(bird) {
    switch (bird.type) {
      case "유럽 제비":
        return new EuropeanSwallow(bird);
      case "아프리카 제비":
        return new AfricanSwallow(bird);
      case "노르웨이 파랑 앵무":
        return new NorwegianBlueParrot(bird);
      default:
        return new Bird(bird);
    }
  }

  class Bird {
    constructor(birdObject) {
      Object.assign(this, birdObject);
    }

    get plumage() {
      return "알 수 없다";
    }

    get airSpeedVelocity() {
      return null;
    }
  }

  class EuropeanSwallow extends Bird {
    get plumage() {
      return "보통이다";
    }

    get airSpeedVelocity() {
      return 35;
    }
  }

  class AfricanSwallow extends Bird {
    get plumage() {
      return this.numberOfCoconuts > 2 ? "지쳤다" : "보통이다";
    }

    get airSpeedVelocity() {
      return 40 - 2 * this.numberOfCoconuts;
    }
  }

  class NorwegianBlueParrot extends Bird {
    get plumage() {
      return this.voltage > 100 ? "그을렸다" : "예쁘다";
    }

    get airSpeedVelocity() {
      return this.isNailed ? 0 : 10 + this.voltage / 10;
    }
  }
  ```

  최종 코드를 보니 슈퍼클래스인 Bird는 없어도 괜찮아 보인다. 자바스크립트에서는 타입 계층 구조 없이도 다형성을 표현할 수 있다.
  객체가 적절한 이름의 메서드만 구현하고 있다면 아무 문제없이 같은 타임으로 취급하기 때문이다.(이를 덕 타이핑(duck typing)이라 한다.
  하지만 이번 예에서는 슈퍼클래스가 클래스들의 관계를 잘 설명해주기 때문에 그대로 두기로 했다.

- 변형 동작을 다형성으로 표현하기
    <aside>
    💡 이번 예시는 ‘절차’의 과정과 상당히 다르게 진행되어 번호를 표기하지 않았다.
    
    </aside>
    
    앞의 예에서는 계층 구조를 정확히 새의 종 분류에 맞게 구성했다.
    
    많은 교재에서 서브클래싱과 다형성을 설명하는 전형적인 방식이다. 하지만 상속이 이렇게만 쓰이는 것은 아니다.
    
    아니, 심지어 가장 흔하거나 최선인 방식도 아닐 것이다.
    
    또 다른 쓰임새로, 거의 똑같은 객체지만 다른 부분도 있음을 표현할 때도 상속을 쓴다.
    
    이러한 예로, 신용 평가 기관에서 선박의 항해 투자 등급을 계산하는 코드를 생각해보자.
    
    평가 기관은 위험요소와 잠재 수익에 영향을 주는 다양한 요인을 기초로 항해 등급을 ‘A’와 ‘B’로 나눈다.
    
    위험요소로는 항해 경로의 자연조건과 선장의 항해 이력을 고려한다.
    
    ```jsx
    function rating(voyage, history) { // 투자 등급
      const vpf = voyageProfitFactor(voyage, history);
      const vr = voyageRisk(voyage);
      const chr = captainHistoryRisk(voyage, history);
      if(vpf * 3 > (vr + chr * 2)) return "A";
      else return "B";
    }
    
    function voyageRisk(voyage) { // 항해 경로 위험요소
      let result = 1;
      if (voyage.length > 4) result += 2;
      if(voyage.length > 8) result += voyage.length - 8;
      if(["중국", "동인도"].includes(voyage.zone)) result += 4;
      return Math.max(result, 0);
    }
    
    function captainHistoryRisk(voyage, history) { // 선장의 항해 이력 위험요소
      let result = 1;
      if(history.length < 5) result += 4;
      result += history.filter(v => v.profit < 0).length;
      if (voyage.zone === "중국" && hasChina(history)) result -= 2;
      return Math.max(result, 0);
    }
    
    function hasChina(history) { // 중국을 경유하는가 ?
      return history.some(v => "중국" === v.zone);
    }
    
    function voyageProfitFactor(voyage, history) { // 수익 요인
      let result = 2;
      if(voyage.zone === "중국") result += 1;
      if(voyage.zone === "동인도") result += 1;
      if(voyage.zone === "중국" && hasChina(history)){
        result += 3;
        if(history.length > 10) result += 1;
        if(voyage.length > 12) result += 1;
        if(voyage.length > 18) result -= 1;
      }else{
        if(history.length > 8) result += 1;
        if(voyage.length > 14) result -= 1;
      }
      return result;
    }
    ```
    
    votageRisk()와 captainHistoryRisk() 함수의 점수는 위험요소에, votageProfitFactor() 점수는 잠재 수익에 반영된다.
    
    rating() 함수는 두 값을 종합하여 요청한 항해의 최종 등급을 계산한다.
    
    호출하는 쪽 코드가 다음과 같다고 해보자.
    
    ```jsx
    const voyage = {zone : "제주도", length : 10};
    const history = [
    	{zone : "제주도", profit : 5}
    	, {zone : "서인도", profit : 15}
      , {zone : "중국", profit : -2}
      , {zone : "서아프리카", profit : 7}
    ]
    
    const myRating = rating(voyage, history);
    ```
    
    여기서 주목할 부분은 두 곳으로, 중국까지 항해해본 선장이 중국을 경유해 항해할 때를 다루는 조건부 로직들이다.
    
    ```jsx
    ...
    
    function captainHistoryRisk(voyage, history) {
    	...
    	if(voyage.zone === "중국" && hasChina(history)) result -= 2;
    	...
    }
    
    ...
    
    function voyageProfitFactor(voyage, history) {
    	...
    	if(voyage.zone === "중국" && jasChina(history)) {
    		...
    	}
    	...
    }
    
    ...
    ```
    
    이 특수한 상황을 다루는 로직들을 기본 동작에서 분리하기 위해 상속과 다형성을 이용할 것이다.
    
    다녀온 바 있는 중국으로의 항해 시 추가될 특별 로직이 더 많았다면 이번 리팩터링의 효과가 더욱 컸겠지만,
    
    지금 상황에서도 이 특수 상황을 검사하는 로직이 반복되어 기본 동작을 이해하는 데 방해가 되고 있다.
    
    함수가 많은데, 세부 계산을 수행하는 함수들을 먼저 처리해보자. 다형성을 적용하려면 클래스를 만들어야 하니 여러 함수를 클래스로 묶기부터 적용할 것이다.
    
    그러면 코드가 다음처럼 변한다.
    
    ```jsx
    function rating(voyage, history) {
      return new rating(voyage, history);
    }
    
    class Rating {
      constructor(voyage, history) {
        this.voyage = voyage;
        this.history = history;
      }
    
      get value() {
        const vpf = this.voyageProfitFactor;
        const vr = this.voyageRisk;
        const chr = this.captainHistoryRisk;
        if (vpf * 3 > vr + chr * 2) return "A";
        else return "B";
      }
    
      get voyageRisk() {
        let result = 1;
        if (this.voyage.length > 4) result += 2;
        if (this.voyage.length > 8) result += this.voyage.length - 8;
        if (["중국", "동인도"].includes(this.voyage.zone)) result += 4;
        return Math.max(result, 0);
      }
    
      get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter((v) => v.profit < 0).length;
        if (this.voyage.zone === "중국" && hasChina(this.history)) result -= 2;
        return Math.max(result, 0);
      }
    
      get hasChina() {
        return this.history.some((v) => "중국" === v.zone);
      }
    
      get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        if (this.voyage.zone === "중국" && hasChina(this.history)) {
          result += 3;
          if (this.history.length > 10) result += 1;
          if (this.voyage.length > 12) result += 1;
          if (this.voyage.length > 18) result -= 1;
        } else {
          if (this.history.length > 8) result += 1;
          if (this.voyage.length > 14) result -= 1;
        }
        return result;
      }
    }
    ```
    
    기본 동작을 담당할 클래스가 만들어졌다. 다음 차례는 변형 동작을 담을 빈 서브클래스 만들기다.
    
    ```jsx
    class ExperiencedChinaRating extends Rating {
    
    }
    ```
    
    그런 다음 적절한 변형 클래스를 반환해줄 팩터리 함수를 만든다.
    
    ```jsx
    function createRating(voyage, history) {
      if (voyage.zone === "중국" && history.some(v => "중국" === v.zone))
        return new ExperiencedChinaRating(voyage, history);
      else return new Rating(voyage, history);
    }
    ```
    
    이제 생성자를 호출하는 코드를 모두 찾아서 이 팩터링 함수를 대신 사용하도록 수정한다. 지금 예에서는 rating() 함수 하나뿐이다.
    
    ```jsx
    function rating(voyage, history) {
      return createRating(voyage, history).value;
    }
    ```
    
    서브클래스로 옮길 동작은 두 가지다. captainHistoryRisk() 안의 로직부터 시작하자.
    
    서브클레스에서 이 메서드를 오버라이드한다.
    
    ```jsx
    class Rating {
    	...
    	get captainHistoryRisk() {
    	    let result = 1;
    	    if (this.history.length < 5) result += 4;
    	    result += this.history.filter((v) => v.profit < 0).length;
    	    if (this.voyage.zone === "중국" && hasChina(this.history)) result -= 2;
    	    return Math.max(result, 0);
    	}
    	...
    }
    ```
    
    ```jsx
    class ExperiencedChinaRating extends Rating {
      get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
      }
    }
    ```
    
    ```jsx
    class Rating {
    	...
    	get captainHistoryRisk() {
    	    let result = 1;
    	    if (this.history.length < 5) result += 4;
    	    result += this.history.filter((v) => v.profit < 0).length;
    
    	    return Math.max(result, 0);
    	}
    	...
    }
    ```
    
    voyageProfitFactor()에서 변형 동작을 분리하는 작업은 살짝 더 복잡하다. 이 함수에는 다른 경로가 존재하므로.
    
    단순히 변형 동작을 제거하고 슈퍼클래스의 메서드를 호출하는 방식은 적용할 수 없다.
    
    또한 슈퍼클래스의 메서드를 통째로 서브클래스로 복사하고 싶지도 않다.
    
    그래서 내 해답은 해당 조건부 블록 전체를 함수로 추출하는 것이다.
    
    ```jsx
    get voyageProfitFactor() {
      let result = 2;
      if (this.voyage.zone === "중국") result += 1;
      if (this.voyage.zone === "동인도") result += 1;
      if (this.voyage.zone === "중국" && hasChina(this.history)) {
        result += 3;
        if (this.history.length > 10) result += 1;
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
      } else {
        if (this.history.length > 8) result += 1;
        if (this.voyage.length > 14) result -= 1;
      }
      return result;
    }
    ```
    
    ```jsx
    get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.voyageAndHistoryLengthFactor;
        return result;
    }
    
    get voyageAndHistoryLengthFactor() {
      let result = 0;
      if (this.voyage.zone === "중국" && hasChina(this.history)) {
        result += 3;
        if (this.history.length > 10) result += 1;
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
      } else {
        if (this.history.length > 8) result += 1;
        if (this.voyage.length > 14) result -= 1;
      }
      return result;
    }
    ```
    
    함수 이름에 “그리고”를 뜻하는 “And”가 들어 있어서 악취가 꽤 나지만, 서브클래스 구성을 마무리하는 잠깐 동안만 견뎌보기로 하자.
    
    ```jsx
    class Rating {
    	...
    	get voyageAndHistoryLengthFactor() {
        let result = 0;
        if (this.history.length > 8) result += 1;
        if (this.voyage.length > 14) result -= 1;
        return result;   
      }
    }
    
    class ExperiencedChinaRating() {
    	...
    	get voyageAndHistoryLengthFactor() {
        let result = 0;
        result += 3;
        if (this.history.length > 10) result += 1;
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
      }
    }
    ```
    
    **더 가다듬기**
    
    변형 동작을 서브클래스로 뽑아냈으니 공식적으로는 여기까지가 이 리팩터링의 끝이다.
    
    슈퍼클래스의 로직은 간소화되어 이해하고 다루기 쉬워졌다. 변형 동작은 슈퍼클래스와의 차이를 표현해야 하는 서브클래스에서만 신경쓰면 된다.
    
    하지만 악취를 풍기는 메서드를 새로 만들었으니 처리 방법을 대략으로나마 설명해줘야할 것 같다.
    
    이번 예와 같이 ‘기본 동작-변형 동작’ 상속에서는 서브클래스에서 순전히 오버라이드만을 위해 메서드를 추가하는 일이 흔하다.
    
    하지만 이런 조잡한 메서드는 로직을 부각하기보다는 일의 진행 과정을 모호하게 만들곤 한다.
    
    메서드 이름의 “And”는 이 메서드가 두 가지 독립된 일을 수행한다고 소리친다.
    
    그러니 둘을 분리하는 게 현명할 것이다.
    
    이력 길이를 수정하는 부분을 함수로 추출하면 해결되는데, 이때 슈퍼클래스와 서브클래스 모두에 적용해야 한다.
    
    슈퍼클래스부터 보자.
    
    ```jsx
    class Rating {
    	...
    	get voyageAndHistoryLengthFactor() {
        let result = 0;
        result += this.historyLengthFactor;
        if (this.voyage.length > 14) result -= 1;
        return result;    
      }
    
      get historyLengthFactor() {
        return (this.history.length > 8) ? 1 : 0; 
      }
    }
    ```
    
    같은 작업을 서브클래스에도 해준다.
    
    ```jsx
    class ExperiencedChinaRating {
    	...
    	get voyageAndHistoryLengthFactor() {
        let result = 0;
        result += 3;
        result += this.historyLengthFactor;
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
      }
      
      get historyLengthFactor() {
        return (this.history.length > 10) ? 1 : 0;
      }
    }
    ```
    
    이제 슈퍼클래스에서는 문장을 호출한 곳으로 옮기기(8.4)를 적용할 수 있다.
    
    ```jsx
    class Rating {
    	...
    	get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.historyLengthFactor;
        result += this.voyageAndHistoryLengthFactor;
        return result;
      }
      get voyageAndHistoryLengthFactor() {
        let result = 0;
    //  result += this.historyLengthFactor;
        if (this.voyage.length > 14) result -= 1;
        return result;    
      }
    	get historyLengthFactor() {
        return (this.history.length > 8) ? 1 : 0; 
      }
    }
    
    class ExperiencedChinaRating extends Rating {
    	...
      get voyageAndHistoryLengthFactor() {
        let result = 0;
        result += 3;
    //  result += this.historyLengthFactor;
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
      }
      
      get historyLengthFactor() {
        return (this.history.length > 10) ? 1 : 0;
      }
    }
    ```
    
    이어서 함수 이름을 바꿔주고, Rating의 voyageLengthFactor()를 간소화한다.
    
    ```jsx
    class Rating {
    	...
    	get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.historyLengthFactor;
        result += this.voyageLengthFactor;
        return result;
      }
    
      get voyageLengthFactor() {
        return (this.voyage.length > 14) ? -1 : 0;    
      }
    }
    
    class ExperiencedChinaRating extends Rating {
    	...
    
      get voyageLengthFactor() {
    		...
      }
    	...
    }
    ```
    
    마지막 하나가 더 남았다. 항해 거리 요인을 계산할 때 3점을 더하고 있는데, 이 로직은 전체 결과를 계산하는 쪽으로 옮기는게 좋아 보인다.
    
    ```jsx
    class ExperiencedChinaRating extends Rating {
      get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
      }
    
      get voyageProfitFactor() {
        return super.voyageProfitFactor + 3;
      }
    
      get voyageLengthFactor() {
        let result = 0;
    //  result += 3;
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
      }
      
      get historyLengthFactor() {
        return (this.history.length > 10) ? 1 : 0;
      }
    }
    ```
    
    리팩4터링 결과로 다음 코드를 얻었다. 중국 항해 경험이 있는지와 관련한 복잡한 코드에서 벗어난 기본 Rating 클래스를 먼저 만나보자.
    
    ```jsx
    class Rating {
      constructor(voyage, history) {
        this.voyage = voyage;
        this.history = history;
      }
    
      get value() {
        const vpf = this.voyageProfitFactor;
        const vr = this.voyageRisk;
        const chr = this.captainHistoryRisk;
        if (vpf * 3 > vr + chr * 2) return "A";
        else return "B";
      }
    
      get voyageRisk() {
        let result = 1;
        if (this.voyage.length > 4) result += 2;
        if (this.voyage.length > 8) result += this.voyage.length - 8;
        if (["중국", "동인도"].includes(this.voyage.zone)) result += 4;
        return Math.max(result, 0);
      }
    
      get captainHistoryRisk() {
        let result = 1;
        if (this.history.length < 5) result += 4;
        result += this.history.filter((v) => v.profit < 0).length;
        return Math.max(result, 0);
      }
    
      get voyageProfitFactor() {
        let result = 2;
        if (this.voyage.zone === "중국") result += 1;
        if (this.voyage.zone === "동인도") result += 1;
        result += this.historyLengthFactor;
        result += this.voyageLengthFactor;
        return result;
      }
    
      get voyageLengthFactor() {
        return (this.voyage.length > 14) ? -1 : 0;
      }
    
      get historyLengthFactor() {
        return (this.history.length > 8) ? 1 : 0; 
      }
    }
    ```
    
    중국 항해 경험이 있을 때를 담당하는 다음 클래스는 기본 클래스와의 차이만 담고 있다.
    
    ```jsx
    class ExperiencedChinaRating extends Rating {
      get captainHistoryRisk() {
        const result = super.captainHistoryRisk - 2;
        return Math.max(result, 0);
      }
    
      get voyageLengthFactor() {
        let result = 0;
        if (this.voyage.length > 12) result += 1;
        if (this.voyage.length > 18) result -= 1;
        return result;
      }
      
      get historyLengthFactor() {
        return (this.history.length > 10) ? 1 : 0;
      }
    
      get voyageProfitFactor() {
        return super.voyageProfitFactor + 3;
      }
    }
    ```

---

## 특이 케이스 추가하기(Introduce Special Case)

```jsx
if (aCustomer === "미확인 고객") customerName = "거주자";
```

```jsx
class UnknownCustomer {
  get name() {
    return "거주자";
  }
}
```

### 배경

데이터 구조의 특정 값을 확인한 후 똑같은 동작을 수행하는 코드가 곳곳에 등장하는 경우가 더러 있는데, 흔히 볼 수 있는 중복 코드 중 하나다.

이처럼 코드베이스에서 특정 값에 대해 똑같이 반응하는 코드가 여러 곳이라면 그 반응들을 한 데로 모으는 게 효율적이다.

특수한 경우의 공통 동작을 요소 하나에 모아서 사용하는

특이 케이스 패턴(Special Case Pattern)이라는 것이 있는데, 바로 이럴 때 적용하면 좋은 메커니즘이다.

이 패턴을 활용하면 특이 케이스를 확인하는 코드 대부분을 단순한 함수 호출로 바꿀 수 있다.

특이 케이스는 여러 형태로 표현할 수 있다. 특이 케이스 객체에서 단순히 데이터를 읽기만 한다면 반환할 값들을 담은 리터럴 객체 형태로 준비하면 된다.

그 이상의 어떤 동작을 수행해야 한다면 필요한 메서드를 담은 객체를 생성하면 된다.

특이 케이스 객체는 이를 캑슐화한 클래스가 반환하도록 만들 수도 있고, 변환(transform)을 거쳐 데이터 구조에 추가시키는 형태도 될 수 있다.

null은 특이 케이스로 처리해야 할 때가 많다. 그래서 이 패턴을 널 객체 패턴(Null Object Pattern)이라고도 한다.

하지만 널 외의 다른 특이 케이스에도 같은 패턴을 적용할 수 있으니, 널 객체가 특이 케이스의 특수한 예라고 보는게 맞을 것이다.

---

### 절차

이번 리팩터링의 대상이 될 속성을 담은 데이터 구조(혹은 클래스)에서 시작하자. 이 데이터 구조를 컨테이너라 하겠다.

컨테이너를 사용하는 코드에서는 해당 속성이 특이한 값인지를 검사한다.

우리는 이 대상이 가질 수 있는 값 중 특별하게 다뤄야 할 값을 특이 케이스 클래스(혹은 데이터 구조)로 대체하고자 한다.

1. 컨테이너에 특이 케이스인지를 검사하는 속성을 추가하고, false를 반환하게 한다.
2. 특이 케이스 객체를 만든다. 이 객체는 특이 케이스인지를 검사하는 속성만 포함하며, 이 속성은 true를 반환하게 한다.
3. 클라이언트에서 특이 케이스인지를 검사하는 코드를 함수로 추출한다. 모든 클라이언트가 값을 직접 비교하는 대신 방금 추출한 함수를 사용하도록 고친다.
4. 코드에 새로운 특이케이스 대상을 추가한다. 함수의 반환 값으로 받거나 변환 함수를 적용하면 된다.
5. 특이 케이스를 검사하는 함수 본문을 수정하여 특이 케이스 객체의 속성을 사용하도록 한다.
6. 테스트한다.
7. 여러 함수를 클래스로 묶기(6.9)나 여러 함수를 변환 함수로 묶기(6.10)를 적용하여 특이 케이스를 처리하는 공통 동작을 새로운 요소로 옮긴다.

   → 특이 케이스 클래스는 간단한 요청에는 항상 같은 값을 반환하는 게 보통이므로, 해당 특이 케이스의 리터럴 레코드를 만들어 활용할 수 있을 것이다.

8. 아직도 특이 케이스 검사 함수를 이용하는 곳이 남아 있다면 검사 함수를 인라인한다.

---

### 예시

- 기본 예시
  전력 회사는 전력이 필요한 현장(site)에 인프라를 설치해 서비스를 제공한다.

  ```jsx
  class Site {
    get customer() {
      return this.customer;
    }
  }
  ```

  customer 클래스에는 수 많은 속성이 있겠지만, 그중 다음 세 가지만 고려해보자.

  ```jsx
  class Customer {
    get name() { ... } // 고객 이름
    get billingPlan() { ... } // 요금제
    set billingPlan(arg) { ... }
    get paymentHistory() { ... } // 납부 이력
  }
  ```

  일반적으로 현장에는 고객이 거주하는 게 보통이지만 꼭 그렇지만은 않다.
  누군가 이사를 나가고 (아직 누구인지 모르는) 다른 누군가가 이사왔을 수도 있다.
  이럴 때는 데이터 레코드의 고객 필드를 “미확인 고객”이란 문자열로 채운다.
  이런 상황을 감안하여 Site 클래스를 사용하는 클라이언트 코드들은 알려지지 않은 미확인 고객도 처리할 수 있어야 한다.
  이런 코드의 예를 몇 개 가져와봤다.

  ```jsx
  function clientCode1() {
    const aCustomer = site.customer;
    // ... 수많은 코드 ...
    let customerName;
    if (aCustomer === "미확인 고객") customerName = "거주자";
    else customerName = aCustomer.name;
  }

  function clientCode2() {
    const plan =
      aCustomer === "미확인 고객"
        ? registy.billingPlans.basic
        : aCustomer.billingPlan;
  }

  function clientCode3() {
    if (aCustomer !== "미확인 고객") aCustomer.billingPlan = newPlan;
  }

  function clientCode4() {
    const weeksDelinquent =
      aCustomer === "미확인 고객"
        ? 0
        : aCustomer.paymentHistory.weeksDelinquentInLastYear;
  }
  ```

  코드베이스를 훑어보니 미확인 고객을 처리해야 하는 클라이언트가 여러 개 발견됐고, 그 대부분에서 똑같은 방식으로 처리했다.
  고객 이름으로는 “거주자”를 사용하고, 기본 요금제(bilingPlan)를 청구하고, 연체(delinquent) 기간은 0주(week)로 분류한 것이다.
  많은 곳에서 이뤄지는 이 특이 케이스 검사와 공통된 반응이 우리에게 특이 케이스를 도입할 때임을 말해준다.
  [1] 먼저 미확인 고객인지를 나타내는 메서드를 고객 클래스에 추가한다.

  ```jsx
  class Customer {
  	...
  	get isUnknown() { return false; }
  }
  ```

  [2]그런 다음 미확인 고객 전용 클래스를 만든다.

  ```jsx
  class UnknownCustomer {
    get isUnknown() {
      return true;
    }
  }
  ```

  UnknownCustomer를 Customer의 서브클래스로 만들지 않았음에 주목하자. 다른 언어, 특히 정적 타입 언어였다면 서브클래스로 만들었을 것이다.
  하지만 자바스크립트의 서브클래스 규칙과 동적 타이핑 능력 덕분에 이 경우엔 지금 예처럼 만드는 편이 낫다.
  [3] 지금부터 좀 까다롭다. “미확인 고객”을 기대하는 곳 모두에 새로 만든 특이케이스 객체(UnknownCustomer)를 반환하도록 하고,
  역시 값이 “미확인 고객”인지를 검사하는 곳 모두에서 새로운 isUnknown() 메서드를 사용하도록 고쳐야 한다.
  한 번에 조금씩만 변경하고 테스트할 수 있는 잘 정돈된 코드가 필요해 보인다.
  그런데 Customer 클래스를 수정하여 “미확인 고객” 문자열 대신 UnknownCustomer 객체를 반환하게 한다면,
  클라이언트들 각각에서 “미확인 고객”인지를 확인하는 코드 모두를 isUnknown() 호출로 바꾸는 작업을 한 번에 해야만 한다. 다시 말해, 전혀 매력적이지 않다.
  이런 상황에 봉착할 때마다 내가 자주 사용하는 기법이 있다. 여러 곳에서 똑같이 수정해야만 하는 코드를 별도 함수로 추출하여 한데로 모으는 것이다.
  지금 예에서는 특이 케이스인지 확인하는 코드가 추출 대상이다.

  ```jsx
  function isUnkown(arg) {
    if (!(arg instanceof Customer || arg === "미확인 고객"))
      throw new Error(`잘못된 값과 비교: <${arg}>`);
    return arg === "미확인 고객";
  }
  ```

    <aside>
    💡 의도치 않은 값이 입력되면 에러를 던지도록 했다. 이렇게 하면 리팩터링 도중 실수를 저지르거나, 혹은 이상하게 동작하는 위치를 찾는 데 도움이 된다.
    
    </aside>
    
    이제 이 isUnknown() 함수를 이용해 미확인 고객인지를 확인할 수  있다. 이 변경을 한 번에 하나씩만 적용하고 테스트해보자.
    
    ```jsx
    function clientCode1() {
      const aCustomer = site.customer;
      // ... 수많은 코드 ...
      let customerName;
      if (isUnknown(aCustomer)) customerName = "거주자";
      else customerName = aCustomer.name;
    }
    
    function clientCode2() {
      const plan = isUnknown(aCustomer) ?
            registy.billingPlans.basic
            : aCustomer.billingPlan;
    }
    
    function clientCode3() {
      if (isUnknown(aCustomer)) aCustomer.billingPlan = newPlan;
    }
    
    function clientCode4() {
      const weeksDelinquent = isUnknown(aCustomer) ?
            0
            : aCustomer.paymentHistory.weeksDelinquentInLastYear;
    }
    ```
    
    호출하는 곳 모두에서 isUnknown() 함수를 사용하도록 수정했다면 [4] 특이 케이스일 때 Site 클래스가 UnknownCustomer 객체를 반환하도록 수정하자.
    
    ```jsx
    class Site {
      get customer() { 
        return (this.customer === "미확인 고객") ? new UnknownCustomer() : this.customer;
      }
    }
    ```
    
    [5] isUnknown() 함수를 수정하여 고객 객체의 속성을 사용하도록 하면 “미확인 고객” 문자열을 사용하던 코드는 완전히 사라진다.
    
    ```jsx
    function isUnknown(arg) {
      if( !((arg instanceof Customer) || (arg instanceof UnknownCustomer)))
        throw new Error(`잘못된 값과 비교: <${arg}>`);
      return arg.isUnknown;
    }
    ```
    
    [6] 모든 기능이 잘 동작하는지 테스트한다.
    
    [7] 이제부터가 재미있다. 각 클라이언트에서 수행하는 특이 케이스 검사를 
    
    일반적으로 기본값으로 대체할 수 있다면 이 검사 코드에 여러 함수를 클래스로 묶기를 적용할 수 있다.
    
    지금 예에서는 미확인 고객의 이름으로 “거주자”를 사용하는 코드가 많다. 다음 코드처럼 말이다.
    
    ```jsx
    function clientCode1() {
      const aCustomer = site.customer;
      let customerName;
      if (isUnknown(aCustomer)) customerName = "거주자";
      else customerName = aCustomer.name;
    }
    ```
    
    다음과 같이 적절한 메서드를 UnknownCustomer 클래스에 추가하자.
    
    ```jsx
    class UnknownCustomer {
      get isUnknown() { return true; }
      get name() { return "거주자"; }
    }
    ```
    
    그러면 조건부 코드는 지워도 된다.
    
    ```
    function clientCode1() {
      let customerName = aCustmer.name;
    }
    ```
    
    지금까지의 코드가 동작하는지 테스트한다. 그리고 나라면 이 변수를 인라인할 것이다.
    
    다음은 요금제 속성 차례다.
    
    ```
    function clientCode2() {
      const plan = isUnknown(aCustomer) ?
            registy.billingPlans.basic
            : aCustomer.billingPlan;
    }
    
    function clientCode3() {
      if (isUnknown(aCustomer)) aCustomer.billingPlan = newPlan;
    }
    ```
    
    요금제 속성을 읽는 동작은 앞서 이름 속성을 처리한 과정을 그대로 반복할 것이다.
    
    즉, 일반적인 기본값을 반환한다. 쓰는 동작은 조금 다르다.
    
    현재 코드에서는 미확인 고객에 대해서는 세터를 호출하지 않는다. 
    
    겉보기 동작을 똑같게 만들어야 하므로 특이 케이스에서도 세터가 호출되도록 하되, 세터에서는 아무런 일도 하지 않는다.
    
    ```jsx
    class UnknownCustomer {
      get isUnknown() { return true; }
      get name() { return "거주자"; }
      get billingPlan() { return registy.billingPlan.basic; }
      set billingPlan(arg) { /* 무시한다 */ }
    }
    
    // 클라이언트(읽는 경우)
    const plan = aCustomer.billingPlan;
    // 클라이언트(쓰는 경우)
    aCustomer.billingPlan = newPlan;
    ```
    
    특이 케이스 객체는 값 객체다. 따라서 항상 불변이여야 한다. 대체하려는 값이 가변이라도 마찬가지다.
    
    마지막 상황은 좀 더 복잡하다. 특이 케이스가 자신만의 속성을 갖는 또 다른 객체(지불 이력)를 반환해야 하기 때문이다.
    
    ```jsx
    function clientCode4() {
      const weeksDelinquent = isUnknown(aCustomer) ?
            0
            : aCustomer.paymentHistory.weeksDelinquentInLastYear;
    }
    ```
    
    특이 케이스 객체가 다른 객체를 반환해야 한다면 그 객체 역시 특이 케이스여야 하는 것이 일반적이다.
    
    그래서 NullPaymentHistory를 만들기로 했다.
    
    ```jsx
    class UnknownCustomer {
    	...
      get paymentHistory() { return new NullPaymentHistory(); }
    }
    
    class NullPaymentHistory {
      get weekDelinquentInLastYear() { return 0; }
    }
    
    function clientCode4() {
      const weeksDelinquent = aCustomer.paymentHistory.weeksDelinquentInLastYear;
    }
    ```
    
    [8] 계속해서 모든 클라이언트의 코드를 이 다형적 행위(타입에 따라 동작이 달라진다는 뜻)로 대체할 수 있는지 살펴본다.
    
    예외가 있을 수 있기 때문이다. 특이 케이스로부터 다른 클라이언트와는 다른 무언가를 원하는 독특한 클라이언트가 있을 수 있다.
    
    예컨대 미확인 고객의 이름으로 “거주자”를 사용하는 클라이언트가 23개나 되더라도 튀는 클라이언트가 하나쯤은 있을 수 있다.
    
    다음 클라이언트처럼 말이다.
    
    ```jsx
    const name = !isUnknown(aCustomer) ? aCustmer.name : "미확인 거주자";
    ```
    
    이런 경우엔 원래의 특이 케이스 검사 코드를 유지해야 한다. 이 코드는 Customer에 선언된 메서드를 사용하도록 수정하면 되는데,
    
    구체적으로는 isUnknown 함수를 인라인하면 된다.
    
    ```jsx
    const name = aCustomer.isUnknown ? "미확인 거주자" : aCustomer.name;
    ```
    
    모든 클라이언트를 수정했다면, 호출하는 곳이 없어진 전역 isUnknown() 함수를 죽은 코드 제거하기로 없애준다.

- 객체 리터럴 이용하기
  앞의 예처럼 정말 단순한 값을 위해 클래스까지 동원하는 건 좀 과한 감이 있다. 하지만 고객 정보가 갱신될 수 있어서 클래스가 꼭 필요했다.
  한편, 데이터 구조를 읽기만 한다면 클래스 대신 리터럴 객체를 사용해도 된다.
  같은 예를 다시 살펴보자. 단, 이번에는 고객 정보를 갱신하는 클라이언트가 없다.

  ```jsx
  class Site {
    get customer() { return this.customer; }
  }

  class Customer {
    get name() { ... } // 고객 이름
    get billingPlan() { ... } // 요금제
    set billingPlan(arg) { ... }
    get paymentHistory() { ... } // 납부 이력
  }

  function clientCode1() {
    const aCustomer = site.customer;
    // ... 수많은 코드 ...
    let customerName;
    if (aCustomer === "미확인 고객") customerName = "거주자";
    else customerName = aCustomer.name;
  }

  function clientCode2() {
    const plan = (aCustomer === "미확인 고객") ?
          registy.billingPlans.basic
          : aCustomer.billingPlan;
  }

  function clientCode4() {
    const weeksDelinquent = (aCustomer === "미확인 고객") ?
          0
          : aCustomer.paymentHistory.weeksDelinquentInLastYear;
  }
  ```

  [1] 앞의 예와 같이, 먼저 고객에 isUnknown() 속성을 추가하고 [2] 이 필드를 포함하는 특이 케이스 객체를 생성한다.
  차이점이라면 이번에는 특이 케이스가 리터럴이다.

  ```jsx
  class Customer {
  	...
    get isUnknown() { return false; }
  }

  function createUnknownCustomer() {
    return {
      isUnknown : true ,
    }
  }
  ```

  [3] 특이 케이스 조건 검사 부분을 함수로 추출한다.

  ```jsx
  function isUnknown(arg) {
    return arg === "미확인 고객";
  }

  function clientCode1() {
    const aCustomer = site.customer;
    // ... 수많은 코드 ...
    let customerName;
    if (isUnknown(aCustomer)) customerName = "거주자";
    else customerName = aCustomer.name;
  }

  function clientCode2() {
    const plan = isUnknown(aCustomer)
      ? registy.billingPlans.basic
      : aCustomer.billingPlan;
  }

  function clientCode4() {
    const weeksDelinquent = isUnknown(aCustomer)
      ? 0
      : aCustomer.paymentHistory.weeksDelinquentInLastYear;
  }
  ```

  [4] 조건을 검사하는 코드와 Site 클래스에서 이 특이 케이스를 이용하도록 수정한다.

  ```jsx
  class Site {
    get customer() {
      return this.customer === "미확인 고객"
        ? createUnknownCustomer()
        : this.customer;
    }
  }

  function isUnknown(arg) {
    return arg.isUnknown;
  }
  ```

  [7] 다음으로, 각각의 표준 응답을 적절한 리터럴 값으로 대체한다. 이름부터 해보자.

  ```jsx
  function createUnknownCustomer() {
    return {
      isUnknown: true,
      name: "거주자",
    };
  }

  function clientCode1() {
    const customerName = aCustomer.name;
  }
  ```

  다음은 요금제 차례다.

  ```jsx
  function createUnknownCustomer() {
    return {
      isUnknown: true,
      name: "거주자",
      billingPlan: registy.billingPlans.basic,
    };
  }

  function clientCode2() {
    const plan = aCustomer.billingPlan;
  }
  ```

  비슷한 방법으로 납부 이력이 없다는 정보는 중첩 리터럴로 생성한다.

  ```
  function createUnknownCustomer() {
    return {
      isUnknown : true,
      name : "거주자",
      billingPlan : registy.billingPlans.basic,
      paymentHistory : {
        weeksDelinquentInLastYear : 0,
      },
    }
  }

  function clientCode4() {
    const weeksDelinquent = aCustomer.paymentHistory.weeksDelinquentInLastYear;
  }
  ```

  리터럴을 이런 식으로 사용하려면 불변으로 만들어야 한다(freeze() 메서드를 이용하면 된다).
  참고로 나는 이 방식보다는 클래스를 사용하는 쪽을 선호한다.

- 변환 함수 사용하기
  앞의 두 예는 모두 클래스와 관련 있지만, 변환 단계를 추가하면 같은 아이디어를 레코드에도 적용할 수 있다.
  입력이 다음처럼 단순한 레코드 구조라고 가정하자(JSON 문서다)

  ```jsx
  {
    name : "애크미 보스턴"
    , location : "Malden MA"
    // 더 많은 현장(site) 정보
    , customer : {
      name : "애크미 산업"
      , billingPlan : "plan-451"
      , paymentHistory : {
        weekDelinqientInLastYear : 7
        // 중략
      }
      // 중략
    }
  }
  ```

  고객이 알려지지 않은 경우도 있을 텐데, 앞서와 똑같이 “미확인 고객”으로 표시하자.

  ```jsx
  {
  	name : "물류창고 15"
  	, location : "Malden MA"
  	// 더 많은 현장(site) 정보
  	, customer : "미확인 고객"
  }
  ```

  이번에도 앞서의 예시들과 비슷하게, 미확인 고객인지를 검사하는 클라이언트 코드들이 있다.

  ```jsx
  function client1() {
    const site = acquireSiteData();
    const aCustomer = site.customer;
    // 수 많은 코드
    let customerName;
    if (aCustomer === "미확인 고객") customerName = "거주자";
    else customerName = aCustomer.name;
  }

  function client2() {
    const plan =
      aCustomer === "미확인 고객"
        ? registy.billingPlans.basic
        : aCustomer.billingPlan;
  }

  function client3() {
    const weeksDelinquent =
      aCustomer === "미확인 고객"
        ? 0
        : aCustomer.paymentHistory.weeksDelinquentInLastYear;
  }
  ```

  처음 할 일은 현장 데이터 구조를 변환 함수인 enrichSite()에 통과시키는 것이다. 이 함수는 아직 특별한 작업 없이 깊은 복사만 수행한다.
    <aside>
    💡 참고로 나는 본질은 같고 부가 정보만 덧붙이는 변환 함수의 이름을 “enrich”라 하고, 형태가 변할 때만 “transform”이라는 이름을 쓴다.
    
    </aside>
    
    ```jsx
    function client1() {
      const rawSite = acquireSiteData();
      const site = enrichSite(rawSite);
      const aCustomer = site.customer;
      // 수 많은 코드 
      let customerName;
      if(aCustomer === "미확인 고객") customerName = "거주자";
      else customerName = aCustomer.name;
    }
    
    function enrichSite(inputSite) {
      return cloneDeep(inputSite);
    }
    ```
    
    [3] 알려지지 않은 고객인지 검사하는 로직을 함수로 추출한다.
    
    ```jsx
    function isUnknown(aCustomer) {
      return aCustomer === "미확인 고객";
    }
    
    function client1() {
      const rawSite = acquireSiteData();
      const site = enrichSite(rawSite);
      const aCustomer = site.customer;
      // 수 많은 코드 
      let customerName;
      if(isUnknown(aCustomer)) customerName = "거주자";
      else customerName = aCustomer.name;
    }
    
    function client2() {
      const plan = isUnknown(aCustomer) ?
            registy.billingPlans.basic
            : aCustomer.billingPlan;
    }
    
    function client3() {
      const weeksDelinquent = isUnknown(aCustomer) ?
            0
            : aCustomer.paymentHistory.weeksDelinquentInLastYear;
    }
    ```
    
    [1][2] 고객 레코드에 isUnknown() 속성을 추가하여 현장 정보를 보강(enrich)한다.
    
    ```jsx
    function enrichSite(inputSite) {
      const result = cloneDeep(inputSite);
      const unknwonCustomer = {
        isUnknown : true,
      };
    
      if(isUnknown(result.customer)) result.customer = unknwonCustomer;
      else result.customer.isUnknown = false;
      return result;
    }
    ```
    
    [5] 그런 다음 특이 케이스 검사 시 새로운 속성을 이용하도록 수정한다.
    
    원래의 검사도 유지하여 입력이 원래의 rowSite든 보강(변환)된 site든 상관없이 테스트가 동작하도록 해준다.
    
    ```jsx
    function isUnknown(aCustomer) {
      if(aCustomer === "미확인 고객") return true;
      return aCustomer.isUnknown;
    }
    ```
    
    [6] 모든 기능이 잘 동작하는지 테스트한 다음 [7] 특이 케이스에 여러 함수를 변환 함수로 묶기(6.10)를 적용한다.
    
    먼저 이름 선택 부분을 enrichSite() 함수로 옮긴다.
    
    ```jsx
    function enrichSite(inputSite) {
      const result = cloneDeep(inputSite);
      const unknwonCustomer = {
        isUnknown : true,
        name : "거주자",
      };
    
      if(isUnknown(result.customer)) result.customer = unknwonCustomer;
      else result.customer.isUnknown = false;
      return result;
    }
    
    function client1() {
      const rawSite = acquireSiteData();
      const site = enrichSite(rawSite);
      const aCustomer = site.customer;
      // 수 많은 코드 
      const customerName = aCustomer.name;
    }
    ```
    
    테스트한 후 요금제에도 적용한다.
    
    ```jsx
    function enrichSite(inputSite) {
      const result = cloneDeep(inputSite);
      const unknwonCustomer = {
        isUnknown : true,
        name : "거주자",
        billingPlan : registy.billingPlans.basic
      };
    
      if(isUnknown(result.customer)) result.customer = unknwonCustomer;
      else result.customer.isUnknown = false;
      return result;
    }
    
    function client2() {
      const plan = aCustomer.billingPlan;
    }
    ```
    
    또 테스트한 후 마지막 클라이언트까지 수정한다.
    
    ```jsx
    function enrichSite(inputSite) {
      const result = cloneDeep(inputSite);
      const unknwonCustomer = {
        isUnknown : true,
        name : "거주자",
        billingPlan : registy.billingPlans.basic,
        paymentHistory : {
          weeksDelinquentInLastYear : 0
        }
      };
    
      if(isUnknown(result.customer)) result.customer = unknwonCustomer;
      else result.customer.isUnknown = false;
      return result;
    }
    
    function client3() {
      const weeksDelinquent = aCustomer.paymentHistory.weeksDelinquentInLastYear;
    }
    ```

---

## 어서션 추가하기(Introduce Assertion)

```jsx
if (this.discountRate) base = base - this.discountRate * base;
```

```jsx
assert(this.discountRate >= 0);
if (this.discountRate) base = base - this.discountRate * base;
```

### 배경

특정 조건이 참일 때만 제대로 동작하는 코드 영역이 있을 수 있다.

단순한 예로, 제곱근 계산은 입력이 양수일 때만 정상 동작한다.

객체로 눈을 돌리면 여러 필드 중 최소 하나에는 값이 들어 있어야 동작하는 경우를 생각할 수 있다.

이런 가정이 코드에 항상 명시적으로 기술되어 있지는 않아서 알고리즘을 보고 연역해서 알아내야 할 때도 있다.

주석에라도 적혀 있다면 그나마 형편이 좀 낫다. 더 나은 방법은 어서션을 이용해서 ㅗ드 자체에 삽입해놓는 것이다.

어서션은 항상 참이라고 가정하는 조건부 문장으로, 어서션이 실패했다는 건 프로그래머가 잘 못했다는 뜻이다.

어서션 실패는 시스템의 다른 부분에서는 절대 검사하지 않아야 하며,

어서션이 있고 없고가 프로그램 기능의 정상 동작에 아무런 영향을 주지 않도록 작성돼야 한다.

그래서 어어션을 컴파일타임에 켜고 끌 수 있는 스위치를 제공하는 프로그래밍 언어도 있다.

어서션을 오류 찾기에 활용하라고 추천하는 사람도 왕왕 본다. 물론 좋은 일이긴 하지만 어서션의 쓰임은 여기서 끝나지 않는다.

어서션은 프로그램이 어떤 상태임을 가정한 채 실행되는지를 다른 개발자에게 알려주는 훌륭한 소통 도구인 것이다.

디버깅하기도 편하고 이런 소통 수단으로서의 가치도 있어서, 나는 추적하던 버그를 잡은 뒤에도 어서션을 코드에 남겨두곤 한다.

한편, 테스트 코드가 있다면 어서션의 디버깅 용도로서의 효용은 줄어든다.

단위 테스트를 꾸준히 추가하여 사각을 좁히면 어서션보다 나을 때가 많다.

하지만 소통 측면에서는 어서션이 여전히 매력적이다.

---

### 절차

1. 참이라고 가정하는 조건이 보이면 그 조건을 명시하는 어서션을 추가한다.

<aside>
💡 어서션은 시스템 운영에 영향을 주면 안 되므로 어서션을 추가한다고 해서 동작이 달라지지는 않는다.

</aside>

---

### 예시

할인과 관련한 간단한 예를 준비했다. 다음과 같이 고객은 상품 구입 시 할인율을 적용받는다.

```jsx
class Customer {
  applyDiscount(aNumber) {
    return this.discountRate
      ? aNumber - (this.discountRate = aNumber)
      : aNumber;
  }
}
```

이 코드에는 할인율이 항상 양수라는 가정이 깔려 있다. 어서션을 이용해 이 가정을 명시해보자.

그런데 3항 표현식에는 어서션을 넣을 장소가 적당치 않으니, 먼저 if-then 문장으로 재구성하자.

```jsx
class Customer {
  applyDiscount(aNumber) {
    if (!this.discountRate) return aNumber;
    else return aNumber - this.discountRate * aNumber;
  }
}
```

[1] 이제 간단한 어서션을 추가할 수 있다.

```jsx
class Customer {
  applyDiscount(aNumber) {
    if (!this.discountRate) return aNumber;
    else {
      assert(this.discountRate >= 0);
      return aNumber - this.discountRate * aNumber;
    }
  }
}
```

이번 예에서는 어서션을 세터 메서드에 추가하는 게 나아 보인다.

어서션이 applyDiscount()에서 실패한다면 이 문제가 언제 처음 발생했는지를 찾는 문제를 다시 풀어야 하기 떄문이다.

```jsx
class Customer {
  set discountRate(aNumber) {
    assert(null === aNumber || aNumber >= 0);
    this.discountRate === aNumber;
  }
}
```

이런 어서션은 오류 출처를 특정하기 어려울 때 특히 제값을 한다.

이 예에서라면 입력 데이터가 잘못됐거나 코드의 다른 어딘가에서 부호를 반전시켰을 수도 있을 것이다.

한편, 어서션을 남발하는 것 역시 위험하다. 나는 참이라고 생각하는 가정 모두에 어서션을 달지 않는다.

“반드시 참이어야 하는” 것만 검사한다. 이런 종류의 조건(가정)은 미세하게 자주 조정되기 때문에 중복된 코드가 있다면 큰 문제가 된다.

그래서 이런 조건들에서의 중복은 반드시 남김없이 제거해야 하며, 이때 함수 추출하기가 아주 효과적이다.

나는 프로그래머가 일으킬만한 오류에만 어서션을 사용한다.

데이터를 외부에서 읽어 온다면 그 값을 검사하는 작업은 (어서션의 대상인) 가정이 아니라 (예외 처리로 대응해야 하는) 프로그램 로직의 일부로 다뤄야 한다.

외부 데이터 출처를 전적으로 신뢰할 수 있는 상황이 아니라면 말이다.

어서션은 버그 추적을 돕는 최후의 수단이다. 하지만 모순되게도 나는 절대 실패하지 않으리라 믿는 곳에만 사용한다.

---

## 제어 플래그를 탈출문으로 바꾸기(Replace Control Flag with Break)

```jsx
for (const p of people) {
  if (!found) {
    if (p === "조커") {
      sendAlert();
      found = true;
    }
  }
}
```

```jsx
for (const p of people) {
  if (p === "조커") {
    sendAlert();
    break;
  }
}
```

### 배경

제어 플래그란 코드의 동작을 변경하는 데 사용되는 변수를 말하며,
어딘가에서 값을 계산해 제어 플래그에 설정한 후 다른 어딘가의 조건문에서 검사하는 형태로 쓰인다.

나는 이런 코드를 항상 악취로 본다.

리팩터링으로 충분히 간소화할 수 있음에도 복잡하게 작성된 코드에서 흔히 나타나기 때문이다.

제어 플래그의 주 서식지는 반복문 안이다. break문이나 continue문 활용에 익숙하지 않은 사람이 심어놓기도 하고,

함수의 return문을 하나로 유지하고자 노력하는 사람이 심기도 한다.

모든 함수의 return문은 하나여야 한다고 주장하는 사람도 있지만, 나는 동의하지 않는다.

함수에서 할 일을 다 마쳤다면 그 사실을 return문으로 명확히 알리는 편이 낫지 않을까?

---

### 절차

1. 제어 플래그를 사용하는 코드를 함수로 추출할지 고려해본다.
2. 제어 플래그를 갱신하는 코드 각각을 적절한 제어문으로 바꾼다. 하나 바꿀 때마다 테스트한다.

   → 제어문으로는 주로 return, break, continue가 쓰인다.

3. 모두 수정했다면 제어 플래그를 제거한다.

---

### 예시

다음은 사람 목록을 훑으면서 악당(miscreant)을 찾는 코드다. 악당 이름을 하드코딩되어 있다.

```jsx
let found = false;
// 생략(중요하지 않은 코드)
for (const p of people) {
  if (!found) {
    if (p === "조커") {
      sendAlert();
      found = true;
    }
    if (p === "사루만") {
      sendAlert();
      found = true;
    }
  }
}
// 생략(중요하지 않은 코드)
```

[1] 여기서 제어 플래그는 found 변수이고, 제어 흐름을 변경하는 데 쓰인다.

이처럼 정리해야 할 코드양이 제법 된다면 가장 먼저 함수 추출하기를 활용해서 서로 밀접한 코드만 담은 함수를 뽑아내보자.

그러면 관련된 코드만 따로 떼어서 볼 수 잇다.

```
// 생략(중요하지 않은 코드)
checkForMiscreants(people);
// 생략(중요하지 않은 코드)

function checkForMiscreants(people) {
	let found = false;
  for (const p of people) {
    if(!found){
      if(p === "조커"){
        sendAlert();
        found = true;
      }
      if(p === "사루만"){
        sendAlert();
        found = true;
      }
    }
  }
}
```

[2] 제어 플래그가 참이면 반복문에서는 더 이상 할 일이 없다. break문으로 반복문에서 벗어나거나 return을 써서 함수에서 아예 빠져나오면 된다.

이 함수에서는 더 할 일이 없으니 return을 사용하자. 언제나처럼 작은 단계로 나눠 진행할 것이다.

가장 먼저 return문을 넣은 후 테스트해보자.

```
function checkForMiscreants(people) {
	let found = false;
  for (const p of people) {
    if(!found){
      if(p === "조커"){
        sendAlert();
        return;
      }
      if(p === "사루만"){
        sendAlert();
        found = true;
      }
    }
  }
}
```

제어 플래그가 갱신되는 장소를 모두 찾아서 같은 과정을 반복한다.

```jsx
function checkForMiscreants(people) {
  let found = false;
  for (const p of people) {
    if (!found) {
      if (p === "조커") {
        sendAlert();
        return;
      }
      if (p === "사루만") {
        sendAlert();
        return;
      }
    }
  }
}
```

[3] 갱신 코드를 모두 제거했다면 제어 플래그를 참조하는 다른 코드도 모두 제거한다.

```jsx
function checkForMiscreants(people) {
  for (const p of people) {
    if (p === "조커") {
      sendAlert();
      return;
    }
    if (p === "사루만") {
      sendAlert();
      return;
    }
  }
}
```

**더 가다듬기**

이번 리팩터링은 여기서 끝이다. 하지만 이런 코드가 내 눈앞에 있다면 다음과 같은 모습으로 정리할 것이다.

```jsx
function checkForMiscreants(people) {
  if (people.some((p) => ["조커", "사루만"].includes(p))) sendAlert();
}
```

그리고 이 코드를 보고 있자니, 자바스크립트도 어서 다음과 같은 근사한 집합 연산을 지원해줬으면 하는 마음이 간절해진다.

```jsx
["조커", "사루만"].isDisjointWith(people);
```

---
