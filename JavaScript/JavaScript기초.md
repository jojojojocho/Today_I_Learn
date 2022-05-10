## JavaScrpt란? 
HTML과 CSS로 이루어진 정적인 페이지를 동적으로 바꾸어주는 언어라고 볼 수 있을 것 같다.

백엔드를 공부하면서 프론트도 한번 쯤은 경험해 보는 것도 좋을 것 같아서 자바스크립트 공부를 시작하게 되었다. (프레임워크도 하나정도는 경험해 볼 예정)

이전에 자바스크립트는 그냥 var 변수와 fuction선언해서 돌아가고, 굉장히 러프한 언어라고 생각했다.

자바와 달리 각 타입을 strict하게 설정해 줄 필요도 없고, 그냥 var 하나면 오류 나지 않고 동작 했기 때문에, 머릿속에 굉장히 특이한 언어다. 라고 생각하고 있었던 것 같다.

근데 최근에 나온 Vanila.js 는 더이상 var문법을 권장하지 않는 모양이다.

### 새로나온 const 와 let

대신에 const, let 이 나오게 되었다.

const는 변하지 않는 상수를 선언할 때 쓰이고, let은 변할 수 있는 변수 선언에 사용된다.

대부분 const 선언을 하고 필요시 let을 사용하는 것이 권장되고, var은 현재 권장 되지 않는다.

### fuction의 사용법

fuction의 사용법도 자바와는 조금 다르다.

```java
const 변수이름 = {
  함수이름 : fuction() {
  실행문...
  }
}
```
이런 식으로 변수안에 선언 하기도 하고,
```java
fuction 함수이름(){
  실행문...
}
```
이렇게 선언하기도 한다.

### css selector를 사용하여 페이지 자원에 접근

그외에 파이썬으로 크롤링할 때 많이 사용했었던 css selector를 이용하여 페이지안의 데이터들을 조작하기도 한다.
```java
document.querySelector("selector 작성...")
```
을 사용한다면 굉장히 쉽게 페이지 내의 태그에 접근할 수 있다.

### element.addEventListener을 통한 동적 웹페이지 구성

자바스크립트는 어떠한 이벤트가 발생하면 함수를 작동하게 할 수 있도록 EventListener를 제공해주고 있다.

button, a 등과 같은 태그(element)를 선언 후 이벤트리스너를 추가해주면 된다. 사용방법은 다음과 같다.

```java
h1.addEventListener("click", clickHandler);
button.addEventListener("mouseenter", mouseEnterHandler);
button.addEventListener("mouseleave", mouseLeaveHandler );

```

또한 웹브라우저의 변화를 감지하고 그에 대한 액션을 취할 수도 있다. 이럴 경우에는 window 엘리먼트를 이용한다.

```java
window.addEventListener("resize", windowResizeHandler);
window.addEventListener("copy", windowCopyHandler);
window.addEventListener("offline", windowOfflineHandler);
window.addEventListener("online", windowOnlineHandler);
```
