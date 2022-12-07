## 4. 테스트 구축하기

> 리팩터링을 하지 않더라도 좋은 테스트를 작성하는 일은 개발 효율을 높여준다.

### 4.1 자가 테스트 코드의 가치

<br />

"클래스마다 테스트 코드를 갖춰야 한다."

프로그램이 제대로 된 값을 출력했는지를 컴퓨터를 통해 확인  
 → 테스트가 성공했는지 확인하려면, 의도한 결과와 테스트 결과가 같은지만 비교

> 모든 테스트를 완전히 자동화하고 그 결과까지 스스로 검사하게 만들자.

컴파일할 때마다 테스트도 함께 진행  
 → 생산성 ↑, 디버깅 시간 ↓  
 → 테스트를 자주 수행했기 때문에 버그가 발생한 지점에 대한 파악이 쉽다.  
 → 몇 분이면 버그 해결 가능  
 → 테스트를 자주 수행하는 습관도 버그를 찾는 강력한 도구

테스트를 작성하기 가장 좋은 시점은 프로그래밍을 시작하기 전  
 → 기능을 추가해야 할 때 테스트부터 작성  
 → 테스트를 작성하다 보면 `원하는 기능을 추가하기 위해 무엇이 필요한지 고민`  
 → 테스트를 모두 통과한 시점이 바로 코드를 완성한 시점

**테스트 주도 개발(TDD)**  
 → 테스트를 작성하고 이 테스트를 통과하게끔 코드를 작성, 결과 코드를 깔끔하게 리팩터링 하는 과정을 짧은 주기로 반복

> 리팩터링에는 테스트가 필요하다. 그러니 리팩터링하고 싶다면 테스트를 반드시 작성해야 한다.

---

### 4.2 테스트할 샘플 코드

<br />

코드는 항상 성격에 따라 분리하는 것이 좋다.

비즈니스 로직 코드는 두 개로 구성  
→ 생산자를 표현하는 Producer, 지역 전체를 표현하는 Province

```javascript
constructor(doc) {
  this._name = doc.name;
  this._producers = [];
  this._totalProduction = 0;
  this._demand = doc.demand;
  this._price = doc.price;

  doc.producers.forEach(d => this.addProducer(new Producer(this, d)));
}

addProducer(arg) {
  this._producers.push(arg);
  this._totalProduction += arg.production;
}

get name() {return this._name;}
get producers() {return this._producers.slice();}
get totalProduction() {return this._totalProduction;}
set totalProduction(arg) {this._totalProduction = arg;}
get demand() {return this._demand;}
set demand(arg) {this._demand = parseInt(arg);}
get price() {return this._price;}
set price(arg) {this._price = parseInt(arg);}
get shortfall() { return this._demand - this.totalProduction;}  // 생산 부족분 계산

// 수익 계산
get profit() {
  return this.demandValue - this.demandCost;
}
get demandValue() {
  return this.satisfiedDemand * this.price;
}
get satisfiedDemand() {
  return Math.min(this._demand, this.totalProduction);
}
get demandCost() {
  let remainingDemand = this.demand;
  let result = 0;
  this.producers
  .sort((a,b) => a.cost - b.cost)
  .forEach(p => {
    const contribution = Math.min(remainingDemand, p.production);
    remainingDemand -= contribution;
    result += contribution * p.cost;
  });

  return result;
}
```

Province 클래스

```javascript
function sampleProvinceData() {
  return {
    name: 'Asia',
    producers: [
      { name: 'Byzantium', cost: 10, production: 9 },
      { name: 'Attalia', cost: 12, production: 10 },
      { name: 'Sinope', cost: 10, production: 6 },
    ],
    demand: 30,
    price: 20,
  };
}
```

최상위

```java
constructor(aProvince, data) {
  this._province = aProvince;
  this._cost = data.cost;
  this._name = data.name;
  this._production = data.production || 0;
}

get name() {return this._name;}
get cost() {return this._cost;}
set cost(arg) {this._cost = parseInt(arg);}
get production() {return this._production;}
set production(amountStr) { const amount = parseInt(amountStr);

const newProduction = Number.isNaN(amount) ? 0 : amount;

this._province.totalProduction += newProduction - this._production;
this._production = newProduction;
}
```

Producer 클래스

### 4.3 첫 번째 테스트

<br />

테스트 프레임워크인 **모카** 사용

```javascript
describe('province', function () {
  it('shortfall', function () {
    const asia = new Province(sampleProvinceData()); // 픽스처 설정
    assert.equal(asia.shortfall, 5); // 검증
  });
});
```

1. 테스트에 필요한 데이터와 객체를 뜻하는 `픽스처(fixture)를 설정`  
   → 샘플 지역 정보로부터 생성한 Province 객체로 설정
2. 이 픽스처의 `속성을 검증`  
   → 주어진 초깃값에 기초하여 생산 부족분을 정확히 계산했는지 확인

```javascript
get shortfall() {
  return this._demand - this.totalProduction * 2; // 오류 주입
}
```

일시적으로 코드에 오류를 주입하여 테스트를 다시 확인

모카 프레임워크를 이용하면 문제가 생겼을 때 즉시 알 수 있다.  
→ 어느 테스트가 실패했는지 짚어주고, 실패 원인을 추론해볼 수 있는 단서 제공

> 자주 테스트하라. 작성 중인 코드는 최소한 몇 분 간격으로 테스트하고, 적어도 하루에 한 번은 전체 테스트를 돌려보자.

간결한 피드백은 자가 테스트에서 매우 중요하다.

**차이 라이브러리**

```javascript
describe('province', function () {
  it('shortfall', function () {
    const asia = new Province(sampleProvinceData());
    // assert.equal(asia.shortfall, 5);
    expect(asia.shortfall).equal(5);
  });
});
```

---

### 4.4 테스트 추가하기

<br />

클래스가 하는 일을 모두 살펴보고 각각의 기능에서 오류가 생길 수 있는 조건을 하나씩 테스트

테스트를 너무 많이 만들다 보면 오히려 필요한 테스트를 놓치기 쉽다.  
→ 가장 걱정되는 영역을 집중적으로 테스트  
→ 테스트에 쏟는 노력의 효과를 극대화

> 완벽하게 만드느라 테스트를 수행하지 못하느니, 불완전한 테스트라도 작성해 실행하는 게 낫다.

```javascript
describe('province', function () {
  let asia;

  // 각각의 테스트가 실행되기 전에 asia 초기화
  beforeEach(function () {
    asia = new Province(sampleProvinceData()); // 테스트가 자신만의 asia를 사용할 수 있다.
  });

  it('shortfall', function () {
    // const asia = new Province(sampleProvinceData()); → 테스트끼리 상호작용하게 하는 공유 픽스처
    expect(asia.shortfall).equal(5);
  });
  it('profit', function () {
    // const asia = new Province(sampleProvinceData());
    expect(asia.profit).equal(230);
  });
});
```

총수익이 제대로 계산되는지 검사

먼저 기대값 자리에 임의의 값을 넣고 테스트를 수행한 다음, 해당 값(230)으로 대체  
→ 테스트가 제대로 작동된다면, 총수익 계산 로직에 \* 2를 덧붙여서 잘못된 값이 나오도록 수정  
→ 일부러 주입한 오류를 테스트가 걸러낸다면, 원래 코드로 되돌린다.

`테스트끼리 상호작용하게 하는 공유 픽스처`를 생성  
→ 테스트 관련 버그 중 가장 지저분한 유형

개별 테스트를 실행할 때마다 픽스처를 새로 만들면 모든 테스트를 독립적으로 구성할 수 있다.  
→ 결과를 예측할 수 없어서 골치를 썩는 사태를 예방

Q. 매번 픽스처를 생성하느라 테스트가 느려지지 않을까?  
→ 눈에 띄게 느려지는 일은 없고 공유 픽스처를 사용하다가 디버깅하는 데 고생한 가능성 많기 때문에 `새로운 픽스처를 만드는 방법을 지향`하자.

---

### 4.5 픽스처 수정하기

<br />

```javascript
describe('province', function () {
  ...

  it('change production', function () {
    asia.producers[0].production = 20;
    expect(asia.shortfall).equal(-6);
    expect(asia.profit).equal(292);
  });
});
```

설정 - 실행 - 검증  
조건 - 발생 - 결과  
준비 - 수행 - 단언

---

### 4.6 경계 조건 검사하기

<br />

범위를 벗어나는 경계 지점에서 문제가 생기면 어떤 일이 벌어지는지 확인하는 테스트도 함께 작성하면 좋다.

<br />

```javascript
describe('no producers', function () {
  // 생산자가 없는 경우
  let noProducers;

  beforeEach(function () {
    const data = { name: 'No prouducers', producers: [], demand: 30, price: 20 };
    noProducers = new Province(data);
  });

  it('shortfall', function () {
    expect(noProducers.shortfall).equal(30);
  });

  it('profit', function () {
    expect(noProducers.profit).equal(0);
  });
});
```

컬렉션이 비었을 경우

<br />

```javascript
describe('province', function () {
  ...

  it('zero demand', function () {
    asia.demand = 0;
    expect(asia.shortfall).equal(-25);
    expect(asia.profit).equal(0);
  });
});
```

숫자형이 0일 경우

<br />

```javascript
describe('province', function () {
  ...

  it('zero demand', function () {
    asia.demand = 0;
    expect(asia.shortfall).equal(-25);
    expect(asia.profit).equal(-10);
  });
});
```

음수인 경우

<br />

경계를 확인하는 테스트를 작성해보면 프로그램에서 특이 상황을 어떻게 처리하는게 좋을지 생각해 볼 수 있다.

> 문제가 생길 가능성이 있는 경계 조건을 생각해보고 그 부분을 집중적으로 테스트하자.

<br />

검증보다 앞선 과정에서 발생한 예외 상황  
→ 에러 상황을 지금보다 잘 처리할 수 있도록 코드를 추가

하지만 리팩터링하기 전이라면 이런 테스트를 작성하지 않을 것.  
→ 리팩터링은 겉보기 동작에 영향을 주면 ×

프로그램에서 발생할 수 있는 모든 경우를 테스트하기 위한 다양한 기법  
→ 너무 빠져들 필요 ×  
→ 오히려 테스트를 너무 많이 작성하다 보면 오히려 의욕이 감소  
→ 위험한 부분에만 집중하자.

---

### 4.7 끝나지 않은 여정

<br />

테스트는 리팩터링에 반드시 필요한 토대이자, 그 자체로도 프로그래밍에 중요한 역할.

테스트도 반복적으로 진행.  
→ 기능을 새로 추가할 때마다 테스트도 추가, 기존 테스트도 재검증

리팩터링 후 테스트 결과가 모두 초록색인 것만 보고도 리팩터링 과정에서 생겨난 버그가 하나도 없다고 확신할 수 있다면 충분히 좋은 테스트 스위트.
