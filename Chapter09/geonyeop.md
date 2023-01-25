# [9] 데이터 조직화

데이터 구조는 프로그램에서 중요한 역할을 수행하니 데이터 구조에 집중한 리팩터링만 한 묶음 따로 준비했다.

하나의 값이 여러 목적으로 사용된다면 혼란과 버그를 낳는다. 그러니 이런 코드를 발견하면 변수 쪼개기(9.1)를 적용해 용도별로 분리하자.

다른 프로그램 요소와 마찬가지로 변수 이름을 제대로 짓는 일은 까다로우면서도 중요하다.

그래서 변수 이름 바꾸기와는 반드시 친해져야 한다.

한편, 파생 변수를 질의 함수로 바꾸기(9.3)를 활용하여 변수 자체를 없애는게 가장 좋은 해법일 때도 있다.

참조(reference)인지 값(value)인지 헷갈려 문제가 되는 코드도 자주 볼 수 있는데,

둘 사이를 전환할 때는 **참조를 값으로 바꾸기**(9.4)와 **값을 참조로 바꾸기**(9.5)를 사용한다.

코드에 의미를 알기 어려운 리터럴이 보이면 매직 리터럴 바꾸기(9.6)로 명확하게 바꿔준다.

## 변수 쪼개기(Split Variable)

```jsx
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```

```jsx
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

### 배경

변수는 다양한 용도로 쓰인다. 그중 변수에 값을 여러 번 대입할 수 밖에 없는 경우도 있다.

예컨대 (반복문에서 i 변수 같은) 루프 변수는 반복문을 한 번 돌 때마다 값이 바뀐다.

수집 변수(collecting variable)는 메서드가 동작하는 중간중간 값을 저장한다.

그 외에도 변수는 긴 코드의 결과를 저장했다가 나중에 쉽게 참조하려는 목적으로 흔히 쓰인다.

이런 변수에는 값을 단 한 번만 대입해야 한다. 대입이 두 번 이상 이뤄진다면 여러 가지 역할을 수행한다는 신호다.

역할이 둘 이상인 변수가 있다면 쪼개야 한다. 예외는 없다. 약할 하나당 변수 하나다.

여러 용도로 쓰인 변수는 코드를 읽는 이에게 커다란 혼란을 주기 때문이다.

---

### 절차

1. 변수를 선언한 곳과 값을 처음 대입하는 곳에서 변수 이름을 바꾼다.

   → 이후의 대입이 항상 i = i + <무언가> 형태라면 수집 변수이므로 쪼개면 안 된다.

   수집 변수는 총합 계산, 문자열 연결, 스트림에 쓰기, 컬렉션에 추가하기 등의 용도로 흔히 쓰인다.

2. 가능하면 이때 불변으로 선언한다.
3. 이 변수에 두 번째로 값을 대입하는 곳 앞까지의 모든 참조(이 변수가 쓰인 곳)를 새로운 변수 이름으로 바꾼다.
4. 두 번째 대입 시 변수를 원래 이름으로 다시 선언한다.
5. 테스트한다.
6. 반복한다. 매 반복에서 변수를 새로운 이름으로 선언하고 다음번 대입 때까지의 모든 참조를 새 변수명으로 바꾼다.

   이 과정을 마지막 대입까지 반복한다.

---

### 예시

- 기본 예시
  이번 예에서는 해기스(haggis)라는 음식이 다른 지역으로 전파된 거리를 구하는 코드를 살펴볼 것이다.
  해기스가 발생지에서 초기 힘을 받아 일정한 가속도로 전파되다가,
  시간이 흐른 후 어떠한 계기로 두 번째 힘을 받아 전파 속도가 빨라진다고 가정해보자.
  이를 일반적인 물리 법칙을 적용해 전파 거리를 다음과 같이 계산했다.
  ```jsx
  function distanceTravelled(scenario, time) {
    let result;
    let acc = scenario.primaryForce / scenario.mass; // 가속도(a) = 힘(F) / 질량(m)
    let primaryTime = Math.min(time, scenario.delay);
    result = 0.5 * acc * primaryTime * primaryTime; // 전파된 거리
    let secondaryTime = time - scenario.delay;
    if (secondaryTime > 0) {
      let primaryVelocity = acc * scenario.delay;
      acc = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass;
      result +=
        primaryVelocity * secondaryTime +
        0.5 * acc * secondaryTime * secondaryTime;
    }
    return result;
  }
  ```
  괜찮아 보이는 작은 함수가 만들어졌다. 이 예시에서 흥미로운 부분은 acc 변수에 값이 두 번 대입된다는 점이다.
  역할이 두 개라는 신호다. 하나는 첫 번째 힘이 유발한 초기 가속도를 저장하는 역할이고,
  다른 하나는 두 번째 힘까지 반영된 후의 가속도를 저장하는 역할이다. 쪼개야 할 변수다.
  **함수나 파일에서 특정 심벌이 쓰인 위치를 시각적으로 강조해주는 코드 편집기를 사용하면 변수의 쓰임을 분석하는 데 도움이 된다.**
  **물론 요즘 편집기들은 대부분 지원한다.**
  첫 단계로 [1] 변수에 새로운 이름을 지어주고 [2] 선언 시 const를 붙여 불변으로 만든다.
  [3] 그런 다음 두 번째 대입 전까지의 모든 참조를 새로운 이름으로 바꾼다.
  [4] 그리고 두 번째로 대입할 때 변수를 다시 선언한다.
  ```jsx
  function distanceTravelled(scenario, time) {
    let result;
    let primaryAcceleration = scenario.primaryForce / scenario.mass; // [1][2]
    let primaryTime = Math.min(time, scenario.delay);
    result = 0.5 * primaryAcceleration * primaryTime * primaryTime; // [3]
    let secondaryTime = time - scenario.delay;
    if (secondaryTime > 0) {
      let primaryVelocity = primaryAcceleration * scenario.delay; // [3]
      let acc =
        (scenario.primaryForce + scenario.secondaryForce) / scenario.mass; // [4]
      result +=
        primaryVelocity * secondaryTime +
        0.5 * acc * secondaryTime * secondaryTime;
    }
    return result;
  }
  ```
  수정한 코드를 보면 [1]에서 변수의 첫 번째 용도만을 대표하는 이름을 선택했음을 알 수 있다.
  그리고 const로 선언해서 값을 다시 대입하지 못하도록 했다.
  그리고 두 번째 대입하는 곳[4]에서 변수를 원래 이름으로 다시 선언했다.
  [5] 이제 컴파일하여 테스트해보자. 잘 동작해야 한다.
  [6] 두 번째 대입을 처리할 차례다. 이번에는 두 번째 용도에 적합한 이름으로 수정하므로 이 변수의 원래 이름은 완전히 사라지게 된다.
  ```jsx
  function distanceTravelled(scenario, time) {
    let result;
    let primaryAcceleration = scenario.primaryForce / scenario.mass;
    let primaryTime = Math.min(time, scenario.delay);
    result = 0.5 * primaryAcceleration * primaryTime * primaryTime;
    let secondaryTime = time - scenario.delay;
    if (secondaryTime > 0) {
      let primaryVelocity = primaryAcceleration * scenario.delay;
      const secondaryAccleration =
        (scenario.primaryForce + scenario.secondaryForce) / scenario.mass;
      result +=
        primaryVelocity * secondaryTime +
        0.5 * secondaryAccleration * secondaryTime * secondaryTime;
    }
    return result;
  }
  ```
  이 외에도 리팩터링할 만한 곳이 많이 보일 것이다. 한번 해보길 바란다.
- 입력 매개변수의 값을 수정할 때
  변수 쪼개기의 또 다른 예로 입력 매개변수를 생각해볼 수 있다.
  ```jsx
  function discount(inputValue, quantity) {
    if (inputValue > 50) inputValue = inputValue - 2;
    if (quantity > 100) inputValue = unputValue - 1;
    return inputValue;
  }
  ```
  여기서 inputValue는 함수에 데이터를 전달하는 용도와 결과를 호출자에 반환하는 용도로 쓰였다.
  (자바스크립트의 매개변수는 값에 의한 호출(call by value)방식으로 전달되므로 inputValue를 수정해도 호출자에 영향을 주지 않는다.)
  이 상황이라면 먼저 다음과 같인 inputValue를 쪼개야 한다.
  ```jsx
  function discount(originalInputValue, quantity) {
    let inputValue = originalInputValue;
    if (inputValue > 50) inputValue = inputValue - 2;
    if (quantity > 100) inputValue = inputValue - 1;
    return inputValue;
  }
  ```
  그런 다음 변수 이름 바꾸기를 두 번 수행해서 각각의 쓰임에 어울리는 이름을 지어준다.
  ```jsx
  function discount(inputValue, quantity) {
    let result = inputValue;
    if (inputValue > 50) result = inputValue - 2;
    if (quantity > 100) result = inputValue - 1;
    return result;
  }
  ```
  첫 번째 if문에서 inputValue와 비교하도록 수정한 게 보일 것이다.
  사실 어떤 변수를 써도 똑같이 동작하지만, 이 코드가 입력 값에 기초하여 결괏값을 누적해 계산한다는 사실을 더 명확히 드러낸 것이다.

---

## 필드 이름 바꾸기(Rename Field)

```jsx
class Organization {
	get name() { ... }
}
```

```jsx
class Organization {
	get title() { ... }
}
```

### 배경

이름은 중요하다. 그리고 프로그램 곳곳에서 쓰이는 레코드 구조체의 필드 이름들은 특히 더 중요하다.

데이터 구조는 프로그램을 이해하는 데 큰 역할을 한다.

수년 전 프레드 브룩스는 이런 말을 했다.

“데이터 테이블 없이 흐름도(flow chart)만 보여줘서는 나는 여전히 혼란스러울 것이다.

하지만 데이터 테이블을 보여준다면 흐름도는 웬만해선 필요조차 없을 것이다. 테이블만으로 명확하기 때문이다.”

요즘은 흐름도를 그리는 사람을 찾기 어렵지만 이 격언은 여전히 유효하다.

데이터 구조는 무슨 일이 벌어지는지를 이해하는 열쇠다.

데이터 구조가 중요한 만큼 반드시 깔끔하게 관리해야 한다. 다른 요소와 마찬가지로 개발을 진행할수록 데이터를 더 잘 이해하게 된다.

따라서 그 깊어진 이해를 프로그램에 반드시 반영해야 한다.

이 과정에서 레코드의 필드 이름을 바꾸고 싶을 수 있는데, 클래스에서도 마찬가지다.

게터와 세터 메서드는 클래스 사용자 입장에서는 필드와 다를 바 없다.

따라서 게터와 세터 이름 바꾸기도 레코드 구조체의 필드 이름 바꾸기와 똑같이 중요하다.

---

### 절차

1. 레코드의 유효 범위가 제한적이라면 필드에 접근하는 모든 코드를 수정한 후 테스트한다. 이후 단계는 필요 없다.
2. 레코드가 캡슐화되지 않았다면 우선 레코드를 캡슐화한다.
3. 캡슐화된 객체 안의 private 필드명을 변경하고, 그에 맞게 내부 메서드들을 수정한다.
4. 테스트한다.
5. 생성자의 매개변수 중 필드와 이름이 겹치는 게 있다면 함수 선언 바꾸기로 변경한다.
6. 접근자들의 이름도 바꿔준다.

---

### 예시

다음의 상수가 하나 있다.

```jsx
const prganization = { name: "애크미 구스베리", country: "GB" };
```

여기서 name을 ‘title’로 바꾸고 싶다고 해보자. 이 객체는 코드베이스 곳곳에서 사용되며, 그중 이 제목(title)을 변경하는 곳도 있다.

[2] 그래서 우선 organization 레코드를 클래스로 캡슐화한다.

```jsx
class Organization {
  constructor(data) {
    this.name = data.name;
    this.country = data.country;
  }
  get name() {
    return this.name;
  }
  set name(aString) {
    this.name = aString;
  }
  get country() {
    return this.country;
  }
  set country(aCountryCode) {
    this.country = aCountryCode;
  }
}
```

자, 레코드를 클래스로 캡슐화하자 이름을 변경할 곳이 네 곳이 되었다. 게터, 세터, 생성자, 내부 데이터 구조다.

일을 더 키워버린 게 아닌가 싶지만, 실제로는 더 쉬워진 것이다.

모든 변경을 한 번에 수행하는 대신 작은 단계들로 나눠 독립적으로 수행할 수 있게 됐으니 말이다.

여기서 ‘작은 단계’라 함은 각 단계에서 잘못될 일이 적어졌고, 그래서 할 일도 줄었다는 뜻이다.

물론 실수를 저지르지 않는다면 줄어들 일이 없지만, ‘실수 없이’란 내가 아주 예전에 포기한 판타지일 뿐이다.

입력 데이터 구조를 내부 데이터 구조로 복제했으므로 둘을 구분해야 독립적으로 작업할 수 있다.

[3] 별도의 필드를 정의하고 생성자와 접근자에서 둘을 구분해 사용하도록 하자.

```jsx
class Organization {
  constructor(data) {
    this.title = data.name;
    this.country = data.country;
  }
	...
}
```

다음으로 생성자에서 ‘title’도 받아들일 수 있도록 조치한다.

```jsx
class Organization {
  constructor(data) {
    this.title = data.title ? data.title : data.name;
    this.country = data.country;
  }
  get title() {
    return this.title;
  }
  set title(aString) {
    this.title = aString;
  }
  get country() {
    return this.country;
  }
  set country(aCountryCode) {
    this.country = aCountryCode;
  }
}
```

이상으로 생성자를 호출하는 쪽에서는 name, title을 모두 사용할 수 있게 되었다.

이제 이 생성자를 호출하는 곳을 모두 찾아서 새로운 이름인 ‘title’을 사용하도록 하나씩 수정한다.

```jsx
const prganization = { title: "애크미 구스베리", country: "GB" };
```

모두 수정했다면 생성자에서 ‘name’을 사용할 수 있게 하던 코드를 제거한다.

```jsx
class Organization {
  constructor(data) {
    this.title = data.title;
    this.country = data.country;
  }
	...
}
```

[6] 이제 생성자와 데이터가 새로운 이름을 사용하게 되었으니 접근자도 수정할 수 있게 되었다.

쉬운 작업이다. 단순히 접근자 각각의 이름을 바꿔주면 된다.

```jsx
class Organization {
	...
  get title() { return this.title; }
  set title(aString) { this.title = aString; }
	...
}
```

지금까지 보여준 과정은 널리 참조되는 데이터 구조일 때 적용되는 가장 복잡한 형태다.

한 함수의 안에서만 쓰였다면 캡슐화할 필요 없이 그저 원하는 속성들의 이름을 바꿔주는 것으로 끝났을 일이다.

전체 과정을 적용할지는 상황에 맞게 잘 판단하기 바란다.

단, 리팩터링 도중 테스트에 실패한다면 더 작은 단계로 나눠서 진행해야 한다는 신호임을 잊지 말자.

데이터 구조를 불변으로 만들 수 있는 프로그래밍 언어도 있다.

그런 언어를 사용한다면 캡슐화하는 대신 데이터 구조의 값을 복제해 새로운 이름으로 선언한다.

그런 다음 사용하는 곳을 찾아 하나씩 새 데이터를 사용하도록 수정하고, 마지막으로 원래의 데이터 구조를 제거하면 된다.

가변 데이터 구조를 이용한다면 데이터를 복제하는 행위가 재앙으로 이어질 수 있다.

불변 데이터 구조가 널리 쓰이게 된 이유는 바로 이 재앙을 막기 위해서다.

---

## 파생 변수를 질의 함수로 바꾸기(Replace Derived Variable with Query)

```jsx
get discountedTotal() { return this.discountedTotal; }
set discount(aNumber) {
	const old = this.discount;
	this.discount = aNumber;
	this.discountedTotal += old - aNumber;
}
```

```jsx
get discountedTotal() { return this.baseTotal - this.discount; }
set discount(aNumber) { this.discount = aNumber; }
```

### 배경

가변 데이터는 소프트웨어에 문제를 일으키는 가장 큰 골칫거리에 속한다.

가변 데이터는 서로 다른 두 코드를 이상한 방식으로 결합하기도 하는데,

예컨대 한 쪽 코드에서 수정한 값이 연쇄 효과를 일으켜 다른 쪽 코드에 원인을 찾기 어려운 문제를 야기하기도 한다.

그렇다고 가변 데이터를 완전히 배제하기란 현실적으로 불가능할 때가 많지만,

가변 데이터의 유효 범위를 가능한 좁혀야 한다고 힘주어 주장해본다.

효과가 좋은 방법으로, 값을 쉽게 계산해낼 수 있는 변수들을 모두 제거할 수 있다.

계산 과정을 보여주는 코드 자체가 데이터의 의미를 더 분명히 드러내는 경우도 자주 있으며 변경된 값을 깜빡하고 결과 변수에 반영하지 않는 실수를 막아준다.

여기에는 합당한 예외가 있다. 피연산자 데이터가 불변이라면 계산 결과도 일정하므로 역시 불변으로 만들 수 있다.

그래서 새로운 데이터 구조를 생성하는 변형 연산(transformation operation)이라면 비록 계산 코드로 대체할 수 있더라도 그대로 두는 것도 좋다.

변형 연산에는 두 가지가 있다.

1. 데이터 구조를 감싸며 그 데이터에 기초하여 계산한 결과를 속성으로 제공하는 객체
2. 데이터 구조를 받아 다른 데이터 구조로 변환해 반환하는 함수

소스 데이터가 가변이고 파생 데이터 구조의 수명을 관리해야 하는 상황에서는 객체를 사용하는 편이 확실히 유리하다.

반면 소스 데이터가 불변이거나 파생 데이터를 잠시 쓰고 버릴 거라면 어느 방식을 써도 상관없다.

---

### 절차

1. 변수 값이 갱신되는 지점을 모두 찾는다. 필요하면 변수 쪼개기(9.1)를 활용해 각 갱신 지점에서 변수를 분리한다.
2. 해당 변수의 값을 계산해주는 함수를 만든다.
3. 해당 변수가 사용되는 모든 곳에 어서션을 추가(10.6)하여 함수의 계산 결과가 변수의 값과 같은지 확인한다.

   → 필요하면 변수 캡슐화하기(6.6)를 적용하여 어서션이 들어갈 장소를 마련해준다.

4. 테스트하낟.
5. 변수를 읽는 코드를 모두 함수 호출로 대체한다.
6. 테스트한다.
7. 변수를 선언하고 갱신하는 코드를 죽은 코드 제거하기(8.9)로 없앤다.

---

### 예시

- 기본 예시
  작지만 확실하고 보기 흉한 예를 준비했다.

  ```jsx
  class ProductionPlan {
    get production() {
      return this.production;
    }
    applyAdjustment(anAdjustment) {
      this.adjustments.push(anAdjustment);
      this.production += anAdjustment.amount;
    }
  }
  ```

  흉하다는 기준은 사람마다 다르지만, 이 예에서는 중복이 내 눈에 거슬린다.
  일반적인 코드 중복은 아니고, 데이터 중복이다.
  이 코드는 조정 값 adjustment를 적용하는 과정에서 직접 관련이 없는 누적 값 production까지 갱신했다.
  그런데 이 누적 값은 매번 갱신하지 않고도 계산할 수 있다.
  하지만 나는 신중한 사람이다. 이 값을 계산해낼 수 있다는 건 내 가정일 뿐이니 어서션을 추가하여 검증해보자.

  ```jsx
  class ProductionPlan {
    get production() {
      assert(this.production === this.calculateProduction);
      return this.production;
    }

    get calculateProduction() {
      return this.adjustments.reduce((sum, a) => sum + a.amount, 0);
    }
  }
  ```

  어서션을 추가했으면 테스트해본다. 어서션이 실패하지 않으면 필드를 반환하던 코드를 수정하여 계산 결과를 직접 반환하도록 한다.

  ```jsx
  class ProductionPlan {
  	...
  	get production() {
  		return this.production;
  	}
  	...
  }
  ```

  그런 다음 calculateProduction() 메서드를 인라인한다.

  ```jsx
  class ProductionPlan {
  	...
  	get production() {
  		return this.adjustments.reduce((sum, a) => sum + a.amount, 0);
  	}
  	...
  }
  ```

  마지막으로, 옛 변수를 참조하는 모든 코드를 죽은 코드 제거하기로 정리한다.

  ```jsx
  class ProductionPlan {
  	...
  	applyAdjustment(anAdjustment) {
  		this.adjustments.push(anAdjustment);
  	}
  	...
  }
  ```

- 소스가 둘 이상일 때
  앞의 예는 production() 값에 영향을 주는 요소가 하나뿐이라 깔끔하고 이해하기 쉬웠다. 하지만 때에 따라서는 둘 이상의 요소가 관여되기도 한다.

  ```jsx
  class ProductionPlan {
    constructor(production) {
      this.production = production;
      this.adjustments = [];
    }

    get production() {
      return this.production;
    }

    applyAdjustment(anAdjustment) {
      this.adjustments.push(anAdjustment);
      this.production += anAdjustment.amount;
    }
  }
  ```

  어서션 코드를 앞의 예와 똑같이 작성한다면 production의 초기값이 0이 아니면 실패하고 만다.
  이 파생 데이터를 대체할 방법은 아직 있고, 사실 간단하다. 앞에서와의 차이라면 변수 쪼개기를 먼저 적용하는 것뿐이다.

  ```jsx
  class ProductionPlan {
    constructor(production) {
      this.production = production;
      this.productionAccumulator = 0;
      this.adjustments = [];
    }

    get production() { return this.productionAccumulator; }
  	...
  }
  ```

  이제 어서션을 추가한다.

  ```jsx
  class ProductionPlan {
    constructor(production) {
      this.production = production;
      this.productionAccumulator = 0;
      this.adjustments = [];
    }

    get production() {
      assert(this.productionAccumulator === this.calculatedProductionAccumulator);
       return this.initialProduction + this.productionAccumulator;
    }

    get calculateProductionAccumulator() {
      return this.adjustments.reduce((sum, a) => sum + a.amount, 0);
    }

  	...
  }
  ```

  그 다음은 이전과 거의 같다. 다만 이번에는 calculateProductionAccumulator()를 인라인하지 않고 속성으로 남겨두는 편이 나아 보인다.

---

## 참조를 값으로 바꾸기(Change Reference to Value)

```jsx
class Product {
  applyDiscount(arg) {
    this.price.amount -= arg;
  }
}
```

```jsx
class Product {
	this.price = new Money(this.price.amount - arg, this.price.currency);
}
```

### 배경

객체(데이터 구조)를 다른 객체(데이터 구조)에 중첩하면 내부 객체를 참조 혹은 값으로 취급할 수 있다.

참조냐 값이냐의 차이는 내부 객체의 속성을 갱신하는 방식에서 가장 극명하게 드러난다.

참조로 다루는 경우에는 내부 객체는 그대로 둔 채 그 객체의 속성만 갱신하며,

값으로 다루는 경우에도 새로운 속성을 담은 객체로 기존 내부 객체를 통째로 대체한다.

필드를 값으로 다룬다면 내부 객체의 클래스를 수정하여 값 객체(VO)로 만들 수 있다.

VO는 대체로 자유롭게 활용하기 좋은데, 특히 불변이기 때문이다. 일반적으로 불변 데이터 구조는 다루기 더 쉽다.

불변 데이터 값은 프로그램 외부로 건네줘도 나중에 그 값이 나 몰래 바뀌어서 내부에 영향을 줄까 염려하지 않아도 된다.

값을 복제해 이곳저곳에서 사용하더라도 서로 간의 참조를 관리하지 않아도 된다.

그래서 VO는 분산 시스템과 동시성 시스템에서 특히 유용하다.

한편 VO의 이러한 특성 때문에 이번 리팩터링을 적용하면 안 되는 상황도 있다.

예컨대 특정 객체를 여러 객체에서 공유하고자 한다면,

그래서 공유 객체의 값을 변경했을 때 이를 객체 모두에 알려줘야 한다면 공유 객체를 참조로 다뤄야 한다.

---

### 절차

1. 후보 클래스가 불변인지, 혹은 불변이 될 수 있는지 확인한다.
2. 각각의 세터를 하나씩 제거한다.
3. 이 값 객체의 필드들을 사용하는 동치성 비교 메서드를 만든다.

   → 대부분의 언어는 이런 상황에 사용할 수 있도록 오버라이딩 가능한 동치성 비교 메서드를 제공한다.

   동치성 비교 메서드를 오버라이드할 때는 보통 해시코드 생성 메서드도 함께 오버라이드해야 한다.

---

### 예시

사람(Person) 객체가 있고, 이 객체는 다음 코드처럼 생성 시점에는 전화번호가 올바로 설정되지 못하게 짜여있다.

```jsx
class Person {
  constructor() {
    this.telephoneNumber = new TelephoneNumber();
  }

  get officeAreaCode() {
    return this.telephoneNumber.areaCode;
  }
  set officeAreaCode(arg) {
    this.telephoneNumber.areaCode = arg;
  }
  get officeNumber() {
    return this.telephoneNumber.officeNumber;
  }
  set officeNumber(arg) {
    this.telephoneNumber.number = arg;
  }
}
```

```jsx
class TelephoneNumber {
  get areaCode() {
    return this.areaCode;
  }
  set areaCode(arg) {
    this.areaCode = arg;
  }
  get number() {
    return this.number;
  }
  set number(arg) {
    this.number = arg;
  }
}
```

클래스를 추출하다 보면 종종 이런 상황이 벌어지곤 한다. 추출해서 새로 만들어진 객체(이 예에서는 TelephoneNumber)

를 갱신하는 메서드들은 여전히 추출 전 클래스(이 예에서는 Person)에 존재할 것이다.

어쨌든 새 클래스를 가리키는 참조가 하나뿐이므로 참조를 값으로 바꾸기에 좋은 상황이다.

[1] 가장 먼저 할 일은 전화번호를 불변으로 만들기다.

[2] 필드들의 세터들만 제거하면 된다.

세터 제거의 첫 단계로, 세터로 설정하던 두 필드를 생성자에서 입력받아 설정하도록 한다.(함수 선언 바꾸기)

```jsx
class TelephoneNumber {
	constructor(areaCode, number) {
    this.areaCode = areaCode;
    this.number = number;
  }
	...
}
```

이제 세터를 호출하는 쪽을 살펴서 전화번호를 매번 다시 대입하도록 바꿔야 한다. areaCode 부터 바꿔보자.

```jsx
class Person {
	...
	set officeAreaCode(arg) {
    this.telephoneNumber = new TelephoneNumber(arg, this.officeNumber);
  }
	...
}
```

나머지 필드에도 같은 작업을 해준다.

```jsx
class Person {
	...
	set officeNumber(arg) {
    this.telephoneNumber = new TelephoneNumber(this.areaCode, arg);
  }
	...
}
```

[3]이제 전화번호는 불변이 되었으니 진짜 ‘값’이 될 준비가 끝났다. 값 객체로 인정받으려면 동치성을 값 기반으로 평가해야 한다.

이 점에서 자바스크립트는 살짝 아쉬운데, 자바스크립트는 참조 기반 동치성을

값 기반 동치성으로 대체하는 일과 관련하여 언어나 핵심 라이브러리 차원에서 지원해주는 게 없다.

내가 생각해낸 최선의 방법은 equals 메서드를 직접 작성하는 것이다.

```jsx
class TelephoneNumber {
	...
	equals(other) {
    if(!(other instanceof TelephoneNumber)) return false;
    return this.areaCode === other.areaCode
           && this.number === other.number;
  }
	...
}
```

다음과 같이 테스트 해주는 것도 잊으면 안 된다.

```jsx
it("telephone equals", function () {
  assert(
    new TelephoneNumber("312", "555-0142").equals(
      new TelephoneNumber("312", "555-0142")
    )
  );
});
```

전화번호를 사용하는 곳이 둘 이상이라도 절차는 똑같다. 세터를 제거할 때 해당 사용처 모두를 수정하면 된다.

번호가 다른 전화번호들로 비교해보고, 유효하지 않은 번호나 널 값과도 비교해보면 좋다.

---

## 값을 참조로 바꾸기(Change Value to Reference)

```jsx
let customer = new Customer(customerData);
```

```jsx
let customer = new Customer(customerData);
```

### 배경

하나의 데이터 구조 안에 논리적으로 똑같은 제 3의 데이터 구조를 참조하는 레코드가 여러 개 있을 때가 있다.

예컨대 주문 목록을 읽다 보면 같은 고객이 요청한 주문이 여러 개 섞여 있을 수 있다.

이때 고객을 값으로도, 혹은 참조로도 다룰 수 있다. 값으로 다룬다면 고객 데이터가 각 주문에 복사되고,

참조로 다룬다면 여러 주문이 단 하나의 데이터 구조를 참조하게 된다.

고객 데이터를 갱신할 일이 없다면 어느 방식이든 상관없다. 같은 데이터를 여러 벌 복사하는게 조금 꺼림칙할지 모르지만,

별달리 문제되는 경우는 많지 않아서 흔히 사용하는 방식이다.

복사본이 많이 생겨서 가끔은 메모리가 부족할 수도 있지만, 다른 성능 이슈와 마찬가지로 아주 드문 일이다.

논리적으로 같은 데이터를 물리적으로 복제해 사용할 때 가장 크게 문제되는 상황은 그 데이터를 갱신해야 할 때다.

모든 복제본을 찾아서 빠짐없이 갱신해야 하며, 하나라도 놓치면 데이터 일관성이 깨져버린다.

이런 상황이라면 복제된 데이터들을 모두 참조로 바꿔주는 게 좋다. 데이터가 하나면 갱신된 내용이 해당 고객의 주문 모두에 곧바로 반영되기 때문이다.

값을 참조로 바꾸면 엔티티(entity) 하나 당 객체도 단 하나만 존재하게 되는데, 그러면 보통 이런 객체들을 한데 모아놓고

클라이언트들의 접근을 관리해주는 일종의 저장소가 필요해진다.

각 엔티티를 표현하는 객체를 한 번만 만들고, 객체가 필요한 곳에서는 모두 이 저장소로부터 얻어 쓰는 방식이 된다.

---

### 절차

1. 같은 부류에 속하는 객체들을 보관할 저장소를 만든다(이미 있다면 생략)
2. 생성자에서 이 부류의 객체들 중 특정 객체를 정확히 찾아내는 방법이 있는지 확인한다.
3. 호스트 객체의 생성자들을 수정하여 필요한 객체를 이 저장소에서 찾도록 한다.

   하나 수정할 때마다 테스트한다.

---

### 예시

Order 클래스를 준비했다. 이 클래스는 주문 데이터를 생성자에서 JSON 문서로 입력받아 필드들을 채운다.

이 과정에서 주문 데이터에 포함된 고객 ID를 사용해 고객(customer) 객체를 생성한다.

```jsx
class Oreder {
  constructor(data) {
    this.number = data.number;
    this.customer = new this.customer(data.customer);
    // 다른 데이터를 읽어 들인다.
  }

  get customer() {
    return this.customer;
  }
}

class Customer {
  constructor(id) {
    this.id = id;
  }

  get id() {
    return this.id;
  }
}
```

이런 방식으로 생성한 고객 객체는 값이다. 고객 ID가 123인 주문을 다섯 개 생성한다면 독립된 고객 객체가 다섯 개 만들어진다.

이 중 하나를 수정하더라도 나머지 네 개에는 반영되지 않는다.

이 상황에서, 예컨대 고객 서비스에서 얻어온 데이터를 고객 객체에 추가해야 한다면 다섯 객체 모두를 같은 값으로 갱신해야 한다.

이처럼 객체가 중복해서 만들어지는 상황은 항상 내 신경을 곤두세운다.

이번 예처럼 엔티티를 표현하는 객체가 여러 개 만들어지면 혼란이 생긴다.

설상가상으로 이 객체가 불변이 아니라면 일관성이 깨질 수 있어서 다루기가 더욱 까다로운 문제로 돌변한다.

[1] 항상 물리적으로 똑같은 고객 객체를 사용하고 싶다면 먼저 이 유일한 객체를 저장해둘 곳이 있어야 한다.

객체를 어디에 저장해야 할지는 애플리케이션에 따라 다르겠지만, 간단한 상황이라면 나는 저장소 객체(repository object)를 사용하는 편이다.

```jsx
let repositoryData;

export function initialize() {
  repositoryData = {};
  repositoryData.customers = new Map();
}

export function registerCustomer(id) {
  if (!repositoryData.customer.has(id)) {
    repositoryData.customer.set(id, new Customer(id));
  }
  return findCustomer(id);
}

export function findCustomer(id) {
  return repositoryData.customer.get(id);
}
```

이 저장소는 고객 객체를 ID와 함께 등록할 수 있으며, ID 하나당 오직 하나의 고객 객체만 생성됨을 보장한다.

저장소가 준비되었으니 이제 주문 클래스의 생성자가 이 저장소를 사용하도록 수정할 수 있다.

쓸만한 저장소가 이미 존재할 때도 왕왕 있는데, 그렇다면 그저 그 저장소를 사용하기만 하면 된다.

[2] 다음 단계로는 주문의 생성자에서 올바른 고객 객체를 얻어오는 방법을 강구해야 한다.

이번 예에서는 고객 ID가 입력 데이터 스트림으로 전달되니 쉽게 해결할 수 있다. [3] 수정해보자.

```jsx
class Order {
  constructor(data) {
    this.number = number;
    this.customer = registerCustomer(data.customer);
    // 다른 데이터를 읽어 들인다.
  }

  get customer() {
    return this.customer;
  }
}
```

이제 특정 주문과 관련된 고객 정보를 갱신하면 같은 고객을 공유하는 주문 모두에서 갱신된 데이터를 사용하게 된다.

이 예에서는 특정 고객 객체를 참조하는 첫 번째 주문에서 해당 고객 객체를 생성했다.

또 다른 방법으로, 고객 목록을 미리 다 만들어서 저장소에 저장해놓고 주문 정보를 읽을 때 연결해주는 방법도 자주 사용한다.

이 방식에서는 저장소에 없는 고객 ID를 사용하는 주문에서는 오류가 난다.

이 예시 코드는 생성자 본문이 전역 저장소와 결합된다는 문제가 있다.

전역 객체는 독한 약처럼 신중히 다뤄야 한다.

소량만 사용하면 이로울 수도 있지만 과용하면 독이 된다. 이 점이 염려된다면 저장소를 생성자 매개변수로 전달하도록 수정하자.

---

### 매직 리터럴 바꾸기(Replace Magic Literal)

```jsx
function potentialEnergy(mass, height) {
  return mass * 9.81 * height;
}
```

```jsx
const STANDARD_GRAVITY = 9.81;

function potentialEnergy(mass, height) {
  return mass * STANDARD_GRAVITY * height;
}
```

### 배경

Magic Literal이란 소스 코드에 (보통은 여러 곳에) 등장하는 일반적인 리터럴 값을 말한다.

예컨대 움직임을 계산하는 코드에서라면 9.80665라는 숫자가 산재해 있는 모습을 목격할 수 있다.

내가 물리 수업을 들은 게 언제인지 까마득하지만 이 숫자가 표준중력을 뜻한다는건 기억이 난다.

하지만 코드를 읽는 사람이 이 값의 의미를 모른다면 숫자 자체로는 의미를 명확히 알려주지 못하므로 매직 리터럴이라 할 수 있다.

의미를 알고 있다고 해도 결국 각자의 머리에서 해석해낸 것일 뿐이라서, 이보다는 코드 자체가 뜻을 분명하게 드러내는 게 좋다.

상수를 정의하고 숫자 대신 상수를 사용하도록 바꾸면 될 것이다.

매직 리터럴은 대체로 숫자가 많지만 다른 타입의 리터럴도 특별한 의미를 지닐 수 있다.

예컨대 “1월 1일”은 새로운 해의 시작을, “M”은 남성을, “서울”은 본사를 뜻할 수도 있다.

일반적으로 해당 값이 쓰이는 모든 곳을 적절한 이름의 상수로 바꿔주는 방법이 가장 좋다.

다른 선택지도 있는데, 그 상수가 특별한 비교 로직에 주로 쓰이는 경우에 고려해볼 수 있는 방법이다.

예컨대 나는 aValue === “M” 을 aValue === MALE_GENDER로 바꾸기보다 isMale(Value)라는 함수 호출로 바꾸는 쪽을 선호한다.

상수를 과용하는 모습도 종종 본다. const ONE = 1 같은 선언은 의미가 없다. 의미 전달 면에서 값을 바로 쓰는 것보다 나을 게 없기 때문이다.

또한 리터럴이 함수 하나에서만 쓰이고 그 함수가 맥락 정보를 충분히 제공하여 헷갈릴 일이 없다면 상수로 바꿔 얻는 이득이 줄어든다.

---

### 절차

1. 상수를 선언하고 매직 리터럴을 대입한다.
2. 해당 리터럴이 사용되는 곳을 모두 찾는다.
3. 찾은 곳 각각에서 리터럴이 새 상수와 똑같은 의미로 쓰였는지 확인하여, 같은 의미라면 상수로 대체한 후 테스트한다.

이 리팩터링이 제대로 되었는지는 어떻게 테스트할까? 좋은 방법을 하나 소개하겠다.

상수의 값을 바꾸고, 관련 테스트 모두가 바뀐 값에 해당하는 결과를 내는지 확인하는 것이다.

항상 적용할 수 있는 방법은 아니지만, 가능한 경우라면 아주 편리하다.

---
