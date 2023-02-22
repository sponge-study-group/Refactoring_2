# [11] API 리팩터링

모듈과 함수는 소프트웨어를 구성하는 빌딩 블록이며, API는 이 블록들을 끼워 맞추는 연결부다.

이런 API를 이해하기 쉽고 사용하기 쉽게 만드는 일은 중요한 동시에 어렵기도 하다.

그래서 API를 개선하는 방법을 새로 깨달을 때마다 그에 맞게 리팩터링해야 한다.

좋은 API는 데이터를 갱신하는 함수와 그저 조회만 하는 함수를 명확히 구분한다.

두 기능이 섞여 있다면 **질의 함수와 변경 함수 분리하기**를 적용해 갈라놔야 한다.

값 하나 때문에 여러 개로 나뉜 함수들은 **함수 매개변수화하기**를 적용해 하나로 합칠 수 있다.

한편, 어떤 매개변수는 그저 함수의 동작 모드를 전환하는 용도로만 쓰이는데, 이럴 때는 **플래그 인수 제거하기**를 적용하면 좋다.

데이터 구조가 함수 사이를 건너 다니면서 필요 이상으로 분해될 때는 **객체 통째로 넘기기**를 적용해 하나로 유지하면 깔끔해진다.

무언가를 매개변수로 건네 피호출 함수가 판단할지 아니면 호출 함수가 직접 정할지에 관해서는 만고불변의 진리는 없으니,

상황이 바뀌면 **매개변수를 질의 함수로 바꾸기**와 **질의 함수를 매개변수로 바꾸기**로 균형점을 옮길 수 있다.

클래스는 대표적인 모듈이다. 나는 내가 만든 객체가 되도록 불변이길 원하므로 기회가 될 때마다 **세터 제거하기**를 적용한다.

한편, 호출자에 새로운 객체를 만들어 반환하려 할 때 일반적인 생성자의 능력만으로 부족할 때가 있다.

이럴 땐 **생성자를 팩터리 함수로 바꾸기**가 좋은 해법일 수 있다.

마지막 두 리팩터링은 수많은 데이터를 받는 복잡한 함수를 잘게 쪼개는 문제를 다룬다.

**함수를 명령으로 바꾸기**를 적용하면 이런 함수를 객체로 변환할 수 있는데, 그러면 해당 함수의 본문에서 함수를 추출하기가 수월해진다.

나중에 이함수를 단순화하여 명령 객체가 더는 필요 없어진다면 **명령을 함수로 바꾸기**를 적용해 함수로 되돌릴 수 있다.

함수 안에서 데이터가 수정됐음을 확실히 알리려면 **수정된 값 반환하기**를 적용한다.

오류 코드에 의존하는 과거 방식 코드는 **오류 코드를 예외로 바꾸기**로 정리한다.

단, 예외는 올바른 상황에서만 정확하게 적용해야 한다.

특히, 문제가 되는 조건을 함수 호출 전에 검사할 수 있다면 **예외를 사전 확인으로 바꾸기**로 예외 남용을 줄일 수 있다.

## 질의 함수와 변경 함수 분리하기(Separate Query from Modiffier)

```jsx
function getTotalOutstandingAndSendBill() {
  const result = customer.invoices.reduce(
    (total, each) => each.amount + total,
    0
  );
  sendBill();
  return result;
}
```

```jsx
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);
}

function sendBill() {
  emailGateway.send(formatBill(customer));
}
```

### 배경

우리는 외부에서 관찰할 수 있는 겉보기 부수효과가 전혀 없이 값을 반환해주는 함수를 추구해야 한다.

이런 함수는 어느 때건 원하는 만큼 호출해도 아무 문제가 없다.

호출하는 문장의 위치를 호출하는 함수 안 어디로든 옮겨도 되며 테스트하기도 쉽다.

한마디로, 이용할 때 신경 쓸 거리가 매우 적다.

겉보기 부수효과가 있는 함수와 없는 함수는 명확히 구분하는 것이 좋다.

이를 위한 한 가지 방법은 ‘질의함수(읽기 함수)는 모두 부수효과가 없어야 한다’는 규칙을 따르는 것이다.

이를 명령-질의 분리(command-query separation)라 하는데, 이 규칙을 절대적으로 신봉하는 프로그래머도 있다.

나는 이 견해에(혹은 다른 무엇이라도) 100% 동의하지는 않지만 되도록 따르려 노력하고 있으며, 그동안 효과도 톡톡히 봤다.

나는 값을 반환하면서 부수효과도 있는 함수를 발견하면 상태를 변경하는 부분과 질의하는 부분을 분리하여 시도한다. 무조건이다 !

내가 ‘겉보기’ 부수효과라고 한 데는 이유가 있다. 흔히 쓰는 최적화 기법 중 요청된 값을 캐시해두고 다음번 호출 때 빠르게 응답하는 방법이 있는데,

이러한 캐싱도 객체의상태를 변경하지만 객체 밖에서는 관찰할 수 없다. 즉, 겉보기 부수효과 없이 어떤 순서로 호출하든 모든 호출에 항상 똑같은 값을 반환할 뿐이다.

---

### 절차

1. 대상 함수를 복제하고 질의 목적에 충실한 이름을 짓는다.

   → 함수 내부를 살펴 무엇을 반환하는지 찾는다. 어떤 변수의 값을 반환한다면 그 변수 이름이 훌륭한 단초가 될 것이다.

2. 새 질의 함수에서 부수효과를 모두 제거한다.
3. 정적 검사를 수행한다.
4. 원래 함수(변경 함수)를 호출하는 곳을 모두 찾아낸다. 호출하는 곳에서 반환 값을 사용한다면 질의 함수를 호출하도록 바꾸고,

   원래 함수를 호출하는 코드를 바로 아래 줄에 새로 추가한다. 하나 수정할 때마다 테스트한다.

5. 원래 함수에서 질의 관련 코드를 제거한다.
6. 테스트한다.

이 리팩터링을 마친 후에는 새로 만든 질의 함수와 원래 함수에 (정리해야 할) 중복이 남아 있을 수 있다.

---

### 예시

이름 목록을 훑어 악당(miscreant)을 찾는 함수를 준비했다. 악당을 찾으면 그 사람의 이름을 반환하고 경고를 울린다.

이 함수는 가장 먼저 찾은 악당만 취급한다.

```jsx
function alertForMiscreant(people) {
  for (const p of people) {
    if (p === "조커") {
      setOffAlarms();
      return "조커";
    }
    if (p === "시루만") {
      setOffAlarms();
      return "시루만";
    }
  }
  return "";
}
```

[1] 첫 단계는 함수를 복제하고 질의 목적에 맞는 이름짓기다.

```jsx
function findMiscreant(people) {
	... // 위와 동일
}
```

[2] 새 질의 함수에서 부수효과를 낳는 부분을 제거한다.

```jsx
function findMiscreant(people) {
  for (const p of people) {
    if (p === "조커") {
      //     setOffAlarms();
      return "조커";
    }
    if (p === "시루만") {
      //     setOffAlarms();
      return "시루만";
    }
  }
  return "";
}
```

[4] 이제 원래 함수를 호출하는 곳을 모두 찾아서 새로운 질의 함수를 호출하도록 바꾸고, 이어서 원래의 변경 함수를 호출하는 코드를 바로 아래에 삽입한다.

```jsx
// 변경 전
const found = alertForMiscreant(people);

// 변경 후
const found = findMiscreant(people);
alertForMiscreant(people);
```

[5] 이제 원래의 변경 함수에서 질의 관련 코드를 없앤다.

```jsx
function alertForMiscreant(people) {
  for (const p of people) {
    if (p === "조커") {
      setOffAlarms();
      return;
    }
    if (p === "시루만") {
      setOffAlarms();
      return;
    }
  }
  return;
}
```

리팩터링은 마쳤지만 변경 함수와 새 질의 함수에는 중복된 코드가 많아 보인다. 이번 경우엔 변경 함수에서 질의 함수를 사용하도록 고치면 해결된다.

```jsx
function alertForMiscreant(people) {
  if (findMiscreant(people) !== "") setOffAlarms();
}
```

---

## 함수 매개변수화하기(Parameterize Function)

```jsx
function tenPercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.05);
}
```

```jsx
function raise(aPerson, factor) {
  aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

### 배경

두 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면, 그 다른 값만 매개변수로 받아 처리하는 함수 하나로 합쳐서 중복을 없앨 수 있다.

이렇게 하면 매개변수 값만 바꿔서 여러 곳에서 쓸 수 있으니 함수의 유용성이 커진다.

---

### 절차

1. 비슷한 함수 중 하나를 선택한다.
2. 함수 선언 하꾸기(6.5)로 리터럴들을 매개변수로 추가한다.
3. 이 함수를 호출하는 곳 모두에 적절한 리터럴 값을 추가한다.
4. 테스트한다.
5. 매개변수로 받은 값을 사용하도록 함수 본문을 수정한다. 하나 수정할 때마다 테스트한다.
6. 비슷한 다른 함수를 호출하는 코드를 찾아 매개변수화된 함수를 호출하도록 하나씩 수정한다. 하나 수정 하나 테스트다.

   → 매개변수화된 함수가 대체할 비슷한 함수와 다르게 동작한다면, 그 비슷한 함수의 동작도 처리할 수 있도록 본문 코드를 적절히 수정한 후 진행한다.

---

### 예시

먼저 명백한 예를 보자.

```jsx
function tenPercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.05);
}
```

앞의 두 함수는 확실히 다음 함수로 대체할 수 있다.

```jsx
function raise(aPerson, factor) {
  aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```

하지만 이렇게 간단히 끝나지 않는 경우도 있다. 다음 코드를 보자.

```jsx
function baseCharge(usage) {
  if (usage < 0) return use(0);
  const amount =
    bottomBand(usage) * 0.03 + middleBand(usage) * 0.05 + topBand(usage) * 0.07;
  return usd(amount);
}

function bottomBand(usage) {
  return Math.min(usage, 100);
}

function middleBand(usage) {
  return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}

function topBand(usage) {
  return usage > 200 ? usage - 200 : 0;
}
```

대역(band)를 다루는 세 함수의 로직이 상당히 비슷한 건 사실이지만, 과연 매개변수화 함수로 통합할 수 있을 만큼 비슷한가 ?

그렇다. 하지만 앞의 간단한 예보다는 덜 직관적이다.

[1] 비슷한 함수들을 매개변수화하여 통합할 때는 먼저 대상 함수 중 하나를 골라 매개변수를 추가한다.

단, 다른 함수들까지 고려해 선택해야 한다. 지금 예처럼 범위를 다루는 로직에서는 대게 중간에 해당하는 함수에서 시작하는 게 좋다.

그러니 middleBand()에 매개변수를 추가하고 다른 호출들을 여기에 맞춰보자.

[2] middleBand()는 리터럴을 두 개(100, 200) 사용하며, 그 각각은 중간 대역의 하한(bottom)과 상한(top)을 뜻한다.

함수 선언 바꾸기(6.5)를 적용하고 [3] 이 리터럴들을 호출 시점에 입력하도록 바꿔보자.

이 과정에서 함수 이름도 매개변수화된 기능에 어울리게 수정한다.

```jsx
function baseCharge(usage) {
  if (usage < 0) return use(0);
  const amount =
    bottomBand(usage) * 0.03 + withinBand(usage) * 0.05 + topBand(usage) * 0.07;
  return usd(amount);
}

function bottomBand(usage) {
  return Math.min(usage, 100);
}

function withinBand(usage, bottom, top) {
  return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}

function topBand(usage) {
  return usage > 200 ? usage - 200 : 0;
}
```

[5] 함수에서 사용하던 리터럴들을 적절한 매개변수로 대체한다.

```jsx
function withinBand(usage, bottom, top) {
  return usage > bottom ? Math.min(usage, top) - bottom : 0;
}
```

[6] 대역의 하한을 호출하는 부분도 새로 만든 매개변수화 함수를 호출하도록 바꾼다.

```jsx
function baseCharge(usage) {
  if (usage < 0) return use(0);
  const amount =
    withinBand(usage, 0, 100) * 0.03 +
    withinBand(usage, 100, 200) * 0.05 +
    topBand(usage) * 0.07;
  return usd(amount);
}

// function bottomBand(usage) {
//   return Math.min(usage, 100);
// }

function withinBand(usage, bottom, top) {
  return usage > bottom ? Math.min(usage, top) - bottom : 0;
}

function topBand(usage) {
  return usage > 200 ? usage - 200 : 0;
}
```

대역의 상한 호출을 대체할 때는 무한대를 뜻하는 Infinity를 이용하면 된다.

```jsx
function baseCharge(usage) {
  if (usage < 0) return use(0);
  const amount =
    withinBand(usage, 0, 100) * 0.03 +
    withinBand(usage, 100, 200) * 0.05 +
    withinBand(usage, 200, Infinity) * 0.07;
  return usd(amount);
}

function withinBand(usage, bottom, top) {
  return usage > bottom ? Math.min(usage, top) - bottom : 0;
}

// function topBand(usage) {
//   return usage > 200 ? usage - 200 : 0;
// }
```

이제 로직이 의도한 대로 동작하니 초기의 보호 구문을 제거해도 된다.

하지만 논리적으로는 필요 없어졌다고 해도, 예외 상황에서의 대처 방식을 잘 설명해주므로 그냥 두기로 했다.

---

## 플래그 인수 제거하기(Remove Flag Argument)

```jsx
function setDimension(name, value) {
  if (name === "height") {
    this.height = value;
    return;
  }
  if (name === "width") {
    this.width = value;
    return;
  }
}
```

```jsx
function setHeight(value) {
  this.height = value;
}
function setWidth(value) {
  this.width = value;
}
```

### 배경

플래그 인수(flag argument)란 호출되는 함수가 실행할 로직을 호출하는 쪽에서 선택하기 위해 전달하는 함수다.

```jsx
function bookContert(aCustomer, isPremium) {
  if (isPremium) {
    // 프리미엄 예약용 로직
  } else {
    // 일반 예약용 로직
  }
}
```

콘서트를 프리미엄으로 예약하려면 다음처럼 호출해야 한다.

```jsx
bookConcert(aCustomer, true);
```

플래그 인수가 enum일 수도 있다.

```jsx
bookConcert(aCustomer, CustomerType.PREMIUM);
```

문자열 (혹은 해당 프로그래밍 언어가 제공하는 또 다른 타입)을 쓰기도 한다.

```jsx
bookConcert(aCustomer, "premium");
```

내가 플래그 인수를 싫어하는 이유가 있다. 호출할 수 있는 함수들이 무엇이고 어떻게 호출해야 하는지를 이해하기가 어려워지기 때문이다.

나는 API를 익힐 때 주로 함수 목록부터 살펴보는데, 플래그 인수가 있으면 함수들의 기능 차이가 잘 드러나지 않는다.

사용할 함수를 선택한 후에도 플래그 함수로 어떤 값을 넘겨야 하는지를 또 알아내야 한다.

boolean flag는 코드를 읽는 이에게 뜻을 온전히 전달하지 못하기 때문에 더욱 좋지 못하다.

함수에 전달한 true의 의미가 대체 뭐란 말인가? 이보다는 다음처럼 특정한 기능 하나만 수행하는 명시적인 함수를 제공하는 편이 훨씬 깔끔하다.

```jsx
premiumBookConcert(aCustomer);
```

이렇게 생긴 인수라고 해서 다 플래그 인수는 아니다.

플래그 인수가 되려면 호출하는 쪽에서 boolean 값으로 (프로그램에서 사용되는 데이터가 아닌) 리터럴 값을 건네야 한다.

또한, 호출되는 함수는 그 인수를 (다른 함수에 전달하는 데이터가 아닌) 제어 흐름을 결정하는 데 사용해야 한다.

플래그 인수를 제거하면 코드가 깔끔해짐은 물론 프로그래밍 도구에도 도움을 준다.

예컨대 코드 분석 도구는 프리미엄 로직 호출과 일반 로직 호출의 차이를 더 쉽게 파악할 수 있게 된다.

함수 하나에서 플래그 인수를 두 개 이상 사용하면 플래그 인수를 써야 하는 함당한 근거가 될 수 있다.

플래그 인수 없이 구현하려면 플래그 인수들의 가능한 조합 수만큼 함수를 만들어야 하기 때문이다.

그런데 다른 관점에서 보자면, 플래그 인수가 둘 이상이면 함수 하나가 너무 많은 일을 처리하고 있다는 신호기도 하다.

그러니 같은 로직을 조합해내는 더 간단한 함수를 만들 방법을 고민해봐야 한다.

---

### 절차

1. 매개변수로 주어질 수 있는 값 각각에 대응하는 명시적 함수들을 생성한다.

   → 주가 되는 함수에 깔끔한 분배 조건문이 포함되어 있다면 조건문 분해가기(10.1)로 명시적 함수들을 생성하자.

   그렇지 않다면 래핑 함수(wrapping function) 형태로 만든다.

2. 원래 함수를 호출하는 코드들을 모두 찾아서 각 리터럴 값에 대응되는 명시적 함수를 호출하도록 수정한다.

---

### 예시

- 일반 예시
  코드를 살펴보던 중 배송일자를 계산하는 호출을 발견했다고 치자. 그중 일부는 다음처럼 호출한다.

  ```jsx
  aShipment.deliveryDate = deliveryDate(anOrder, true);
  ```

  다른 곳에서는 다음처럼 호출한다.

  ```jsx
  aShipment.deliveryDate = deliveryDate(anOrder, false);
  ```

  이 코드들을 확인한 나는 곧바로 ‘이 boolean 값이 뭘 의미하지?’ 란 의문이 떠올랐다.
  deliveryDate() 함수의 코드는 다음과 같았다.

  ```jsx
  function deliveryDate(anOrder, isRush) {
    if (isRush) {
      let deliveryTime;
      if (["MA", "CT"].includes(anOrder.deliveryState)) deliveryTime = 1;
      else if (["NY", "NH"].includes(anOrder.deliveryState)) deliveryTime = 2;
      else deliveryTime = 3;
      return anOrder.placeOn.plusDays(1 + deliveryTime);
    } else {
      let deliveryTime;
      if (["MA", "CT", "NY"].includes(anOrder.deliveryState)) deliveryTime = 2;
      else if (["ME", "NH"].includes(anOrder.deliveryState)) deliveryTime = 3;
      else deliveryTime = 4;
      return anOrder.placeOn(plusDays(2 + deliveryTime));
    }
  }
  ```

  즉, 호출하는 쪽에서는 이 불리언 리터럴 값을 이용해서 어느 쪽 코드를 실행할지를 정한 것이다.
  전형적인 플래그 인수다. 이 함수가 어느 코드를 실행할지는 전적으로 호출자의 지시에 따른다.
  따라서 명시적인 함수를 사용해 호출자의 의도를 분명히 밝히는 편이 나을 것이다.
  [1] 이 예에서라면 조건문 분해하기(10.1)를 적용할 수 있는데, 결과는 다음과 같다.

  ```jsx
  function deliveryDate(anOrder, isRush) {
    if (isRush) {
      rushDeliveryDate(anOrder);
    } else {
      regularDeliveryDate(anOrder);
    }
  }

  function rushDeliveryDate(anOrder) {
    let deliveryTime;
    if (["MA", "CT"].includes(anOrder.deliveryState)) deliveryTime = 1;
    else if (["NY", "NH"].includes(anOrder.deliveryState)) deliveryTime = 2;
    else deliveryTime = 3;
    return anOrder.placeOn.plusDays(1 + deliveryTime);
  }

  function regularDeliveryDate(anOrder) {
    let deliveryTime;
    if (["MA", "CT", "NY"].includes(anOrder.deliveryState)) deliveryTime = 2;
    else if (["ME", "NH"].includes(anOrder.deliveryState)) deliveryTime = 3;
    else deliveryTime = 4;
    return anOrder.placeOn(plusDays(2 + deliveryTime));
  }
  ```

  보다시피 새로 만든 두 함수가 호출자의 의도를 더 잘 드러낸다.
  따라서 다음과 같이 변경할 수 있다.

  ```jsx
  // 변경 전
  aShipment.deliveryDate = deliveryDate(anOrder, true);

  // 변경 후
  aShipment.deliveryDate = rushDeliveryDate(anOrder);
  ```

  나머지 호출도 같은 식으로 바꿔주면 된다.
  모든 호출을 대체했다면 deliveryDate()를 제거한다.
  플래그 인수를 싫어하는 이유는 불리언 값을 사용해서가 아니라, 불리언 값을 변수와 같은 데이터가 아닌 리터럴로 설정하기 때문이다.
  가령 deliveryDate()를 호출하는 코드가 모두 다음처럼 생겼다면 어떨까?

  ```jsx
  const isRush = determineIfRush(anOrder);
  aShipment.deliveryDate = deliveryDate(anOrder, isRush);
  ```

  이런 상황이라면 나는 deliveryDate()의 시그니처에 불만이 없었을 것이다(여전히 조건문 분해하기는 적용할 것이다)
  한편 리터럴을 사용하는 곳과 데이터를 사용하는 곳이 혼재할 수도 있다. 그렇더라도 나라면 플래그 인수를 제거할 것이다.
  단, 데이터를 사용하는 코드는 수정하지 않고 deliveryDate()도 제거하지 않을 것이다.
  이렇게 하여 쓰임새가 다른 두 인터페이스를 모두 지원할 수 있다.

- 매개변수를 까다로운 방식으로 사용할 때
  앞에서 본 것처럼 조건문을 쪼개면 이 리팩터링을 수행하는 게 수월해진다.
  하지만 매개변수에 따른 분배 로직이 함수 핵심 로직의 바깥에 해당할 때만 이용할 수 있다.
  (물론 그렇지 않더라도 이런 형태로 쉽게 리팩터링할 수 있다)
  그런데 매개변수가 훨씬 까다로운 방식으로 사용될 때도 있다. 다음은 까다로운 버전의 deliveryDate()다.
  ```jsx
  function deliveryDate(anOrder, isRush) {
    let result;
    let deliveryTime;
    if (anOrder.deliveryState === "MA" || anOrder.deliveryState === "CT") {
      deliveryTime = isRush ? 1 : 2;
    } else if (
      anOrder.deliveryState === "NY" ||
      anOrder.deliveryState === "NH"
    ) {
      deliveryTime = 2;
      if (anOrder.deliveryState === "NH" && !isRush) {
        deliveryTime = 3;
      }
    } else if (isRush) {
      deliveryTime = 3;
    } else if (anOrder.deliveryState === "ME") {
      deliveryTime = 3;
    } else {
      deliveryTime = 4;
    }
    result = anOrder.placeOn.plusDays(2 + deliveryTime);
    if (isRush) result = result.minusDays(1);
    return result;
  }
  ```
  이 코드에서 isRush를 최상위 분배 조건문으로 뽑아내려면 생각보다 일이 커질 수도 있어 보인다.
  [1] 그렇다면 deliveryDate()를 감싸는 래핑 함수를 생각해볼 수 있다.
  ```jsx
  function rushDeliveryDate(anOrder) {
    return deliveryDate(anOrder, true);
  }
  function regularDeliveryDate(anOrder) {
    return deliveryDate(anOrder, false);
  }
  ```

래핑 함수들을 독립적으로 정의했지만, 새로운 기능을 추가한 게 아니라 각각이 deliveryDate()의 기능 일부만을 제공한다.

[2] 이 두 함수를 추가했다면 호출하는 코드들을 앞에서 조건문을 쪼갰을 때와 똑같은 방식으로 대체할 수 있다.

매개 변수를 데이터로 사용한 호출자가 하나도 없다면 이 함수들의 가시 범위를 제한하거나, 함수 이름을 (아마도 deliveryDateHelperOnly())

바꿔서 직접 호출하지 말라는 뜻을 명시했을 것이다.

---

## 객체 통째로 넘기기(Preserve Whole Object)

```jsx
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))
```

```jsx
if (aPlan.withinRange(aRoom.daysTempRange))
```

### 배경

하나의 레코드에서 값 두어 개를 가져와 인수를 넘기는 코드를 보면, 나는 그 값들 대신 레코드를 통째로 넘기고 함수 본문에서 필요한 값들을 꺼내 쓰도록 수정하곤 한다.

레코드를 통째로 넘기면 변화에 대응하기 쉽다. 예컨대 그 함수가 더 다양한 데이터를 사용하도록 바뀌어도 매개변수 목록은 수정할 필요가 없다.

그리고 매개변수 목록이 짧아져서 일반적으로는 함수 사용법을 이해하기 쉬워진다.

한편, 레코드에 담긴 데이터 중 일부를 받는 함수가 여러 개라면 그 함수들끼리는 같은 데이터를 사용하는 부분이 있을 것이고, 그 부분의 로직이 중복될 가능성이 커진다.

레코드를 통째로 넘긴다면 이런 로직 중복도 없앨 수 있다.

하지만 함수가 레코드 자체에 의존하기를 원치 않을 때는 이 리팩터링을 수행하지 않는데, 레코드와 함수가 서로 다른 모듈에 속한 상황이면 특히 더 그렇다.

어떤 객체로부터 값 몇 개를 얻은 후 그 값들만으로 무언가를 하는 로직이 있다면, 그 로직을 객체 안으로 집어넣어야 함을 알려주는 악취로 봐야 한다.

그래서 객체 통째로 넘기기는 특히 매개변수 객체 만들기(6.8) 후, 즉 산재한 수많은 데이터 더미를 새로운 객체로 묶은 후 적용하곤 한다.

한편, 한 객체가 제공하는 기능 중 항상 똑같은 일부만을 사용하는 코드가 많다면, 그 기능만 따로 묶어서 클래스로 추출하라는 신호일 수 있다.

많은 사람이 놓치는 사례가 하나 더 있다. 다른 객체의 메서드를 호출하면서 호출하는 객체 자신이 가지고 있는 데이터 여러 개를 건네는 경우다.

이런 상황이면 데이터 여러 개 대신 객체 자신의 참조만 건네도록 수정할 수 있다.(자바스크립트라면 this를 건넬 것이다)

---

### 절차

1. 매개변수들을 원하는 형태로 받는 빈 함수를 만든다.

   → 마지막 단계에서 이 함수의 이름을 변경해야 하니 검색하기 쉬운 이름으로 지어준다.

2. 새 함수의 본문에서는 원래 함수를 호출하도록 하며, 새 매개변수와 원래 함수의 매개변수를 매핑한다.
3. 정적 검사를 수행한다.
4. 모든 호출자가 새 함수를 사용하게 수정한다. 하나씩 수정하며 테스트하자.

   → 수정 후에는 원래의 매개변수를 만들어내는 코드 일부가 필요 없어질 수 있다. 따라서 죽은 코드 제거하기로 없앨 수 있을 것이다.

5. 호출자를 모두 수정했다면 원래 함수를 인라인한다.
6. 새 함수의 이름을 적절히 수정하고 모든 호출자에 반영한다.

---

### 예시

- 기본 예시
  실내온도 모니터링 시스템을 생각해보자. 이 시스템은 일일 최저-최고 기온이 난방 계획(heating plan)에서 정한 범위를 벗어나는지 확인한다.
  > 호출자
  ```jsx
  const low = aRoom.daysTempRange.low;
  const high = aRoom.daysTempRange.high;
  if (aPlan.withinRange(low, high))
    alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```
  > HeatingPlan 클래스
  ```jsx
  withinRange(bottom, top) {
  	return (bottom >= this.temperatureRange.low)
  			&& (top <= this.temperatureRange.high);
  }
  ```
  그런데, 최저-최고 기온을 뽑아내어 인수로 건네는 대신 범위 객체를 통째로 건넬 수도 있다.
  [1] 가장 먼저 원하는 인터페이스를 갖춘 빈 메서드를 만든다.
  ```jsx
  class HeatingPlan {
    xxNEWwithinRange(aNumberRange) {}
  }
  ```
  이 메서드로 기존 withinRange() 메서드를 대체할 생각이다. 그래서 똑같은 이름을 쓰되, 나중에 찾아 바꾸기 쉽도록 적당한 접두어만 붙여줬다.
  [2] 그런 다음 새 메서드의 본문은 기존 withinRange()를 호출하는 코드로 채운다. 자연스럽게 새 매개변수를 기존 매개변수와 매핑하는 로직이 만들어진다.
  ```jsx
  class HeatingPlan {
    xxNEWwithinRange(aNumberRange) {
      return this.withinRange(aNumberRange.low, aNumberRange.high);
    }
  }
  ```
  이제 본격적인 작업을 시작할 준비가 되었다. [4] 기존 함수를 호출하는 코드를 찾아서 새 함수를 호출하게 수정하자.
  ```jsx
  const low = aRoom.daysTempRange.low;
  const high = aRoom.daysTempRange.high;
  if (aPlan.xxNEWwithinRange(aRoom.daysTempRange))
    alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```
  [4] 이렇게 다 수정하고 나면 기존 코드 중 더는 필요 없는 부분이 생길 수 있다. 죽은 코드이니 제거한다.
  ```jsx
  // const low = aRoom.daysTempRange.low;
  // const high = aRoom.daysTempRange.high;
  if (aPlan.xxNEWwithinRange(aRoom.daysTempRange))
    alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```
  이런식으로 한 번에 하나씩 수정하면서 테스트한다.
  [5] 모두 새 함수로 대체했다면 원래 함수를 인라인해준다.
  ```jsx
  class HeatingPlan {
    xxNEWwithinRange(aNumberRange) {
      return (
        aNumberRange.low >= this.temperatureRange.low &&
        aNumberRange.high <= this.temperatureRange.high
      );
    }
  }
  ```
  [6] 마지막으로 새 함수에서 보기 흉한 접두어를 제거하고 호출자들에도 모두 반영한다.
  접두어를 활용하면 이름 변경 기능을 제공하지 않는 코드 편집기를 사용하더라도 전체 바꾸기를 간단히 수행할 수 있다.
  ```jsx
  class HeatingPlan {
    withinRange(aNumberRange) {
      return (
        aNumberRange.low >= this.temperatureRange.low &&
        aNumberRange.high <= this.temperatureRange.high
      );
    }
  }
  ```
  ```jsx
  if (aPlan.withinRange(aRoom.daysTempRange))
    alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```
- 새 함수를 다른 방식으로 만들기
  앞 예시에서는 새 메서드(함수)의 코드를 직접 작성했는데, 대부분 상황에서 꽤나 간단하고 가장 쉽게 적용할 수 있는 방법이다.
  그런데 다른 상황에서 유용하게 쓰이는 변형된 방법도 있다.
  코드 작성 없이 순전히 다른 리팩터링들을 연달아 수행하여 새 메서드를 만들어내는 방법이다.

  > 호출자

  ```jsx
  const low = aRoom.daysTempRange.low;
  const high = aRoom.daysTempRange.high;
  if (aPlan.withinRange(low, high))
    alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```

  이번에는 코드를 재정렬해서 기존 코드 일부를 메서드로 추출하는 방식으로 새 메서드를 만들려 한다.
  지금의 호출자 코드는 이에 적합하지 않지만 변수 추출하기를 몇 번 적용하여 원하는 모습으로 둔갑한다.
  먼저, 조건문에서 기존 메서드를 호출하는 코드들을 해방시켜보자.

  ```jsx
  const low = aRoom.daysTempRange.low;
  const high = aRoom.daysTempRange.high;
  const isWithinRange = aPlan.withinRange(low, high);
  if (!isWithinRange) alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```

  그런 다음 입력 매개변수를 추출한다.

  ```jsx
  const tempRange = aRoom.daysTempRange;
  const low = tempRange.low;
  const high = tempRange.high;
  const isWithinRange = aPlan.withinRange(low, high);
  if (!isWithinRange) alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```

  다 끝났으면 함수 추출하기로 새 메서드를 만들 수 있다.

  ```jsx
  function xxNEWwithinRange(aPlan, tempRange) {
  	const low = tempRange.low;
  	const high = tempRange.high;
  	const isWithinRange = aPlan.withinRange(low, high));
  	return isWithinRange;
  }

  const tempRange = aRoom.daysTempRange;
  const isWithinRange = xxNEWwithinRange(aPlan, tempRange);
  if(isWithinRange)
  	alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```

  원래 메서드는 다른 컨텍스트(HeatingPlan 클래스 안)에 있으니 함수 옮기기를 수행해야 한다.

  ```jsx
  class HeatingPlan {
  	xxNEWwithinRange(aPlan, tempRange) {
  		const low = tempRange.low;
  		const high = tempRange.high;
  		const isWithinRange = aPlan.withinRange(low, high));
  		return isWithinRange;
  	}
  }

  const tempRange = aRoom.daysTempRange;
  const isWithinRange = xxNEWwithinRange(aPlan, tempRange);
  if(isWithinRange)
  	alerts.push("방 온도가 지정 범위를 벗어났습니다.");
  ```

  그 다음은 앞 예시와 같다. 다른 호출자들을 수정한 다음 옛 메서드를 새 메서드 안으로 인라인한다.
  추출한 변수들도 인라인해주면 새로 추출한 메서드를 깔끔히 분리할 수 있다.
  이 변형 방식은 순전히 리팩터링으로만 구성되므로 추출과 인라인 리팩터링을 지원하는 편집기를 이용하면 아주 손쉽게 수행할 수 있다.

---

## 매개변수를 질의 함수로 바꾸기(Replace Parameter with Query)

- 반대 리팩터링 : 질의 함수를 매개변수로 바꾸기(11.6)

```jsx
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
  // 연휴 계산
}
```

```jsx
availableVacation(anEmployee);

function availableVacation(anEmployee) {
  const grade = amEmployee.grade;
}
```

### 배경

매개변수 목록은 함수의 변동 요인을 모아놓은 곳이다. 즉, 함수의 동작에 변화를 줄 수 있는 일차적인 수단이다.

다른 코드와 마찬가지로 이 목록에서도 중복은 피하는 게 좋으며 짧을수록 이해하기 쉽다.

피호출 함수가 스스로 ‘쉽게’ 결정할 수 있는 값을 매개변수로 건네는 것도 일종의 중복이다.

이런 함수를 호출할 때 매개변수의 값은 호출자가 정하게 되는데, 이 결정은 사실 하지 않아도 되었을 일이니 의미 없는 코드만 복잡해질 뿐이다.

이번 리팩터링의 한계는 ‘쉽게’라는 단어에 있다. 해당 매개변수를 제거하면 값을 결정하는 책임 주체가 달라진다.

매개변수가 있다면 결정 주체가 호출자가 되고, 매개변수가 없다면 피호출 함수가 된다.

나는 습관적으로 호출하는 쪽을 간소하게 만든다. 즉, 책임 소재를 피호출 함수로 옮긴다는 뜻인데, 물론 피호출 함수가 그 역할을 수행하기에 적합할 때만 그렇게 한다.

매개변수를 질의 함수로 바꾸지 말아야 할 상황도 있다. 가장 흔한 예는 매개변수를 제거하면 피호출 함수에 원치 않는 의존성이 생길 때다.

즉, 해당 함수가 알지 못했으면 하는 프로그램 요소에 접근해야 하는 상황을 만들 때다.

새로운 의존성이 생기거나 제거하고 싶은 기존 의존성을 강화하는 경우라 할 수 있다.

이런 상황은 주로 함수 본문에서 문제의 외부 함수를 호출해야 하거나 나중에 함수 밖으로 빼내길 원하는 수용 객체(receiver object)에 담긴 데이터를 사용해야 할 때 일어난다.

제거하려는 매개변수의 값을 다른 매개변수에 질의해서 얻을 수 있다면 안심하고 질의 함수로 바꿀 수 있다.

다른 매개변수에서 얻을 수 있는 값을 별도 매개변수로 전달하는 것은 아무 의미가 없다.

주의사항이 하나 있다. 대상 함수가 참조 투명(referential transparency)해야 한다는 것이다.

참조 투명이란 ‘함수에 똑같은 값을 건네 호출하면 항상 똑같이 동작한다’는 뜻이다.

이런 함수는 동작을 예측하고 테스트하기가 훨씬 쉬우니 이 특성이 사라지지 않도록 주의하자.

따라서 매개변수를 없애는 대신 가변 전역 변수를 이용하는 일은 하면 안된다.

---

### 절차

1. 필요하다면 대상 매개변수의 값을 계산하는 코드를 별도 함수로 추출해놓는다.
2. 함수 본문에서 대상 매개변수로의 참조를 모두 찾아서 그 매개변수의 값을 만들어주는 표현식을 참조하도록 바꾼다.
3. 함수 선언 바꾸기로 대상 매개변수를 없앤다.

---

### 예시

다른 리팩터링을 수행한 뒤 특정 매개변수가 더는 필요 없어졌을 때가 있는데, 바로 이번 리팩터링을 적용하는 가장 흔한 사례다.

```jsx
class Order {
	get finalPrice() {
		const basePrice = this.quantity * this.itemPrice;
		let discountLevel;
		if (this.quantity > 100) discountLevel = 2;
		else discountLevel = 1;
		return this.discountedPrice(basePrice, discountLevel);
	}

	discountedPrice(basePrice, discountLevel) {
		swich(discountLevel) {
			case 1: return basePrice * 0.95;
			case 2: return basePrice * 0.9;
		}
	}
}
```

함수를 간소화하다 보면 임시 변수를 질의 함수로 바꾸기(7.4)를 적용할 때가 많다. 이를 앞의 finalPrice() 함수에 적용하면 다음처럼 변한다.

```jsx
class Order {
  get finalPrice() {
    const basePrice = this.quantity * this.itemPrice;
    return this.discountedPrice(basePrice, this.discountLevel);
  }

  get discountLevel() {
    return this.quantity > 100 ? 2 : 1;
  }
}
```

그 결과로 discountedLevel() 함수에 discountLevel()의 반환 값을 건넬 이유가 사라졌다.

필요할 때 직접 호출하면 되기 때문이다.

[2] 이 매개변수를 참조하는 코드를 모두 함수 호출로 바꿔보자.

```jsx
class Order {
	discountedPrice(basePrice, discountLevel) {
		swich(this.discountLevel) {
			case 1: return basePrice * 0.95;
			case 2: return basePrice * 0.9;
		}
	}
}
```

[3]이제 함수 선언 바꾸기로 이 매개변수를 없앨 수 있다.

---

## 질의 함수를 매개변수로 바꾸기(Replace Query with Parameter)

- 반대 리팩터링 : 매개변수를 질의 함수로 바꾸기(11.5)

```jsx
targetTemparature(aPlan);

function targetTemparature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
  // 생략
}
```

```jsx
targetTemparature(aPlan, thermostat.currentTemperature);

function targetTemparature(aPlan, currentTemperature) {
  // 생략
}
```

### 배경

코드를 읽다 보면 함수 안에 두기엔 거북한 참조를 발견할 때가 있다.

전역 변수를 참조하거나(같은 모듈에 안에서라도) 제거하길 원하는 원소를 참조하는 경우가 여기 속한다.

이 문제는 해당 참조를 매개변수로 바꿔 해결할 수 있다. 참조를 풀어내는 책임을 호출자로 옮기는 것이다.

이런 상황 대부분은 코드의 의존 관계를 바꾸려 할 때 벌어진다. 예컨대 대상 함수가 더 이상 매개변수화하려는 특정 원소에 의존하길 원치 않을 때 일어난다.

이때 두 극단 사이에서의 적절한 균형을 찾아야 한다.

한쪽 끝은 모든 것을 배개변수로 바꿔 아주 길고 반복적인 매개변수 목록을 만드는 것이고,

다른 쪽 끝은 함수들끼리 많은 것을 공유하여 수많은 결합을 만들어내는 것이다.

대다수 까다로운 결정이 그렇듯, 이 역시 한 시점에 내린 결정이 영원히 옳다고 할 수는 없는 문제다.

따라서 프로그램을 더 잘 이해하게 됐을 때 더 나은 쪽으로 개선하기 쉽게 설계해두는 게 중요하다.

똑같은 값을 건네면 매번 똑같은 결과를 내는 함수는 다루기 쉽다. 이런 성질을 ‘참조 투명성’이라 한다.

참조 투명하지 않은 원소에 접근하는 모든 함수는 참조 투명성을 잃게 되는데, 이 문제는 해당 원소를 매개변수로 바꾸면 해결된다.

책임이 호출자로 옮겨진다는 점을 고려해야 하지만, 모듈을 참조 투명하게 만들어 얻는 장점은 대체로 아주 크다.

그래서 모듈을 개발할 때 순수 함수들을 따로 구분하고, 프로그램의 입출력과 기타 가변 원소들을 다루는 로직으로 순수 함수들의 겉을 감싸는 패턴을 많이 활용한다.

그리고 이번 리팩터링을 활용하면 프로그램의 일부를 순수 함수로 바꿀 수 있으며, 결과적으로 그 부분은 테스트하거나 다루기가 쉬워진다.

이 리팩터링에도 단점은 있다. 질의 함수를 매개변수로 바꾸면 어떤 값을 제공할지를 호출자가 알아내야 한다.

결국 호출자가 복잡해지는데, 이왕이면 고객(호출자)의 삶이 단순해지도록 설계하자는 내 평소 지론과 배치된다.

이 문제는 결국 책임 소재를 프로그램의 어디에 배정하느냐의 문제로 귀결된다.

답을 찾기가 쉽지 않으며 항상 정답이 있는 것도 아니다.

프로젝트를 진행하면서 균형점이 이리저리 옮겨질 수 있으니 이 리팩터링(과 그 반대 리팩터링)과는 아주 친해져야 한다.

---

### 절차

1. 변수 추출하기로 질의 코드를 함수 본문의 나머지 코드와 분리한다.
2. 함수 본문 중 해당 질의를 호출하지 않는 코드들을 별도 함수로 추출한다.

   → 이 함수의 이름은 나중에 수정해야 하니 검색하기 쉬운 이름으로 짓는다.

3. 방금 만든 변수를 인라인하여 제거한다.
4. 원래 함수도 인라인한다.
5. 새 함수의 이름을 원래 함수의 이름으로 고쳐준다.

---

### 예시

간단하지만 거추장스러운 코드를 준비했다. 실내온도 제어 시스템이다.

사용자는 온도조절기(thermostat)로 온도를 설정할 수 있지만, 목표 온도는 난방 계획에서 정한 범위에서만 선택할 수 있다.

```jsx
class HeatingPlan {
  get targetTemperature() {
    if (thermostat.selectedTemperature > this.max) return this.max;
    else if (thermostat.selectedTemperature < this.min) return this.min;
    else return thermostat.selectedTemperature;
  }
}

if (thePlan.targetTemperature > thermostat.currentTemperature) setToHeat();
else if (thePlan.targetTemperature < thermostat.currentTemerature) setToCool();
else setOff();
```

내가 이 시스템의 사용자라면 내 바람보다 난방 계획이 우선한다는 사실에 짜증 날 것이다.

하지만 개발자로서의 나는 targetTemperature() 메서드가 전역 객체인 thermostat에 의존한다는 데 더 신경이 쓰인다.

그러니 이 전역 객체는 건네는 질의 메서드를 매개변수로 옮겨서 의존성을 끊어보자.

[1] 첫 번째로 할 일은 변수 추출하기를 이용해서 이 메서드에서 사용할 매개변수를 준비하는 것이다.

```jsx
class HeatingPlan {
  get targetTemperature() {
    const selectedTemperature = thermostat.selectedTemperature;
    if (selectedTemperature > this.max) return this.max;
    else if (selectedTemperature < this.min) return this.min;
    else return selectedTemperature;
  }
}
```

[2] 이제 매개변수의 값을 구하는 코드를 제외한 나머지를 메서드로 추출하기가 한결 수월해졌다.

```jsx
class HeatingPlan {
  get targetTemperature() {
    const selectedTemperature = thermostat.selectedTemperature;
    return this.xxNEWtargetTemperature(selectedTemperature);
  }

  xxNEWtargetTemperature(selectedTemperature) {
    if (selectedTemperature > this.max) return this.max;
    else if (selectedTemperature < this.min) return this.min;
    else return selectedTemperature;
  }
}
```

[3] 다음으로 방금 추출한 변수를 인라인하면 원래 메서드에는 단순한 호출만 남게 된다.

```jsx
class HeatingPlan {
  get targetTemperature() {
    return this.xxNEWtargetTemperature(thermostat.selectedTemperature);
  }

  xxNEWtargetTemperature(selectedTemperature) {
    if (selectedTemperature > this.max) return this.max;
    else if (selectedTemperature < this.min) return this.min;
    else return selectedTemperature;
  }
}
```

[4] 이어서 이 메서드까지 인라인한다.

```jsx
if(thePlan.xxNEWtargetTemperature(thermostat.selectedTemperature) >
	 thermostat.currentTemperature)
 setToHeat();
else if(thePlan.xxNEWtargetTemperature(thermostat.selectedTemperature <
	 thermostat.currentTemerature)
 setToCool();
else
 setOff();
```

[5] 이제 새 메서드의 이름을 원래 메서드의 이름으로 바꿀 차례다. 앞서 이 메서드의 이름을 검색하기 쉽게 만들어놓은 덕에 쉽게 바꿀 수 있다.

```jsx
class HeatingPlan {
	targetTemperature(selectedTemperature) {
		if(selectedTemperature > this.max) return this.max;
		else if(selectedTemperature < this.min) return this.min;
		else return selectedTemperature;
	}
}

if(thePlan.targetTemperature(thermostat.selectedTemperature) >
	 thermostat.currentTemperature)
 setToHeat();
else if(thePlan.targetTemperature(thermostat.selectedTemperature <
	 thermostat.currentTemerature)
 setToCool();
else
 setOff();
```

이 리팩터링을 수행하면 호출하는 쪽 코드는 전보다 다루기 어려워지는 게 보통이다.

‘의존성을 모듈 바깥으로 밀어낸다’함은 그 의존성을 처리하는 책임을 호출자에게 치운다는 뜻이기 때문이다.

결합도를 낮춘 효과에 대한 반대급부인 셈이다.

그런데 이 리팩터링으로 얻은 것이 온도조절기 객체와의 결합을 제거한 것만은 아니다.

HeatingPlan 클래스는 불변이 되었다. 모든 필드가 생성자에서 설정되며, 필드를 변경할 수 있는 메서드는 없다.

(이 클래스의 다른 코드를 생략한 것은 독자 여러분이 굳이 다 살펴보지 않도록 배려한 것이니 그냥 나를 믿어주기 바란다)

난방 계획도 불변이므로 온도조절기 참조를 메서드 밖으로 옮긴 것이 targetTemperature()를 참조 투명하게 만들어준다.

따라서 같은 객체의 targetTemperature()에 같은 인수를 넘겨 호출하면 언제나 똑같은 결과를 돌려줄 것이다.

난방 계획의 메서드도 모두 참조 투명하다면 이 클래스는 테서트하고 다루기가 훨씬 쉬워졌을 것이다.

> **자바스크립트와 불변 클래스**

자바스크립트의 클래스 모델에서는 객체 안의 데이터를 직접 얻어낼 방법이 항상 존재하기 때문에 불변 클래스임을 보장하는 수단이 없다는 문제가 있다.

하지만 클래스를 불변으로 설계했음을 알리고 그렇게 사용하라고 제안하는 것만으로 충분한 값어치를 할 때가 많다.

클래스에 불변 성격을 부여하는 건 훌륭한 전략이며, 질의 함수를 매개변수로 바꾸기 리팩터링은 이 전략을 실행하는 데 큰 도움이 된다.

---

## 세터 제거하기(Remove Setting Method)

```jsx
class Person {
	get name() { ... }
	set name(aString) { ... }
}
```

```jsx
class Person {
	get name() { ... }
}
```

### 배경

세터 메서드가 있다고 함은 필드가 수정될 수 있다는 뜻이다.

객체 생성 후에는 수정되지 않길 원하는 필드라면 세터를 제공하지 않았을 것이다 (그래서 그 필드를 불변으로 만들었을 것이다)

그러면 해당 필드는 오직 생성자에서만 설정되며, 수정하지 않겠다는 의도가 명명백백해지고, 변경될 가능성이 봉쇄된다.

(자바의 리플렉션처럼 언어에 따라 변경할 수 있는 특별한 메커니즘을 제공하기도 한다)

세터 제거하기 리팩터링이 필요한 상황은 주로 두 가지다. 첫째, 사람들이 무조건 접근자 메서드를 통해서만 필드를 다룰려 할 때다.

심지어 생성자 안에서도 말이다. 이러면 오직 생성자에서만 호출하는 세터가 생겨나곤 한다.

하지만 나라면 세터를 제거해서 객체가 생성된 후에는 값이 바뀌면 안된다는 뜻을 분명히 할 것이다.

두 번째 상황은 클라이언트에서 생성 스크립트를 사용해 객체를 생성할 때다.

생성 스크립트란 생성자를 호출한 후 일련의 세터를 호출하여 객체를 완성하는 형태의 코드를 말한다.

필드 일부(혹은 전체)는 변경되지 않으리라 기대한다. 즉, 해당 세터들은 처음 생성할 때만 호출되리라 가정한다.

이런 경우에도 세터들을 제거하여 의도를 더 정확하게 전달하는 게 좋다.

---

### 절차

1. 설정해야 할 값을 생성자에서 받지 않는다면 그 값을 받을 매개변수를 생성자에 추가한다(함수 선언 바꾸기)

   그런 다음 생성자 안에서 적절한 세터를 호출한다.

   → 세터 여러 개를 제거하려면 해당 값을 모두 한꺼번에 생성자에 추가한다. 그러면 이후 과정이 간소해진다.

2. 생성자 밖에서 세터를 호출하는 곳을 찾아 제거하고, 대신 새로운 생성자를 사용하도록 한다. 하나 수정할 때마다 테스트한다.

   → 갱신하려는 대상이 공유 참조 객체라서 새로운 객체를 생성하는 방식으로는 세터 호출을 대체할 수 없다면 이 리팩터링을 취소한다.

3. 세터 메서드를 인라인한다. 가능하다면 해당 필드를 불변으로 만든다.
4. 테스트한다.

---

### 예시

간단한 사람(Person) 클래스를 준비했다.

```jsx
class Person {
  get name() {
    return this.name;
  }
  set name(arg) {
    this.name = arg;
  }
  get id() {
    return this.id;
  }
  set id(arg) {
    this.id = arg;
  }
}
```

그리고 다음 코드로 사람 객체를 하나 생성한다.

```jsx
const martin = new Person();
martin.name = "마틴";
martin.id = "1234";
```

사람 속성 중 이름은 객체를 생성한 뒤라도 변경될 수 있겠지만 id는 그러면 안 된다. 이 의도를 명확히 알리기 위해 ID 세터를 없애보자.

[1] 최초 한 번은 ID를 설정할 수 있어야 하므로 함수 선언 바꾸기로 생성자에서 ID를 받도록 한다.

```jsx
class Person {
  constructor(id) {
    this.id = id;
  }
}
```

[2] 그런 다음 생성 스크립트가 이 생성자를 통해 ID를 설정하게끔 수정한다.

```jsx
cost martin = new Person("1234");
martin.name = "마틴";
// martin.id = "1234";
```

이 작업을 사람 객체를 생성하는 모든 곳에서 수행하며, 하나 수정할 때마다 테스트한다.

[3] 모두 수정했다면 세터 메서드를 인라인한다.

```jsx
class Person {
  constructor(id) {
    this.id = id;
  }
  get name() {
    return this.name;
  }
  set name(arg) {
    this.name = arg;
  }
  get id() {
    return this.id;
  }
  //	set id(arg) { this.id = arg; }
}
```

---

## 생성자를 팩터리 함수로 바꾸기(Replace Constructor with Factory Function)

```jsx
leadEngineer = new Employee(document.leadEngineer, "E");
```

```jsx
leadEngineer = createEngineer(document.leadEngineer);
```

### 배경

많은 객체 지향 언어에서 제공하는 생성자는 객체를 초기화하는 특별한 용도의 함수다.

실제로 새로운 객체를 생성할 때면 주로 생성자를 호출한다.

하지만 생성자에는 일반 함수에는 없는 이상한 제약이 따라붙기도 한다.

가령 자바 생성자는 반드시 그 생성자를 정의한 클래스의 인스턴스를 반환해야 한다.

서브클래스의 인스턴스나 프록시를 반환할 수는 없다.

생성자의 이름도 고정되어, 기본 이름보다 더 적절한 이름이 있어도 사용할 수 없다.

생성자를 호출하려면 특별한 연산자(많은 언어에서 new를 쓴다)를 사용해야 해서 일반 함수가 오길 기대하는 자리에는 쓰기 어렵다.

팩터리 함수에는 이런 제약이 없다. 팩터리 함수를 구현하는 괒어에서 생성자를 호출할 수는 있지만, 원한다면 다른 무언가로 대체할 수 있다.

---

### 절차

1. 팩터리 함수를 만든다. 팩터리 함수의 본문에서는 원래 생성자를 호출한다.
2. 생성자를 호출하던 코드를 팩터리 함수 호출로 바꾼다.
3. 하나씩 바꿀때마다 테스트한다.
4. 생성자의 가시 범위가 최소가 되도록 제한한다.

---

### 예시

직원(employee) 유형을 다루는, 간단하지만 이상한 예를 살펴보자.

```jsx
class Employee {
  constructor(name, typeCode) {
    this.name = name;
    this.typeCode = typeCode;
  }

  get name() {
    return this.name;
  }
  get type() {
    return Employee.legalTypeCodes[this.typeCode];
  }
  static get legalTypeCodes() {
    return { E: "Engineer", M: "Manager", S: "Salesperson" };
  }
}
```

다음은 이 클래스를 사용하는 코드다.

```jsx
candidate = new Employee(document.name, document.empType);

const leadEngineer = new Employee(document.leadEngineer, "E");
```

[1] 첫 번째로 할 일은 팩터리 함수 만들기다. 팩터리 본문은 단순히 생성자에 위임하는 방식으로 구현한다.

```jsx
function createEmployee(name, typeCode) {
  return new Employee(name, typeCode);
}
```

[2]그런 다음 생성자를 호출하는 곳을 찾아 수정한다. 한 번에 하나씩, 생성자 대신 팩터리 함수를 사용하게 바꾼다.

첫 번째 코드는 쉽게 바꿀 수 있다.

```jsx
candidate = createEmployee(document.name, document.empType);
```

두 번째 코드를 바꾸려면 다음과 같은 모습의 새로운 팩터리 함수를 사용할 수도 있을 것이다.

```jsx
const leadEngineer = createEmployee(document.leadEngineer, "E");
```

하지만 이건 내가 권장하는 코드 스타일이 아니다 (함수에 문자열 리터럴을 건네는 건 악취로 봐야 한다)

그 대신 직원 유형을 팩터리 함수의 이름에 녹이는 방식을 권한다.

```jsx
function createEngineer(name) {
  return new Employee(name, "E");
}

const leadEngineer = createEngineer(document.leadEngineer);
```

---

## 함수를 명령으로 바꾸기(Replace Function with Command)

- 반대 리팩터링 : 명령을 함수로 바꾸기(11.10)

```jsx
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  // 긴 코드 생략
}
```

```jsx
class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this.candidate = candidate;
    this.medicalExam = medicalExam;
    this.scoringGuide = scoringGuide;
  }

  execute() {
    this.result = 0;
    this.healthLevel = 0;
    // 긴 코드 생략
  }
}
```

### 배경

독립된 함수든 객체에 소속된 메서드는 프로그래밍의 기본적인 빌딩 블록 중 하나다.

그런데 함수를 그 함수만을 위한 객체 안으로 캡슐화하면 더 유용해지는 상황이 있다.

이런 객체를 가리켜 ‘명령 객체’ 혹은 단순히 ‘명령’(Command)이라 한다.

명령 객체 대부분은 메서드 하나로 구성되며, 이 메서드를 요청해 실행하는 것이 이 객체의 목적이다.

명령은 평범한 함수 메커니즘보다 훨씬 유연하게 함수를 제어하고 표현할 수 있다.

명령은 되돌리기(undo) 같은 보조 연산을 제공할 수 있으며, 수명주기를 더 정밀하게 제어하는 데 필요한 매개변수를 만들어주는 메서드도 제공할 수 있다.

상속과 훅(hock)을 이용해 사용자 맞춤형으로 만들 수도 있다.

객체는 지원하지만 일급 함수(first-class function)를 지원하지 않는 프로그래밍 언어를 사용할 때는 명령을 이용해 일급 함수의 기능 대부분을 흉내 낼 수 있다.

비슷하게, 중첩 함수를 지원하지 않는 언어에서도 메서드와 필드를 이용해 복잡한 함수를 잘게 쪼갤 수 있고, 이렇게 쪼갠 메서드들을 테스트와 디버깅에 직접 이용할 수 있다.

이처럼 명령을 사용해 얻는 이점이 많으므로 함수를 명령으로 리팩터링할 채비를 갖춰야 할 것이다.

하지만 유연성은 언제나 그렇듯 복잡성을 키우고 얻는 대가임을 잊지 말아야 한다.

그래서 일급 함수와 명령 중 선택해야 한다면, 나라면 95%는 일급 함수의 손을 들어준다.

내가 명령을 선택할 때는 명령보다 더 간단한 방식으로는 얻을 수 없는 기능이 필요할 때 뿐이다.

```
소프트웨어 개발 용어 중에는 여러 가지 의미로 사용되는 게 많은데, '명령'도 마찬가지다.
지금 맥락에서의 명령은 요청을 캡슐화한 객체로, 디자인 패턴 중 명령 패턴(command pattern)에서 말하는 명령과 같다.
이 책에서 이 의미의 명령을 이야기할 때는 먼저 '명령 객체'라고 한번 언급해준 후 '명령'으로 줄여 쓸 것이다.
한편 명령-질의 분리 원칙(command-query separation principle)에서도 명령이 등장한다.
이 원칙에서의 명령은 객체의 겉보기 상태를 변경하는 메서드를 가리킨다.
이 책에서는 이 의미의 명령을 이야기할 때는 명령이란 단어를 쓰지 않도록 노력할 것이다. 대신 '변경 함수'라 하겠다.
```

---

### 절차

1. 대상 함수의 기능을 옮길 빈 클래스를 만든다. 클래스 이름은 함수 이름에 기초해 짓는다.
2. 방금 생성한 빈 클래스로 함수를 옮긴다.

   → 리팩터링이 끝날 때까지는 원래 함수를 전달 함수 역할로 남겨두자.

   → 명령 관련 이름은 사용하는 프로그래밍 언어의 명명규칙을 따른다. 규칙이 딱히 없다면 ‘execute’나 ‘call’같이 명령의 실행 함수에 흔히 쓰이는 이름을 택하자.

3. 함수의 인수들 각각은 명령의 필드로 만들어 생성자를 통해 설정할지 고민해본다.

---

### 예시

자바스크립트는 허점이 많은 언어다. 하지만 함수를 일급으로 만든 선택은 아주 훌륭했다.

그래서 일급 함수를 지원하지 않는 언어에서라면 필요했을 일반적인 작업에는 굳이 명령을 만들어 해결할 이유가 없다.

하지만 명령을 사용하는 편이 나을 때가 없는 건 아니다.

예컨대 복잡한 함수를 잘게 쪼개서 이해하거나 수정하기 쉽게 만들고자 할 때가 있다.

그래서 사실 이 리팩터링의 가치를 잘 보여주려면 길고 복잡한 함수를 준비해야 한다.

하지만 그렇게 하면 책에 싣기에는 너무 길고 여러분이 읽기에도 불편할 것이므로 길이를 최소한으로 제한했다.

다음은 건강보험 애플리케이션에서 사용하는 점수 계산 함수다.

```jsx
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  let highMedicalRiskFlag = false;

  if (medicalExam.isSmoker) {
    healthLevel += 10;
    highMedicalRiskFlag = true;
  }
  let certificationGrade = "regular";
  if (scoringGuide.stateWithLowCertification(candidate.originState)) {
    certificationGrade = "low";
    result -= 5;
  }
  // 비슷한 코드가 이어짐
  result -= Math.max(healthLevel - 5, 0);
  return result;
}
```

[1] 시작은 빈 클래스를 만들고 [2] 이 함수를 그 클래스로 옮기는 일부터다.

```jsx
function score(candidate, medicalExam, scoringGuide) {
  return new Scorer().execute(candidate, medicalExam, scoringGuide);
}

class Scorer {
  execute(candidate, medicalExam, scoringGuide) {
    let result = 0;
    let healthLevel = 0;
    let highMedicalRiskFlag = false;

    if (medicalExam.isSmoker) {
      healthLevel += 10;
      highMedicalRiskFlag = true;
    }
    let certificationGrade = "regular";
    if (scoringGuide.stateWithLowCertification(candidate.originState)) {
      certificationGrade = "low";
      result -= 5;
    }
    // 비슷한 코드가 이어짐
    result -= Math.max(healthLevel - 5, 0);
    return result;
  }
}
```

주로 나는 명령이 받는 인수들을 생성자로 옮겨서 execute() 메서드는 매개변수를 받지 않게 하는 편이다.

지금 예 정도의 단순한 시나리오에서는 큰 차이가 없지만, 명령의 수명주기나 사용자 정의 기능 등을 지원해야 해서 매개변수가 복잡할 때는 아주 편리하다.

예컨대 이 방식이라면 매개변수 목록이 서로 다른 여러 형태의 명령들을 하나의 실행 대기열(queue)을 통해 전달할 수도 있다.

매개변수 옮기기는 한 번에 하나씩 수행하자.

```jsx
function score(candidate, medicalExam, scoringGuide) {
  return new Scorer(candidate, medicalExam, scoringGuide).execute();
}

class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this.candidate = candidate;
    this.medicalExam = medicalExam;
    this.scoringGuide = scoringGuide;
  }

  execute() {
    let result = 0;
    let healthLevel = 0;
    let highMedicalRiskFlag = false;

    if (this.medicalExam.isSmoker) {
      healthLevel += 10;
      highMedicalRiskFlag = true;
    }
    let certificationGrade = "regular";
    if (
      this.scoringGuide.stateWithLowCertification(this.candidate.originState)
    ) {
      certificationGrade = "low";
      result -= 5;
    }
    // 비슷한 코드가 이어짐
    result -= Math.max(healthLevel - 5, 0);
    return result;
  }
}
```

이상으로 함수를 명령으로 바꿔봤다. 하지만 이 리팩터링의 본래 목적은 복잡한 함수를 잘게 나누는 것이다.

이 목적을 이루기 위한 단계들을 개략적으로 설명해보겠다.

먼저 모든 지역 변수를 필드로 바꿔야 한다. 역시 한 번에 하나씩 진행한다.

```jsx
function score(candidate, medicalExam, scoringGuide) {
  return new Scorer(candidate, medicalExam, scoringGuide).execute();
}

class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this.candidate = candidate;
    this.medicalExam = medicalExam;
    this.scoringGuide = scoringGuide;
  }

  execute() {
    this.result = 0;
    this.healthLevel = 0;
    this.highMedicalRiskFlag = false;

    this.scoreSmoking();
    this.certificationGrade = "regular";
    if (
      this.scoringGuide.stateWithLowCertification(this.candidate.originState)
    ) {
      this.certificationGrade = "low";
      this.result -= 5;
    }
    // 비슷한 코드가 이어짐
    this.result -= Math.max(this.healthLevel - 5, 0);
    return this.result;
  }

  scoreSmoking() {
    if (this.medicalExam.isSmoker) {
      this.healthLevel += 10;
      this.highMedicalRiskFlag = true;
    }
  }
}
```

이제 명령을 중첩 함수처럼 다룰 수 있다. 사실 자바스크립트에서라면 중첩 함수는 명령의 합리적인 대안이 될 수 있다.

그래도 나는 여전히 명령을 사용한다. 내가 명령에 더 익숙하기도 하거니와, 명령을 사용하면 (execute 외의) 서브함수들을 테스트와 디버깅에 활용할 수 있기 때문이다.

---

## 명령을 함수로 바꾸기(Replace Command with Function)

- 반대 리팩터링 : 함수를 명령으로 바꾸기(11.9)

```jsx
class ChargeCalculator {
  constructor(customer, usage) {
    this.customer = customer;
    this.usage = usage;
  }
  execute() {
    return this.customer.rate * this.usage;
  }
}
```

```jsx
function charge(customer, usage) {
  return customer.rate * usage;
}
```

### 배경

명령 객체는 복잡한 연산을 다룰 수 있는 강력한 메커니즘을 제공한다.

구체적으로는, 큰 연산 하나를 여러 개의 작은 메서드로 쪼개고 필드를 이용해 쪼개진 메서드들끼리 정보를 공유할 수 있다.

또한 어떤 메서드를 호출하냐에 따라 다른 효과를 줄 수 있고 각 단계를 거치며 데이터를 조금씩 완성해갈 수도 있다.

명령의 이런 능력은 공짜가 아니다. 명령은 그저 함수를 하나 호출해 정해진 일을 수행하는 용도로 주로 쓰인다.

이런 상황이고 로직이 크게 복잡하지 않다면 명령 객체는 장점보다 단점이 크니 평범한 함수로 바꿔주는 게 낫다.

---

### 절차

1. 명령을 생성하는 코드와 명령의 실행 메서드를 호출하는 코드를 함께 함수로 추출한다.

   → 이 함수가 바로 명령을 대체할 함수다.

2. 명령의 실행 함수가 호출하는 보조 메서드들 각각을 인라인한다.

   → 보조 메서드가 값을 반환한다면 함수 인라인에 앞서 변수 추출하기를 적용한다.

3. 함수 선언 바꾸기를 적용하여 생성자의 매개변수 모두를 명령의 실행 메서드로 옮긴다.
4. 명령의 실행 메서드에서 참조하는 필드들 대신 대응하는 매개변수를 사용하게끔 바꾼다. 하나씩 수정할 때마다 테스트한다.
5. 생성자 호출과 명령의 실행 메서드 호출을 호출자(대체 함수) 안으로 인라인한다.
6. 테스트한다.
7. 죽은 코드 제거하기로 명령 클래스를 없앤다.

---

### 예시

```jsx
class ChargeCalculator {
  constructor(customer, usage, provider) {
    this.customer = customer;
    this.usage = usage;
    this.provider = provider;
  }
  get baseCharge() {
    return this.customer.baseRage * this.usage;
  }
  get charge() {
    return this.baseCharge + this.provider.connectionCharge;
  }
}
```

다음은 호출하는 쪽의 코드다.

```jsx
monthCharge = new ChargeCalculator(customer, usage, provider).charge;
```

이 명령 클래스는 간단한 편이므로 함수로 대체하는 게 나아 보인다.

[1] 첫 번째로, 이 클래스를 생성하고 호출하는 코드를 함께 함수로 추출한다.

```jsx
monthCharge = charge(customer, usage, provider);

function charge(customer, usage, provider) {
  return new ChargeCalculator(customer, usage, provider).charge;
}
```

[2] 이때 보조 메서드들을 어떻게 다룰지 정해야 하는데, baseCharge()가 이러한 보조 메서드에 속한다.

값을 반환하는 메서드라면 먼저 반환할 값을 변수로 추출한다.

```jsx
class ChargeCalculator {
  get baseCharge() {
    return this.customer.baseRage * this.usage;
  }
  get charge() {
    const baseCharge = this.baseCharge;
    return baseCharge + this.provider.connectionCharge;
  }
}
```

그런 다음 보조 메서드를 인라인한다.

```jsx
class ChargeCalculator {
  get charge() {
    const baseCharge = this.customer.baseRage * this.usage;
    return baseCharge + this.provider.connectionCharge;
  }
}
```

[3] 이제 로직 전체가 한 메서드에서 이뤄지므로, 그다음으로는 생성자에 전달되는 모든 데이터를 주 메서드로 옮겨야 한다.

먼저 생성자가 받던 모든 매개변수를 charge() 메서드로 옮기기 위해 함수 선언 바꾸기를 적용한다.

```jsx
class ChargeCalculator {
	constructor(customer, usage, provider) {
		this.customer = customer;
		this.usage = usage;
		this.provider = provider;
	}
	get charge(customer, usage, provider) {
		const baseCharge = this.customer.baseRage * this.usage;
		return baseCharge + this.provider.connectionCharge;
	}
}

function charge(customer, usage, provider) {
	return new ChargeCalculator(customer, usage, provider)
											.charge(customer, usage, provider);
}
```

[4] 이제 charge()의 본문에서 필드 대신 건네받은 매개변수를 사용하도록 수정한다. 이번에도 한 번에 하나씩 진행한다.

```jsx
class ChargeCalculator {
	get charge(customer, usage, provider) {
		const baseCharge = customer.baseRage * usage;
		return baseCharge + provider.connectionCharge;
	}
}
```

[5] 다 됐다면 최상위 charge() 함수로 인라인할 수 있다. 이는 생성자와 메서드 호출을 함께 인라인하는 특별한 형태의 함수 인라인하기다.

```jsx
function charge(customer, usage, provider) {
  const baseCharge = customer.baseRage * usage;
  return baseCharge + provider.connectionCharge;
}
```

[7] 명령 클래스는 이제 죽은 코드가 되었으니 죽은 코드 제거하기로 영면에 들게 해준다.

---

## 수정된 값 반환하기(Return Modified Value)

```jsx
let totalAscent = 0;
calculateAscent();

function calculateAscent() {
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    totalAscent += verticalChange > 0 ? verticalChange : 0;
  }
}
```

```jsx
let totalAscent = calculateAscent();

function calculateAscent() {
  let result = 0;
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    result += verticalChange > 0 ? verticalChange : 0;
  }
  return result;
}
```

### 배경

데이터가 어떻게 수정되는지를 추적하는 일은 코드에서 이해하기 가장 어려운 부분 중 하나다.

특히 같은 데이터 블록을 읽고 수정하는 코드가 여러 곳이라면 데이터가 수정되는 흐름과 코드의 흐름을 일치시키기가 상당히 어렵다.

그래서 데이터가 수정된다면 그 사실을 명확히 알려주어서, 어느 함수가 무슨 일을 하는지 쉽게 알 수 있게 하는 일이 대단히 중요하다.

데이터가 수정됨을 알려주는 좋은 방법이 있다. 변수를 갱신하는 함수라면 수정된 값을 반환하여 호출자가 그 값을 변수에 담아두도록 하는 것이다.

이 방식으로 코딩하면 호출자 코드를 읽을 때 변수가 갱신될 것임을 분명히 인지하게 된다.

해당 변수의 값을 단 한번만 정하면 될 때 특히 유용하다.

이 리팩터링은 값 하나를 계산한다는 분명한 목적이 있는 함수들에 가장 효과적이고, 반대로 값 여러 개를 갱신하는 함수에는 효과적이지 않다.

한편, 함수 옮기기의 준비 작업으로 적용하기에 좋은 리팩터링이다.

---

### 절차

1. 함수가 수정된 값을 반환하게 하여 호출자가 그 값을 자신의 변수에 저장하게 한다.
2. 테스트한다.
3. 피호출 함수 안에 반환할 값을 가리키는 새로운 변수를 선언한다.

   → 이 작업이 의도대로 이뤄졌는지 검사하고 싶다면 호출자에서 초깃값을 수정해보자. 제대로 처리했다면 수정된 값이 무시된다.

4. 테스트한다.
5. 계산이 선언과 동시에 이뤄지도록 통합한다.(즉, 선언 시점에 계산 로직을 바로 실행해 대입한다.)

   → 프로그래밍 언어에서 지원한다면 이 변수를 불변으로 지정하자.

6. 테스트한다.
7. 피호출 함수의 변수 이름을 새 역할에 어울리도록 바꿔준다.
8. 테스트한다.

---

### 예시

여기 GPS 위치 목록으로 다양한 계산을 수행하는 코드가 있다.

```jsx
let totalAscent = 0;
let totalTime = 0;
let totalDistance = 0;
calculateAscent();
calculateTime();
calculateDistance();
const pace = totalTime / 60 / totalDistance;
```

이번 리팩터링에서는 고도 상승분(ascent) 계산만을 고려할 것이다.

```jsx
function calculateAscent() {
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    totalAscent += verticalChange > 0 ? verticalChange : 0;
  }
}
```

이 코드에서는 calculateAscent() 안에서 totalAscent가 갱신된다는 사실이 드러나지 않으므로

calculdateAscent()와 외부 환경이 어덯게 연결돼 있는지가 숨겨진다. 갱신 사실을 밖으로 알려보자.

[1] 먼저 totalAscent 값을 반환하고, 호출한 곳에서 변수에 대입하게 고친다.

```jsx
let totalAscent = 0;
let totalTime = 0;
let totalDistance = 0;
totalAscent = calculateAscent();
calculateTime();
calculateDistance();
const pace = totalTime / 60 / totalDistance;

function calculateAscent() {
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    totalAscent += verticalChange > 0 ? verticalChange : 0;
  }
  return totalAscent;
}
```

[3] 그런 다음 calculateAscent() 안에 반환할 값을 담을 변수인 totalAscent를 선언한다.

그런데 이 결과 부모 코드에 있는 똑같은 이름의 변수가 가려진다.

```jsx
function calculateAscent() {
  let totalAscent = 0;
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    totalAscent += verticalChange > 0 ? verticalChange : 0;
  }
  return totalAscent;
}
```

[7] 이 문제를 피하기 위해 변수의 이름을 일반적인 명명규칙에 맞게 수정한다.

```jsx
function calculateAscent() {
  let result = 0;
  for (let i = 1; i < points.length; i++) {
    const verticalChange = points[i].elevation - points[i - 1].elevation;
    result += verticalChange > 0 ? verticalChange : 0;
  }
  return result;
}
```

[5] 그런 다음 이 계산이 변수 선언과 동시에 수행하도록 하고, 변수에 const를 붙여서 불변으로 만든다.

```jsx
const totalAscent = calculateAscent();
let totalTime = 0;
let totalDistance = 0;
calculateTime();
calculateDistance();
```

같은 과정을 다른 함수들에도 반복해 적용해주면 호출하는 쪽 코드가 다음처럼 바뀔 것이다.

```jsx
const totalAscent = calculateAscent();
let totalTime = calculateTime();
let totalDistance = calculateDistance();
```

---

## 오류 코드를 예외로 바꾸기(Replace Error Code with Exception)

```jsx
if (data) return new ShippingRules(data);
else return -23;
```

```jsx
if (data) return new ShippingRules(data);
else throw new OrderProcessingError(-23);
```

### 배경

내가 프로그래밍을 시작할 당시엔 오류 코드(error code)를 사용하는 게 보편적이었다.

함수를 호출하면 언제든 오류가 반환될 수 있었고, 그래서 오류 코드 검사를 빼먹으면 안 됐다.

오류 코드를 검사해서 발생한 오류를 직접 처리하거나 다른 누군가가 처리해주길 기대하며 콜스택 위로 던져보냈다.

예외는 프로그래밍 언어에서 제공하는 독립적인 오류 처리 메커니즘이다. 오류가 발견되면 예외를 던진다.

그러면 적절한 예외 핸들러를 찾을 때까지 콜스택을 타고 위로 전파된다(핸들러를 찾지 못하면 보통은 단순할 정도로 극단적인 기본 동작이 수행된다)

예외를 사용하면 오류 코드를 일일이 검사하거나 오류를 식별해 콜스택 위로 던지는 일을 신경 쓰지 않아도 된다.

예외에는 독자적인 흐름이 있어서 프로그램의 나머지에서는 오류 발생에 따른 복잡한 상황에 대처하는 코드를 작성하거나 읽을 일이 없게 해준다.

예외는 정교한 메커니즘이지만 대다수의 다른 정교한 메커니즘과 같이 정확하게 사용할 때만 최고의 효과를 낸다.

예외는 정확히 예상 밖의 동작일 때만 쓰여야 한다. 달리 말하면 프로그램의 정상 동작 범주에 들지 않는 오류를 나타낼 때만 쓰여야 한다.

괜찮은 경험 법칙이 하나 있다. 예외를 던지는 코드를 프로그램 종료 코드로 바꿔도 프로그램이 여전히 정상 동작할지를 따져보는 것이다.

정상 동작하지 않을 것 같다면 예외를 사용하지 말라는 신호다. 예외 대신 오류를 검출하여 프로그램을 정상 흐름으로 되돌리게끔 처리해야 한다.

---

### 절차

1. 콜스택 상위에 해당 예외를 처리할 예외 핸들러를 작성한다.

   → 이 핸들러는 처음에는 모든 예외를 다시 던지게 해둔다.

   → 적절한 처리를 해주는 핸들러가 이미 있다면 지금의 콜스택도 처리할 수 있도록 확장한다.

2. 테스트한다.
3. 해당 오류 코드를 대체할 예외와 그 밖의 예외를 구분할 식별 방법을 찾는다.

   → 사용하는 프로그래밍 언어에 맞게 선택하면 된다. 대부분 언어에서는 서브클래스를 사용하면 될 것이다.

4. 정적 검사를 수행한다.
5. catch절을 수정하여 직접 처리할 수 있는 예외는 적절히 대처하고 그렇지 않은 예외는 다시 던진다.
6. 테스트한다.
7. 오류 코드를 반환하는 곳 모두에서 예외를 던지도록 수정한다. 하나씩 수정할 때마다 테스트한다.
8. 모두 수정했다면 그 오류 코드를 콜스택 위로 전달하는 코드를 모두 제거한다. 하나씩 수정할 때마다 테스트한다.

   → 먼저 오류 코드를 검사하는 부분을 함정(trap)으로 바꾼 다음, 함정에 걸려들지 않는지 테스트한 후 제거하는 전략을 권한다.

   함정에 걸려드는 곳이 있다면 오류 코드를 검사하는 코드가 아직 남아 있다는 뜻이다.

   함정을 무사히 피했다면 안심하고 본문을 정리하자(죽은 코드 제거하기)

---

### 예시

전역 테이블에서 배송지의 배송 규칙을 알아내는 코드를 생각해보자.

```jsx
function localShippingRules(country) {
  const data = countryData.shippingRules[country];
  if (data) return new ShippingRules(data);
  else return -23;
}
```

이 코드는 국가 정보(country)가 유효한지를 이 함수 호출 전에 다 검증했다고 가정한다.

따라서 이 함수에서 오류가 난다면 무언가 잘못됐음을 뜻한다. 다음과 같이 호출한 곳에서는 반환된 오류 코드를 검사하여 오류가 발견되면 위로 전파한다.

```jsx
function calculdateShippingCosts(anOrder) {
  // 관련 없는 코드
  const shippingRules = localShippingRules(anOrder.country);
  if (shippingRules < 0) return shippingRuels; // 오류 전파
  // 더 관련 없는 코드
}
```

더 윗단 함수는 오류를 낸 주문을 오류 목록(errorList)에 넣는다.

```jsx
const status = calculateShippingCosts(orderData);
if (status < 0) errorList.push({ order: orderData, errorCode: status });
```

여기서 가장 먼저 고려할 것은 이 오류가 ‘예상된 것이냐’다. localShippingRules()는 배송 규칙들이 countryData에 제대로 반영되어 있다고 가정해도 되나 ?

country 인수가 전역 데이터에 저장된 키들과 일치하는 곳에서 가져온 것인가, 아니면 앞서 검증을 받았나?

이 질문들의 답이 긍정적이면 (즉, 예상할 수 있는 정상 동작 범주에 든다면) 오류 코드를 예외로 바꾸는 이번 리팩터링을 적용할 준비가 된 것이다.

[1] 가장 먼저 최상위에 예외 핸들러를 갖춘다. localShippingRules() 호출을 try 블록으로 감싸려 하지만 처리 로직은 포함하고 싶지 않다.

그런데 다음처럼 할 수는 ‘없다’

```jsx
try {
  const status = calculateShippingCosts(orderData);
} catch (e) {
  // 예외 처리 로직
}
if (status < 0) errorList.push({ order: orderData, errorCode: status });
```

이렇게 하면 status의 유효범위가 try 블록으로 국한되어 조건문에서 검사할 수 없기 때문이다.

그래서 status 선언과 초기화를 분리해야 한다. 평소라면 좋아하지 않을 방식이지만 지금은 어쩔 수 없다.

```jsx
let status;
try {
  status = calculateShippingCosts(orderData);
} catch (e) {
  throw e;
}
if (status < 0) errorList.push({ order: orderData, errorCode: status });
```

잡은 예외는 모두 다시 던져야 한다. 다른 곳에서 발생한 예외를 무심코 삼켜버리고 싶진 않을테니 말이다.

호출하는 쪽 코드의 다른 부분에서도 주문을 오류 목록에 추가할 일이 있을 수 있으니 적절한 핸들러가 이미 구비되어 있을 수 있다.

그렇다면 그 try 블록을 수정해서 calculateShippingCosts() 호출을 포함시킨다.

[3] 이번 리팩터링으로 추가된 예외만을 처리하고자 한다면 다른 예외와 구별할 방법이 필요하다.

별도의 클래스를 만들어서 할 수도 있고 특별한 값을 부여하는 방법도 있다.

예외를 클래스 기반으로 처리하는 프로그래밍 언어가 많은데, 이런 경우라면 서브클래스를 만드는 게 가장 자연스럽다.

자바스크립트는 여기 해당하지 않지만 나는 다음처럼 하는 걸 좋아한다.

```jsx
class OrderProcessingError extends Error {
  constructor(errorCode) {
    super(`주문 처리 오류: ${errorCode}`);
    this.code = errorCode;
  }
  get name() {
    return "OrderProcessingError";
  }
}
```

[5] 이 클래스가 준비되면 오류 코드를 처리할 때와 같은 방식으로 이 예외 클래스를 처리하는 로직을 추가할 수 있다.

```jsx
let status;
try {
  status = calculateShippingCosts(orderData);
} catch (e) {
  if (e instanceof OrderProcessingError)
    errorList.push({ order: orderData, errorCode: status });
  throw e;
}
if (status < 0) errorList.push({ order: orderData, errorCode: status });
```

[7] 그런 다음 오류 검출 코드를 수정하여 오류 코드 대신 이 예외를 던지도록 한다.

```jsx
function localShippingRules(country) {
  const data = countryData.shippingRules[country];
  if (data) return new ShippingRules(data);
  else throw new OrderProcessingError(-23);
}
```

[8] 코드를 다 작성했고 테스트도 통과했다면 오류 코드를 전파하는 임시 코드를 제거할 수 있다.

하지만 나라면 먼저 다음처럼 함정을 추가한 후 테스트해볼 것이다.

```jsx
function calculdateShippingCosts(anOrder) {
  // 관련 없는 코드
  const shippingRules = localShippingRules(anOrder.country);
  if (shippingRules < 0) throw new Error("오류 코드가 다 사라지지 않았습니다.");
  // 더 관련 없는 코드
}
```

이 함정에 걸려들지 않는다면 이 줄 전체를 제거해도 안전하다. 오류를 콜스택 위로 전달하는 일은 예외 메커니즘이 대신 처리해줄 것이기 때문이다.

```jsx
function calculdateShippingCosts(anOrder) {
  // 관련 없는 코드
  const shippingRules = localShippingRules(anOrder.country);
  // if(shippingRules < 0) throw new Error("오류 코드가 다 사라지지 않았습니다.");
  // 더 관련 없는 코드
}
```

```jsx
// let status;
try {
  // status = calculateShippingCosts(orderData);
  calculateShippingCosts(orderData);
} catch (e) {
  if (e instanceof OrderProcessingError)
    errorList.push({ order: orderData, errorCode: status });
  throw e;
}
// if(status < 0) errorList.push({order : orderData, errorCode : status});
```

---

## 예외를 사전확인으로 바꾸기(Replace Exception with Precheck)

```java
double getValueForPeriod(int periodNumber) {
	try{
		return values[periodNumber];
	}catch(ArrayIndexOutOfBoundsException e) {
		return 0;
	}
}
```

```java
double getValueForPeriod (int periodNumber){
	return (periodNumber >= values.length) ? 0 : value[periodNumber];
}
```

### 배경

예외라는 개념은 프로그래밍 언어의 발전에 의미 있는 한걸음이었다.

오류 코드를 연쇄적으로 전파하던 긴 코드를 예외로 바꿔 깔끔히 제거할 수 있게 되었으니 말이다(오류 코드를 예외로 바꾸기[11.12])

하지만 좋은 것들이 늘 그렇듯, 예외도 더 이상 좋지 않을 정도까지 과용되곤 한다.

예외는 ‘뜻밖의 오류’라는, 말 그대로 예외적으로 동작할 때만 쓰여야 한다.

함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 검사하도록 해야 한다.

---

### 절차

1. 예외를 유발하는 상황을 검사할 수 있는 조건문을 추가한다.

   catch 블록의 코드를 조건문의 조건절 중 하나로 옮기고, 남은 try 블록의 코드를 다른 조건절로 옮긴다.

2. catch 블록에 어서션을 추가하고 테스트한다.
3. try문과 catch 블록을 제거한다.
4. 테스트한다.

---

### 예시(자바)

데이터베이스 연결 같은 자원들을 관리하는 자원 풀(resource pool) 클래스가 있다고 해보자.

자원이 필요한 코드는 풀에서 하나씩 꺼내 사용한다. 풀은 어떤 자원이 할당되었고 가용한지를 추적하고, 자원이 바닥나면 새로 생성한다.

```java
class ResourcePool {
	public Resource get() {
		Resource result;
		try {
			result = available.pop();
			allcated.add(result);
		}catch(NoSuchElementException e) {
			result = Resource.create();
			allcated.add(result);
		}
		return result;
	}

	private Deque<Resource> available;
	private List<Resource> allocated;
}
```

풀에서 자원이 고갈되는 건 예상치 못한 조건이 아니므로 예외 처리로 대응하는 건 옳지 않다.

사용하기 전에 allocated 컬렉션의 상태를 확인하기란 아주 쉬운 일이며, 예상 범주에 있는 동작임을 더 뚜렷하게 드러내주는 방식이다.

[1] 조건을 검사하는 코드를 추가하고 catch 블록의 코드를 조건문의 조건절로 옮기고, 남은 try 블록 코드를 다른 조건절로 옮겨보자.

```java
class ResourcePool {
	public Resource get() {
		Resource result;
		if(available.isEmpty()) {
			result = Resource.create();
			allacated.add(result);
		}else {
			try {
				result = available.pop();
				allcated.add(result);
			}catch(NoSuchElementException e) {
			}
		}
		return result;
	}

	private Deque<Resource> available;
	private List<Resource> allocated;
}
```

[2] catch 절은 더 이상 호출되지 않으므로 어서션을 추가한다.

```java
class ResourcePool {
	public Resource get() {
		Resource result;
		if(available.isEmpty()) {
			result = Resource.create();
			allacated.add(result);
		}else {
			try {
				result = available.pop();
				allcated.add(result);
			}catch(NoSuchElementException e) {
				throw new AssertionError("도달 불가");
			}
		}
		return result;
	}

	private Deque<Resource> available;
	private List<Resource> allocated;
}
```

나는 AssertionERror를 던지기보다는 assert 키워드를 사용할 때가 많지만, 그러면 result가 초기화되지 않을 수 있다며 컴파일러가 불평할 것이다.

어서션까지 추가한 후 테스트에 통과하면 [3] try 키워드와 catch 블록을 제거한다.

```java
class ResourcePool {
	public Resource get() {
		Resource result;
		if(available.isEmpty()) {
			result = Resource.create();
			allacated.add(result);
		}else {
				result = available.pop();
				allcated.add(result);
		}
		return result;
	}

	private Deque<Resource> available;
	private List<Resource> allocated;
}
```

[4] 한번 더 테스트에 통과하면 이번 리팩터링은 끝이다.

그런데 이번 리팩터링 결과로 얻어진 코드에는 정리할 거리가 더 있을 때가 많다.

이번에도 마찬가지다. 먼저 문장 슬라이드하기부터 적용해보자.

```java
class ResourcePool {
	public Resource get() {
		Resource result;
		if(available.isEmpty()) {
			result = Resource.create();
		}else {
				result = available.pop();
		}
		allacated.add(result);
		return result;
	}

	private Deque<Resource> available;
	private List<Resource> allocated;
}
```

그런 다음 if/else 쌍을 3항 연산자로 바꾼다.

```java
class ResourcePool {
	public Resource get() {
		Resource result = available.isEmpty() ? Resource.create() : available.pop();
		allacated.add(result);
		return result;
	}

	private Deque<Resource> available;
	private List<Resource> allocated;
}
```

---
