# 07 캡슐화

* 레코드 캡슐화하기
* 컬렉션 캡슐화하기

## 7.1 레코드 캡슐화하기
### Encapsulate Record

>

* 1판에서의 이름 : 레코드를 데이터 클래스로 전환

```javascript
const organization = {name: "애크미 구스베리", country: "GB"};

class Organization {
    constructor(data){
        this._name = data.name;
        this._country = data.country;
    }

    get name() { return this._name; }
    set name(arg) { this._name = arg; }
    get country() { return this._name; }
    set country(arg) { this._country = arg; }
}
```

## 7.2 컬렉션 캡슐화하기
### Encapsulation Collection

```javascript
class Person {
    get courses() { return this._courses; }
    set courses(aList) { this._courses = aList; }
}
⬇️
class Person {
    get courses() { return this._courses; }
    addCourse(aCourse) {}
    removeCourse(aCourse) {}
}
```

## 7.3 기본형을 객체로 바꾸기
### Replace Primitive with Object

* 1판에서의 이름
  - 데이터 값을 객체로 전환
  - 분류 부호를 클래스로 전환

```javascript
orders.filter(o => "hight" === o.priority 
|| "rush" === o.priority);
⬇️
orders.filter(o => o.priority.higherThan(new Priority("normal")));
```
* filter는 조건에 일치하는 배열의 value를 리턴한다.

```javascript
class Priority {
    constructor(value) {
        if (value instanceof Priority) return value;
        if (Priority.legalValues().includes(value))
            this._value = value;
        else 
            throw new Error(`<${value}>는 유효하지 않은 우선수위입니다.`);
    }
    toString() { return this._value; }
    get _index() { return Priority.legalValues().findIndex(s => s === this._value); }
    static legalValues() { return ['low', 'normal', 'high', 'rush']; }
    equals(other) { return this._index === other._index; }
    higherThan(other) { return this._index > other._index; }
    lowerThan(other) { return this._index < other._index; }
}

highPriorityCount = orders.filter(o => o.priority.higherThan(new Priority("normal"))).length;
```

* instanceof 연산자는 생성자의 prototype 속성이 객체의 프로토타입 체인 어딘가 존재하는지 판별합니다.


## 7.4 임시 변수를 질의 함수로 바꾸기
### Replace Temp with Query

>다시 참조할 목적으로 사용하는 임시 변수는 함수로 만드는 것을 고려해본다.

```javascript
const basePrice = this._quantity * this._itemPrice;
if (basePrice > 1000)
    return basePrice * 0.95;
else
    return basePrice * 0.98;
⬇️
get basePrice() { this._quantity * this._itemPrice; }
...
if (this.basePrice > 1000)
    return this.basePrice * 0.95;
else
    return this.basePrice * 0.98;
```

## 7.5 클래스 추출하기
### Extract Class

```javascript
class Person {
    get officeAreaCode() { return this._officeAreaCode; }
    get officeNumber() { return this._officeNumber; }
}
⬇️
class Person {
    get officeAreaCode() { return this._telephoneNumber.areaCode; }
    get officeNumber() { return this._telephoneNumber.number; }
}
class TelephoneNumber {
    get areaCode() { return this._areaCode; }
    get number() { return this._number; }
}
```

## 7.6 클래스 인라인하기
### Inline Class

>역할을 다 한 클래스는 인라인한다.

* 반대 리팩터링 : 클래스 추출하기

```javascript
class Person {
    get officeAreaCode() { return this._telephoneNumber.areaCode; }
    get officeNumber() { return this._telephoneNumber.number; }
}
class TelephoneNumber {
    get areaCode() { return this._areaCode; }
    get number() { return this._number; }
}
⬇️
class Person {
    get officeAreaCode() { return this._officeAreaCode; }
    get officeNumber() { return this._officeNumber; }
}
```

## 7.7 위임 숨기기
### Hide Delegate

>위임 객체가 수정되더라도 서버 코드만 고치면 되고 클라이언트는 영향을 받지 않도록 한다.

```javascript
manager = aPerson.department.manager;
⬇️
manager = aPerson.manager;

class Person {
    get manager() { return this.department.manager; }
}
```

## 7.8 중개자 제거하기
### Remove Middle Man

>캡슐화가 단순히 전달만 하는 역할로 전락했을 때, 위임 객체를 직접 호출하는 방식을 고려해본다.

```javascript
manager = aPerson.manager;

class Person {
    get manager() { return this.department.manager; }
}
⬇️
manager = aPerson.department.manager;
```


## 7.9 알고리즘 교체하기
### Substitute Algorithm

>복잡한 기존 코드를 가명한 방식으로 바꾼다.

```javascript
function foundPerson(people) {
    for(let i = 0; i < people.length; i++){
        if (people[i] === "Don") {
            return "Don";
        }
        if (people[i] === "John") {
            return "John";
        }
        if (people[i] === "Kent") {
            return "Kent";
        }
    }
    return "";
}
⬇️
function foundPerson(people) {
    const candidates = ["Don", "John", "Kent"];
    return people.find(p => candidates.includes(p)) || '';
}
```

* find는 배열의 value를 리턴하고 includes는 boolean을 리턴한다.



