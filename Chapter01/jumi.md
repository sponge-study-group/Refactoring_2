## 1. 리팩터링: 첫 번째 예시

---

### 1.1 자. 시작해보자

<br />

```javascript
{
  "hamlet": {name: "Hamlet", type: "tragedy"},
  "as-like": {name: "As You Like It", type: "comedy"},
  "othello": {name: "Othello", type: "tragedy"},
}
```

play.json

<br />

```javascript
[
  {
    customer: 'BigCo',
    perfomences: [
      {
        playID: 'hamlet',
        audience: 55,
      },
      {
        playID: 'as-like',
        audience: 35,
      },
    ],
  },
];
```

invoices.json

<br />

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', minimumFractionDigits: 2 }).format;

  for (let perf of invoice.performances) {
    const play = plays[perf.playID];
    let thisAmount = 0;

    switch (play.type) {
      case 'tragedy': // 비극
        thisAmount = 40000;
        if (perf.audience > 30) {
          thisAmount += 1000 * (perf.audience - 30);
        }
        break;
      case 'comedy': // 희극
        thisAmount = 30000;
        if (perf.audience > 20) {
          thisAmount += 10000 + 500 * (perf.audience - 20);
        }
        thisAmount += 300 * perf.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`);
    }

    //포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);

    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ('comedy' === play.type) volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    result += `${play.name}: ${format(thisAmount / 100)} (${perf.audience}석)\n`;
    totalAmount += thisAmount;
  }

  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;

  return result;
}
```

<br/>

---

<br/>

### 1.2 예시 프로그램을 본 소감

<br/>

프로그램이 새로운 기능을 추가하기에 편한 구조가 아니라면,  
먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 원하는 기능을 추가한다.  
👉 정책이 복잡해질수록 수정할 부분을 찾기 어려워지고 수정 과정에서 실수할 가능성도 커진다.

<br/>

❗️ 리팩터링이 필요한 이유  
다른 사람이 읽고 이해해야 할 일이 생겼는데 로직을 파악하기 어렵다면 대책이 필요하기 때문.

<br/>

---

<br/>

### 1.3 리팩터링의 첫 단계

<br/>

❗️ 리팩터링의 첫 단계  
리팩터링 할 코드 영역을 꼼꼼하게 검사해줄 테스트 코드들부터 마련  
👉 성공/실패를 스스로 판단하는 자가진단 테스트를 만든다.

<br/>

❗️ 간단한 수정이라도 리팩터링 후에는 항상 테스트하는 습관을 들이자!  
한번에 너무 많이 수정하려다 실수를 저지르면, 디버깅하기 어려워서 결과적으로 작업 시간이 늘어난다.  
👉 조금씩 수정하여 피드백 주기를 짧게 가져가는 습관을 통해 이러한 재앙을 피하자.

<br/>

---

<br/>

### 1.4 statement() 함수 쪼개기

- 함수 추출하기

```javascript
function statement(invoice, plays) {
  ...

  for (let perf of invoice.performances) {
    const play = plays[perf.playID];
    let thisAmount = amountFor(perf, play); // 추출한 함수 사용

    //포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);

     ...
  }
  ...

  function amountFor(aPerformance, play) {
    let result = 0;

    switch (play.type) {
      case 'tragedy': // 비극
        result = 40000;
        if (aPerformance.audience > 30) {
          result += 1000 * (aPerformance.audience - 30);
        }
        break;
      case 'comedy': // 희극
        result = 30000;
        if (aPerformance.audience > 20) {
          thisAmount += 10000 + 500 * (aPerformance.audience - 20);
        }
        result += 300 * aPerformance.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`);
      }

      return result;
  }
}
```

`수정하고 나면 곧바로 컴파일하고 테스트해서 실수한게 없는지 확인!`  
👉 한 가지를 수정할 떄마다 테스트하면, 오류가 생기더라도 변경 폭이 작기 때문에 살펴볼 범위도 좁아서  
문제를 찾고 해결하기가 훨씬 쉽다.

> 이처럼 조금씩 변경하고 매번 테스트하는 것은 리팩터링 절차의 핵심이다.

<br/>

- 임시 변수를 질의 함수로 바꾸기

```javascript
function statement(invoice, plays) {
  ...

  for (let perf of invoice.performances) {
    const play = playFor(perf); // 함수로 추출
    let thisAmount = amountFor(perf, play);

    ...
  }
  ...

  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }
}
```

<br/>

- 변수 인라인하기

```javascript
function statement(invoice, plays) {
  ...

  for (let perf of invoice.performances) {
    // const play = playFor(perf);
    // let thisAmount = amountFor(perf);

    ...

    // 청구 내역을 출력한다.
    result += `${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${perf.audience}석)\n`;
    totalAmount += amountFor(perf);
  }
    ...

  function amountFor(aPerformance) {
    let result = 0;

    switch (playFor(aPerformance).type) {
      ...

      default:
        throw new Error(`알 수 없는 장르: ${playFor(aPerformance).type}`);
    }

    return result;
  }
}
```

❗️ 리팩터링한 코드에서는 공연을 3번이나 조회하는데 성능상 문제는 없을까?  
👉 이렇게 변경해도 성능에 큰 영향은 없다. 설사 심각하게 느려지더라도 제대로 리팩터링된 코드베이스는 그렇지 않은 코드보다 성능을 개선하기가 훨씬 수월하다.

<br/>

- 적립 포인트 계산 코드 추출

```javascript
function statement(invoice, plays) {
    ...

    for (let perf of invoice.performances) {
      volumeCredits += volumeCreditsFor(perf);  // 추출한 함수를 이용해 값 누적

      // 청구 내역을 출력한다.
      result += `${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${perf.audience}석)\n`;
      totalAmount += amountFor(perf);
    }

    ...

  function volumeCreditsFor(aPerformance) {
    let result = 0;
    result += Math.max(aPerformance.audience - 30, 0);

    if ("comedy" === playFor(aPerformance).type) result += Math.floor(aPerformance.audience / 5);

    return result;
  }
}
```

<br/>

- format 변수 제거하기

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  // const format = new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', minimumFractionDigits: 2 }).format;

  ...

  result += `총액: ${usd(totalAmount)}\n`; // 임시 변수였던 format을 함수 호출로 대체

  ...

}

function usd(aNumber) {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', minimumFractionDigits: 2 }).format(aNumber / 100);
}
```

1. 함수로 추출하기
2. format 함수 선언 변경(usd)

<br/>

> 임시 변수는 자신이 속한 루틴에서만 의미가 있어서 루틴이 길고 복잡해지기 쉽다.

> 처음에는 당장 떠오르는 최선의 이름을 사용 👉 나중에 더 좋은 이름이 떠오를 때 변경

<br/>

- volumeCredits 변수 제거

<br/>

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  // let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;

  ...

  // let volumeCredits = totalVolumeCredits(); // 변수 선언을 반복문 앞으로 이동(문장 슬라이드 하기)

  // 값 누적 로직을 별도의 for문으로 분리
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }

  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;

  return result;

  function totalVolumeCredits() {
    let result = 0;

    for (let perf of invoice.performances) {
      result += volumeCreditsFor(perf);
    }

    return result;
  }

  ...
}
```

1. 반복문 쪼개기
2. 문장 슬라이드 하기
3. 함수로 추출하기
4. 변수 인라인하기

<br/>

Q. 반복문을 쪼개서 성능이 떨어지지 않을까?  
👉 이 정도 중복은 성능에 미치는 영향이 미미할 때가 많다.  
똑똑한 컴파일러들은 최신 캐싱 기법 등으로 무장하고 있어서 우리의 직관을 초월하는 결과를 내어주기 때문.

👉 리팩터링 과정에서 성능이 크게 떨어졌다면, `리팩터링 후 성능을 개선`한다.  
결과적으로 더 깔끔하면서 더 빠른 코드를 얻게 된다.

<br/>

- totalAmount 변수 제거

```javascript
function statement(invoice, plays) {
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;

  for (let perf of invoice.performances) {
    result += `${playFor(perf).name}: ${usd(amountFor(perf))} (${perf.audience}석)\n`;
  }

  result += `총액: ${usd(totalAmount())}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;

  return result;

  function totalAmount() {
    let result = 0;

    for (let perf of invoice.performances) {
      result += amountFor(perf);
    }

    return result;
  }

  ...
}
```

volumeCredits 변수 제거를 제거할 떄와 동일한 절차로 제거.

`각 단계가 끝날때마다 컴파일 - 테스트 - 커밋`

<br/>

---

<br/>

### 1.5 중간 점검: 난무하는 중첩 함수

<br/>

계산 로직은 모두 여러 개의 보조 함수로 빼냈다.  
👉 각 계산 과정은 물론 전체 흐름을 이해하기가 훨씬 쉬워졌다.

<br/>

---

<br/>

### 1.6 계산 단계와 포맷팅 단게 분리하기

<br/>

- 단계 쪼개기

첫 단계에서는 statement()에 필요한 데이터를 처리,  
다음 단계에서는 앞서 처리한 결과를 텍스트나 HTML로 표현.

```javascript
function statement(invoice, plays) {
  return renderPlainText(createStatementData(invoice, plays)); // 본문 전체를 별도 함수로 추출
}

function renderPlainText(data, plays) {
  let result = `청구 내역 (고객명: ${data.customer})\n`;

  for (let perf of data.performances) {
    result += `${perf.play.name}: ${usd(perf.amount)} (${perf.audience}석)\n`;
  }

  result += `총액: ${usd(data.totalAmount())}\n`;
  result += `적립 포인트: ${data.totalVolumeCredits()}점\n`;

  return result;
}

function htmlStatement(invoice, plays) {
  return renderHtml(createStatementData(invoice, plays));
}

function renderHtml(data) {
  let result = `<h1>청구 내역 (고객명: ${data.customer})</h1>\n`;
  result += '<table>\n';
  result += '<tr><th>연극</th><th>좌석 수</th><th>금액</th></tr>';

  for (let perf of data.performances) {
    result += ` <tr><td>${perf.play.name}</td><td>(${perf.audience}석)</td>`;
    result += `<td>${usd(perf.amount)}</td></tr>\n`;
  }

  result += '</table>\n';
  result += `<p>총액: <em>${usd(data.totalAmount)}</em></p>\n`;
  result += `<p>적립 포인트: <em>${data.totalVolumeCredits}</em>점</p>\n`;

  return result;
}
```

statement.js

<br/>

```javascript
export default function createStatementData(invoice, plays) {
  const statementData = {}; // 중간 데이터 구조 역할을 할 객체

  statementData.customer = invoice.customer; // 고객 정보
  statementData.performances = invoice.performances.map(enrichPerformance); // 공연 정보

  statementData.totalAmount = totalAmount(statementData);
  statementData.totalVolumeCredits = totalVolumeCredits(statementData);

  return statementData;

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance); // 얕은 복사 수행

    result.play = playFor(result);
    result.amount = amountFor(result);
    result.volumeCredits = volumeCreditsFor(result);

    return result;
  }

  function totalAmount(data) {
    return data.performances.reduce((total, p) => total + p.amount, 0); // 기존 for문을 파이프라인으로 변경
  }

  function totalVolumeCredits(data) {
    return data.performances.reduce((total, p) => total + p.volumeCredits, 0); // 기존 for문을 파이프라인으로 변경
  }

  function playFor(aPerformance) { ... }
  function amountFor(aPerformance) { ... }
  function volumeCreditsFor(aPerformance) { ... }
}
```

createStatementData.js

<br/>

1. renderPlainText() 함수 추출
2. 중간 데이터 구조 역할을 할 객체를 만들어 renderPlainText()에 인수로 전달
3. 고객 정보를 중간 데이터로 옮김
4. 공연 정보를 중간 데이터로 옮김
5. invoice 매개 변수 삭제
6. 공연 객체 복시
7. renderPlainText()의 중첩 함수였던 playFor()을 statement()로 옮김 👉 playFor()을 호출하던 부분을 중간 데이터를 통해 사용
8. amountFor(), volumeCreditsFor(), totalAmount(), totalVolumeCredits()도 동일한 방식으로 변경
9. 반복문을 파프라인으로 바꾸기
10. statement()에 필요한 데이터 처리에 해당하는 코드를 모두 함수로 추출하고 별도 파일에 저장
11. HTML 출력 함수 생성

<br/>

---

<br/>

### 1.7 중간 점검: 두 파일(과 두 단계)로 분리됨

<br/>

전체 로직을 구성하는 요소 각각이 더 뚜렷이 부각되고, 계산하는 부분과 출력 형식을 다루는 부분이 분리되었다.  
👉 이렇게 모듈화하면 각 부분이 하는일과 그 부분들이 맞물려 돌아가는 과정을 파악하기 쉬워진다.

<br/>

---

<br/>

### 1.8 다형성을 활용해 계산 코드 재구성하기

<br/>

- 조건부 로직을 다형성으로 바꾸기

```javascript
function createStatementData() {
  ...

  function enrichPerformance(aPerformance) {
    const calculator = new PerformanceCalculator(aPerformance, playFor(aPerformance)); // 공연료 계산기 생성
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);

    result.amount = calculator.amount; // amountFor() 대신 계산기 함수 사용
    result.volumeCredits = calculator.volumeCredits;

    return result;
  }

  function amountFor(aPerformance) {
    return new PerformanceCalculator(aPerformance, playFor(aPerformance)).amount; // 계산기 사용
  }
}

class PerformenceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }

  get amount() {
    let result = 0;

    switch (this.play.type) {
      case 'tragedy': // 비극
        result = 40000;
        if (this.performance.audience > 30) {
          result += 1000 * (this.performance.audience - 30);
        }
        break;

      case 'comedy': // 희극
        result = 30000;
        if (this.performance.audience > 20) {
          result += 10000 + 500 * (this.performance.audience - 20);
        }
        result += 300 * this.performance.audience;
        break;
      default:
        throw new Error(`알 수 없는 장르: ${this.play.type}`);
    }

    return result;
  }

  get volumeCredits() {
    let result = 0;
    result += Math.max(this.performance.audience - 30, 0);
    if ('comedy' === this.play.type) result += Math.floor(this.performance.audience / 5);
    return result;
  }
}
```

1. 공연료 계산기 클래스 생성
2. 함수 선언 바꾸기 👉 공연 정보를 계산기로 전달
3. 함수 옮기기 👉 공연료 계산 코드를 계산기 클래스 안으로 복사
4. amountFor()도 계산기를 사용하도록 수정
5. 함수 인라인 👉 enrichPerformance()에서 amountFor() 대신 계산기 함수 사용

<br/>

- 공연료 계산기를 다형성 버전으로 만들기

```javascript
function createStatementData() {
  function enrichPerformance(aPerformance) {
    const calculator = createPerformanceCalculator(aPerformance, playFor(aPerformance)); // 생성자 대신 factory 함수 사용
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);

    result.amount = calculator.amount;
    result.volumeCredits = calculator.volumeCredits;

    return result;
  }

  function amountFor(aPerformance) {
    return new PerformanceCalculator(aPerformance, playFor(aPerformance)).amount;
  }
}

function createPerformanceCalculator(aPerformance, aPlay) {
  switch (aPlay.type) {
    case 'tragedy':
      return new TragedyCalculator(aPerformance, aPlay);
    case 'comedy':
      return new ComedyCalculator(aPerformance, aPlay);
    default:
      throw new Error(`알 수 없는 장르: ${aPlay.type}`);
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

  // 포인트 방식이 다른 희극 처리 로직을 해당 서브클래스로 내린다
  get volumeCredits() {
    return super.volumeCredits + Math.floor(this.performance.audience / 5);
  }
}

class PerformenceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }

  get amount() {
    // 서브 클래스에서 처리
  }

  get volumeCredits() {
    return Math.max(this.performance.audience - 30, 0); // 일반적인 경우만 남겨둠
  }
}
```

1. 타입 코드를 서브클래스로 바꾸기를 위해  
   1-1. 생성자를 factory 함수로 바꾸기  
   1-2. 조건부 로직을 다형성으로 바꾸기

<br/>

---

<br/>

### 1.9 상태 점검: 다형성을 활용하여 데이터 생성하기

<br/>

같은 타입의 다형성을 기반으로 실행되는 함수가 많을수록 하나의 생성 함수로 옮겨서 구성하는 쪽이 유리하다.

<br/>

---

<br/>

### 1.10 마치며

<br/>

- 리팩터링 단계
  1. 원본 함수를 중첩 함수 여러개로 분리
  2. 단계 쪼개기를 적용해서 계산 코드와 출력 코드 분리
  3. 계산 로직을 다형성으로 표현
  4. 각 단계를 거칠 때마다 코드 구조를 보강

<br/>

`리팩터링은 대부분 코드가 하는 일을 파악하는 데서 시작`한다.  
그래서 코드를 읽고, 개선점을 찾고, 리팩터링 작업을 통해 개선점을 코드에 반영하는 식으로 진행한다.  
그 결과 코드가 명확해지고 이해하기 더 쉬워진다. 그러면 또 다른 개선점이 떠오르며 선순환이 형성된다.

<br/>

> 좋은 코드를 가늠하는 확실한 방법은 `'얼마나 수정하기 쉬운가'`다.

<br/>

`코드는 명확`해야 하며, 코드를 수정해야 할 상황이 생기면 고쳐야할 곳을 쉽게 찾을 수 있고 오류 없이 빠르게 수정할 수 있어야 한다.  
건강한 코드베이스는 생산성을 극대화하고, 고객에게 필요한 기능을 더 빠르고 저렴한 비용으로 제공하도록 해준다.
코드를 건강하게 관리하려면 프로그래밍 팀의 현재와 이상의 차이를 항상 신경쓰면서 이상에 가까워지록 리팩터링 해야 한다.

<br/>

- 리팩터링을 효과적으로 하는 핵심
  - 단계를 잘게 나눠야 더 빠르게 처리할 수 있다.
  - 코드는 절대 깨지지 않는다.  
    👉 이러한 작은 단계들이 모여서 상당히 큰 변화를 이룰 수 있다는 사실을 깨닫는 것.
