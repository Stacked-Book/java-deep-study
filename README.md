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
  

## 참고자료
https://arc.dev/developer-blog/java-interview-questions/  
https://github.com/sirloin-dev/meatplatform  
https://github.com/Lob-dev/Junior-Back-end-Developer-Concepts
