# [2] 리팩터링 원칙

## 1. 리팩터링 정의

리팩터링은 엔지니어들 사이에서 다소 두루뭉실한 의미로 통용된다.

허나, 이 용어를 더 구체적인 의미로 사용하며, 그렇게 엄격하게 정의해야 더 유용하다고 생각한다.

**리팩터링** : [명사] 소프트웨어의 겉보기 동작은 그대로 유지한 채,

코드를 이해하고 수정하기 쉽도록 내부 구조를 변경하는 기법

**리팩터링(하다**) : [동사] 소프트웨어의 겉보기 동작은 그대로 유지한 채,

여러 가지 리팩터링 기법을 적용해서 소프트웨어를 재구성하다.

코드를 정리하는 작업을 모조리 ‘리팩터링’이라고 표현하는데, 앞서 제시한 정의를 따르면

특정한 방식에 따라 코드를 정리하는 것만이 리팩터링이다.

리팩터링하기 전과 후의 코드가 똑같이 동작해야 한다.

리팩터링 과정에서 발견된 버그는 리팩터링 후에도 그대로 남아 있어야 한다.

리팩터링은 성능 최적화와 비슷하지만, 목적이 다르다.

리팩터링의 목적은 코드를 이해하고 수정하기 쉽게 만든다. 성능은 좋아질 수도, 나빠질 수도 있다.

성능 최적화는 오로지 속도 개선에만 신경 쓴다. 그래서, 코드는 다루기 더 어렵게 바뀔 수 있다.

## 2. 두 개의 모자

소프트웨어를 개발할 때 목적이 ‘기능 추가’냐 ‘리팩터링’이냐를 명확히 구분하라.

켄트 벡은 이를 두 개의 모자에 비유했다.

기능을 추가할 때는 ‘기능 추가’모자를 쓴 다음 기존 코드를 절대 건드리지 않고 기능을 추가만 한다.

리팩터링할 때는 ‘리팩터링’ 모자를 쓴 다음 기능 추가는 절대 하지 않고 오로지 코드 재구성에만 전념한다.

테스트도 새로 만들지 않으며, 부득이 인터페이스를 변경해야 할 때만 기존 테스트를 수정한다.

전체 작업 시간이 10분 정도로 짧다 해도, 항상 내가 쓴 모자가 무엇인지와 그에 따른 작업 방식의 차이를 분명히 인식하자.

## 3. 리팩터링하는 이유

리팩터링은 소프트웨어의 모든 문제점을 해결하는 만병통치약은 아니지만,

코드를 건강한 상태로 유지하는 데 도와주는 약임은 분명하다.

### 리팩터링하면 소프트웨어 설계가 좋아진다.

리팩터링하지 않으면 소프트웨어의 내부 설계(아키텍쳐)가 썩기 쉽다.

아키텍처를 충분히 이해하지 못한 채 단기 목표만을 위해 코드를 수정하다보면 기반 구조가 무너지기 쉽다.

그러면 코드만 봐서는 설계를 파악하기 어려워진다.

코드만으로 설계를 파악하기 어려워질수록 설계 유지가 어렵고, 부패되는 속도는 빨라진다.

반면, 규칙적인 리팩터링은 코드의 구조를 지탱해 줄 것이다.

중복 코드 제거는 설계 개선 작업의 중요한 한 축을 차지한다.

코드량이 줄면 수정하는 데 드는 노력은 크게 달라진다.

코드가 길면 문제점

1. 실수 없이 수정하기 어렵다.
2. 이해해야 할 코드량이 늘어난다.
3. 비슷한 일을 하는 코드가 산재해 있다면 한 부분만 살짝 바꿔서는 시스템이 예상대로 동작하지 않을 수 있다.

반면 중복 코드를 제거하면 모든 코드가 언제나 고유한 일을 수행함을 보장하며, 이는 바람직한 설계의 핵심이다.

### 리팩터링하면 소프트웨어를 이해하기 쉬워진다.

프로그래밍은 그저, 컴퓨터에게 시킬 일을 코드로 작성하는 것이다.

코드를 컴파일하는 데(컴퓨터가 코드를 해석하는데) 시간이 살짝 더 걸린다고 문제가 되지 않는다.

허나 다른 프로그래머가 코드를 제대로 이해하지 못해 수정에 시간이 오래 걸린다면, 큰 문제다.

잘 작동하지만 이상적인 구조는 아닌 코드가 있다면, 잠깐 시간을 내서 리팩터링해보자.

그러면 코드의 목적이 더 잘 드러나게, 다시 말해 내 의도를 더 명확하게 전달하도록 개선할 수 있다.

사실 해당 코드를 보는 사람은 나 자신일 때가 많다.

그래서 더더욱 리팩터링이 중요하며,

코드를 보면 알 수 있는 것들은 의도적으로 기억하지 말아라.

기억할 필요가 있는 것들은 최대한 코드에 담아라.

### 리팩터링하면 버그를 쉽게 찾을 수 있다.

코드를 이해하기 쉽다는 말은 버그를 찾기 쉽다는 말이기도 하다.

리팩터링하면 코드가 하는 일을 깊이 파악하게 되면서 새로 깨달은 것들은 곧바로 코드에 반영하게 된다.

프로그램의 구조를 명확하게 다듬으면 불분명했던 기능들이 분명히 드러나며, 버그들이 명확히 보인다.

리팩터링은 견고한 코드를 작성하는 데 무척 효과적이다.

### 리팩터링하면 프로그래밍 속도를 높일 수 있다.

지금까지 제시한 장점을 한 마디로 정리하면 “코드 개발 속도를 높일 수 있다”이다.

얼핏 그 반대라고 생각할 수 있다.

내가 리팩터링에 대해 설명하면 사람들은 품질을 높일 수 있다는 점에는 대부분 쉽게 수긍한다.

내부 설계와 가독성이 개선되고 버그가 줄어든다는 점은 모두 품질 향상에 직결된다.

하지만 리팩터링하는 데 시간이 드니 전체 개발 속도는 떨어질까봐 걱정할 수도 있다.

내부 품질이 떨어지는 소프트웨어는 새로운 기능을 추가하는 작업이 오래걸린다.

게다가 기능을 추가하고 나면 버그가 쉽게 발생하고, 이를 해결하는 시간은 한층 더 걸린다.

이러한 부담이 기능 추가 속도를 계속 떨어뜨리면서, 차라리 처음부터 새로 개발하는게 낫겠다는 지경에 이른다.

반면 내부 설계가 잘 된 소프트웨어는 새로운 기능을 추가할 지점과 어떻게 고칠지를 쉽게 찾을 수 있다.

코드가 명확하면 버그를 만들 가능성도 줄고, 버그가 생겨도 디버깅하기 훨씬 쉽다.

나는 이 효과를 설계 지구력 가설(Design Stamina Hypothesis)라 부른다.

내부 설계가 견고하면 소프트웨어의 지구력이 높아져서 빠르게 개발할 수 있는 상태를 더 오래 지속할 수 있다.

허나 증명할 수는 없어서 ‘가설’이라고 표현했지만, 수많은 뛰어난 프로그래머들의 경험이 이를 뒷받침한다.

20년 전만해도 코딩을 시작하기 전에 설계부터 완벽히 마쳐야 한다는 것이 정설이었다.

한편, 리팩터링을 하면 이를 바로잡을 수 있다.

처음부터 좋은 설계를 마련하기란 매우 어렵다.

그래서 빠른 개발이라는 숭고한 목표를 달성하려면 리팩터링이 반드시 필요하다.

## 4. 언제 리팩터링해야 할까 ?

저자는 거의 한시간 간격으로 리팩터링한다. 그러다 보니 내 작업 흐름에 리팩터링을 녹이는 방법이 여러가지이다.

<aside>
💡 3의 법칙

</aside>

1. 처음에는 그냥 한다.
2. 비슷한 일을 두 번째로 하게 되면 중복이 생겼다는 사실에 당황스럽겠지만 일단 계속 진행한다.
3. 비슷한 일을 세 번째 하게 되면 리팩터링한다.

야구를 좋아하는 사람은 ‘스트라이크 세 번이면 리팩터링하라(삼진 리팩터링)’로 기억하자.

### 준비를 위한 리팩터링 : 기능을 쉽게 추가하게 만들기

리팩터링하기 가장 좋은 시점은 코드베이스에 기능을 새로 추가하기 직전이다.

구조를 살짝 바꾸면 다른 작업을 하기 훨신 쉬워질 만한 부분을 찾는다.

가령 내 요구사항을 거의 만족하지만 리터럴 값 몇 개가 방해되는 함수가 있을 수 있다.

함수를 복제해서 해당 값만 수정하면 되지만, 그러면 중복 코드가 생긴다.

이럴 때는 리팩터링 모자를 쓰고 **함수 매개변수화**하기를 적용한다.

그러고 나면 그 함수에 필요한 매개변수를 지정해서 호출하기만 하면 된다.

버그를 잡을 때도 마찬가지다.

오류를 일으키는 코드가 세 곳에 복제되어 퍼져 있다면, 우선 한 곳으로 합치는 편이 작업하기에 훨씬 편하다.

또는 질의 코드에 섞여 있는 갱신 로직을 분리하면 두 작업이 꼬여서 생기는 오류를 크게 줄일 수 있다.

이처럼 준비를 위한 리팩터링으로 상황을 개선해놓으면 버그가 수정된 상태가

오래 지속될 가능성을 높이는 동시에, 같은 곳에서 다른 버그가 발생할 가능성을 줄일 수도 있다.

### 이해를 위한 리팩터링 : 코드를 이해하기 쉽게 만들기

코드를 수정하려면 먼저 그 코드가 하는 일을 파악해야 한다.

나는 코드를 파악할 때마다 그 코드의 의도가 더 명확하게 드러나도록 리팩터링할 여지는 없는지 찾아본다.

조건부 로직의 구조가 이상하지 않은지,

함수 이름이 잘못되어 실제 하는 일을 파악하는데 오래걸리지 않는지 살펴본다.

워드 커닝햄이 말하길, 리팩터링하면 머리로 이해한 것을 코드에 옮겨 담을 수 있다.

그런 다음 수정한 코드를 테스트해보면 내 생각이 맞았는지 확인할 수 있다.

내가 이해한 것을 코드에 반영해두면 더 오래 보존하며, 다른 개발자들도 알 수 있다.

이해를 위한 리팩터링을 하다보면 나중은 물론 지금 당장 효과를 볼 때도 많다.

코드가 깔끔하게 정리되어 전에는 보이지 않던 설계가 눈에 들어오기 시작한다.

랄프 존슨(Ralph Johnson)은 이런 초기 단계의 리팩터링을 밖을 잘 내다보기 위한 창문 닦기에 비유한다.

코드를 분석할 때 리팩터링을 해보면, 그렇지 않았더라면 도달하지 못했을 더 깊은 수준까지 이해하게 된다.

### 쓰레기 줍기 리팩터링

코드를 파악하던 중에 일을 비효율적으로 처리하는 모습을 발견할 때가 있다.

로직이 쓸데없이 복잡하거나, 매개변수화한 함수 하나면 될 일을 거의 똑같은 함수 여러 개로 작성해놨을 수 있다.

이때 약간 절충을 해야 한다.

원래 하려던 작업과 관련 없는 일에 많은 시간을 뺏기긴 싫으며,

그렇다고 쓰레기가 나뒹굴게 방치해서 나중에 일을 방해하도록 내버려두는 것도 좋지 않다.

간단히 수정할 수 있는 것은 즉시 고치고, 시간이 걸리는 일은 짧은 메모만 남긴 후, 작업을 끝내고 처리한다.

이것이 이해를 위한 리팩터링의 변형인 쓰레기 줍기 리팩터링이다.

물론 수정하려면 오래 걸리거나 당장은 더 급한 일이 있을 수 있다.

캠핑 규칙이 제안하듯, 조금이나마 개선해두는 것이 좋다.

코드를 훑어볼 때마다 조금씩 개선하다 보면 결국 문제가 해결될 것이다.

리팩터링의 멋진 점은 잘게 나눈 단계들 덕에 몇 달에 걸쳐 진행되어도 코드가 깨지지 않는다는 것이다.

### 오래 걸리는 리팩터링

리팩터링은 대부분 몇 분, 길어야 몇 시간 정도지만, 팀 전체가 달려들어도 몇 주 걸리는 대규모 리팩터링도 있다.

나는 이런 상황에 처하더라도 팀 전체를 투입하는 것보다 주어진 문제를 몇 주에 걸쳐 조금씩 해결하는 편이다.

누구든지 리팩터링해야할 코드와 관련한 작업을 하게 될 때마다 원하는 방향으로 조금씩 개선하는 식이다.

일부를 변경해도 모든 기능이 항상 올바르게 작동한다.

예컨대 라이브러리를 교체할 때는 기존 것과 새 것 모두 포용하는 추상 인터페이스부터 마련한다.

기존 코드가 이 추상 인터페이스를 호출하게 만들고 나면 라이브러리를 훨씬 쉽게 교체할 수 있다.

(이 전략을 추상화 갈아타기[Branch By Abstraction]라 한다)

### 코드 리뷰에 리팩터링 활용하기

코드리뷰는 개발팀 전체에 지식을 전파하는데 좋다. 경험이 많은 개발자의 노하우를 전수할 수 있다.

대규모 소프트웨어 시스템의 다양한 측면을 더 많은 사람이 이해하는 데도 도움된다.

내눈에는 명확한 코드가 다른 팀원에게는 그렇지 않을 수 있다.

코드 리뷰를 하면 다른 사람의 좋은 아이디어를 많이 수집할 수 있다.

리팩터링은 다른 이의 코드를 리뷰하는 데도 도움된다.

새로운 아이디어가 떠오르면 리팩터링하여 쉽게 구현해넣을 수 있는지 살펴보고, 쉽다면 리팩터링한다.

이 과정을 몇 번 반복하면 내가 떠오른 아이디어를 실제로 적용했을 때의 모습을 구체적으로 볼 수 있다.

그러다 보면 리팩터링해보지 않고는 절대 떠올릴 수 없던 한 차원 높은 아이디어가 떠오르기도 한다.

리팩터링은 코드 리뷰의 결과를 더 구체적으로 도출하는 데에도 도움된다.

개선안들을 제시하는 데서 그치지 않고, 그중 상당수를 즉시 구현해볼 수 있기 때문이다.

코드리뷰에 리팩터링을 접목하는 구체적인 방법은 리뷰의 성격에 따라 다르다.

흔히 풀 요청 모델(코드 작성자 없이 검토하는 방식)에서는 그리 효과적이지 않다.

내가 경험한 가장 좋은 방법은 작성자와 나란히 앉아서 코드를 훑어가면서 리팩터링하는 것이다.

이렇게 하면 자연스럽게 프로그래밍 과정 안에 지속적인 코드 리뷰가 녹아있는 pair programming이 된다.

### 관리자에게는 뭐라고 말해야 할까 ?

관리자와 고객이 ‘리팩터링은 누적된 오류를 잡는 일이거나, 혹은 가치 있는 기능을 만들어내지 못하는 작업’

이라고 오해하여 리팩터링이 금어가 돼버린 조직도 있었다.

관리자가 기술에 정통하고 설계 지구력 가설도 잘 이해하고 있다면 리팩터링의 필요성을 쉽게 설득할 수 있다.

물론 기술을 모르는 상당수의 관리자와 고객에게는 “리팩터링 한다고 말하지 말라”고 조언하겠다.

일정을 최우선으로 여기는 관리자는 최대한 빨리 끝내는 방향으로 진행하기를 원한다.

그리고 구체적인 방법은 개발자가 판단해야 한다.

프로 개발자는 새로운 기능을 빠르게 구현해야 하며, 그 방법은 리팩터링이다. 그래서 리팩터링부터 한다.

### 리팩터링하지 말아야 할 때

1. 지저분한 코드라도 외부 API 다루듯 호출해서 쓰는 코드라면 지저분해도 그냥 둔다.
   1. 내부 동작을 이해해야 할 시점에 리팩터링해야 효과를 제대로 볼 수 있다.
2. 리팩터링하는 것보다 새로 작성하는게 쉬울 때도 리팩터링하지 않는다.
   1. 물론, 한 마디 조언으로 표현할만큼 간단하지 않으며 뛰어난 판단력과 경험이 뒷받침돼야 한다.

## 5. 리팩터링 시 고려할 문제

리팩터링이 많은 팀에서 적극적으로 도입해야 할 중요한 기법이라 믿지만,

리팩터링의 문제점도 있기에 이런 문제가 언제 발생하고 어떻게 대처해야 할지를 반드시 알고 있어야 한다.

### 새 기능 개발 속도 저하

리팩터링의 궁극적인 목적은 개발 속도를 높이는 데 있다.

하지만 리팩터링으로 인해 진행이 느려진다고 생각하는 사람이 여전히 많다.

아마도 이 점이 실전에서 리팩터링을 제대로 적용하는 데 가장 큰 걸림돌일 것이다.

<aside>
💡 **리팩터링의 궁극적인 목적은 개발속도를 높여서, 더 적은 노력으로 더 많은 가치를 창출하는 것이다.**

</aside>

그렇더라도 상황에 맞게 조율해야 한다.

대대적인 리팩터링이 필요해 보이지만, 추가하려는 기능이 아주 작아 추가부터 하고 싶은 상황이라면,

프로 개발자로서 가진 경험을 잘 발휘해서 결정한다.

준비를 위한 리팩터링을 하면 변경을 훨씬 쉽게 할 수 있다고 확신한다.

새 기능을 구현해넣기 편해지겠다 싶은 리팩터링이라면 주저하지 않고 리팩터링부터 한다.

한번 본 문제일 때도 리팩터링부터 하는 편이다.(물론 여러 차례 마주친 뒤에 할때도 있다)

반면 내가 직접 건드릴 일이 거의 없거나, 그리 심하게 불편하지 않다면 리팩터링하지 않는 편이다.

때로는 어떻게 개선해야 할지 확실히 떠오르지 않아서 리팩터링을 미루기도 한다.

아직도 리팩터링을 과도하게 하는 경우보다 거의 하지 않는 경우가 훨씬 많다.

건강한 코드의 위력을 경험해보지 않고서는 생산청의 차이를 체감하기 어렵다.

관리자 뿐만 아니라 개발자 스스로가 리팩터링이 개발 속도를 저하시키는 원인으로 삼는 경우도 보았다.

리팩터링을 할지 말지를 판단하는 능력은 수년에 걸친 경험을 통해서 서서히 형성된다.

리더는 리팩터링 경험이 부족한 이들이 이런 능력을 빠르게 갖추도록 개발 과정에서 이끌어줘야 한다.

하지만 리팩터링을 ‘클린 코드’나 ‘바람직한 엔지니어링 습관’처럼 도덕적인 이유로 정당화해서는 안된다.

리팩터링의 본질은 코드를 이쁘게 꾸미는데에 있지 않다.

오로지 경제적인 이유로 하는 것이다.

리팩터링은 기능 추가 시간을 줄이고, 버그 수정 시간을 줄여 개발 기간을 단축시킨다.

### 코드 소유권

리팩터링하다 보면 모듈의 내부뿐 아니라 시스템의 다른 부분과 연동하는 방식에도 영향을 주는 경우가 많다.

함수 이름을 바꾸고 싶고 그 함수가 호출하는 곳을 모두 찾을 수 있다면,

간단히 **함수 선언 바꾸기**로 선언 자체와 호출하는 곳 모두를 한 번에 바꿀 수 있다.

하지만 이렇게 간단하지 않을 때도 있다.

함수를 호출하는 코드의 소유자가 다른 팀이라서 쓰기 권한이 없을 수 있으며,

바꾸려는 함수가 고객에게 API로 제공된 것이라면 절대 변경할 수 없다.

이런 함수는 인터페이스를 누가 선언했는지에 관계없이 클라이언트가 사용하는 ‘공개된 인터페이스’에 속한다.

코드 소유권이 나뉘어 있으면 클라이언트에 영향을 주지 않고서는 원하는 형태로 변경할 수 없기에 방해가 된다.

그렇다고 할 수 없지는 않고, 제약이 생긴다.

함수 이름을 변경할 때는 기존 함수를 그대로 유지하되 함수 본문에서 새 함수를 호출하도록 수정한다.

하지만 인터페이스는 복잡해질 수 밖에 없으며, 이때문에 코드 소유권을 작은 단위로 나누는 것을 반대한다.

내가 선호하는 방식은 코드의 소유권을 팀단위로 두는 것이다.

프로그래머마다 각자 책임지는 영역이 있을 수는 있지만, 수정은 모두가 가능하게 말이다.

어떤 팀은 다른 팀이 자기 팀 코드의 브랜치를 따서 수정하고 커밋을 요청하는 흡사 오픈소스 개발 모델을 권장하기도 한다.

이렇게 하면 함수의 클라이언트도 바꿀 수 있으며, 이 방식은 위에서 설명한 두 방식의 절충안으로, 대규모 시스템 개발 시 잘 어울린다.

### 브랜치

흔히 팀원마다 브랜치를 따서 결과물이 쌓이면 마스터 브랜치에 통합하는 방식으로 작업을 한다.

이 방식을 선호하는 이들은 기능이 추가될 때마다 버전을 명확히 나눌 수 있고, 문제가 생길 시 롤백이 쉬워서이다.

하지만 이 방식은 독립 브랜치로 작업하는 기간이 길수록 마스터로 통합하기 어려워진다는 단점이 있다.

이를 해결하고자 수시로 ‘리베이스’ 혹은 ‘머지’ 하지만, 많은 브랜치에서 동시에 개발이 진행되면 해결할 수 없다.

여기서 머지와 통합의 뜻을 짚고 가겠다.

머지 : 마스터를 브랜치로 머지하는 단반향 작업(브랜치만 바뀌고 마스터는 유지)

통합 : 마스터를 개인 브랜치로 pull한 후 작업한 결과를 마스터에 push하는 양방향 작업(마스터, 브랜치 모두 변경)

즉, 어디선가 통합이 이루어지면 변경된 마스터와 내 브랜치를 머지해야하는데, 상당한 노력이 필요하다.

예를 들어, 특정 함수를 사용하는 코드가 통합되었을 때, 내 브랜치에서 그 함수의 이름을 변경하면 프로그램이 동작하지 않는다.

이처럼 머지가 복잡해지는 문제는 개발되는 기간이 길어질수록 기하급수적으로 늘어난다.

이 때문에 기능별 브랜치의 통합 주기를 2~3일 단위로 짧게 관리해야 한다고 주장하는 사람들이 많지만, 나는 더 짧아야 한다고 본다.

이 방식을 지속적 통합[CI](Continuous Integration) 또는 트렁크 기반 개발[TBD](Trunk-Based-Development)라 한다.

CI에 따르면 모든 팀원이 하루에 최소 한 번은 마스터와 통합한다.

하지만 CI를 적용하기 위해선 마스터를 건강하게 유지하고, 거대한 기능을 잘게 쪼개며,

각 기능을 끌 수 있는 기능 토글을 적용해야 시스템 전체가 망가지는 것을 막을 수 있다.

CI는 머지의 복잡도를 줄일 뿐 아니라 리팩터링과 궁합이 매우 좋다.

켄트 벡이 CI와 리팩터링을 합쳐서 eXtreme Programming(XP)을 만든 이유도 두 기법의 궁합이 잘 맞기 때문이다.

그렇다고 기능별 브랜치를 절대 사용하지 말라는 뜻은 아니고, 자주 통합할 수 있다면 사용해도 된다.

실제로 CI를 적용하는 이들도 기능별 브랜치를 많이 사용하며, 오픈 소스 프로젝트 또한 해당 방식이 적합하다.

하지만 풀타임 개발팀에게 해당 기법은 리팩터링 부담이 크며, CI를 완벽히 적용하진 못해도 통합 주기는 짧아야 한다.

CI를 적용하는 편이 소프트웨어를 배포하는 데 훨씬 효과적이라는 객관적 증거도 있다.

### 테스팅

리팩터링은 프로그램의 겉보기 동작이 똑같이 유지되며, 절차를 지켜 동작이 깨지지 않게 해야한다.

오류가 발생해도 원인이 될만한 코드 범위가 넓지 않다.

여기서 핵심은 오류를 재빨리 잡는 데 있다. 그에 필요한건 코드의 다양한 측면을 검사하는 테스트 슈트가 필요하다.

그리고 이를 빠르게 실행할 수 있어야 부담이 적으며 즉, 자가 테스트 코드를 마련해야 한다.

자가 테스트 코드는 구현할 때 어느정도 노력을 기울여야 하지만, 효과는 보장되어 있으며, 이를 이용하는 팀도 여럿 봤다.

테스트 코드는 아래와 같은 효능이 있다.

1. 리팩터링을 할 수 있게 해준다.
2. 새 기능 추가도 훨씬 안전하게 진행할 수 있다.
3. 실수로 만든 버그를 빠르게 찾아 제거할 수 있다.
   1. 테스트 실패 시 가장 최근에 통과한 버전에서 무엇이 달라졌는지 바로 파악 가능하다.
4. 리팩터링 과정에서 버그 발생률이 높다는 불안감을 해소할 수 있다.

뛰어난 자동 리팩터링 기능을 가진 IDE를 사용한다면 굳이 테스트를 안해도 오류가 없다고 확신할 수 있다.

물론 활용할 수 있는 리팩터링 기법 수가 제한되지만, 의미 있는 효과를 보기에 충분하다.

그래도 나는 갖춰두면 유용한 점이 많기 때문에 자가 테스트 코드를 마련하는 편이다.

이러한 생각 때문에 검증된 몇 가지 리팩터링 기법만 조합해서 사용하자는 흐름이 생겨나기도 했다.

실제로 테스트 커버리지가 넓지 않은 대규모 코드베이스도 효과적으로 리팩터링 할 수 있음을 확인했다.

자가 테스트 코드는 통합 과정에서 발생하는 충돌을 잡는 메커니즘으로 활용 가능해서 CI와도 밀접하게 연관된다.

CI에 통합된 테스트는 XP의 권장사항이자 지속적 배포[CD](Contimuous Delivery)의 핵심이기도 하다.

### 레거시 코드

물려받은 레거시 코드는 대체로 복잡하고 테스트도 제대로 갖춰지지 않은 것이 많다.

레거시 시스템을 파악할 때 리팩터링이 굉장히 도움되지만, 테스트가 없는 경우가 대다수다.

정답은 당연히 테스트 보강이지만, 테스트를 염두하고 설계한 시스템이 아니라면 쉽게 해결할 방법은 없다.

그나마 ‘레거시 코드 활용 전략’에 나온 말처럼 ‘프로그램에서 테스트를 추가할 틈새를 찾아서 시스템을 테스트 해야 한다’를 적용하는 것이 좋다.

이마저 상당히 어려운 작업이며, 내가 자가 테스트 코드를 작성해야 한다고 강조한 이유기도 하다.

테스트를 갖추고 있더라도 단번에 리팩터링하는 데는 낙관적이지 않다.

캠핑 규칙에 따라 서로 관련된 부분끼리 나눠서 하나씩 공략하자.

규모가 크다면 자주 보는 부분을 더 많이 리팩터링하자.

자주 본다는 말은 이해하기 쉽게 개선했을 때 얻는 효과도 크다는 뜻이다.

### 데이터베이스

책의 초판에서 데이터베이스 리팩터링은 어려운 영역이라 했지만, 내 동료가 개발한

진화형 데이터베이스 설계와 데이터베이스 리팩터링 기법은 현재 널리 적용되고 있다.

기법의 핵심은 데이터 마이그레이션 스크립트를 작성하고,

접근 코드와 스키마에 대한 구조적 변경을 이 스크립트로 처리하게끔 통합하는 데 있다.

간단한 예로 필드의 이름을 변경하는 경우, 선언과 호출부분을 모두 한 번에 수정해야 한다.

이때 변환을 수행하는 코드를 만들고,

데이터베이스를 다른 버전으로 이전할 때마다 모든 마이그레이션 스크립트를 실행한다.

다른 리팩터링과 마찬가지로 전체 변경 과정을 작고 독립된 단계로 쪼개는 것이 핵심이다.

여러 단계를 순차적으로 연결해서 데이터베이스 구조와 그 안에 담긴 데이터를 큰 폭으로 변경할 수 있다.

해당 리팩터링은 프로덕션 환경에 여러 단계로 나눠서 릴리스 하는 것이 대체로 좋은 점에서

다른 리팩터링과 다르며, 이렇게 하면 프로덕션 환경에서 문제가 생겼을 때 변경을 되돌리기 쉽다.

예시로 필드 이름을 바꿀 때 첫 번째 커밋에서는 새로운 필드만 추가하고 사용하지 않는다.

그 뒤 기존 필드와 새 필드를 동시에 업데이트하도록 설정한다.

그 다음에는 데이터베이스를 읽는 클라이언트들을 새 필드를 사용하는 버전으로 조금씩 교체한다.

이 과정에서 발생하는 버그도 해결하면서 교체 작업이 끝나면, 예전 필드를 삭제한다.

이런 방식은 병렬 수정(parallel change)[또는 팽창-수축(expend-contract)]의 일반적인 예다.

## 6. 리팩터링, 아키텍처, 애그니(YAGNI)

이전에는 소프트웨어 설계와 아키텍처는 코드가 작성된 후에는 바꿀 수 없으며, 부패할 일만 남았다고 여기곤 했다.

하지만 리팩터링은 기존 코드의 설계를 개선할 수 있다. (허나, 테스트 부재 및 레거시 코드는 어렵다)

리팩터링이 아키텍처에 미치는 효과는 요구사항 변화에 자연스럽게 대응하도록 코드베이스를 잘 설계해준다는 데 있다.

코딩 전 아키텍처를 확정지으려면 요구사항을 사전에 모두 파악해야 하는데, 사실 실현할 수 없는 목표일 때가 많다.

한 가지 방법으로 유연성 메커니즘(flexibility mechanism)을 소프트웨어에 심어두는 것이다.

가령 함수를 정의하다 보면 범용적으로 사용할 수 있다는 생각에 매개변수들을 추가하는데,

이런 매개변수가 바로 유연성 메커니즘이다.

하지만 매개변수를 마구잡이로 추가하다 보면 당장의 쓰임에 비해 함수가 너무 복잡해진다.

간혹 유연성 메커니즘을 잘못 구현할 때도 있으며, 오히려 변화에 대응하는 능력을 떨어뜨릴 때가 대부분이다.

유연성이 필요한 부분을 추측하지 않고, 현재까지 파악한 요구사항만 ‘완벽히’ 해결하도록 설계한다.

진행하면서 요구사항을 더 잘 이해하게 되면 아키텍처도 그에 맞게 리팩터링해서 바꾼다.

복잡도에 지장을 주지 않는 메커니즘은 마음껏 추가하지만, 그렇지 않다면 반드시 검증 후 추가한다.

호출하는 측에서 항상 같은 값인 매개변수는 추가하지 않는다.

추가할 시점이 오면 **함수 매개변수화하기**로 해결한다.

리팩터링을 미루면 훨씬 힘들어진다는 확신이 들 때만 유연성 메커니즘을 미리 추가한다.

이런 식으로 설계하는 방식을 간결한 설계, 점진적 설계, YAGNI 등으로 부른다.

**YAGNI : you aren’t going to neet it (필요 없을 거다)**

물론, YAGNI가 아키텍처를 전혀 고려하지 말라는 뜻은 아니다.

아키텍처와 설계를 개발 프로세스에 녹이는 또 다른 방식이며, 리팩터링의 뒷바침 없이는 효과를 볼 수 없다.

리팩터링으로는 변경하기 어려워 미리 생각해두어 시간이 절약되는 경우도 얼마든지 있다.

다만 균형점이 크게 달라졌으며, 나는 나중에 문제를 더 깊이 이해했을 때 처리하는 쪽이다.

이러한 경향은 진화형 아키텍처 원칙이 발전하는 계기가 됐다.

(진화형 아키텍처는 아키텍처 관련 결정을 시간을 두고 반복해 내릴 수 있따는 장점을 활용하는 패턴과 실천법을 추구한다.)

## 7. 리팩터링과 소프트웨어 개발 프로세스

실제로 리팩터링이 퍼지기 시작한 것은 익스트림 프로그래밍(XP)에 도입됐기 때문이다.

XP의 두드러진 특징은 지속적 통합, 자가 테스트코드, 리팩터링 등의

개성이 강하면서 상호 의존하는 기법들을 하나로 묶은 프로세스라는 점이다.

참고로 자가 테스트 코드와 리팩터링을 묶어서 TDD(테스트 주도 개발)이라 한다.

최초의 애자일 소프트웨어 방법론 중 하나였던 XP는 수년에 걸쳐 애자일의 부흥을 이끌었다.

상당수 프로젝트에서 애자일을 적용하고 있어서 애자일 사고가 주류로 자리 잡았다.

하지만 이름만 애자일인 경우가 대부분이다.

제대로 적용하려면 프로세스 전반에 리팩터링이 자연스럽게 스며들도록 해야 한다.

리팩터링의 첫 번째 토대는 자가 테스트 코드이며, 굉장히 중요하기 때문에 5장 전체가 테스트이다.

팀으로 개발할 때는 각 팀원이 다른 사람의 작업을 방해하지 않으면서 언제든지 리팩터링할 수 있어야 한다.

지속적 통합(CI)를 통해 이를 충원하며, 리팩터링 시 그로 인한 다른 팀원의 오류 발생을 즉시 알아낼 수 있다.

따라서 자가 테스트 코드, 지속적 통합, 리팩터링이라는 세 기법은 서로 강력한 상승효과를 발휘한다.

세 실천법을 적용한다면 YAGNI 설계 방식으로 개발을 진행할 수 있다.

추측에 근거한 수많은 유연성 메커니즘을 갖춘 시스템보단 단순한 시스템이 변경하기 훨씬 쉽다.

세 가지 실천법을 잘 조화시키면 변화에 재빠르게 대응하고 안정적인 선순환 구조를 만들 수 있다.

이러한 핵심 실천법을 갖췄다면 애자일의 다른 요소가 주는 이점까지 흡수할 수 있다.

1. 소프트웨어를 언제든 릴리스 가능한 상태로 만들어 준다.
2. 위험요소도 줄이고 비즈니스 요구에 맞춰 릴리스 일정을 계획할 수 있다.
3. 버그의 수를 줄여줘서 소프트웨어의 신뢰성도 높일 수 있다.

## 8. 리팩터링과 성능

나는 실제로 이해하기 쉽게 만들기 위해 속도가 느려지는 방향으로 수정하는 경우가 많다.

리팩터링하면 소프트웨어가 느려질 수도 있지만, 그와 동시에 성능을 튜닝하기는 더 쉬워진다.

빠른 소프트웨어를 작성하는 방법 세 가지를 경험했는데,

그중 가장 엄격한 방법은 시간 예산 분배(time budgeting) 방식이다.

이 방식에 따르면 설계를 여러 컴포넌트로 나눠서 컴포넌트마다 자원(시간과 공간) 예산을 할당한다.

컴포넌트는 할당된 예산을 초과할 수 없지만 주어진 자원을 서로 주고받는 메커니즘을 제공할 순 있다.

이 방식은 특히 엄격한 시간 엄수를 강조하지만, 사내 정보 시스템과 같은 부류에는 맞지 않는 기법이다.

두 번째 방법은 끊임없이 관심을 기울이는 것이다.

직관적이어서 흔히 사용하는 방식이지만, 성능을 개선하기 위해 수정하다 보면 코드는 어려워지고,

실제로 성능이 좋아지는 경우는 별로 없다.

이 방식은 성능 향상을 위한 최적화가 프로그램 전반에 퍼지는데,

정작 컴파일러와 런타임과 하드웨어 동작을 제대로 이해하지 못한 채 작성할 때도 많다.

이전 프로젝트에서 느린 프로그램의 성능 개선을 위해

여러 팀원들과 함께 잘 알고 있는 코드에서 잘못될 만한 부분이 없는지 집중적으로 살펴보았다.

심지어 개선안의 설계를 그려보기까지 했지만, 정작 원인은 다른 곳에 특정 코드에 있었다.

이 일의 교훈은 간단하다. 섣불리 추측하지 말고 성능을 측정해봐야 한다.

성능에 대한 흥미로운 사실은, 대부분 전체 코드 중 극히 일부에서 대부분의 시간을 소비한다는 것이다.

코드 전체를 고르게 최적화한다면 그중 90%는 효과가 거의 없다.

성능 개선을 위한 세 번째 방법은 이 ‘90%의 시간은 낭비’라는 통계에서 착안한 것이다.

성능 최적화에 돌입하기 전까지는 성능에 신경 쓰지 않고 코드를 다루기 쉽게 만드는 데 집중한다.

그러다 성능 최적화 단계가 되면 구체적인 절차에 따라 프로그램을 튜닝한다.

프로그램을 잘 리팩터링해두면 기능 추가가 빨리 끝나서 성능에 집중할 시간을 더 벌 수 있다.

또한 리팩터링이 잘 되어있는 프로그램은 성능을 더 세밀하게 분석할 수 있다.

리팩터링은 성능 좋은 소프트웨어를 만드는데 기여한다. 단기적으론 성능이 느려질 수 있지만,

최적화 단계에서 코드를 튜닝하기 훨씬 쉽기 때문에 결국 더 빠른 소프트웨어를 얻게 된다.

## 9. 리팩터링의 유래

‘리팩터링’이란 용어의 정확한 유래는 찾을 수 없었다.

실력 있는 프로그래머는 항상 자신의 코드를 정리하는데 어느 정도의 시간을 할애해왔다.

리팩터링은 소프트웨어 개발 프로세스 전반의 핵심 요소라고 주장한다.

아쉽게도 이전에는 리팩터링 전문가 중에서 이런 책을 쓰겠다는 사람은 없었다.

그래서 내가 그들의 도움을 받아서 이 책의 초판을 작성했으며,

다행히 리팩터링이란 개념을 업계에서 잘 받아들였다.

## 10. 리팩터링 자동화

리팩터링과 관련하여 일어난 가장 큰 변화는 자동 리팩터링을 지원하는 도구가 등장한 것이다.

자동 리팩터링 기능은 스몰토크용 ‘리팩터링 브라우저’에서 최초로 등장했다.

이후 인텔리제이, 이클립스, 비주얼 스투디오 등 여러 에디터가 지원한다.

자동 리팩터링을 제대로 구현하려면 코드를 텍스트 상태가 아닌, 구문 트리로 해석해서 다뤄야 한다.

그래야 코드의 원래 의미를 보존하는데 훨씬 유리하기 때문이다.

IDE는 구문 트리를 분석해서 리팩터링하기 때문에 단순한 텍스트 에디터보다 훨씬 유리하다.

정적 타입의 언어라면 훨씬 안전하게 구현할 수 있는 리팩터링 수가 늘어난다.

정적 타입 능력을 활용하여 메서드가 속한 클래스를 정확히 알아낼 수 있기 때문이다.

최근에는 언어 서버(Language Server)라는 기술이 뜨고 있다.

구문 트리를 구성해서 텍스트 에디터에 API 형태로 제공하는 소프트웨어다.

## 11. 더 알고 싶다면

이 책은 특정 리팩터링 기법이 궁금할 때 찾아볼 수 있는 레퍼런스를 제공하는 데 주력했다.

리팩터링 연습에 주력한 책을 원한다면 윌리엄 웨이크가 쓴 ‘리팩터링 워크북’을 추천한다.

‘패턴을 활용한 리팩터링’은 GoF의 디자인패턴에서 가장 핵심적인 패턴을 활용한 리팩터링 기법을 다룬다.

특정 분야에 특화된 리팩터링 책으로는 ‘리팩토링 데이터베이스’, ‘리팩토링 HTML’ 등이 있다.

최신 자료를 보고 싶다면 이 책의 깃허브 지원 페이지(https://github.com/WegraLee/Refactoring)와

리팩터링 웹 사이트(https://refactoring.com/)를 참고하기 바란다.
