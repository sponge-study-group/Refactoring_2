## 8. 기능 이동

### 8.1 함수 옮기기

<br />

```javascript
class Account {
  get overdraftCharge() {...}
}
```

↓

```javascript
class AccountType {
  get overdraftCharge() {...}
}
```

<br />

**배경**  
좋은 소프트웨어 설계의 핵심 → 모듈화가 얼마나 잘 되어 있느냐(모듈성)

모듈성을 높이려면  
→ 서로 연관된 요소들을 함께 묶고, 요소 사이의 연결 관계를 쉽게 찾고 이해할 수 있도록 ○

`어떤 함수가 자신이 속한 모듈 A의 요소들보다 다른 모듈 B의 요소들을 더 많이 참조한다면 모듈 B로 옮겨줘야 마땅하다.`

**절차**  
① 선택한 함수가 현재 컨텍스트에서 사용 중인 모든 프로그램 요소 중에서 함께 옮겨야 할 게 있는지 고민  
→ 함께 옮길 함수가 있다면, 그 함수부터 먼저 옮기는 게 낫다.  
→ 함수가 여러 개라면 다른 곳에 미치는 영향이 적은 함수부터  
② 선택한 함수가 다형 메서드인지 확인  
③ 선택한 함수를 타깃 컨텍스트로 복사  
④ 정적 분석 수행  
⑤ 소스 컨텍스트에서 타깃 함수를 참조할 방법을 찾아 반영  
⑥ 소스 함수를 타깃 함수의 위임 함수가 되도록 수정  
⑦ 테스트  
⑧ 소스 함수를 인라인(선택)

**예시:중첩함수를 최상위로 옮기기**

```javascript
function trackSummary(points) {
  const totalTime = calculateTime();
  const totalDistance = calculateDistance();
  const pace = totalTime / 60 / totalDistance;

  return { time: totalTime, distance: totalDistance, pace: pace };

  function calculateDistance() {
    // 총 거리 계산
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      result += distance(points[i - 1], points[i]);
    }
    return result;
  }

  function distance(p1, p2) { ... }
  function radians(degrees) { ... }
  function calculateTime() { ... }
}
```

③ 함수를 최상위로 복사  
distance()은 radians()만 사용하며, radians()는 현재 컨텍스트에 있는 어떤 것도 사용 ×  
→ 두 함수를 함께 옮기는 것이 낫다.

```javascript
function trackSummary(points) {
  const totalTime = calculateTime();
  const totalDistance = calculateDistance();
  const pace = totalTime / 60 / totalDistance;

  return { time: totalTime, distance: totalDistance, pace: pace };

  // 총 거리 계산
  function calculateDistance() {
    let result = 0;

    for (let i = 1; i < points.length; i++) {
      result += distance(points[i - 1], points[i]);
    }

    function distance(p1, p2) { ... } // 두 지점의 거리 계산
    function radians(degrees) { ... } // 라디안 값으로 계산

    return result;
  }

  function calculateTime() { ... }  // 총 시간 계산
}

// 최상위로 복사하면서 새로운 이름을 임시로 붙여줌
function top_calculateDistance(points) {
  let result = 0;

  for (let i = 1; i < points.length; i++) {
    result += distance(points[i - 1], points[i]);
  }

  // 같은 내용을 해당 함수에도 복사
  function distance(p1, p2) { ... } // 두 지점의 거리 계산
  function radians(degrees) { ... } // 라디안 값으로 계산

  return result;
}
```

④ 정적 분석 수행  
⑥ calculateDistance()의 본문에서 top_calculateDistance()을 호출하도록 수정  
```javascript
function trackSummary(points) {
  const totalTime = calculateTime();
  const totalDistance = calculateDistance();
  const pace = totalTime / 60 / totalDistance;

  return { time: totalTime, distance: totalDistance, pace: pace };

  // 총 거리 계산
  function calculateDistance() {
    return top_calculateDistance(points);
  }

  function calculateTime() { ... }  // 총 시간 계산
}

// 최상위로 복사하면서 새로운 이름을 임시로 붙여줌
function top_calculateDistance(points) {
  let result = 0;

  for (let i = 1; i < points.length; i++) {
    result += distance(points[i - 1], points[i]);
  }

  // 같은 내용을 해당 함수에도 복사
  function distance(p1, p2) { ... } // 두 지점의 거리 계산
  function radians(degrees) { ... } // 라디안 값으로 계산

  return result;
}
```

⑦ 테스트를 통해 옮겨진 함수가 잘 정책했는지 확인  
→ 소스 함수는 호출자가 많지 않은 지역화된 함수이므로, 소스 함수는 제거
```javascript
function trackSummary(points) {
  const totalTime = calculateTime();
  const pace = totalTime / 60 / totalDistance;

  return { time: totalTime, distance: totalDistance(points), pace: pace };

  function calculateTime() { ... }  // 총 시간 계산
}

// 변수 인라인하기
function totalDistance(points) {
  let result = 0;

  for (let i = 1; i < points.length; i++) {
    result += distance(points[i - 1], points[i]);
  }

  return result;
}

function distance(p1, p2) { ... } // 두 지점의 거리 계산
function radians(degrees) { ... } // 라디안 값으로 계산
```


---

### 8.2 필드 옮기기

<br />

```javascript
class Customer {
  get plan() {
    return this._plan;
  }
  get discounRate() {
    return this._discountRate;
  }
}
```

↓

```javascript
class Customer {
  get plan() { return this._plan; }
  get discounRate() { return this.plan.discountRate; }
```

<br />

**배경**  
주어진 문제에 적합한 데이터 구조를 활용  
→ 동작 코드는 단순하고 직관적으로 짜여진다.

데이터 구조가 적절히 않음을 깨닫게 되면 곧바로 수정  
→ 고치지 않으면 나중에 작성하게 될 코드를 더욱 복잡하게 만들어버린다.

**절차**  
① 소스 필드가 캡슐화되어 있지 않다면 캡슐화  
② 테스트  
③ 타깃 객체에 필드를 생성  
④ 정적 검사 수행  
⑤ 소스 객체에서 타깃 객체를 참조할 수 있는지 확인  
⑥ 접근자들이 타깃 필드를 사용하도록 수정  
⑦ 테스트  
⑧ 소스 필드 제거  
⑨ 테스트

**예시**

```javascript
constructor(name, discountRate) {
  this._name = name;
  this._discountRate = discountRate;
  this._contract = new CustomerContract(dateToday());
}

get discounRate() { return this._discountRate }

becomePreferred() {
  this._discountRate += 0.03;

  ...
}

applyDiscount(amount) {
  return amount.subtract(amount.multiply(this.discountRate));
}
```

discountRate 필드를 Customer에서 CustomerContract로 옮기고 싶을때.

① 필드 캡슐화  
③ CustomerContract 클래스에 필드 하나와 접근자들을 추가  
⑥ Customer의 접근자들이 새로운 필드를 사용하도록 수정  
→ 문장 슬라이드를 적용해 \_serDiscountRate() 호출을 계약 생성 뒤로 옮겨야한다.

```javascript
constructor(name, discountRate) {
  this._name = name;
  this._contract = new CustomerContract(dateToday());
  this._serDiscountRate(discountRate);  // ①
}

get discounRate() { return this._contract.discountRate } // ⑥
_serDiscountRate(aNumber) { this._contract.discountRate = aNumber }  // ①, ⑥

becomePreferred() {
  this._serDiscountRate(this._discountRate + 0.03);

  ...
}

applyDiscount(amount) {
  return amount.subtract(amount.multiply(this.discountRate));
}
```

```javascript
// ③
constructor(startDate, discountRate) {
  this._startDate = startDate;
  this._discountRate = discountRate;
}

get discounRate() { return this._discountRate }
set discounRate(arg) { this._discountRate = arg }
```

---

### 8.3 문장을 함수로 옮기기

<br />

```javascript
result.push(`<p>제목: ${person.photo.title}</p>`);
result.concat(photoData(person.photo));

function photoData(aPhoto) {
  return [`<p>위치: ${aPhoto.location}</p>`, `<p>날짜: ${aPhoto.date.toDateString()}</p>`];
}
```

↓

```javascript
result.concat(photoData(person.photo));

function photoData(aPhoto) {
  return [`<p>제목: ${aPhoto.title}</p>`, `<p>위치: ${aPhoto.location}</p>`, `<p>날짜: ${aPhoto.date.toDateString()}</p>`];
}
```

**배경**  
중복 코드 제거 → 코드를 건강하게 관리하는 가장 효과적인 방법 중 하나

문장들을 함수로 옮기려면 그 문장들이 피호출 함수의 일부라는 확신 ○  
→ 피호출 함수와 함께 호출돼야 하는 경우라면 해당 문장들과 피호출 함수를 통째로 또 하나의 함수로 추출

**절차**  
① 반복 코드를 문장 슬라이드를 적용해 함수 호출 부분 근처로 옮긴다.  
② 타깃 함수를 호출하는 곳이 한 곳뿐이면, 해당 코드를 잘라내서 피호출 함수로 복사하고 테스트  
③ 호출하는 곳이 둘 이상이면 호출자 중 하나에서 '타깃 함수 호출 부분과 옮기려는 문장들을 함께 다른 함수로 추출'  
④ 호출자가 방금 추출한 함수를 사용하도록 수정  
⑤ 모든 호출자가 변경됐으면 원래 함수를 새로운 함수 안으로 인라인한 후, 원래 함수 제거  
⑥ 새로운 함수의 이름을 원래 함수 이름으로 변경

---

### 8.4 문장을 호출한 곳으로 옮기기

<br />

```javascript
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>`);
  outStream.write(`<p>위치: ${photo.location}</p>`);
}
```

↓

```javascript
emitPhotoData(outStream, person.photo);
outStream.write(`<p>위치: ${photo.location}</p>`);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>`);
}
```

<br />

**배경**  
함수는 프로그래머가 쌓아 올리는 추상화의 기본 빌딩 블록  
→ 코드베이스의 기능 범위가 달라지면 추상화의 경계도 움직이게 된다.

**절차**  
① 호출자가 적고 피호출 함수도 단순하면, 피호출 함수의 처음이나 마지막 줄을 잘라 호출자로 복사해 넣는다.  
② 더 복잡한 상황에서는 이동을 원하지 않는 모든 문장을 함수로 추출한 다음 검색하기 쉬운 이름을 지어준다.  
③ 원래 함수 인라인  
④ 추출된 함수의 이름을 원래 이름으로 변경

---

### 8.5 인라인 코드를 함수 호출로 바꾸기

<br />

```javascript
let appliesToMass = false;

for (const s of states) {
  if (s === 'MA') appliesToMass = true;
}
```

↓

```javascript
appliesToMass = states.inclues('MA');
```

<br />

**배경**  
함수는 여러 동작을 하나로 묶어준다.  
→ 함수의 이름이 코드의 목적을 말해주기 떄문에 함수를 활용하면 코드를 이해하기가 쉬워진다.  
→ 중복을 없애는 데도 효과적

이름을 잘 지었다면 인라인 코드 대신 함수 이름을 넣어도 말이 된다.  
→ 말이 되지 않는다면 함수 이름이 적절하지 않거나, 그 함수의 목적이 인라인 코드의 목적과 다르기 때문.

**절차**  
① 인라인 코드를 함수 호출로 대체  
② 테스트

---

### 8.6 문장 슬라이드하기

<br />

```javascript
const pricingPlan = retrievePrcingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;
```

↓

```javascript
const pricingPlan = retrievePrcingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```

<br />

**배경**  
관련된 코드들이 가까이 모여 있다면 이해하기가 더 쉽다.  
→ 문장 슬라이드 리팩터링을 통해 한데 모아둔다.

관련 코드끼리 모으는 작업  
→ 다른 리팩터링의 준비 단계로 자주 실행

**절차**  
① 코드 조각을 이동할 위치를 찾는다.  
아래와 같은 간섭이 있다면 리팩터링을 포기  
→ 코드 조각에서 참조하는 요소를 선언하는 문장 앞으로 이동 ×  
→ 코드 조각을 참조하는 요소의 뒤로는 이동 ×  
→ 코드 조각에서 참조하는 요소를 수정하는 문장을 건너뛰어서 이동 ×  
→ 코드 조각이 수정하는 요소를 참조하는 요소를 건너뛰어 이동 ×  
② 코드 조각을 원래 위치에서 잘라내어 목표 위치에 붙여 넣는다.
③ 테스트

---

### 8.7 반복문 쪼개기

<br />

```javascript
let averageAge = 0;
let totalSalary = 0;

for (const p of people) {
  averageAge += p.age;
  totalSalary += p.salary;
}

averageAge = averageAge / people.length;
```

↓

```javascript
let totalSalary = 0;

for (const p of people) {
  totalSalary += p.salary;
}

let averageAge = 0;

for (const p of people) {
  averageAge += p.age;
}

averageAge = averageAge / people.length;
```

<br />

**배경**  
반복문에서 두 가지 일을 수행  
→ 반복문을 수정해야 할 때마다 두 가지 일 모두 잘 이해하고 진행해야 한다.  
→ 각각의 반복문으로 분리해두면 수정할 동작 하나만 이해하면 된다.

한 가지 값만 계산하는 반복문이라면 그 값만 곧바로 반환  
→ 여러 일을 수행하는 반복문이라면 구조체를 반환하거나 지역 변수를 활용

> 리팩터링과 최적화를 구분하자.

최적화는 코드를 깔끔히 정리한 이후에 수행.  
→ 반복문을 두 번 실행하는게 병목이라 밝혀지면 그때 다시 하나로 합치자.

**절차**  
① 반복문을 복제해 두 개로 만든다.  
② 반복문이 중복되어 생기는 부수효과를 파악해서 제거한다.  
③ 테스트  
④ 완료했으면, 각 반복문을 함수로 추출할지 고민

**예시**

```javascript
let youngest = people[0] ? people[0].age : Infinify;
let totalSalary = 0;

for (const p of people) {
  if (p.age < youngest) youngest = p.age;
  totalSalary += p.salary;
}

return `최연소: ${youngest}, 총 급여: ${totalSalary}`;
```

① 반복문 복제  
② 중복 제거

```javascript
let youngest = people[0] ? people[0].age : Infinify;
let totalSalary = 0;

for (const p of people) {
  // if (p.age < youngest) youngest = p.age ②
  totalSalary += p.salary;
}

// ①
for (const p of people) {
  if (p.age < youngest) youngest = p.age;
  // totalSalary += p.salary ②
}

return `최연소: ${youngest}, 총 급여: ${totalSalary}`;
```

④ 함수로 추출

```javascript
return `최연소: ${youngestAge()}, 총 급여: ${totalSalary()}`;

function totalSalary() {
  return people.reduce((total, p) => total + p.salary, 0);
}

function youngestAge() {
  return Math.min(...people.map((p) => p.age));
}
```

---

### 8.8 반복문을 파이프라인으로 바꾸기

<br />

```javascript
const names = [];

for (const i of input) {
  if (i.job === 'programmer') names.push(i.name);
}
```

↓

```javascript
const names = input.filter((i) => i.job === 'programmer').map((i) => i.name);
```

<br />

**배경**  
컬렉션 파이프라인을 이용하면 처리 과정을 일련의 연산으로 표현할 수 있다.  
→ 논리를 파이프라인으로 표현하면 이해하기 훨씬 쉬워진다.  
→ 객체가 파이프라인을 따라 흐르며 어떻게 처리되는지를 읽을 수 있기 때문

**절차**  
① 반복문에서 사용하는 컬렉션을 가리키는 변수를 하나 만든다.  
② 반복문의 첫 줄부터 적절한 컬렉션 파이프라인 연산으로 대체.  
→ ①에서 만든 반복문 컬렉션 변수에서 시작, 이전 연산의 결과를 기초로 연쇄적으로 수행  
③ 반복문의 모든 동작을 대체했다면 해당 반복문 삭제

**예시**

```javascript
function acquireData(input) {
  const lines = input.split('\n');
  let firstLine = true;
  const result = [];

  for (const line of lines) {
    if (firstLine) {
      firstLine = false;
      continue;
    }

    if (line.trim() === '') continue;

    const record = line.split(',');

    if (record[1].trim() === 'India') {
      result.push({ city: record[0].trim(), phone: record[2].trim() });
    }
  }

  return result;
}
```

① 반복문에서 사용하는 컬렉션을 가리키는 별도 변수 생성(루프 변수)  
② 코드의 반복문에서 첫 if문은 CSV 데이터의 첫 줄을 건너뛰는 작업  
→ slice() 연산을 루프 변수에서 수행하고 반복문 안의 if문은 제거  
→ 다음 작업은 빈 줄 지우기로 filter() 연산으로 대체  
→ map() 연산을 통해 여러 줄짜리 CSV 데이터를 문자열로 변환
```javascript
function acquireData(input) {
  const lines = input.split('\n');
  // let firstLine = true;
  const result = [];

  const loopItems = lines
                      .slice(1)
                      .filter(line => line.trim() !== "")
                      .map(line => line.split(","))
                      ;

  for (const line of loopItems) {
    /*
    if (firstLine) {
      firstLine = false;
      continue;
    }
    */

    // if (line.trim() === '') continue;

    // const record = line.split(',');
    const record = line;

    if (record[1].trim() === 'India') {
      result.push({ city: record[0].trim(), phone: record[2].trim() });
    }
  }

  return result;
}
```

→ filter() 연산을 통해 인도에 위치한 사무실 레코드를 뽑아낸다.  
→ map()을 통해 결과 레코드 생성
```javascript
function acquireData(input) {
  const lines = input.split('\n');
  const result = [];

  const loopItems = lines
                      .slice(1)
                      .filter(line => line.trim() !== "")
                      .map(line => line.split(","))
                      .filter(record => record[1].trim() === 'India')
                      .map(record => ({ city: record[0].trim(), phone: record[2].trim() }))
                      ;

  for (const line of loopItems) {
    const record = line;

    /*
    if (record[1].trim() === 'India') {
      result.push({ city: record[0].trim(), phone: record[2].trim() });
    }
    */

    result.push(line);
  }

  return result;
}
```

③ 파이프라인의 결과를 누적 변수에 대입해주면 해당 코드 제거 가능  
```javascript
function acquireData(input) {
  const lines = input.split('\n');
  const result = lines
                  .slice(1)
                  .filter(line => line.trim() !== "")
                  .map(line => line.split(","))
                  .filter(record => record[1].trim() === 'India')
                  .map(record => ({ city: record[0].trim(), phone: record[2].trim() }))
                  ;

  return result;
}
```

---

### 8.9 죽은 코드 제거하기

<br />

```javascript
if (false) {
  doSomethingThatUsedToMatter();
}
```

↓

```javascript

```

<br />

**배경**  
쓰이지 않는 코드가 몇 줄 있다고 해서 시스템이 느려지는 것도 ×, 메모리를 많이 잡아먹지도 ×  
→ 사용되지 않는 코드가 있다면 해당 소프트웨어의 동작을 이해하는 데는 커다란 걸림돌이 될 수도 있다.

> 코드가 더 이상 사용되지 않게 됐다면 지워야 한다.

**절차**  
① 죽은 코드를 외부에서 참조할 수 있는 경우라면, 호출하는 곳이 있는지 확인  
② 없다면 죽은 코드 제거  
③ 테스트

---
