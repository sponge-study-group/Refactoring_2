### 11.1 질의 함수와 변경 함수 분리하기
겉보기 부수효과가 있는 함수와 부수효과가 없는 함수를 명확히 구분하라 (CQS)
```javascript
//전
function getTotalOutstandingAndSendBill() {
 const result = customer.invoices.reduce( (total, each) => each.amount + total, 0));
 sendBill();
 return result;
}


//후
function getTotalOutstandingAndSendBill() {
 const result = customer.invoices.reduce( (total, each) => each.amount + total, 0));
}

function sendBill() {
 emailGateway.send(formatBill(customer));
}
```
활용될 수 있는 악취들
가변 데이터
뒤엉킨 변경

### 11.2 함수 매개변수화하기
매개변수를 추가해 함수의 유용성을 키울수 있다.
```javascript
//전
function tenPercentRaise(aPerson) {
 aPerson.salary = aPerson.salary.multiply(1.1);
}

function fivePercentRaise(aPerson) {
 aPerson.salary = aPerson.salary.multiply(1.05);
}

//후
function raise(aPerson, factor) {
 aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

활용될 수 있는 악취들
중복제거

### 11.3 플래그 인수 제거하기
플래그 인수를 사용하면 함수들의 기능 차이가 잘 드러나지 않는다.
플래그 인수를 두개 이상 사용한다면 플래그 인수를 사용하는 합당한 근거가 될 수 있다. 왜냐하면 플래그 인수 없이는 조합 가능한 수만큼 함수를 만들어야 하기 때문이다. 하지만 이런 상황에서는 하나의 함수가 너무 많은 일을 처리하고 있다는 신호인지 확인해야 한다.
```javascript
//전
function setDimension(name, value) {
	if (name === "height) {
    	this._height = value;
        return;
    }
    
    if (name === "width") {
    	this._width = value;
        return;
    }
}

//후
function setHeight(value) {
	this._height =value;
}

function setWidth(value) {
	this._width = value;
}
```
활용될 수 있는 악취들
긴 매개변수 목록

### 11.4 객체 통째로 넘기기
레코드를 통째로 넘기면 변화에 대응하기 쉽다.
매개변수 목록이 짧아져서 함수 사용법을 이해하기 쉬워진다.
레코드의 담긴 데이터중 일부를 받는 함수가 여러 개라면 레코드 내부의 특정 데이터를 사용하는 로직이 중복될수 있다.
함수가 레코드에 의존하길 원치 않을 경우 사용하지 말라.
```javascript
//전
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))

//후
if (aPlan.withinRange(aRoom.daysTempRange)))
```
활용될 수 있는 악취들
긴 매개변수 목록
긴 함수
데이터 뭉치

### 11.5 매개변수를 질의 함수로 바꾸기
매개변수를 제거함으로써 호출하는 쪽을 간소하게 만든다. (값을 결정하는 주체를 피호출 함수로 옮긴다.)
피호출 함수가 스스로 쉽게 결정할수 있는 값을 매개변수로 건네는 것도 중복이다.
매개변수를 제거하면 피호출 함수에 원치 않는 의존성이 생길때는 주의하라.
함수가 참조 투명하게 유지하라. 즉 함수에 똑같은 값을 건네 호출하면 항상 똑같이 동작해야 한다. 따라서 매개변수를 제거하는 대신 가변 전역 변수를 이용하면 안된다.
```javascript
//전
availableVacation (anEmployee, anEmployee.grade);

function availableVacation (anEmployee, grade) {
	// 연휴계산..
}

//후
availableVacation(anEmployee)

function availableVacation(anEmployee) {
	const grade = anEmployee.grade
	// 연휴계산..
}
```

### 11.6 질의 함수를 매개변수로 바꾸기
함수안에 두기엔 거북한 참조(전역 변수)를 해결하기 위해 해당 참조를 매개변수로 바꿔 해결할 수 있다.
의존성을 끊어내기 위해 모든 것을 매개변수로 바꿔 길고 반복적인 매개변수 목록을 만드는것과 간단하지만 많은 결합이 있는 함수를 만든는것 사이에 적절한 균형을 찾아야 한다. 한 시점에 내린 결정이 영원히 옳다고 할수는 없기 때문에 프로그램에 대한 이해가 깊어졌을때 더 나은 쪽으로 개선하기 쉽게 설계해야 한다.
장점은 참조 투명성을 얻을 수 있고 단점은 호출자가 복잡해진다.
```javascript
//전
targetTemperature(aPlan)

function targetTemperature(aPlan) {
	currentTemperature = thermostat.currentTemperature;
    //생략
}

//후
targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
	//생략
}
```

### 11.7 세터 제거하기
객체 생성후 값이 바뀌면 안 된다는 뜻을 분명히 하려면 세터를 제거하라

```javascript
//전
class Person {
	get name() {...}
    set name(aString) {...}
}

//후
class Person {
	get name() {...}
}
```

활용될 수 있는 악취들
가변 데이터
데이터 클래스

### 11.8 생성자를 팩터리 함수로 바꾸기
객체 생성시 서브 클래스, 프록시 객체를 반환하고 싶을때 사용.
보통 생성자보다 더 적절한 이름을 사용하고 싶을때 사용.
일반 함수가 기대되는 곳에 사용할 수 없다. (java의 콜백으로 생성자를 넘겨주기)

```javascript
//전
leadEngineer = new Employee(document.leadEngineer, 'E');

//후
leadEngineer = createEngineer(document.leadEngineer);
```

활용될 수 있는 악취들
없음

### 11.9 함수를 명령으로 바꾸기
함수를 함수만을 위한 객체안으로 캡슐화 하면(명령 객체)되돌리기 같은 보조연산을 제공할 수 있고 수명주기를 더 정밀하게 제어 가능하다.
함수를 명령객체로 바꾸면 복잡한 함수를 응집도 있고 이해하고 수정하기 쉽게 만들 수 있다. 또한 서브함수들을 테스트와 디버깅에 활용할 수 있다.(중첩함수가 대안이 될 수 있다.)
일급함수를 지원하지 않는 프로그래밍 언어를 사용할 때 일급 함수의 기능 대부분을 흉내 낼 수 있다.

```javascript
//전
function score(candidate, medicalExam, scoringGuide) {
	let result = 0;
    let healthLevel = 0;
    // 긴 코드 생략
}

//후
class Scorer {
	constructor(candidate, medicalExam, scoringGuide) {
    	this._cadidate = candidate;
        this._medicalExam = medicalExam;
        this._scoringGuide = scoringGuide;
    }
    
    excute() {
    	this._result = 0;
        this._healthLevel = 0;
        //긴 코드 생략
    }
}
```

활용될 수 있는 악취들
긴 함수

### 11.10 명령을 함수로 바꾸기
명령 객체는 복잡한 연산을 다룰때 유용하다. 왜냐하면 큰 연산 하나를 여러 개의 작은 메서드로 쪼개고 필드를 이용해 쪼개진 메서드 끼리 정보를 공유하도록 할 수 있기때문이다. 하지만 로직이 크게 복잡하지 않으면 명령 객체는 장점보다 단점이 크다.

```javascript
//전
class ChargeCalculator {
	constructor(customer, usage) {
    	this._customer = customer;
        this._usage = usage;
    }
    
    execute() {
    	return this._customer.rate * this._usage;
    }
}

//후
function charge(customer, usage) {
	return customer.rate * usage;
}
```

활용될 수 있는 악취들
성의없는 요소

### 11.11 수정된 값 반환하기
데이터가 어떻게 수정되는지를 명확하게 알려 줄 수 있다. (부수효과 예방 가능)
변수를 갱신하는 함수라면 수정된 값을 반환하도록 하고 호출자가 그 값을 변수에 담아두도록 하라.
값 하나를 계산한다는 목적이 있는 함수들에 효과적이고, 반대로 값 여러 개를 갱신하는 함수에는 효과적이지 않다.

```javascript
//전
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
	for (let i = 1; i < points.length; i++) {
    	const verticalChange = points[i].elevation - points[i-1].elevation;
        totalAscent += (verticalChange > 0) ? verticalChange : 0;
    }
}

//후
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

활용될 수 있는 악취들
가변 데이터(부수효과)

### 11.12 오류 코드를 예외로 바꾸기
예외는 예상 밖의 동작일 때만 쓰여야 한다. 예외를 던지는 코드를 프로그램 종료 코드로 바꾼다고 프로그램이 정상 동작하지 않는다면 예외를 사용하지 말자. 이때는 오류를 검출하여 정상 흐름으로 되돌려야 한다.
예외를 사용하면 오류 코드를 일일이 검사하거나 오류를 식별해 콜스택 위로 던지는 일을 신경 쓰지 않아도 된다.
오류는 실패 체인 확인이 불가 하다.
오류는 Api 호출자만 검사가 가능하나 exception은 상단 어디서든 잡아서 처리가 가능하다.

```javascript
//전
if (data)
 return new ShippinRules(data);
 else
 return -23
 
//후
if (data)
	return new ShippingRules(data);
  else
   throw new OrderProcessingError(-23);
```
   
활용될 수 있는 악취들
없음

### 11.13 예외를 사전확인으로 바꾸기
예외는 예외적인 동작할 때만 쓰여야 한다. 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 사전에 검사해야 한다.

```javascript
//전
double getValueForPeriod (int periodNumber) {
	try {
    	return values[periodNumber]
    } catch (ArrayIndexOutOfBoundsException e) {
    	return 0;
    }

}
//후
double getValueForPeriod (int periodNumber) {
	return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```

활용될 수 있는 악취들
성의없는 요소


## 🧩Reference [@wnwl1216/리팩터링-11장-API-리팩터링](https://velog.io/@wnwl1216/%EB%A6%AC%ED%8C%A9%ED%84%B0%EB%A7%81-11%EC%9E%A5-API-%EB%A6%AC%ED%8C%A9%ED%84%B0%EB%A7%81)
