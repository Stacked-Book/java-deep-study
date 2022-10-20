## `java.lang.String` 의 `length` 메소드는 **정확한** 결과를 반환하지 않는 경우가 종종 있습니다. **정확한** 의 의미란 무엇이고, 왜 그럴까요?

실제로 왜그런지 파악하기 위해서 내부적인 코드를 보고 문제가 생길 수 있는 상황을 추측해봤습니다. 자바 버전 9를 기준으로 작성했습니다.

### String이 상속받는 클래스는 무엇일까

- Serializable을 구현해 직렬화가 가능하다.
- Comparable을 구현해 정렬이 가능하다. String 내부에 compareTo를 구현하고 있다. 인코딩이 달라도 비교가 가능하다.
- CharSequence는 문자열을 나타내며, String은 이 인터페이스를 구현했다고 보면된다. 

> 즉, String은 CharSequence을 구현함과 동시에 Serializable와 Comparable 기능들을 제공하고 있다.

### String이 들고 있는 인자는 무엇일까

1. `byte[] value`

String은 value라는 바이트 배열을 가지고 있는데, character 담는 배열이라 생각하며 된다.

> 처음 한 번만 할당되는 필드는 @Stable 애너테이션을 달 수 있다. 필드의 값은 해당 구성 요소 값으로 계산된다. 확장에 의해, 안정적이라고 주석이 달린 변수(배열 또는 필드 중 하나)는 stable variable 라고 불리고, Null이 아닌 값 또는 0이 아닌 값을 stable value이라고 부른다.

JDK 6 버전부터는 `char` 배열을 담았는데, 압축할 수 있는 상황에서는 압축할 수 있도록 `byte` 배열로 수정됐다.

2. `byte coder`

값의 바이트를 인코딩하는 데 사용되는 인코딩의 식별자며, 지원되는 값은 LATAN1, UTF16 두 개 뿐이다.

> LATIN-1은 1 바이트만 할당되고, UTF16은 2 바이트씩 할당된다.

3. `int hash`

문자열의 hash code를 캐싱하기 위한 방법이다.

4. `serialVersionUID`

SerialiVersionUID를 통해 직렬화가 가능한 지를 사전에 파악한다.

5. `boolean COMPACT_STRINGS`

**_문자열_** **을 생성할 때마다 _문자열_ 의 모든 문자가 바이트(LATIN-1 표현)를 사용하여 표시될 수 있는 경우 바이트 배열이** 내부적으로 사용되어 한 문자에 대해 1바이트가 제공된다는 것을 의미한다.

> 즉, UTF16을 사용하더라도 LATIN1로 압축이 되어 있을 가능성이 있기 때문에 coder 필드를 확인하기 전, COMPACT_STRINGS을 명시적으로 검사한다.

그렇기에 문자열 압축이 비활성화된 경우 값의 바이트는 항상 UTF16으로 인코딩된다. 

> `COMPACT_STRINGS`의 실제 값은 JVM에 의해 주입된다.

그럼 LATIN-1과 UTF-16 표현을 구분하는 방법은 coder에 의해 값을 찾게 된다.

6. `ObjectStreamField[] serialPersistentFields =  new ObjectStreamField[0]`

### Immutable한 String의 특징
#### Immutable하기에 가능한 String 메서드
1. hashCode 캐싱
	1. 내부 값이 stable하기에 캐싱해서 사용할 수 있다.
2. 원자성을 보존하지 않는 length
	1. 길이를 구하는 length 메서드는 동기화 문제가 발생할 수 있지만, 내부 값이 stable하기 때문에 안전하게 사용할 수 있다.

#### String 리터럴과 String 객체
String 리터럴은 메모리를 효율적으로 사용하는 방법이고, String 객체로 문자열을 생성하는 방식은 효율적이지 못하다.

#### 어떻게 메모리를 효율적으로 관리하는가
`intern`  메서드를 통해 동일한 문자열이 있는 경우 동일한 문자열의 주소를 참조하게 된다.

#### 문자열 연산에서는 문자 리터럴은 효율적이지 못하다.
불변 객체를 서로 더하려면 새로운 객체를 만들어야 하기 때문에 메모리를 효율적으로 사용할 수 없다. 그래서 연산자가 있는 경우 자바 내부에서 StringBuilder를 통해 append 하는 방식으로 객체를 새로 생성하지 않고 기존 문자열에 더하게 된다.


### 그럼 문제에 대해 고민을 해보겠습니다.

문제의 키워드는 총 두 개입니다.

1. 정확한은 어떤 의미에서인지
2. 왜 그런 상황이 발생하는지

자바 9에서는 아래와 같이 구현되어 있고, byte 배열로 값을 표현하고 있다. LATIN-1은 1 바이트만 할당되어 바이트 배열의 크기가 곧 길이지만, UTF16은 2 바이트씩 할당되어있기 때문에 바이트 배열의 절반이 길이가 된다.

```java
public int length() { 
	return value.length >> coder; 
}
```

구현 메서드를 봤을 때, 런타임에서 length 메서드의 연산이 문제가 되지 않았습니다. 그러면, 문자열을 생성하는 시점에서 발생한다고 가정할 수 있습니다.

입력받는 문자열이 UTF-8이라면 String에서는 UTF-8을 지원하지 않기 때문에 원하는 문자열이 나오지 않게 되고, 문자열 길이에 문제가 생기는게 아닐까 추측해볼 수 있습니다.

```java
String rawString = "Entwickeln Sie mit Vergnügen"; 

byte[] bytes = rawString.getBytes(StandardCharsets.UTF_8); 

String utf8EncodedString = new String(bytes, StandardCharsets.UTF_8);
```

인코딩 종류에 따라 문자 하나의 크기를 1 바이트에서 2 바이트로 산정할 수 있습니다. 각 바이트에 맞게 문자열의 크기를 고려해야하는데, 그렇지 않은 경우 문제가 발생할 수 있습니다.

### 결론

결론은 인코딩에 따라 문자 하나의 바이트 크기가 달라지는데, 잘못된 인코딩으로 문자열의 길이를 구하는 경우 원하지 않는 결과값이 나올 수 있습니다. 또한  1바이트로 압축된 문자열을 2바이트로 계산하거나 2바이트 문자열을 압축된 문자열로 착각해 계산하는 방식에서 문제가 발생할 수 있습니다.
