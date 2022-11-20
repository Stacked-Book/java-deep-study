## 진행방식

- 매주 금요일 오후 10시 진행
- 진행자가 해당주차 주제에대해 미리 깊게 공부하고 꼬리질문
- 잘못된 정보 업로드 혹은 전달시 벌금 5천원(최대한 신뢰할수있는 자료를 근거로 공부해보기)
- 하나의 질문에대해 파고들었을때 막히는부분이 생기는주제에대해 대답못한사람중 사다리타기로 선정된사람이 정리하여 업로드(업로드 못했을시 벌금)

## 학습 내용

### 1주차

 1. 돈과관련된 연산을할때 float,double 자료형을 쓰지말아야할 이유에대해 설명해주세요.   
    - 컴퓨터가 실수를 표현하는방식(부동소수점, 고정소수점)
    - BigDecimal이 어떻게 정밀도를 보장하는지(내부 확인)

 2. static 이란 무엇인가요?
    - 메모리관점
    - 생성과 소멸시기
    - 주의점

 3. primitive type vs reference type의 차이점이 무엇인가요.
 
 4. hashcode()에 대해 설명해주세요.
     - 재정의 하는 이유
     - hashCode를 재정의해서 리턴값을 1로 고정한다면 생기는문제 (6번과 이어짐)
 
 5. call by value, call by reference
 
 ### 2주차
 
 6. HashMap에 대해서 설명해주시고, Hash Collision 발생 시 자바는 어떻게 대처하나요?
      - 충돌시 발생하는 최적화, 자료구조 , 시간복잡도
      - rehashing
      - rehashing의 문제점?
      - 보조해시함수
      - 보조해시함수는 왜 그렇게 만들어졌는지?
 
 7. String.length() 메서드의 정확성에 대한 이야기
 
 8. String ="a", new String("a")의 메모리 적재방식
       - 왜 적재 영역이 이동되었을까?
       - == vs equals() 으로 비교했을때 차이
       - String값은 위의 두가지방식 모두다 값을 비교할수있는데?
       - String값을 ==으로 비교하면 상수풀에 있는 리터럴들이 100% 같다고 나올까?
 
 9. String 클래스는 왜 final일까?
 
 10. 민감한정보를 String에 저장하는것과 char[] 또는 StringBuilder/StringBuffer 에 저장하는것의 차이
 
 ### 3주차
 
 11. Wrapper 클래스는 무엇이고 왜 만들었는지?
 
 12. 제네릭은 무엇이고 왜사용하나요?
     - 제네릭 타입엔 왜 primitive type을 사용할수없나요?
     - 소거(erasure)
     - 와일드카드 제네릭
 
 13. ArrayList, LinkedList의 차이
     - 언제 어떤것을 사용할건지? 왜?
     - ArrayList의 기본크기는?
     - ArrayList의 크기가 가득찼을때 데이터를 하나 추가한다면 무슨일이 벌어지는지

 14. HashMap vs HashTable vs ConcurrentHashMap
     - ConCurrentHashMap과 HashTable의 동기화 방식의 차이
     - fail-fast, fail-safe iterator
     
### 4주차
 
 15. 자바의 synchronized가 무엇인지 Reentrant Lock 와 차이는 무엇인지 말씀해주세요.
     - 어떤 상황에 무엇을 사용할건지?
 
 16. atomic Type과 CAS는 무엇이고 언제 사용되는 것인가요?
     - Atomic Type vs Reentrant Lock
     
 17. IO / NIO 차이   
     - 버퍼를 사용하면 뭐가좋은지? 왜사용하는지?   
     - 파일다운로드시 IO/NIO 선택

### 5주차

 18. JVM 동작방식
     - 컴파일러와 인터프리터
     
 19. GC의 종류, JDK별 default GC
     - 각 gc별 동작방식
     - gc별 주요 튜닝옵션
     
 20. mark and sweep , young and old
     - stw가 무엇이고 왜발생하는지? 발생시 생기는문제점
     - 서바이버 영역은 왜 2개인가?
     - 트래픽이 무지많이 몰리는 이벤트가 예정되어있다면 young, old 영역의 비율 어떻게 설정할것인가?
     
### 6주차
[JDK 8~13의 feature들을 읽고 각자 새로 알게되었던점, 기억에 남는점등 자유롭게 이야기하기](https://github.com/Stacked-Book/java-deep-study/blob/main/java/6%EC%A3%BC%EC%B0%A8/JDK%208~13.md)

### 7주차(完)
JDK 14~19의 feature들을 읽고 각자 새로 알게되었던점, 기억에 남는점등 자유롭게 이야기하기
   
## 참고자료
https://arc.dev/developer-blog/java-interview-questions/  
https://github.com/sirloin-dev/meatplatform  
https://github.com/Lob-dev/Junior-Back-end-Developer-Concepts
