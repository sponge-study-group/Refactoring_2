# [6] 기본적인 리팩터링

카탈로그의 첫머리는 가장 기본적이고 많이 사용해서 제일 먼저 배워야 하는 리팩터링들로 시작한다.

내가 가장 많이 사용하는 리팩터링은 **함수 추출하기**와 **변수 추출하기**다.

리팩터링은 본래 코드를 변경하는 작업인 만큼, 두 리팩터링의 반대인 **함수 인라인하기**와 **변수 인라인하기**도 자주 사용한다.

추출은 결국 이름 짓기며, 코드 이해도가 높아지다 보면 이름을 바꿔야 할 때가 많다.

**함수 선언 바꾸기**는 함수의 이름을 변경할 때 많이 쓰인다.

함수의 인수를 추가하거나 제거할 때도 이 리팩터링을 적용한다.

바꿀 대상이 변수라면 **변수 이름 바꾸기**를 사용하는데, 이는 **변수 캡슐화하기**와 관련이 깊다.

자주 함께 뭉쳐다니는 인수들은 **매개변수 객체 만들기**를 적용해 객체 하나로 묶으면 편리할 때가 많다.

함수 구성과 이름 짓기는 가장 기본적인 저수준 리팩터링이다.

그런데 일단 함수를 만들고 나면 다시 고수준 모듈로 묶어야 한다.

이렇게 함수를 그룹으로 묶을 때는 **여러 함수를 클래스로 묶기**를 이용한다.

이때 이 함수들이 사용하는 데이터도 클래스로 함께 묶는다.

또 다른 방법으로는 **여러 함수를 변환 함수로 묶기**도 있는데, 읽기 전용 데이터를 다룰 때 특히 좋다.

나는 한 걸음 나아가, 한데 묶은 모듈들의 작업 처리 과정을 명확한 단계로 구분 짓는 **단계 쪼개기**를 적용할 때도 많다.

## 1. 함수 추출하기 (Extract Function)

> 반대 리팩터링

[함수 인라인하기]

```jsx
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();

  //세부 사항 출력
  console.log(`고객명 : ${invoice.customer}`);
  console.log(`채무액 : ${outstanding}`);
}
```

```jsx
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();
  printDetails(outstanding);

  function printDetails(outstanding) {
    console.log(`고객명 : ${invoice.customer}`);
    console.log(`채무액 : ${outstanding}`);
  }
}
```

### 배경

함수 추출하기는 내가 가장 많이 사용하는 리팩터링 중 하나다.

코드 조각을 찾아 무슨 일을 하는지 파악한 다음 독립된 함수로 추출하고 목적에 맞는 이름을 붙인다.

코드를 언제 독립된 함수로 묶어야 할지에 관한 의견은 수 없이 많다.

1. 길이
   1. 가령 함수 하나가 한 화면을 넘어가면 안 된다는 규칙을 떠올릴 수 있다.
2. 재사용성
   1. 두 번 이상 사용될 코드는 함수로 만들고, 한 번만 쓰이는 코드는 인라인 상태로 둔다.

하지만 ‘목적과 구현을 분리’하는 방식이 가장 합리적인 기준으로 보인다.

코드를 보고 무슨 일을 하는지 파악하는데 한참 걸린다면 함수로 추출한 뒤 ‘무슨 일’에 걸맞는 이름을 짓는다.

이렇게 해두면 나중에 코드를 다시 읽을 때 함수의 목적이 눈에 확 들어오고, 본문 코드는 신경쓸 일이 거의 없다.

이 원칙을 적용한 뒤로는 함수를 아주 짧게, 대체로 단 몇 줄만 담도록 작성하는 습관이 생겼다.

내 경험상 함수 안에 들어갈 코드가 대여섯 줄을 넘어갈 때부터 슬슬 냄새가 풍기기 시작했고,

단 한 줄짜리 함수를 만드는 일도 적지 않았다.

길이가 그리 중요하지 않다는 사실을 깨닫게 된 계기는 켄트 백이 보여준 오리지널 스몰토크 시스템이었다.

당시 스몰토크는 흑백 시스템에서 실행됐으며, 화면에서 텍스트나 그래픽을 강조하려면 해당 부분의 색상을 반전시켜야 했다.

이 목적으로 쓰이는 highlight() 메서드의 구현 코드는 reverse()라는 메서드만 호출하고 있었다.

코드의 목적(highlight)과 구현(reverse) 사이의 차이가 그만큼 컸기 때문이다.

함수를 짧게 만들면 함수 호출이 많아져서 성능이 느려진다는 사람이 있는데,

함수가 짧으면 캐싱하기가 더 쉽기 때문에 컴파일러가 최적화하는 데 유리할 때가 많다.

성능 최적화에 대해서는 항상 일반 지침을 따르도록 하자.

```
**일반 지침
1. 하지 마라
2. 아직 하지 마라.**
```

이러한 짧은 함수의 이점은 이름을 잘 지어야만 발휘되므로 이름 짓기에 특별히 신경 써야 한다.

이름을 잘 짓기까지는 어느 정도 훈련이 필요하다.

하지만 일단 요령을 터득한 후로는 별도 문서 없이 코드 자체만으로 내용을 충분히 설명되게 만들 수 있다.

긴 함수에는 각각의 코드 덩어리 첫 머리에 그 목적을 설명하는 주석이 달려 있을 때가 많다.

해당 코드 덩어리를 추출한 함수의 이름을 지을 때 이 주석을 참고하면 도움이 될 것이다.

### 절차

1. 함수를 새로 만들고 목적이 잘 드러나는 이름을 붙인다.

   (’어떻게’가 아닌 ‘무엇을’ 하는지가 드러나야 한다)

   - 세부항목
     대상 코드가 함수 호출문 하나처럼 매우 간단하더라도 함수로 뽑아서 목적이 더 잘 드러나는 이름을 붙일 수 있다면 추출한다.
     이런 이름이 떠오르지 않는다면 함수로 추출하면 안 된다는 신호다.
     하지만 처음부터 최선의 이름부터 짓고 시작할 필요는 없다.
     일단 함수로 추출해서 사용해보고 효과가 크지 않다면 다시 원래 상태로 인라인해도 된다.
     중첩 함수를 지원하는 언어를 사용한다면 중첩시킨다. 이후 단계인 ‘유효범위를 벗어난 함수 처리하기’ 작업을 줄일 수 있다.
     원래 함수의 바깥으로 꺼내야 할 때가 오면 언제든 **함수 옮기기**를 적용하면 된다.

2. 추출할 코드를 원본 함수에서 복사하여 새 함수에 붙여넣는다.
3. 추출한 코드 중 원본 함수의 지역 변수를 참조하거나 추출한 함수의 유효범위를 벗어나는 변수는 없는지 검사한다. 있다면 매개변수로 전달한다.
   - 세부항목
     - 가장 일반적인 처리 방법은 이런 변수를 모두 인수로 전달하는 것이다.
       - 갱신되지 않는 변수가 이 방법으로 처리할 수 있다.
     - 추출한 코드에서만 사용하는 변수가 추출한 함수 밖에 선언되어 있다면 함수 안에서 선언하도록 수정한다.
     - 갱신되는 변수 중에서 값으로 전달되는 것들은 주의해서 처리한다.
       - 이런 변수가 하나 뿐이라면 추출한 코드를 질의 함수로 취급해서 그 결과를 해당 변수에 대입한다.
     - 때로는 값을 수정하는 지역 변수가 너무 많을 수 있다.
       - 함수 추출을 멈추고, 변수 쪼개기나 임시 변수를 질의 함수로 바꾸기를 적용해 단순하게 바꾼 후 함수 추출을 다시 시도한다.
4. 변수를 다 처리했다면 컴파일한다.
5. 원본 함수에서 추출한 코드 부분을 새로 만든 함수를 호출하는 문장으로 바꾼다.

   (즉, 추출한 함수로 일을 위임한다).

6. 테스트한다.
7. 다른 코드에 방금 추출한 것과 똑같거나 비슷한 코드가 없는지 살핀다. 있다면 함수를 호출하게 바꿀지 검토한다.

### 예시

- 유효 범위를 벗어나는 변수가 없을 때
  아주 간단한 코드에서는 함수 추출하기가 굉장히 쉽다.

  ```jsx
  function printOwing(invoice) {
    let outstanding = 0;

    console.log("*****************");
    console.log("**** 고객 채무 ****");
    console.log("*****************");

    // 미해결 채무(outstanding)를 계산한다.
    for (const o of invoice.orders) {
      outstanding += o.amount;
    }

    // 마감일(dueDate)을 기록한다.
    const today = Clock.today;
    invoice.dueDate = new Date(
      today.getFullYear(),
      today.getMonth(),
      today.getDate() + 30
    );

    // 세부 사항을 출력한다.
    console.log(`고객명 : ${invoice.customer}`);
    console.log(`채무액 : ${outstanding}`);
    console.log(`마감일 : ${invoice.dueDate.toLocaleDateString()}`);
  }
  ```

  여기서 Clock.today는 내가 Clock Wrapper로 부르는 것으로, 시스템 시계를 감싸는 객체다.
  나는 Date.now()처럼 시스템 시간을 알려주는 함수는 직접 호출하지 않는다. 직접 호출하면 테스트할 때마다 결과가 달라지기 때문이다.
  배너(banner)를 출력하는 코드는 다음과 같이 간단히 추출할 수 있다.

  ```jsx
  function printOwing(invoice) {
    let outstanding = 0;

  	printBanner();  // <- 배너 출력 로직을 함수로 추출

  	...

  	function printBanner() {
  	  console.log("*****************");
  	  console.log("**** 고객 채무 ****");
  	  console.log("*****************");
  	}
  }
  ```

  마찬가지로 세부 사항을 출력하는 코드도 간단히 추출할 수 있다.

  ```jsx
  function printOwing(invoice) {
    let outstanding = 0;

  	printBanner();  // <- 배너 출력 로직을 함수로 추출

  	...

  	printDetails(); // <- 세부 사항 출력 로직을 함수로 추출

  	function printBanner() {
  		...
  	}

  	function printDetails() {
  	  console.log(`고객명 : ${invoice.customer}`);
  	  console.log(`채무액 : ${outstanding}`);
  	  console.log(`마감일 : ${invoice.dueDate.toLocaleDateString()}`);
  	}
  }
  ```

- 지역 변수를 사용할 때
  지역 변수와 관련하여 가장 간단한 경우는 변수를 사용하지만 다른 값을 다시 대입하지는 않을 때다.
  이 경우에는 지역 변수들을 그냥 매개변수로 넘기면 된다.

  ```jsx
  function printOwing(invoice) {
    let outstanding = 0;

  	printBanner();

    // 미해결 채무(outstanding)를 계산한다.
    for (const o of invoice.orders) {
      outstanding += o.amount;
    }

    // 마감일(dueDate)을 기록한다.
    const today = Clock.today;
    invoice.dueDate = new Date(
  									    today.getFullYear(),
  									    today.getMonth(),
  									    today.getDate() + 30
    );

  	// 세부 사항을 출력한다.
    console.log(`고객명 : ${invoice.customer}`);
    console.log(`채무액 : ${outstanding}`);
    console.log(`마감일 : ${invoice.dueDate.toLocaleDateString()}`);

  	function printBanner() {
  		...
  	}
  }
  ```

  세부 사항을 출력하는 코드를 다음과 같이 지역 변수 두 개를 매개변수로 받는 함수로 추출한다.

  ```jsx
  function printOwing(invoice) {

  	...

  	// 세부 사항을 출력한다.
    console.log(`고객명 : ${invoice.customer}`);
    console.log(`채무액 : ${outstanding}`);
    console.log(`마감일 : ${invoice.dueDate.toLocaleDateString()}`);

  	printDetails(invoice, outstanding); // <- 지역 변수를 매개변수로 전달
  }

  function printDetails(invoice, outstanding) {
    console.log(`고객명 : ${invoice.customer}`);
    console.log(`채무액 : ${outstanding}`);
    console.log(`마감일 : ${invoice.dueDate.toLocaleDateString()}`);
  }
  ```

  지역 변수가 배열, 레코드, 객체와 같은 데이터 구조라면 똑같이 매개변수로 넘긴 후 필드 값을 수정할 수 있다.
  가령 마감일을 설정하는 코드는 다음과 같이 추출한다.

  ```jsx
  function printOwing(invoice) {

  	...

  	recordDueDate(invoice);

  	...

  }

  function recordDueDate(invoice) {.
  		const today = Clock.today;

  		invoice.dueDate = new Date(
  										    today.getFullYear(),
  										    today.getMonth(),
  										    today.getDate() + 30
  		);
  }
  ```

- 지역 변수의 값을 변경할 때
  만약 매개변수에 값을 대입하는 코드를 발견하면 곧바로 그 변수를 쪼개서 임시 변수를 새로 하나 만들어 그 변수에 대입하게 한다.
  대입 대상이 되는 임시 변수는 크게 두 가지로 나눌 수 있다.

  1. 변수가 추출된 함수 안에서만 사용될 때
     1. 이런 경우, 추출된 함수에서 선언하여 사용하면 된다.
  2. 변수가 추출한 함수 밖에서 사용될 때
     - 이럴 때는 변수에 대입된 새 값을 반환해야 한다.

  ```jsx
  function printOwing(invoice) {
    let outstanding = 0;

    printBanner();
    // 미해결 채무(outstanding)을 계산한다.
    for (const o of invoice.orders) {
      outstanding += o.amount;
    }

    recordDueDate(invoice);
    printDetails(invoice, outstanding);
  }
  ```

  1. 선언문을 변수가 사용되는 코드 근처로 슬라이드한다.

  ```jsx
  function printOwing(invoice) {
  	...

  	// 미해결 채무(outstanding)을 계산한다.
  	let outstanding = 0; // <- 변수 위치 이동
  	for(const o of invoice.orders) {
  		outstanding += o.amount;
  	}

  	...
  }
  ```

  1. 추출할 부분을 새로운 함수로 복사한다.

  ```jsx
  function printOwing(invoice) {
  	...

  	// 미해결 채무(outstanding)을 계산한다.
  	let outstanding = 0; // <- 변수 위치 이동
  	for(const o of invoice.orders) {
  		outstanding += o.amount;
  	}

  	...
  }

  function calculateOutstanding(invoice) {
  	let outstanding = 0;
  	for(const o of invoice.orders) {
  		outstanding += o.amount;
  	}
  	return outstanding;
  }
  ```

  1. outstanding의 선언문을 추출할 코드 앞으로 옮겼기 때문에 매개변수로 전달하지 않아도 된다.

     추출한 코드에서 값이 변경된 변수는 outstanding 뿐이다. 따라서 이 값을 반환한다.

  2. 컴파일한다.
  3. 추출한 코드의 원래 자리를 새로운 함수로 대체한다. 추출한 함수에서 새 값을 반환하니, 이 값을 원래 변수에 저장한다.

  ```jsx
  function printOwing(invoice) {
  	...

  	let outstanding = calculateOutstanding(invoice);

  	...
  }

  function calculateOutstanding(invoice) {
  	let result = 0;
  	for(const o of invoice.orders) {
  		result += o.amount;
  	}
  	return result;
  }
  ```

- 값을 반환할 변수가 여러 개라면 ?
  몇가지 방법이 있지만, 나는 주로 추출할 코드를 다르게 재구성하는 방향으로 처리한다.
  개인적으로 함수가 값 하나만 반환하는 방식을 선호하기 때문에 각각을 반환하는 함수 여러 개로 만든다.
  굳이 한 함수에서 여러 값을 반환해야 한다면, 값들을 레코드로 묶어서 반환해도 되지만,
  임시 변수 추출 작업을 다른 방식으로 처리하는 것이 나을 때가 많다.
  여기서는 **임시 변수를 질의 함수로 바꾸거나(7.4)** **변수를 쪼개는 식(9.1)**으로 처리하면 좋다.
  이렇게 추출한 함수를 최상위 수준 같은 다른 문맥(context)으로 이동하려면
  원본 함수와 같은 수준의 문맥으로 먼저 추출해본 후, 진행하라.

## 2. 함수 인라인하기(inline Function)

> 반대 리팩터링

[함수 추출하기]

```jsx
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}
```

```jsx
function getRating(driver) {
  return driver.numberOfLateDeliveries > 5 ? 2 : 1;
}
```

### 배경

이 책은 목적이 드러나는 이름의 짤막한 함수를 이용하기를 권한다.

그래야 코드가 명료해지고 이해하기 쉬워지기 때문이다.

허나 때로는 함수 본문이 이름만큼 명확한 경우도 있다.

또는 함수 본문 코드를 이름만큼 깔끔하게 리팩터링할 때도 있다. 이럴 때는 그 함수를 제거한다.

간접 호출은 유용할 수도 있지만 쓸데없는 간접 호출은 거슬릴 뿐이다.

리팩터링 과정에서 잘못 추출된 함수들도 다시 인라인한다.

간접 호출을 너무 과하게 쓰는 코드도 흔한 인라인 대상이다.

가령 다른 함수로 단순히 위임하기만 하는 함수들이 너무 많아서 위임 관계가 복잡하게 얽혀 있으면 인라인해버린다.

그중 간접 호출을 유지하는 편이 나은 경우도 있겠지만, 모두 그렇지는 않을 것이다.

함수 인라인하기를 활용하면 유용한 것만 남기고 나머지는 제거할 수 있다.

### 절차

1. 다형 메서드인지 확인한다.
   - 서브클래스에서 오버라이드하는 메서드는 인라인하면 안 된다.
2. 인라인할 함수를 호출하는 곳을 모두 찾는다.
3. 각 호출문을 함수 본문으로 교체한다.
4. 하나씩 교체할 때마다 테스트한다.
   - 인라인 작업을 한 번에 처리할 필요는 없다.
   - 인라인하기가 까다로운 부분이 있다면 일단 남겨두고 여유가 생길 때마다 틈틈이 처리한다.
5. 함수 정의(원래 함수)를 삭제한다.

말로는 아주 간단해보이지만 실제로는 그렇지 않을 때가 많다.

재귀 호출, 반환문이 여러 개인 함수, 접근자가 없는 다른 객체에 메서드를 인라인하는 방법

등을 일일히 설명하자면 몇 쪽은 필요할 것이다.

각각의 경우를 다 설명하지 않는 이유는, 그 정도로 복잡하다면 함수 인라인하기를 적용하면 안 되기 떄문이다.

### 예시

```jsx
function reportLines(aCustomer) {
  const lines = [];
  gatherCustomerData(lines, aCustomer);
  return lines;
}

function gatherCustomerData(out, aCustomer) {
  out.push(["name", aCustomer.name]);
  out.push(["location", aCustomer.location]);
}
```

단순히 잘라 붙히는 식으로 gatherCustomerData()를 reportLines()로 인라인할 수 없다.

아주 복잡하지는 않고 여전히 단번에 옮기고 약간 수정해주면 될 때도 많지만, 실수하지 않으려면 한 번에 한 문장씩 옮기는 것이 좋다.

그러니 먼저 **여러 문장을 호출한 곳으로 옮기기(8.4)**를 적용해서 첫 문장부터 시작해보자**.**

```jsx
function reportLines(aCustomer) {
  const lines = [];
  lines.push(["name", aCustomer.name]);
  gatherCustomerData(lines, aCustomer);
  return lines;
}

function gatherCustomerData(out, aCustomer) {
  out.push(["location", aCustomer.location]);
}
```

나머지 문장도 같은 식으로 처리한다.

```jsx
function reportLines(aCustomer) {
  const lines = [];
  lines.push(["name", aCustomer.name]);
  lines.push(["location", aCustomer.location]);
  gatherCustomerData(lines, aCustomer);
  return lines;
}
```

여기서 핵심은 항상 단계를 잘게 나눠서 처리하는 데 있다.

평소 내 스타일대로 함수를 작게 만들어뒀다면 인라인을 단번에 처리할 수 있을 때가 많다.

그러다 상황이 복잡해지면 다시 한 번에 한 문장씩 처리한다.

한 문장을 처리하는 데도 얼마든지 복잡해질 수 있다.

이럴 때는 더 정교한 리팩터링인 **문장을 호출한 곳으로 옮기기(8.4)**로 작업을 더 잘게 나눈다.

어느 정도 자신감이 붙으면 다시 작업을 크게 묶어서 처리한다.

## 3. 변수 추출하기(Extract Variable)

> **반대 리팩터링**

[변수 인라인하기]

```jsx
function price(order) {
  return (
    order.quantity * order.itemPrice -
    Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
    Math.min(order.quantity * order.itemPrice * 0.1, 100)
  );
}
```

```jsx
function price(order) {
	const basePrice = order.quantity * order.itemPrice;
	const quantityDiscount = Math.max(0, order.quantity - 500)
													 * order.itemPrice * 0,05;
	const shipping = Math.min(basePrice * 0.1, 100);
	return basePrice = quantityDiscount + shipping;
}
```

### 배경

표현식이 너무 복잡해서 이해하기 어려울 때가 있다.

이럴 때 지역 변수를 활용하면 표현식을 쪼개 관리하기 더 쉽게 만들 수 있다.

그러면 복잡한 로직을 구성하는 단계마다 이름을 붙일 수 있어서 코드의 목적을 훨씬 명확하게 드러낼 수 있다.

이 과정에서 추가한 변수는 디버깅에도 도움된다.

변수 추출을 고려한다고 함은 표현식에 이름을 붙히고 싶다는 뜻이다.

이름을 붙이기로 했다면 그 이름이 들어갈 문맥도 살펴야 한다.

현재 함수 안에서만 의미가 있다면 변수로 추출하는 것이 좋다.

그러나 함수를 벗어난 의미가 된다면 그 넓은 범위에서 통용되는 이름을 생각해야 한다.

다시말해 변수가 아닌 함수로 추출해야 한다. 그렇게 되면 중복을 제거해주며, 의도가 잘 드러난다.

이름이 통용되는 문맥을 넓힐 때 생기는 단점은 할 일이 늘어난다는 것이다.

많이 늘어날 것 같다면 **임시 변수를 질의 함수로 바꾸기(7.4)**를 적용할 수 있을 때까지 일단 놔둔다.

간단히 처리할 수 있다면 즉시 넓혀서 다른 코드에서도 사용할 수 있게 한다.

가령 클래스 안의 코드를 다룰 때는 함수 추출하기를 아주 쉽게 적용할 수 있다.

### 절차

1. 추출하려는 표현식에 부작용은 없는지 확인한다.
2. 불변 변수를 하나 선언하고 이름을 붙일 표현식의 복제본을 대입한다.
3. 원본 표현식을 새로 만든 변수로 교체한다.
4. 테스트한다.
5. 표현식을 여러 곳에서 사용한다면 각각을 새로 만든 변수로 교체한다.

   하나 교체할 때마다 테스트한다.

### 예시

- 기본 예시
  ```jsx
  function price(order) {
    // 가격(price) = 기본가격 - 수량 할인 + 배송비
    return (
      order.quantity * order.itemPrice -
      Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
      Math.min(order.quantity * order.itemPrice * 0.1, 100)
    );
  }
  ```
  간단한 코드지만 더 쉽게 만들 수 있다.
  1. 기본 가격은 상품 가격 (itemPrice)에 수량(quantity)을 곱한 값임을 파악해내야 한다.
  2. 이 로직을 이해했다면 기본 가격을 담을 변수를 만들고 적절한 이름을 지어준다.
  ```jsx
  function price(order) {
    // 가격(price) = 기본가격 - 수량 할인 + 배송비
    const basePrice = order.quantity * order.itemPrice;
    return (
      order.quantity * order.itemPrice -
      Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
      Math.min(order.quantity * order.itemPrice * 0.1, 100)
    );
  }
  ```
  1. 이 변수를 실제로 사용해야 하므로 원래 표현식에서 새로 만든 변수에 해당하는 부분을 교체한다.
  ```jsx
  function price(order) {
    // 가격(price) = 기본가격 - 수량 할인 + 배송비
    const basePrice = order.quantity * order.itemPrice;
    return (
      basePrice -
      Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
      Math.min(basePrice * 0.1, 100)
    );
  }
  ```
  수량 할인, 배송비도 똑같이 수정한 후 주석을 지우자.
  ```jsx
  function price(order) {
    const basePrice = order.quantity * order.itemPrice;
    const quantityDiscount =
      Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
    const shipping = Math.min(basePrice * 0.1, 100);
    return basePrice - quantityDiscount + shipping;
  }
  ```
- 클래스 안에서

  ```jsx
  class Order {
    constructor(aRecord) {
      this.data = aRecord;
    }

    get quantity() {
      return this.data.quantity;
    }
    get itemPrice() {
      return this.data.itemPrice;
    }

    get price() {
      return (
        this.quantity * this.itemPrice -
        Math.max(0, this.quantity - 500) * this.itemPrice * 0.05 +
        Math.min(this.quantity * this.itemPrice * 0.1, 100)
      );
    }
  }
  ```

  추출하려는 이름은 같지만, 그 이름이 가격을 계산하는 price() 메서드의 범위를 넘어 Order 클래스 전체에 적용된다.
  이처럼 클래스 전체에 영향을 줄 때는 변수가 아닌 메서드로 추출하는 편이다.

  ```jsx
  class Order {
    constructor(aRecord) {
      this.data = aRecord;
    }

    get quantity() {
      return this.data.quantity;
    }
    get itemPrice() {
      return this.data.itemPrice;
    }

    get price() {
      return this.basePrice - this.quantityDiscount - this.shipping;
    }

    get basePrice() {
      return this.quantity * this.itemPrice;
    }
    get quantityDiscount() {
      Math.max(0, this.quantity - 500) * this.itemPrice * 0.05;
    }
    get shipping() {
      Math.min(this.basePrice * 0.1, 100);
    }
  }
  ```

  객체는 특정 로직과 데이터를 외부와 공유하려 할 때 공유할 정보를 설명해주는 적당한 크기의 문맥이 되어준다.
  이 예처럼 간단한 경우라면 효과가 크지 않지만, 덩치가 큰 클래스에서 공통 동작을 별도 이름으로 뽑아내서 추상화해두면
  그 객체를 다룰 때 쉽게 활용할 수 있어서 매우 유용하다.

## 4. 변수 인라인하기(inline Variable)

> 반대 리팩터링

\*\*[변수 추출하기]

```jsx
let basePrice = anOrder.basePrice;
return basePrice > 1000;
```

```jsx
return anOrder.basePrice > 1000;
```

### 배경

변수는 함수 안에서 표현식을 가리키는 이름으로 쓰이며, 대채로 긍정적인 효과를 준다.

하지만 그 이름이 원래 표현식과 다를 바 없을 때도 있다.

또 변수가 주변 코드를 리팩터링하는 데 방해가 되기도 한다.

이럴 때는 그 변수를 인라인하는 것이 좋다.

### 절차

1. 대입문의 우변(표현식)에서 부작용이 생기지는 않는지 확인한다.
2. 변수가 불변으로 선언되지 않았다면 불변으로 만든 후 테스트한다.

   → 이렇게 하면 변수에 값이 단 한번만 대입되는지 확인할 수 있다.

3. 이 변수를 가장 처음 사용하는 코드를 찾아서 대입문 우변의 코드로 바꾼다.
4. 테스트한다.
5. 변수를 사용하는 부분을 모두 교체할 때까지 이 과정을 반복한다.
6. 변수 선언문과 대입문을 지운다.
7. 테스트한다.

## 5. 함수 선언 바꾸기(Change Function Declaration)

> **다른 이름**

함수 이름 바꾸기, 시그니처 바꾸기

```jsx
function circumference(radius) { ... }
```

```jsx
function circum(radius) { ... }
```

### 배경

함수는 프로그램을 작은 부분으로 나누는 주된 수단이다.

함수 선언은 각 부분이 서로 맞물리는 방식을 표현하며,, 실질적으로 소프트웨어 구성 요소를 조립하는 연결부 역할을 한다.

연결부를 잘 정의하면 새로운 부분을 추가하기 쉬운 반면,

잘못 정의하면 소프트웨어 동작 파악이 어려워지고 요구사항이 바뀔 때 적절히 수정하기 어렵게 한다.

다행히 소프트웨어는 소프트하기 때문에 연결부를 수정할 수 있다. 단, 주의해서 해야 한다.

이러한 연결부에서 가장 중요한 요소는 함수의 이름이다.

이름이 좋으면 함수의 구현 코드를 살펴볼 필요 없이 호출문만 보고도 무슨 일을 하는지 파악할 수 있다.

적합한 이름은 단번에 짓기 어려우며, 이름이 잘못된 함수를 발견하면 더 나은 이름이 떠오르는 즉시 바꿔라.

그래야 나중에 그 코드를 볼 때 무슨 일을 하는지 ‘또’ 고민하지 않게 된다.

<aside>
💡 주석을 달아 함수의 목적을 설명해보면 주석이 멋진 이름으로 바뀌어 되돌아올 때가 있다.
</aside>

함수의 매개변수도 마찬가지다. 매개변수는 함수가 외부 셰계와 어우러지는 방식을 정의한다.

매개변수는 함수를 사용하는 문맥을 설정한다.

예컨대 전화번호 포매팅 함수가 매개변수로 사람을 받는다면, 회사 전화번호 포매팅에는 사용할 수 없다.

사람 대신 전화번호를 받으면 이 함수의 활용 범위를 넓힐 수 있다.

그 뿐 아니라, 다른 모듈과의 결합(coupling)을 제거할 수도 있다.

동작에 필요한 모듈 수가 줄어들수록 무언가를 수정할 때 머리에 담아둬야 하는 내용도 적어진다.

매개변수를 올바르게 선택하기란 단순한 규칙 몇 개로 표현할 수 없다.

대여한지 30일이 지났는지를 기준으로 지불 기한이 넘었는지 판단하는 함수가 있다고 해보자.

이 함수의 매개변수는 지불 객체가 적절할까 아니면 마감일로 해야 할까?

지불 객체로 정하면 이 함수는 지불 객체의 인터페이스와 결합돼버린다.

대신 지불이 제공하는 여러 속성에 쉽게 접근할 수 있어서

내부 로직이 복잡하더라도 이 함수를 호출하는 코드를 일일이 찾아서 변경할 필요가 없다.

실질적으로 함수의 캡슐화 수준이 높아지는 것이다.

이 문제의 정답은 바로 정답이 없다는 것이다. 특히 시간이 흐를수록 더욱더 그렇다.

따라서 어떻게 연결하는 것이 더 나은지 더 잘 이해하게 될 때마다 리팩터링으로 개선하라.

나는 다른 리팩터링을 지칭할 때 대체로 대표 명칭만 사용한다.

하지만 함수 선언 바꾸기에서 ‘이름 바꾸기’가 차지하는 비중이 높기 때문에,

단순히 이름만 바꿀 때는 ‘함수 이름 바꾸기’라고 표현해서 확실히 구분할 것이다.

이름을 바꿀 때든 매개변수를 변경할 때든 절차는 똑같다.

### 절차

이 책에서 다른 리팩터링들은 대부분 상황에서 대체로 효과적인 방법 1가지만 소개했다.

하지만 함수 선언 바꾸기는 사정이 다르다.

‘간단한 절차’만으로 충분할 때도 있지만, 더 세분화된 ‘마이그레이션 절차’가 훨씬 적합한 경우도 많다.

따라서 이 리팩터링을 할 때는 먼저 변경사항을 살펴보고 함수 선언과 호출문들을 단번에 고칠 수 있는지 가늠해본다.

마이그레이션 절차를 적용하면 호출문들을 점진적으로 수정할 수 있다.

호출하는 곳이 많거나, 호출 과정이 복잡하거나, 호출 대상이 다형 메서드거나, 선언을 복잡하게 변경할 때는 이렇게 해야한다.

- 간단한 절차
  1. 매개변수를 제거하려거든 먼저 함수 본문에서 제거 대상 매개변수를 참조하는 곳은 없는지 확인한다.
  2. 메서드 선언을 원하는 형태로 바꾼다.
  3. 기존 메서드 선언을 참조하는 부분을 모두 찾아서 바뀐 형태로 수정한다.
  4. 테스트한다.

변경할 게 둘 이상이면 나눠서 처리하는 편이 나을 때가 많다.

따라서 이름 변경과 매개변수 추가를 모두 하고 싶으면 각각을 독립적으로 처리하자

(그러다 문제가 생기면 작업을 되돌리고 ‘마이그레이션 절차’를 따른다.

- 마이그레이션 절차

  1. 이어지는 추출 단계를 수월하게 만들어야 한다면 함수 본문을 적절히 리팩터링한다.
  2. 함수 본문을 새로운 함수로 추출한다.

     → 새로 만들 함수 이름이 기존 함수와 같다면 일단 검색하기 쉬운 이름을 임시로 붙혀준다.

  3. 추출한 함수에 배개변수를 추가해야 한다면 ‘간단한 절차’를 따라 추가한다.
  4. 테스트한다.
  5. 기존 함수를 인라인한다.
  6. 이름을 임시로 붙혀뒀다면 함수 선언 바꾸기를 한 번 더 적용해서 원래 이름으로 되돌린다.
  7. 테스트한다.

다형성을 구현한 클래스, 즉 상속 구조에 있는 클래스의 메서드를 변경할 때는 다형 관계인 다른 클래스들에도 변경이 반영되어야 한다.

이때, 상황이 복잡하기 때문에 간접 호출 방식으로 우회(혹은 중간 단계로 활용)하는 방법도 쓰인다.

먼저 원하는 형태의 메서드를 새로 만들어서 원래 함수를 호출하는 전달(forward) 메서드로 활용하는 것이다.

단일 상속 구조라면 전달 메서드를 슈퍼클래스에 정의하면 해결된다.

슈퍼클래스와의 연결을 제공하지 않는 언어라면 전달 메서드를 모든 구현 클래스 각각에 추가해야 한다.

공개된 API를 리팩터링할 때는 새 함수를 추가한 다음 리팩터링을 잠시 멈출 수 있다.

이 상태에서 예전 함수를 폐기 대상(deprecated)으로 지정하고 모든 클라이언트가 새 함수로 이전할 때까지 기다린다.

클라이언트들이 모두 이전했다는 확신이 들면 예전 함수를 지운다.

### 예시

- **함수 이름 바꾸기(간단한 절차)**
  함수 이름을 너무 축약한 예를 준비했다.
  ```jsx
  function cirum(radius) {
    return 2 * Math.PI * radius;
  }
  ```
  이 함수의 이름을 이해하기 더 쉽게 바꾸려 한다. [2]먼저 함수 선언부터 수정하자.
  ```jsx
  function cirumference(radius) {
    return 2 * Math.PI * radius;
  }
  ```
  [3] 다음으로 cirum()을 호출하는 곳을 모두 찾아서 변경된 이름으로 바꾼다.
  기존 함수를 참조하는 곳을 얼마나 쉽게 찾을 수 있는가는 개발 언어에 영향을 받는다.
  정적 타입 언어와 뛰어난 IDE의 조합이라면 함수 이름 바꾸기를 자동으로 처리할 수 있고, 그 과정에서 오류가 날 가능성도 거의 없다.
  정적 타입 언어가 아니라면 검색 능력이 뛰어난 도구라도 잘 못찾는 경우가 꽤 있어서 일거리가 늘어난다.
  매개변수 추가나 제거도 똑같이 처리한다.
  각각의 단계를 순서대로 처리하는 편이 대체로 좋다.
  함수 이름 바꾸기와 매개변수 추가를 모두 할때는 이름을 변경하고, 테스트하고, 매개변수를 추가, 테스트하는 식으로 진행한다.
  간단한 절차의 단점은 호출문과 선언문(다형성을 구현했으면 여러 선언문 모두를) 한 번에 수정해야 한다는 것이다.
  수정할 부분이 많거나 같은 이름이 여러개일 때는 문제가 생긴다.
  나는 변경 작업이 복잡할수록 한 번에 진행하기를 꺼린다. 그래서 이런 상황에 처하면 마이그레이션 절차를 따른다.
  마찬가지로 간단한 절차를 따르다가 문제가 생겨도 코드를 가장 최근의 정상 상태로 돌리고 나서 마이그레이션 절차를 진행한다.
- 함수 이름 바꾸기(마이그레이션 절차)

  ```jsx
  function cirum(radius) {
    return 2 * Math.PI * radius;
  }
  ```

  [2] 먼저 함수 본문 전체를 새로운 함수로 추출한다.

  ```jsx
  function circum(radius) {
    return circumference(radius);
  }

  function circumference(radius) {
    return 2 * Math.PI * radius;
  }
  ```

  [4] 수정한 코드를 테스트한 뒤 [5] 예전 함수를 호출하는 부분을 모두 새 함수를 호출하도록 바꾼다.
  [7] 하나를 변경할 때마다 테스트하면서 한 번에 하나씩 처리하자. 모두 바꿨다면 기존 함수를 삭제한다.
  리팩터링 대상은 대부분 직접 수정할 수 있는 코드지만, 함수 선언 바꾸기만큼은 공개된 API
  다시 말해 직접 고칠 수 없는 외부 코드가 사용하는 부분을 리팩터링하기에 좋다.
  가령 circumference() 함수를 만들고 나서 잠시 멈춘 후, 가능하다면 circum()을 deprecated임을 표시한다.
  그런 다음 circum()의 클라이언트들 모두가 새 함수를 사용하게 바뀔 때까지 기다린다.
  모두 바꿨다면 circum()을 삭제한다. 모두 바꾸지 않아도, 새로운 이들은 더 나은 이름의 함수를 사용할 수 있게 된다.

- 매개변수 추가하기
  도서 관리 프로그램의 Book 클래스에 예약 기능이 구현되어 있다고 하자.
  ```jsx
  class Book {
  	...
  	addReservation(customer) {
  		this.reservations.push(customer);
  	}
  	...
  }
  ```
  예약 시 우선순위 큐를 지원하라는 새로운 요구가 추가되었다.
  그래서 addReservation()을 호출할 때 예약 정보를 일반 큐에 넣을지 우선순위 큐에 넣을지 지정하는 매개변수를 추가하려고 한다.
  addReservation()을 호출하는 곳을 모두 찾고 고치기가 쉽다면 곧바로 변경하면 된다.
  그렇지 않다면 마이그레이션 절차대로 진행해야 한다. 여기선 후자라고 가정한다.
  [2] 먼저 addReservation()의 본문을 새로운 함수로 추출한다. 함수 명은 찾기 쉬운 임시 이름을 붙인다.
  ```jsx
  class Book {
  	...
  	addReservation(customer) {
  		this.zzAddReservation(customer);
  	}
  	zzAddReservation(customer) {
  		this.reservations.push(customer);
  	}
  	...
  }
  ```
  [3] 그런 다음 새 함수의 선언문과 호출문에 원하는 매개변수를 추가한다.
  ```jsx
  class Book {
  	...
  	addReservation(customer) {
  		this.zzAddReservation(customer, false);
  	}
  	zzAddReservation(customer, isPriority) {
  		this.reservations.push(customer, isPriority);
  	}
  	...
  }
  ```
  나는 자바스크립트로 프로그래밍한다면, 호출문을 변경하기 전에
  어서션을 추가하여 호출하는 곳에서 추가한 매개변수를 실제로 사용하는지 확인한다.
  ```jsx
  class Book {
  	...
  	zzAddReservation(customer, isPriority) {
  		assert(isPriority === true || isPriority === false);
  		this.reservations.push(customer, isPriority);
  	}
  	...
  }
  ```
  이렇게 하면 호출문을 수정하는 과정에서 실수로 새 매개변수를 빠뜨린 부분을 찾는데 도움된다.
  [5] 이제 기존 함수를 인라인하여 호출 코드들이 새 함수를 이용하도록 고친다.
  호출문은 한번에 하나씩 변경한다.
  [6] 다 고쳤다면 새 함수의 이름을 기존 함수의 이름으로 바꾼다.
  이상의 작업은 대부분 간단한 절차만으로도 무리가 없지만, 필요하면 마이그레이션 절차를 따르기도 한다.
- 매개변수를 속성으로 바꾸기
  고객이 뉴 잉글랜드에 살고 있는지 확인하는 함수가 있다고 하자.
  ```jsx
  function inNewEngland(aCustomer) {
  	return ["MA"], "CT", "ME", "VT", "NH", "RI"].includes(aCustomer.address.state);
  }
  ```
  다음은 이 함수를 호출하는 코드 중 하나다.
  ```jsx
  const newEnglanders = someCustomers.filter((c) => inNewEngland(c));
  ```
  inNewEngland()는 고객이 거주하는 주 이름을 보고 뉴잉글랜드에 사는지 판단한다.
  나라면 이 함수가 state 식별 코드를 매개변수로 받도록 리팩터링할 것이다.
  그러면 고객에 대한 의존성이 제거되어 더 넓은 문맥에 활용할 수 있기 때문이다.
  [1] 함수 선언을 바꿀 때 함수 추출부터 하는 편이다.
  하지만 이번 코드는 함수 본문을 살짝 리팩터링해두면 이후 작업이 더 수월해질 터라 우선 배개면수로 사용할 코드를 변수로 추출해둔다.
  ```jsx
  function inNewEngland(aCustomer) {
  	const stateCode = aCustomer.address.state;
  	return ["MA"], "CT", "ME", "VT", "NH", "RI"].includes(stateCode);
  }
  ```
  [2] 이제 함수 추출하기로 새 함수를 만든다.
  ```jsx
  function inNewEngland(aCustomer) {
  	const stateCode = aCustomer.address.state;
  	return xxinNewEngland(stateCode);
  }
  function xxinNewEngland(stateCode) {
  	return ["MA"], "CT", "ME", "VT", "NH", "RI"].includes(stateCode);
  }
  ```
  그런 다음 기존 함수 안에 변수로 추출해둔 입력 매개변수를 인라인한다.
  ```jsx
  function inNewEngland(aCustomer) {
    return xxinNewEngland(aCustomer.address.state);
  }
  ```
  [5] 함수 인라인하기로 기존 함수의 본문을 호출문들에 집어넣는다.
  실질적으로 기존 함수 호출문을 새 함수 호출문으로 교체하는 셈이다. 이 작업은 한 번에 하나씩 처리한다.
  ```jsx
  const newEnglanders = someCustomers.filter((c) =>
    xxinNewEngland(c.address.state)
  );
  ```
  [6] 함수 선언 바꾸기를 다시 한번 적용하여 새 함수의 이름을 기존 함수의 이름으로 바꾼다.
  ```jsx
  const newEnglanders = someCustomers.filter((c) =>
    inNewEngland(c.address.state)
  );
  ```
  ```jsx
  function inNewEngland(stateCode) {
  	return ["MA"], "CT", "ME", "VT", "NH", "RI"].includes(stateCode);
  }
  ```
  자동 리팩터링 도구는 마이그레이션 절차의 활용도를 떨어뜨리기도 하고 효과를 배가하기도 한다.
  도구가 자동 리팩터링을 안전히 수행해줘서 이 절차를 사용할 일이 적어지기 때문이다.
  하지만 이 예시처럼 모든 작업을 자동으로 처리할 수 없을 때는 상당한 도움을 준다.
  추출과 인라인같은 핵심적인 변경을 훨씬 빠르고 안전하게 할 수 있기 때문이다.

## 6.변수 캡슐화하기(Encapsulate Variable)

```jsx
let defaultOwner = { firstName: "마틴", lastName: "파울러" };
```

```jsx
let defaultOwnerData = { firstName: "마틴", lastName: "파울러" };
export function defaultOwner() {
  return defaultOwnerData;
}
export function setDefaultOwner(arg) {
  defaultOwnerData = arg;
}
```

### 배경

리팩터링은 결국 프로그램의 요소를 조작하는 일이다. 함수는 데이터보다 다루기가 수월하다.

함수를 사용한다는 건 대체로 호출한다는 뜻이고, 함수의 이름을 바꾸거나 다른 모듈로 옮기기는 어렵지 않다.

여차하면 기존 함수를 그대로 둔 채 전달(forward) 함수로 활용할 수도 있기 때문이다.

이런 전달 함수를 오래 남겨둘 일은 별로 없지만 리팩터링 작업을 간소화하는데 큰 역할을 한다.

반대로 데이터는 함수보다 다루기가 까다로운데, 그 이유는 이런 식으로 처리할 수 없기 때문이다.

데이터는 참조하는 모든 부분을 한 번에 바꿔야 코드가 제대로 작동한다.

짧은 함수 안의 임시 변수처럼 유효범위가 좁은 데이터는 어려울 게 없지만, 크다면 어려워진다.

전역 데이터가 골칫거리인 이유도 바로 여기에 있다.

그래서 접근할 수 있는 범위가 넓은 데이터를 옮길 때는 먼저

그 데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화하는 것이 가장 좋은 방법일 때가 많다.

데이터 재구성이라는 어려운 작업을 함수 재구성이라는 더 단순한 작업으로 변환하는 것이다.

데이터 캡슐화는 다른 경우에도 도움을 준다.

데이터를 변경하고 사용하는 코드를 감시할 수 있는 확실한 통로가 되어주기 때문에

데이터 변경 전 검증이나 변경 후 추가 로직을 쉽게 끼워넣을 수 있다.

유효범위가 함수 하나보다 넓은 가변 데이터는 모두 이런식으로 캡슐화한다.

데이터 유효범위가 넓을수록 캡슐화해야 한다.

레거시 코드를 다룰 때는 이런 변수를 참조하는 코드를 추가하거나 변경할 때마다 최대한 캡슐화한다.

그래야 자주 사용하는 데이터에 대한 결합도가 높아지는 일을 막을 수 있다.

객체 지향에서 객체의 데이터를 항상 private으로 유지해야 한다고 그토록 강조하는 이유가 바로 여기에 있다.

public 필드를 발견할 때마다 캡슐화해서 ( 이 경우는 흔히 ‘필드 캡슐화하기’로 부른다) 가시 범위를 제한하려 한다.

클래스 안에서 필드를 참조할 때조차 반드시 접근자를 통하게 하는 자가 캡슐화(self-encapsulate)를 주장하는 사람도 있다.

필드를 자가 캡슐화해야 할 정도로 클래스가 크다면 잘게 쪼개야 하겠지만, 쪼개기 전 단계로 활용하면 도움은 된다.

불변 데이터는 가변 데이터보다 캡슐화할 이유가 적다.

### 절차

1. 변수로의 접근과 갱신을 전담하는 캡슐화 함수들을 만든다.
2. 정적 검사를 수행한다.
3. 변수를 직접 참조하던 부분을 모두 적절한 캡슐화 함수 호출로 바꾼다. 하나씩 바꿀때마다 테스트한다.
4. 변수의 접근 범위를 제한한다.

   → 변수로의 직접 접근을 막을 수 없을 때도 있다. 그럴 때는 변수 이름을 바꿔서 테스트하면 쉽게 찾아낼 수 있다.

5. 테스트한다.
6. 변수 값이 레코드라면 레코드 캡슐화하기(7.1)를 적용할지 고려해본다.

### 예시

- 기본 캡슐화 기법
  전역 변수에 중요한 데이터가 담겨 있는 경우를 생각해보자
  ```jsx
  let defaultOwner = { firstName: "마틴", lastName: "파울라" };
  ```
  데이터라면 당연히 다음과 같이 참조하는 코드가 있을 것이다.
  ```jsx
  spaceship.owner = defaultOwnwer;
  ```
  갱신하는 코드 역시 있을 것이다.
  ```jsx
  defaultOwner = { firstName: "레베카", lastName: "파슨스" };
  ```
  [1] 기본적인 캡슐화를 위해 가장 먼저 데이터를 읽고 쓰는 함수부터 정의한다.
  ```jsx
  function getDefaultOwner() {
    return defaultOwner;
  }
  function setDefaultOwner(arg) {
    defaultOwner = arg;
  }
  ```
  [3] 그런 다음 defaultOwner를 참조하는 코드를 찾아서 방금 만든 함수를 호출하도록 고친다.
  ```jsx
  spaceship.owner = getDefaultOwner();
  setDefaultOwner({ firstName: "레베카", LASTNAME: "파슨스" });
  ```
  하나씩 바꿀 때마다 테스트한다.
  [4] 모든 참조를 수정했다면 이제 변수의 가시 범위를 제한한다.
  자바스크립트로 작성할 때는 변수와 접근자 메서드를 같은 파일로 옮기고 접근자만 노출(export) 시키면 된다.
  > defaultOwner.js
  ```jsx
  let defaultOwner = { firstName: "마틴", lastName: "파울러" };
  export function getDefaultOwner() {
    return defaultOwner;
  }
  export function setDefaultOwner(arg) {
    defaultOwner = arg;
  }
  ```
  변수로의 접근을 제한할 수 없을 때는 최대한 변수 이름에 녹여낸다.
  마지막으로, 나는 게터 이름 앞에 get을 붙이는 것을 싫어해서 get을 빼도록 하겠다.
- 값 캡슐화하기
  위의 기본 캡슐화 기법으로 데이터 구조로의 참조를 캡슐화하면, 그 구조로의 접근이나 구조 자체를 다시 대입하는 행위는 제어할 수 있다.
  하지만 필드 값을 변경하는 일은 제어할 수 없다.
  기본 캡슐화 기법은 데이터 항목을 참조하는 부분만 캡슐화한다.
  대부분 이 정도로 충분하지만, 변수뿐 아니라 변수에 담긴 내용을 변경하는 행위까지 제어할 수 있게 캡슐화하고 싶을 때도 많다.
  이렇게 하는 방법은 크게 두가지다.
  가장 간단한 방법은 그 값을 바꿀 수 없게 만드는 것이다.
  나는 주로 게터가 데이터의 복제본을 반환하도록 수정하는 식으로 처리한다.

  ```jsx
  let defaultOwnerData = { firstName: "마틴", lastName: "파울러" };
  export function defaultOwner() {
    return Object.assign({}, defaultOwnerData);
  }
  export function setDefaultOwner(arg) {
    defaultOwnerData = arg;
  }
  ```

  특히 리스트에 이 기법을 많이 적용한다.
  데이터의 복제본을 반환하면 클라이언트는 게터로 얻은 데이터를 변경할 수 있지만 원본에는 영향을 주지 못한다.
  단, 공유 데이터(원본)을 변경하기 원하는 클라이언트가 있을 수 있다.
  이럴 때 나는 문제가 될만한 부분을 테스트로 찾는다.
  아니면 아예 변경할 수 없게 만들 수도 있다.
  이를 위한 좋은 방법이 레코드 캡슐화하기(7.1)다.

  ```jsx
  let defaultOwnerData = { firstName: "마틴", lastName: "파울러" };
  export function defaultOwner() {
    return new Person(defaultOwnerData);
  }
  export function setDefaultOwner(arg) {
    defaultOwnerData = arg;
  }

  class Person {
    constructor(data) {
      this.lastName = data.lastName;
      this.firstName = data.firstName;
    }
    get lastName() {
      return this.lastName;
    }
    get firstName() {
      return this.firstName;
    }
  }
  ```

  이렇게 하면 defaultOwnerData의 속성을 다시 대입하는 연산은 모두 무시된다.
  이런 변경을 감지하거나 막는 방법은 언어마다 다르다.
  이런 기법을 임시로 활용해보면 변경하는 부분을 없앨 수도 있고, 적절한 변경 함수를 제공할 수도 있다.
  적절히 다 처리하고 난 뒤 게터가 복제본을 반환하도록 수정하면 된다.
  지금까지는 게터에서 데이터를 복제하는 방법을 살펴봤는데, 세터에서도 복제본을 만드는 편이 좋을 수도 있다.
  정확한 기준은 그 데이터가 어디서 오는지, 원본 데이터의 모든 변경을 그대로 반영할 수 있도록
  원본으로의 링크를 유지해야하는 지에 따라 다르다.
  링크가 필요 없다면 데이터를 복제해 저장하여 나중에 원본이 변경되서 발생하는 사고를 방지할 수 있다.
  복제본 만들기가 번거로울 때가 많지만, 이런 복제가 성능에 주는 영향은 대체로 미미하다.
  반면, 원본을 그대로 사용하면 나중에 디버깅하기 어렵고 시간도 오래 걸릴 위험이 있다.
  여기서 명심해야 할 점은 복제본 만들기와 클래스로 감싸는 방식은 레코드 구조에서 깊이가 1인 속성들만 효과가 있다.
  더 깊이 들어가면 단계가 더 늘어나게 된다.
  데이터 캡슐화는 굉장히 유용하지만 그 과정은 간단하지 않을 때가 많다.
  캡슐화의 구체적인 대상과 방법은 캡슐화할 데이터를 사용하는 방식과 그 데이터의 변경 방식에 따라 달라진다.
  하지만 분명한 사실은 데이터의 사용 범위가 넓을수록 적절히 캡슐화하는 게 좋다는 것이다.

## 7. 변수 이름 바꾸기(Rename Variable)

```jsx
let a = height * width;
```

```jsx
let area = height * width;
```

### 배경

명확한 프로그래밍의 핵심은 이름짓기다.

변수는 프로그래머가 하려는 일에 관해 많은 것을 설명해준다. 단, 이름을 잘 지었을 때만 그렇다.

이름의 중요성은 그 사용 범위에 영향을 많이 받는다.

한 줄짜리 람다식에서 사용하는 변수는 대체로 쉽게 파악할 수 있다.

맥락으로부터 변수의 목적을 명확히 알 수 있어서 한 글자로 된 이름을 짓기도 한다.

마찬가지로 간단한 함수의 매개변수 이름도 짧게 지어도 될 때가 많다.

함수 호출 한 번으로 끝나지 않고 값이 영속되는 필드라면 이름에 더 신경 써야 한다.

### 절차

1. 폭 넓게 쓰이는 변수라면 [변수 캡슐화하기]를 고려한다.
2. 이름을 바꿀 변수를 참조하는 곳을 모두 찾아서, 하나씩 변경한다.

   → 다른 코드베이스에서 참조하는 변수는 외부에 공개된 변수이므로 이 리팩터링을 적용할 수 없다.

   → 변수 값이 변하지 않는다면 다른 이름으로 복제본을 만들어서 하나씩 점진적으로 테스트하며 변경한다.

3. 테스트한다.

### 예시

- 일반 변수 이름 바꾸기
  변수 이름 바꾸기에 가장 간단한 예는 임시 변수나 인수처럼 유효범위가 함수 하나로 국한된 변수다.
  변수를 참조하는 코드를 찾아서 하나씩 바꾸면 되며, 다 바꾼 뒤에는 테스트한다.
  함수 밖에서도 참조할 수 있는 변수라면 조심해야 한다. 코드베이스 전체에서 두루 참조할 수도 있다.

  ```jsx
  let tpHd = "untitled";
  ```

  어떤 참조는 다음과 같이 변수를 읽기만 한다.

  ```jsx
  result += `<h1>${tpHd}</h1>`;
  ```

  값을 수정하는 곳도 있다고 해보자.
  [1] 나는 이럴 때 주로 [변수 캡슐화하기]로 처리한다.

  ```jsx
  result += `<h1>${title()}</h1>`;

  setTitle(obj["articleTitle"]);

  function title() {
    return tpHd;
  }
  function setTitle(arg) {
    tpHd = arg;
  }
  ```

  캡슐화 후에는 변수 이름을 바꿔도 된다.

  ```jsx
  let title = "untitled";

  function title() {
    return title;
  }
  function setTitle(arg) {
    title = arg;
  }
  ```

  [2] 그런 다음 래핑 함수들을 인라인해서 모든 호출자가 변수에 직접 접근하게 하는 방법도 있지만, 나는 지양한다.
  이름을 바꾸기 위해 캡슐화부터 해야 할 정도로 널리 사용되는 변수라면
  나중을 위해서라도 함수 안에 캡슐화된 채로 두는 편이 좋다고 생각하기 때문이다.

- 상수 이름 바꾸기
  상수(또는 상수 처럼 사용되는 변수)의 이름은 캡슐화하지 않고노 복제 방식으로 점진적으로 바꿀 수 있다.
  ```jsx
  const cpyNm = "애크미 구스베리";
  ```
  먼저 원본의 이름을 바꾼 후, 원본의 원래 이름(기존 이름)과 같은 복제본을 만든다.
  ```jsx
  const companyName = "애크미 구스베리";
  const cpyNm = companyName;
  ```
  이제 기존 이름(복제본)을 참조하는 코드들을 새 이름으로 점진적으로 바꿀 수 있다.
  다 바꿨다면 복제본을 삭제한다.
  기존 이름을 삭제했다가 테스트에 실패하면 앞의 예시가 조금이라도 쉽다면 이를 선택한다.
  이 방식은 상수는 물론, 읽기 전용 변수(javascript export variable)에도 적용할 수 있다.

## 8. 매개변수 객체 만들기(Introduce Parameter Object)

```jsx
function amountInvoice(startDeate, endDate) {...}
function amountReceived(startDeate, endDate) {...}
function amountOverdue(startDeate, endDate) {...}
```

```jsx
function amountInvoice(aDateRange) {...}
function amountReceived(aDateRange) {...}
function amountOverdue(aDateRange) {...}
```

### 배경

데이터 항목 여러 개가 여러 함수에 함께 몰려다니는 경우를 자주 본다.

이런 데이터 무리를 발견하면 데이터 구조 하나로 모아주곤 한다.

데이터 뭉치를 데이터 구조로 묶으면 데이터 사이의 관계가 명확해진다는 이점을 얻는다.

게다가 함수가 이 데이터 구조를 받게 하면 매개변수 수가 줄어든다.

같은 데이터 구조를 사용하는 모든 함수가 원소를 참조할 때 항상 똑같은 이름을 사용하기 때문에 일관성도 높여준다.

하지만 이 리팩터링의 진정한 힘은 코드를 더 근본적으로 바꿔준다는 데 있다.

나는 이런 데이터 구조를 새로 발견하면 이 데이터 구조를 활용하는 형태로 프로그램 동작을 재구성한다.

데이터 구조에 담길 데이터에 공통으로 적용되는 동작을 추출해서 함수로 만드는 것이다.

(공용 함수를 나열하는 식으로 작성할 수도 있고, 이 함수들과 데이터를 합쳐 클래스로 만들 수도 있다)

이 과정에서 새로 만든 데이터 구조가 문제 영역을 훨씬 간결하게 표현하는 새로운 추상 개념으로 격상되면서,

코드의 개념적인 그림을 다시 그릴 수도 있으며, 이는 놀라운 강력한 효과를 낸다.

하지만 이 모든 것의 시작은 매개변수 객체 만들기부터다.

### 절차

1. 적당한 데이터 구조가 아직 마련되어 있지 않다면 새로 만든다.

   → 개인적으로 클래스를 만드는 걸 선호한다. 나중에 동작까지 함께 묶기 좋기 때문이다.

2. 테스트한다.
3. [함수 선언 바꾸기]로 새 데이터 구조를 매개변수로 추가한다.
4. 테스트한다.
5. 함수 호출 시 새로운 데이터 구조 인스턴스를 넘기도록 수정한다. 하나씩 수정할 때마다 테스트한다.
6. 기존 매개변수를 사용하던 코드를 새 데이터 구조의 원소를 사용하도록 바꾼다.
7. 다 바꿨다면 기존 매개변수를 제거하고 테스트한다.

### 예시

온도 측정값(reading) 배열에서 정상 작동 범위를 벗어난 것이 있는지 검사하는 코드다.

> 온도 측정값을 표현하는 데이터

```jsx
const station = {
  name: "ZB1",
  readings: [
    { temp: 47, time: "2016-11-10 09:10" },
    { temp: 53, time: "2016-11-10 09:20" },
    { temp: 58, time: "2016-11-10 09:30" },
    { temp: 53, time: "2016-11-10 09:40" },
    { temp: 51, time: "2016-11-10 09:50" },
  ],
};
```

> 정상 범위를 벗어난 측정값을 찾는 함수

```jsx
function readingsOutsideRange(station, min, max) {
  return station.readings.finter((r) => r.temp < min || r.temp > max);
}
```

이 함수는 다음과 같이 호출될 수 있다.

```jsx
alerts = readingsOutsideRange(
  station,
  operatingPlan.temperatureFloor, // 최저 온도
  operatingPlan.temperatureCeiling
); // 최고 온도
```

operatingPlan의 데이터 항목 두 개를 쌍으로 가져와서 함수로 전달한다.

그러고 operatingPlan은 범위의 시작과 끝 이름을 함수와 다르게 표현한다.

이와 같은 범위(range)라는 개념은 객체 하나로 묶어 표현하는 게 나은 대표적인 예다.

[1] 묶은 데이터를 표현하는 클래스부터 선언한다.

```jsx
class NumberRange {
  constructor(min, max) {
    this.data = { min: min, max: max };
  }
  get min() {
    return this.data.min;
  }
  get max() {
    return this.data.max;
  }
}
```

이 리팩터링은 새로 생성한 객체로 동작까지 옮기는 더 큰 작업의 첫 단계로 수행되는 경우가 많기 떄문에 class로 선언했다.

한편 값 객체로 만들 가능성이 높기 때문에 세터는 만들지 않는다. 나는 이 리팩터링을 할 때는 대부분 값 객체로 만들게 된다.

[3] 그런 다음 새로 만든 객체를 사용하던 함수의 매개변수로 추가하도록 함수 선언을 바꾼다.

```jsx
function readingsOutsideRange(station, min, max, range) {
  return station.readings.finter((r) => r.temp < min || r.temp > max);
}
```

자바스크립트라면 호출문을 예전 상태로 둬도 되지만, 다른 언어라면 새 매개변수 자리에 null을 넣는다.

[4] 테스트는 문제없이 통과하며, [5] 온도 범위를 객체 형태로 전달하도록 호출문을 하나씩 바꾼다.

```jsx
const range = new NumberRange(
  operatingPlan.temperatureFloor,
  operatingPlan.temperatureCeiling
);
alerts = readingsOutsideRange(
  station,
  operatingPlan.temperatureFloor,
  operatingPlan.temperatureCeiling,
  range
);
```

[6] 기존 매개변수를 사용하는 부분을 변경한다.

이때, 하나씩 변경하며, 테스트도 진행한다.

```jsx
function readingsOutsideRange(station, range) {
  return station.readings.finter(
    (r) => r.temp < range.min || r.temp > range.max
  );
}

const range = new NumberRange(
  operatingPlan.temperatureFloor,
  operatingPlan.temperatureCeiling
);
alerts = readingsOutsideRange(station, range);
```

### 진정한 값 객체(VO)로 거듭나기

앞서 말했듯 매개변수 그룹을 객체로 교체하는 일은 준비단계일 뿐이다.

이 예에서 사용하던 메서드를 클래스에 추가할 수 있다.

```jsx
function readingsOutsideRange(station, range){
	return station.readings.finter(r => r.temp < range.min || r.temp > range.max);
}

class NumberRange{
	...
	contains(arg) { return (arg >= this.min && arg <= this.max); }
	...
}
```

지금까지 한 작업은 여러 가지 유용한 동작을 갖춘 범위(Range) 클래스를 생성하기 위한 첫 단계다.

코드에 범위 개념이 필요함을 깨달았다면 min-max 쌍을 사용하는 코드를 발견할 때마다 범위 객체로 바꾸자

이러한 값 쌍이 어떻게 사용되는지 살펴보면 다른 유용한 동작도

범위 클래스로 옮겨서 코드베이스 전반에서 값을 활용하는 방식을 간소화할 수 있다.

나라면 진정한 VO로 만들기 위해 값에 기반한 동치성 검사 메서드(equality method)부터 추가할 것이다.

## 9. 여러 함수를 클래스로 묶기(Combine Functions into Class)

```jsx
function base(aReading) {...}
function taxableCharge(aReading) {...}
function calculateBaseCharge(aReading) {...}
```

```jsx
class Reading {
	base() {...}
	taxableCharge() {...}
	calculateBaseCharge() {...}
}
```

### 배경

클래스는 대다수의 최신 프로그래밍 언어가 제공하는 기본적인 빌딩 블록이다.

클래스는 데이터와 함수를 하나의 공유 환경으로 묶은 후, 다른 프로그램 요소와 어우러질 수 있도록 그중 일부를 외부에 제공한다.

클래스는 객체 지향 언어의 기본인 동시에 다른 패러다임 언어에도 유용하다.

나는 흔히 함수 호출 시 인수로 전달되는 공통 데이터를 중심으로

긴밀하게 엮여 작동하는 함수 무리를 발견하면 클래스 하나로 묶고 싶어진다.

클래스로 묶으면 이 함수들이 공유하는 공통 환경을 더 명확하게 표현할 수 있고,

각 함수에 전달되는 인수를 줄여서 객체 안에서의 함수 호출을 간결하게 만들 수 있다.

또한 이런 객체를 시스템의 다른 부분에 전달하기 위한 참조를 제공할 수 있다.

이 리팩터링은 이미 만들어진 함수들을 재구성할 때는 물론,

새로 만든 클래스와 관련하여 놓친 연산을 찾아서 새 클래스의 메서드로 뽑아내는 데도 좋다.

함수를 한데 묶는 또 다른 방법은 [여러 함수를 변환 함수로 묶기]도 있다.

어느 방식으로 진행할지는 프로그램 문맥을 넓게 살펴보고 정해야 한다.

클래스로 묶을 때 장점은 클라이언트가 객체의 핵심 데이터를 변경할 수 있고,

파생 객체들을 일관되게 관리할 수 있다는 것이다.

이런 함수들을 중첩 함수 형태로 묶어도 된다.

나는 중첩 함수보다 클래스를 선호하는 편인데, 중첩 함수는 테스트하기 까다롭기 때문이다.

또한 한 울타리로 묶을 함수들 중 외부에 공개할 함수가 여러 개일 때는 클래스를 사용해야 한다.

클래스를 지원하지 않는 언어를 사용할 때는 ‘함수를 객체처럼(Function As Object) 패턴’을 이용해 구현하기도 한다.

### 절차

1. 함수들이 공유하는 공통 데이터 레코드를 캡슐화(7.1)한다.

   → 공통 데이터가 레코드 구조로 묶여 있지 않다면 먼저 [매개변수 객체 만들기]를 진행한다.

2. 공통 레코드를 사용하는 함수 각각을 새 클래스로 옮긴다(함수 옮기기[8.1])

   → 공통 레코드의 멤버는 함수 호출문의 인수 목록에서 제거한다.

3. 데이터를 조작하는 로직들은 [함수로 추출]해서 새 클래스로 옮긴다.

### 예시

정부에서 차(tea)를 수돗물처럼 제공하는 예를 떠올려봤다.

사람들은 매달 차 계량기를 읽어서 측정값(reading)을 다음과 같이 기록한다고 하자.

```jsx
reading = { customer: "ivan", quantity: 10, month: 5, year: 2017 };
```

이런 레코드를 처리하는 코드를 훑어보니 이 데이터로 비슷한 연산을 수행하는 부분이 상당히 많았다.

기본 요금을 계산하는 코드는 다음과 같다.

```jsx
const aReading = acquireReading();
const baseCharge = baseRate(aReading.month, aReading.year) * aReading.quantity;
```

차에도 세금이 부과되지만, 기본적인 차 소비량만큼은 면세이다.

```jsx
const aReading = acquireReading();
const base = baseRate(aReading.month, aReading.year) * aReading.quantity;
const taxableCharge = Math.min(0, base - taxThreshold(aReading.year));
```

여기서도 기본요금 계산 공식이 똑같이 등장하는 것을 발견했다.

함수로 추출하려는데, 마침 이런 코드를 발견했다.

```jsx
const aReading = acquireReading();
const basicChargeAmount = calculateBaseCharge(aReading);

function calculateBaseCharge(aReading) {
  return baseRate(aReading.month, aReading.year) * aReading.quantity;
}
```

앞에 두 예시도 이 함수를 사용하게 고치려고 하지만, 최상위 함수로 두면 못 보고 지나치기 쉽다.

나라면 이런 함수를 데이터 처리 코드 가까이에 둔다. 그러기 위한 좋은 방법으로 데이터를 클래스로 만든다.

[1] 먼저 레코드를 클래스로 변환하기 위해 레코드를 캡슐화(7.1)한다.

```jsx
class Reading {
  constructor(data) {
    this.customer = data.customer;
    this.quantity = data.quantity;
    this.month = data.month;
    this.year = data.year;
  }
  get customer() {
    return this.customer;
  }
  get quantity() {
    return this.quantity;
  }
  get month() {
    return this.month;
  }
  get year() {
    return this.year;
  }
}
```

[2] 이미 만들어져 있는 calculateBaseCharge()부터 옮긴다. 새 클래스를 사용하려면 데이터를 얻자마자 객체로 만들어야 한다.

```jsx
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const basicChargeAmount = calculateBaseCharge(aReading);
```

그런 다음 calculateBaseCharge()를 새로 만든 클래스로 옮긴다(함수 옮기기[8.1])

```jsx
class Reading {
	...
	get calculateBaseCharge() {
		return baseRate(this.month, this.year) * this.quantity;
	}
	...
}
```

```jsx
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const basicChargeAmount = aReading.calculateBaseCharge;
```

이 과정에서 메서드 이름을 원하는대로 바꾼다([함수 이름 바꾸기])

```jsx
class Reading {
	...
	get baseCharge() {
		return baseRate(this.month, this.year) * this.quantity;
	}
	...
}
```

이렇게 이름을 바꾸고 나면 Reading 클래스의 클라이언트는 baseCharge가 필드인지, 함수 호출인지 구분할 수 없다.

이는 단일 접근 원칙(Uniform Access Principle)을 따르므로 권장하는 방식이다.

이제 첫 번째 클라이언트에서 중복된 계산 코드를 고쳐 앞의 메서드를 호출하게 한다.

```jsx
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const baseCharge = aReading.baseCharge;
```

이런 코드들을 보면 baseCharge [변수를 인라인]하고 싶어진다.

하지만 세금을 계산하는 클라이언트부터 인라인하는 일이 절실하다.

```jsx
const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const taxableCharge = Math.min(
  0,
  aReading.baseCharge - taxThreshold(aReading.year)
);
```

[3] 이어서 세금을 부과할 소비량을 계산하는 코드도 동일한 절차로 추출한다.

```jsx
class Reading {
	...
	get taxableCharge() {
		return  Math.min(0, this.baseCharge- taxThreshold(this.year));
	}
	...
}

const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const taxableCharge = aReading.taxableCharge;
```

파생 데이터 모두를 필요한 시점에 계산되게 만들었으니 저장된 데이터를 갱신하더라도 문제가 생길 일이 없다.

대체로 불변 데이터를 선호하지만 어쩔 수 없이 가변 데이터를 사용해야 할 때가 많다.

프로그램의 다른 부분에서 데이터를 갱신할 가능성이 꽤 있을 때는 클래스로 묶어두면 큰 도움이 된다.

## 10. 여러 함수를 변환 함수로 묶기(Combine Function into Transform)

```jsx
function base(aReading) {...}
function taxableCharge(aReading) {...}
```

```jsx
function enrichReading(argReading) {
	const aReading = cloneDeep(argReading);
	aReading.baseCharge = base(aReading);
	aReading.taxableCharge  taxableCharge(aReading);
	return aReading;
}
```

### 배경

소프트웨어는 데이터를 입력받아서 여러가지 정보를 도출하곤 한다.

도출된 정보는 여러 곳에서 사용되는데, 이 정보가 사용되는 곳마다 같은 도출 로직이 반복되기도 한다.

나는 이런 도출 작업들을 한데로 모아두길 좋아한다.

이럭헤 하기 위한 방법으로 변환 함수(transform)를 사용할 수 있다.

변환 함수는 원본 데이터를 입력받아서 필요한 정보를 모두 도출한 뒤,

각각을 출력 데이터의 필드에 넣어 반환한다.

이렇게 해두면 도출 과정을 검토할 일이 생겼을 때 변환 함수만 살펴보면 된다.

이 리팩터링 대신 [여러 함수를 클래스로 묶기]로 처리해도 된다.
그런데 둘 사이에는 중요한 차이가 하나 있다.

원본 데이터가 코드 안에서 갱신될 때는 클래스로 묶는 편이 훨씬 났다.

변환 함수로 묶으면 가공한 데이터를 새로운 레코드에 저장하므로, 원본 데이터가 수정되면 일관성이 깨진다.

여러 함수를 묶는 이유 하나는 도출 로직이 중복되는 것을 피하기 위해서다.

이 로직을 함수로 추출하는 것만으로도 같은 효과를 볼 수 있지만,

데이터 구조와 사용하는 함수가 근처에 있지 않으면 함수를 발견하기 어려울 때가 많다.

변환 함수(또는 클래스)로 묶으면 이런 함수들을 쉽게 찾아 쓸 수 있다.

### 절차

1. 변환할 레코드를 입력받아서 값을 그대로 반환하는 반환 함수를 만든다.

   → 이 작업은 대체로 깊은 복사로 처리해야 한다.

   변환 함수가 원본 레코드를 바꾸지 않는지 검사하는 테스트를 마련해두면 도움이 된다.

2. 묶을 함수 중 함수 하나를 골라서 본문 코드를 변환 함수로 옮기고,

   처리 결과를 레코드에 새 필드로 기록한다. 그런 다음 클라이언트 코드가 이 코드를 사용하도록 수정한다.

3. 테스트한다.
4. 나머지 관련 함수도 위 과정에 따라 처리한다.

### 예시

서민에게 차(tea)를 수돗물처럼 제공하는 서비스를 상상하게 됐다.

이런 서비스가 있다면 매달 사용자가 마신 차의 양을 측정해야 한다.

```jsx
reading = { customer: "ivan", quantity: 10, month: 5, year: 2017 };
```

코드 곳곳에서 다양한 방식으로 차 소비량을 계산한다고 해보자.

그중 사용자에게 요금을 부과하기 위해 기본요금을 계산하는 코드도 있다.

```jsx
const aReading = acquireReading();
const baseCharge = baseRate(aReading.month, aReading.year) * aReading.quantity;
```

세금을 부과할 소비량을 계산하는 코드도 필요하다.

모든 시민이 차 세금을 일부 면제받을 수 있게 이 값은 기본 소비량보다 작다.

```jsx
const aReading = acquireReading();
const base = baseRate(aReading.month, aReading.year) * aReading.quantity;
const taxableCharge = Math.min(0, base - taxThreshold(aReading.year));
```

이 코드에는 이와 같은 계산 코드가 여러 곳에 반복된다고 해보자.

중복 코드는 나중에 로직을 수정할 때 골치를 썩인다.

함수 추출하기로 처리할 수도 있지만, 아래와 같이 이미 함수가 추출되어 있어도, 그런 함수가 있는지 조차 모를 수도 있다.

```jsx
const aReading = acquireReading();
const basicChargeAmount = calculateBaseCharge(aReading);

function calculateBaseCharge(aReading) {
  return baseRate(aReading.month, aReading.year) * aReading.quantity;
}
```

이를 해결하는 방법으로, 변환 단계로 모을 수 있다.

변환 단계에서 미가공 측정값을 입력받아서 다양한 가공 정보를 덧붙여 반환하는 것이다.

[1] 우선 입력 객체를 그대로 복사해 반환하는 변환 함수를 만든다.

```jsx
function enrichReading(original) {
  const result = cloneDeep(original);
  return result;
}
```

깊은 복사는 lodash 라이브러리가 제공하는 cloneDeep()으로 처리했다.

<aside>
🗒️ 본질이 같고 부가 정보만 덧붙이는 변환 함수는 ‘enrich’라 하고, 형태가 변하면 transform이라 한다.

</aside>

[2] 이제 변경하려는 계산 로직 중 하나를 고른다.

먼저 이 계산 로직에 측정값을 전달하기 전에 부가 정보를 덧붙이도록 수정했다.

```jsx
const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const basicChargeAmount = calculateBaseCharge(aReading);
```

calculateBaseCharge()를 부가 정보를 덧붙이는 코드 근처로 옮긴다.

```jsx
function enrichReading(original) {
  const result = cloneDeep(original);
  result.baseCharge = calculateBaseCharge(result);
  return result;
}
```

변환 함수 안에서는 결과 객체를 매번 복제할 필요 없이 마음껏 변경해도 된다.

나는 불변 데이터를 선호하지만 널리 사용되는 언어는 대부분 불변 데이터를 다루기 어렵게 돼 있다.

데이터가 모듈 경계를 넘나든다면 어려움을 기꺼이 감내하며 불변으로 사용하겠지만,

데이터의 유효범위가 좁을 때는 마음껏 변경한다.

또한 나는 변환 함수로 옮기기 쉬운 이름을 붙인다. (여기서는 aReading이라고 지었다)

이어서 함수를 사용하던 클라이언트가 부가 정보를 담은 필드를 사용하도록 수정한다.

```jsx
const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const basicChargeAmount = aReading.baseCharge;
```

calculateBaseCharge()를 호출하는 코드를 모두 수정했따면, 이 함수를 변환 함수 안에 중첩시킬 수 있다.

그러면 ‘기본요금을 이용하는 클라이언트는 변환된 레코드를 사용해야 한다’는 의도를 명확히 표현할 수 있다.

여기서 주의할 점은 enrichReading()처럼 정보를 추가해 반환할 때 원본 측정값 레코드는 변경하지 않아야 한다는 것이다.

따라서 이를 확인하는 테스트를 작성해두는 것이 좋다.

```jsx
it("check reading unchanged", function () {
  const baseReading = { customer: "ivan", quantity: 10, month: 5, year: 2017 };
  const oracle = cloneDeep(baseReading);
  enrichReading(baseReading);
  assert.deepEqual(baseReading, oracle);
});
```

그런 다음 첫 번째 예시 코드도 이 필드를 사용하도록 수정한다.

```jsx
const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const baseCharge = aReading.baseCharge;
```

이때 baseCharge 변수도 인라인하면 좋다.

[4] 이제 세금을 부과할 소비량 계산도 동일한 방식으로 수정한다.

```jsx
function enrichReading(original) {
  const result = cloneDeep(original);
  result.baseCharge = calculateBaseCharge(result);
  result.taxableCharge = Math.min(
    0,
    result.baseCharge - taxThreshold(result.year)
  );
  return result;
}

const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const taxableCharge = aReading.taxableCharge;
```

테스트에 성공하면 taxableCharge 변수를 인라인한다.

### 자바스크립트에서의 문제점

예시 코드에서 측정값에 부가 정보를 추가하는 지금 방식에서 클라이언트가 데이터를 변경하면 심각한 문제가 생긴다.

사용량 필드를 변경하면 데이터의 일관성이 깨진다. 자바스크립트에서는 여러 함수를 클래스로 묶기가 더 적절하다.

불변 데이터 구조를 지원하는 언어라면 이런 문제가 생길 일이 없다.

하지만 불변성을 제공하지 않는 언어라도,

웹 페이지에 출력할 부가 데이터를 도출할 때처럼 데이터가 읽기전용 문맥에서 사용될 때는 변환 방식을 활용할 수 있다.

## 11. 단계 쪼개기(Split Phase)

```jsx
const orderData = orderString.split(/\s+/);
const productPrice = priceList[orderData[0].split("-")[1]];
const orderPrice = parseInt(orderData[1]) * productPrice;
```

```jsx
const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString) {
  const values = aString.split(/\s+/);
  return {
    productID: values[0].split("-")[1],
    quantity: parseInt(values[1]),
  };
}
function price(order, priceList) {
  return order.quantity * priceList[order.productID];
}
```

### 배경

나는 서로 다른 두 대상을 한꺼번에 다루는 코드를 발견하면 각각을 별개 모듈로 나누는 방법을 모색한다.

코드를 수정해야 할 때 두 대상을 동시에 생각할 필요 없이 하나에만 집중하기 위해서다.

모듈이 잘 분리되어 있다면 다른 모듈의 상세 내용은 전혀 기억하지 못해도 원하는 대로 수정을 할 수 있다.

이렇게 분리하는 가장 간편한 방법 하나는 동작을 연이은 두 단계로 쪼개는 것이다.

입력이 처리 로직에 적합하지 않은 형태로 들어오는 경우를 예로 생각해보자.

이럴 때는 본 작업에 들어가기 전에 입력값을 다루기 편한 형태로 가공한다.

아니면 로직을 순차적인 단계들로 분리해도 된다.

이때 각 단계는 서로 확연히 다른 일을 수행해야 한다.

가장 대표적인 예는 컴파일러다. 컴파일러는 기본적으로 어떤 텍스트(프로그래밍 언어로 작성된 코드)를

입력받아서 실행 가능한 형태(예컨대 특정 하드웨어에 맞는 목적 코드[object code])로 변환한다.

컴파일러의 역사가 오래되다 보니 사람들은 컴파일 작업을 여러 단계가 순차적으로 연결된 형태로 분리하면 좋다는 사실을 깨달았다.

즉, 텍스트를 토큰화하고, 토큰을 파싱해서 구문 트리를 만들고, 최적화 등 구문 트리를 변환하는 다양한 단계를 거친 다음,

마지막으로 목적 코드를 생성하는 식이다.

각 단계는 자신만의 문제에 집중하기 때문에 나머지 단계에 관해서는 자세히 몰라도 이해할 수 있다.

이렇게 단계를 쪼개는 기법은 주로 덩치 큰 소프트웨어에 적용된다. 가령 컴파일러의 매 단계는 다수의 함수와 클래스로 구성된다.

하지만 나는 규모에 관계없이 여러 단계로 분리하면 좋을만한 코드를 발견할 때마다 기본적인 단계 쪼개기 리팩터링을 한다.

다른 단계로 볼 수 있는 코드 영역들이 마침 서로 다른 데이터와 함수를 사용한다면 단계 쪼개기에 적합하다는 뜻이다.

이 코드 영역들을 별도 모듈로 분리하면 그 차이를 코드에서 훨씬 분명하게 드러낼 수 있다.

### 절차

1. 두 번째 단계에 해당하는 코드를 독립 함수로 추출한다.
2. 테스트한다.
3. 중간 데이터 구조를 만들어서 앞에서 추출한 함수의 인수로 추가한다.
4. 테스트한다.
5. 추출한 두 번째 단계 함수의 매개변수를 하나씩 검토한다.

   그중 첫 번째 단계에서 사용되는 것은 중간 데이터 구조로 옮긴다. 하나씩 옮길 때마다 테스트한다.

   → 간혹 두 번째 단계에서 사용하면 안 되는 매개변수가 있다. 이럴 때는 각 매개변수를 사용한 결과를

   중간 데이터 구조의 필드로 추출하고, 이 필드의 값을 설정하는 문장을 호출한 곳으로 옮긴다.

6. 첫 번째 단계 코드를 함수로 추출하면서 중간 데이터 구조를 반환하도록 만든다.

   → 이때 첫 번째 단계를 변환기 객체로 추출해도 좋다.

### 예시

- 기본 예시
  상품의 결제 금액을 계산하는 코드로 시작해보자.
  ```jsx
  function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount =
      Math.max(quantity - product.discountThreshold, 0) *
      product.basePrice *
      product.discountRate;
    const shippingPerCase =
      basePrice > shippingMethod.discountThreshold
        ? shippingMethod.discountFee
        : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = basePrice - discount + shippingCost;
    return price;
  }
  ```
  간단한 예지만, 가만 보면 계산이 두 단계로 이뤄짐을 알 수 있다.
  앞에 몇 줄은 상품 정보를 이용해서 결제 금액 중 상품 가격을 계산한다.
  반면 뒤의 코드는 배송 정보를 이용하여 결제 금액 중 배송비를 계산한다.
  나중에 상품 가격과 배송비 계산을 더 복잡하게 만드는 변경이 생긴다면 이 코드는 두 단계로 나누는 것이 좋다.
  [1] 먼저 배송비 계산 부분을 함수로 추출한다.
  ```jsx
  function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
    const discount =
      Math.max(quantity - product.discountThreshold, 0) *
      product.basePrice *
      product.discountRate;
    const price = applyShipping(basePrice, shippingMethod, quantity, discount); // <- 함수로 추출
    return price;
  }

  function applyShipping(basePrice, shippingMethod, quantity, discount) {
    const shippingPerCase =
      basePrice > shippingMethod.discountThreshold
        ? shippingMethod.discountFee
        : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = basePrice - discount + shippingCost;
    return price;
  }
  ```
  두 번째 단계에 필요한 데이터를 모두 개별 매개변수로 전달했다.
  실전에서는 이런 데이터가 상당히 많을 수 있는데, 어차피 나중에 걸러내기 때문에 걱정할 필요는 없다.
  [3] 다음으로 첫 번째 단계와 두 번째 단계가 주고받을 중간 데이터 구조를 만든다.
  ```jsx
  function priceOrder(product, quantity, shippingMethod){
  	...
  	const priceData = {};
    const price = applyShipping(priceData, basePrice, shippingMethod, quantity, discount); // <- 함수로 추출
    return price;
  }

  function applyShipping(priceData, basePrice, shippingMethod, quantity, discount){
    const shippingPerCase =
    (basePrice > shippingMethod.discountThreshold)
    ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = basePrice - discount + shippingCost;
    return price;
  }
  ```
  [5] 이제 applyShipping()에 전달되는 다양한 매개변수를 살펴보자.
  이중 basePrice는 첫 번째 단계를 수행하는 코드에서 생성된다. 따라서 중간 데이터 구조로 옮기고 매개변수 목록에서 제거한다.
  ```jsx
  function priceOrder(product, quantity, shippingMethod){
  	...
  	const priceData = {basePrice : basePrice};
    const price = applyShipping(priceData, shippingMethod, quantity, discount);
    return price;
  }

  function applyShipping(priceData, shippingMethod, quantity, discount){
    const shippingPerCase =
    (priceData.basePrice > shippingMethod.discountThreshold)
    ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = quantity * shippingPerCase;
    const price = priceData.basePrice - discount + shippingCost;
    return price;
  }
  ```
  다음은 shippingMethod를 보자. 이 매개변수는 첫 번째 단계에서는 사용하지 않으니 그대로 둔다.
  그 다음 나오는 quantity는 첫 번째 단계에서 사용하지만 거기서 생성된 것은 아니다.
  그래서 그냥 매개변수로 놔둬도 된다. 하지만 나는 최대한 중간 데이터 구조에 담는걸 선호하기 때문에 이 매개변수도 옮긴다.
  discount도 같은 방법으로 처리한다.
  ```jsx
  function priceOrder(product, quantity, shippingMethod){
  	...
  	const priceData = {basePrice : basePrice, quantity : quantity, discount : discount};
    const price = applyShipping(priceData, shippingMethod); // <- 함수로 추출
    return price;
  }

  function applyShipping(priceData, shippingMethod){
    const shippingPerCase =
    (priceData.basePrice > shippingMethod.discountThreshold)
    ? shippingMethod.discountFee : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    const price = priceData.basePrice - priceData.discount + shippingCost;
    return price;
  ```
  매개변수들을 모두 처리하면 중간 데이터 구조가 완성된다.
  [6] 이제 첫 번째 단계 코드를 함수로 추출하고 이 데이터 구조를 반환하게 한다.
  ```jsx
  function priceOrder(product, quantity, shippingMethod) {
    const priceData = calculatePricingData(product, quantity);
    return applyShipping(priceData, shippingMethod);
  }

  function calculatePricingData(product, quantity) {
    const basePrice = product.basePrice * quantity;
    const discount =
      Math.max(quantity - product.discountThreshold, 0) *
      product.basePrice *
      product.discountRate;
    return { basePrice: basePrice, quantity: quantity, discount: discount };
  }

  function applyShipping(priceData, shippingMethod) {
    const shippingPerCase =
      priceData.basePrice > shippingMethod.discountThreshold
        ? shippingMethod.discountFee
        : shippingMethod.feePerCase;
    const shippingCost = priceData.quantity * shippingPerCase;
    const price = priceData.basePrice - priceData.discount + shippingCost;
    return price;
  }
  ```
- 명령줄 프로그램 쪼개기 (자바)
  JSON 파일에 담긴 주문의 개수를 세는 자바 프로그램을 살펴보자.
  ```java
  public static void main(String[] args) {
    try{
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
      String filename = args[args.length - 1];
      File input = Paths.get(filename).toFile();
      ObjectMapper mapper = new ObjectMapper();
      Order[] orders = mapper.readValue(input, Order[].class);
      if (Stream.of(args).anyMatch(arg -> "r".equals(arg)))
        System.out.println(Stream.of(orders)
                                 .filter(o -> "ready".equals(o.status))
                                 .count());
      else
        System.out.println(orders.length);
    }catch(Excedption e){
      System.err.println(e);
      System.exit(1);
    }
  }
  ```
  이 프로그램은 명령줄에서 실행할 때 주문이 담긴 파일 이름을 인수로 받는다.
  이때 옵션인 -r 플래그를 지정하면 “ready” 상태인 주문만 센다.
  ***
  주문을 읽는 부분은 Jackson 라이브러리를 이용했다. ObjectMapper는 각 JSON 레코드를 간단히 public 데이터 필드로 구성된 자바 객체로 매핑한다.
  JSON 레코드의 status 필드는 Order 클래스의 public 필드인 status에 매핑되는 식이다.
  ***
  이 코드는 두 가지 일을 한다.
  1. 주문 목록을 읽어서 개수 세기
  2. 명령줄 인수를 담은 배열을 읽어서 프로그램의 동작을 결정
  따라서 단계 쪼개기 리팩터링의 대상으로 적합하다.
  첫 번째 단계는 명령줄 인수의 구문을 분석해서 의미를 추출한다.
  두 번째 단계는 이렇게 추출된 정보를 이용하여 데이터를 적절히 가공한다.
  이렇게 분리해두면, 프로그램에서 지정할 수 있는 옵션이나 스위치가 늘어나더라도 코드를 수정하기 쉽다.
  그런데 단계 쪼개기와 상관없는 작업부터 할 것이다. 리팩터링할 때는 테스트를 작성하고 자주 수행해야 하지만,
  자바로 작성된 명령줄 프로그램은 테스트하기가 고통스럽다.
  매번 JVM을 구동해야 하는데, 그 과정이 느리고 복잡하기 때문이다.
  특히 Maven의 단점을 싫어하다면 고통이 배가된다.
  이 문제를 개선하려면 일반적인 JUnit 호출로 자바 프로세스 하나에서 테스트 할 수 있는 상태로 만들면 된다.
  이를 위해 먼저 핵심 작업을 수행하는 코드 전부를 함수로 추출한다.
  ```java
  public static void main(String[] args) {
      try{
        run(args);
      }catch(Excedption e){
        System.err.println(e);
        System.exit(1);
      }
    }

    static void run(String[] args) thorws IOException{
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
      String filename = args[args.length - 1];
      File input = Paths.get(filename).toFile();
      ObjectMapper mapper = new ObjectMapper();
      Order[] orders = mapper.readValue(input, Order[].class);
      if (Stream.of(args).anyMatch(arg -> "r".equals(arg)))
        System.out.println(Stream.of(orders)
                                 .filter(o -> "ready".equals(o.status))
                                 .count());
      else
        System.out.println(orders.length);
    }
  ```
  run() 메서드를 테스트 코드에서 쉽게 호출할 수 있도록 접근 범위를 패키지로 설정했다.
  이로써 이 메서드를 자바 프로세스 안에서 호출할 수 있지만, 결과를 받아보려면 표준 출력으로 보내는 방식을 수정해야 한다.
  이 문제는 System.out을 호출하는 문장을 호출한 곳으로 옮겨(8.4) 해결한다.
  ```java
  public class Hello{
    public static void main(String[] args) {
      try{
        System.out.println(run(args));
      }catch(Excedption e){
  			...
      }
    }

    static long run(String[] args) thorws IOException{
  			...
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
      else
        return orders.length;
    }
  }
  ```
  이렇게 하면 기본 동작을 망치지 않으면서 run() 메서드를 검사하는 JUnit 테스트를 작성할 수 있다.
  이로써 명령줄에서 매번 자바 프로세스를 새로 띄울 때보다 훨씬 빨라졌다.
  지금까지의 단계는 리팩터링 시 중요하다.
  테스트가 느리거나 불편하면 리팩터링 속도가 느려지고 오류가 생길 가능성도 커진다.
  따라서 먼저 테스트를 쉽게 수행할 수 있도록 수정한 다음 리팩터링하는 게 좋다.
  이번 예에서는 명령줄 호출과 표준 출력에 쓰는 느리고 불편한 작업과
  자주 테스트해야 할 복잡한 동작을 분리함으로써 테스트를 더 쉽게 수행하게 만들었다.
  이 원칙을 흔히 [험블 객체 패턴(Humble Object Pattern)]이라 한다.
  단, 여기서는 객체가 아니라 main() 메서드에 적용했다.
  main()에 담긴 로직을 최대한 간소하게 만들어서 문제가 생길 여지가 줄은 것이다.
  이제 단계를 쪼갤 준비가 끝났다.
  [1] 가장 먼저 할 일은 두 번째 단계에 해당하는 코드를 독립된 메서드로 추출하는 것이다.
  ```java
  static long run(String[] args) thorws IOException{
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
      String filename = args[args.length - 1];
      return countOrders(args, filename);
    }

    private static long countOrders(String[] args, String filename) throws IOException{
      File input = Paths.get(filename).toFile();
      ObjectMapper mapper = new ObjectMapper();
      Order[] orders = mapper.readValue(input, Order[].class);
      if (Stream.of(args).anyMatch(arg -> "r".equals(arg)))
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
      else
        return orders.length;
    }
  ```
  [3] 다음으로 중간 데이터 구조를 추가한다. 레코드는 단순한 게 좋은데, 자바이므로 클래스로 구현한다.
  ```java
  static long run(String[] args) thorws IOException{
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
      String filename = args[args.length - 1];
      CommandLine commandLine = new CommandLine();
      return countOrders(commandLine, args, filename);
    }

    private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException{
  		...
    }

    private static class CommandLine{}
  ```
  [5] 이제 두 번째 단계 메서드인 countOrders()로 전달되는 다른 인수들을 살펴본다.
  args는 첫 번째 단계에서 사용되는데, 이를 두 번째 단계에까지 노출하는 건 적절치 않다.
  지금 단계를 쪼개는 목적이 args를 사용하는 부분을 모두 첫 번째 단계로 분리하는 것이기 때문이다.
  args를 처리하기 위해 가장 먼저 할 일은 이 값을 사용하는 부분을 찾아서 그 결과를 추출하는 것이다.
  여기서 단 한 번, 개수를 세는 코드가 “ready” 상태인 주문만 세는지 확인하는 데 사용하므로 이 조건식을 변수로 추출한다.
  ```java
  public static void main(String[] args) {
  	...
  }

  static long run(String[] args) thorws IOException{
  	...
  }

  private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException{
  	...
    boolean onlyCountReady = Stream.of(args).anyMatch(arg -> "r".equals(arg));
    if (onlyCountReady)
      return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else
      return orders.length;
  }

  private static class CommandLine{}
  ```
  그런 다음 이 값을 중간 데이터 구조로 옮긴다.
  ```java
    public static void main(String[] args) {
  		...
    }

    static long run(String[] args) thorws IOException{
  		...
    }

    private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException{
  		...
      commandLine.onlyCountReady = Stream.of(args).anyMatch(arg -> "r".equals(arg));
      if (commandLine.onlyCountReady)
  		...
    }
  	// public 필드를 두는 것을 선호하지 않지만, 사용되는 범위가 좁기 떄문에 채택했다.
    private static class CommandLine{
      boolean onlyCountReady;
    }
  ```
  다음으로 onlyCountReady에 값을 설정하는 문장을 호출하는 곳으로 옮긴다.
  ```java
  public static void main(String[] args) {
  		...
    }

    static long run(String[] args) thorws IOException{
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
      CommandLine commandLine = new CommandLine();
      String filename = args[args.length - 1];
      commandLine.onlyCountReady = Stream.of(args).anyMatch(arg -> "r".equals(arg));
      return countOrders(commandLine, args, filename);
    }

    private static long countOrders(CommandLine commandLine, String filename) throws IOException{
  		...
    }

    private static class CommandLine{
      boolean onlyCountReady;
    }
  ```
  이어서 filename 매개변수를 CommandLine 객체로 옮긴다.
  ```java
  public static void main(String[] args) {
  		...
    }

    static long run(String[] args) thorws IOException{
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
      CommandLine commandLine = new CommandLine();
      commandLine.filename = args[args.length - 1];
      commandLine.onlyCountReady = Stream.of(args).anyMatch(arg -> "r".equals(arg));
      return countOrders(commandLine);
    }

    private static long countOrders(CommandLine commandLine) throws IOException{
      File input = Paths.get(commandLine.filename).toFile();
      ObjectMapper mapper = new ObjectMapper();
      Order[] orders = mapper.readValue(input, Order[].class);
      if (commandLine.onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
      else
        return orders.length;
    }

    private static class CommandLine{
      boolean onlyCountReady;
      String filename;
    }
  ```
  매개변수 처리가 다 끝났다.
  [6] 이제 첫 번째 단계의 코드를 메서드로 추출한다.
  ```java
  public static void main(String[] args) {
  		...
    }

    static long run(String[] args) thorws IOException{
      CommandLine commandLine = parseCommandLine(args);
      return countOrders(commandLine);
    }

    private static CommandLine parseCommandLine(String[] args){
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
      CommandLine commandLine = new CommandLine();
      commandLine.filename = args[args.length - 1];
      commandLine.onlyCountReady = Stream.of(args).anyMatch(arg -> "r".equals(arg));
      return commandLine;
    }

    private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException{
      File input = Paths.get(filename).toFile();
      ObjectMapper mapper = new ObjectMapper();
      Order[] orders = mapper.readValue(input, Order[].class);
      if (commandLine.onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
      else
        return orders.length;
    }

    private static class CommandLine{
      boolean onlyCountReady;
      String filename;
    }
  ```
  단계 쪼개기 리팩터링의 핵심은 끝났지만, 나라면 이름바꾸기와 인라인하기로 조금 더 정리한다.
  ```java
  public static void main(String[] args) {
      try{
        System.out.println(run(args));
      }catch(Excedption e){
        System.err.println(e);
        System.exit(1);
      }
    }

    static long run(String[] args) thorws IOException{
      return countOrders(parseCommandLine(args));
    }

    private static CommandLine parseCommandLine(String[] args){
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
      CommandLine result = new CommandLine();
      result.filename = args[args.length - 1];
      result.onlyCountReady = Stream.of(args).anyMatch(arg -> "r".equals(arg));
      return result;
    }

    private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException{
      File input = Paths.get(filename).toFile();
      ObjectMapper mapper = new ObjectMapper();
      Order[] orders = mapper.readValue(input, Order[].class);
      if (commandLine.onlyCountReady)
        return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
      else
        return orders.length;
    }

    private static class CommandLine{
      boolean onlyCountReady;
      String filename;
    }
  ```
  이제 두 단계가 명확하게 분리됐다.
  parseCommandLine()은 오로지 명령줄 관련 작업만 처리하고, countOrders()는 실제로 처리할 작업만 수행한다.
  이제 두 메서드를 독립적으로 테스트하기 쉬워졌다. 여기서 로직이 더 복잡해진다면 아마도
  parseCommandLine()을 더 전문화된 라이브러리도 대체할 것이다.
- 첫번째 단계에 변환기 사용하기(자바)
  명령줄 프로그램 쪼개기 예시는 첫 번째 단계에서 간단한 데이터 구조를 만들어서 두 번째 단계로 전달했다.
  이렇게 하지 않고 명령줄 인수를 담은 문자열 배열을 두 번째 단계에 적합한 인터페이스로 바꿔주는
  변환기(transformer) 객체를 만들어도 된다.
  이 방식을 설명하기 위해 앞 예시에서 두 번째 단계에 전달할 CommandLine 객체를 생성하는 부분으로 돌아가보자.
  ```java
  static long run(String[] args) thorws IOException{
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    String filename = args[args.length - 1];
    CommandLine commandLine = new CommandLine();
    return countOrders(commandLine, args, filename);
  }

  private static long countOrders(CommandLine commandLine, String[] args, String filename) throws IOException{
    File input = Paths.get(filename).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (Stream.of(args).anyMatch(arg -> "r".equals(arg)))
      return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else
      return orders.length;
  }

  private static class CommandLine{}
  ```
  앞 예시에서는 동작을 포함할 수 있는 객체 대신 레코드 구조를 만들었기 때문에,
  내부 클래스를 만들고 나중에 public 데이터 멤버로 채웠다.
  하지만 다음과 같이 동작까지 포함하는 최상위 클래스로 빼내는 방법도 있다.
  ```java
  public class CommandLine {
    String[] args;

    public CommandLine(String[] args){
      this.args = args;
    }
  }
  ```
  이 클래스는 생성자에서 인수 배열을 받아서 첫 단계 로직이 할 일을 수행한다.
  즉, 입력받은 데이터를 두 번째 단계에 맞게 변환하는 메서드들을 제공할 것이다.
  처리 과정을 확실히 이해하기 위해 countOrders()의 인수를 뒤에서부터 살펴보자.
  filename은 임시 변수를 질의 함수로 바꾸기(7.4)를 적용한다.
  ```java
  static long run(String[] args) thorws IOException{
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine, args, filename(args));
  }

  private static String filename(String[] args){
    return args[args.length - 1];
  }
  ```
  이어서 이 질의 메서드를 CommandLine 클래스로 옮긴다.(함수 옮기기[8.1])
  ```java
  static long run(String[] args) thorws IOException{
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine, args, commandLine.fileName());
  }

  private static long countOrders(String[] args, String filename) throws IOException{
  	...
  }

  public class CommandLine {
    String[] args;

    public CommandLine(String[] args){
      this.args = args;
    }

    String filename() {
      return args[args.length - 1];
    }
  }
  ```
  이제 함수 선언 바꾸기로 countOrders()가 새로 만든 메서드를 사용하도록 고친다.
  ```java
  static long run(String[] args) thorws IOException{
    if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
    CommandLine commandLine = new CommandLine(args);
    return countOrders(commandLine, args);
  }

  private static long countOrders(CommandLine commandLine, String args) throws IOException{
    File input = Paths.get(commandLine.filename()).toFile();
  	...
  }
  ```
  args는 아직 조건문을 사용하고 있기 때문에 제거하지 않는다. args를 삭제하려면 조건식부터 추출해야 한다.
  ```java
  private static long countOrders(CommandLine commandLine, String args) throws IOException{
  	...
    if (onlyCountReady(args))
  	...
  }

  private static boolean onlyCountReady(String[] args){
    return Stream.of(args).anyMatch(arg -> "r".equals(arg));
  }
  ```
  그런 다음 이 메서드를 CommandLine 클래스로 옮기고 args 매개변수를 제거한다.
  ```java
  static long run(String[] args) thorws IOException{
  	...
    return countOrders(commandLine);
  }

  private static long countOrders(CommandLine commandLine) throws IOException{
    File input = Paths.get(commandLine.filename()).toFile();
    ObjectMapper mapper = new ObjectMapper();
    Order[] orders = mapper.readValue(input, Order[].class);
    if (commandLine.onlyCountReady())
      return Stream.of(orders).filter(o -> "ready".equals(o.status)).count();
    else
      return orders.length;
  }

  public class CommandLine {
    String[] args;

    public CommandLine(String[] args){
      this.args = args;
    }

    String filename() {
      return args[args.length - 1];
    }

    boolean onlyCountReady(){
      return Stream.of(args).anyMatch(arg -> "r".equals(arg));
    }
  }
  ```
  지금까지는 변환 로직을 새 클래스로 옮기는 방식으로 리팩터링했다.
  추가로 명령줄 인수가 존재하는지 검사하는 부분도 옮겨준다.(문장을 함수로 옮기기)
  ```java
  public CommandLine(String[] args){
      this.args = args;
      if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
  }
  ```
  나는 이렇게 단순한 데이터 구조는 사용하기 꺼리는 편이다.
  하지만 이 예시처럼 순차적으로 실행되는 두 함수 사이에서
  단순 통신용으로 사용하는 것처럼 제한된 문맥에서만 사용할 때는 개의치 않는다.
  이렇게 객체를 변환기로 바꾸는 방식도 나름 장점이 있다.
  나는 두 방식 중 어느 하나를 특별히 선호하지 않는다.
  핵심은 어디까지나 단계를 명확히 분리하는 데 있기 때문이다.
