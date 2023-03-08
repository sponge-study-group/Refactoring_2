## 12. 상속 다루기

### 12.1 메서드 올리기

<br />

```javascript
class Employee {...}

class Salesman extends Employee {
  get name() {...}
}

class Engineer extends Employee {
  get name() {...}
}
```

↓

```javascript
class Employee {
  get name() {...}
}

class Salesman extends Employee {...}
class Engineer extends Employee {...}
```

<br />

**배경**  
메서드 올리기를 적용하기 가장 쉬운 상황  
→ **메서드 들의 본문 코드가 똑같을 때**

메서드 올리기 리팩터링을 적용하려면 선행 단계를 거쳐야 할 때가 많다.  
예를 들어, 서로 다른 두 클래스의 두 메서드를 각각 매개변수화면 같은 메서드가 되기도 한다.  
→ 이런 경우에는 각각의 **함수를 매개변수화**한 다음 메서드를 상속 계층의 위로 올리면 된다.

해당 메서드의 본문에서 참조하는 필드들이 서브클래스에만 있다면  
→ **필드를 먼저 슈퍼클래스로 올린 후**에 메서드를 올리자.

두 메서드의 전체 흐름은 같지만 세부 내용이 다르다면  
→ **템플릿 메서드 만들기**를 고려

**절차**  
① 똑같이 동작하는 메서드인지 살펴본다.  
② 메서드 안에서 호출하는 다른 메서드와 참조하는 필드들을 슈퍼클래스에서도 호출하고 참조할 수 있는지 확인  
③ 메서드 시그니처가 다르다면 **함수 선언 바꾸기**로 슈퍼클래스에서 사용하고 싶은 형태로 통일  
④ 슈퍼 클래스에 새로운 메서드를 생성, 대상 메서드의 코드를 복사해서 넣는다.  
⑤ 정적 검사 수행  
⑥ 서브클래스 중 하나의 메서드 제거  
⑦ 테스트  
⑧ 모든 서브클래스의 메서드가 없어질 때까지 다른 서브클래스의 메서드를 하나씩 제거

---

### 12.2 필드 올리기

<br />

```javascript
class Employee {...} // 자바 코드

class Salesman extends Employee {
  private String name;
}

class Engineer extends Employee {
  private String name;
}
```

↓

```javascript
class Employee {
  protected String name;
}

class Salesman extends Employee {...}
class Engineer extends Employee {...}
```

<br />

**배경**  
필드들이 비슷한 방식으로 쓰인다고 판단  
→ 슈퍼클래스로 끌어올리자.

해당 리팩터링을 통해 두 가지 중복 제거

1. 데이터 중복 선언 제거
2. 해당 필드를 사용하는 동작을 서브클래스에서 슈퍼클래스로 이동

**절차**  
① 후보 필드들을 사용하는 곳 모두가 그 필드들을 똑같은 방식으로 사용하는지 확인  
② 필드들의 이름이 각기 다르다면 똑같은 이름으로 변경(**필드 이름 바꾸기**)
③ 슈퍼클래스에 새로운 필드 생성  
④ 서브클래스의 필드들을 제거  
⑤ 테스트

---

### 12.3 생성자 본문 올리기

<br />

```javascript
class Party {...}

class Employee extends Party {
  constructor(name, id, monthlyCost) {
    super();
    this._id = id;
    this._name = name;
    this._monthlyCost = monthlyCost;
  }
}
```

↓

```javascript
class Party {
  constructor(name) {
    this._name = name;
  }
}

class Employee extends Party {
  constructor(name, id, monthlyCost) {
    super(name);
    this._id = id;
    this._monthlyCost = monthlyCost;
  }
}
```

<br />

**배경**  
생성자 → 일반 메서드와 달라, 다루기 까다로워서 일부 제약을 둔다.

생성자는 할 수 있는 일과 호출 순서에 제약이 있어 다른 식으로 접근해야 한다.

**절차**  
① 슈퍼클래스에 생성자가 없다면 하나 정의. 서브클래스의 생성자들에서 이 생성자가 호출되는지 확인  
② **문장 슬라이드하기**를 통해 공통 문장 모두를 super() 호출 직후로 이동  
③ 공통 코드를 슈퍼클래스에 추가하고 서브클래스에서는 제거. 생성자 매개변수 중 공통 코드에서 참조하는 값들을 모두 super()로 건넨다.  
④ 테스트  
⑤ 생성자 시작 부분으로 옮길 수 없는 공통 코드에는 **함수 추출하기**와 **메서드 올리기**를 차례로 적용

---

### 12.4 메서드 내리기

<br />

```javascript
class Employee {
  get quota {...}
}

class Engineer extends Employee {...}
class Salesman extends Employee {...}
```

↓

```javascript
class Employee {...}
class Engineer extends Employee {...}

class Salesman extends Employee {
  get quota {...}
}
```

<br />

**배경**  
**서브클래스가 정확히 무엇인지를 호출자가 알고 있을 때**만 해당 리팩터링 적용 가능  
→ 그렇지 않은 상황이라면, 서브클래스에 따라 다르게 동작하는 슈퍼클래스의 **조건부 로직을 다형성으로 바꿔야 한다.**

**절차**  
① 대상 메서드를 모든 서브클래스에 복사  
② 슈퍼클래스에서 그 메서드를 제거  
③ 테스트  
④ 이 메서드를 사용하지 않는 모든 서브클래스에서 제거  
⑤ 테스트

---

### 12.5 필드 내리기

<br />

```javascript
class Employee { // 자바 코드
  private String quota;
}

class Engineer extends Employee {...}
class Salesman extends Employee {...}
```

↓

```javascript
class Employee {...}
class Engineer extends Employee {...}

class Salesman extends Employee {
  protected String quota;
}
```

<br />

**배경**  
서브클래스 하나에서만 사용하는 필드는 해당 서브클래스로 옮긴다.

**절차**  
① 대상 필드를 모든 서브클래스에 정의  
② 슈퍼클래스에서 그 필드 제거  
③ 테스트  
④ 이 필드를 사용하지 않는 모든 서브클래스에서 제거  
⑤ 테스트

---

### 12.6 타입 코드를 서브클래스로 바꾸기

<br />

```javascript
function createEmployee(name, type) {
  return new Employee(name, type);
}
```

↓

```javascript
function createEmployee(name, type) {
  switch (type) {
    case "engineer": return new Engineer(name);
    case "salesman": return new Salesman(name);
    case "manager":  return new Manager (name);
  }

  ...
}
```

<br />

**배경**  
비슷한 대상들을 특성에 따라 구분하기 위해, 타입 코드 이상의 무언가가 필요할 때  
→ 서브클래스를 통해 해결

서브클래스의 장점

1. 조건에 따라 다르게 동작해주도록 해주는 다형성 제공
2. 특정 타입에서만 의미가 있는 값을 사용하는 필드나 메서드가 있을 때 발현  
   → 예를 들어, '판매 목표'는 '영업자' 유형일 때만 의미가 있어서, 이런 상황에선 서브클래스를 만들고 필요한 서브클래스만 필드를 갖도록 정의하자.

대상 클레스에 직접 적용  
→ 하위 타입을 만든다.  
→ 간단하지만 유형을 다른 용도로 사용할 수 없다.  
→ 유형이 불변일 때도 직접 서브클래싱 방식은 이용할 수 ×

타입 코드 자체에 적용  
→ 유형 속성을 부여하고, 해당 속성을 클래스로 정의해 같은 서브클래스를 만든다.  
→ 타입 코드에 **기본형을 객체로 바꾸기**를 적용하여 직원 유형 클래스를 만든 다음, 이 클래스에 해당 리팩터링 적용

**절차**  
① 타입 코드 필드를 자가 캡슐화.  
② 타입 코드 값 하나를 선택하여 그 값에 해당하는 서브클래스 생성. 타입 코드 게터 메서드를 오버라이드하여 해당 타입 코드의 리터럴 값 반환.  
③ 매개변수로 받은 타입 코드와 방금 만든 서브클래스를 매핑하는 선택 로직 추가  
④ 테스트  
⑤ 타입 코드 값 각각에 대해 서브클래스 생성과 선택 로직 추가 반복. 클래스 하나가 완성될 때마다 테스트.  
⑥ 타입 코드 필드 제거  
⑦ 테스트  
⑧ 타입 코드 접근자를 이용하는 메서드 모두 **메서드 내리기**와 **조건부 로직을 다형성으로 바꾸기** 적용

---

### 12.7 서브클래스 제거하기

<br />

```javascript
class Person {
  get genderCode() {
    return 'X';
  }
}

class Male extends Person {
  get genderCode() {
    return 'M';
  }
}

class Female extends Person {
  get genderCode() {
    return 'F';
  }
}
```

↓

```javascript
class Person {
  get genderCode() {
    return this._genderCode;
  }
}
```

<br />

**배경**  
시스템이 커지면서 서브클래스로 만든 변종이 다른 모듈로 이동하거나 완전히 사라지기도 한다.  
→ 서브클래스를 슈퍼클래스의 필드로 대체해 제거하자.

**절차**  
① 서브클래스의 생성자를 **팩터리 함수로 변경**  
② 서브클래스의 타입을 검사하는 코드 → 검사 코드에 **함수 추출하기**와 **함수 옮기기**를 적용해 슈퍼클래스로 이동. 변경할 떄마다 테스트.  
③ 서브클래스의 타입을 나타내는 필드를 슈퍼클래스에 추가  
④ 서브클래스를 참조하는 메서드가 해당 타입 필드를 이용하도록 수정  
⑤ 서브클래스 삭제  
⑥ 테스트

---

### 12.8 슈퍼클래스 추출하기

<br />

```javascript
class Department {
  get totalAnnualCost() {...}
  get name() {...}
  get headCount() {...}
}

class Employee {
  get annualCost() {...}
  get name() {...}
  get id() {...}
}
```

↓

```javascript
class Party {
  get name() {...}
  get annualCost() {...}
}

class Department extends Party {
  get annualCost() {...}
  get headCount() {...}
}

class Employee extends Party {
  get annualCost() {...}
  get id() {...}
}
```

<br />

**배경**
비슷한 일을 수행하는 두 클래스가 보이면 상속 매커니즘을 이용해 비슷한 부분을 공통의 슈퍼클래스로 옮겨 담을 수 있다.  
→ 공통된 부분이 데이터라면 **필드 올리기**를 활용  
→ 동작이라면 **메서드 올리기** 활용

선택과 해결방법(상속/위임)에 따라 **슈퍼클래스 추출하기**와 **클래스 추출하기**로 나눌 수 있다.  
→ 슈퍼클래스 추출하기 방법이 더 간단할 경우가 많아, 해당 리팩터링을 먼저 시도

**절차**  
① 빈 슈퍼클래스 생성. 원래의 클래스들이 새 클래스에 상속하도록 만든다.  
② 테스트  
③ **생성자 본문 올리기**, **메서드 올리기**, **필드 올리기**를 차례로 적용해 공통 원소를 슈퍼클래스로 옮긴다.  
④ 서브클래스에 남은 메서드들 검토. 공통되는 부분이 있다면 **함수로 추출**한 다음 **메서드 올리기** 적용  
⑤ 원래 클래스들을 사용하는 코드를 검토 → 슈퍼클래스의 인터페이스를 사용할지 고민

---

### 12.9 게층 합치기

<br />

```javascript
class Employee {...}
class Salesman extends Employee {...}
```

↓

```javascript
class Employee {...}
```

<br />

**배경**
클래스 계층구조가 진화하면서 클래스와 그 부모가 너무 비슷해져 더는 독립적으로 존재할 이유가 사라지는 경우가 생긴다.  
→ 둘을 하나로 합쳐야 할 시점

**절차**  
① 두 클래스 중 제거할 것을 선택  
② **필드 올리기**와 **메서드 올리기** 혹은 **필드 내리기**와 **메서드 내리기**를 적용하여 모든 요소를 하나의 클래스로 이동  
③ 제거할 클래스를 참조하던 모든 코드가 남겨질 클래스를 참조하도록 수정  
④ 빈 클래스 제거  
⑤ 테스트

---

### 12.10 서브클래스를 위임으로 바꾸기

<br />

```javascript
class Order {
  get daysToShip() {
    return this._warehouse.daysToShip;
  }
}

class PriorityOrder extends Order {
  get daysToShip() {
    return this._priorityPlan.daysToShip;
  }
}
```

↓

```javascript
class Order {
  get daysToShip() {
    return this._priorityDelegate ? this._priorityDelegate.daysToShip : this._warehouse.daysToShip;
  }
}

class PriorityOrderDelegate {
  get daysToShip() {
    return this._priorityPlan.daysToShip;
  }
}
```

<br />

**배경**
속한 갈래에 따라 동작이 달라지는 객체들은 상속으로 표현하는게 자연스럽다.  
→ 공통 데이터와 동작은 모두 슈퍼클래스에 두고 서브클래스는 자신에 맞게 기능을 추가하거나 오버라이드

상속에서의 단점

1. 한 번만 사용할 수 있다.  
   → 달라져야 하는 이유가 여러개여도 상속에서는 단 하나의 이유만 선택해 기준으로 삼아야 한다.
2. 상속은 클래스들의 관계를 아주 긴밀하게 결합한다.  
   → 부모를 수정하면 자식들의 기능을 해치지 쉽다.  
   → 부모와 자식이 서로 다른 모듈에 속하거나 다른 팀에서 구현한다면 문제가 더 복잡해진다.

**위임**을 통해 해당 문제를 해결할 수 있다.
→ 다양한 클래스에 서로 다른 이유로 위임할 수 있다.  
→ 위임은 객체 사이의 일반적은 관계이므로 상호작용에 필요한 인터페이스를 명확히 정의할 수 있다.

처음에는 상속으로 접근한 다음, 문제가 생기기 시작하면 위임으로 갈아탄다.

서브클래스를 위임으로 바꾸는 모든 경우에서 위임을 계층구조로 설게해야 하는 건 아니지만,
_상태나 전략에 계층구조를 적용하면 유용할 때가 많다._

**절차**  
① 생성자를 호출하는 곳이 많다면 **생성자를 팩터리 함수로 변경**  
② 위임으로 활용할 빈 클래스 생성  
→ 이 클래스의 생성자는 서브클래스에 특화된 데이터를 전부 받아야 한다.  
→ 슈퍼클래스를 가리키는 역참조도 필요  
③ 위임을 저장할 필드를 슈퍼클래스에 추가  
④ 서브클래스 생성 코드를 수정  
→ 위임 인스턴스를 생성하고 위임 필드에 대입해 초기화  
⑤ 서브클래스의 메서드 중 위임 클래스로 이동할 것을 선택  
⑥ **함수 옮기기**를 적용해 위임 클래스로 이동. 원래 메서드에서 위임하는 코드는 삭제 ✕  
⑦ 서브클래스 외에도 원래 메서드를 호출하는 코드가 있다면 서브클래스의 위임 코드를 슈퍼클래스로 이동.  
→ 위임이 존재하는지 검사하는 보호 코드로 감싸야 한다.  
→ 외부 코드가 없다면 원래 메서드는 **죽은 코드가 되므로 제거**  
⑧ 테스트  
⑨ 서브클래스의 모든 메서드가 옮겨질 때까지 해당 과정 반복  
⑩ 서브클래스들의 생성자를 호출하는 코드를 찾아서 슈퍼클래스의 생성자를 사용하도록 수정  
⑪ 테스트  
⑫ 서브클래스 삭제(**죽은 코드 제거하기**)

---

### 12.11 슈퍼클래스를 위임으로 바꾸기

<br />

```javascript
class List {...}
class Stack extends List {...}
```

↓

```javascript
class Stack {
  constructor() {
    this._storage = new List();
  }
}

class List {...}
```

<br />

**배경**
상속이 오히려 혼란과 복잡도를 키울 수도 있다.  
→ 자바의 스택은 리스트를 상속  
→ 리스트의 연산 중 스택에는 적용되지 않는게 많지만 모든 연산이 스택 인터페이스에 그대로 노출된다.  
→ 슈퍼클래스의 기능들이 서브클래스에 어울리지 않는다면 그 기능들을 상속을 통해 이용하면 안된다.

제대로 된 상속이라면  
→ 서브클래스가 슈퍼클래스의 모든 기능을 사용함을 물론, 서브클래스의 인스턴스를 슈퍼클래스의 인스턴스로 취급할 수 있어야 한다.  
→ 슈퍼클래스가 사용 되는 모든 곳에서 서브클래스의 인스턴스를 대신 사용해도 이상없이 동작해야 한다.

서브클래스 방식 모델링이 합리적일 때라도 슈퍼클래스를 위임으로 바꾸기도 한다.

위임의 기능을 이용할 호스트의 함수 모두를 전달 함수로 만들어야 한다.  
→ 단순해서 문제가 생길 가능성 ↓

의미상 적합한 조건이라면 상속은 간단하고 효과적인 매커니즘  
→ 상황이 변해 상속이 최선의 방법이 아니게 된다면 언제든 이번 리팩터링을 통해 슈퍼클래스를 위임으로 바꿀 수 있다.  
→ 그래서 상속을 먼저 적용하고 나중에 문제가 생기면 슈퍼클래스를 위임으로 바꾸자.

**절차**  
① 슈퍼클래스 객체를 참조하는 필드를 서브클래스에 만든다(위임 참조). 위임 참조를 새로운 슈퍼클래스 인스턴스로 초기화한다.  
② 슈퍼클래스의 동작 각각에 대응하는 전달 함수를 서브클래스에 만든다. 서로 관련된 함수끼리 그룹으로 묶어 진행하며, 하나씩 만들 떄마다 테스트  
③ 슈퍼클래스의 동작 모두가 전달 함수로 오버라이드되었다면 상속 관계를 끊는다.
