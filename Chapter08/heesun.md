# Chapter08 기능이동 

## 8.1 함수 옮기기 
Move Function

```javascript
class Account {
    get overdraftCharge(){ ... }
}
⬇️
class Account {
    et overdraftCharge(){ ... }
}
```

## 8.2 필드 옮기기
Move Field

```javascript
class Customer {
    get plan(){ return this._plan; }
    get discountRate(){ return this._discountRate; }
}
⬇️
class Customer {
    get plan(){ this._plan; }
    get discountRate(){ return this.plan.discountRate; }
}
```

## 8.3 문장을 함수로 옮기기 
Move Statements into Function

```javascript
result.push(`<p>제목: ${person.photo.title}</p>`);
result.concat(photoData(person.photo));

function photoData(aPhoto){
    return [
        `<p>위치: ${aPhoto.location}</p>`,
        `<p>날짜: ${aPhoto.date.toDateString()}</p>`,
    ];
}
⬇️
result.concat(photoData(person.photo));

function photoData(aPhoto){
    return [
        `<p>제목: ${aPhoto.title}</p>`,
        `<p>위치: ${aPhoto.location}</p>`,
        `<p>날짜: ${aPhoto.date.toDateString()}</p>`,
    ];
}
```

## 8.4 문장을 호출한 곳으로 옮기기
Move Statements to Callers


```javascript
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo){
    outStream.write(`<p>제목: ${photo.title}</p>`);
    outStream.write(`<p>위치: ${photo.location}</p>\n`);
}
⬇️
emitPhotoData(outStream, person.photo);
outStream.write(`<p>위치: ${person.photo.location}</p>\n`);

function emitPhotoData(outStream, photo){
    outStream.write(`<p>제목: ${photo.title}</p>\n`);
}
```

## 8.5 인라인 코드를 함수 호출로 바꾸기
Replace Inline Code with Function Call

>인라인 코드는 비슷한 동작을 함수 하나로 해결 할 수 있기 때문에 함수 호출로 바꿔준다.
라이브러리가 제공하는 함수로 대체할 수 있다면(ex. includes) 함수 본문을 작성할 필요조차 없기 때문에 좋다.

```javascript
let appliesToMass = false;
for(const s of states){
    if(s==="MA") appliesToMass = true;
}
⬇️
appliesToMass = states.includes("MA");
```

## 8.6 문장 슬라이드하기
Slide Statements

>관련된 코드는 가까이에 모아둔다.

```javascript
const pricingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;
⬇️
const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```

## 8.7 반복문 쪼개기
Split Loop

>반복문 하나에서 두 가지 일을 하게 될 경우 수정할 때마다 두 가지 일 모두를
이해하고 진행해야 하므로 분리하는 것이 좋다.

```javascript
let averageAge = 0;
let totalSalary = 0;
for (const p of people){
    averageAge += p.age;
    totalSalary += p.salary;
}
averageAge = averageAge / people.length;
⬇️
let totalSalary = 0;
for (const p of people){
    totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people){
    averageAge += p.age;
}
averageAge = averageAge / people.length;
```

## 8.8 반복문을 파이프라인으로 바꾸기
Replace Loop with Pipeline

>컬렉션을 순회할 때 반복문을 사용하라고 배운다. 하지만 컬렉션 파이프라인을 이요하면 처리 과정을
일련의 연산으로 표현할 수 있다. 대표적인 연산은 map과 filter다. 
map은 함수를 사용해 입력 컬렉션의 각 원소를 변환하고, filter는 또 다른 함수를 사용해 입력 컬렉션을
필터링해 부분집합을 만든다. 이 부분집합은 파이프라인의 다음 단계를 위한 컬렉션으로 쓰인다.

* Collection?
사전적 의미 : 요소를 수집해서 저장하는 것.

>자바에서 컬렉션 프레임워크란,
다수의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 클래스의 집합을 의미한다.
즉, 데이터를 저장하는 자료구조와 데이터를 처리하는 알고리즘을 구조화하여 클래스로 구현해놓은 것.

* 자바에서의 자료구조 유형은 다음과 같다.
- 순서가 있는 목록인 List형
- 순서가 중요하지 않은 목록인 Set형
- 먼저 들어온 것이 먼저 나가는 Queue형
- KEY-VALUE의 형태로 저장되는 Map형


>자바 컬렉션은 객체를 수집해서 저장하는 역할을 한다.
자바 컬렉션 프레임워크는 몇 가지 인터페이스를 통해 다양한 컬렉션을 이용할 수 있도록 한다.
그중 크게 3가지 타입으로 나누어 핵심이 되는 주요 인터페이스를 정의하였다.

* List 인터페이스
* Set 인터페이스
* Map 인터페이스


```javascript
const names = [];
for (const i of input){
    if(i.job === "programer"){
        names.push(i.name);
    }
}
⬇️
const names = input
.filter(i => i.job === "programer")
.map(i => i.name);
```

## 8.9 죽은 코드 제거하기
Remove Dead Code

>죽은 코드 때문에 운 나쁜 프로그래머는 이 코드의 동작을 이해하기 위해, 그리고 코드를 수정했는데도
기대한 결과가 나오지 않는 이유를 파악하기 위해 시간을 허비하게 된다.
다시 필요해지지 않을까 걱정된다면 버전관리 시스템이 있으며 
이전 코드를 다시 불러오는 경우는 아주 드물다...

```javascript
if(false){
    doSomethingThatUsedToMatter();
}
```

















