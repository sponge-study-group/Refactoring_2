
# 상속 다루기

## 12.1 메서드 올리기 (Pull Up Method)

```javascript
// before

class Employee {...}

class Salesman extends Employee {
  get name() {...}
}

class Engineer extends Employee {
  get name() {...}
}

// after

class Employee {
  get name() {...}
}

class Salesman extends Employee {...}
class Engineer extends Employee {...}
```

<br/>

> java 에서 부모 클래스의 메서드를 extends 로 상속받아오는 것과 동일하다.
> 부모 클래스에서 메서드를 그대로 가져오기 때문에 코드를 간소화시키고 중복 코드를 제거하는 효과가 있다.


## 12.2 필드 올리기 (Pull Up Field)

```javascript
// before

class Employee {...} // Java

class Salesman extends Employee {
  private String name;
}

class Engineer extends Employee {
  private String name;
}
// after

class Employee {
  protected String name;
}

class Salesman extends Employee {...}
class Engineer extends Employee {...}
```

<br/>

> before 코드에서 Salesman과 Engineer 클래스에서 name 필드를 각각 정의하고 있습니다. 이러한 경우에는 Employee 클래스에서 name 필드를 정의하지 않았기 때문에, Salesman과 Engineer 객체에서는 name 필드를 호출할 때 상속 체인을 타고 올라가서 Employee 클래스에는 name 필드가 없으므로 직접 정의한 필드를 사용하게 됩니다. <br/><br/>
> 하지만 after 코드에서는 Employee 클래스에서 name 필드를 정의하고, Salesman과 Engineer 클래스에서는 name 필드를 상속받아 사용합니다. 이 경우에는 Employee 클래스에서 name 필드의 접근 지정자를 protected로 지정하여, 하위 클래스에서 상속받아 사용할 수 있도록 허용하였습니다. <br/><br/>
> 이러한 변경사항은 Salesman과 Engineer 클래스에서 중복되는 name 필드를 제거하여 코드를 간소화시키고, Employee 클래스에서 name 필드를 정의하므로 모든 하위 클래스에서 공통으로 사용할 수 있는 기능을 제공하게 됩니다. 또한, protected 접근 지정자를 사용하여 Employee 클래스에서 정의한 필드에 하위 클래스에서 직접 접근하지 않고도 상속을 통해 사용할 수 있도록 했습니다.


## 12.3 생성자 본문 올리기 (Pull Up Constructor Body)

```javascript
// before

class Party {...}

class Employee extends Party {
  constructor(name, id, monthlyCost) {
    super();
    this._id = id;
    this._name = name;
    this._monthlyCost = monthlyCost;
  }
}
// after

class Party {
  constructor(name){
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

<br/>

> before 코드에서 Employee 클래스가 Party 클래스를 상속하고 있으며, Employee 객체를 생성할 때 name, id, monthlyCost 값을 전달받아 각각 _name, _id, _monthlyCost 필드에 저장하고 있습니다. <br/><br/>
하지만 after 코드에서는 Party 클래스의 생성자를 수정하여 name 값을 전달받도록 변경하였습니다. 그리고 Employee 클래스에서 super 키워드를 사용하여 Party 클래스의 생성자를 호출하고, name 값을 전달하도록 수정하였습니다. 이렇게 함으로써 Employee 클래스에서는 name 필드를 상속받아 사용할 수 있게 되며, Party 클래스에서는 name 필드를 직접 정의하여 Employee 클래스에서 사용할 수 있게 되었습니다. <br/><br/>
이러한 변경사항은 Party 클래스에서 name 필드를 정의함으로써 코드의 재사용성을 높이고, Employee 클래스에서는 name 필드를 상속받아 사용할 수 있으므로 중복 코드를 제거하고 코드를 간소화시킬 수 있게 되었습니다.


## 12.4 메서드 내리기 (Push Down Method)

```javascript
// before

class Employee {
  get quota {...}
}

class Engineer extends Employee {...}
class Salesman extends Employee {...}
// after

class Employee {...}
class Engineer extends Employee {...}
class Salesman extends Employee {
  get quota {...}  
}
```

<br/>

> 특정 서브클래스와만 관련된 메서드는 슈퍼클래스에서 제거하고 해당 서브클래스에서 추가하는 것이 깔끔하다.


## 12.5 필드 내리기 (Push Down Field)

```javascript
// before

class Employee {        // Java
  private String quota;
}

class Engineer extends Employee {...}
class Salesman extends Employee {...}
// after

class Employee {...}
class Engineer extends Employee {...}

class Salesman extends Employee {
  protected String quota;
}
```

<br/><br/>

> 특정 서브클래스와만 관련된 필드는 슈퍼클래스에서 제거하고 해당 서브클래스에서 추가하는 것이 깔끔하다.


## 12.6 타입 코드를 서브클래스로 바꾸기 (Replace Type Code with Subclasses)

```javascript
// before

function createEmployee(name, type) {
  return new Employee(name, type);
}
// after

function createEmployee(name, type) {
  switch (type) {
    case "engineer": return new Engineer(name);
    case "salesman": return new Salesman(name);
    case "manager":  return new Manager (name);
  }
```

<br/><br/>

* 비슷한 대상들을 특정 특성에 따라 구분해야 할 때 사용
  - 예) 담당 업무 (엔지니어, 관리자, 영업자) 주문 시급성 (급함, 보통 등)

> before 코드에서는 createEmployee 함수가 name과 type 매개변수를 받아서 Employee 객체를 생성하여 반환합니다.
<br/><br/>
하지만 after 코드에서는 createEmployee 함수가 type 매개변수를 확인하여, Engineer, Salesman, Manager 중 적절한 클래스를 선택하여 객체를 생성합니다. 이러한 변경사항은 다형성의 개념을 활용하여, Employee 클래스를 상속받은 클래스들을 일관된 방법으로 생성할 수 있도록 하였습니다. 이로 인해 새로운 직원 유형이 추가될 경우, createEmployee 함수에서 해당 유형을 추가하는 것으로 쉽게 처리할 수 있으며, 코드의 확장성과 유지보수성이 향상됩니다.


## 12.7 서브클래스 제거하기 (Remove Subclass)

```javascript
// before

class Person {
  get genderCode() {return "X";}
}
class Male extends Person {
  get genderCode() {return "M";}
}
class Female extends Person {
  get genderCode() {return "F";}
}
// after

class Person {
  get genderCode() {return this._genderCode;}
}
```

<br/><br/>

> before 코드에서는 Person, Male, Female 클래스들이 각각 genderCode 속성을 가지고 있으며, Male 클래스는 "M", Female 클래스는 "F", Person 클래스는 "X" 값을 반환하도록 오버라이딩하고 있습니다.
<br/><br/>
하지만 after 코드에서는 Person 클래스가 genderCode 속성만을 가지고 있으며, 이 속성은 _genderCode로 선언되어 있습니다. 이러한 변경사항은 Person 클래스에 기본 구현을 두고, Male과 Female 클래스에서 이를 오버라이딩하는 대신에, _genderCode 속성을 각 클래스에서 선언하고, 이 값을 Person 클래스의 생성자에서 초기화함으로써, 각 클래스마다 다른 값을 가질 수 있도록 구현하고 있습니다.
<br/><br/>
이러한 변경사항은 Person 클래스를 더욱 일반화된 형태로 구현함으로써, 새로운 성별 유형이 추가될 경우, 쉽게 Person 클래스에서 처리할 수 있도록 하고, 코드의 확장성과 유지보수성을 향상시킵니다.


## 12.8 슈퍼클래스 추출하기 (Extract Superclass)

```javascript
// before

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
// after

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

<br/><br/>

> before 코드에서는 Department와 Employee 클래스가 각각 고유한 속성들을 가지고 있습니다. 그러나 이러한 방식으로는 속성들이 클래스들에 중복되어 있을 가능성이 높아져, 코드의 중복성과 유지보수성이 저하될 수 있습니다.
<br/><br/>
따라서 after 코드에서는 공통된 속성들을 추상화하여 Party 클래스에 정의하고, Department와 Employee 클래스들은 이를 상속받아 필요한 속성들을 추가로 정의하는 방식으로 변경되었습니다. 이러한 변경사항은 코드의 중복성을 줄이고, 유지보수성을 향상시킵니다.


## 12.9 계층 합치기 (Collapse Hierarchy)

```javascript
// before

class Employee {...}
class Salesman extends Employee {...}
// after

class Employee {...}
```

<br/><br/>

> 클래스 계층구조를 리팩터링하다 보면 기능들을 위로 올리거나 아래로 내리는 일은 다반사로 벌어진다.
> 예컨대 계층구조도 진화하면서 어떤 클래스와 그 부모가 너무 비슷해져서 더는 독립적으로 
> 존재해야 할 이유가 사라지는 경우가 생기기도 한다. 바로 그 둘을 하나로 합쳐야 할 시점이다.


## 12.10 서브클래스를 위임으로 바꾸기 (Replace Subclass with Delegate)

```javascript
// before

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
// after

class Order {
  get daysToShip() {
    return (this._priorityDelegate)
      ? this._priorityDelegate.daysToShip
      : this._warehouse.daysToShip;
  }
}

class PriorityOrderDelegate {
  get daysToShip() {
    return this._priorityPlan.daysToShip
  }
}
```

<br/><br/>

> before 코드에서는 Order 클래스와 PriorityOrder 클래스가 존재합니다. PriorityOrder 클래스는 Order 클래스를 상속받아 daysToShip 속성을 오버라이딩하여 재정의합니다. 이러한 구조에서는 우선순위 주문(PriorityOrder)의 경우 daysToShip 속성을 조회할 때 상위 클래스인 Order의 daysToShip 속성이 아닌 하위 클래스인 PriorityOrder의 daysToShip 속성을 사용합니다.
<br/><br/>
하지만 after 코드에서는 PriorityOrder 클래스가 없어졌습니다. 대신 Order 클래스에 우선순위 주문의 daysToShip 속성을 처리하는 PriorityOrderDelegate 클래스가 추가되었습니다. Order 클래스는 daysToShip 속성을 조회할 때 PriorityOrderDelegate 객체가 존재하는 경우에는 이를 사용하고, 존재하지 않는 경우에는 이전과 같이 _warehouse 객체의 daysToShip 속성을 사용합니다.
<br/><br/>
이러한 변경사항은 코드의 복잡성을 줄이고 유연성을 향상시키는 효과가 있습니다. 또한 PriorityOrderDelegate 클래스를 PriorityOrder 클래스보다 더 적극적으로 재사용할 수 있게 되었습니다. 


## 12.11 슈퍼클래스를 위임으로 바꾸기 (Replace Superclass with Delegate)

```javascript
// before

class List {...}
class Stack extends List {...}
// after

class Stack {
  constructor() {
    this._storage = new List();
  }
}
class List {...}
```

<br/><br/>

> before 코드에서는 List 클래스와 Stack 클래스가 상속 관계에 있었습니다. 이것은 Stack 클래스가 List 클래스의 모든 기능과 특성을 상속하게 되는 것을 의미합니다. <br/><br/>
after 코드에서는 List 클래스와 Stack 클래스가 더 이상 상속 관계에 있지 않습니다. 대신, Stack 클래스는 List 클래스의 인스턴스를 내부적으로 사용합니다. 이것은 Stack 클래스가 List 클래스의 기능 중 필요한 것만 사용할 수 있도록 하는 것입니다. <br/><br/>
이 변경 사항은 코드 구조를 더 간단하고 유연하게 만들어줍니다. 또한, Stack 클래스는 이제 List 클래스의 구현 세부 정보에 덜 의존하게 되므로 코드 수정이 더 쉬워질 수 있습니다.


<br/><br/>
### 🐈‍⬛ Reference 
* [이재원의 티스토리](https://slog2.tistory.com/25)
* [ChatGPT](https://chat.openai.com/)




