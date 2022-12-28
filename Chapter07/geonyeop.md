# [7] 캡슐화

모듈을 분리하는 가장 중요한 기준은 아마도 시스템에서 각 모듈이 자신을 제외한

다른 부분에 드러내지 않아야 할 비밀을 얼마나 잘 숨기느냐에 있을 것이다.

이러한 비밀 중 대표적인 형태인 데이터 구조는 **레코드 캡슐화하기**와 **컬렉션 캡슐화하기**로 캡슐화해서 숨길 수 있다.

심지어 기본형 데이터도 기본형을 객체로 바꾸기로 캡슐화할 수 있다.

리팩터링할 때 임시 변수가 자주 걸리적거리는데,

정확한 순서로 계산해야 하고 리팩터링 후에도 그 값을 사용하는 코드에서 접근할 수 있어야 하기 때문이다.

이럴 때는 **임시 변수를 질의 함수로 바꾸기**가 상당히 도움된다.

특히 길이가 너무 긴 함수를 쪼개는 데 유용하다.

클래스는 본래 정보를 숨기는 용도로 설계되었다.

앞 장에서는 여러 함수를 클래스로 묶기로 클래스를 만드는 방법을 소개했다.

이 외에도 흔히 사용하는 **추출/인라인하기**는 리팩터링의 클래스 버전인 **클래스 추출하기**와 **클래스 인라인하기**도 활용할 수 있다.

클래스는 내부 정보뿐 아니라 클래스 사이의 연결 관계를 숨기는 데도 유용하다.

이 용도로는 **위임 숨기기**가 있다. 하지만 너무 많이 숨기려다 보면 인터페이스가 비대해질 수 있으니 반대 기법인 **중개자 제거하기**도 필요하다.

가장 큰 캡슐화 단위는 클래스와 모듈이지만 함수도 구현을 캡슐화한다.

때로는 알고리즘을 통째로 바꿔야 할 때가 있는데, 함수 추출하기로 알고리즘 전체를 함수 하나에 담은 뒤 **알고리즘 교체하기**를 적용하면 된다.

## 레코드 캡슐화하기(Encapsulate Record)

```jsx
organization = { name: "에크미 구스베리", country: "GB" };
```

```jsx
class Organization {
  constructor(data) {
    this.name = data.name;
    this.country = data.country;
  }

  get name() {
    return this.name;
  }
  set name(arg) {
    this.name = arg;
  }
  get country() {
    return this.country;
  }
  set country(arg) {
    this.country = arg;
  }
}
```

### 배경

대부분 프로그래밍 언어는 데이터 레코드를 표현하는 구조를 제공한다.

레코드는 연관된 여러 데이터를 직관적인 방식으로 묶을 수 있어서 각각을 따로 취급할 때보다 훨씬 의미 있는 단위로 전달할 수 있게 해준다.

하지만 단순한 레코드에는 단점이 있다.

특히, 계산해서 얻을 수 있는 값과 그렇지 않은 값을 명확히 구분해 저장해야 하는 점이 까다롭다.

가령 값의 범위(range)를 표현하려면 {start : 1, end : 5}나 {start : 1, length : 5}등의 방식으로 저장할 수 있다.

어떤 식으로 저장하든 ‘시작’과 ‘끝’과 ‘길이’를 알 수 있어야 한다.

바로 이 때문에 나는 가변 데이터를 저장하는 용도로는 레코드보다 객체를 선호하는 편이다.

객체를 사용하면 어떻게 저장했는지를 숨긴 채 세 가지 값을 각각의 메서드로 제공할 수 있다.

사용자는 무엇이 저장된 값이고 무엇이 계산된 값인지 알 필요가 없다.

캡슐화하면 이름을 바꿀 때도 좋다.

필드 이름을 바꿔도 기존 이름과 새 이름 모두를 각각의 메서드로 제공할 수 있어서

사용자 모두가 새로운 메서드로 옮겨갈 때까지 점진적으로 수정할 수 있다.

나는 ‘가변’ 데이터일 때 객체를 선호한다.

값이 불변이면 단순히 ‘시작’과 ‘끝’과 ‘길이’를 모두 구해서 레코드에 저장한다.

이름을 바꿀 때는 그저 필드를 복제한다.

그러면 앞서 객체를 활용해 수정 전후의 두 메서드를 동시에 제공한 방식과 비슷하게 점진적으로 수정할 수 있다.

레코드 구조는 두 가지로 구분할 수 있다. 하나는 필드 이름을 노출하는 형태고,

하나는 필드를 외부로부터 숨겨서 내가 원하는 이름을 쓸 수 있는 형태다.

후자는 주로 라이브러리에서 hash, map, hashmap, dictionary, associative array 등의 이름으로 제공한다.

해시맵은 다양한 작업에 유용하지만, 필드를 명확히 알려주지 않는다는 게 단점이 될 수 있다.

범위를 {시작, 끝} 혹은 {시작, 길이} 중 어떤 방식인지 확인하는 유일한 길은 코드를 직접 확인하는 방법뿐이다.

프로그램에서 해시맵을 쓰는 부분이 적다면 문제되지 않지만 사용하는 곳이 많을수록 불분명함으로 인해 발생하는 문제가 커진다.

이러한 불투명한 레코드를 명시적인 레코드로 리팩터링해도 되지만, 그럴 바에는 레코드 대신 클래스를 사용하는 편이 낫다.

코드를 작성하다 보면 중첩된 리스트나 해시맵을 받아서 JSON이나 XML 같은 포맷으로 직렬화할 때가 많다.

이런 구조 역시 캡슐화할 수 있는데, 그러면 나중에 포맷을 바꾸거나 추적하기 어려운 데이터를 수정하기가 수월해진다.

---

### 절차

1. 레코드를 담은 변수를 캡슐화한다.

   → 레코드를 캡슐화하는 함수의 이름은 검색하기 쉽게 지어준다.

2. 레코드를 감싼 단순한 클래스로 해당 변수의 내용을 교체한다. 이 클래스의 원본 레코드를 반환하는 접근자도 정의하고,

   변수를 캡슐화하는 함수들이 이 접근자를 사용하도록 수정한다.

3. 테스트한다.
4. 원본 레코드 대신 새로 정의한 클래스 타입의 객체를 반환하는 함수들을 새로 만든다.
5. 레코드를 반환하는 예전 함수를 사용하는 코드를 4에서 만든 새 함수를 사용하도록 바꾼다.

   필드에 접근할 때는 객체의 접근자를 사용한다. 적절한 접근자가 없으면 추가한다. 한 부분을 바꿀 때마다 테스트한다.

   → 중첩된 구조처럼 복잡한 레코드라면, 먼저 데이터를 갱신하는 클라이언트들에 주의해서 살펴본다.

   → 클라이언트가 데이터를 읽기만 한다면 데이터의 복제본이나 읽기전용 프락시를 반환할지 고려해보자.

6. 클래스에서 원본 데이터를 반환하는 접근자와 원본 레코드를 반환하는 함수들을 제거한다.
7. 테스트한다.
8. 레코드의 필드도 데이터 구조인 중첩 구조라면 레코드 캡슐화하기와 컬렉션 캡슐화하기를 재귀적으로 적용한다.

---

### 예시

- 간단한 레코드 캡슐화하기
  프로그램 전체에서 널리 사용되는 상수를 예로 살펴보자.

  ```jsx
  organization = { name: "에크미 구스베리", country: "GB" };
  ```

  이 상수는 프로그램 곳곳에서 레코드 구조로 사용하는 객체로서, 다음과 같이 읽고 쓴다.

  ```jsx
  result += `<h1>${organization.name}</h1>`; // 읽기 예
  organization.name = newName; // 쓰기 예
  ```

  [1] 가장 먼저 이 상수를 캡슐화한다.

  ```jsx
  function getRawDataOfOrganization() {
    return organization;
  }
  ```

  그러면 읽고 쓰는 코드는 다음처럼 바뀐다.

  ```jsx
  result += `<h1>${getRawDataOfOrganization().name}</h1>`; // 읽기 예
  getRawDataOfOrganization().name = newName; // 쓰기 예
  ```

  그런데 방금 변수 캡슐화하기를 정식으로 따르지 않고, 게터를 찾기 쉽도록 의도적으로 이상한 이름을 붙였다.
  이 게터는 임시로만 사용할 것이기 때문이다.
  레코드를 캡슐화하는 목적은 변수 자체는 물론 그 내용을 조작하는 방식도 통제하기 위해서다.
  이렇게 하려면 [2]레코드를 클래스로 바꾸고, [4] 새 클래스의 인스턴스를 반환하는 함수를 새로 만든다.

  ```jsx
  const organization = new Organization({
    name: "에크미 구스베리",
    country: "GB",
  });
  function getRawDataOfOrganization() {
    return organization.data;
  }
  function getOrganization() {
    return organization;
  }

  class Organization {
    constructor(data) {
      this.data = data;
    }
  }
  ```

  객체로 만드는 작업이 끝났으니 [5] 레코드를 사용하던 코드를 살펴보자. 레코드를 갱신하던 코드는 모두 세터를 사용하도록 고친다.

  ```jsx
  getOrganization().name = newName;

  class Organization {
  	...
  	set name(aString) { this.data.name = aString; }
  }
  ```

  마찬가지로, 레코드를 읽는 코드는 모두 게터를 사용하게 바꾼다.

  ```jsx
  result += `<h1>${getOrganization().name}</h1>`; // 읽기 예

  class Organization {
  	...
  	get name() { return this.data.name; }
  }
  ```

  [6] 다 바꿨다면 앞에서 이상한 이름으로 지었던 임시 함수를 제거한다.

  ```jsx
  // function getRawDataOfOrganization() {return organization.data;} <- 제거 !
  function getOrganization() {
    return organization;
  }
  ```

  > 최종

  ```jsx
  class Organization {
    constructor(data) {
      this.name = data.name;
      this.country = data.country;
    }

    get name() {
      return this.name;
    }
    set name(arg) {
      this.name = arg;
    }
    get country() {
      return this.country;
    }
    set country(arg) {
      this.country = arg;
    }
  }
  ```

  이렇게 하면 입력 데이터 레코드와의 연결을 끊어준다는 이점이 생긴다.
  특히 이 레코드를 참조하여 캡슐화를 깰 우려가 있는 코드가 많을 때 좋다.
  데이터를 개별 필드로 펼지지 않았다면 data를 대입할 때 복제하는 식으로 처리했을 것이다.

- 중첩된 레코드 캡슐화하기
  JSON 문서처럼 중첩된 레코드는 리팩터링의 기반은 비슷하지만, 읽는 코드를 다룰 때는 선택지가 몇가지 더 생긴다.
  아래의 예시 데이터는 고객 정보를 저장한 해시맵으로, 고객 ID를 키로 사용한다.

  ```json
  "1920" : {
    name : "마틴 파울러",
    id : "1920",
    usages : {
      "2016" : {
        "1" : 50,
        "2" : 55,
        // 나머지 달(month)은 생략
      },
      "2015" : {
        "1" : 70,
        "2" : 63,
        // 나머지 달은 생략
      }
    }
  },
  "38673" : {
    name : "닐 포드",
    id : "38673",
    // 다른 고객 정보도 같은 형식으로 저장된다.
  }
  ```

  중첩 정도가 심할수록 읽거나 쓸 때 데이터 구조 안으로 더 깊숙히 들어가야 한다.

  ```jsx
  // 쓰기 예
  customerData[customerID].usage[year][month] = amount;
  // 읽기 예
  function compareUsage(customerID, laterYear, month) {
    const later = customerData[customerID].usages[laterYear][month];
    const earlier = customerData[customerID].usages[laterYeaer - 1][month];
    return { laterAmount: later, change: later - earlier };
  }
  ```

  이번 캡슐화도 앞에서와 마찬가지로 변수 캡슐화부터 시작한다.

  ```jsx
  function getRawDataOfCustomers() {
    return customerData;
  }
  function setRawDataOfCustomers(arg) {
    customerData = arg;
  }
  ```

  ```jsx
  // 쓰기 예
  getRawDataOfCustomers()[customerID].usage[year][month] = amount;
  // 읽기 예
  function compareUsage(customerID, laterYear, month) {
    const later = getRawDataOfCustomers()[customerID].usages[laterYear][month];
    const earlier =
      getRawDataOfCustomers()[customerID].usages[laterYeaer - 1][month];
    return { laterAmount: later, change: later - earlier };
  }
  ```

  그런 다음 전체 데이터 구조를 표현하는 클래스를 정의하고, 이를 반환하는 함수를 새로 만든다.

  ```jsx
  function getCustomerData() {
    return customerData;
  }
  function getRawDataOfCustomers() {
    return customerData;
  }
  function setRawDataOfCustomers(arg) {
    customerData = arg;
  }

  class CustomerData {
    constructor(data) {
      this.data = data;
    }
  }
  ```

  여기서 가장 중요한 부분은 데이터를 쓰는 코드다. 따라서 getRawDataOfCustomers()를 호출한 후 데이터를 변경하는 경우도 주의해야 한다.
  기본 절차에 따르면 고객 객체를 반환하고 필요한 접근자를 만들어서 사용하게 하면 된다.
  현재 고객 객체에는 이 값을 쓰는 세터가 없어서 데이터 구조 안으로 깊이 들어가서 값을 바꾸고 있다.
  따라서 데이터 구조 안으로 들어가는 코드를 세터로 뽑아내는 작업부터 한다.

  ```jsx
  function setUsage(customerID, year, month, amount) {
    getRawDataOfCustomers()[customerID].usage[year][month] = amount;
  }

  setUsage(customerID, year, month, amount);
  ```

  그런 다음 이함수를 고객 데이터 클래스로 옮긴다.

  ```jsx
  getCustomerData().setUsage(customerID, year, month, amount);

  class CustomerData {
  	...
  	setUsage(customerID, year, month, amount) {
  		this.data[customerID].usage[year][month] = amount;
  	}
  }
  ```

  나는 덩치 큰 데이터 구조를 다룰수록 쓰기 부분에 집중한다.
  캡슐화에서는 값을 수정하는 부분을 명확하게 드러내고 한 곳에 모아두는 일이 굉장히 중요하다.
  이렇게 쓰는 부분을 수정하다 보면 빠진 건 없는지 궁금해질 것이다.
  확인하는 방법은 여러가지다.
  우선 getRawDataOfCustomers()에서 데이터를 깊은 복사(deep copy)하여 반환하는 방법이다.
  테스트 커버리지가 충분하다면 깜빡 잊고 수정하지 않은 부분을 테스트가 걸러내줄 것이다.

  ```jsx
  function getCustomerData() {return customerData;}
  function getRawDataOfCustomers() { return customerData.rawData; }
  function setRawDataOfCustomers(arg) { customerData = new CustomerData(arg); }

  class CustomerData {
  	...
  	get rawData() {
  		return cloneDeep(this.data);
  	}
  }
  ```

  깊은 복사는 lodash 라이브러리가 제공하는 cloneDeep()으로 처리했다.
  데이터 구조의 읽기 전용 프락시를 반환하는 방법도 있다.
  클라이언트에서 내부 객체를 수정하려 하면 프락시가 예외를 던지도록 하는 것이다.
  이 기능은 손쉽게 구현할 수 있는 언어도 있지만, 자바스크립트는 아니다.
  또한 복제본을 만들고 이를 재귀적으로 동결(freeze)해서 쓰기 동작을 감지하는 방법도 있다.
  쓰기는 주의해서 다뤄야 한다. 그렇다면 읽기에 대한 몇 가지 방법을 소개하겠다.

  1. 세터 때와 같은 방법을 적용할 수 있다. 즉, 읽는 코드를 모두 독립 함수로 추출한 뒤 클래스로 옮기는 것이다.

     ```jsx
     function compareUsage (customerID, laterYear, month) {
     	const later = getCustomerData().usage(customerID, laterYear, month);
     	const earlier = getCustomerData().usage(customerID, laterYear - 1, month);
     	return { laterAmount : later, change : later - earlier };
     }

     class CustomerData {
     	...
     	usage(customerID, year, month) {
     		return this.data[customerID].usage[year][month];
     	}
     }
     ```

     이 방법의 가장 큰 장점은 customerData의 모든 쓰임을 명시적인 API로 제공한다는 것이다.

     이 클래스만 보면 데이터 사용 방법을 모두 파악할 수 있다. 하지만 읽는 패턴이 다양하면 그 만큼 코드도 늘어난다.

     요즘 언어들에서는 list-and-hash 데이터 구조를 쉽게 다룰 수 있는데, 이런 언어를 사용한다면 이 형태로 넘겨주는 것도 좋다.

     다른 방법으로, 클라이언트가 데이터 구조를 요청할 때 실제 데이터를 제공한다.

     하지만 클라이언트가 데이터를 직접 수정하지 못하게 막을 방법이 없어서

     ‘모든 쓰기를 함수 안에서 처리한다’는 캡슐화의 핵심 원칙이 깨지는 게 문제다.

     따라서 가장 간단한 방법은 앞에서 작성한 rawData() 메서드를 사용해서 내부 데이터를 복제해서 제공하는 것이다.

     ```jsx
     function compareUsage (customerID, laterYear, month) {
     	const later = getCustomerData().rawData[customerID].usages[laterYear][month];
     	const earlier = getCustomerData().rawData[customerID].usages[laterYeaer - 1][month];
     	return { laterAmount : later, change : later - earlier };
     }

     class CustomerData {
     	...
     	get rawData() {
     		return cloneDeep(this.data);
     	}
     }
     ```

     이 방법의 문제는 데이터 구조가 클수록 복제 비용이 커져서 성능이 느려질 수 있다는 것이다.

     하지만 성능 비용을 감당할 수 있는 상황일 수도 있다.

     나라면 막연히 걱정만 하지 않고 실제로 측정 해본다.

     또 다른 문제는 클라이언트가 원본을 수정한다고 착각할 수 있다는 것이다.

     이럴 때는 읽기 전용 프락시를 제공하거나 복제본을 동결시켜서 데이터를 수정하려 할 때 에러를 던진다.

  2. 레코드 캡슐화를 재귀적으로 한다.

     할 일은 늘어나지만 가장 확실하게 제어할 수 있다.

     이 방법을 적용하려면 먼저 고객 정보 레코드를 클래스로 바꾸고,

     **컬렉션 캡슐화하기**로 레코드를 다루는 코드를 리팩터링해서 고객 정보를 다루는 클래스를 생성한다.

     그런 다음 접근자를 이용해서 갱신을 함부로 하지 못하게 만든다.

     가령 **참조를 값으로 바꾸기**로 고객 정보를 다루는 객체를 리팩터링할 수 있다.

     하지만 데이터 구조가 거대하면 일이 상당히 커진다.

     게다가 그 데이터 구조를 사용할 일이 그리 많지 않다면 효과도 별로 없다.

     때로는 새로 만든 클래스와 게터를 잘 혼합해서, 게터는 데이터 구조를 깊이 탐색하게 만들되

     원본 데이터를 그대로 반환하지 말고 객체로 감싸서 반환하는 게 효과적일 수 있다.

     이 방법은 [Refactoring Code to Load a Decoument] 에서 자세히 다룬다.

---

## 컬렉션 캡슐화하기 (Encapsulate Collection)

```jsx
class Person {
  get courses() {
    return this.courses;
  }
  set courses(aList) {
    this.courses = aList;
  }
}
```

```jsx
class Person {
	get courses() { return this.courses; }
	addCourse(aCourse) { ... }
	removeCourse(aCourse) { ... }
}
```

### 배경

나는 가변 데이터를 모두 캡슐화하는 편이다. 그러면 데이터 구조가 언제 어떻게 수정되는지 파악하기 쉬워져

필요한 시점에 데이터 구조를 변경하기도 쉬워지기 떄문이다.

특히 객체 지향 개발자들은 캡슐화를 적극 권장하는데 컬렉션을 다룰 때는 곧잘 실수를 저지르곤 한다.

예컨대 컬렉션 변수로의 접근을 캡슐화하면서 게터가 컬렉션 자체를 반환하도록 한다면,

그 컬렉션을 감싼 클래스가 눈치채지 못하는 상태에서 컬렉션의 원소들이 바뀌어버릴 수 있다.

나는 이런 문제를 방지하기 위해 컬렉션을 감싼 클래스에 흔히 add()와 remove()라는 이름의 컬렉션 변경자 메서드를 만든다.

이렇게 해서 항상 컬렉션을 소유한 클래스를 통해서만 원소를 변경하도록 하면

프로그램을 개선하면서 컬렉션 변경 방식도 원하는 대로 수정할 수 있다.

모든 팀원이 원본 모듈 밖에서는 컬렉션을 수정하지 않는 습관을 갖고 있다면 이런 메서드를 제공하는 것만으로도 충분하다.

하지만 실수 한번이 굉장히 찾기 어려운 버그로 이어질 수 있으니 바람직하지 않다.

이보다는 컬렉션 게터가 원본 컬렉션을 반환하지 않게 만들어서 클라이언트가 실수로 컬렉션을 바꿀 가능성을 차단하라.

내부 컬렉션을 직접 수정하지 못하게 막는 방법 중 하나로, 절대로 컬렉션 값을 반환하지 않게 할 수 있다.

컬렉션에 접근하려면 컬렉션이 소속된 클래스의 적절한 메서드를 반드시 거치게 하는 것이다.

예컨대 aCustomer.orders.size() 처럼 접근하는 코드를 aCustomer.numberOfOrders()로 바꾸는 것이다.

나는 이 방식에 동의하지 않는다. 최신 언어는 다양한 컬렉션 클래스들을 표준화된 인터페이스로 제공하며,

컬렉션 파이프라인과 같은 패턴을 적용해서 다채롭게 조합할 수 있다.

표준 인터페이스 대신 전용 메서드들을 사용하게 하면 부가적인 코드가 상당히 늘어나며 컬렉션 연산을 조합해 쓰기도 어렵다.

또 다른 방법은 컬렉션을 읽기전용으로 제공할 수 있다. 자바에서는 컬렉션의 읽기전용 프락시를 반환하게 만들기가 쉽다.

프락시가 내부 컬렉션을 읽는 연산은 그대로 전달하고, 쓰기는 예외를 던져 모두 막는 것이다.

Iterator나 열거형 객체를 기반으로 컬렉션을 조합하는 라이브러리들도 비슷한 방식을 쓴다.

가령 iterator에서 내부 컬렉션을 수정할 수 없게 한다.

가장 흔히 사용하는 방식은 아마도 컬렉션 게터를 제공하되 내부 컬렉션의 복제본을 반환하는 것이다.

복제본을 수정해도 캡슐화된 원본 컬렉션에는 아무런 영향을 주지 않는다.

컬렉션이 상당히 크다면 성능 문제가 발생할 수 있다. 하지만 그런 경우는 드무니 일반 규칙을 따르도록 하자.

한편, 프락시 방식에서는 원본 데이터를 수정하는 과정이 겉으로 드러나지만 복제 방식에서는 그렇지 않다는 차이도 있다.

하지만 이 사실이 문제가 될 일은 거의 없다. 이런 식으로 접근하는 컬렉션은 대체로 짧은 시간 동안만 사용되기 때문이다.

여기서 중요한 점은 코드베이스에서 일관성을 주는 것이다. 앞에서 나온 방식 중에서 한 가지만 적용하여 방식을 통일하라.

---

### 절차

1. 아직 컬렉션을 캡슐화하지 않았다면 변수 캡슐화하기부터 한다.
2. 컬렉션에 원소를 추가/제거하는 함수를 추가한다.

   → 컬렉션 자체를 통째로 바꾸는 세터는 제거한다. 세터를 제거할 수 없다면 인수로 받은 컬렉션을 복제해 저장하도록 만든다.

3. 정적 검사를 수행한다.
4. 컬렉션을 참조하는 부분을 모두 찾는다. 컬렉션의 변경자를 호출하는 코드가 모두 앞에서 추가한 함수를 호출하도록 수정한다.
5. 컬렉션 게터를 수정해서 원본 내용을 수정할 수 없는 읽기전용 프락시나 복제본을 반환하게 한다.
6. 테스트한다.

---

### 예시

```jsx
class Person {
  constructor(name) {
    this.name = name;
    this.courses = [];
  }
  get name() {
    return this.name;
  }
  get courses() {
    return this.courses;
  }
  set courses(aList) {
    this.courses = aList;
  }
}

class Course {
  constructor(name, isAdvanced) {
    this.name = name;
    this.isAdvanced = isAdvanced;
  }
  get name() {
    return this.name;
  }
  get isAdvanced() {
    return this.isAdvanced;
  }
}
```

클라이언트는 Person이 제공하는 수업 컬렉션에서 수업 정보를 얻는다.

```jsx
numAdvancedCourses = aPerson.courses.filter((c) => c.isAdvanced).length;
```

모든 필드가 접근자 메서드로 보호받고 있으니 안일한 개발자는 이렇게만 해도 데이터를 제대로 캡슐화했다고 생각하기 쉽다.

하지만 세터를 이용해 수업 컬렉션을 통째로 설정한 클라이언트는 누구든 마음대로 수정할 수 있다.

```jsx
const basicCourseNames = readBasicCourseNames(filename);
aPerson.courses = basicCourseNames.map((name) => new Course(name, false));
```

클라이언트는 아래와 같이 직접 수정하는 것이 훨씬 편할 수 있다.

```jsx
for (const name of readBasicCourseNames(filename)) {
  aPerson.courses.push(new COurse(name, false));
}
```

하지만 이런 경우 Person 클래스가 더는 컬렉션을 제어할 수 없으니 캡슐화가 깨진다.

필드를 참조하는 과정만 캡슐화했을 뿐 필드에 담긴 내용은 캡슐화하지 않은 게 원인이다.

[2] 제대로 캡슐화하기 위해 먼저 클라이언트가 수업을 하나씩 추가하고 제거하는 메서드를 Person에 추가해보자.

```jsx
class Person{
	...
	addCourse(aCourse) {
		this.courses.push(aCourse);
	}
	removeCourse(aCourse, fnIfAdsent = () => {throw new RangeError();}) {
		const index = this.course.indexOf(aCourse);
		if ( index === -1) fnIfAdsent();
		else this.courses.splice(index, 1);
	}
}
```

제거 메서드에서는 클라리언트가 컬렉션에 없는 원소를 제거하려 할 때의 대응 방식을 정해야 한다.

그냥 무시하거나 에러를 던질 수 있다. 여기서는 기본적으로 에러를 던지되, 호출자가 원하는 방식으로 처리할 여지도 남겨뒀다.

[4] 그런 다음 컬렉션의 변경자를 직접 호출하던 코드를 모두 찾아서 방금 추가한 메서드를 사용하게 변경한다.

```jsx
for (const name of readBasicCourseNames(filename)) {
  aPerson.addCourse(new COurse(name, false));
}
```

[2] 이렇게 개별 원소를 추가하고 제거하는 메서드를 제공하기 때문에 setCourses()를 사용할 일이 없어졌으니 제거한다.

세터를 제공해야 할 특별한 이유가 있다면 인수로 받은 컬렉션의 복제본을 필드에 저장하게 한다.

```jsx
class Person {
  set courses(aList) {
    this.courses = aList.slice();
  }
}
```

이렇게 하면 클라이언트는 용도에 맞는 변경 메서드를 사용하도록 할 수 있다.

[5] 하지만 나는 이 메서드들을 사용하지 않고서는 아무도 목록을 변경할 수 없게 만드는 방식을 선호한다.

다음과 같이 복제본을 제공하면 된다.

```jsx
class Person {
	...
	get courses() { return this.courses.slice(); }
}
```

내 경험에 따르면 컬렉션에 대해서는 어느 정도 강박증을 갖고 불필요한 복제본을 만드는 편이,

예상치 못한 수정이 촉발한 오류를 디버깅하는 것보다 낫다.

때론 명확히 드러나지 않는 수정도 일어날 수 있다.

가령 다른 언어들은 컬렉션을 수정하는 연산들이 기본적으로 복제본을 만들어 처리하지만,

자바스크립트에서는 배열을 정렬할 때 원본을 수정한다.

컬렉션 관리를 책임지는 클래스라면 항상 복제본을 제공해야 한다.

그리고 나는 컬렉션을 변경할 가능성이 있는 작업을 할 때도 습관적으로 복제본을 만든다.

---

## 기본형을 객체로 바꾸기(Replace Primitive with Object)

```jsx
orders.filter((o) => "high" === o.priority || "rush" === o.priority);
```

```jsx
orders.filter((o) => o.priority.higherThen(new Priority("normal")));
```

### 배경

개발 초기에는 단순한 정보나 숫자나 문자열 같은 간단한 데이터 항목으로 표현할 때가 많다.

간단했던 이 정보들은 개발이 진행되면서 더 이상 간단하지 않게 변한다.

예컨대 처음에는 전화번호를 문자열로 표현했는데 나중에 포매팅이나 지역 코드 추출 같은 특별한 동작이 필요해질 수 있다.

이런 로직들로 금세 중복 코드가 늘어나서 사용할 때마다 드는 노력도 늘어나게 된다.

나는 단순한 출력 이상의 기능이 필요해지는 순간 그 데이터를 표현하는 전용 클래스를 정의하는 편이다.

시작은 기본형 데이터를 단순히 감싼 것과 큰 차이가 없을 것이라 효과는 미미하다.

하지만 나중에 특별한 동작이 필요해지면 이 클래스에 추가하면 되니 프로그램이 커질수록 점점 유용한 도구가 된다.

그리 대단해 보이지 않을지 모르지만 코드베이스에 미치는 효과는 놀라울 만큼 크다.

초보 프로그래머에게는 직관에 어긋나 보일 수 있다. 하지만 경험 많은 개발자들은 여러 가지 리팩터링 중에서도 가장 유용한 것으로 손꼽는다.

### 절차

1. 아직 변수를 캡슐화하지 않았다면 캡슐화한다.
2. 단순한 값 클래스(value class)를 만든다. 생성자는 기존 값을 인수로 받아 저장하고, 게터를 추가한다.
3. 정적 검사를 수행한다.
4. 값 클래스의 인스턴스를 새로 만들어서 필드에 저장하도록 세터를 수정한다. 이미 있다면 필드의 타입을 적절히 변경한다.
5. 새로 만든 클래스의 게터를 호출환 결과를 반환하도록 게터를 수정한다.
6. 테스트한다.
7. 함수 이름을 바꾸면 원본 접근자의 동작을 더 잘 드러낼 수 있는지 검토한다.

   → 참조를 값으로 바꾸거나(9.4) 값을 참조로 바꾸면(9.5) 새로 만든 객체의 역할이 더 잘 드러나는지 검토한다.

### 예시

레코드 구조에서 데이터를 읽어 들이는 단순한 주문(order) 클래스를 살펴보자.

이 클래스의 우선 순위(priority) 속성은 값을 간단히 문자열로 표현한다.

```jsx
class Order {
	constructor(data) {
		this.priority = data.priority;
		// 나머지 초기화 코드 생략
	}
	...
}
```

클라이언트에서는 이 코드를 다음처럼 사용한다.

```jsx
highPriorityCount = orders.filter(
  (o) => "high" === o.priority || "rush" === o.priority
).length;
```

[1] 나는 데이터 값을 다루기 전에 항상 변수부터 캡슐화한다.

```jsx
class Order {
	...
	get priority() { return this.priority; }
	set priority(aString) { this.priority = aString; }
}
```

이제 우선순위 속성을 초기화하는 생성자에서 방금 정의한 세터를 사용할 수 있다.

이렇게 필드를 자가 캡슐화(self-encapsulation)하면 필드 이름을 바꿔도 클라이언트 코드는 유지할 수 있다.

[2] 다음으로 우선순위 속성을 표현하는 값 클래스 Priority를 만든다.

이 클래스는 표현할 값을 받는 생성자와 그 값을 문자열로 반환하는 변환 함수로 구성한다.

```jsx
class Priority {
  constructor(value) {
    this.value = value;
  }
  toString() {
    return this.value;
  }
}
```

이 상황에서는 개인적으로 게터(value())보다는 변환 함수(toString())를 선호한다.

클라이언트 입장에서 보면 속성 자체를 받은 게 아니라 해당 속성을 문자열로 표현한 값을 요청한게 되기 때문이다.

[4][5] 그런 다음 방금 만든 Priority 클래스를 사용하도록 접근자들을 수정한다.

```jsx
class Order {
	...
	get priority() { return this.priority.toString(); }
	set priority(aString) { this.priority = new Priority(aString); }
}
```

[7] 이렇게 Priority 클래스를 만들고 나면 Order 클래스의 게터가 이상해진다.

이 게터가 반환하는 값은 우선순위 자체가 아니라 우선순위를 표현하는 문자열이다.

그러니 즉시 함수 이름을 바꿔준다.

```jsx
class Order {
	...
	get priorityString() { return this.priority.toString(); }
	set priority(aString) { this.priority = new Priority(aString); }
}
```

```jsx
highPriorityCount = orders.filter(
  (o) => "high" === o.priorityString || "rush" === o.priorityString
).length;
```

지금처럼 매개변수 이름만으로 세터가 받는 데이터의 유형을 쉽게 알 수 있다면 세터의 이름은 그대로 둬도 좋다.

- 더 가다듬기

      공식적인 리팩터링은 여기까지다. 그런데 Priority 클래스를 사용하는 코드를 살펴보면서

      이 클래스를 직접 사용하는 것이 과연 좋은지 고민해봤다.

      그 결과 Priority 객체를 제공하는 게터를 Order 클래스에 만드는 편이 낫겠다고 판단했다.

      ```jsx
      class Order {
      	...
      	get priority() { return this.priority; }
      	get priorityString() { return this.priority.toString(); }
      	set priority(aString) { this.priority = new Priority(aString); }
      }
      ```

      ```jsx
      highPriorityCount = orders.filter(o => "high" === o.priority.toString()
      																		|| "rush" === o.priority.toString()).length;
      ```

      Priority 클래스는 다른 곳에서도 유용할 수 있으니 Order의 세터가 Priority 인스턴스를 받도록 해주면 좋다.

      이를 위해 Priority의 생성자를 다음과 같이 변경한다.

      ```jsx
      class Priority {
      	...
      	constructor(value) {
      		if(value instanceof Priority) return value;
      		this.value = value;
      	}
      }
      ```

      이렇게 하는 목적은 어디까지나 Priority 클래스를 새로운 동작을 담는 장소로 활용하기 위해서다.

      새로운 동작이란 새로 구현한 것일 수도, 다른 곳에서 옮겨온 것일 수도 있다.

      우선순위 값을 검증하고 비교하는 로직을 추가한 예를 준비했다.

      ```jsx
      class Priority {
        constructor(value){
          if(value instanceof Priority) return value;
          if(Priority.legalValues().includes(value))
            this.value = value;
          else
            throw new Error(`<${value}>는 유효하지 않은 우선순위입니다.`);
        }
        toString() { return this.value; }
        get index() { return Priority.legalValues().findIndex(s => s === this.value); }
        static legalValues() { return ['low', 'normal', 'high', 'rush'] }
        equals(other) { return this.index === other.index; }
        higherThan(other) { return this.index > other.index; }
        lowerThan(other) { return this.index < other.index; }
      }
      ```

      이렇게 수정하면서 우선순위 값 객체를 만들어야겠다고 판단했다. 따라서 equals() 메서드를 추가하고 불변이 되도록 만들었다.

      이처럼 동작을 추가하면 클라이언트 코드를 더 의미 있게 작성할 수 있다.

      ```jsx
      highPriorityCount = orders.filter((o) => o.priority.higherThan(new Priority("normal"))).length;
      ```

      ## 임시 변수를 질의 함수로 바꾸기(Replace Temp with Query)

      ```jsx

  const basePrice = this.quantity _ this.itemPrice;
  if (basePrice > 1000)
  return basePrice _ 0.95;
  else
  return basePrice \* 0.98;

````

```jsx
get basePrice() { this.quantity * this.itemPrice; }
...

if (basePrice > 1000)
	return basePrice * 0.95;
else
	return basePrice * 0.98;
````

### 배경

함수 안에서 어떤 코드의 결과값을 뒤에서 다시 참조할 목적으로 임시 변수를 쓰기도 한다.

임시 변수를 사용하면 값이 계산하는 코드가 반복되는 걸 줄이고 값의 의미를 설명할 수 있어서 유용하다.

그런데 한 걸음 더 나아가 아예 함수로 만들어 사용하는 편이 나을 때가 많다.

긴 함수의 한 부분을 별도 함수로 추출하고자 할 때 먼저 변수들을 각각의 함수로 만들면 일이 수월해진다.

추출한 함수에 변수를 따로 전달할 필요가 없어지기 때문이다. 또한 이 덕분에 추출한 함수와 원래 함수의 경계가 더 분명해지기도 하는데,

그러면 부자연스러운 의존 관계나 부수효과를 찾고 제거하는 데 도움이 된다.

변수를 대신 함수로 만들어두면 비슷한 계산을 수행하는 다른 함수에서도 사용할 수 있어 코드 중복이 줄어든다.

그래서 나는 여러 곳에서 똑같은 방식으로 계산되는 변수를 발견할 때마다 함수로 바꿀 수 있는지 살펴본다.

이번 리팩터링은 클래스 안에서 적용할 때 효과가 가장 크다.

클래스는 추출할 메서드들에 공유 컨텍스트를 제공하기 때문이다.

클래스 바깥의 최상위 함수로 추출하면 매개변수가 너무 많아져서 함수를 사용하는 장점이 줄어든다.

중첩 함수를 사용하면 이런 문제는 없지만 관련 함수들과 로직을 널리 공유하는 데 한계가 있다.

임시 변수를 질의 함수로 바꾼다고 다 좋아지는 건 아니다.

자고로 변수는 값을 한 번만 계산하고, 그 뒤로는 읽기만 해야 한다.

가장 단순한 예로, 변수에 값을 한번 번 대입한 뒤 더 복잡한 코드 덩어리에서 여러 차례 다시 대입하는 경우는 모두 질의 함수로 추출해야 한다.

또한 이 계산 로직은 변수가 다음번에 사용될 때 수행해도 똑같은 결과를 내야 한다. 그래서 ‘옛날 주소’처럼 스냅숏 용도로 쓰이는 변수에는

이 리팩터링을 적용하면 안 된다.

### 절차

1. 변수가 사용되기 전에 값이 확실히 결정되는지, 변수를 사용할 때마다 계산 로직이 매번 다른 결과를 내지는 않는지 확인한다.
2. 읽기 전용으로 만들 수 있는 변수는 읽기 전용으로 만든다.
3. 테스트한다.
4. 변수 대입문을 함수로 추출한다.

   → 변수와 함수가 같은 이름을 가질 수 없다면 함수 이름을 임시로 짓는다. 또한, 추출한 함수가 부수 효과를 일으키지는 않는지 확인한다.

   부수효과가 있다면 질의 함수와 변경 함수 분리하기(11.1)로 대처한다.

5. 테스트한다.
6. 변수 인라인하기로 임시 변수를 제거한다.

### 예시

```jsx
class Order {
  constructor(quantity, item) {
    this.quantity = quantity;
    this.item = item;
  }

  get price() {
    let basePrice = this.quantity * this.item.price;
    let discountFactor = 0.98;

    if (basePrice > 1000) discountFactor -= 0.03;
    return basePrice * discountFactor;
  }
}
```

여기서 임시 변수인 basePrice와 discountFactor를 메서드로 바꿔보자

[2] 먼저 basePrice에 const를 붙여 읽기 전용으로 만들고 [3] 테스트해본다. 이렇게 하면 못 보고 지나친 재대입 코드를 찾을 수 있다.

지금처럼 코드가 간단할 때는 그럴 일이 없겠지만, 코드가 길면 흔히 벌어지는 일이다.

```jsx
class Order {
	...
	get price() {
		const basePrice = this.quantity * this.item.price;
		...
	}
}
```

[4] 그런 다음 대입문의 우변을 게터로 추출한다.

```jsx
class Order {
	...
	get price() {
		const basePrice = this.basePrice;
		...
		if(this.basePrice > 1000) discountFactor -= 0.03;
		return this.basePrice * discountFactor;
	}
}
```

[5] 테스트를 한 다음 [6] 변수를 인라인한다.

```jsx
class Order {
	...
	get price() {
		...
		if(this.basePrice > 1000) discountFactor -= 0.03;
		return this.basePrice * discountFactor;
	}
}
```

discountFactor 변수도 같은 순서로 처리한다. [4] 먼저 함수 추출하기다.

```jsx
class Order {
  get price() {
    const discountFactor = this.discountFactor;
    return this.basePrice * discountFactor;
  }
  get discountFactor() {
    let discountFactor = 0.98;
    if (this.basePrice > 1000) discountFactor -= 0.03;
    return discountFactor;
  }
}
```

이번에는 discountFactor에 값을 대입하는 문장이 둘인데, 모두 추출한 함수에 넣어야 한다.

원본 변수는 마찬가지로 const로 만든 후, 변수를 인라인한다.

```jsx
class Order {
	...
	get price() {
		return this.basePrice * this.discountFactor;
	}
}
```

## 클래스 추출하기 (Extract Class)

```jsx
class Person {
  get officeAreaCode() {
    return this.officeAreaCode;
  }
  get officeNumber() {
    return this.officeNubmer;
  }
}
```

```jsx
class Person {
  get officeAreaCode() {
    return this.telephoneNumber.areaCode;
  }
  get officeNumber() {
    return this.telephoneNumber.nubmer;
  }
}
class TelephoneNumber {
  get areaCode() {
    return this.areaCode;
  }
  get number() {
    return this.number;
  }
}
```

### 배경

클래스는 반드시 명확하게 추상화하고 소수의 주어진 역할만 처리해야 한다는 가이드라인이 있다.

하지만 실무에서는 몇 가지 연산을 추가하고 데이터도 보강하면서 클래스가 점점 커진다.

기존 클래스를 굳이 쪼갤 필요까지는 없다고 생각하여 새로운 역할을 덧씌우기 쉬운데,

역할이 갈수록 많아지고 새끼를 치면서 클래스가 굉장히 복잡해진다.

그러다보면 어느새 전자레인지로 바짝 익힌 음식처럼 딱딱해지고 만다.

메서드와 데이터가 너무 많은 클래스는 이해하기가 쉽지 않으니 잘 살펴보고 분리하는 것이 좋다.

특히 일부 데이터와 메서드를 따로 묶을 수 있다면 어서 분리하라는 신호다.

함께 변경되는 일이 많거나 서로 의존하는 데이터들도 분리한다.

특정 데이터나 메서드 일부를 제거하면 어떤 일이 일어나는지 자문해보면 판단에 도움이 된다.

제거해도 다른 필드나 메서드들이 논리적으로 문제가 없다면 분리할 수 있다는 뜻이다.

개발 후반으로 접어들면 서브클래스가 만들어지는 방식에서 왕왕 징후가 나타나기도 한다.

예컨대 작은 일부의 기능만을 위해 서브클래스를 만들거나, 확장해야 할 기능이 무엇이냐에 따라

서브클래스를 만드는 방식도 달라진다면 클래스를 나눠야 한다는 신호다.

### 절차

1. 클래스의 역할을 분리할 방법을 정한다.
2. 분리될 역할을 담당할 클래스를 새로 만든다.

   → 원래 클래스에 남은 역할과 클래스 이름이 어울리지 않는다면 적절히 바꾼다.

3. 원래 클래스의 생성자에서 새로운 클래스의 인스턴스를 생성하여 필드에 저장해둔다.
4. 분리될 역할에 필요한 필드들을 새 클래스로 옮긴다. 하나씩 옮길때마다 테스트한다.
5. 메서드들도 새 클래스로 옮긴다. 이때 저수준 메서드, 즉 다른 메서드를 호출하기보다는 호출을 당하는 일이 많은 메서드부터 옮긴다.
6. 양쪽 클래스의 인터페이스를 살펴보면서 불필요한 메서드를 제거하고, 이름도 새로운 환경에 맞게 바꾼다.
7. 새 클래스를 외부로 노출할지 정한다. 노출하려거든 새 클래스에 참조를 값으로 바꾸기(9.4)를 적용할 지 고민해본다.

### 예시

```jsx
class Person {
  get name() {
    return this.name;
  }
  set name(arg) {
    return (this.name = arg);
  }
  get telephoneNumber() {
    return this.telephoneNumber;
  }
  get officeAreaCode() {
    return this.officeAreaCode;
  }
  set officeAreaCode(arg) {
    this.officeAreaCode = arg;
  }
  get officeNumber() {
    return this.officeNumber;
  }
  set officeNumber(arg) {
    return this.officeNumber;
  }
}
```

[1] 여기서 전화번호 관련 동작을 별도 클래스로 뽑아보자.

[2] 먼저 민 번화번호를 표현하는 TelephoneNumber 클래스를 정의한다.

```jsx
class TelephoneNumber {}
```

[3] 다음으로 Person 클래스의 인스턴스를 생성할 때 전화번호 인스턴스도 함께 생성해 저장해둔다.

```jsx
class Person {
	constructor() {
    this.telephoneNumber = new TelephoneNumber();
	}
	...
}
```

그런 다음 필드들을 하나씩 새 클래스로 옮기며 테스트한다.

```jsx
class Person {
  constructor() {
    this.telephoneNumber = new TelephoneNumber();
  }
  get name() {
    return this.name;
  }
  set name(arg) {
    return (this.name = arg);
  }
  get telephoneNumber() {
    return this.telephoneNumber.telephoneNumber;
  }
  get officeAreaCode() {
    return this.telephoneNumber.officeAreaCode;
  }
  set officeAreaCode(arg) {
    this.telephoneNumber.officeAreaCode = arg;
  }
  get officeNumber() {
    return this.telephoneNumber.officeNumber;
  }
  set officeNumber(arg) {
    return this.telephoneNumber.officeNumber;
  }
}

class TelephoneNumber {
  get telephoneNumber() {
    return `(${this.officeAreaCode}) ${this.officeNumber}`;
  }
  get officeAreaCode() {
    return this.officeAreaCode;
  }
  set officeAreaCode(arg) {
    this.officeAreaCode = arg;
  }
  get officeNumber() {
    return this.officeNumber;
  }
  set officeNumber(arg) {
    return this.officeNumber;
  }
}
```

[6] 이제 정리할 차례다. 새로 만든 클래스는 순수한 전화번호를 뜻하므로 office란 단어는 지운다.

마찬가지로 전화번호란 뜻도 강조할 필요가 없다.

그러니 메서드들의 이름을 적절히 바꿔주자.

```jsx
class Person {
	...
  get officeAreaCode() { return this.telephoneNumber.areaCode; }
  set officeAreaCode(arg) { this.telephoneNumber.areaCode = arg; }
  get officeNumber() { return this.telephoneNumber.number; }
  set officeNumber(arg) { return this.telephoneNumber.number; }
}

class TelephoneNumber{
	...
  get areaCode() { return this.areaCode; }
  set areaCode(arg) { this.areaCode = arg; }
  get number() { return this.number; }
  set number(arg) { return this.number; }
}
```

마지막으로 전화번호를 사람이 읽기 좋은 포맷으로 출력하는 역할도 전화번호 클래스에 맡긴다.

```jsx
class Person {
	...
  get telephoneNumber() { return this.telephoneNumber.toString(); }
	...
}

class TelephoneNumber{
  toString() { return `(${this.areaCode}) ${this.number}`; }
	...
}
```

[7] 전화번호는 여러모로 쓸모가 많으니 이 클래스는 클라이언트에게 공개하는 것이 좋겠다.

그러러면 office로 시작하는 메서드들을 없애고 TelephoneNumber의 접근자를 바로 사용하도록 바꿀 수 있다.

그런데 기왕 이렇게 쓸 거라면 전화번호 값 객체로 만드는 게 나으니 참조를 값으로 바꾸기(9.4)부터 적용한다.

## 클래스 인라인하기(Inline Class)

> 반대 리팩터링

[클래스 추출하기]

```jsx
class Person {
  get officeAreaCode() {
    return this.telephoneNumber.areaCode;
  }
  get officeNumber() {
    return this.telephoneNumber.nubmer;
  }
}
class TelephoneNumber {
  get areaCode() {
    return this.areaCode;
  }
  get number() {
    return this.number;
  }
}
```

```jsx
class Person {
  get officeAreaCode() {
    return this.officeAreaCode;
  }
  get officeNumber() {
    return this.officeNubmer;
  }
}
```

### 배경

클래스 인라인하기는 클래스 추출하기의 반대 리팩터링이다. 나는 더 이상 제 역할을 못해서 그대로 두면 안 되는 클래스는 인라인해버린다.

역할을 옮기는 리팩터링을 하고 나니 특정 클래스에 남은 역할이 거의 없을 때 이런 현상이 자주 생긴다.

이럴 땐 이 불쌍한 클래스를 가장 많이 사용하는 클래스로 흡수시키자.

두 클래스의 기능을 지금과 다르게 배분하고 싶을 때도 클래스를 인라인한다.

클래스를 인라인해서 하나로 합친 다음 새로운 클래스를 추출하는 게 쉬울 수도 있기 때문이다.

이는 코드를 재구성할 때 흔히 사용하는 방식이기도 하다.

상황에 따라 한 컨텍스트의 요소들을 다른 쪽으로 하나씩 옮기는 게 쉬울 수 있고,

인라인 리팩터링으로 하나로 합친 후 추출하기 리팩터링으로 다시 분리하는 게 쉬울 수도 있다.

### 절차

1. 소스 클래스의 각 public 메서드에 대응하는 메서드들을 타깃 클래스에 생성한다. 이 메서드들은 단순히 작업을 소스 클래스로 위임해야 한다.
2. 소스 클래스의 메서드를 사용하는 코드를 모두 타킷 클래스의 위임 메서드를 사용하도록 바꾼다.
3. 소스 클래스의 메서드와 필드를 모두 타깃 클래스로 옮긴다.
4. 소스 클래스를 삭제하고 조의를 표한다.

### 예시

배송 추적 정보를 표현하는 TrackingInfomation 클래스가 있다.

```jsx
class TrackingInfomation {
  get shippingCompany() {
    return this.shippingCompany;
  }
  set shippingCompany(arg) {
    this.shippingCompany = arg;
  }
  get trackingNumber() {
    return this.trackingNumber;
  }
  set trackingNumber(arg) {
    this.trakingNumber = arg;
  }
  get display() {
    return `${this.shippingCompany}: ${this.trackingNumber}`;
  }
}
```

이 클래스는 shipment 클래스의 일부처럼 사용된다.

```jsx
class Shipment {
  get trakingInfo() {
    return this.trackingInfomation.display;
  }
  get trackingInfomation() {
    return this.trackingInfomation;
  }
  set trackingInfomation(aTrakingInfomation) {
    this.trackingInfomation = aTrakingInfomation;
  }
}
```

TrackingInfomation이 예전에는 유용했을지 몰라도 현재는 제역할을 못 하고 있으니 Shipment 클래스로 인라인한다.

먼저 TrackingInfomation의 메서드를 호출하는 코드를 찾는다.

```jsx
aShipment.trackingInfomation.shippingCompany = request.vendor;
```

[1] 이처럼 외부에서 직접 호출하는 TrackingInfomation의 메서드들을 모조리 Shipment로 옮긴다.

그런데 보통 함수 옮기기(8.1)와는 약간 다르게 진행해보자.

먼저 Shipment에 위임 함수를 만들고 [2] 클라이언트가 이를 호출하도록 수정하는 것이다.

```jsx
class Shipment {
  set shippingCompany(arg) {
    this.trackingInfomation.shippingCompany = arg;
  }
}

aShipment.shippingCompany = request.vendor;
```

클라이언트에서 사용하는 TrackingInfomation의 모든 요소들을 이런 식으로 처리한다.

[3] 다 고쳤다면 TrackingInfomation의 모든 요소를 Shipment로 옮긴다.

먼저 display() 메서드를 인라인한다.

```jsx
class Shipment {
	...
	get trackingInfo() {
		return `${this.shippingCompany}: ${this.trackingNumber}`;
	}
}
```

다음은 배송 회사 필드 차례다.

```jsx
class Shipment {
	...
  get shippingCompany() { return this.shippingCompany; }
  set shippingCompany(arg) { this.shippingCompany = arg; }
}
```

여기서는 이동할 목적지인 Shipment에서 shippingCompany()만 참조하므로 필드 옮기기의 절차는 모두 수행하지 않아도 된다.

그래서 타깃을 참조하는 링크를 소스에 추가하는 단계는 생략한다.

이 과정을 반복하고 [4] 다 옮겼다면 TrackingInfomation 클래스를 삭제한다.

```jsx
class Shipment {
  get trakingInfo() {
    return `${this.shippingCompany}: ${this.trackingNumber}`;
  }
  get shippingCompany() {
    return this.shippingCompany;
  }
  set shippingCompany(arg) {
    this.shippingCompany = arg;
  }
  get trackingNumber() {
    return this.trackingNumber;
  }
  set trackingNumber(arg) {
    this.trakingNumber = arg;
  }
}
```

## 위임 숨기기(Hide Delegate)

```jsx
manager = aPerson.department.manager;
```

```jsx
manager = aPerson.manager;

class Person {
  get manager() {
    return this.department.manager;
  }
}
```

### 배경

모듈화 설계를 제대로 하는 핵심은 캡슐화다. 어쩌면 가장 중요한 요소다.

캡슐화는 모듈들이 시스템의 다른 부분에 대해 알아야 할 내용을 줄여준다.

캡슐화가 잘 되어 있다면 무언가 변경해야 할 때 함께 고려해야 할 모듈 수가 적어져서 코드를 변경하기가 쉬워진다.

객체 지향을 처음 배울 때는 캡슐화란 필드를 숨기는 것이라고 배운다.

그러다 경험이 쌓이면서 캡슐화의 역할이 그보다 많다는 사실을 깨닫는다.

예컨대 서버 객체의 필드가 가리키는 객체(위임 객체[delegate])의 메서드를 호출하려면 클라이언트는 이 위임 객체를 알아야 한다.

위임 객체의 인터페이스가 바뀌면 이 인터페이스를 사용하는 모든 클라이언트가 코드를 수정해야 한다.

이러한 의존성을 없애려면 서버 자체에 위임 메서드를 만들어서 위임 객체의 존재를 숨기면 된다.

그러면 위임 객체가 수정되더라도 서버 코드만 고치면 되며, 클라이언트는 아무런 영향을 받지 않는다.

### 절차

1. 위임 객체의 각 메서드에 해당하는 위임 메서드를 서버에 생성한다.
2. 클라이언트가 위임 객체 대신 서버를 호출하도록 수정한다. 하나씩 바꿀때마다 테스트한다.
3. 모두 수정했다면, 서버로부터 위임 객체를 얻는 접근자를 제거한다.
4. 테스트한다.

### 예시

사람(person)과 사람이 속한 부서(department)를 다음처럼 정의했다.

```jsx
class Person {
  constructor(name) {
    this.name = name;
  }
  get name() {
    return this.name;
  }
  get department() {
    return this.department;
  }
  set department(arg) {
    this.department = arg;
  }
}

class Department {
  get chargeCode() {
    return this.chargeCode;
  }
  set chargeCode(arg) {
    this.chargeCode = arg;
  }
  get manager() {
    return this.manager;
  }
  set manager(arg) {
    this.manager = arg;
  }
}
```

클라이언트에서 어떤 사람이 속한 부서의 관리자를 알고싶다고 하자.

그러기 위해서는 부서 객체부터 얻어와야 한다.

```jsx
manager = aPerson.department.manager;
```

보다시피 클라이언트는 부서 클래스의 작동 방식, 다시 말해 부서 클래스가 관리자 정보를 제공한다는 사실을 알아야 한다.

[1] 이러한 의존성을 줄이려면 클라이언트가 부서 클래스를 볼 수 없게 숨기고, 대신 사람 클래스에 간단한 위임 메서드를 만들면 된다.

```jsx
class Person {
  get manager() {
    return this.department.manager;
  }
}
```

[2] 이제 모든 클라이언트가 이 메서드를 사용하도록 고친다.

```jsx
manager = aPerson.manager;
```

[3] 클라이언트 코드를 다 고쳤다면 Person의 department 게터를 삭제한다.

## 중개자 제거하기(Remove Middle Man)

> 반대 리팩터링

[위임 숨기기]

```jsx
manager = aPerson.manager;

class Person {
  get manager() {
    return this.department.manager;
  }
}
```

```jsx
manager = aPerson.department.manager;
```

### 배경

위임 숨기기(7.7)의 ‘배경’ 절에서 위임 객체를 캡슐화하는 이점을 설명했다.

하지만 그 이점이 거저 주어지는 건 아니다.

클라이언트가 위임 객체의 또 다른 기능을 사용하고 싶을 때마다 서버에 위임 메서드를 추가해야 하는데,

이렇게 기능을 추가하다 보면 단순히 전달만 하는 위임 메서드들이 점점 성가셔진다.

그러면 서버 클래스를 그저 중개자 역할로 전락하여, 차라리 클라이언트가 위임 객체를 직접 호출하는 게 나을 수 있다.

```
이 냄새는 데메테르 법칙(Law of Demeter)을 너무 신봉할 때 자주 나타난다.
나는 이 법칙을 '이따금 유용한 데메테르의 제안' 정도로 부르는 게 훨씬 낫다고 생각한다.
```

어느 정도까지 숨겨야 적절한지를 판단하기란 쉽지 않지만,

우리에게는 다행히 위임 숨기기와 중개자 제거하기 리팩터링이 있으니 크게 문제되지는 않는다.

필요하면 언제든 균형점을 옮길 수 있으니 말이다.

시스템이 바뀌면 ‘적절하다’의 기준도 바뀌기 마련이다. 6개월 전에는 바람직했던 캡슐화가 이제는 어색할 수 있다.

리팩터링은 결코 미안하다는 말을 하지 않는다. 즉시 고칠 뿐이다.

### 절차

1. 위임 객체를 얻는 게터를 만든다.
2. 위임 메서드를 호출하는 클라이언트가 모두 이 게터를 거치도록 수정한다.
3. 모두 수정했따면 위임 메서드를 삭제한다.

   → 자동 리팩터링 도구를 사용할 때는 위임 필드를 캡슐화한 다음, 이를 사용하는 모든 메서드를 인라인한다.

### 예시

자신이 속한 부서 (department) 객체를 통해 manager를 찾는 Person 클래스를 살펴보자.

```jsx
manager = aPerson.manager;
```

```jsx
class Person {
  get manager() {
    return this.department.manager;
  }
}
class Depaertment {
  get manager() {
    return this.manager;
  }
}
```

사용하기 쉽고 부서는 캡슐화되어 있다.

하지만 이런 위임 메서드가 많아지면 사람 클래스의 상당 부분이 그저 위임하는 데만 쓰일 것이다.

그럴 떄는 중개자를 제거하는 편이 낫다.

[1] 먼저 위임 객체 (부서)를 얻는 게터를 만들자.

```jsx
class Person {
	...
	get department() { return this.department; }
}
```

[2] 이제 각 클라이언트가 부서 객체를 직접 사용하도록 고친다.

```jsx
manager = aPerson.department.manager;
```

[3] 클라이언트를 모두 고쳤다면 Person()의 manager() 메서드를 삭제한다.

Person에 단순한 위임 메서드가 없을 때까지 이 작업을 반복한다.

위임 숨기기나 중개자 제거하기를 적당히 섞어도 된다. 자주 쓰는 위임은 그대로 두는 편이 클라이언트 입장에서 편하다.

둘 중 하나를 반드시 해야 한다는 법은 없다. 상황에 맞게 처리하면 되고, 합리적인 사람이라면 어떻게 해야 가장 효과적인지 판단할 수 있을 것이다.

- 자동 리팩터링을 사용한다면
  다른 방식으로도 처리할 수 있다.
  먼저 부서 필드를 캡슐화 한다. 그러면 관리자 게터에서 부서의 public 게터를 사용할 수 있다.
  ```jsx
  class Person {
    get manager() {
      return this.department.manager;
    }
  }
  ```
  자바스크립트에서는 이 변화가 잘 드러나지 않지만, department 앞에 밑줄이 없다면
  더 이상 필드를 직접 접근하지 않고 새로 만든 게터를 사용한다는 뜻이다.
  그런 다음 manager() 메서드를 인라인하여 모든 호출자를 한 번에 교체한다.

## 알고리즘 교체하기 (Substitute Algorithm)

```jsx
function foundPerson(people) {
  for (let i = 0; i < people.length; i++) {
    if (people[i] === "Don") {
      return "Don";
    }
    if (people[i] === "John") {
      return "John";
    }
    if (people[i] === "Kent") {
      return "Kent";
    }
  }
  return "";
}
```

```jsx
function foundPerson(people) {
  const candidates = ["Don", "John", "Kent"];
  return people.find((p) => candidates.includes(p)) || "";
}
```

### 배경

어떤 목적을 달성하는 방법은 여러가지가 있기 마련이다.

그중에서도 다른 것보다 더 쉬운 방법이 분명히 존재한다. 알고리즘도 마찬가지다.

나는 더 간명한 방법을 찾아내면 복잡한 기존 코드를 간명한 방식으로 고친다.

리팩터링하면 복잡한 대상을 단순한 단위로 나눌 수 있지만, 때로는 알고리즘 전체를 걷어내고 훨씬 간결한 알고리즘으로 바꿔야 할 때가 있다.

내 코드와 똑같은 기능을 제공하는 라이브러리를 찾았을 때도 마찬가지다.

알고리즘을 살짝 다르게 동작하도록 바꾸고 싶을 때도 이 변화를 더 쉽게 가할 수 있는 알고리즘으로 통째로 바꾼 후에 처리하면 편할 수 있다.

이 작업에 착수하려면 반드시 메서드를 가능한 잘게 나눴는지 확인해야 한다.

거대하고 복잡한 알고리즘을 교체하기란 상당히 어려우니 알고리즘을 간소화하는 작업부터 해야 교체가 쉬워진다.

### 절차

1. 교체할 코드를 함수 하나에 모은다.
2. 이 함수만을 이용해 동작을 검증하는 테스트를 마련한다.
3. 대체할 알고리즘을 준비한다.
4. 정적 검사를 수행한다.
5. 기존 알고리즘과 새 알고리즘의 결과를 비교하는 테스트를 수행한다. 두 결과가 같다면 리팩터링이 끝난다.

   그렇지 않다면 기존 알고리즘을 참고해서 새 알고리즘을 테스트하고 디버깅한다.
