# [8] 기능 이동

지금까지는 프로그램 요소를 생성 혹은 제거하거나 이름을 변경하는 리팩터링을 다뤘다.

여기에 더해 요소를 다른 컨텍스트(클래스나 모듈 등)로 옮기는 일 역시 리팩터링의 중요한 축이다.

다른 클래스나 모듈로 함수를 옮길 때는 **함수 옮기기**를 사용한다. 필드 역시 **필드 옮기기**로 옮길 수 있다.

옮기기는 문장 단위에서도 이뤄진다. 문장을 함수 안이나 바깥으로 옮길 때는

**문장을 함수로 옮기기**나 **문장을 호출한 곳으로 옮기기**를 사용한다.

같은 함수 안에서 옮길 때는 **문장 슬라이드하기**를 사용한다.

때로는 한 덩어리의 문장들이 기존 함수와 같은 일을 할 때가 있다.

이럴 때는 **인라인 코드를 함수 호출로 바꾸기**를 적용해 중복을 제거한다.

반복문과 관련하여 자주 사용하는 리팩터링은 두 가지다.

첫 번째는 각각의 반복문이 단 하나의 일만 수행하도록 보장하는 **반복문 쪼개기**고,

두 번째는 반복문을 완전히 없애버리는 **반복문을 파이프라인으로 바꾸기**다.

마지막으로 많은 훌륭한 프로그래머가 즐겨 사용하는 리팩터링인 **죽은 코드 제거하기**가 있다.

필요 없는 문장들은 디지털 화염방사기로 태워버리는 것만큼 짜릿한 일도 없다.

## 함수 옮기기 (Move Function)

```jsx
class Account {
	get overdraftCharge() {...}
}
```

```jsx
class AccountType {
	get overdraftCharge() {...}
}
```

### 배경

좋은 소프트웨어 설계의 핵심은 모듈화가 얼마나 잘 되어 있느냐를 뜻하는 모듈성(modularity)이다.

모듈성이란 프로그램의 어딘가를 수정하려 할 때 해당 기능과 깊이 관련된 작은 일부만 이해해도 가능하게 해주는 능력이다.

모듈성을 높이려면 서로 연관된 요소들을 함께 묶고, 요소의 연결 관계를 쉽게 찾고 이해할 수 있도록 해야 한다.

하지만 프로그램을 얼마나 잘 이해했느냐에 따라 구체적인 방법이 달라질 수 있다.

보통은 이해도가 높아질수록 소프트웨어 요소들을 더 잘 묶는 새로운 방법을 깨우치게 된다.

그래서 높아진 이해를 반영하려면 요소들을 이리저리 옮겨야 할 수 있다.

모든 함수는 어떤 컨텍스트 안에 존재한다. 전역 함수도 있지만 대부분은 특정 모듈에 속한다.

객체지향 프로그래밍의 핵심 모듈화 컨텍스트는 클래스다. 또한 함수를 다른 함수에 중첩시켜도 또 다른 공통 컨텍스트를 만들게 된다.

프로그래밍 언어들은 저마다의 모듈화 수단을 제공하며, 각각의 수단이 함수가 살아 숨쉬는 컨텍스트를 만들어준다.

어떤 함수가 자신이 속한 모듈 A의 요소들보다 다른 모듈 B의 요소들을 더 많이 참조한다면 모듈 B로 옮겨줘야 마땅하다.

이렇게 하면 캡슐화가 좋아져서, 이 소프트웨어의 나머지 부분은 모듈 B의 세부사항에 덜 의존하게 된다.

이와 비슷하게, 호출자들의 현재 위치(호출자가 속한 모듈)나 다음 업데이트 때 바뀌리라 예상되는 위치에 따라서 함수를 옮겨야 할 수 있다.

예컨대 다른 함수 안에서 도우미 역할로 정의된 함수 중 독립적으로도 고유한 가치가 있는 것은 접근하기 더 쉬운 장소로 옮기는 게 낫다.

또한 다른 클래스로 옮겨두면 사용하기 더 편한 메서드도 있다.

함수를 옮길지 말지를 정하기란 쉽지 않다. 그럴 땐 대상 함수의 현재 컨텍스트와 후보 컨텍스트를 둘러보면 도움이 된다.

대상 함수를 호출하는 함수, 그 대상 함수가 호출하는 함수, 대상 함수가 사용하는 데이터가 무엇인지 살펴봐야 한다.

서로 관련된 여러 함수를 묶을 새로운 컨텍스트가 필요해질 때도 많은데,

이럴 때는 여러 함수를 클래스로 묶기(6.9)나 클래스 추출하기(7.5)로 해결할 수 있다.

함수의 최적 장소를 정하기가 어려울 수 있으나, 선택이 어려울수록 큰 문제가 아닌 경우가 많다.

경험상 함수들을 한 컨텍스트에 두고 작업해보는 것도 괜찮다.

그 곳이 얼마나 적합한지는 차차 깨달아갈 것임을 알고 있고, 잘 맞지 않다고 판단되면 위치는 언제든 옮길 수 있다.

---

### 절차

1. 선택한 함수가 현재 컨텍스트에서 사용중인 모든 프로그램 요소를 살펴본다. 이 요소들 중에도 함께 옮겨야 할 게 있는지 고민해본다.

   → 호출되는 함수 중 함께 옮길 게 있다면 대체로 그 함수를 먼저 옮기는 게 좋다.

   얽혀 있는 함수가 여러개라면 다른 곳에서 미치는 영향이 적은 함수부터 옮기도록 하자.

   → 하위 함수들의 호출자가 고수준 함수 하나뿐이면 먼저 하위 함수들을

   고수준 함수에 인라인한 다음, 고수준 함수를 옮기고, 옮긴 위치에서 개별 함수로 다시 추출하자

2. 선택한 함수가 다형 메서드인지 확인한다.

   → 객체 지향 언어에서는 같은 메서드가 슈퍼 클래스나 서브클래스에도 선언되어 있는지까지 고려해야 한다.

3. 선택한 함수를 타깃 컨텍스트로 복사한다.(이때 원래 함수를 소스 함수, 복사해서 만든 새로운 함수를 타깃 함수라 한다.

   타깃 함수가 새로운 터전에 잘 자리 잡도록 다듬는다.

   → 함수 본문에서 소스 컨텍스트의 요소를 사용한다면 해당 요소들을 매개변수로 넘기거나 소스 컨텍스트 자체를 참조로 넘겨준다.

   → 함수를 옮기게 되면 새로운 컨텍스트에 어울리는 새로운 이름으로 바꿔줘야 할 경우가 많다. 필요하면 바꿔준다.

4. 정적 분석을 수행한다.
5. 소스 컨텍스트에서 타깃 함수를 참조할 방법을 찾아 반영한다.
6. 소스 함수를 타깃 함수의 위임 함수가 되도록 수정한다.
7. 테스트한다.
8. 소스 함수를 인라인할지 고민한다.

   → 소스 함수는 언제까지라도 위임 함수로 남겨둘 수 있다.

   하지만 소스 함수를 호출하는 곳에서 타깃 함수를 직접 호출하는 데 무리가 없다면 소스 함수는 제거하는 편이 좋다.

---

### 예시

- 중첩 함수를 최상위로 옮기기
  GPS 추적 기록의 총 거리를 계산하는 함수로 시작해보자.

  ```jsx
  function trackSummary(points) {
    const totalTime = calculateTime();
    const totalDistance = calculateDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
      time : totalTime,
      distance : totalDistance,
      pace : pace
    };

    function calculateDistance() { // 총 거리 계산
      let result = 0;
      for (let i = 1; i < points.length; i++) {
        result += distance(points[i - 1], points[i]);
      }
      return result;
    }

    function distance(p1, p2) { ... } // 두 지점의 거리 계산
    function radians(degrees) { ... } // 라디안 값으로 변환
    function calculateTime() { ... } // 총 시간 계산
  }
  ```

  이 함수에서 중첩 함수이인 calculateDistance()를 최상위로 옮겨서 추적 거리를 다른 정보와는 독립적으로 계산하고 싶다.
  [3] 가장 먼저 할 일은 이 함수를 최상위로 복사하는 것이다.

  ```jsx
  function trackSummary(points) {
  	...
  }

  function top_calculateDistance() {
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      result += distance(points[i - 1], points[i]);
    }
    return result;
  }
  ```

  이렇게 함수를 복사할 때 이름을 달리해주면 코드에서나 머릿속에서나 소스 함수와 타깃 함수가 쉽게 구별된다.
  지금은 가자 적합한 이름을 고민할 단계가 아니므로 임시로 지어주면 된다.
  이 프로그램은 지금 상태로도 동작은 하지만 내 정적 분석기는 불만을 토로한다. 새 함수가 정의되지 않는 심벌(distance, points)를 사용하기 때문이다.
  points는 매개변수로 넘기면 자연스러울 것이다.

  ```jsx
  function top_calculateDistance(points) {
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      result += distance(points[i - 1], points[i]);
    }
    return result;
  }
  ```

  [1] distance() 함수도 똑같이 처리할 수 있지만 calcuateDistance()와 함께 옮기는 게 합리적으로 보인다.
  다음은 distance() 자신과 distnace()가 의존하는 코드다.

  ```jsx
  function distance(p1, p2) {
    // 허바사인 공식 (haversine formula)은 다음 사이트를 참고하자.
    // http://www.movable-type.co.uk/scripts/latlong.html
    const EARTH_RADIUS = 3959; // 단위 : 마일(mile)
    const dLat = radians(p2.lat) - radians(p1.lat);
    const dLon = radians(p2.lon) - radians(p1.lon);
    const a =
      Math.pow(Math.sin(dLat / 2), 2) +
      Math.cos(radians(p2.lat)) *
        Math.cos(radians(p1.lat)) *
        Math.pow(Math.sin(dLon / 2), 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    return EARTH_RADIUS * c;
  }
  ```

  보다시피 distance()는 radians()만 사용하며, radians()는 현재 컨텍스트에 있는 어떤 것도 사용하지 않는다.
  따라서 두 함수를 매개변수로 넘기기보다는 함께 옮겨버리는 것이 낫다.
  이를 위한 작은 첫 단추로, 현재 컨텍스트에서 이 함수들을 calculateDistance() 함수 안으로 옮겨보자.

  ```jsx
  function trackSummary(points) {

  	...

    function calculateDistance() { // 총 거리 계산
      let result = 0;
      for (let i = 1; i < points.length; i++) {
        result += distance(points[i - 1], points[i]);
      }
      return result;

      function distance(p1, p2) { ... }
      function radians(degrees) { ... } // 라디안 값으로 변환
    }
    function calculateTime() { ... } // 총 시간 계산
  }
  ```

  그런 다음 정적 분석과 테스트를 활용해 어딘가에서 문제가 생기는지 검증해보자.
  지금 경우엔 아무 문제가 없으니 같은 내용을 새로 만든 top_calculateDistance() 함수로도 복사한다.

  ```jsx
  function top_calculateDistance(points) { // 총 거리 계산
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      result += distance(points[i - 1], points[i]);
    }
    return result;

    function distance(p1, p2) { ... }
    function radians(degrees) { ... } // 라디안 값으로 변환
  }
  ```

  이번에도 복사한 코드가 프로그램 동작에 아무런 영향을 주지 않지만 [4] 다시 한번 정적 분석을 수행해볼 타이밍이다.
  내가 만약 distance()가 radians()를 호출하는 걸 발견하지 못했더라도 정적 분석기가 지금 단계에서 이 문제를 찾았을 것이다.
  [6] 이제 밥상은 다 차렸으니 핵심이 되는 수정을 진행할 차례다.
  즉, 소스 함수인 calculateDistance()의 본문을 수정하여 top_calculateDistance()를 호출하게 하자.

  ```jsx
  function trackSummary(points) {
    const totalTime = calculateTime();
    const totalDistance = calculateDistance();
    const pace = totalTime / 60 / totalDistance;
    return {
      time : totalTime,
      distance : totalDistance,
      pace : pace
    };

    function calculateDistance() { // 총 거리 계산
      return top_calculateDistance(points);
    }
    function calculateTime() { ... } // 총 시간 계산
  }
  ```

  [7] 이 시점에서 ‘반드시’ 모든 테스트를 수행하여 옮겨진 함수가 제대로 동작하는지 확인해야 한다.
  테스트를 통과하면 가장 먼저 소스 함수를 대리자 역할로 그대로 둘 지를 정하낟.
  이 예의 소스 함수는 호출자가 많지 않은, 상당히 지역화된 함수다.
  그러니 소스 함수(calculateDistance())는 제거하는 편이 낫겠다.
  이제 새 함수에 이름을 지어줄 시간이다. 최상위 함수는 가시성이 가장 높으니 적합한 이름을 신중히 지어주는 게 좋다.
  titalDistance() 정도면 부족하지 않을 것이다.
  그런데 trackSummary() 안에 정의된 똑같은 이름의 변수가 새 함수를 가릴 것이라 곧바로 적용할 수는 없다.
  근데 이 변수를 남겨둘 이유가 없으니 변수 인라인하기로 해결하자.

  ```jsx
  function trackSummary(points) {
    const totalTime = calculateTime();
    const pace = totalTime / 60 / totalDistance(points);
    return {
      time : totalTime,
      distance : totalDistance(points),
      pace : pace
    };

    function calculateTime() { ... } // 총 시간 계산
  }
  ```

  혹시나 이 변수를 남겨둬야 한다면 변수 이름을 totalDistanceCache나 distance 정도로 바꿔주면 된다.
  distance()와 radians() 함수도 totalDistance() 안의 어떤 것에도 의존하지 않으니,
  나라면 이들 역시 최상위로 옮길 것이다. 그러면 네 함수 모두 최상위가 된다.

  ```jsx
  function trackSummary(points) { ... }
  function totalDistance(points) { ... }
  function distance(p1, p2) { ... }
  function radians(degrees) { ... }
  ```

  distance()와 radians()를 totalDistance() 안에 그대로 두어 가시성을 줄이는 쪽을 선호할 수도 있다.
  언어에 따라 이 방식도 고려해봄 직하지만, ES2015 이후의 자바스크립트라면 멋진 모듈 메커니즘을 이용해 함수 가시성을 제어할 수 있다.
  중첩 함수를 사용하다보면 숨겨진 데이터끼리 상호 의존하기가 아주 쉬우니 중첩 함수는 되도록 만들지 말자.

- 다른 클래스로 옮기기
  이번엔 함수 옮기기 리팩터링의 다채로움을 보여주기 위한 예를 준비했다.

  ```jsx
  class Account {
    get bankCharge() {
      // 은행 이자 계산
      let result = 4.5;
      if (this.daysOverDrawn > 0) result += this.overdraftCharge;
      return result;
    }

    get overdraftCharge() {
      // 초과 인출 이자 계산
      if (this.type.isPremium) {
        const baseCharge = 10;
        if (this.daysOverdrawn <= 7) return baseCharge;
        else return baseCharge + (this.daysOverdrawn - 7) * 0.85;
      } else return this.daysOverdrawn * 1.75;
    }
  }
  ```

  이제부터 계좌 종류에 따라 이자 책정 알고리즘이 달라지도록 고쳐보자.
  그러러면 마이너스 통장의 초과 인출 이자를 계산하는 overdraftCharge()를
  계좌 종류 클래스인 AccountType으로 옮기는 게 자연스러울 것이다.
  [1] 첫 단계로 overdraftCharge() 메서드가 사용하는 기능들을 살펴보고,
  그 모두를 한꺼번에 옮길만한 가치가 있는지 고민해보자.
  이 예에서는 daysOverdrawn() 메서드는 Acoount 클래스에 남겨둬야 한다.
  계좌 종류가 아닌 계좌별로 달라지는 메서드이기 때문이다.
  [3] 다음으로 overdraftCharge() 메서드 본문을 AccountType 클래스로 복사한 후 새 보금자리에 맞게 정리한다.

  ```jsx
  class AccountType {
    overdraftCharge(daysOverdrawn) {
      if (this.isPremium) {
        const baseCharge = 10;
        if (daysOverdrawn <= 7) return baseCharge;
        else return baseCharge + (daysOverdrawn - 7) * 0.85;
      } else return daysOverdrawn * 1.75;
    }
  }
  ```

  이 메서드를 새 보금자리에 맞추려면 호출 대상 두 개의 범위를 조정해야 한다.
  isPremium은 단순히 this를 통해 호출했다. 한편 daysOverdrawn은 값을 넘길지, 아니면 계좌채로 넘길지 정해야 한다.
  우선 간단히 값으로 넘기기로 하자. 하지만 초과 인출된 일수 외에 다른 정보가 필요해지면
  추후 계좌채로 넘기도록 변경할 수도 있을 것이다.
  계좌에서 원하는 정보가 계좌 종류에 따라 달라진다면 더욱 그렇다.
  [6] 다음으로, 원래 메서드의 본문을 수정하여 새 메서드를 호출하도록 한다.
  이제 원래 메서드는 위임 메서드가 된다.

  ```jsx
  Account {
  	...
  	get overdraftCharge() { // <- 위임 메서드
      return this.type.overdraftCharge(this.daysOverdrawn);
    }
  }
  ```

  [8] 이제 위임 메서드인 overdraftCharge()를 남겨둘지 아니면 인라인할지 정해야 한다.
  인라인 쪽을 선택하면 다음과 같다.

  ```jsx
  class Account {
    get bankCharge() {
      // 은행 이자 계산
      let result = 4.5;
      if (this.daysOverDrawn > 0)
        result += this.type.overdraftCharge(this.daysOverdrawn);
      return result;
    }
  }
  ```

  - 소스 컨텍스트에서 가져와야 할 데이터가 많다면?
    이전 단계들에서 daysOverdrawn을 매개변수로 넘겼지만,
    만약 계좌에서 가져와야 할 데이터가 많았다면 다음과 같이 계좌 자체를 넘겼을 것이다.

    ```jsx
    class Account {
      get bankCharge() {
        // 은행 이자 계산
        let result = 4.5;
        if (this.daysOverDrawn > 0) result += this.overdraftCharge;
        return result;
      }

      get overdraftCharge() {
        return this.type.overdraftCharge(this);
      }
    }

    class AccountType {
      overdraftCharge(account) {
        if (this.isPremium) {
          const baseCharge = 10;
          if (account.daysOverdrawn <= 7) return baseCharge;
          else return baseCharge + (account.daysOverdrawn - 7) * 0.85;
        } else return account.daysOverdrawn * 1.75;
      }
    }
    ```

---

```jsx
class Customer {
  get plan() {
    return this.plan;
  }
  get discountRate() {
    return this.discountRate;
  }
}
```

```jsx
class Customer {
  get plan() {
    return this.plan;
  }
  get discountRate() {
    return this.plan.discountRate;
  }
}
```

## 필드 옮기기(Move Field)

### 배경

프로그램의 상당 부분이 동작을 구현하는 코드로 이뤄지지만 프로그램의 진짜 힘은 데이터 구조에서 나온다.

주어진 문제에 적합한 데이터 구조를 활용하면 동작 코드는 자연스럽게 단순하고 직관적으로 짜여진다.

반면 데이터 구조를 잘못 선택하면 아귀가 맞지 않는 데이터를 다루기 위한 코드로 범벅이 된다.

이해하기 어려운 코드가 만들어지는 데서 끝나지 않고, 데이터 구조 자체도 그 프로그램이 어떤 일을 하는지 파악하기 어렵게 한다.

그래서 데이터 구조가 중요하다. 하지만 훌륭한 프로그램이 갖춰야 할 다른 요인들과 마찬가지로, 제대로 하기가 어렵다.

가장 적합한 데이터 구조를 알아내고자 프로젝트 초기에 분석을 해본 결과, 경험과 도메인 주도 설계 같은 기술이 내 능력을 개선해줌을 알아냈다.

하지만 나의 모든 기술과 경험에도 불구하고 초기 설계에서는 실수가 빈번했다.

프로젝트를 진행할수록 우리는 문제 도메인과 데이터 구조에 대해 더 많은 것을 배우게 된다.

그래서 오늘까지는 합리적이고 올바랐던 설계가 다음 주가 되면 잘못된 것으로 판명나곤 한다.

현재 데이터 구조가 적절치 않음을 깨닫게 되면 곧바로 수정해야 한다. 고치지 않고 데이터 구조에 남겨진 흠들은

우리 머릿속을 혼란스럽게 하고 훗날 작성하게 될 코드를 더욱 복잡하게 만들어버린다.

예컨대 함수에 어떤 레코드를 넘길 때마다 또 다른 레코드의 필드도 함께 넘기고 있다면 데이터 위치를 옮겨야 할 것이다.

함수에 항상 함께 건네지는 데이터 조각들은 상호 관계가 명확하게 드러나도록 한 레코드에 담는게 가장 좋다.

변경 역시 주요한 요인이다. 한 레코드를 변경하려 할 때 다른 레코드의 필드까지 변경해야 한다면

필드의 위치가 잘못되었다는 신호다.

구조체 여러 개에 정의된 똑같은 필드들을 갱싱해야 한다면 한 번만 갱신해도 되는 다른 위치로 옮기라는 신호다.

필드 옮기기 리팩터링은 대체로 더 큰 변경의 일환으로 수행된다. 예컨대 필드 하나를 잘 옮기면,

그 필드를 사용하던 많은 코드가 원래 위치보다 옮겨진 위치에서 사용하는 게 더 수월할 수 있다.

그렇다면 리팩터링을 마저 진행하여 호출 코드들까지 모두 변경한다.

비슷하게, 옮기려는 데이터가 쓰이는 패턴 때문에 당장은 필드를 옮길 수 없을 때도 있다.

이럴 땐 사용 패턴을 먼저 리팩터링한 다음에 필드를 옮겨준다.

지금까지의 설명에서 레코드라는 용어를 썼지만, 레코드 대신 클래스나 객체가 와도 똑같다.

클래스는 함수가 곁들여진 레코드라 할 수 있으며, 다른 데이터와 마찬가지로 건강하게 관리돼야 한다.

클래스의 데이터들은 접근자 메서드들 뒤에 감춰져 (캡슐화되어) 있으므로 클래스에 곁들여진 함수(메서드)

들은 데이터를 이리저리 옮기는 작업을 쉽게 해준다.

데이터의 위치를 옮기더라도 접근자만 그에 맞게 수정하면 클라이언트 코드들은 아무 수정 없이도 동작할 것이다.

따라서 클래스를 사용하면 이 리팩터링을 수행하기가 더 쉬워지며, 그래서 이어지는 설명에서는 클래스를 사용한다고 가정한다.

캡슐화되지 않은 날(bare) 레코드를 사용해도 똑같이 변경할 수는 있지만, 더 까다로울 것이다.

---

### 절차

1. 소스 필드가 캡슐화되어 있지 않다면 캡슐화한다.
2. 테스트한다.
3. 타깃 객체에 필드와 접근자 메서드들을 생성한다.
4. 정적 검사를 수행한다.
5. 소스 객체에서 타깃 객체를 참조할 수 있는지 확인한다.

   → 기존 필드나 메서드 중 타깃 객체를 넘겨주는 게 있을지 모른다. 없다면 이런 기능의 메서드를 쉽게 만들 수 있는지 살펴본다.

   간단치 않다면 타깃 객체를 저장할 새 필드를 소스 객체에 생성하자. 이는 영구적인 변경이 되겠지만,

   더 넒은 맥락에서 리팩터링을 충분히 하고 나면 다시 없앨 수 있을 때도 있다.

6. 접근자들이 타깃 필드를 사용하도록 수정한다.

   → 여러 소스에서 같은 타깃을 공유한다면, 먼저 세터를 수정하여 타깃 필드와 소스 필드 모두를 갱신하게 하고,

   이어서 일관성을 깨뜨리는 갱신을 검출할 수 있도록 어서션을 추가(10.8)하자.

   모든게 잘 마무리되었다면 접근자들이 타깃 필드를 사용하도록 수정한다.

7. 테스트한다.
8. 소스 필드를 제거한다.
9. 테스트한다.

---

### 예시

- 기본 예시

  ```jsx
  class Customer {
    constructor(name, discountRate) {
      this.name = name;
      this.discountRate = discountRate;
      this.contract = new CustomerContract(dateToday());
    }

    get discountRate() {
      return this.discountRate;
    }
    becomePreferred() {
      this.discountRate += 0.03;
      // 로직 요약
    }
    applyDiscount(amount) {
      return amount.subtract(amount.multiply(this.discountRate));
    }
  }

  class CustomerContract {
    constructor(startDate) {
      this.startDate = startDate;
    }
  }
  ```

  여기서 할인율을 뜻하는 discountRate 필드를 Customer에서 CustomerConstract로 옮기고 싶다고 가정해보자.
  [1] 가장 먼저 할 일은 이 필드를 캡슐화하는 것이다.

  ```jsx
  class Customer {
    constructor(name, discountRate) {
      ...
      this.setDiscountRate(discountRate);
      ...
    }

    ...
    setDiscountRate(aNumber) { this.discountRate = aNumber }
    becomePreferred() {
      this.setDiscountRate(this.discountRate + 0.03);
      // 로직 요약
    }
  }
  ```

  할인율을 수정하는 public 세터를 만들고 싶지는 않아서 세터 속성이 아니라 메서드를 이용했다.
  [3] 이제 CustomerContract 클래스에 필드 하나와 접근자들을 추가한다.

  ```jsx
  class CustomerContract {
    constructor(startDate, discountRate) {
      this.startDate = startDate;
      this.discountRate = discountRate;
    }

    get discountRate() {
      return this.discountRate;
    }
    set discountRate(arg) {
      this.discountRate = arg;
    }
  }
  ```

  [6] 그런 다음 Customer의 접근자들이 새로운 필드를 사용하도록 수정한다. 다 수정하고 나면
  “Cannot set property ‘discountRate’ of undefined”라는 오류가 날 것이다.
  생성자에서 Contract 객체를 생성하기도 전에 setDiscountRate()를 호출하기 때문이다.
  이 오류를 고치려면 먼저 기존 상태로 되돌린 다음, 문장 슬라이드하기를 적용해 setDiscountRate() 호출을 계약 생성 뒤로 옮겨야 한다.

  ```jsx
  class Customer {
    constructor(name, discountRate) {
      this.name = name;
      this.contract = new CustomerContract(dateToday());
      this.setDiscountRate(discountRate);
    }
  	...
  }
  ```

  테스트에 성공하면 접근자들을 다시 수정하여 새로운 계약 인스턴스를 사용하도록 한다.

  ```jsx
  class Customer {
    get discountRate() {
      return this.contarct.discountRate;
    }
    setDiscountRate(aNumber) {
      this.contract.discountRate = aNumber;
    }
  }
  ```

  [8] 자바스크립트를 사용하고 있으므로 소스 필드를 미리 선언할 필요는 없었다. 그래서 제거해야 할 것도 없다.

- 날 레코드 변경하기
  이 리팩터링은 대체로 객체를 활용할 때가 더 수월하다. 캡슐화 덕에 데이터 접근을 메서드로 자연스럽게 감싸주기 때문이다.
  반면, 여러 함수가 날 레코드를 직접 사용하는 경우라면 이 리팩터링은 훨씬 까다롭다.
  이럴 때는 접근자 함수들을 만들고, 날 레코드를 읽고 쓰는 모든 함수가 접근자를 거치도록 고치면 된다.
  옮길 필드가 불변이라면 값을 처음 설정할 때 소스와 타깃 필드를 한꺼번에 갱신하게 하고, 읽기 함수들은 점진적으로 마이그레이션하자.
  나라면 가장 먼저 레코드를 캡슐화하여 클래스로 바꿀 것이다. 물론 가능할 때의 얘기지만, 이렇게 하면 필드 옮기기 리팩터링이 수월해진다.
- 공유 객체로 이동하기
  다음 코드는 이자율(interest rate)을 계좌(account)별로 설정하고 있다.

  ```jsx
  class Account {
    constructor(number, type, interestRate) {
      this.number = number;
      this.type = type;
      this.interestRate = interestRate;
    }

    get interestRate() {
      return this.interestRate;
    }
  }

  class AccountType {
    constructor(nameString) {
      this.name = nameString;
    }
  }
  ```

  이 코드를 수정하여 이자율이 계좌 종류에 따라 정해지도록 하려고 한다.
  [1] 이자율 필드는 이미 잘 캡슐화되어 있으니 [3] 가볍게 타깃 AccountType에 이자율 필드와 필요한 접근자 메서드를 생성해보자.

  ```jsx
  class AccountType {
    constructor(nameString, interestRate) {
      this.name = nameString;
      this.interestRate = interestRate;
    }

    get interestRate() {
      return this.interestRate;
    }
  }
  ```

  [4] 그런데 Account가 AccountType의 이자율을 가져오도록 수정하면 문제가 생길 수 있다.
  이 리팩터링 전에는 각 계좌가 자신만의 이자율을 갖고 있었고, 지금은 종류가 같은 모든 계좌가 이자율을 공유하기를 원한다.
  만약 수정 전에도 이자율이 계좌 종류별로 같게 설정되어 있었다면 겉보기 동작이 달라지지 않으니 그대로 리팩터링하면 된다.
  하지만 이자율이 다른 계좌가 하나라도 있었다면, 이건 더 이상 리팩터링이 아니다.
  수정 전과 후의 겉보기 동작이 달라지기 때문이다.
  따라서 계좌 데이터를 데이터베이스에 보관한다면 먼저 데이터베이스를 확인해서 모든 계좌의 이자율이 계좌 종류에 부합하게 설정되어 있는지 확인해야 한다.
  계좌 클래스에 어서션을 추가하는 것도 도움이 된다.

  ```jsx
  class Account {
  	constructor(number, type, interestRate) {
  		this.number = number;
  		this.type = type;
  		**assert(interestRate == this.type.interestRate);**
  		this.interestRate = interestRate;
  	}

  	get interestRate() { return this.interestRate; }
  }
  ```

  이와 같이 어서션을 적용한 채 시스템을 잠시 운영해보며 오류가 생기는지 지켜보는 것이다.
  어서션을 추가하는 대신 문제 발생 시 로깅하는 방법도 있다.
  [6] 시스템의 겉보기 동작이 달라지지 않는다는 확신이 서면 이자율을 가져오는 부분을 변경하고
  [8] Account에서 이자율을 직접 수정하던 코드를 완전히 제거한다.

  ```jsx
  class Account {
    constructor(number, type) {
      this.number = number;
      this.type = type;
    }

    get interestRate() {
      return this.type.interestRate;
    }
  }
  ```

---

## 문장을 함수로 옮기기(Move Statements into Function)

> **반대 리팩터링**

문장을 호출한 곳으로 옮기기

```jsx
result.push(`<p>제목: ${person.photo.title}</p>`);
result.concat(photoData(person.photo));

function photoData(aPhoto) {
  return [
    `<p>위치 : ${aPhoto.location}</p>`,
    `<p>날짜 : ${aPhoto.date.toDateString()}</p>`,
  ];
}
```

```jsx
result.concat(photoData(person.photo));

function photoData(aPhoto) {
  return [
    `<p>제목: ${person.photo.title}</p>`,
    `<p>위치 : ${aPhoto.location}</p>`,
    `<p>날짜 : ${aPhoto.date.toDateString()}</p>`,
  ];
}
```

### 배경

중복 제거는 코드를 건강하게 관리하는 가장 효과적인 방법 중 하나다.

예컨대 특정 함수를 호출하는 코드가 나올 때마다 그 앞이나 뒤에서 똑같은 코드가 추가로 실행되는 모습을 보면,

나는 그 반복되는 부분을 피호출 함수로 합치는 방법을 궁리한다.

이렇게 해두면 추후 반복되는 부분에서 무언가 수정할 일이 생겼을 때 단 한 곳만 수정하면 된다.

호출하는 곳이 아무리 많더라도 말이다. 혹시 나중에 이 코드의 동작을 여러 변형들로 나눠야 하는 순간이 오면

문장을 호출한 곳으로 옮기기를 적용하여 쉽게 다시 뽑아낼 수 있다.

문장들을 함수로 옮기려면 그 문장들이 피호출 함수의 일부라는 확신이 있어야 한다.

피호출 함수와 한 몸은 아니지만 여전히 함께 호출돼야 하는 경우라면 단순히 해당 문장들과 피호출 함수를 통째로 또 하나의 함수로 추출한다.

이 방법도 절차는 똑같다. 단, 마지막의 인라인과 이름 바꾸기 단계만 제외하면 된다.

이 역시 심심치 않게 활용되는 방법이며, 나중에 필요하다면 생략했던 마지막 단계들을 마저 수행할 수도 있다.

---

### 절차

1. 반복 코드가 함수 호출 부분과 멀리 떨어져 있다면 문장 슬라이드하기를 적용해 근처로 옮긴다.
2. 타깃 함수를 호출하는 곳이 한 곳뿐이면, 단순히 소스 위치에서 해당 코드를 잘라내어 피호출 함수로 복사하고 테스트한다.

   이 경우라면 나머지 단계는 무시한다.

3. 호출자가 둘 이상이면 호출자 중 하나에서 ‘타깃 함수 호출 부분과 그 함수로 옮기려는 문장들을 함께’

   다른 함수로 추출한다. 추출한 함수에 쉬운 임시 이름을 지어준다.

4. 다른 호출자 모두가 방금 추출한 함수를 사용하도록 수정한다. 하나씩 수정할 때마다 테스트한다.
5. 모든 호출자가 새로운 함수를 사용하게 되면 원래 함수를 새로운 함수 안으로 인라인한 후 원래 함수를 제거한다.
6. 새로운 함수의 이름을 원래 함수의 이름으로 바꿔준다.

---

### 예시

사진 관련 데이터를 HTML로 내보내는 코드이다.

```jsx
function renderPerson(outStream, person) {
  const result = [];
  result.push(`<p>${person.name}</p>`);
  result.push(renderPhoto(person.photo));
  result.push(`<p>제목 : ${person.photo.title}</p>`);
  result.push(emitPhotoData(person.photo));
  return result.join("\n");
}

function photoDiv(p) {
  return ["<div>", `<p>제목: ${p.title}</p>`, emitPhotoData(p), "</div>"].join(
    "\n"
  );
}

function emitPhotoData(aPhoto) {
  const result = [];
  result.push(`<p>위치 : ${aPhoto.location}</p>`);
  result.push(`<p>날짜 : ${aPhoto.date.toDateString()}</p>`);
  return result.join("\n");
}
```

이 코드에서는 총 두 곳에서 emitPhotoData()를 호출하며, 두 곳 모두 바로 앞에는 제목(title) 출력 코드가 나온다.

제목을 출력하는 코드를 emitPhotoData() 안으로 옮겨 이 중복을 없애보자.

호출자가 하나였다면 단순히 해당 코드를 잘라 붙이면 되지만, 호출자 수가 늘어날수록 더 안전한 길을 택해야 한다.

[3] 가장 먼저 호출자 중 하나에 함수 추출하기를 적용하자. 다음과 같이 emitPhotoData()로 옮기려는 코드와 emitPhotoData() 호출문을 함께 추출하면 된다.

```jsx
function photoDiv(p) {
  return ["<div>", `<p>제목: ${p.title}</p>`, zznew(p), "</div>"].join("\n");
}

function zznew(p) {
  return [`<p>제목: ${p.title}</p>`, emitPhotoData(p)].join("\n");
}
```

[4] 이제 다른 호출자들로 눈을 돌려서, 하나씩 차례로 새로운 함수를 호출하도록 수정한다.

```jsx
function renderPerson(outStream, person) {
  const result = [];
  result.push(`<p>${person.name}</p>`);
  result.push(renderPhoto(person.photo));
  result.push(zznew(person.photo));
  return result.join("\n");
}
```

[5] 호출자들을 빠짐없이 수정했다면 emitPhotoData() 함수를 인라인한다.

```jsx
function zznew(p) {
  return [
    `<p>제목: ${p.title}</p>`,
    `<p>위치 : ${aPhoto.location}</p>`,
    `<p>날짜 : ${aPhoto.date.toDateString()}</p>`,
  ].join("\n");
}
```

[6] 그리고 함수 이름을 바꿔 마무리한다. (함수 이름 바꾸기)

```jsx
function renderPerson(outStream, person) {
  const result = [];
  result.push(`<p>${person.name}</p>`);
  result.push(renderPhoto(person.photo));
  result.push(emitPhotoData(person.photo));
  return result.join("\n");
}

function photoDiv(p) {
  return ["<div>", `<p>제목: ${p.title}</p>`, emitPhotoData(p), "</div>"].join(
    "\n"
  );
}

function emitPhotoData(p) {
  return [
    `<p>제목: ${p.title}</p>`,
    `<p>위치 : ${aPhoto.location}</p>`,
    `<p>날짜 : ${aPhoto.date.toDateString()}</p>`,
  ].join("\n");
}
```

이 과정에서 매개변수 이름이 여러분의 규약과 맞지 않다면 적절히 수정하자.

---

## 문장을 호출한 곳으로 옮기기(Move Statements to Callers)

```jsx
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>\n`);
  outStream.write(`<p>위치: ${photo.location}</p>\n`);
}
```

```jsx
emitPhotoData(outStream, person.photo);
outStream.write(`<p>위치: ${photo.location}</p>\n`);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>\n`);
}
```

### 배경

함수는 프로그래머가 쌓아 올리는 추상화의 기본 빌딩 블록이다.

그런데 추상화라는 것이 그 경계를 항상 올바르게 긋기가 만만치 않다.

그래서 코드베이스의 기능 범위가 달라지만 추상화의 경계도 움직이게 된다.

함수 관점에서 생각해보면, 초기에는 응집도 높고 한 가지 일만 수행하던 함수가 어느새 둘 이상의 다른 일을 수행하게 바뀔 수 있다는 뜻이다.

예컨대 여러 곳에서 사용하던 기능이 일부 호출자에게는 다르게 동작하도록 바뀌어야 한다면 이런 일이 벌어진다.

그렇다면 개발자는 달라진 동작을 함수에서 꺼내 해당 호출자로 옮겨야 한다.

이런 상황에 맞닥뜨리면 우선 문장 슬라이드하기를 적용해 달라지는 동작을 함수의 시작 혹은 끝으로 옮긴 다음,

바로 이어서 문장을 호출한 곳으로 옮기기 리팩터링을 적용하면 된다.

달라지는 동작을 호출자로 옮긴 뒤에는 필요할 때마다 독립적으로 수정할 수 있다.

작은 변경이라면 문장을 호출한 곳으로 옮기는 것으로 충분하지만, 호출자와 호출 대상의 경계를 완전히 다시 그어야 할 때도 있다.

후자의 경우라면 함수 인라인하기부터 적용한 다음, 문장 슬라이드하기와 함수 추출하기로 더 적합한 경계를 설정하면 된다.

---

### 절차

1. 호출자가 한두 개뿐이고 피호출 함수도 간단, 단순한 상황이면 피호출 함수의 처음(혹은 마지막) 줄(들)을 잘라내어 호출자(들)로 복사해 넣는다.

   테스트만 통과하면 이번 리팩터링은 여기서 끝이다.

2. 더 복잡한 상황에서는, 이동하지 ‘않길’ 원하는 모든 문장을 함수로 추출한 다음 검색하기 쉬운 임시 이름을 지어준다.

   → 대상 함수가 서브클래스에서 오버라이드됐다면 오버라이드한 서브클래스들의 메서드 모두에서 동일하게 남길 부분을 메서드로 추출한다.

   이때 남겨질 메서드의 본문은 모든 클래스에서 똑같아야 한다. 그런 다음 서브 클래스들의 메서드를 제거한다.

3. 원래 함수를 인라인한다.
4. 추출된 함수의 이름을 원래 함수의 이름으로 변경한다.

---

### 예시

호출자가 둘뿐인 단순한 상황을 살펴보자

```jsx
function renderPerson(outStream, person) {
  outStream.write(`<p>${person.name}</p>\n`);
  renderPhoto(outStream, person.photo);
  emitPhotoData(outStream, person.photo);
}

function listRecentPhotos(outStream, photos) {
  photos
    .filter((p) => p.date > recentDateCutoff())
    .forEach((p) => {
      outStream.write("<div>\n");
      emitPhotoData(outStream, p);
      outStream.write("</div>\n");
    });
}

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>\n`);
  outStream.write(`<p>날짜 : ${photo.date.toDateString()}</p>\n`);
  outStream.write(`<p>위치 : ${photo.location}</p>\n`);
}
```

이 소프트웨어를 수정하여 renderPerson()은 그대로 둔 채 listRecentPhotos()가 위치 정보(location)를 다르게 렌더링하도록 만들어야 한다고 해보자.

이 변경을 쉽게 처리하기 위해 마지막 문장을 호출한 곳으로 옮겨보겠다.

[1] 사실 이렇게 단순한 상황에서는 renderPerson()의 마지막 줄을 잘라내어 두 호출 코드 아래 붙여 넣으면 끝이다.

하지만 여러분이 더 까다로운 상황에도 대처할 수 있도록 더 복잡한, 하지만 더 안전한 길로 가보겠다.

[2] 첫 단계로 emitPhotoData()에 남길 코드를 함수로 추출한다.

```jsx
function renderPerson(outStream, person) {
  ...
}

function listRecentPhotos(outStream, photos){
  ...
}

function emitPhotoData(outStream, photo) {
	zztmp(outStream, photo);
  outStream.write(`<p>위치 : ${photo.location}</p>\n`);
}

function zztmp(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>\n`);
  outStream.write(`<p>날짜 : ${photo.date.toDateString()}</p>\n`);
}
```

추출된 함수의 이름은 임시로만 쓰이는 게 보통이니 의미 없는 이름을 사용해도 괜찮지만 이왕이면 검색하기 쉬운 이름이 좋다.

이쯤에서 수정된 코드가 함수 호출 경계를 넘어 잘 동작하는지 테스트한다.

[3] 다음으로 피호출 함수를 호출자들로 한 번에 하나씩 인라인한다. renderPerson()부터 시작하자.

```jsx
function renderPerson(outStream, person) {
  outStream.write(`<p>${person.name}</p>\n`);
  renderPhoto(outStream, person.photo);
  zztmp(outStream, person.photo);
  outStream.write(`<p>위치 : ${photo.location}</p>\n`);
}
```

이 호출이 올바로 동작하는지 테스트한 후 다음 함수에도 인라인한다.

```jsx
function listRecentPhotos(outStream, photos) {
  photos
    .filter((p) => p.date > recentDateCutoff())
    .forEach((p) => {
      outStream.write("<div>\n");
      zztmp(outStream, person.photo);
      outStream.write(`<p>위치 : ${photo.location}</p>\n`);
      outStream.write("</div>\n");
    });
}
```

이제 원래 함수를 지워 함수 인라인하기를 마무리한 후 zztmp()의 이름을 원래 함수의 이름으로 되돌린다.

```jsx
function renderPerson(outStream, person) {
  outStream.write(`<p>${person.name}</p>\n`);
  renderPhoto(outStream, person.photo);
  emitPhotoData(outStream, person.photo);
  outStream.write(`<p>위치 : ${photo.location}</p>\n`);
}

function listRecentPhotos(outStream, photos) {
  photos
    .filter((p) => p.date > recentDateCutoff())
    .forEach((p) => {
      outStream.write("<div>\n");
      emitPhotoData(outStream, person.photo);
      outStream.write(`<p>위치 : ${photo.location}</p>\n`);
      outStream.write("</div>\n");
    });
}

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${photo.title}</p>\n`);
  outStream.write(`<p>날짜 : ${photo.date.toDateString()}</p>\n`);
}
```

---

## 인라인 코드를 함수 호출로 바꾸기(Replace Inline Code with Function call)

```jsx
let appliesToMass = false;
for (const s of states) {
  if (s === "MA") appliesToMass = true;
}
```

```jsx
appliesToMass = states.includes("MA");
```

### 배경

함수는 여러 동작을 하나로 묶어준다. 그리고 함수의 이름이 코드의 동작 방식보다는 목적을 말해주기 때문에 함수를 활용하면 코드를 이해하기가 쉬워진다.

함수는 중복을 없애는데도 효과적이다. 똑같은 코드를 반복하는 대신 함수를 호출하면 된다.

이렇게 해두면 동작을 변경할 때도, 비슷해 보이는 코드들을 일일이 찾아 수정하는 대신 함수 하나만 수정하면 된다.

(물론 모든 호출자가 수정된 코드를 사용하는 게 맞는지 확인해야 하지만, 이 편이 더 쉽고 일부 호출자만 다른 코드를 원하는 일은 흔치 않다.)

이미 존재하는 함수와 똑같은 일을 하는 인라인 코드를 발견하면 보통은 해당 코드를 함수 호출로 대체하길 원할 것이다.

예외가 있다면, 순전히 우연히 비슷한 코드가 만들어졌을 때 뿐이다.

즉, 기존 함수의 코드를 수정하더라도 인라인 코드의 동작은 바뀌지 않아야 할 때 뿐이다.

이 경우인가를 판단하는 데는 함수 이름이 힌트가 된다.

이름을 잘 지었다면 인라인 코드 대신 함수 이름을 넣어도 말이 된다.

말이 되지 않는다면 함수 이름이 적절하지 않거나 그 함수의 목적이 인라인 코드의 목적과 다르기 때문일 것이다.

특히 라이브러리가 제공하는 함수로 대체할 수 있다면 훨씬 좋다. 함수 본문을 작성할 필요조차 없어지기 때문이다.

---

### 절차

1. 인라인 코드를 함수 호출로 대체한다.
2. 테스트한다.

---

함수 추출하기와 이번 리팩터링의 차이는 인라인 코드를 대체할 함수가 이미 존재하느냐 여부다.

아직 없어서 만들어야 한다면 함수 추출하기이며, 이미 존재한다면 인라인 코드를 함수 호출로 바꾸기다.

사용 중인 프로그래밍 언어의 표준 라이브러리나 플랫폼(혹은 프레임워크, 서드파티 라이브러리 등)이 제공하는

API를 잘 파악하고 있을수록 이번 리팩터링의 활용 빈도가 높아질 것이다.

보통은 직접 짠 코드보다 라이브러리가 제공하는 API가 더 효율적일 가능성이 크니, 사용 중인 언어와 플랫폼이 버전업될 때

어떤 기능이 생기고 변경되는지를 평소에 잘 챙겨두면 코드를 개선하는데 많은 도움이 된다.

물론 외부 라이브러리에 지나치게 의존하면 설계 유연성이 떨어지니 처한 상황에 맞게 신중히 판단해야 한다.

---

## 문장 슬라이드하기 (Slide Statements)

```jsx
const princingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = princingPlan.unit;
```

```jsx
const princingPlan = retrievePricingPlan();
const chargePerUnit = princingPlan.unit;
const order = retreiveOrder();
let charge;
```

### 배경

관련된 코드들이 가까이 모여 있다면 이해하기가 더 쉽다.

예컨대 하나의 데이터 구조를 이용하는 문장들은 한데 모여 있어야 좋다.

실제로 나는 문장 슬라이드하기 리팩터링으로 이런 코드들을 한데 모아둔다.

가장 흔한 사례는 변수를 선언하고 사용할 때다.

모든 변수 선언을 함수 첫머리에 모아두는 사람도 있는데, 나는 변수를 처음 사용할 때 선언하는 스타일을 선호한다.

관련 코드끼리 모으는 작업은 다른 리팩터링(주로 함수 추출하기)의 준비 단계로 자주 행해진다.

관련 있는 코드들을 명확히 구분되는 함수로 추출하는 게 그저 문장을 한데로 모으는 것보다 나은 분리법이다.

하지만 코드들이 모여 있지 않다면 함수 추출은 애초에 수행할 수 조차 없다.

---

### 절차

1. 코드 조각(문장들)을 이동할 목표 위치를 찾는다. 코드 조각의 원래 위치와 목표 위치 사이의 코드들을 훑어보면서,

   조각을 모으고 나면 동작이 달라지는 코드가 있는지 살핀다. 다음과 같은 간섭이 있다면 이 리팩터링을 포기한다.

   1. 코드 조각에서 참조하는 요소를 선언하는 문장 앞으로는 이동할 수 없다.
   2. 코드 조각을 참조하는 요소의 뒤로는 이동할 수 없다.
   3. 코드 조각에서 참조하는 요소를 수정하는 문장을 건너뛰어 이동할 수 없다.
   4. 코드 조각이 수정하는 요소를 참조하는 요소를 건너뛰어 이동할 수 없다.

2. 코드 조각을 원래 위치에서 잘라내어 목표 위치에 붙여 넣는다.
3. 테스트한다.

테스트가 실패한다면 더 작게 나눠 시도해보라. 이동 거리를 줄이는 방법과 한 번에 옮기는 조각의 크기를 줄이는 방법이 있다.

---

### 예시

코드 조각을 슬라이드 할 때는 두 가지를 확인해야 한다. 무엇을 슬라이드할지와 슬라이드할 수 있는지 여부다.

무엇을 슬라이드할지는 맥락과 관련이 깊다. 가장 단순하게는, 요소를 선언하는 곳과 사용하는 곳을 가까이 두기를 좋아하는 나는

선언 코드를 슬라이드하여 처음 사용하는 곳까지 끌어내리는 일을 자주 한다.

그 외에도 다른 리팩터링을 하기 위해서는 거의 항상 코드를 슬라이드하게 된다.

예컨대 함수를 추출하기전에 추출할 코드를 한데 모을 때 적용한다.

코드 조각을 슬라이드하기로 했다면, 다음 단계로는 그 일이 실제로 가능한지를 점검해야 한다.

그러려면 슬라이드할 코드 자체와 그 코드가 건너뛰어야 할 코드를 모두 살펴본다.

이 코드들의 순서가 바뀌면 프로그램의 겉보기 동작이 달라지는가?

다음 코드를 예로 살펴보자.

```jsx
1 const pricingPlan = retrievePricingPlan();
2 const order = retrieveOrder();
3 const baseCharge = pricingPlan.base;
4 let charge;
5 const chargePerUnit = pricingPlan.unit;
6 const units = order.units;
7 let discount;
8 charge = baseCharge + units * chargePerUnit;
9 let discounttableUnit = Math.max(units - pricingPlan.discountThreshold, 0);
10 discount = discounttableUnit * pricingPlan.discountFactor;
11 if(order.isRepeat) discount += 20;
12 charge = charge - discount;
13 chargeOrder(charge);
```

처음 일곱 줄은 선언이므로 이동하기가 상대적으로 쉽다. 예컨대 할인 관련 코드를 한데 모으고 싶다면 7번 줄을 10번 줄 바로 위까지 내리면 된다.

선언은 부수효과가 없고 다른 변수를 참조하지도 않으므로 discount 자신을 참조하는 첫 번째 코드 바로 앞까지는 어디로든 옮겨도 안전하다.

이런 이동은 여러 상황에서 공통적으로 행해진다.

예컨대 할인 로직을 별도 함수로 추출하고 싶다면, 추출하기 전에 이 선언의 위치부터 옮겨줘야 한다.

부수효과가 없는 다른 코드에도 비슷한 분석을 수행해보면 2번 줄도 6번 줄 바로 위로 옮겨도 문제가 없음을 알 수 있다.

이 경우 건너뛰어지는 코드들도 부수효과가 없다는 점이 도움이 됐다.

사실 부수효과가 없는 코드끼리는 마음 가는 대로 재배치할 수 있다.

현명한 프로그래머들이 되도록 부수효과 없는 코드들로 프로그래밍하는 이유 중 하나다.

여기서 짚고 넘어갈 게 있다. 나는 2번 줄이 부수효과가 없다는 걸 어떻게 알았을까? 확실히 하려면 retrieveOrder()의 내부(그 내부의 내부의 내부도..)

를 살펴 아무 부수효과가 없음을 확인해야 한다.

하지만 나는 거의 명령-질의 분리 원칙을 지켜가며 코딩하므로, 내가 직접 작성한 코드라면 값을 반환하는 함수는 모두 부수효과가 없음을 알고 있다.

단, 코드베이스에 대해 잘 알때만 이 점을 확신할 수 있다. 잘 모르는 코드베이스에서 작업한다면 더욱 주의해야 한다.

어쨌든 사용하는 코드가 부수효과가 없음을 안다는 것의 가치는 아주 크므로,

나는 내 코드에서만이라도 항상 명령-질의 분리(Command-Query Separation) 원칙을 지키려 노력한다.

부수효과가 있는 코드를 슬라이드하거나 부수효과가 있는 코드를 건너뛰어야 한다면 훨씬 신중해야 한다.

두 코드 조각 사이에 간섭이 있는지를 확인해야 한다. 자, 11번 줄을 코드 끝으로 슬라이드하고 싶다고 해보자.

이 작업은 12번 줄 때문에 가로막히는데, 11번 줄에서 상태를 수정한 변수 discount를 12번 줄에서 참조하기 때문이다.

비슷하게, 13번 줄도 12번 줄 앞으로 이동할 수 없다. 13번 줄이 참조하는 변수 charge를 12번 줄에서 수정하기 때문이다.

하지만 8번 줄은 9~11번 줄을 건너뛸 수 있다. 이 코드들에서는 공통된 상태를 수정하는 일이 전혀 없기 때문이다.

슬라이드할 코드 조각과 건너뛸 코드 중 어느 한쪽이 다른 쪽에서 참조하는 데이터를 수정한다면 슬라이드를 할 수 없다.

이것이 가장 직관적인 규칙이다. 하지만 완벽한 규칙은 아닌 것이, 다음 두 줄은 순서를 바꿔도 안전하다.

```jsx
a = a + 10;
a = a + 5;
```

슬라이드가 안전한 지를 판단하려면 관련된 연산이 무엇이고 어떻게 구성되는지를 완벽히 이해해야 한다.

상태 갱신에 특히나 신경 써야 하기 때문에 상태를 갱신하는 코드 자체를 최대한 제거하는 게 좋다.

그래서 나라면 이 코드에 다른 어떤 슬라이드를 시도하기 앞서 charge 변수를 쪼갤 것(9.1)이다.

지금 예에서는 지역 번수만 수정하고 있으니 분석하기가 상대적으로 쉽다.

데이터 구조가 더 복잡했다면 간섭 여부를 확인하기가 훨씬 어려웠을 것이다.

그래서 테스트가 중요한 역할을 한다. 조각을 슬라이드한 후 테스트를 수행하라.

테스트 커버리지가 높다면 마음 편히 리팩터링할 수 있다.

테스트를 믿을 수 없다면 리팩터링을 더 신중히 진행한다.

혹은 (더 흔하게는) 당장의 리팩터링에 영향받는 코드의 테스트를 보강한다.

슬라이드 후 테스트가 실패했을 때 가장 좋은 대처는 더 작게 슬라이드해보는 것이다.

열 줄을 건너뛰는 대신 다섯 줄만 건너뛰거나, 위험해 보이는 줄까지만 슬라이드해보자.

테스트 실패는 그 슬라이드를 수행할 가치가 없거나, 다른 무언가를 먼저 하라는 뜻일 수도 있다.

- 조건문이 있을 때의 슬라이드
  조건문의 안팎으로 슬라이드해야 할 때도 있다.
  조건문 밖으로 슬라이드할 때는 중복이 제거될 것이고, 안으로 할때는 반대로 중복이 추가될 것이다.
  다음 조건문의 두 분기에는 똑같은 문장이 포함되어 있다.

  ```jsx
  let result;
  if (availableResources.length == 0) {
    result = createResource();
    allocatedResources.push(result);
  } else {
    result = availableResources.pop();
    allocatedResources.push(result);
  }

  return result;
  ```

  이때 중복된 문장들을 조건문 밖으로 슬라이드할 수 있는데, 조건문 블록 밖으로 꺼내는 순간 한 문장으로 합쳐진다.

  ```jsx
  let result;
  if (availableResources.length == 0) {
    result = createResource();
  } else {
    result = availableResources.pop();
  }
  allocatedResources.push(result);
  return result;
  ```

  반대의 상황, 즉 코드 조각을 조건문 안으로 슬라이드하면 조건문의 모든 분기에 복제되어 들어간다.

---

### 더 읽을거리

문장 교환하기(Swap Statement)라는 이름이 거의 똑같은 리팩터링도 있다.

문장 교환하기는 인접한 코드 조각을 이동하지만, 문장 하나짜리 조각만 취급한다.

따라서 이동할 조각과 건너뛸 조각 모두 단일 문장으로 구성된 문장 슬라이드로 생각해도 된다.

이 리팩터링도 매력적이다. 나는 항상 단계를 잘게 나눠 리팩터링하는데,

리팩터링을 처음 접하는 이들이 보기에는 어리석어 보일 정도로 작은 단계들을 밟는다.

하지만 종국에는 큰 조각을 다루는 리팩터링만 책에 싣기로 했다. 실제로 내가 그렇게 하기 때문이다.

나는 큰 슬라이드를 수행하기 어려울 때만 한 문장씩 이동한다. 큰 슬라이드에서 어려움을 겪은 일은 드물었다.

하지만 더 지저분한 코드를 정리할 때는 더 작게 슬라이드하는 편이 쉬울 것이다.

---

## 반복문 쪼개기 (Split loop)

```jsx
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
  averageAge += p.age;
  totalSalary += p.salary;
}
averageAge = averageAge / people.length;
```

```jsx
let totalSalary = 0;
for (const p of people) {
  totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
  averageAge += p.age;
}
avarageAge = averageAge / people.length;
```

### 배경

종종 반복문 하나에서 두 가지 일을 수행하는 모습을 보게 된다.

그저 두 일을 한꺼번에 처리할 수 있다는 이유에서 말이다.

하지만 이렇게 하면 반복문을 수정해야 할 때마다 두 가지 일 모두를 잘 이해하고 진행해야 한다.

반대로 각각의 반복문으로 분리해두면 수정할 동작 하나만 이해하면 된다.

반복문을 분리하면 사용하기도 쉬워진다. 한가지 값만 계산하는 반복문이라면 그 값만 곧바로 반환할 수 있다.

반면 여러 일을 수행하는 반복문이라면 구조체를 반환하거나 지역 변수를 활용해야 한다.

참고로 반복문 쪼개기는 서로 다른 일들이 한 함수에서 이뤄지고 있다는 신호일 수 있고,

그래서 반복문 쪼개기와 함수 추출하기는 연이어 수행하는 일이 잦다.

반복문을 두 번 실행해야 하므로 이 리팩터링을 불편해하는 프로그래머도 많다.

다시 한번 이야기하지만, 리팩터링과 최적화를 구분하자. 최적화는 코드를 깔끔히 정리한 이후에 수행하자.

반복문을 두 번 실행하는 게 병목이라 밝혀지면 그때 다시 하나로 합치기는 식은 죽 먹기다.

하지만 심지어 긴 리스트를 반복하도라도 병목으로 이어지는 경우는 매우 드물다.

오히려 반복만 쪼개기가 다른 더 강력한 최적화를 적용할 수 있는 길을 열어주기도 한다.

---

### 절차

1. 반복문을 복제해 두 개로 만든다.
2. 반복문이 중복되어 생기는 부수효과를 파악해서 제거한다.
3. 테스트한다.
4. 완료됐으면, 각 반복문을 함수로 추출할지 고민해본다.

---

### 예시

전체 급여와 가장 어린 나이를 계산하는 코드에서 시작해보자.

```jsx
let youngest = people[0] ? people[0].age : Infinity;
let totalSalary = 0;
for (const p of people) {
  if (p.age < youngest) youngest = p.age;
  totalSalary += p.salary;
}

return `최연소 : ${youngest}, 총 급여 : ${totalSalary}`;
```

아주 간단한 반복문이지만 관련 없는 두 가지 계산을 수행한다.

[1] 자, 반복문 쪼개기의 첫 단계는 단순히 복제하는 것이다.

```jsx
let youngest = people[0] ? people[0].age : Infinity;
let totalSalary = 0;
for (const p of people) {
  if (p.age < youngest) youngest = p.age;
  totalSalary += p.salary;
}
for (const p of people) {
  if (p.age < youngest) youngest = p.age;
  totalSalary += p.salary;
}

return `최연소 : ${youngest}, 총 급여 : ${totalSalary}`;
```

[2] 반복문을 복제했으면 잘못된 결과를 초래할 수 있는 중복을 제거해야 한다.

부수효과가 없는 코드라면 반복문 안에 그대로 둬도 되지만, 지금 예에서는 부수효과가 있으니 찾아서 없애자.

```jsx
let youngest = people[0] ? people[0].age : Infinity;
let totalSalary = 0;
for (const p of people) {
  if (p.age < youngest) youngest = p.age;
}
for (const p of people) {
  totalSalary += p.salary;
}

return `최연소 : ${youngest}, 총 급여 : ${totalSalary}`;
```

공식적인 반복문 쪼개기 리팩터링은 여기서 끝이다.

하지만 반복문 쪼개기의 묘미는 그 자체가 아닌, 다음 단계로 가는 디딤돌 역할에 있다.

[4] 이 리팩터링을 할 때는 나뉜 각 반복문을 각각의 함수로 추출하면 어떨지까지 한 묶음으로 고민하자.

지금의 경우라면 코드 일부에 문장 슬라이드하기부터 적용해야 한다.

```jsx
let youngest = people[0] ? people[0].age : Infinity;
for (const p of people) {
  if (p.age < youngest) youngest = p.age;
}
let totalSalary = 0;
for (const p of people) {
  totalSalary += p.salary;
}

return `최연소 : ${youngest}, 총 급여 : ${totalSalary}`;
```

그런 다음 각 반복문을 함수로 추출한다.

```jsx
return `최연소 : ${youngest}, 총 급여 : ${totalSalary}`;

function totalSalary() {
  let totalSalary = 0;
  for (const p of people) {
    totalSalary += p.salary;
  }
  return totalSalary;
}

function youngestAge() {
  let youngest = people[0] ? people[0].age : Infinity;
  for (const p of people) {
    if (p.age < youngest) youngest = p.age;
  }
  return youngest;
}
```

추출된 총 급여 계산 함수의 코드를 보면 반복문을 파이프라인으로 바꾸고 싶은 충동을 떨치기 어렵고,

최연소 계산 코드에는 알고리즘 교체하기를 적용하면 좋을 것 같다. 둘 다 적용해보자.

```jsx
return `최연소 : ${youngestAge()}, 총 급여 : ${totalSalary()}`;

function totalSalary() {
  return people.reduce((total, p) => total + p.salary, 0);
}

function youngestAge() {
  return Math.min(...people.map((p) => p.age));
}
```

---

## 반복문을 파이프라인으로 바꾸기(Replace Loop with Pipeline)

```jsx
const names = [];
for (const i of input) {
  if (i.job === "programer") names.push(i.name);
}
```

```jsx
const names = input.filter((i) => i.jop === "programer").map((i) => i.name);
```

### 배경

프로그래머 대부분이 그렇듯 나도 객체 컬렉션을 순회할 때 반복문을 사용하라고 배웠다.

하지만 언어는 계속해서 더 나은 구조를 제공하는 쪽으로 발전해왔다.

예컨대 이번 이야기의 주인공인 컬렉션 파이프라인을 이용하면 처리 과정을 일련의 연산으로 표현할 수 있다.

이때 각 연산은 컬렉션을 입력받아 다른 컬렉션을 내뱉는다.

대표적인 연산은 map과 filter다.

map은 함수를 이용해 입력 컬렉션의 각 원소를 변환하고, filter는 또 다른 함수를 사용해 입력 컬렉션을 필터링해 부분집합을 만든다.

이 부분집합은 파이프라인의 다음 단계를 위한 컬렉션으로 쓰인다. 논리를 파이프라인으로 표현하면 이해하기 훨씬 쉬워진다.

객체가 파이프라인을 따라 흐르며 어떻게 처리되는지를 읽을 수 있기 때문이다.

---

### 절차

1. 반복문에서 사용하는 컬렉션을 가리키는 변수 하나를 만든다.

   → 기존 변수를 단순히 복사한 것일 수도 있다.

2. 반복문의 첫 줄부터 시작해서, 각각의 단위 행위를 적절한 컬렉션 파이프라인 연산으로 대체한다.

   이때 컬렉션 파이프라인 연산은 1에서 만든 반복문 컬렉션 변수에서 시작하여,
   이전 연산의 결과를 기초로 연쇄적으로 수행된다. 하나를 대체할 때마다 테스트한다.

3. 반복문의 모든 동작을 대체했다면 반복문 자체를 지운다.

   → 반복문이 결과를 누적 변수에 대입했다면 파이프라인의 결과를 그 누적 변수에 대입한다.

---

### 예시

다음은 예시를 위한 데이터로, 내 회사의 지점 사무실 정보를 CSV 형태로 정리한 것이다.

```
office, contry, telephone
Chicago, USA, +1 312 373 1000
Beijing, China, +86 4008 900 505

...
```

다음 함수는 인도에 자리한 사무실을 찾아서 도시명과 전화번호를 반환한다.

```jsx
function acquireData(input) {
  const lines = input.split("\n"); // <- 컬렉션
  let firstLine = line;
  const result = [];
  for (const line of lines) {
    // <- 반복문
    if (firstLine) {
      firstLine = false;
      continue;
    }
    if (line.trim() === "") continue;
    const record = line.split(",");
    if (record[1].trim() === "India") {
      result.push({ city: record[0].trim(), phone: record[2].trim() });
    }
  }
  return result;
}
```

이 코드의 반복문을 컬렉션 파이프라인으로 바꿔보자.

[1] 첫 번째로 할 일은 반복문에서 사용하는 컬렉션을 가리키는 별도 변수를 새로 만드는 것이다.

이 변수를 루프 변수라 하겠다.

```jsx
function acquireData(input) {
	...
  const loopItems = lines;
  for (const line of loopItems) {
    if(firstLine) {
      firstLine = false;
      continue;
    }
	...
}
```

[2] 이 코드의 반복문에서 첫 if문은 SCV 데이터의 첫 줄을 건너뛰는 역할이다.

이 작업은 slice() 연산을 떠올리게 한다. 자, 이 slice() 연산을 루프 변수에서 수행하고 반복문 안에 if문은 제거하자.

```jsx
function acquireData(input) {
	...
  const loopItems = lines.slice(1);
  for (const line of loopItems) { // <- 반복문
    if(line.trim() === "") continue;
    const record = line.split(",");
    if(record[1].trim() === "India"){
      result.push({city : record[0].trim(), phone: record[2].trim()});
    }
  }
  return result;
}
```

이 단계에서 보너스로 firstLine 변수도 지울 수 있게 됐다. 제어용 변수를 지우는 일은 언제나 즐겁다.

반복문에서 수행하는 다음 작업은 빈 줄 지우기(trim)다. 이 작업은 filter() 연산으로 대체할 수 있다.

```jsx
function acquireData(input) {
	...
  const loopItems = lines
				.slice(1)
				.filter(line => line.trim() !== "")
				;
  for (const line of loopItems) { // <- 반복문
    const record = line.split(",");
    if(record[1].trim() === "India"){
      result.push({city : record[0].trim(), phone: record[2].trim()});
    }
  }
  return result;
}
```

파이프라인을 사용할 때는 문장 종료 세미콜론을 별도 줄에 적어주면 편하다.

다음으로 map() 연산을 사용해 여러 줄짜리 CSV 데이터를 문자열 배열로 변환한다.

수정 전 코드에서의 record라는 변수 이름은 적절치 않은데, 리팩터링을 안전하게 하기 위해 지금은 그냥 두고 나중에 수정하겠다.

```jsx
function acquireData(input) {
	...
  const loopItems = lines
				.slice(1)
				.filter(line => line.trim() !== "")
				.map(line => line.split(","))
				;
  for (const line of loopItems) { // <- 반복문
    const record = line;
    if(record[1].trim() === "India"){
      result.push({city : record[0].trim(), phone: record[2].trim()});
    }
  }
  return result;
}
```

다시 한번 filter() 연산을 수행하여 인도에 위치한 사무실 레코드를 뽑아낸다.

```jsx
function acquireData(input) {
	...
  const loopItems = lines
				.slice(1)
				.filter(line => line.trim() !== "")
				.map(line => line.split(","))
				.filter(record => record[1].trim() === "India")
				;
  for (const line of loopItems) { // <- 반복문
    const record = line;
    result.push({city : record[0].trim(), phone: record[2].trim()});
  }
  return result;
}
```

map()을 사용해 결과 레코드를 생성한다.

```jsx
function acquireData(input) {
	...
  const loopItems = lines
				.slice(1)
				.filter(line => line.trim() !== "")
				.map(line => line.split(","))
				.filter(record => record[1].trim() === "India")
				.map(record => ({city: record[0].trim(), phone: record[2].trim())
				;
  for (const line of loopItems) { // <- 반복문
    const record = line;
    result.push(line);
  }
  return result;
}
```

반복문이 하는 일은 이제 하나만 남았다. 결과를 누적 변수에 추가하는 일이다.

[3]파이프라인의 결과를 누적 변수에 대입해주면 이 코드도 제거할 수 있다.

```jsx
function acquireData(input) {
	...
  const result = lines
				.slice(1)
				.filter(line => line.trim() !== "")
				.map(line => line.split(","))
				.filter(record => record[1].trim() === "India")
				.map(record => ({city: record[0].trim(), phone: record[2].trim());
  return result;
}
```

여기까지가 이번 리팩터링의 핵심이다. 하지만 코드를 좀 더 정리해보자.

result 변수를 인라인하고, 람다 변수 중 일부의 이름을 바꾸고, 코드를 읽기 쉽도록 레이아웃을 표 형태로 정돈하면 다음처럼 된다.

```jsx
function acquireData(input) {
	const lines = input.split("\n");
  return lines
				.slice(1)
				.filter   (line => line.trim() !== "")
				.map      (line => line.split(","))
				.filter (fields => fields[1].trim() === "India")
				.map    (fields => ({city: fields[0].trim(), phone: fields[2].trim());
}
```

lines도 인라인할까 생각했지만, 그대로 두는 편이 코드가 수행하는 일을 더 잘 설명해 준다고 판단했다.

반복문을 파이프라인으로 대체하는 예를 더 보고 싶다면 내 블로그의 ‘Refactoring with Loop and Collection Pipelines’를 참고하기 바란다.

---

## 죽은 코드 제거하기(Remove Dead Code)

```jsx
if (false) {
  doSomethingThatUsedToMatter();
}
```

```jsx

```

### 배경

소프트웨어를 납품할 때, 심지어 모바일 기기용 소프트웨어라도 코드의 양에는 따로 비용을 매기지 않는다.

쓰이지 않는 코드가 몇 줄 있다고 해서 시스템이 느려지는 것도 아니고 메모리를 많이 잡아먹지도 않는다.

사실 최신 컴파일러들은 이런 코드를 알아서 제거해준다.

그렇더라도 사용되지 않는 코드가 있다면 그 소프트웨어의 동작을 이해하는 데는 커다란 걸림돌이 될 수 있다.

이 코드들 스스로는 ‘절대 호출되지 않으니 무시해도 되는 함수다’라는 신호를 주지 않기 때문이다.

그래서 운 나쁜 프로그래머는 이 코드의 동작을 이해하기 위해,

그리고 코드를 수정했는데도 기대한 결과가 나오지 않는 이유를 파악하기 위해 시간을 허비하게 된다.

코드가 더 이상 사용되지 않게 됐다면 지워야 한다. 혹시 다시 필요해질 날이 오지 않을까 걱정할 필요 없다.

우리에겐 버전 관리 시스템이 있다! 그러니 그런 날이 진짜로 온다면 그저 다시 살려내면 된다.

그런 날이 반드시 올 거라 생각된다면 어느 리비전에서 삭제했는지를 커밋 메시지로 남겨놓자.

하지만 솔직히, 내가 마지막으로 이렇게 했던 게 언제인지 기억도 나지 않으며, 이렇게 하지 않아서 후회한 기억도 없다.

한때는 죽은 코드를 주석 처리하는 방법이 널리 쓰였다. 버전 관리 시스템이 보편화되지 않았거나 아직은 쓰기 불편했던 시절엔 유용한 방법이었다.

지금은 코드가 몇 줄 안되는 초기 단계부터 버전 관리 시스템을 사용하므로, 더 이상 필요치 않다.

---

### 절차

1. 죽은 코드를 외부에서 참조할 수 있는 경우라면 (예컨대 함수 하나가 통째로 죽었을 때) 혹시라도 추출하는 곳이 있는지 확인한다.
2. 없다면 죽은 코드를 제거한다.
3. 테스트한다.

---
