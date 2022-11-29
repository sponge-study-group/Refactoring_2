# Chapter03 코드에서 나는 악취 

이 장의 내용과 부록 B를 참고해서 리팩터링이 필요한 코드의 감을 잡고
6~12장에서 리팩터링 기법을 찾아서 도움이 될지 생각해본다.

## 3.1 기이한 이름

함수, 모듈, 클래스 등은 이름만 보고도 무슨 일을 하고 어떻게 사용해야 하는지
명확히 알 수 있도록 신경써야 한다.

## 3.2 중복 코드

* 한 클래스에 두 메서드가 똑같은 표현식을 사용하는 경우 
  <b>함수 추출하기</b>로 양쪽 모두 추출된 메서드를 호출한다.

* 코드가 비슷한데 완전 같지 않다면?
  <b>문장 슬라이드</b>하기로 비슷한 부분을 모두 함수 추출하기를 더 쉽게 적용할 수 있는지 살펴본다.

* 같은 부모로부터 파생된 서브 클래스들에 코드가 중복되어 있다면?
  각자 따로 호출되지 않도록 <b>메서드 올리기</b>를 적용해 부모로 올린다.

>✅ **함수 쪼개기 단계**
1. 반복문 쪼개기
2. 문장 슬라이드하기 : 변수 초기화 문장을 변수 값 누적 코드  바로 앞으로 옮긴다.
3. 함수 추출하기 : 계산된 코드를 별도 함수로 추출한다.
4. 변수 인라인하기 : 변수를 제거하고 별도로 추출한 함수를 인라인한다.


## 3.3 긴 함수

>코드를 이해하고, 공유하고, 선택하기 쉬워진다는 장점은 함수를 짧게 구성할 때 나온다. 이것을 간접 호출 효과라고 한다.

예전 언어는 서브루틴을 호출하는 비용이 컸기 때문에 짧은 함수를 꺼렸다.
하지만 요즘 언어는 프로세스 안에서 함수 호출 비용을 거의 없애버렸다.
\* 서브 루틴(Sub-routine)이란 ? 프로그래밍에서는 함수 안에 함수가 있을 경우
바로 안쪽의 함수를 서브루틴이라 부른다. [참고](https://kotlinworld.com/214#%EC%--%-C%EB%B-%-C%EB%A-%A-%ED%-B%B-%EC%-D%B-%EB%-E%--%--%EB%AC%B-%EC%--%--%EC%-D%B-%EA%B-%--%-F)

```javascript
function ruotine1(){
  routine() <= 서브루틴
}
function routine(){
  console.log(11);
}
```

그럼에도 함수 호출부와 선언부를 왔다갔다 해야하는 부담이 있다. 이것을 해결하기 위한 가장 확실한 방법은 함수 이름을 잘 짓는 것이다.

주석을 달아야 할 만한 부분은 무조건 함수로 만든다. 주석으로 설명하려던 코드가 담기고 함수 이름은 동작 방식이 아닌 의도가 드러나게 짓는다. 원래 코드보다
길어지더라도 함수로 뽑는다. 단, 함수 이름에 코드의 목적을 드러내야 한다.

>핵심은 함수의 길이가 아닌, 함수의 목적(의도)과 구현 코드의 괴리가 얼마나 큰가다. 즉 '무엇을 하는지'를 코드가 잘 설명해주지 못할수록 함수로 만드는 게 유리하다.

함수가 매개변수와 힘시 변수를 많이 사용한다면 추출 작업에 방해가 된다.
이런 상황에서 함수를 추출하다 보면 추출된 함수에도 매개변수가 많아져서 
리팩터링 전보다 난해해질 수 있다. 그렇다면 임시 변수를 질의 함수로 바꾸기로 임시 변수의 수를, 매개변수 객체 만들기와 객체 통째로 넘기기로는 매개변수의 
수를 줄일 수 있을 것이다.
이 리팩터링을 적용해도 임시 변수와 매개변수가 많다면 함수 명령으로 바꾸기를 고려해본다.

조건문이나 반복문도 추출 대상의 실마리를 제공한다. 조건문으 조건문 분해하기로
swich문을 구성하는 case문마다 함수 추출하기를 적용해서 각 case의 본문을 
함수 호출문 하나로 바꾼다.  같은 조건을 기준으로 나뉘는 swich문이 여러 개라면
조건부 로직을 다형성으로 바꾸기를 적용한다.

반복문도 그 안의 코드와 함께 추출해서 독립된 함수로 만든다. 추출한 반복문 코드에
적합한 이름이 떠오르지 않는다면 성격이 다른 두 가지 작업이 섞여 있기 때문일 수 있다. 이럴때는 반복문 쪼개기를 적용해서 작업을 분리한다.

## 3.4 긴 매개변수 목록
Long Parameter List

>클래스는 매개변수 목록을 줄이는 데 효과적인 수단이기도 하다.

* 매개변수에서 값을 얻어올 수 있는 매개변수는 매개변수 질의 함수로 바꾸기로 제거한다.
* 사용중인 데이터 구조에서 값들을 뽑아 각각을 별개의 배개변수로 전달하는 코드는 객체 통째로 넘기기를 적용해서 원본 데이터 구조를 그대로 전달한다.
* 항상 함께 전달되는 매개변수들은 매개변수 객체 만들기로 하나로 묶는다.
* 함수의 동작 방식을 정하는 플래그 역할의 매개변수는 플래그 인수 제거하기로 없앤다.
* 여러 함수가 특정 매개변수들의 값을 공통으로 사용할 때 여러 함수를 클래스로 묶기를 이용해 공통 값들을 클래스 필드로 정의한다.

## 3.5 전역 데이터
Global Data

전역 데이터는 코드베이스 어디에서든 건드릴 수 있고 값을 누가 바꿨는지 찾아낼 메커니즘이 없다는 게 문제다.
이를 방지하기 위해 변수 캡슐화하기를 사용한다.
이런 데이터를 함수로 감싸는 것만으로 데이터를 수정하는 부분을 쉽게 찾을 수 있고 
접근을 통제할 수 있게 된다. 더 나아가 함수들을 클래스나 모듈에 집어넣고 그 안에서만 사용할 수 있도록 접근 범위를 최소로 줄이는 것도 좋다.

## 3.6 가변 데이터
Mutable Data

코드의 다른 곳에서 다른 값을 기대한다는 사실을 인식하지 못한 채 수정해버리면
프로그램이 오작동한다.

>함수형 프로그래밍에서는 데이터는 절대 변하지 않고, 데이터를 변경하려면 
반드시 (원래 데이터는 그대로 둔 채) 변경하려는 값에 해당하는 복사본을 
만들어서 반환한다는 개념을 기본으로 삼고 있다.

* 변수 캡슐화하기로 정해놓은 함수를 거쳐야만 값을 수정할 수 있도록 한다.
* 변수 쪼개기로 용도별로 독립 변수에 저장하게 하여 값 갱신이 문제를 일으킬 여지를 없앤다. (갱신 로직은 다른 코드와 떨어트려 놓는다.)
* 문장 슬라이드하기와 함수 추출하기로 갱신하는 코드로부터 부작용이 없는 코드를 분리한다.
* API를 만들 때는 질의 함수와 변경 함수 분리하기를 활용해서 꼭 필요한 경우가
아니라면 부작용이 있는 코드를 호출할 수 없게 한다.
(세터 제거하기도 적용한다.)
* 여러 함수를 클래스로 묶기나 여러 함수를 변환 함수로 묶기를 활용해서 
변수를 갱신하는 코드들의 유효범위를 제한한다. (유효범위가 넓어지는 변수가 위험하므로) 
* 내부 필드에 데이터를 담고 있는 변수라면 참조를 값으로 바꾸기를 적용하여,
내부 필드를 직접 수정하지 말고 구조체를 통째로 교체하는 편이 낫다.

## 3.7 뒤엉킨 변경
Divergent Change

뒤엉킨 변경은 단일 책임 원칙(single Responsibility Principle, SRP)이 제대로 지켜지지 않을 때 나타난다.
즉, 하나의 모듈이 서로 다른 이유들로 인해 여러 가지 방식으로 변경되는 일이
많을 때 발생한다.
(데이터베이스가 추가될 때마다 함수 세 개를 바꿔야 하고, 금융 삼품이 추가될 때마다 또 다른 함수 네 개를 바꿔야 하는 경우 등)

* 순차적 실행은 단계 쪼개기
* 전체 처리 과정에 각기 다른 맥락의 함수를 호출하는 빈도가 높다면, 각 맥락에 해당하는 적당한 모듈들을 만들어서 관련 함수를 모으는 함수 옮기기
* 맥락의 일에 관여하는 함수가 있다면 함수 추출하기부터 수행
* 모듈이 클래스라면 클래스 추출하기로 맥락별 분리 방법을 찾는다.

## 3.8 산탄총 수술
Shotgun Surgery

이 냄새는 코드를 변경할 때마다 자잘하게 수정해야 하는 클래스가 많은 경우 풍긴다.

| 뒤엉킨 변경 | 산탄총 수술
원인 | 맥락을 잘 구분하지 못함
해법(원리) | 맥락을 명확히 구분
발생 과정(현상) | 한 코드에 섞여 들어감 | 여러 코드에 흩뿌려짐
해법(실제 행동) | 맥락별로 분리 | 맥락별로 모음

## 3.9 기능 편애
Feature Envy

## 3.10 데이터 뭉치
Data Clumps

## 3.11 기본형 집착
Primitive Obsession

대부분의 프로그래밍 언어는 정수, 부동소수점 수, 문자열 같은 기본형을 제공한다. 라이브러리를 통해 날짜 같은 간단한 객체를 추가로 제공하기도 한다. 

\* 기본 타입(Primitive type)
타입(data type)은 해당 데이터가 메모리에 어떻게 저장되고, 프로그램에서 어떻게 처리되어야 하는지를 명시적으로 알려주는 역할을 합니다.
자바에서는 여러 형태의 타입을 미리 정의하여 제공하고 있는데, 이것을 기본 타입(primitive type)이라고 합니다.
자바의 기본 타입은 모두 8종류가 제공되며, 크게는 정수형, 실수형, 문자형 그리고 논리형 타입으로 나눌 수 있습니다. [참고](http://www.tcpschool.com/java/java_datatype_basic)

한편 프로그래머 중에는 문제에 맞는 기초 타입(화폐, 좌표, 구간 등)을 직접 정의하기 않고 금액을 숫자형으로 계산하거나, 물리량을 계산할 때 밀리미터나 인치 같은 단위를 무시하고 자료형들을 문자열로만 표현하기도 한다. 이것을 문자열화된 변수라고 부른다.

* 기본형을 객체로 바꾸기
* 기본형으로 표현된 코드가 조건부 동작을 제어하는 타입 코드로 쓰였다면 타입 코드를 서브클래스로 바꾸기와 조건부 로직을 다형성으로 바꾸기
* 기본형 그룹은 클래스 추출하기와 매개변수 객체 만들기

## 3.12 반복되는 swich문
Repeated Swiches

똑같은 조건부 로직이 여러 곳에서 반복해 등장하는 코드를 다형성으로 바꿔준다.

## 3.13 반복문
Loops 

일급 함수(first-class function)를 지원하는 언어를 사용한다면 <b>반복문을 파이프라인으로 바꾸기</b>를 적용해서 반복문을 제거한다.
<strong>파이프라인 연산</strong>
* filter
* map

## 3.14 성의 없는 요소
Lazy Element

사용하지 않는 프로그램 요소는 제거한다.
(\* 프로그래밍 언어가 제공하는 함수(메서드), 클래스, 인터페이스 등 코드 구조를 잡는 데  활용되는 요소)

* 함수 인라인하기
* 클래스 인라인하기
* 계층 합치기

## 3.15 추측성 일반화
Speculative Generality

나중에 필요할거라고 추측하고 미리 만든 코드를 제거한다.

## 3.16 임시 필드
Temporary Feild

특정 상황에서만 값이 설정되는 필드를 가진 클래스는 
1. 클래스 추츨하기 후
2. 함수 옮기기로 임시 필드들과 관련된 코드를 새 클래스에 몰아넣는다.
3. 특히 케이스 추가하기로 필드들이 유효하지 않을 때는 위한 대안 클래스를 만들어서 제거한다.

## 3.17 메시지 체인
Message Chains

```javascript
menagerName = aPerson.department.manager.name;
/**
 * 체인을 구성하는 모든 객체에 위임 숨기기를 적용할 수 있다고 함은 부서장 이름을 
 * 바로 반환하는 메서드를 사람 클래스에 추가할 수도 있고 부서 클래스에 추가할 수도 있다는
 * 뜻이다. 혹은 부서장을 얻는 메서드를 사람 클래스에 추가할 수도 있다.
*/

managerName = aPerson.department.managerName; // 관리자 객체(manager)의 존재를 숨김
managerName = aPerson.manager.name; // 부서 객체(department)의 존재를 숨김
managerName = aPerson.managerName; // 부서 객체와 관리자 객체 모두의 존재를 숨기
/**
 * 이 체인의 최종 결과 객체는 name이 반환하는 부서장의 이름이다. 이 객체가 다음처럼 쓰인다고 해보자.
*/

managerName = aPerson.department.menager.name
report = `${managerName}께 ${aPerson.name}님의 작업 로그 ...`
/**
 * 여기서 보고서 생성 로직을 함수로 추출한 다음 적당한 모듈로 옮기면 체인의 존재가 감춰진다.
*/

console.log(report);

console.log(reportAutoGenerator.report(aPerson));
/**
 * 마지막으로 체인의 중간인 부서 정보를 얻어 사용하는 다수의 클라이언트가 부서장 이름도 
 * 함께 사용한다면 부서 클래스에 managerName() 메서드를 추가하여 체인을 단축할 수 있다.
*/

```

## 3.18 중개자
Middle Man


## 3.19 내부자 거래
Insider Trading


## 3.20 거대한 클래스
Large Class

클래스가 너무 많은 일을 하려고 하면 필드 수가 늘어난다. 그리고 중복 코드가 생기기 쉽다.

* 클래스 추출하기로 필드들 일부를 따로 묶는다.
한 클래스 안에서 접두어나 접미어가 같은 필드들이 함께 추출할 후보들이다.
* 클래스와 상속 관계로 만드는게 좋다면(클래스 추출보다) 슈퍼클래스 추출하기나(실질적으로 서브클래스 추출하기) 타입 코드를 서브클래스로 바꾸기를 적용한다.
* 로직이 같은 100줄짜리 메서드가 있다면 공통 부분을 작은 메서드로 뽑아낸다.
* 클라이언트들의 거대 클래스 이용 패턴을 파악하여 주로 사용하는 기능을 개별 클래스로 추출한다. 
클래스 추출하기, 슈퍼 클래스 추출하기, 타입 코드를 서브클래스로 바꾸기 등을 활용하여 여러 클래스로 분리한다.

## 3.21 서로 다른 인터페이스의 대안 클래스들
Alternative Classes with Different Interfaces

## 3.22 데이터 클래스
Data Class

데이터 클래스란 데이터 필드와 게터/세터 메서드로만 구성된 클래스를 말한다.
데이터 저장 용도로만 쓰이다 보니 다른 클래스가 너무 깊이까지 함부로 다룰 때가 많다.
이런 클래스에 public 필드가 있다면 레코드 캡슐화하기로 숨긴다. 
변경하면 안되는 필드는 세터 제거하기로 접근을 원천 봉쇄한다.


## 3.23 상속 포기
Refused Bequest

부모 클래스의 메서드와 데이터를 물려받고 싶지 않다면 메서드 내리기와 필드 내리기를 활용하여 
물려받지 않을 부모 코드를 새로 만든 서브클래스로 넘긴다.

>서브클래스는 부모의 구현을 따르지 않을 수는 있지만 인터페이스는 따르는 것이 좋다.

서브클래스가 부모의 동작을 필요로하지만 인터페이스는 따르고 싶지 않을 경우,
서브클래스 위임으로 바꾸기나 슈퍼클래스 위임으로 바꾸기를 활용해서 상속 메커니즘에서 벗어난다.

## 3.24 주석
Comments

주석을 사용하는 것은 좋지만 장황하게 달았을 경우 코드를 다시 돌아보자.

* 특정 코드 블록이 하는 일에 주석을 남기고 싶다면 함수 추출하기
* 추출되어 있는 함수임에도 설명이 필요하다면 함수 선언 바꾸기 
* 시스템이 동작하기 위한 선행조건을 명시하고 싶다면 어서션 추가하기 

>주석을 남겨야겠다는 생각이 들면, 가장 먼저 주석이 필요 없는 코드로 리팩터링해본다.

<주석을 달아도 괜찮을 때>
* 뭘 할지 모를 때, 진행 상황 뿐 아니라 확실하지 않은 부분에 주석을 남긴다. 
* 코드를 작성한 이유를 설명하는 용도로 달 수도 있다. 이런 정보는 나중에 코드를 수정해야 할 프로그래머에게 도움이 될 것이다.













