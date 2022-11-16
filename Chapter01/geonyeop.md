# [1] 첫번 째 예시

전통적인 방식에 따라 리팩터링의 역사와 여러 원칙 등을 하나씩 나열하는 방식보다는

리팩터링을 실제로 해보는 예를 책 앞쪽에 배치했다.

예시 프로그램이 너무 길면 따라오기 어렵고, 너무 간단하면 리팩터링의 필요를 느끼기 어려울 것이다.

물론 이 책에 수록된 예시는 굳이 책에서 제시되는 리팩터링을 전부 적용할 필요는 없지만,

대규묘 시스템의 일부의 코드라면, 리팩터링은 필수이다.

항상 이 책의 예시 코드가 대규모 시스템에서 발췌한 코드라고 상상하며 따라오기 바란다.

이러한 연상 방식은 실전용 기법을 제한된 지면으로 설명하려 할 때 많이 활용하는 기법이다.

## 1.1 자, 시작해보자 !

예시로 사용될 프로그램은 다양한 연극을 외주로 받아 공연을 개최하는 극단에서 사용한다.

공연 요청이 들어오면 연극의 장르와 관객 규모를 기초로 비용을 책정한다.

현재 이 극단은 비극(tragedy)과 희극(comedy)만 공연한다.

공연료와 별개로 포인트를 지급해서 다음번 의뢰 시 공연료를 할인받을 수도 있다.

> plays.json (연극 정보)

```json
{
  "hamlet": { "name": "Hamlet", "type": "tragedy" },
  "as-like": { "name": "As You Like It", "type": "comedy" },
  "othello": { "name": "Othello", "type": "tragedy" }
}
```

> invoices.json (공연료 청구서)

```json
[
  {
    "customer": "BigCo",
    "performances": [
      {
        "playID": "hamlet",
        "audience": 55
      },
      {
        "playID": "as-like",
        "audience": 35
      },
      {
        "playID": "othello",
        "audience": 40
      }
    ]
  }
]
```

공연료 청구서를 출력하는 코드는 다음과 같다.

```jsx
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명 : ${invoice.customer})\n`;
  const format = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 2,
  }).format;

  for (let perf of invoice.performances) {
    const play = plays[perf.payID];
    let thisAmont = 0;

    switch (play.type) {
      case "tragedy": // 비극
        thisAmont = 40000;
        if (perf.audience > 30) {
          thisAmont += 1000 * (perf.audience - 30);
        }
        break;
      case "comedy": // 희극
        thisAmont = 30000;
        if (perf.audience > 20) {
          thisAmont += 10000 + 500 * (perf.audience - 20);
        }
        thisAmont += 300 * perf.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르 : ${play.type}`);
    }
    // 포인트를 적립한다.
    volumeCredits += Math.max(pref.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ("comedy" === play.type) volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내용을 출력한다.
    result += ` ${play.name} : ${format(thisAmont / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmont;
  }

  result += `총액 : ${format(totalAmount / 100)}\n`;
  result += `적립 포인트 : ${volumeCredits}점\n`;
  return result;
}
```

이 코드에 앞에 두 테스트 데이터 파일(json)을 입력해 실행한 결과는 다음과 같다.

```
청구 내역 (고객명 : BigCo)
	Hamlet : $650.00 (55석)
	As You Like It : $580.00 (35석)
	Othello : $500.00 (40석)
총액 : $1,730.00
적립 포인트 : 47점
```

## 1.2 예시 프로그램을 본 소감

사실 그럭저럭 쓸만하다는 생각이 들 수도 있지만,

이런 코드가 수백 줄짜리 프로그램의 일부라면 간단한 인라인 함수도 이해하기 쉽지 않다.

프로그램이 잘 작동하는 상황에서 그저 ‘지저분하다’는 이유로 불평하는 것은 너무 미적인 기준 아닐까?

컴파일러는 코드가 깔끔하든 지저분하든 개의치 않으니 말이다.

하지만 그 코드를 수정하려면 사람이 개입되고, 사람은 코드의 미적 상태에 민감하다.

설계가 나쁜 시스템은 수정하기 어렵다.

무엇을 수정할지 찾기 어렵다면 실수를 저질러서 버그가 생길 가능성도 높아진다.

그래서 나는 먼저 작동 방식을 더 쉽게 파악할 수 있도록 코드를 여러 함수와 프로그램 요소로 재구성한다.

프로그램의 구조가 빈약하다면 대체로 구조부터 바로잡은 뒤에 기능을 수정하는 편이 작업하기가 훨씬 수월하다.

```
프로그램이 새로운 기능을 추가하기에 편한 구조가 아니라면,
먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 원하는 기능을 추가한다.
```

이 코드에서 사용자의 입맛에 맞게 수정할 부분을 몇 개 발견했다.

가장 먼저 청구 내역을 HTML로 출력하는 기능이 필요하다.

우선 HTML 태그를 삽입해야 하니 청구 결과에 문자열을 추가하는 문장 각각을 조건문으로 감싸야 한다.

그러면 statement() 함수의 복잡도가 매우 증가한다.

만일, 해당 함수를 복사해서 HTML 출력 용으로 새로 만든다면,중복 코드가 발생하며 골칫거리가 될 것이다.

배우들은 사극, 전원극, 역사 비극 등 많은 장르를 연기하고 싶어 한다.

언제 어떤 연극을 할지는 아직 결정하지 못했지만, 이 변경은 공연료와 적립 포인트 계산법에 영향을 줄 것이다.

장담하건데, 어떤 방식으로 정하든 반드시 6개월 안에 다시 변경하게 될 것이다.

이처럼 연극 장르와 공연료 정책이 달라질 때마다 statement() 함수를 수정해야 한다.

만일, htmlStatement()와 같은 별도 함수를 만들었다면 모든 수정이 두 함수에 적용되어야 한다.

리팩터링이 필요한 이유는 바로 이러한 변경 때문이다. 잘 작동하고 나중에 변경할 일이 절대 없다면 아무런 문제가 없다.

하지만 다른 사람이 읽고 이해해야 할 일이 생겼는데 로직을 파악하기 어렵다면 뭔가 대책을 마련해야 한다.

## 1.3 리팩터링의 첫 단계

리팩터링의 첫 단계는 항상 똑같다. 리팩터링할 코드 영역을 꼼꼼하게 검사해줄 테스트 코드들부터 마련해야 한다.

리팩터링에서 테스트는 매우 중요하다. 실수가 발생할 수 있으니까.

statement()함수는 문자열을 반환하므로, 다양한 장르의 공연들로 구성된 공연료 청구서 몇 개를 미리 작성하며 문자열 형태로 준비해둔다.

그런 다음 statement()가 반환한 문자열과 준비해둔 정답 문자열을 비교한다.

그리고 테스트 프레임워크를 이용하여 모든 테스트를 단축키 하나로 실행할 수 있도록 설정해둔다.

여기서 중요한 부분은 테스트 결과를 보고하는 방식이다.

출력된 문자열이 정답 문자열과 똑같다면 통과의 의미로 초록불을 키고, 조금이라도 다르면 실패의 빨간불을 킨다.

즉, 성공/실패를 스스로 판단하는 자가진단 테스트로 만든다.

자가진단 여부는 매우 중요하다. 최신 프레임워크는 이러한 모든 기능을 제공한다.

나는 리팩터링 시 테스트에 상당히 의지한다. 내가 저지른 실수로부터 보호해주는 버그 검출기 역할을 해주기 때문이다.

소스 코드와 테스트 코드 양쪽에 원하는 내용을 적으면, 두 번 실수하지 않는 이상 무조건 걸린다.

테스트를 작성하는데 시간이 좀 걸리지만, 디버깅 시간이 줄어서 전체 작업 시간은 오히려 단축된다.

## 1.4 statement()함수 쪼개기

이처럼 긴 함수를 리팩터링할 때는 먼저 전체 동작을 각각의 부분으로 나눌 수 있는 지점을 찾는다.

그러면 중간 즈음의 switch문이 가장 먼저 눈에 띌 것이다.

```jsx
for (let perf of invoice.performances) {
  const play = plays[perf.payID];
  let thisAmont = 0;

  switch (play.type) { // <-- 해당 switch문
    case "tragedy": // 비극
      thisAmont = 40000;
      if (perf.audience > 30) {
        thisAmont += 1000 * (perf.audience - 30);
      }
      break;
    case "comedy": // 희극
      thisAmont = 30000;
      if (perf.audience > 20) {
        thisAmont += 10000 + 500 * (perf.audience - 20);
      }
      thisAmont += 300 * perf.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르 : ${play.type}`);
  }
```

이 switch문은 한 번의 공연에 대한 요금을 계산하고 있다.

이러한 사실은 코드를 분석해서 얻은 정보다.

워드 커닝햄(Ward Cunningham)이 말하길,

이런 식으로 파악한 정보는 휘발성이 높으므로 재빨리 코드에 반영하라 했다.

그러면 다음번에 코드를 볼 때, 다시 분석하지 않아도 코드 스스로가 자신이 하는 일이 무엇인지 이야기해 줄 것이다.

여기서는 코드 조각을 별도 함수로 추출하는 방식으로 앞서 파악한 정보를 코드에 반영할 것이다.

추출한 함수에는 그 코드가 하는 일을 설명하는 이름을 지어준다.

amountFor(aPerformance) 정도면 적당해 보인다.

이렇게 코드 조각을 함수로 추출할 때 실수를 최소화해주는 절차를 마련해뒀다.

이 절차를 따로 기록해두고, 나중에 참조하기 쉽도록 ‘함수 추출하기’란 이름을 붙였다.

먼저 별도로 함수를 빼냈을 때 유효범위를 벗어나는 변수, 즉 새 함수에서는 곧바로 사용할 수 없는 변수가 있는지 확인한다

perf, play, thisAmount가 여기 속한다.

perf, play는 추출한 새 함수에서도 필요하지만 값을 변경하지 않기 때문에 매개변수로 전달하면 된다.

한편 thisAmount는 함수 안에서 값이 바뀌는데, 이런 변수는 조심히 다뤄야 한다.

이번 예에서는 이런 변수가 하나뿐이므로 이 값을 반환하도록 작성했다.

```jsx
function amountFor(perf, play) {
  let thisAmount = 0;
  switch (play.type) {
    case "tragedy": // 비극
      thisAmont = 40000;
      if (perf.audience > 30) {
        thisAmont += 1000 * (perf.audience - 30);
      }
      break;
    case "comedy": // 희극
      thisAmont = 30000;
      if (perf.audience > 20) {
        thisAmont += 10000 + 500 * (perf.audience - 20);
      }
      thisAmont += 300 * perf.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르 : ${play.type}`);
  }

  return thisAmount;
}
```

이제, statement()에서는 thisAmount값을 채울 때 해당 함수를 호출한다.

```jsx
let thisAmount = amountFor(perf, play);
```

이렇게 수정하고 나면 곧바로 컴파일하고 테스트해서 실수한 게 없는지 확인한다.

아무리 간단한 수정이라도 리팩터링 후에는 항상 테스트하는 습관을 들여라.

한 가지를 수정할 대마다 테스트하면, 오류가 생기더라도 변경 폭이 작기 때문에 살펴볼 범위도 좁아서 해결하기가 훨씬 쉽다.

이처럼 조금씩 변경하고 매번 테스트하는 것은 리팩터링 절차의 핵심이다.

한 번에 너무 많이 수정하려다 실수를 저지르면 디버깅하기 어려워서 결과적으로 작업 시간이 늘어난다.

지금 예는 자바스크립트 코드이므로 amountFor()를 statement()의 중첩 함수로 만들 수 있었다.

이렇게 하면 바깥 함수에서 쓰던 변수를 새로 추출한 함수에 매개변수로 전달할 필요가 없어서 편하다.

지금 경우에는 달라질 게 없지만 일반적으로는 중첩 함수로 만들면 할 일 하나가 줄어드는 셈이다.

방금 수정한 사항을 테스트 해보니 문제가 없었으므로 로컬 VCS에 커밋한다.

하나의 리팩터링을 문제없이 끝낼 때마다 커밋해야 중간에 문제가 생겨도 이전의 정상 상태로 돌아갈 수 있다.

이렇게 자잘한 변경들이 어느 정도 의미 있는 단위로 뭉쳐지면 공유 저장소로 푸시한다.

**함수 추출하기**는 흔히 IDE에서 자동으로 수행해준다.

함수를 추출하고 나면 추출된 함수 코드를 자세히 들여다보면서 지금보다 명확하게 표현할 수 있는 간단한 방법은 없는지 확인한다.

가장 먼저 변수의 이름을 더 명확하게 바꿔보자. 가령 thisAmount를 result로 변경할 수 있다.

```jsx
let result = 0;
... 하위 thisAmount들도 모두 result로 변경
```

저자는 함수 반환 값에는 항상 result라는 이름을 쓴다. // 나는 result보다 명확한 변수 명이 더 좋지 않나 싶다.

그러면 그 변수의 역할을 쉽게 알 수 있다.

이번에도 마찬가지로 컴파일하고, 테스트하고, 커밋한다.

다음은 첫 번째 인수인 perf를 aPerformance로 리팩터링해보자.

```jsx
function amountFor(aPerformance, play) {
	...
}
```

이번에도 내 코딩 스타일에 따라 처리했다.

자바스크립트와 같은 동적 타입 언어를 사용할 때는 타입이 드러나게 작성하면 도움된다.

그래서 나는 매개변수 이름에 접두어로 타입 이름을 적는데,

지금처럼 매개변수의 역할이 뚜렷하지 않을 때는 부정 관사(a/an)를 붙인다.

이 방식은 켄트 백에게 배웠는데 쓰면 쓸수록 정말 유용한 것 같다.

```
컴퓨터가 이해하는 코드는 바보도 작성할 수 있다.
사람이 이해하도록 작성하는 프로그래머가 진정한 실력자다.
```

이렇게 이름을 바꿀만한 가치가 있을까? 물론이다.

좋은 코드라면 하는 일이 명확히 드러나야 하며, 이때 변수 이름은 커다란 역할을 한다.

그러니 명확성을 높이기 위한 이름 바꾸기에는 조금도 망설이지 말기 바란다.

또한 IDE에서는 이런 이름 바꾸기에 적절한 기능을 많이 제공한다.

다음으로 play 매개변수의 이름을 바꿀 차례다. 그런데 이 변수는 좀 다르게 처리해야 한다.

### play 변수 제거하기

amountFor()의 매개변수를 살펴보면서 이 값들이 어디서 오는지 알아봤다.

aPerformance는 루프 변수에서 오기 때문에 루프가 돌 때마다 값이 변경된다.

하지만 play는 aPerformance에서 얻기 때문에 애초에 매개변수로 전달할 필요가 없다.

그냥 amountFor() 안에서 다시 계산하면 된다.

난 긴 함수를 잘게 쪼갤 때마다 play같은 변수를 최대한 제거한다.

이런 임시 변수들 때문에 로컬 범위에 존재하는 이름이 늘어나서 추출 작업이 복잡해지기 때문이다.

이를 해결해주는 리팩터링으로는 **임시 변수를 질의 함수로 바꾸기가 있다.**

먼저 대입문 (=)의 우변을 함수로 추출한다.

```jsx
for (let perf of invoice.performances) {
		// const play = plays[perf.payID];
    const play = playFor(perf);
		let thisAmount = amountFor(perf, play);
...
};

function playFor(aPerformance) {
  return plays[aPerformance.playID];
}
```

컴파일 - 테스트 - 커밋한 다음 **변수 인라인하기를 적용한다.**

```jsx
for (let perf of invoice.performances) {
		let thisAmount = amountFor(perf, playFor(perf));
...
};

function playFor(aPerformance) {
  return plays[aPerformance.playID];
}
```

변수를 인라인한 덕분에 amountFor()에 **함수 선언 바꾸기**를 적용해서 play 매개변수를 제거할 수 있었다.

이 작업은 두 단계로 진행한다. 먼저 playFor()를 사용하도록 amountFor()를 수정한다.

```jsx
function amountFor(aPerformance, play) {
  let result = 0;
  switch (
    playFor(aPerformance).type // <-
  ) {
    case "tragedy": // 비극
      result = 40000;
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case "comedy": // 희극
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르 : ${playFor(aPerformance).type}`); // <-
  }

  return result;
}
```

이후 play 매개변수를 삭제한다.

```jsx
let thisAmount = amountFor(perf);

function amountFor(aPerformance) {
...
}
```

이후 컴파일 - 테스트 - 커밋한다. (위 모든 단계에서도 동일하다)

방금 수행한 리팩터링에서 주목할 점이 몇 가지 있다.

이전 코드는 루프를 한 번 돌때마다 공연을 조회했는데 반해 리팩터링한 코드에서는 세 번이나 조회한다.

뒤에서 리팩터링과 성능의 관계를 자세히 설명하겠지만, 지금 확인한 바로는 이렇게 변경해도 성능에 큰 영향은 없다.

설사 심각하게 느려지더라도 제대로 리팩터링된 코드베이스는 그렇지 않은 코드보다 성능을 개선하기가 훨씬 수월하다.

지역 변수를 제거해서 얻는 가장 큰 장점은 추출 작업이 훨씬 쉬워진다는 것이다.

유효범위를 신경 써야 할 대상이 줄어들기 때문이다.

실제로 나는 추출 작업 전에는 거의 항상 지역 변수부터 제거한다.

amountFor()에 전달할 인수를 모두 처리했으니 이 함수를 호출하는 코드로 돌아가보자.

여기서 amountFor()는 임시 변수인 thisAmount에 값을 설정하는데 사용되는데, 그 값이 다시 바뀌지 않는다.

따라서 **변수 인라인하기**를 적용한다.

```jsx
for (let perf of invoice.performances) {
  // 포인트를 적립한다.
  volumeCredits += Math.max(pref.audience - 30, 0);
  // 희극 관객 5명마다 추가 포인트를 제공한다.
  if ("comedy" === play.type) volumeCredits += Math.floor(perf.audience / 5);

  // 청구 내용을 출력한다.
  result += ` ${play.name} : ${format(amountFor(perf) / 100)} (${
    // <--
    perf.audience
  }석)\n`;
  totalAmount += amountFor(perf); // <--
}
```

### 적립 포인트 계산 코드 추출하기

지금까지 statement() 함수를 리팩터링한 결과는 다음과 같다.

```jsx
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명 : ${invoice.customer})\n`;
  const format = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 2,
  }).format;

  for (let perf of invoice.performances) {
    // 포인트를 적립한다.
    volumeCredits += Math.max(pref.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ("comedy" === play.type) volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내용을 출력한다.
    result += ` ${play.name} : ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }

  result += `총액 : ${format(totalAmount / 100)}\n`;
  result += `적립 포인트 : ${volumeCredits}점\n`;
  return result;

  function amountFor(aPerformance) {
    let result = 0;
    switch (playFor(aPerformance).type) {
      case "tragedy": // 비극
        result = 40000;
        if (aPerformance.audience > 30) {
          result += 1000 * (aPerformance.audience - 30);
        }
        break;
      case "comedy": // 희극
        result = 30000;
        if (aPerformance.audience > 20) {
          result += 10000 + 500 * (aPerformance.audience - 20);
        }
        result += 300 * aPerformance.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르 : ${playFor(aPerformance).type}`);
    }

    return result;
  }

  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }
}
```

앞에서 play 변수를 제거한 결과 로컬 유효범위의 변수가 하나 줄어서 적립 포인트 계산 부분을 추출하기가 훨씬 쉬워졌다.

여기서도 perf는 간단히 전달만 하면 된다.

하지만 volumneCredits는 루프 내에서 값을 갱신하기 때문에 살짝 더 까다롭다.

최선의 방법은 추출한 함수에서 volumeCredits의 복제본을 초기화한 뒤 계산 결과를 반환토록 하는 것이다.

```jsx
for (let perf of invoice.performances) {
  volumeCredits += volumeCreditsFor(perf);

  // 청구 내용을 출력한다.
  result += ` ${play.name} : ${format(amountFor(perf) / 100)} (${
    perf.audience
  }석)\n`;
  totalAmount += amountFor(perf);
}

result += `총액 : ${format(totalAmount / 100)}\n`;
result += `적립 포인트 : ${volumeCredits}점\n`;
return result;

function volumeCreditsFor(perf) {
  let volumeCredits = 0;
  volumeCredits += Math.max(pref.audience - 30, 0);
  if ("comedy" === playFor(perf).type)
    volumeCredits += Math.floor(perf.audience / 5);
  return volumeCredits;
}
```

새로 추출한 함수에서 쓰이는 변수들 이름을 적절히 바꾼다.

```jsx
for (let perf of invoice.performances) {
  volumeCredits += volumeCreditsFor(perf);

  // 청구 내용을 출력한다.
  result += ` ${play.name} : ${format(amountFor(perf) / 100)} (${
    perf.audience
  }석)\n`;
  totalAmount += amountFor(perf);
}

result += `총액 : ${format(totalAmount / 100)}\n`;
result += `적립 포인트 : ${volumeCredits}점\n`;
return result;

function volumeCreditsFor(perf) {
  let result = 0;
  result += Math.max(pref.audience - 30, 0);
  if ("comedy" === playFor(perf).type) result += Math.floor(perf.audience / 5);
  return result;
}
```

### format 변수 제거하기

다시 최상위 코드인 statement()를 살펴보자

```jsx
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명 : ${invoice.customer})\n`;
  const format = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 2,
  }).format;

  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);

    // 청구 내용을 출력한다.
    result += ` ${play.name} : ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }

  result += `총액 : ${format(totalAmount / 100)}\n`;
  result += `적립 포인트 : ${volumeCredits}점\n`;
  return result;
```

앞에서도 말했듯이 임시 변수는 나중에 문제를 일으킬 수 있다.

임시 변수는 자신이 속한 루틴에서만 의미가 있어서 루틴이 길고 복잡해지기 쉽다.

따라서 다음으로 할 리팩터링은 이런 변수들을 제거하는 것이다.

그 중 format은 임시변수에 함수 포인터처럼 함수를 대입한 형태인데, 나는 함수를 직접 선언해 사용하도록 바꾸는 편이다.

```jsx
function format(aNumber) {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 2,
  }).format;
}
```

그런데, format이라는 이름은 함수가 하는 일을 충분히 설명하지 않는다.

이 함수의 핵심은 화폐 단위 맞추기다. 그래서 다음과 같이 **함수 선언 바꾸기**를 적용했다.

```jsx
function statement() {
...
	// 청구 내용을 출력한다.
  result += ` ${playFor(perf).name} : ${usd(amountFor(perf))} (${perf.audience}석)\n`; // <- 이름 변경
...
}

function usd(aNumber){
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 2,
  }).format(aNumber/100); // <- 단위 변환 로직 함수 내부로 이동
}
```

이름짓기는 중요하면서 쉽지 않은 작업이다.

긴 함수를 작게 쪼개는 리팩터링은 이름을 잘 지어야만 효과가 있다.

이름이 좋으면 함수 본문을 읽지 않아도 무슨 일을 하는지 알 수 있다.

물론, 단번에 짓기는 어려우므로 당장 떠오르는 최선의 이름을 사용하다 추후 변경하는 방식이 좋다.

앞서 이름을 바꿀 때, 100으로 나누는 코드도 추출한 함수로 옮겼다.

미국에서는 금액을 흔히 센트 단위의 정수로 저장한다.

그래야 달러 미만을 표현할 때 부동소수점을 사용하지 않아도 되고 산술 연산도 쉽다.

하지만 출력할 때는 다시 달러 단위로 변환해야 하므로 use() 함수에서 나눗셈까지 처리해주면 좋다.

### volumnCredits 변수 제거하기

이 변수는 루프마다 값을 갱신하기 때문에 리팩터링하기가 까다롭다.

따라서 먼저 **반복문 쪼개기**로 volumeCredits 값이 누적되는 부분을 따로 빼낸다.

```jsx
let volumeCredits = 0; // <- 변수 선언을 사용하는 구문 앞으로 이동
for (let perf of invoice.performances) {
  volumeCredits += volumeCreditsFor(perf);
}
```

volumeCredits 값 갱신과 관련한 문장들을 한데 모아두면 임시 변수를 질의함수로 바꾸기가 수월해진다.

이번에도 역시 volumeCredits 값 계산 코드를 함수로 추출하는 작업부터 한다.

```jsx
let volumeCredits = totalVolumeCredits();
...
function totalVolumeCredits(){
  let volumeCredits = 0;
  for(let perf of invoice.performances){
    volumeCredits += volumeCreditsFor(perf);
  }
  return volumeCredits;
}
```

함수 추출이 끝났다면, 다음은 volumeCredits 변수를 인라인 할 차례다.

```jsx
function statement(invoice, plays) {
  let totalAmount = 0;
  let result = `청구 내역 (고객명 : ${invoice.customer})\n`;

  for (let perf of invoice.performances) {
    // 청구 내용을 출력한다.
    result += ` ${playFor(perf).name} : ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }

  result += `총액 : ${format(totalAmount / 100)}\n`;
  result += `적립 포인트 : ${totalVolumeCredits()}점\n`;
  return result;
...
	function totalVolumeCredits(){
    let volumeCredits = 0;
    for(let perf of invoice.performances){
      volumeCredits += volumeCreditsFor(perf);
    }
    return volumeCredits;
  }
```

여기서 잠시 멈추고 방금 한 일에 대해 생각해보자.

무엇보다 반복문을 쪼개서 성능이 느려지지 않을까 걱정할 수 있지만, 이 정도 중복은 성능에 미치는 영향이 미미할 때가 많다.

실제로 이번 리팩터링 전과 후의 실행시간을 측정해보면 차이를 거의 느끼지 못할 것이다.

경험 많은 프로그래머조차 코드의 실제 성능을 정확히 예측하지 못한다.

하지만 ‘대채로 그렇다’와 ‘항상 그렇다’는 엄연히 다르다.

때로는 리팩터링이 성능에 상당한 영향을 주기도 한다.

그런 경우라도 나는 개의치 않고 리팩터링한다. 잘 다듬어진 코드여야 성능 개선 작업도 훨씬 수월하기 때문이다.

리팩터링 과정에서 성능이 크게 떨어졌다면 리팩터링 후 시간을 내어 성능을 개선한다.

이 과정에서 리팩터링된 코드를 예전으로 되돌리는 경우도 있지만

대체로 리팩터링 덕분에 성능 개선을 더 효과적으로 수행할 수 있다.

결과적으로 더 깔끔하면서 더 빠른 코드를 얻게 된다.

따라서 리팩터링으로 인한 성능 문제에 대한 내 조언은

‘특별한 경우가 아니라면 일단 무시하라’ 라는 것이다.

리팩터링 때문에 성능이 떨어진다면, 하던 리팩터링을 마무리하고 나서 성능을 개선하자.

또 하나, volumeCredits 변수를 제거하는 작업의 단계를 아주 잘게 나눴다는 점에도 주목하자.

다음과 같이 총 네 단계로 수행했으며, 각 단계마다 컴파일-테스트하고 로컬에 커밋했다.

1. **반복문 쪼개기**로 변수 값을 누적시키는 부분을 분리한다.
2. **문장 슬라이드하기**로 변수 초기화 문장을 변수 값 누적 코드 바로 앞으로 옮긴다.
3. **함수 추출하기**로 적립 포인트 계산 부분을 별도 함수로 추출한다.
4. **변수 인라인하기**로 volumeCredits 변수를 제거한다.

항상 단계를 이처럼 잘게 나누는 것은 아니지만, 상황이 복잡해지면 단계를 더 작게 나누기도 한다.

특히 리팩터링 중간에 테스트가 실패하고 원인을 바로 찾지 못하면

가장 최근 커밋으로 돌아가서 테스트에 실패한 리팩터링의 단계를 더 작게 나눠 다시 시도한다.

커밋을 자주 했기 때문이기도 하고, 코드가 복잡할수록 단계를 나누면 작업 속도가 빨라진다.

다음으로 totalAmount도 앞에서아 똑같은 절차로 제거한다.

먼저, 반복문을 쪼개고, 변수 초기화 문장을 옮긴 다음, 함수를 추출한다.

물론 각 단계 끝에선 항상 컴파일-테스트-커밋한다.

추출할 함수 이름은 ‘totalAmount’가 가장 좋지만, 이미 같은 이름의 변수가 있어서 쓸 수 없다.

그래서 일단 아무 이름인 ‘appleSauce’를 붙혀준다.

```jsx
function statement(invoice, plays) {
  let result = `청구 내역 (고객명 : ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    result += ` ${playFor(perf).name} : ${usd(amountFor(perf))} (${perf.audience}석)\n`;
  }
  let totalAmount = appleSauce();

  result += `총액 : ${format(totalAmount / 100)}\n`;
  result += `적립 포인트 : ${totalVolumeCredits()}점\n`;
  return result;

  function appleSauce() {
    let totalAmount = 0;
    for(let perf of invoice.performances){
      totalAmount += amountFor(perf);
    }
    return totalAmount;
  }
```

이제 totalAmount 변수를 인라인한 다음, 함수 이름을 더 의미 있게 고친다.

```jsx
function statement(invoice, plays) {
  let result = `청구 내역 (고객명 : ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    result += ` ${playFor(perf).name} : ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }

  result += `총액 : ${format(totalAmount() / 100)}\n`;
  result += `적립 포인트 : ${totalVolumeCredits()}점\n`;
  return result;
...
	function totalAmount() {
    let result = 0;
    for (let perf of invoice.performances) {
      result += amountFor(perf);
    }
    return result;
  }

  function totalVolumeCredits() {
    let result = 0;
    for (let perf of invoice.performances) {
      result += volumeCreditsFor(perf);
    }
    return result;
  }
```

여기서 잠시 멈춰 서서 지금까지 리팩터링한 결과를 살펴보자.

```jsx
function statement(invoice, plays) {
  let result = `청구 내역 (고객명 : ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    result += ` ${playFor(perf).name} : ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }

  result += `총액 : ${format(totalAmount() / 100)}\n`;
  result += `적립 포인트 : ${totalVolumeCredits()}점\n`;
  return result;

  function totalAmount() {
    let result = 0;
    for (let perf of invoice.performances) {
      result += amountFor(perf);
    }
    return result;
  }

  function totalVolumeCredits() {
    let result = 0;
    for (let perf of invoice.performances) {
      result += volumeCreditsFor(perf);
    }
    return result;
  }

  function usd(aNumber) {
    return new Intl.NumberFormat("en-US", {
      style: "currency",
      currency: "USD",
      minimumFractionDigits: 2,
    }).format(aNumber / 100);
  }

  function volumeCreditsFor(perf) {
    let result = 0;
    result += Math.max(pref.audience - 30, 0);
    if ("comedy" === playFor(perf).type)
      result += Math.floor(perf.audience / 5);
    return result;
  }

  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }

  function amountFor(aPerformance) {
    let result = 0;
    switch (playFor(aPerformance).type) {
      case "tragedy": // 비극
        result = 40000;
        if (aPerformance.audience > 30) {
          result += 1000 * (aPerformance.audience - 30);
        }
        break;
      case "comedy": // 희극
        result = 30000;
        if (aPerformance.audience > 20) {
          result += 10000 + 500 * (aPerformance.audience - 20);
        }
        result += 300 * aPerformance.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르 : ${playFor(aPerformance).type}`);
    }

    return result;
  }
}
```

코드 구조가 한결 나아졌다.

최상위 statement() 함수는 단 일곱 줄뿐이며, 출력할 문장을 생성하는 일만 한다.

계산 로직은 모두 여러 개의 보조 함수로 빼냈다.

결과적으로 각 계산 과정은 물론 전체 흐름을 이해하기가 훨씬 쉬워졌다.

## 1.6 계산 단계와 포맷팅 단계 분리하기

지금까지는 프로그램의 논리적인 요소를 파악하기 쉽도록 코드의 구조를 보강하는데 주안점을 두고 리팩터링했다.

리팩터링 초기 단계에서 흔히 수행하는 일이다.

복잡하게 얽힌 덩어리를 잘게 쪼개는 작업은 이름을 잘 짓는 일만큼 중요하다.

골격은 충분히 개선됐으니 이제 statement()의 HTML 버전을 만드는 작업을 살펴보자.

여러 각도에서 볼 때 확실히 처음 코드보다 작업하기 편해졌다.

계산 코드가 모두 분리됐기 때문에 일곱 줄짜리 최상단 코드에 대응하는 HTML 버전만 작성하면 된다.

그런데, 분리된 계산 함수들이 텍스트 버전인 statement() 안에 중첩 함수로 들어가있어 문제가 된다.

텍스트 버전과 HTML 버전 함수 모두가 똑같은 계산 함수를 사용하게 만들고 싶다.

다양한 해결책 중 선호하는 방식은 **단계 쪼개기다. 내 목표는 statement()의 로직을 두 단계로 나누는 것이다.**

첫 단계에서는 statement()의 필요한 데이터 처리

다음 단계에서는 처리한 결과를 텍스트나 HTML으로 출력하는 처리

다시 말해 첫 번째 단계에서는 두 번째 단계로 전달할 중간 데이터 구조를 생성하는 것이다.

단계를 쪼개려면 먼저 두 번째 단계가 될 코드를 **함수 추출하기**로 뽑아내야 한다.

이 예에서 두 번째 단계는 청구내역을 출력하는 코드인데, 현재 statement()의 본문 전체가 해당된다.

```jsx
function statement(invoice, plays){
  return renderPlainText(invoice, plays);
}

function renderPlainText(invoice, plays) {
  let result = `청구 내역 (고객명 : ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    result += ` ${playFor(perf).name} : ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액 : ${format(totalAmount() / 100)}\n`;
  result += `적립 포인트 : ${totalVolumeCredits()}점\n`;
  return result;

  function totalAmount() {...}
  function totalVolumeCredits() {...}
  function usd(aNumber) {...}
  function volumeCreditsFor(aPerformance) {...}
  function playFor(aPerformance) {...}
  function amountFor(aPerformance) {...}
}
```

다음으로 두 단계 사이에 중간 데이터 구조 역할을 할 객체를 만들어서 renderPlainText()에 인수로 전달한다.

```jsx
function statement(invoice, plays){
  const statementData = {};
  return renderPlainText(statementData, invoice, plays);
}

function renderPlainText(data, invoice, plays) {
	...
}
```

이번에는 renderPlainText()의 두 인수를 살펴보자.

이 인수들을 통해 전달되는 데이터를 모두 방금 만든 중간 데이터 구조로 옮기면

계산 관련 코드는 전부 statement()로 모이고 renderPlainText()는 전달된 데이터만 처리하게 만들 수 있다.

가장 먼저 고객 정보부터 중간 데이터 구조로 옮긴다.

```jsx
function statement(invoice, plays){
  const statementData = {};
  statementData.customer = invoice.customer;
  return renderPlainText(statementData, invoice, plays);
}

function renderPlainText(data, invoice, plays) {
  let result = `청구 내역 (고객명 : ${data.customer})\n`;
  for (let perf of invoice.performances) {
    result += ` ${playFor(perf).name} : ${usd(amountFor(perf))} (${
      perf.audience
    }석)\n`;
  }
  result += `총액 : ${format(totalAmount() / 100)}\n`;
  result += `적립 포인트 : ${totalVolumeCredits()}점\n`;
  return result;
```

같은 방식으로 공연 정보까지 중간 데이터 구조로 옮기고 나면 renderPlainText()의 invoice를 삭제해도 된다.

```jsx
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances;
  return renderPlainText(statementData, plays);
}

function renderPlainText(data, plays) {
```

이제 연극 제목도 중간 데이터 구조에서 가져오도록 한다.

이를 위해 공연 정보 레코드에 연극 데이터를 추가해야 한다.

```jsx
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  return renderPlainText(statementData, plays);
}

function enrichPerformance(aPerformance) {
  const result = Object.assign({}, aPerformance);
  return result;
}
```

여기서는 공연 객체를 복사하기만 했지만, 잠시 후 해당 레코드에 데이터를 채울 것이다.

복사를 한 이유는 함수로 건넨 데이터를 수정하기 싫어서다.

가변 데이터는 금방 상하기 때문에 나는 데이터를 최대한 불변처럼 취급한다.

이제 연극 정보를 담을 자리가 마련됐으니 실제로 데이터를 담아보자.

이를 위해 **함수 옮기기**를 적용하여 playFor() 함수를 statement()로 옮긴다.

```jsx
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  return renderPlainText(statementData, plays);

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    return result;
  }

  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }
}
```

이제 renderPlainText() 안에서 playFor()를 호출하던 부분을 중간 데이터를 사용하도록 바꾸고,

이어서 amountFor()도 비슷한 방법으로 옮긴다.

```jsx
function enrichPerformance(aPerformance){
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    result.amount = amountFor(result);
    return result;
  }

  function playFor(aPerformance){
    return plays[aPerformance.playID];
  }

  function amountFor(aPerformance) {
		...
  }
}

function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명 : ${data.customer})\n`;
  for (let perf of data.performances) {
    result += ` ${data.play.name} : ${usd(perf.amount)} (${
      perf.audience
    }석)\n`;
  }
  result += `총액 : ${format(totalAmount() / 100)}\n`;
  result += `적립 포인트 : ${totalVolumeCredits()}점\n`;
  return result;

  function totalAmount() {
    let result = 0;
    for (let perf of data.performances) {
      result += perf.amount;
    }
    return result;
  }
```

다음으로 적립 포인트 계산 부분을 옮긴다.

```jsx
function statement(invoice, plays) {
	...

	function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    result.amount = amountFor(result);
    result.volumeCredits = volumeCreditsFor(result);
    return result;
  }
	...

	function volumeCreditsFor(aPerformance) {
    let result = 0;
    result += Math.max(pref.audience - 30, 0);
    if ("comedy" === aPerformance.play.type)
      result += Math.floor(perf.audience / 5);
    return result;
  }
}

function renderPlainText(data, plays) {

	...

	function totalVolumeCredits() {
    let result = 0;
    for (let perf of data.performances) {
      result += perf.volumeCredits;
    }
    return result;
  }
}
```

마지막으로 총합을 구하는 부분을 옮긴다.

```jsx
function statement(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  statementData.totalAmount = totalAmount(statementData);
  statementData.totalVolumeCredits = totalVolumeCredits(statementData);
  return renderPlainText(statementData, plays);
	...
	function totalAmount(data) { ... }
	function totalVolumeCredits(data) { ... }
}

function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명 : ${data.customer})\n`;
  for (let perf of data.performances) {
    result += ` ${data.play.name} : ${usd(perf.amount)} (${perf.audience}석)\n`;
  }
  result += `총액 : ${format(data.totalAmount / 100)}\n`;
  result += `적립 포인트 : ${data.totalVolumeCredits}점\n`;
  return result;

	function usd(aNumber) { ... }
}
```

이때 총합을 구하는 두 함수 totalAmount()와 totalVolumeCredits()의 본문에서

statementData 변수를 직접 사용할 수 있지만, 명확히 매개변수로 전달하는 방식을 선호하여 이렇게 수정했다.

이렇게 옮기고나니, 가볍게 **반복문을 파이프라인으로 바꾸기**까지 적용하고 싶어졌다.

```jsx
function totalAmount(data) {
  return data.performances.reduce((total, p) => total + p.amount, 0);
}

function totalVolumeCredits(data) {
  return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
}
```

이제 첫 단계인 ‘statement()에 필요한 데이터 처리’에 해당하는 코드를 모두 별도 함수로 빼낸다.

```jsx
function statement(invoice, plays){
  return renderPlainText(createStatementData(invoice, plays));
}

function createStatementData(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  statementData.totalAmount = totalAmount(statementData);
  statementData.totalVolumeCredits = totalVolumeCredits(statementData);
  return statementData;
	...
}
```

두 단계가 명확히 분리됐으니 각 코드를 별도 파일에 저장한다.

(그러면서 반환 결과를 저장할 변수 이름도 내 코딩 스타일에 맞게 바꾼다.)

> **statement.js**

```jsx
import createStatementData from "./createStatementData.js";

function statement(invoice, plays) {
  return renderPlainText(createStatementData(invoice, plays));
}

function renderPlainText(data) {
  let result = `청구 내역 (고객명 : ${data.customer})\n`;
  for (let perf of data.performances) {
    result += ` ${data.play.name} : ${usd(perf.amount)} (${perf.audience}석)\n`;
  }
  result += `총액 : ${format(data.totalAmount / 100)}\n`;
  result += `적립 포인트 : ${data.totalVolumeCredits}점\n`;
  return result;
}

function htmlStatement(invoice, plays) {
  return renderHtml(createStatementData(invoice, plays));
}

function renderHtml(data) {
  let result = `<h1>청구내역 (고객명 : ${data.customer})</h1>\n`;
  result += "<table>\n";
  result += "<tr><th>연극</th><th>좌석 수</th><th>금액</th></tr>";
  for (let perf of data.performances) {
    result += ` <tr><td>${perf.play.name}</td><td>(${perf.audience}석)</td>`;
    result += `<td>${usd(perf.amount)}</td></tr>\n`;
  }
  result += "</table>\n";
  result += `<p>총액 : <em>${format(data.totalAmount / 100)}</em></p>\n`;
  result += `<p>적립 포인트 : <em>${data.totalVolumeCredits}</em>점</p>\n`;
  return result;
}

function usd(aNumber) {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 2,
  }).format(aNumber / 100);
}
```

> **createStatementData.js**

```jsx
export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    result.amount = amountFor(result);
    result.volumeCredits = volumeCreditsFor(result);
    return result;
  }

  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }

  function amountFor(aPerformance) {
    let result = 0;
    switch (aPerformance.play.type) {
      case "tragedy": // 비극
        result = 40000;
        if (aPerformance.audience > 30) {
          result += 1000 * (aPerformance.audience - 30);
        }
        break;
      case "comedy": // 희극
        result = 30000;
        if (aPerformance.audience > 20) {
          result += 10000 + 500 * (aPerformance.audience - 20);
        }
        result += 300 * aPerformance.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르 : ${aPerformance.play.type}`);
    }
    return result;
  }

  function volumeCreditsFor(aPerformance) {
    let result = 0;
    result += Math.max(pref.audience - 30, 0);
    if ("comedy" === aPerformance.play.type)
      result += Math.floor(perf.audience / 5);
    return result;
  }

  function totalAmount(data) {
    return data.performances.reduce((total, p) => total + p.amount, 0);
  }

  function totalVolumeCredits(data) {
    return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
  }
}
```

처음보다 코드량이 부쩍 늘었다.

원래 44줄짜리 코드가 지금은 htmlStatement()를 빼고도 70줄이나 된다.

늘어난 주된 원인은 함수로 추출하면서 함수 본문을 열고 닫는 괄호가 덧붙었기 때문이다.

그 외에 달라진 점이 없다면 안 좋은 징조지만, 다행히 그렇지는 않다.

추가된 코드 덕분에 전체 로직을 구성하는 요소 각각이 더 뚜렷이 부각되고,

계산하는 부분과 출력 형식을 다루는 부분이 분리됐다.

이렇게 모듈화하면 각 부분이 하는 일과 그 부분들이 맞물아 돌아가는 과정을 파악하기 쉬워진다.

간결함이 지혜의 정수일지 몰라도, 프로그래밍에서만큼은 명료함이 진화할 수 있는 소프트웨어의 징수다.

모듈화한 덕분에 계산 코드를 중복하지 않고도 HTML 버전을 만들 수 있었다.

```jsx
캠핑자들에게는 "도착했을 때보다 깔끔하게 정돈하고 떠난다"는 규칙이 있다.
프로그래밍도 마찬가지다. 항시 코드베이스를 작업 시작 전보다 건강하게 만들어놓고 떠나야 한다.
- 보이스카우트 규칙
```

출력 로직을 더 간결하게 만들 수도 있지만 일단은 이 정도에서 멈추겠다.

나는 항상 리팩터링과 기능 추가 사이의 균형을 맞추려고 한다.

현재 코드에서는 리팩터링이 절실하게 느껴지지 않을 수 있지만 어느정도 균형점을 잡을 수 있다.

‘항시 코드베이스를 작업 전보다 더 건강하게 고친다’라는 캠핑 규칙의 변형 버전을 적용한다.

완벽하지는 않더라도, 분명 더 나아지게 한다.

## 1.8 다형성을 활용해 계산 코드 재구성하기

이번에는 연극 장르를 추가하고 장르마다 공연료와 적립 포인트 계산법을 다르게 지정하도록 기능을 수정해보자.

현재 상테에서 코드를 변경하려면 이 계산을 수행하는 함수에서 조건문을 수정해야 한다.

amountFor() 함수를 보면 연극 장르에 따라 계산 방식이 달라진다는 사실을 알 수 있는데,

이런 형태의 조건부 로직은 코드 수정 횟수가 늘어날수록 골칫거리로 전락하기 쉽다.

이를 방지하려면 프로그래밍 언어가 제공하는 구조적인 요소로 적절히 보완해야 한다.

조건부 로직을 명확한 구조로 보완하는 방법은 다양하지만, 여기서는 다형성을 활용하는 것이 자연스럽다.

자바스크립트 커뮤니티에서 전통적인 객체지향 지원은 오랫동안 논란거리였다.

그러다 ECMAScript 2015(ES6)부터 객체지향을 사용할 수 있는 문법과 구조가 제대로 지원되기 시작했다.

따라서 딱 맞는 상황이라면 이런 기능을 적극 활용하는 것이 좋다.

이번 작업의 목표는 상속 계층을 구성해서

희극 서브클래스와 비극 서브클래스가 각자의 구체적인 계산 로직을 정의하는 것이다.

호출하는 쪽에서는 다형성 버전의 공연료 계산 함수를 호출하기만 하면 되고,

희극이냐 비극이냐에 따라 계산 로직을 연결하는 작업은 언어 차원에서 처리해준다.

적립 포인트 계산도 비슷한 구조로 만들 것이다.

이 과정에서 몇 가지 리팩터링 기법을 적용하는데, 그 중 핵심은 **조건부 로직을 다형성으로 바꾸기**다.

이 리팩터링은 조건부 코드 한 덩어리를 다형성으로 활용하는 방식으로 바꿔준다.

그런데 이 리팩터링을 적용하려면 상속 계층부터 정의해야 한다.

즉, 공연료와 적립 포인트 계산 함수를 담을 클래스가 필요하다.

### 공연료 계산기 만들기

여기서 핵심은 각 공연의 정보를 중간 데이터 구조에 채워주는 enrichPerformance() 함수다.

현재 이 함수는 조건부 로직을 포함한 함수인 amountFor()와 volumeCreditsFor()를 호출하여 계산한다.

이번에 할 일은 두 함수를 전용 클래스로 옮기는 작업이다.

이 클래스는 공연 관련 데이터를 계산하는 함수들로 구성되므로 PerformanceCalculator라 부르기로 하자.

```jsx
class PerformanceCalculator {
  constructor(aPerformance){
    this.aPerformance = aPerformance;
  }
}

export default function createStatementData(invoice, plays) {
	...
	function enrichPerformance(aPerformance) {
    const calculator = new PerformanceCalculator(aPerformance);
		...
  }
	...
}
```

아직 이 클래스의 객체로 할 수 있는 일은 없다.

기존 코드에서 몇 가지 동작을 이 클래스로 옮겨보자.

먼저 가장 간단한 연극 레코드부터 시작하자.

사실 이 작업은 다형성을 사용할만큼 차이가 크지 않으나,

모든 데이터 변환을 한 곳에서 수행할 수 있어서 코드가 더욱 명확해진다.

이를 위해 계산기 클래스의 생성자에 **함수 선언 바꾸기**를 적용하여 공연할 연극을 계산기로 전달한다.

```jsx
class PerformanceCalculator {
  constructor(aPerformance, aPlay){
    this.performance = aPerformance;
		this.play = aPlay;
  }
}

export default function createStatementData(invoice, plays) {
	...
	function enrichPerformance(aPerformance) {
    const calculator =
			new PerformanceCalculator(aPerformance, playFor(aPerformance));
		...
  }
	...
}
```

여기서 한가지, 컴파일-테스트-커밋은 기회가 될 때마다 무조건 실행하자.

나도 귀찮아서 소홀해질 때가 있지만, 그러다 실수하면 다시 꼬박꼬박 수행하게 된다.

### 함수들을 계산기로 옮기기

그 다음 옮길 로직은 공연료 계산에 더 중요하다.

지금까지는 중첩 함수를 재배치하는 것이어서 함수를 옮기는데 부담이 없었다.

하지만 이번에는 함수를 다른 컨텍스트로 옮기는 큰 작업이다.

그러니 이번에는 **함수 옮기기** 리팩터링으로 작업을 단계별로 차근차근 진행해보자.

가장 먼저 할 일은 공연료 계산 코드를 계산기 클래스 안으로 복사하는 것이다.

그런 다음 이 코드가 잘 동작하도록 aPerformance를 this.performance로 바꾸고

playFor(aPerformance)를 this.play로 바꿔준다.

```jsx
class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }

  get amount() {
    let result = 0;
    switch (this.play.type) {
      case "tragedy": // 비극
        result = 40000;
        if (this.performance.audience > 30) {
          result += 1000 * (this.performance.audience - 30);
        }
        break;
      case "comedy": // 희극
        result = 30000;
        if (this.performance.audience > 20) {
          result += 10000 + 500 * (this.performance.audience - 20);
        }
        result += 300 * this.performance.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르 : ${this.play.type}`);
    }
    return result;
  }
}
```

복사한 함수가 동작하게끔 수정했다면 원본 함수가 방금 만든 함수로 작업을 위임하도록 바꾼다.

```jsx
export default function createStatementData(invoice, plays) {
	...
	function amountFor(aPerformance) {
    return new PerformanceCalculator(aPerformance, playFor(aPerformance)).amount;
  }
	...
}
```

그런 다음 코드가 제대로 작동하는지 확인하고,

문제 없다면 **함수를 인라인**하여 새 함수를 직접 호출하도록 수정한다.

```jsx
export default function createStatementData(invoice, plays) {
	...
	function enrichPerformance(aPerformance) {
    const calculator = new PerformanceCalculator(
      aPerformance,
      playFor(aPerformance)
    );
    const result = Object.assign({}, aPerformance);
    result.play = calculator.play;
    result.amount = calculator.amount;
    result.volumeCredits = volumeCreditsFor(result);
    return result;
  }
	...
}
```

적립 포인트를 계산하는 함수도 같은 방법으로 옮긴다.

```jsx
class PerformanceCalculator {
	...
	get volumeCredits() {
    let result = 0;
    result += Math.max(this.performance.audience - 30, 0);
    if ("comedy" === this.play.type)
      result += Math.floor(this.performance.audience / 5);
    return result;
  }
}

export default function createStatementData(invoice, plays) {
	...
	function enrichPerformance(aPerformance) {
		...
    result.amount = calculator.amount;
    result.volumeCredits = calculator.volumeCredits;
		...
  }
	...
}
```

### 공연료 계산기를 다형성 버전으로 만들기

클래스에 로직을 담았으니 이제 다형성을 지원하게 만들어보자.

가장 먼저 할 일은 타입코드 대신 서브클래스를 사용하도록 변경하는 것이다.

(**타입 코드를 서브클래스로 바꾸기**)

PerformanceCalculator의 서브 클래스들을 준비하고

createStatementData()에서 그 중 적합한 서브클래스를 사용하게 만들어야 한다.

그리고 딱 맞는 서브클래스를 사용하려면 생성자 대신 함수를 호출하도록 바꿔야 한다.

자바스크립트에서는 생성자가 서브클래스의 인스턴스를 반환할 수 없기 때문이다.

그래서 **생성자를 팩터리 함수로 바꾸기**를 적용한다.

```jsx
function createPerformanceCalculator(aPerformance, aPlay){
  return new PerformanceCalculator(aPerformance, aPlay);
}

class PerformanceCalculator{ ... }

export default function createStatementData(invoice, plays) {
	...
  function enrichPerformance(aPerformance) {
    const calculator = createPerformanceCalculator(
      aPerformance,
      playFor(aPerformance)
    );
		...
	}
	...
}
```

함수를 이용하면 다음과 같이 PerformanceCalculator의 서브클래스 중에서

어느 것을 생성해서 반환할지 선택할 수 있다.

```jsx
function createPerformanceCalculator(aPerformance, aPlay) {
  switch (aPlay.type) {
    case "tragedy":
      return new TragedyCalculator(aPerformance, aPlay);
    case "comedy":
      return new ComedyCalculator(aPerformance, aPlay);
    default:
      throw new Error(`알 수 없는 장르 : ${aPlay.type}`);
  }
}

class TragedyCalculator extends PerformanceCalculator {}

class ComedyCalculator extends PerformanceCalculator {}
```

이제 다형성을 지원하기 위한 구조는 갖춰졌다.

다음은 **조건부 로직을 다형성**으로 바꾸기를 적용할 차례다.

비극 공연의 공연료 계산부터 시작해보자.

```jsx
class TragedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 40000;
    if (this.performance.audience > 30) {
      result += 1000 * (this.performance.audience - 30);
    }
    return result;
  }
}
```

이 메서드를 서브클래스에 정의하기만 해도

슈퍼클래스(PerformanceCalculator)의 조건부 로직이 오버라이드된다.

하지만 나처럼 편집증이 있는 프로그래머라면 다음과 같이 작성할 것이다.

```jsx
class PerformanceCalculator {
	...
	get amount() {
		let result = 0;
		switch(this.play.type) {
			case "tragedy" :
				throw '오류 발생'; // <- 비극 공연료는 TragedyCalculator를 사용하게 유도
			...
		}
		...
	}
}
```

비극 담당 case문을 제거하면, default 절에서 에러를 받지만, 명시적으로 던지는 방식을 좋아한다.

이후 , 희극 공연료 계산 코드도 옮긴다.

```jsx
class ComedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 30000;
    if (this.performance.audience > 20) {
      result += 10000 + 500 * (this.performance.audience - 20);
    }
    result += 300 * this.performance.audience;
    return result;
  }
}
```

이제 슈퍼클래스의 amount() 메서드는 호출할 일이 없으니 삭제해도 된다.

그래도 미래의 나에게 한 마디 남겨놓는 게 좋을 것 같다.

```jsx
class PerformanceCalculator {
	...
	get amount() {
			throw '서브클래스에서 처리하도록 설계되었습니다.';
		}
		...
	}
}
```

다음으로는 적립 포인트를 계산하는 부분을 교체해야 한다.

향후 제공할 가능성이 있는 연극 장르들을 검토한 결과,

일부 장르에서만 약간씩 다를 뿐 대다수의 연극은 관객 수가 30을 넘는지 확인한다.

이럴 때는 일반적인 경우를 기본으로 삼아 슈퍼클래스에 남겨두고

장르마다 달라지는 부분은 필요할 때 오버라이드하게 만드는 것이 좋다.

```jsx
class PerformanceCalculator {
	...
	get volumeCredits() {
		return Math.max(this.performance.audience - 30, 0);
	}
}

class ComedyCalculator extends PerformanceCalculator {
	...
	get volumeCredits() {
		return super.volumeCredits + Math.floor(this.performance.audience / 5);
	}
}
```

## 1.9 상태 점검 : 다형성을 활용하여 데이터 생성하기

다형성을 추가한 결과로 무엇이 달라졌는지 살펴볼 시간이다.

```jsx
export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;

  function enrichPerformance(aPerformance) {
    const calculator = createPerformanceCalculator(
      aPerformance,
      playFor(aPerformance)
    );
    const result = Object.assign({}, aPerformance);
    result.play = calculator.play;
    result.amount = calculator.amount;
    result.volumeCredits = calculator.volumeCredits;
    return result;
  }

  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }

  function totalAmount(data) {
    return data.performances.reduce((total, p) => total + p.amount, 0);
  }

  function totalVolumeCredits(data) {
    return data.performances.reduce((total, p) => total + p.volumeCredits, 0);
  }
}

function createPerformanceCalculator(aPerformance, aPlay) {
  switch (aPlay.type) {
    case "tragedy":
      return new TragedyCalculator(aPerformance, aPlay);
    case "comedy":
      return new ComedyCalculator(aPerformance, aPlay);
    default:
      throw new Error(`알 수 없는 장르 : ${aPlay.type}`);
  }
}

class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }

  get amount() {
    throw new Error("서브클래스에서 처리하도록 설계되었습니다.");
  }

  get volumeCredits() {
    return Math.max((this.performance.audience = 30), 0);
  }
}

class TragedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 40000;
    if (this.performance.audience > 30) {
      result += 1000 * (this.performance.audience - 30);
    }
    return result;
  }
}

class ComedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 30000;
    if (this.performance.audience > 20) {
      result += 10000 + 500 * (this.performance.audience - 20);
    }
    result += 300 * this.performance.audience;
    return result;
  }

  get volumeCredits() {
    return super.volumeCredits + Math.floor(this.performance.audience / 5);
  }
}
```

앞에서 함수를 추출했을 때처럼, 이번에도 구조를 보강하면서 코드가 늘어났다.

연극 장르별 계산 코드처럼 수정 대부분이 특정 코드에서 이뤄질 것 같으면

이렇게 명확히 분리하여 묶어두는 것이 좋다.

이제 새로운 장르를 추가하려면 createPerformanceCalculator()에 추가하기만 하면 된다.

이번 예를 통해 서브클래스를 언제 사용하면 좋을지 감이 잡힐 것이다.

여기서는 amountFor()와 volumeCreditsFor()의 조번구 로직 생성 함수를 하나로 옮겼다.

같은 타입의 다형성을 기반으로 실행되는 함수가 많을수록 이렇게 구성하는 쪽이 유리하다.

계산기가 중간 데이터 구조를 채우게 한 지금의 코드와 달리

createStatementData()가 계산기 자체를 반환하게 구현해도 된다.

두 방법 중 나는 결과로 나온 데이터 구조를 누가 사용하는가를 기준으로 결정했다.

이번 예에서는 다형성 계산기를 사용한 사실을 숨기기보다는

중간 데이터 구조를 이용하는 방법을 보여주는 편이 낫다고 생각해서 이렇게 작성했다.

## 1.10 마치며

간단한 예였지만 리팩터링이 무엇인지 감을 잡았길 바란다.

함수 추출하기, 변수 인라인하기, 함수 옮기기, 조건부 로직을 다형성으로 바꾸기 등

다양한 리팩터링 기법을 선보였다.

이번 장에서는 리팩터링을 크게 세 단계로 진행했다.

1. 원본 함수를 중첩 함수 여러개로 나눈다.
2. 단계 쪼개기를 적용해서 계산 코드와 출력 코드를 분리한다.
3. 계산 로직을 다형성으로 표현한다.

각 단계에서 코드 구조를 보강했고, 그럴 때마다 코드가 수행하는 일이 더욱 분명하게 드러났다.

리팩터링은 대부분 코드가 하는 일을 파악하는 데서 시작한다.

코드를 읽고, 개선점을 찾고, 리팩터링 작업을 통해 개선점을 코드에 반영한다.

그 결과 코드가 명확해지고 이해하기 더 쉬워진다.

그러면 또 다른 개선점이 떠오르며 선순환이 형성된다.

지금 수정한 코드에도 개선할 게 몇 가지 더 있지만, 이정도면 원본 코드를 크게 개선한다는 목표는 달성했다.

```
좋은 코드를 가늠하는 확실한 방법은 "얼마나 수정하기 쉬운가"다.
```

이 책은 코드를 개선하는 방법을 다룬다.

그런데 프로그래머 사이에서 어떤 코드가 좋은 코드인지에 대한 의견은 분분하다.

내가 선호하는 ‘적절한 이름의 작은함수들’로 만드는 방식에 반대하는 사람도 분명 있을 것이다.

미적 관점으로 접근하면 개인 취향 말고는 어떠한 지침도 세울 수 없게 된다.

하지만, 취향을 넘어서는 관점이 분명 존재하며, 코드를 ‘수정하기 쉬운 정도’야 말로 그 방법이라 생각한다.

코드는 명확해야하며, 수정해야할 때는 수정할 곳을 쉽게 찾고 오류 없이 빠르게 수정해야 한다.

이번 예시의 가장 중요한 배움은 리팩터링하는 리듬이다.

리팩터링하는 과정을 보여줄 때마다,

각 단계를 잘게 나누고 컴파일-테스트하여 작동하는 상태로 유지한다는 사실에 놀란다.

단계를 잘게 나눠야 더 빠르게 처리할 수 있고, 코드는 절대 깨지지 않으며

이러한 작은 단계들이 모여서 상당히 큰 변화를 이룰 수 있다는 사실을 깨닫는 것이다.

이 점을 명심하고 그대로 따라주기 바란다.

- 정리
  1. 프로그램이 새로운 기능을 추가하기에 편한 구조가 아니라면, 먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 기능을 추가하라.
  2. 리팩터링에서 테스트는 매우 중요하다. 테스트를 작성하는데 시간이 좀 걸리지만, 디버깅 시간이 줄어서 전체 작업 시간은 오히려 단축된다.
  3. 특정 구문이 코드를 분석해야만 해석할 수 있다면, 함수로 추출하라.
  4. 저자는 함수 반환 값을 result로 통일한다. 처음에는, amount와 같은 이름이 더 명확하지 않을까 싶었지만,
     만일, 복잡한 로직에서 amount를 반환한다면, 독자는 return문을 봐야만 amount 변수가 반환 값이라는 사실을 알 것이다.
  5. [1.4 - play 변수 제거하기]에서 함수 내에 play값을 2번 사용하는데, playFor() 함수를 2번 호출하는 것보다, play 변수를 만들어서 사용하는 게 좋다고 본다.
  6. 리팩터링 시 성능은 특별한 경우가 아니면 일단 무시하라 성능이 떨어졌다면, 리팩터링을 마무리 후 개선하자.
  7. 반복문을 파이프라인으로 바꾸기(reduce)
  8. 항시 코드베이스를 작업 전보다 더 건강하게 고친다. ( - 보이 스카웃 규칙 )
  9. 변경이 잦고 분기처리가 많은 경우, 다형성으로 처리하라
  10. 좋은 코드를 가늠하는 확실한 방법은 “얼마나 수정하기 편한가” 이다.
