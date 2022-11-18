# 민감한 정보를 String으로 저장하는것과 char[]혹은 StringBuilder/StringBuffer에 저장하는 것의 차이

비밀번호 및 기타 민감한 데이터를 저장하는데에는 String보다 char[] 배열을 선호한다. 그 이유에 대해서 알아보자.

### String은 변경할 수 없으며 문자열 풀에 캐시된다.

<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbqJLP0%2FbtqCD88wyuU%2FuNBx00Hfxib994SZYkOgqk%2Fimg.png" width=300>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F72QbD%2FbtqCF7VlYcY%2FaO3NPmM5Cw5CwoIyIRok71%2Fimg.png" width=277>

- String에서 저장되는 문자열은 private final byte[]의 형태로 Immutable(불변)하기 때문에 한번 생성되면 할당된 메모리 공간이 변하지 않는다.
- 따라서 String의 이전 객체에 대한 변경은 새로운 String를 생성 하고 이전 것을 메모리에 유지하며 이전 정보는 Garbage Collector가 이를 지울 때까지 메모리에서 사용할 수 있다.
- 결과적으로 메모리 덤프에 액세스할 수 있는 사람은 누구나 메모리에 남아있는 정보를 검색할 수 있다.
- String 대신 **char[] 배열을 사용 하면** 의도한 작업을 마친 후 데이터를 명시적으로 지울 수 있다. 이렇게 하면 가비지 수집이 발생하기 전에도 정보를 메모리에서 제거할 수 있다.

### String은 실수로 로그로 출력될 가능성이 높다.

```
Object pw = "Password";
System.out.println("String: " + pw);

pw = "Password".toCharArray();
System.out.println("Array: " + pw);

## 결과
String: Password
Array: [C@5829428e
```

- char[]는 참조형 변수이므로 객체의 주솟값이 저장이 되며 char[] 타입 변수를 출력하면 배열의 주소가 출력된다.
- 하지만 String 타입 변수를 출력하면 마찬가지로 참조형 변수이지만 다른 참조형 변수들과 다르게
배열의 주소가 아니라 배열에 들어있는 문자들이 출력된다.
- https://www.oracle.com/java/technologies/javase/seccodeguide.html#2 에 따르면 민감한 정보는 로그 파일로 보내서는 안되는데 String의 경우 실수로 로그에 출력될 가능성이 더 높아진다.

#### StringBuilder와 StringBuffer는 민감한 정보를 저장해도 될까?

StringBuilder와 StringBuffer의 생성 과정과 delete 메소드 호출 시 동작 과정을 살펴보자.

```
public AbstractStringBuilder delete(int start, int end) {

    byte[] value;
    ...

    AbstractStringBuilder(String str) {
        int length = str.length();
        int capacity = (length < Integer.MAX_VALUE - 16)
                ? length + 16 : Integer.MAX_VALUE;
        final byte initCoder = str.coder();
        coder = initCoder;
        value = (initCoder == LATIN1)
                ? new byte[capacity] : StringUTF16.newBytesFor(capacity);
        append(str);
    }

    public AbstractStringBuilder delete(int start, int end) {
        int count = this.count;
        if (end > count) {
            end = count;
        }
        Preconditions.checkFromToIndex(start, end, count, Preconditions.SIOOBE_FORMATTER);
        int len = end - start;
        if (len > 0) {
            shift(end, -len);
            this.count = count - len;
        }
        return this;
    }

    private void shift(int offset, int n) {
        System.arraycopy(value, offset << coder,
                         value, (offset + n) << coder, (count - offset) << coder);
    }

}
```

- StringBuilder와 StringBuffer 클래스는 AbstractStringBuilder를 상속받으며 문자열 저장을 위해 byte형 배열의 참조 변수를 인스턴스 변수로 선언한다.
- StringBuilder와 StringBuffer의 인스턴스가 생성될 때 byte형 배열이 생성되며 이 때 생성된 byte형 배열을 인스턴스 변수 value가 참조하게 된다.
- 생성된 StringBuilder와 StringBuffer의 부분 문자열을 제거하는 delete 함수를 호출했을 경우 System.arrayCopy()를 호출하여 제거한 후 값을 복사한다.