# BigDecimal

float과 double은 그 값의 정확성을 보장할 수 없으므로 정확한 계산이 요구될 때는 `java.math.BigDecimal` 클래스를 사용해야한다. 

BigDecimal 은 변경할 수 없으며 (= immutable 하며) 임의의 정밀도 와 부호를 가진 십진수이다.

- BigDecimal 은  불변 객체이기 때문에 연산 결과는 기존의 객체의 값을 변화시키지 않고, 새로운 객체를 반환한다.
- BigDecimal는 다음과 같이 표현이 가능하다.
- unscaledValue × 10^-scale
  - unscaledValue : 임의의 유효자리 정수 값
  - scale : 소수점 오른쪽의 자릿수를 나타내는 32비트 정수
- 예를 들어 BigDecimal 3.14는 unscaledValue 314, scale 2이다.

BigDecimal의 내부를 살펴보자.

```
public class BigDecimal extends Number implements Comparable<BigDecimal> {

    private final BigInteger intVal;
    private final int scale;
    private transient int precision;
    ...
    private final transient long intCompact;
    private static final int MAX_COMPACT_DIGITS = 18;
    ...
}
```

- intVal : 정수로 표현된 BigDecimal의 값 (=  unscaledValue )
- scale : 0 또는 양수인 경우 소수점 오른쪽의 자리 수, 음수일 경우 스케일 되지 않은 숫자 값에 10을 스케일 부정의 거듭 제곱
- precision : unscaledValue의 자리 수
- stringCache
- intCompact : BigDecimal의 길이("." 포함)가 18자리 이하일 땐, intVal에 값을 따로 저장하지 않고 intCompact에 정수 값을 저장

```
public BigDecimal(String val) {
    this(val.toCharArray(), 0, val.length());
}

public BigDecimal(char[] in, int offset, int len) {
    this(in,offset,len,MathContext.UNLIMITED);
}

public BigDecimal(char[] in, int offset, int len, MathContext mc) {
    ...
    boolean isCompact = (len <= MAX_COMPACT_DIGITS);
    if (isCompact) {
        /* rs에 관한 연산 */
    } else {
        /* rb에 관한 연산 */
    }
    ...
    this.scale = scl;
    this.precision = prec;
    this.intCompact = rs;
    this.intVal = rb;
}

```

#### java.math.MathContext

- 반올림 모드와 정밀도(precision)을 하나로 묶어 놓은 클래스이다.

#### BigDecimal 주의할 점

1. **이중 생성자**
   ```
   BigDecimal x = new BigDecimal(0.1);
   System.out.println("x=" + x);

   BigDecimal y = new BigDecimal("0.1");
   System.out.println("y=" + y);

   ### 결과 ###
   x=0.10000000000000000555111512312578270...
   y=0.1
   ```
   - double 타입의 값을 그대로 전달할 경우 이진수의 근사치를 가지게 되어 예상과 다른 값을 얻을 수 있다.
   - double 타입으로 부터 BigDecimal 타입을 초기화하는 방법으로 가장 안전한 것은 문자열의 형태로 생성자에 전달하여 초기화하는 것이다.

2. **static valueOf(double) 메소드**
    ```
    BigDecimal x = BigDecimal.valueOf(1.01234567890123456789);
    BigDecimal y = new BigDecimal("1.01234567890123456789");
    System.out.println("x=" + x);
    System.out.println("y=" + y);

    ### 결과 ###
    x=1.0123456789012346
    y=1.01234567890123456789
    ```
    - x의 결과값을 보면 4자리의 십진수를 잃어버린 것을 확인할 수 있다. 이는 double의 정밀도에 제한이 있기 때문이다.

3. **equals(bigDecimal) 메소드**
    ```
    BigDecimal x = new BigDecimal("1");
    BigDecimal y = new BigDecimal("1.0");
    System.out.println("equlse() : " + x.equals(y));
    System.out.println("compareTo() : " + x.compareTo(y) == 0);

    ### 결과 ###
    equlse() : False
    compareTo() : True
    ```
    - 이러한 결과가 나온 이유는 BigDecimal은 unScaledValue인 정수 값과 32비트 정수 scale로 구성이 되어있기 때문이다.
      - x : unscaled value = 1 / scale = 0
      - y : unscaled value = 10 / scale = 1
    - BigDecimal의 두 인스턴스를 비교하기 위해서는 equals() 메소드 대신 compareTo() 메소드를 활용한다. compareTo()는 숫자값을 비교하기 때문이다.
