## 7. 캡슐화

### 7.1 레코드 캡슐화 하기

<br />

```javascript
organization = { name: '애크미 구스베리', county: 'GB' };
```

↓

```javascript
class Organization {
  constructor(data) {
    this._name = data.name;
    this._county = data.county;
  }

  get name() {
    return this._name;
  }
  set name(arg) {
    this._name = arg;
  }
  get county() {
    returnthis._county;
  }
  set county(arg) {
    this._county = arg;
  }
}
```

<br />

**배경**

대부분의 프로그래밍 언어는 데이터 레코드를 표현하는 구조 제공  
→ 단순한 레코드는 계산해서 얻을 수 있는 값과 그렇지 않은 값을 명확히 구분해서 저장해야 하는 점이 번거롭다.  
→ 가변 데이터를 저장하는 용도로는 레코드보다 객체 선호

프로그램에서 해시맵을 쓰는 부분이 많을수록 불분명함으로 인해 발생하는 문제 ↑  
→ 레코드 대신 클래스를 사용하는 편이 낫다.

**절차**

1. 레코드를 담은 변수를 캡슐화
2. 레코드를 감싼 단순한 클래스로 해당 변수의 내용 교체. 이 클래스에 원본 레코드를 반환하는 접근자도 정의 + 변수를 캡슐화하는 함수들이 이 접근자를 사용하도록 수정
3. 테스트
4. 새로 정의한 클래스 타입의 객체를 반환하는 함수 생성
5. 예전 함수를 사용하는 코드를 4에서 만든 새 함수를 사용하도록 변경. 필드에 접근할 때는 객체 접근자 사용. 적절한 접근자가 없다면 추가. 한 부분을 바꿀 때마다 테스트
6. 클래스에서 원본 데이터를 반환하는 접근자와 원본 레코드를 반환하는 함수 제거
7. 테스트
8. 레코드의 필드도 중첩 구조라면 레코드 캡슐화하기와 컬렉션 캡슐화하기를 재귀적으로 적용

**예시: 간단한 레코드 캡슐화하기**

```javascript
organization = { name: '애크미 구스베리', county: 'GB' };
```

① 상수를 캡슐화(변수 캡슐화하기)

```javascript
// 게터를 찾기 쉽도록 임시로 이름을 붙였다.
function getRawDataOfOrganization() {
  return organization;
}
```

② 레코드를 클래스로 변경  
④ 새 클래스의 인스턴스를 반환하는 함수 생성

```javascript
class Organization {
  constructor(data) {
    this._data = data;
  }
}

function getRawDataOfOrganization() {
  return organization._data;
}

function getOrganization() {
  return organization;
}
```

⑤ 레코드를 갱신하던 코드는 모두 세터를 사용하도록 수정

```javascript
class Organization {
  constructor(data) {
    this._data = data;
  }

  get name() {
    return this._name;
  }
  set name(aString) {
    this._name = aString;
  }
}

// 클라이언트
getOrganization().name = newName;
result += `<h1>${getOrganization().name}</h1>`;
```

⑥ 임시 함수 제거

```javascript
/*
function getRawDataOfOrganization() {
  return organization._data;
}
*/

class Organization {
  constructor(data) {
    this._name = data.name;
    this._county = data.county;
  }

  get name() {
    return this._name;
  }
  set name(aString) {
    this._name = aString;
  }
  get county() {
    returnthis._county;
  }
  set county(aString) {
    this._county = aString;
  }
}
```

입력 데이터와 레코드와의 연결을 끊어준다는 이점

---

### 7.2 컬렉션 캡슐화하기

<br />

```javascript
class Person {
  get courses() {
    return this._courses;
  }
  set courses(aList) {
    this._courses = aList;
  }
}
```

↓

```javascript
class Person {
  get courses() { return this._courses.slice() }
  addCourse(aCoures) {...}
  removeCourse(aCoures) {...}
}
```

<br />

**배경**

게터가 컬렉션 자체를 변환한다면, 그 컬렉션을 감싼 클래스가 눈치채지 못하는 상태에서 컬렉션의 원소들이 바뀔 수도 있다.  
→ 컬렉션을 감싼 클래스에 add()와 remove()라는 이름의 컬렉션 변경자 메서드를 생성  
→ 컬렉션을 소유한 클래스를 통해서만 원소를 변경

컬렉션 게터가 원본 컬렉션을 반환하지 않게 만들어서 실수로 컬렉션을 바꿀 가능성을 차단  
→ 절대로 컬렉션 값을 반환하지 않게 만들기  
→ 컬렉션을 읽기 전용으로 제공  
→ 컬렉션 게터를 제공하지만 내부 컬렉션의 복제본을 반환

앞선 방법 중에서 한 가지만 적용해서 컬렉션 접근 함수의 동작 방식을 통일

**절차**

1. 컬렉션을 캡슐화 하지 ×, 변수 캡슐화부터 적용
2. 컬렉션에 원소를 추가/제거하는 함수 추가
3. 정적 검사 수행
4. 컬렉션을 참조하는 부분을 모두 찾아 컬렉션의 변경자를 호출하는 코드가 있다면 원소를 추가/제거하는 함수를 호출하도록 수정. 하나씩 수정할 때마다 테스트
5. 컬렉션 게터를 읽기전용 프락시나 복제본을 반환하도록 수정
6. 테스트

**예시**

```javascript
// Person 클래스
constructor(name) {
  this._name = name;
  this._courses = [];
}

get name() { return this._name; }
get courses() { return this._courses; }
set courses(aList) { this._courses = aList; }

// Course 클래스
constructor(name, isAdvanced) {
  this._name = name;
  this._isAdvanced = isAdvanced;
}

get name() { return this._name; }
get isAdvanced() { return this._isAdvanced; }
```

세터를 이용해 수업 컬렉션 전체를 설정  
→ 누구든 이 컬렉션을 마음대로 수정할 수 있다.

② 클라이언트가 수업을 하나씩 추가하고 제거하는 메서드를 Person에 추가

```javascript
// Person 클래스
addCourse(aCourse) {
  this._Courses.push(aCourse);
}

removeCourse(aCourse, fnIfAbsent = () => { throw new RangeError() })  {
  const index = this._courses.indexOf(aCourse)

  if( index === -1 ) fnIfAbsent()
  else this._courses.splice(index, 1)
}
```

④ 컬렉션의 변경자를 직접 호출하던 코드를 방금 추가한 메서드를 사용하도록 변경

```javascript
// 클라이언트
for (const name of readBasicCourseNames(filename)) {
  aPerson.addCourse(new Course(name, false));
}
```

② 개별 원소를 추가하고 제거하는 메소드를 제공하기 때문에 setCourses()는 제거(세터 제거하기)  
→ 세터를 제공해야 할 이유가 있다면 인수로 받은 컬렉션의 복제본을 필드에 저장

⑤ 메서드들을 사용해야만 목록을 변경 가능

```javascript
// Person 클래스
get courses() { return this._courses.slice() }
```

---

### 7.3 기본형을 객체로 바꾸기

<br />

```javascript
orders.filter((o) => 'high' === o.priority || 'rush' === o.priority);
```

↓

```javascript
orders.filter((o) => o.priority.higherThan(new Priority('normal')));
```

<br />

**배경**

단순한 출력 이상의 기능이 필요해지는 순간, 그 데이터를 표현하는 전용 클래스 정의  
→ 특별한 동작이 필요해지면 해당 클래스에 추가하면 되니 프로그램이 커질수록 유용해진다.

**절차**

1. 아직 변수를 캡슐화 ×, 캡슐화 진행
2. 단순한 값 클래스 생성. 생성자는 기존 값을 인수로 받아서 저장. 이 값을 반환하는 게터 추가
3. 정적 검사 수행
4. 값 클래스의 인스턴스를 새로 생성해서 필드에 저장하도록 세터를 수정. 이미 있다면 필드의 타입을 적절히 변경
5. 새로 만든 클래스의 게터를 호출한 결과를 반환하도록 게터 수정
6. 테스트
7. 함수 이름을 바꾸면 원본 접근자의 동작을 더 잘 드러낼 수 있는지 검토

**예시**

```javascript
// Order 클래스
constructor(data) {
  this.priority = data.priority

  ...
}

// 클라이언트
orders.filter((o) => 'high' === o.priority || 'rush' === o.priority).length;
```

① 변수 캡슐화

```javascript
// Order 클래스
constructor(data) {
  get priority() { return this._priority }
  set priority(aString) { this._priority = aString }
}
```

② 우선수위 속성을 표현하는 값 클래스 Priority 생성

```javascript
// Priority 클래스
class Priority {
  constructor(value) {
    this._value = value;
  }

  // 클라이언트 입장에서는 해당 속성을 문자열로 표현한 값을 요청한게 되기 때문에 변환 함수(toString()) 선호
  toString() {
    return this._value;
  }
}
```

④⑤ Priority 클래스를 사용하도록 접근자 수정
⑦ 게터가 반환하는 값은 우선수위 자체가 아니라 우선순위를 표현하는 문자열 → 함수 선언 바꾸기

```javascript
// Order 클래스
constructor(data) {
  get priorityString() { return this._priority.toString() }
  set priority(aString) { this._priority = new Priority(aString) }
}

// 클라이언트
orders.filter((o) => 'high' === o.priorityString || 'rush' === o.priorityString).length;
```

---

### 7.4 임시 변수를 질의 함수를 바꾸기

<br />

```javascript
const basePrice = this._quantity * this._itemPrice;
if (basePrice > 1000) return basePrice * 0.95;
else return basePrice * 0.98;
```

↓

```javascript
get basePrice() {this._quantity * this._itemPrice;} ...
if (this.basePrice > 1000) return this.basePrice * 0.95;
else return this.basePrice * 0.98;
```

<br />

**배경**

함수 안에서 어떤 코드의 결과값을 다시 참조할 목적으로 임시 변수 사용  
→ 아예 함수로 만들어 사용하는 편이 나을 때가 많다.

변수 대신 함수로 만들어두면 비슷한 계산을 수행하는 다른 함수에서도 사용 ○  
→ 코드 중복 ↓

**절차**

1. 변수가 사용되기 전에 값이 확실히 결정되는지, 변수를 사용할 때마다 계산 로직이 매번 다른 결과를 내지 않는지 확인
2. 읽기전용으로 만들 수 있는 변수는 읽기전용으로 생성
3. 테스트
4. 변수 대입문을 함수로 추출
5. 테스트
6. 변수 인라인하기로 임시 변수를 제거

---

### 7.5 클래스 추출하기

<br />

```javascript
class Person {
  get officeAreaCode() {
    return this._officeAreaCode;
  }

  get officeNumber() {
    return this._officeNumber;
  }
}
```

↓

```javascript
class Person {
  get officeAreaCode() {
    return this._telephoneNumber.areaCode;
  }

  get officeNumber() {
    return this._telephoneNumber.number;
  }
}

class TelephoneNumber {
  get areaCode() {
    return this._areaCode;
  }

  get number() {
    return this._number;
  }
}
```

<br />

**배경**

클래스는 반드시 명확하게 추상화하고 소수의 주어진 역할만 처리해야 한다.  
→ 실무에서는 몇 가지 연산을 추가하고 데이터도 보강하면서 클래스가 비대해진다.  
→ 메서드와 데이터가 너무 많은 클래스는 이해하기 어려우니 적절히 분리하는 것이 좋다.

- 일부 데이터와 메서드를 따로 묶을 수 있다면 분리하라는 신호
- 함께 변경되는 일이 많거나 서로 의존하는 데이터들도 분리
- 특정 데이터나 메서드 일부를 제거하면 어떤 일이 일어나는지 자문  
  → 제거해도 다른 필드나 메서드들이 논리적으로 문제가 없다면 분리 가능

**절차**

1. 클래스의 역할을 분리할 방법을 정한다.
2. 분리될 역할을 담당할 새로운 클래스 생성
3. 원래 클래스의 생성자에서 새로운 클래스의 인스턴스 생성해서 필드에 저장
4. 분리될 역할에 필요한 필드들을 새 클래스로 이동. 하나씩 옮길 때마다 테스트
5. 메서드들도 새 클래스로 이동. 호출 당하는 일이 많은 메서드부터 옮긴다. 하나씩 옮길 때마다 테스트
6. 불필요한 메서드를 제거. 이름도 새로운 환경에 맞춰 변경
7. 새 클래스를 노출한다면 새 클래스에 **참조를 값으로 바꾸기**를 적용할 지 고민

**예시**

```javascript
// Person 클래스
get name() { return this._name }
set name(arg) { this._name = arg }
get telephoneNumber() { return `(${this.officeAreaCode}) ${this.officeNumber}` }
get officeAreaCode() { return this._officeAreaCode }
set officeAreaCode(arg) { this._officeAreaCode = arg }
get officeNumber() { return this._officeNumber }
set officeNumber(arg) { this._officeNumber = arg }
```

① 전화번호 관련 동작을 별도 클래스로 제외  
② 빈 전화번호를 표현하는 TelephoneNumber 클래스 정의
③ Person 클래스의 인스턴스를 생성할 때, 전화번호 인스턴스도 함께 생성해 저장

```javascript
// Person 클래스
constructor() {
  this._telephoneNumber = new TelephoneNumber()
}

// TelephoneNumber 클래스
class TelephoneNumber {
  get officeAreaCode() {
    return this._officeAreaCode;
  }

  set officeAreaCode(arg) {
    this._officeAreaCode = arg
  }
}
```

④ 필드들을 새 클래스로 옮긴다.

```javascript
// Person 클래스
get officeAreaCode() { return this._telephoneNumber.officeAreaCode }
set officeAreaCode(arg) { this._telephoneNumber.officeAreaCode = arg }
```

⑤ telephoneNumber() 메서드를 옮긴다.

```javascript
// Person 클래스
get telephoneNumber() { return this._telephoneNumber.telephoneNumber }

// TelephoneNumber 클래스
get telephoneNumber() { return `(${this.officeAreaCode}) ${this.officeNumber}` }
```

⑥ 함수의 이름을 적절하게 변경 → 함수 선언 바꾸기

```javascript
class Person {
  get officeAreaCode() {
    return this._telephoneNumber.areaCode;
  }

  set officeAreaCode(arg) {
    this._telephoneNumber.areaCode = arg;
  }

  get officeNumber() {
    return this._telephoneNumber.number;
  }

  set officeNumber(arg) {
    this._telephoneNumber.number = arg;
  }

  get telephoneNumber() {
    return this._telephoneNumber.toString();
  }
}

class TelephoneNumber {
  get areaCode() {
    return this._areaCode;
  }

  set areaCode(arg) {
    this._areaCode = arg;
  }

  get number() {
    return this._number;
  }

  set number(arg) {
    this._number = arg;
  }

  toString() {
    return `(${this.areaCode}) ${this.number}`;
  }
}
```

---

### 7.6 클래스 인라인하기

<br />

```javascript
class Person {
  get officeAreaCode() {
    return this._telephoneNumber.areaCode;
  }

  get officeNumber() {
    return this._telephoneNumber.number;
  }
}

class TelephoneNumber {
  get areaCode() {
    return this._areaCode;
  }

  get number() {
    return this._number;
  }
}
```

↓

```javascript
class Person {
  get officeAreaCode() {
    return this._officeAreaCode;
  }

  get officeNumber() {
    return this._officeNumber;
  }
}
```

<br />

**배경**

클래스 인라인하기  
→ 클래스 추출하기를 거꾸로 돌리는 리팩터링  
→ 제 역할을 못 하는 클래스는 이 클래스를 가장 많이 사용하는 클래스로 인라인

두 클래스의 기능을 지금과 다르게 배분하고 싶을 때로 클래스를 인라인  
→ 클래스를 인라인한 다음 새로운 클래스로 추출하는게 쉬울 수도 있다.

**절차**

1. 소스 클래스의 각 public 메서드에 대응하는 메서드들을 타깃 클래스에 생성. 단순히 작업을 소스 클래스로 위임
2. 소스 클래스의 메서드를 사용하는 코드 → 모두 타깃 클래스의 위임 메서드를 사용하도록 변경. 하나씩 변경할 때마다 테스트.
3. 소스 클래스의 메서드와 필드를 모두 타깃 클래스로 이동. 하나씩 옮길 때마다 테스트
4. 소스 클래스를 삭제

---

### 7.7 위임 숨기기

<br />

```javascript
manager = aPerson.department.manager;
```

↓

```javascript
manager = aPerson.manager;

class Person {
  get manager() {
    return this.department.manager;
  }
}
```

<br />

**배경**

모듈화 설계를 제대로 하는 핵심은 캡슐화  
→ 캡슐화는 모듈들이 시스템의 다른 부분에 대해 알아야 할 내용을 ↓
→ 변경할 때 함께 고려해야 할 모듈의 수 ↓, 코드를 변경하기 쉬워진다.

**절차**

1. 위임 객체의 각 메서드에 해당하는 위임 메서드를 서버에 저장
2. 클라이언트가 위임 객체 대신 서버를 호출하도록 수정. 하나씩 변경할 때마다 테스트.
3. 모두 수정했다면, 서버로부터 위임 객체를 얻은 접근자 제거
4. 테스트

**예시**

```javascript
// Person 클래스
constructor(name) {
  this._name = name;
}
get name() { return this._name }
get department() { return this._department }
set department(arg) { this._department = arg }

// Department 클래스
get chargeCode() { return this._chargeCode }
set chargeCode(arg) { this._chargeCode = arg }
get manager() { return this._manager }
set manager(arg) { this._manager = arg }

// 클라이언트
manager = aPerson.department.manager
```

① 클라이언트는 Department 클래스가 관리자 정보를 제공한다는 사실을 알아야 한다.  
→ 이러한 의존성을 줄이려면, Department 클래스를 숨기고 대신 Person 클래스에 간단한 위임 메서드 추가  
② 클라이언트가 이 메서드를 사용하도록 수정  
③ 클라이언트 코드 수정이 끝났다면 Person 클래스의 department() 접근자 삭제

```javascript
// Person 클래스
get manager() { return this._department.manager }

// 클라이언트
manager = aPerson.manager
```

---

### 7.8 중개자 제거하기

<br />

```javascript
manager = aPerson.manager;
class Person {
  get manager() {
    return this.department.manager;
  }
}
```

↓

```javascript
manager = aPerson.department.manager;
```

<br />

**배경**

위임 숨기기를 통해 리팩터링  
→ 클라이언트가 위임 객체의 또 다른 기능을 사용하고 싶을 때마다 서버에 위임 메서드를 추가  
→ 단순히 전달만 하는 위임 메서드들이 성가셔지고 서버 클래스는 그저 중개자 역할로 전략  
→ 차라리 클라이언트가 위임 객체를 직접 호출하는 게 낫다.

**절차**

1. 위임 객체를 얻는 게터 생성
2. 위임 메서드를 호출하는 클라이언트가 모두 이 게터를 거치도록 수정. 하나씩 바꿀 때마다 테스트
3. 모두 수정했다면, 위임 메서드 삭제

**예시**

```javascript
// 클라이언트
manager = aPerson.manager;

// Person 클래스
get manager() { return this._department.manager }

// Department 클래스
get manager() { return this._manager }
```

① 위임 객체(부서)를 얻는 게터 생성  
② 클라이언트가 부서 객체를 직접 사용하도록 수정

```javascript
// Person 클래스
get department() { return this._department }

// 클라이언트
manager = aPerson.department.manager;
```

③ 클라이언트를 모두 고쳤다면 person의 manger() 메서드 삭제  
→ Person에 단순한 위임 메서드가 남지 않을 때까지 해당 작업 반복

---

### 7.9 알고리즘 교체하기

<br />

```javascript
function foundPerson(people) {
  for (let i = 0; i < people.length; i++) {
    if (people[i] === 'Don') {
      return 'Don';
    }
    if (people[i] === 'John') {
      return 'John';
    }
    if (people[i] === 'Kent') {
      return 'Kent';
    }
  }

  return '';
}
```

↓

```javascript
function foundPerson(people) {
  const candidates = ['Don', 'John', 'Kent'];
  return people.find((p) => candidates.includes(p)) || '';
}
```

<br />

**배경**

문제를 더 확실하게 이해하고 훨씬 쉽게 해결하는 방법을 발견했을 때, 알고리즘 전체를 걷어내고 훨씬 간결한 알고리즘으로 바꿔야 할 때가 있다.

알고리즘을 살짝 다르게 동작하도록 바꾸고 싶을 때도 이 변화를 더 쉽게 가할 수 있는 알고리즘을 통째로 바꾼 후에 처리하면 편할 수 있다.

거대하고 복잡한 알고리즘을 교체하기란 상당히 어려우니 알고리즘을 간소화하는 작업부터 해야 교체가 쉬워진다.

**절차**

1. 교체할 코드를 함수 하나에 모은다.
2. 이 함수만을 이용해 동작을 검증하는 테스트를 마련
3. 대체할 알고리즘을 준비
4. 정적 검사 수행
5. 기존 알고리즘과 새 알고리즘의 결과를 비교하는 테스트 수행. 두 결과가 같다면 리팩터링이 끝.  
   그렇지 않다면 기존 알고리즘을 참고해서 새 알고리즘을 테스트하고 디버깅
