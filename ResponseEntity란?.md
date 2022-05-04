Spring Framework 에서는 HttpEntity 클래스를 제공한다.

이 HttpEntity 클래스를 상속받아 만들어진 클래스가 RequestEntity와 ResponseEntity이다.

아마 Request와 Response를 쉽게 사용할 수 있도록 만들어진 클래스 인듯 하다.

ResponseEntity에는 body, header, httpstatus를 담을 수 있도록 설계되어 있다.

ResponseEntity는 bulider 패턴을 통해 status().body() 형식으로 HttpStatus, 그리고 body 부분에는 객체(응답 할 데이터를 넣은)를 넣어서 전송이 가능하다.

