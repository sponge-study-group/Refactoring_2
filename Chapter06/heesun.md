# Chapter06 기본적인 리팩터링

* 가장 기본적이고 많이 사용하는 리팩터링들
   - 함수 추출하기 <-> 함수 인라인하기
   - 변수 추출하기 <-> 변수 인라인하기
   - 함수 선언 바꾸기
   - 변수 이름 바꾸기
   - 변수 캡슐화하기
   - 매개변수 객체 만들기
   - 여러 함수를 클래스로 묶기
   - 여러 함수를 변환 함수로 묶기
   - 단계 쪼개기



## 6.1 함수 추출하기
Extract Function

>함수 추출하기는 길이가 한 화면을 넘어가거나 두 번 이상 사용될 경우 함수로 만들고
한 번만 쓰이는 코드는 인라인 상태로 두는 등의 기준을 가질 수 있지만, 여기서는
목적과 구현을 분리하는 방식을 사용한다. 무슨 일을 하는지 한 눈에 파악할 수 있도록 
하기 위해 함수를 추출한다.

>함수는 객체 지향 언어의 메서드나 절차형 언어의 프로시저/서브루틴에도 똑같이 적용된다.

* 반대 리팩터링 : 함수 인라인하기
* 1판에서 이름 : 메서드 추출

```javascript
function printOwing(invoice) {
   printBanner();
   const outstanding = calculateOutstanding(invoice);
   
   // 세부 사항 출력
   console.log(`고객명 : ${invoice.customer}`);
   console.log(`채무액 : ${outstanding}`);
}
⬇️
function printOwing(invoice) {
   printBanner();
   const outstanding = calculateOutstanding(invoice);
   printDetails(outstanding);

   function printDetails(outstanding){
      console.log(`고객명 : ${invoice.customer}`);
      console.log(`채무액 : ${outstanding}`);
   }
}
```

## 6.2 함수 인라인하기
Inline Function

* 반대 리팩터링 : 함수 추출하기
* 1판에서의 이름 : 메서드 내용 직접 삽입

>함수 본문이 간접 호출되는 함수를 인라인해도 상관 없을 만큼 명확한 이름을 가지고 있을 때 인라인하다.

```javascript
function getRating(driver){
   return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}
function moreThanFiveLateDeliveries(driver){
   return driver.numberOfLateDeliveries > 5;
}
⬇️
function getRating(driver){
   return (driver.numberOfLateDeliveries > 5) ? 2 : 1;
}
```

* 복잡한 코드 예시

```javascript
function reportLines(aCustomer){
   const lines = [];
   gettherCustomerDate(lines, aCustomer);
   return lines;
}
function gettherCustomerData(out, aCustomer){
   out.push(["name", aCustomer.name]);
   out.push(["location", aCustomer.location]);
}
⬇️
function reportLines(aCustomer){
   const lines = [];
   lines.push(["name", aCustomer.name]);
   getherCustomerData(lines, aCustomer);
   return lines;
}
function getherCustomerData(out, aCustomer){
   out.push(["location", aCustomer.location]);
}
```

## 6.3 변수 추출하기
Extract Variable

* 반대 리팩터링 : 변수 인라인하기
* 1판에서의 이름 : 직관적 임시변수 사용

>표현식이 너무 복잡해서 이해하기 어려울 때 사용한다.

```javascript
return order.quantity * order.itemPrice - 
   Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
   Math.min(order.quantity * order.itemPrice * 0.1, 100);
⬇️
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500)
   * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

## 6.4 변수 인라인하기
Inline Variable

* 반대 리팩터링 : 변수 추출하기
* 1판에서의 이름 : 임시변수 내용 직접 삽입

>변수가 표현식과 다를바 없을 때 표현식으로 인라인한다.

```javascript
let basePrice = anOrder.basePrice;
return (basePrice > 1000);
⬇️
return anOrder.basePrice > 1000;
```

## 6.5 함수 선언 바꾸기
Change Function Declaration

* 다른이름 : 함수이름 바꾸기, 시그니처 바꾸기
* 1판에서의 이름 : 메서드명 변경, 매개변수 추가, 매개변수 제거

>정답이 없다.

```javascript
function circum(radius){ ... }
⬇️
function circumference(radius){ ... }
```

## 6.6 변수 캡슐화하기
Encapsulate Variable

* 1판에서의 이름 : 필드 자체 캡슐화, 필드 캡술화

>데이터는 참조하는 모든 부분을 한 번에 바꿔야 코드가 제대로 작동한다. 짧은 함수 안의 임시 변수처럼 유효범위가 좁은 데이터는 어렵지 않지만, 전역 데이터처럼 유효범위가 넓을 수록 어려워진다. 그래서 접근 범위가 넓은 데이터를 옮길 때는 먼저 그 데이터로의 접근을 독점하는 함수를 만드는 식으로 캡슐화하는 것이 좋은 방법일 때가 많다. 데이터 재구성이라는 어려운 작업을 함수 재구성이라는 더 단순한 작업으로 변환하는 것이다.

```javascript
let defaultOwner = {firstName: "마틴", lastName: "파울러"}
⬇️
let defaultOwnerData = {firstName: "마틴", lastName: "파울러"}
export function defaultOwner() { return defaultOwnerData; }
export function setDefaultOwner(arg) { defaultOwnerData = arg; }
```

```javascript
// 전역 변수 담긴 중요한 데이터
let defaultOwner = {firstName: "마틴", lastName: "파울러"}
// 참조하는 코드
spaceship.owner = defaultOwner;
// 갱신하는 코드
defaultOwner = {firstName: "레베카", lastName: "파슨스"}
```
1. 캡술화를 위해 get/set 함수를 정의한다.   
```javascript
export function defaultOwner() { return defaultOwnerData; }
export function setDefaultOwner(arg) { defaultOwnerData = arg; }
```   

2. defaultOwner를 참조하는 코드를 찾아서 게터 함수를 호출한다.
```javascript
spaceship.owner = getDefaultOwner();
```
대입문은 세터 함수를 사용한다.
```javascript
setDefaultOwner({firstName:"레베카", lastName:"파슨스"});
```

* 게터로 얻은 데이터를 변경할 수 있지만 원본에 영향을 주고 싶지 않을 때
```javascript
let defaultOwnerData = {firstName: "마틴", lastName: "파울러"}
export function defaultOwner() { return Object.assign({}, defaultOwnerData); }
export function setDefaultOwner(arg) { defaultOwnerData = arg; }
```

* 원본 데이터 변경을 감지하여 막을 때
```javascript
let defaultOwnerData = {firstName: "마틴", lastName: "파울러"}
export function defaultOwner() { return new Person(defaultaOwnerData); }
export function setDefaultOwner(arg) { defaultOwnerData = arg; }

class Person {
   constructor(data){
      this._lastName = data.lastName;
      this._firstName = data.firstName;
   }
   get lastName() { return this._lastName; }
   get firstName() { return this._firstName; }
}
```

## 6.7 변수 이름 바꾸기
Rename Variable

>함수 호출 한 번ㅇ으로 끝나지 않고 값이 영속되는 필드는 특별히 더 이름 짓기에 신경써야 한다.

```javascript
let a = height * width;
⬇️
let area = heigth * width;
```

## 6.8 매개변수 객체 만들기
Introduce Parameter Object

* [https://martinfowler.com/bliki/ValueObject.html](https://martinfowler.com/bliki/ValueObject.html)

>데이터 항목 뭉치가 이 함수에서 저 함수로 몰려다닐 때 이 무리를 데이터 구조로 만들어준다.

```javascript
function amountInvoiced(startDate, endDate) { ... }
function amountReceived(startDate, endDate) { ... }
function amountOverdue(startDate, endDate) { ... }
⬇️
function amountInvoiced(aDateRange) { ... }
function amountInvoiced(aDateRange) { ... }
function amountInvoiced(aDateRange) { ... }
```

## 6.9 여러 함수를 클래스로 묶기
Combine Function into Class

>클래스는 대다수의 최신 프로그래밍 언어가 제공하는 기본적인 빌딩 블록이다.
클래스는 데이터와 함수를 하나의 공유 환경으로 묶은 후, 다른 프로그램 요소와
어우러질 수 있도록 그중 일부를 외부에 제공한다. 클래스는 객체 지향 언어의 기본인 동시에 다른 패러다임 언어에도 유용하다.

```javascript
function base(aReading) { ... }
function taxableChange(aReading) { ... }
function calculateBaseCharge(aReading) { ... }
⬇️
class Reading {
   base() { ... }
   taxableCharge() { ... }
   calculateBaseCharge() { ... }
}
```

## 6.10 여러 함수를 변환 함수로 묶기
Combine Function into Transform

>반복되는 도출 로직을 한데로 모아둔다.

```javascript
function base(aReading) { ... }
function taxableCharge(aReading) { ... }
⬇️
function enrichReading(argReading){
   const aReading = _.cloneDeep(argReading);
   aReading.baseCharge = base(aReading);
   aReading.taxableCharge = taxableCharge(aReading);
   return aReading;
}
```

## 6.11 단계 쪼개기
Split Phase

>서로 다른 두 대상을 한꺼번에 다루는 코드는 각각 별개 모듈로 나눈다. 모듈을 잘 분리하면 하나를 수정해도 나머지도 잘 수정될 수 있다. 가장 간편한 분리 방법은 동작을 연이은 두 단계로 쪼개는 것이다.

```javascript
const orderData = orderString.split(/\s+/);
const producePrice = priceList[orderData[0].split("-")[1]];
const orderPrice = parseInt(orderData[1])*productPrice;
⬇️
const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString){
   const values = aString.split(/\s+/);
   return ({
      productID: values[0].split("-")[1],
      quantity: parseInt(values[1]),
   });
}
function price(order, priceList){
   return order.quantity * priceList[order.productID];
}
```

* 상품의 결제 금액을 계산하는 코드
  
```javascript
function priceOrder(product, quantity, shoppingMethod) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
  const shippingPerCase = (basePrice > shippingMethod.discountThreshold) ? shippingMethod.discountedFee :                 shippingMethod.feePerCase;
  const shippingCost = quantity * shippingPerCase;
  const price = basePrice - discount + shippingCost;
  return price;
}
```

* 상품 가격과 배송비 계산을 두 단계로 나눔
  
```javascript
function priceOrder(product, quantity, shoppingMethod) {
  const priceData = calculatePricingData(product, quantity);
  return applyShipping(priceData, shippingMethod);
}

// 중간 데이터
function calculatePricingData(product, quantity) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0) * product.basePrice * product.discountRate;
  return {basePrice: basePrice, quantity: quantity, discount: discount};
}

function applyShipping(priceData, shippingMethod) {
  const shippingPerCase = (priceData.basePrice) > shippingMethod.discountThreshold) ? shippingMethod.discountedFee : shippingMethod.feePerCase;
  const shippingCost = priceData.quantity * shippingPerCase;
  return priceData.basePrice - priceData.discount + shippingCost;
}
```

* 명령줄 프로그램 쪼개기 (자바)

<코드가 하는 두 가지 일>
1. 주문 목록을 읽어서 개수를 센다.
2. 명령줄 인수를 담은 배열을 읽어서 프로그램의 동작을 결정한다.

<리팩터링>
1. 명령줄 인수의 구문을 분석해서 의미를 추출한다.
2. 이렇게 추출된 정보를 이용하여 데이터를 적절히 가공한다.
이렇게 분리해두면, 프로그램에서 지정할 수 있는 옵션이나 스위치가 늘어나더라도
코드를 수정하기 쉽다.

```java
public static void main(String[] args){
   try {
      if(args.length === 0) throw new RuntimeException("파일명을 입력하세요.");
      String filename = args[args.length - 1];
      File input = Paths.get(filename).toFile();
      ObjectMapper mapper = new ObjectMapper();
      Order[] orders = mapper.readValue(input, Order[].class);
      if(Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
         System.out.println(Stream.of(orders)
            .filter(o -> "ready".equals(o.status))
            .count())
      else
         System.out.println(orders.length);
   } catch (Exception e){
      System.err.println(e);
      System.exit(1);
   }
}
// 주문을 읽는 부분은 잭슨 라이브러리 이용
```

>자바로 작성된 명령줄 프로그램은 매번 JVM을 구동해야 하는데 그 과정이 느리고 
복잡하기 때문에 JUnit 호출로 자바 프로세스 하나에서 테스트 할 수 있는 상태로 만들어야 한다. 이를 위해 핵심 작업을 수행하는 코드 전부를 함수로 추출한다.

```java
public static void main(String[] args){
   try {
      System.out.println(run(args));
   } catch (Exception e) {
      System.err.println(e);
      System.exit(1);
   }
}

// 단계를 쪼갤 준비
static long run(String[] args) throw IOException {
   if (args.length == 0) throw new RuntimeException("파일명을 입력하세요.");
   String filename = args[args.length - 1];
   File input = Paths.get(filename).tofile();
   ObjectMapper mapper = new ObjectMapper();
   Order[] orders = mapper.readValue(input, Order[].class);
   if (Stream.of(args).anyMatch(arg -> "-r".equals(arg)))
      return Stream.of(orders).filter(o -> "read".equals(o.status)).count();
   else 
      return orders.length;
}
```






