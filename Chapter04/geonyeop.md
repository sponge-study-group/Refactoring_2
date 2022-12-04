# 테스트 구축하기

리팩터링을 제대로 하려면 불가피하게 저지르는 실수를 잡아주는 견고한 test suite가 뒷바침돼야 한다.

자동 리팩터링 도구를 확용하더라도 이 책에서 소개하는 리팩터링 중 다수는 테스트 스위트로 재차 검증해야 할 것이다.

단점은 아니다. 리팩터링을 하지 않더라도 좋은 테스트를 작성하는 일은 개발 효율을 높여준다.

## 1. 자가 테스트 코드의 가치

프로그래머들은 대부분의 시간을 디버깅에 쓴다. 버그 수정은 대체로 금방 끝나지만, 진짜 끔찍한 건 버그를 찾는 여정이다.

또한 버그를 잡는 과정에서 다른 버그를 심기도 하는데, 그 사실을 추후에 발견하기도 한다.

<aside>
💡 모든 테스트를 완전히 자동화하고 그 결과까지 스스로 검사하게 만들자
</aside>

<aside>
💡 테스트 스위트는 강력한 버그 검출 도구로, 버그를 찾는 데 걸리는 시간을 대폭 줄여준다.
</aside>

사실 다른 사람들에게 테스트 코드를 함수 몇 개만 작성해도 곧바로 테스트 코드를 작성하라고 설득하기는 녹록지 않다.

테스트를 작성하려면 주 소프트웨어 외의 부가적인 코드를 상당량 작성해야 한다.

그래서 테스트가 실제로 프로그래밍 속도를 높여주는 경험을 직접 해보지 않고서는 자가 테스트의 진가를 납득하긴 어렵다.

테스트를 작성하기 가장 좋은 시점은 프로그래밍을 시작하기 전이다.

나는 기능을 추가할 때 테스트부터 작성한다. 얼핏 순서가 뒤바뀐 듯 하지만, 전혀 그렇지 않다.

1. 원하는 기능을 추가하기 위해 무엇이 필요한지 고민하게 된다.
2. 구현보다 인터페이스에 집중하게 된다는 장점이 있다.
3. 코딩이 완료되는 시점을 정확하게 판단할 수 있다. 테스트를 모두 통과한 시점이 바로 코드를 완성한 시점이다.

켄트벡은 이처럼 테스트부터 작성하는 습관을 바탕으로 테스트 주도 개발(Test-Driven Development)[TDD]이란 기법을 창시했다.

TDD에서는 테스트를 작성하고, 이 테스트를 통과하게끔 코드를 작성하고, 결과 코드를 최대한 깔끔하게 리팩터링하는 과정을 짧은 주기로 반복한다.

이러한 과정은 코드를 대단히 생상적이면서도 차분하게 작성할 수 있다.

리팩터링에는 테스트가 필요하다. 그러니 리팩터링하고 싶다면 테스트를 반드시 작성해야 한다.

이번 장에서는 자바스크립트 프로그램용으로 테스트 코드를 작성하는 방법을 소개한다.

이 책의 스타일대로, 테스트 기법도 예시 중심으로 소개하겠다.

간혹 테스트가 갖춰지지 않은 코드를 리팩터링해야 한다면, 리팩터링 전에 자가 테스트 코드부터 작성한다.

## 2. 테스트할 샘플 코드

테스트 대상이 될 코드는 사용자가 생상 계획을 검토하고 수정하도록 해주는 간단한 애플리케이션의 일부다.

![image](https://user-images.githubusercontent.com/64470512/205473247-44d65456-a0f3-47e1-bea9-31b7ae0b39af.png)

생산계획은 각 지역의 수요와 가격으로 구성되며, 지역에 위치한 생산자들은 제품들을 특정 수량만큼 생산할 수 있다.

화면 맨 아래에는 생산 부족분과 현재 계획에서 거둘 수 있는 총수익도 보여준다.

사용자는 UI에서 수요, 가격, 생산자별 생산량과 비용을 조정해가며, 그에 따른 생산 부족분과 총수익을 확인할 수 있다.

수익과 생산 부족분을 계산하는 비즈니스 로직 클래스들만 살펴보고 HTML을 생성하는 코드는 생략한다.

이 장의 목적은 자가 테스트 코드 작성법을 파악하는데 있으며, UI, 영속성, 외부 서비스 연동과는 관련 없는 코드부터 확인한다.

참고로 코드는 항상 이렇게 성격에 따라 분리하는 것이 좋다.

만약 비즈니스 로직 코드도 아주 복잡해진다면, UI와 분리하여 코드를 파악하고 테스트하기 편하게 수정했을 것이다.

비즈니스 로직 코드는 클래스 두 개로 구성된다.

하나는 생산자를 표현하는 Produce, 다른 하나는 지역 전체를 표현하는 Province다.

Province의 생성자는 JSON 문서로부터 만들어진 자바스크립트 객체를 인수로 받는다.

> Provunce 클래스

```jsx
class Province {
  constructor(doc) {
    this._name = doc.name;
    this._producers = [];
    this._totalProduction = 0;
    this._demand = doc.demand;
    this._price = doc.price;
    doc.producers.forEach((d) => this.addProducer(new Producer(tyhis, d)));
  }

  addProducer(arg) {
    this._producers.push(arg);
    this._totalProduction += arg.production;
  }
}
```

다음의 sampleProvinceData() 함수는 앞 생성자의 인수로 쓸 JSON 데이터를 생성한다.

이 함수를 테스트하려면 이 함수가 반환한 값을 인수로 넘겨서 Province 객체를 생성해보면 된다.

```jsx
function sampleProvinceData() {
  return {
    name: "Asia",
    producers: [
      { name: "Byzantium", cost: 10, production: 9 },
      { name: "Attalia", cost: 12, production: 10 },
      { name: "Sinope", cost: 10, production: 6 },
    ],
    demand: 30,
    price: 20,
  };
}
```

Province 클래스에는 다양한 데이터에 대한 접근자들이 담겨 있다.

```jsx
get name() {
  return this.name;
}
get producers() {
  return this.producers;
}
get totalProduction() {
  return this.totalProduction;
}
set totalProduction(arg) {
  return (this.totalProduction = arg);
}
get demand() {
  return this.demand;
}
set demand(arg) {
  this.demand = parseInt(arg);
} // 숫자로 파싱해서 저장
get price() {
  return this.price;
}
set price(arg) {
  this.price = parseInt(arg);
} // 숫자로 파싱해서 저장
```

세터는 UI에서 입력한 숫자를 인수로 받는데, 문자열로 전달되므로 숫자로 파싱한다.

Producer 클래스는 주로 단순한 데이터 저장소로 쓰인다.

```jsx
class Producer {
  constructor(aProvince, data) {
    this.province = aProvince;
    this.cost = data.cost;
    this.name = data.name;
    this.production = data.production || 0;
  }

  get name() {
    return this.name;
  }
  get cost() {
    return this.cost;
  }
  set cost(arg) {
    this.cost = parseInt(arg);
  }

  get production() {
    return this.production;
  }
  set production(amountStr) {
    const amount = parseInt(amountStr);
    const newProduction = Number.isNaN(amount) ? 0 : amount;
    this.province.totalProduction += newProduction - this.production;
    this.production = newProduction;
  }
}
```

set production()이 계산 결과를 지역 데이터(province)에 갱신하는 코드가 조금 지저분하다.

나는 이런 코드를 목격하면 리팩터링해서 제거하고 싶어지지만, 그러려면 먼저 테스트를 작성해야 한다.

생산 부족분을 계산하는 코드는 간단하다.

```jsx
class Province {
	...
	get shortfall() {
    return this.demand - this.totalProduction;
  }
	...
}
```

수익을 계산하는 코드는 살짝 복잡하다.

```jsx
class Province {
	...
	get profit() {
    return this.demandValue - this.demandCost;
  }

  get demandValue() {
    return this.satisfiedDemand * this.price;
  }

  get satisfiedDemand() {
    return Math.min(this.demand, this.totalProduction);
  }

  get demandCost() {
    let remainingDemand = this.demand;
    let result = 0;
    this.producers.sort((a, b) => a.cost - b.cost)
    .forEach(p => {
      const contribution = Math.min(remainingDemand, p.production);
      remainingDemand -= contribution;
      result += contribution * p.cost;
    });
    return result;
  }
	...
}
```

## 3. 첫 번째 테스트

이 코드를 테스트하기 위해서는 먼저 테스트 프레임워크를 마련해야 하며, 이 장에선 모카(Mocha)를 사용한다.

> 생산 부족분 계산 테스트 코드

```jsx
describe('province', function(){
	it('shortfall', function(){
		const asia = new Provincee(sampleProvinceData()); // 1. 픽스처 설정
		assert.equal(asia.shortfall, 5)); // 2. 검증
	});
});
```

모카 프레임워크는 테스트 코드를 블록 단위로 나눠서 각 블록에 테스트 스위트를 담는 구조다.

테스트는 it블록에 담기며, 앞의 예에서는 테스트를 두 단계로 진행했다.

1. 테스트에 필요한 데이터와 객체를 뜻하는 픽스처(고정장치)를 설정한다.
2. 두 번째 단계에서는 이 픽스처의 속성들을 검증하는데, 여기서는 초깃값에 기초하여 생산 부족분을 정확히 계산했는지 확인한다.

콘솔에는 다음과 같이 출력된다.

```
1 passing (61ms)
```

<aside>
💡 실패해야 할 상황에서는 반드시 실패하게 만들자.

</aside>

기존 코드를 검증하는 테스트를 작성하고 모두 통과했단 건 좋은 일이다.

하지만 수많은 테스트를 실행했음에도 실패하는 게 없다면 테스트가 실패하는 게 없다면 한번쯤 의심하라.

그래서 각각의 테스트가 실패하는 모습을 최소한 한 번씩은 직접 확인해본다.

이를 위해 흔히 쓰는 방법은 일시적으로 코드에 오류를 주입하는 것이다.

```jsx
class Province {
	...
	get shortfall(){
		return this.demand - this.totalProduction * 2; // 오류 주입
	}
	...
}
```

수정 후 테스트를 다시 실행하면 다음과 같이 출력된다.

```jsx
0 passing (72ms)
1 failing

1) province shortfall:
	AssertionError : expected -20 to equal 5
	at Context.<anonymous> (src/tester.js:10:12)
```

이처럼 무언가 문제가 생겼을 때 즉시 알 수 있다. 게다가 어느 테스트가 실패했는지 짚어주고, 단서도 제공한다.

<aside>
💡 자주 테스트하라. 작성중인 코드는 최소한 몇 분 간격으로 테스트하고, 적어도 하루에 한 번은 전체 테스트를 돌려보자.

</aside>

실전에서는 테스트의 수는 수천 개 이상일 수 있다.

간결한 피드백은 자가 테스트에서 매우 중요하다.

모카 프레임워크는 소위 어서션(assertion) 라이브러리라고 하는 픽스처 검증 라이브러리를 선택해 사용할 수 있다.

이 책ㄹ에서는 Chai(차이) 라이브러리를 사용하겠다.차이를 사용하면 다음과 같이 assert문을 활용할 수 있다.

```jsx
describe('province', function(){
	it('shortfall', function(){
		const asia = new Provincee(sampleProvinceData());
		**assert.equal(asia.shortfall, 5));**
	});
});
```

또는 다음과 같이 expect 문을 이용할 수도 있다.

```jsx
describe("province", function () {
  it("shortfall", function () {
    const asia = new Provincee(sampleProvinceData());
    expect(asia.shortfall).equal(5);
  });
});
```

개인적으로 assert를 선호하지만 자바스크립트를 다룰 때는 expect를 주로 사용할 것이다.

테스트를 실행하는 방식은 테스트 환경마다 다르다.

자바로 프로그래밍할 때는 GUI 테스트러너를 제공하는 IDE를 사용한다.

GUI 환경에서는 테스트가 실행되면 프로그레스바가 초록색으로 표시되다가 하나라도 실패하면 빨간색으로 바뀐다.

그래서 흔히들 “빨간 막대일 때는 리팩터링하지 말라”라고 말한다.

허나, 이게 중요한 것이 아닌 모든 테스트가 통과했다는 사실을 빨리 알 수 있는 것이 핵심이다.

## 4. 테스트 추가하기

이번에는 클래스가 하는 일을 모두 살펴보고 각각의 기능에서 오류가 생길 수 있는 조건을 하나씩 테스트하는 식으로 진행하겠다.

public 메서드를 빠짐없이 테스트 하는 방식이 아닌, 위험 요인을 중심으로 작성해야 한다!

단순히 필드를 읽고 쓰기만 하는 접근자는 테스트할 필요가 없다.

이런 코드는 너무 단순해서 버그가 숨어들 가능성도 별로 없다.

<aside>
💡 완벽하게 만드느라 테스트를 수행하지 못하느니, 불완전한 테스트라도 작성해 실행하는 게 낫다.

</aside>

이 맥락에서 샘플 코드의 또 다른 주요 기능인 총수익 계산 로직을 테스트하겠다.

```jsx
describe("province", function () {
  it("shortfall", function () {
    const asia = new Provincee(sampleProvinceData());
    expect(asia.shortfall).equal(5);
  });
  it("profit", function () {
    const asia = new Provincee(sampleProvinceData());
    expect(asia.profit).equal(230);
  });
});
```

이전과 마찬가지로, 기댓 값을 제대로 수행하는지 확인 후, 오류를 심어 검출해 내는지까지 확인한 후 코드를 원복한다.

또한, 일반 코드와 같이 테스트 코드에서도 중복은 의심해봐야한다.

그러니 첫 줄의 픽스처를 중복 제거해보기 위해 먼저 바깥 범위로 끌어내는 방법을 시도해보자.

```jsx
describe("province", function () {
  const asia = new Provincee(sampleProvinceData()); // 이렇게 하면 안 된다.
  it("shortfall", function () {
    expect(asia.shortfall).equal(5);
  });
  it("profit", function () {
    expect(asia.profit).equal(230);
  });
});
```

일시적인 효과는 있겠지만, 테스트 관련 버그 중 가장 지저분한 유형인 ‘테스트끼리 상호작용하게 하는 공유 픽스처’를 생성하는 원인이 된다.

이렇게 되면 테스트를 실행하는 순서에 따라 결과가 달라질 수 있다.

그래서 나는 다음 방식을 선호한다.

```jsx
describe("province", function () {
  let asia;
  beforeEach(function () {
    asia = new Provincee(sampleProvinceData());
  });
  it("shortfall", function () {
    expect(asia.shortfall).equal(5);
  });
  it("profit", function () {
    expect(asia.profit).equal(230);
  });
});
```

beforeEach 구문은 각각의 테스트 바로 전에 실행되어 asia를 초기화하기 때문에 모든 테스트가 새로운 asia를 사용하게 된다.

매번 픽스처를 생성하느라 테스트가 느려질 수 있겠지만, 거의 없다.

정말 문제가 될 때는 공유 픽스처를 사용하기도 하지만, 이 때 어떠한 테스트도 픽스처 값을 변경하지 못하도록 주의한다.

또한 불변임이 확실한 픽스처는 공유하기도 한다. 그래도 가장 선호하는 방식은 매번 새로운 픽스처를 만드는 것이다.

beforeEach 블록의 등장은 내가 표준 픽스처를 사용한다는 사실을 알려준다.

코드를 읽는 이들은 describe 블록 안의 테스트들이 모두 똑같은 기준 데이터를 사용한다는 사실을 쉽게 알 수 있다.

## 5. 픽스처 수정하기

지금까지 작성한 테스트 코드를 통해 픽스처를 불러와 그 속성을 확인하는 방법을 알 수 있었다.

그런데 픽스처의 내용이 수정되는 경우가 흔하다.

대부분 세터에서 이뤄지는데, 세터는 보통 단순하여 잘 테스트하지 않는다.

하지만 Producer의 production() 세터는 좀 복잡한 동작을 수행하기 때문에 테스트를 진행하겠다.

```jsx
describe('province', function(){
	...
	it('change production', function(){
		asia.producers[0].production = 20;
		expecet(asia.shortfall.equal(-6);
		expecet(asia.profit.equal(292);
	});
});
```

흔히 보는 패턴이다. before 블록에서 ‘설정’한 표준 픽스처를 취해서, 테스트를 ‘수행’하고,

이 픽스처가 일을 기대한 대로 처리했는지를 ‘검증’한다.

이 패턴은 아래 세 가지로 불린다.

1. 설정-실행-검증 (setup-exercise-verify)
2. 조건-발생-결과 (given-when-then)
3. 준비-수행-단언 (arrange-act-assert)

이 세가지 단계가 한 테스트 안에 모두 담겨 있을 수도 있고, 초기 공통 작업을 beforeEach와 같은 표준 설정 루틴에 모으기도 한다.

```
'해체(teardown)' 또는 '청소(cleanup)'라고 하는 네 번째 단계도 있는데 명시적으로 언급하지 않을 때가 많다.
beforeEach처럼 프레임워크가 알아서 해체해주기 때문이다.
드물지만 해체를 명시적으로 수행해야 할 때도 있다. 특히 생성하는데 오래걸려서 여러 테스트가 공유하는 경우가 해당한다.
```

이 테스트는 it 구문 하나에서 두 가지 속성을 검증하고 있다.

일반적으로 구문 하나당 검증도 하나씩만 하는 게 좋다. 앞쪽 검증에 실패하면 나머지는 실행해보지 못하기 떄문이다.

## 6. 경계 조건 검사하기

지금까지 작성한 테스트는 모든 일이 순조롭고 사용자도 우리 의도대로 사용하는, 일명 꽃길 상황에 집중하였다.

그런데 이 범위를 벗어나는 경계 지점에서 문제가 생기면 어떤 일이 벌어지는지 확인하는 테스트도 함께 작성하면 좋다.

나는 이번 예시의 producers와 같은 컬렉션을 마주하면 그 컬렉션이 비었을 때 어떤 일이 일어나는지 확인하는 편이다.

```jsx
describe("no producers", function () {
  // 생산자가 없다.
  let noProducers;
  beforeEach(function () {
    const data = {
      name: "No producers",
      producers: [],
      demand: 30,
      price: 20,
    };
    noProducers = new Province(data);
  });
  it("shortfall", function () {
    expect(noProducers.shortfall).equal(30);
  });
  it("profit", function () {
    expect(noProducers.profit).equal(0);
  });
});
```

숫자형이라면 0일 때와 음수일 때를 검사해본다.

```jsx
describe('province', function(){
	...
	it('zero demand', function(){ // 수요가 없다.
    asia.demand = 0;
    expect(asia.shortfall).equal(-25);
    expect(asia.profit).equal(0);
  })
  it('negative demand', function(){ // 수요가 음수다.
    asia.demand = 0;
    expect(asia.shortfall).equal(-26);
    expect(asia.profit).equal(-10);
  })
```

여기서 수요가 음수일 때 수익이 음수라는 내용이 고객 관점에서 말이 되는 소리일까?

수요의 최솟값은 0이어야 하지 않나 ? 예리한 지적이다.

이처럼 경계를 확인하는 테스트를 작성해보면 프로그램에서 이런 특이 상황을 어떻게 처리하는게 좋을지 생각해볼 수 있다.

<aside>
💡 문제가 생길 가능성이 있는 경계 조건을 생각해보고 그 부분을 집중적으로 테스트하자.

</aside>

이 프로그램의 세터들은 의미상 숫자만 입력받아야 하지만 UI로부터 문자열을 취하고 있다.

그러므로 필드가 아예 비어있을 수도 있으며 이때도 잘 처리되는지 반드시 테스트해야 한다.

```jsx
describe('province', function(){
	...
	it('empty string demand', function(){
		asia.demand = "";
    expect(asia.shortfall).NaN;
    expect(asia.profit).NaN;
	});
});
```

나는 의식적으로 프로그램을 망가뜨리는 방법을 모색하는데, 이런 마음 자세가 생산성과 재미를 끌어올려준다.

이어서 흥미로운 테스트를 준비했다.

```jsx
describe("string for producers", function () {
  it("", function () {
    const data = {
      name: "String producers",
      producers: "",
      demand: 30,
      price: 20,
    };
    const prov = new Province(data);
    expect(prov.shortfall).equal(0);
  });
});
```

이 테스트는 단순히 생산 부족분이 0이 아니라는 실패 메시지를 출력하는 대신 다음과 같이 출력한다.

```
9 passing (74ms)
1 failing

1) string for producers :
	TypeError : doc.producers.forEach is not a function
	at new Province(src/main.js:22:19)
	at Context.<anonymous> (src/tester.js:86:18)
```

모카는 이 경우를 실패로 처리한다. 하지만 모카와 달리 에러와 실패를 구분하는 테스트 프레임워크도 많다.

실패(failure)란 검증 단계에서 실제 값이 예상 범위를 벗어났다는 뜻이다. 에러는 성격이 다르다.

프로그래머는 이 상황에서 에러를 수정하기 위해 코드를 추가하거나, 오류 메시지를 남기거나, producers를 빈 배열로 설정할 수도 있다.

물론 지금 상태로 남겨둘 합당한 이유가 있을 수도 있다.

예컨대 입력 객체를 (같은 코드베이스 안처럼) 신뢰할 수 있는 곳에서 만들어주는 경우가 여기 속한다.

같은 코드베이스의 모듈 사이에 유효성 검사 코드가 너무 많으면 다른 곳에서 확인한 걸 중복으로 검증하여 문제가 될 수 있다.

반면, JSON으로 인코딩된 요청처럼 외부에서 들어온 입력 객체는 유효한지 확인해야 한다.

나는 리팩터링하기 전이라면 이런 테스트를 작성하지 않을 것이다.

리팩터링은 겉보기 동작에 영향을 주지 않아야 하며, 이런 오류는 겉보기 동작에 해당하지 않는다.

따라서 경계 조건에 대응하는 동작이 리팩터링 때문에 변하는지는 신경 쓸 필요가 없다.

```
이런 오류로 인해 프로그램 내부에 잘못된 데이터가 흘러서 디버깅하기 어려운 문제가 발생한다면
**어서션 추가하기**를 적용해서 오류가 최대한 빨리 드러나게 하자.
어서션도 일종의 테스트로 볼 수 있으니 테스트 코드를 따로 작성할 필요는 없다.
```

<aside>
💡 어차피 모든 버그를 잡아낼 수 없다고 테스트를 작성하지 않는다면 대다수의 버그를 잡을 기회를 날리는 셈이다.

</aside>

아무리 테스트해도 버그 없는 완벽한 프로그램을 만들 수는 없다는 말은 많이 들어봤을 것이다.

맞는 말이지만, 테스트가 프로그래밍 속도를 높여준다는 사실에는 변함이 없다.

모든 경우를 테스트하는 기법들이 도움이 되기는 하지만 너무 맹신하지는 말라.

테스트에도 수확 체감 법칙이 적용된다. 또, 너무 많이 작성하면 오히려 의욕이 떨어져 하나도 작성하지 않을 수도 있다.

위험한 부분에 집중하라. 모든 버그는 걸러주지 못해도 안심하고 리팩터링할 수 있는 보호막은 되어준다.

그리고 리팩터링을 하면서 프로그램을 더욱 깊이 이해하게 되어 더 많은 버그를 찾게 된다.

나는 항상 테스트 스위트부터 갖춘 뒤에 리팩터링하지만, 리팩터링하는 동안에도 계속해서 테스트를 추가한다.

## 7. 끝나지 않은 여정

결국 이 책의 주제는 테스트가 아닌 리팩터링이다. 하지만 테스트도 굉장히 중요한 주제다.

리팩터링에 반드시 필요한 토대일 뿐 아니라, 그 자체로도 프로그래밍에 중요한 역할을 한다.

이 장에서 보여준 테스트는 단위 테스트(unit test)에 해당한다. 단위 테스트란 코드의 작은 영역만을 대상으로 빠르게 실행되도록 설계된 테스트다.

단위 테스트는 자가 테스트 코드의 핵심이다. 대부분 단위 테스트가 차지한다.

물론 다른 다양한 유형의 테스트도 있다.

다른 프로그래밍 활동과 마찬가지로 테스트도 반복적으로 진행한다.

한번에 완벽한 테스트를 갖출 순 없다.

기능을 새로 추가할 때마다 테스트도 추가하는 것은 물론, 기존 테스트도 다시 살펴본다.

버그를 발견하는 즉시 발견한 버그를 명확히 잡아내는 테스트부터 작성하는 습관을 들이자.

그러면 해당 버그가 다시 나타나지 않는지 확인할 수 있다. 또한 그 버그와 테스트를 계기로 또 다른 구멍은 없는지까지 살펴본다.

<aside>
💡 버그 리포트를 받으면 가장 먼저 그 버그를 드러내는 단위 테스트부터 작성하자.

</aside>

충분한 테스트의 명확한 기준은 없다. 테스트 커버리지를 기준으로 삼는 사람도 있지만,

코드에서 테스트하지 않은 영역을 찾는 데만 도움될 뿐, 품질과는 크게 상관 없다.

테스트 스위트가 충분한지를 평가하는 기준은 주관적이다. 가령 ‘누군가 결함을 심으면 테스트가 발견할 수 있다는 믿음’을 기준으로 할 수 있다.

리팩터링 후 테스트 결과를 모두 통과한 것만 보고도 리팩터링 과정에서 생겨난 버그가 하나도 없다고 확인할 수 있다면 좋은 테스트 스위트이다.

테스트를 너무 많이 작성할 가능성도 있다. 제품 코드보다 테스트 코드를 수정하는 데 시간이 더 걸린다면,

테스트 때문에 개발 속도가 느려진다고 생각되면 테스트를 과하게 작성한 건 아닌지 의심해보자.

하지만 너무 많은 경우보다는 너무 적은 경우가 훨씬 많다.
