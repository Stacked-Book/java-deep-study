## ArrayList

### capacity
- ArrayList의 **기본 용량(DEFAULT_CAPACITY)** 은 `10`으로 설정되어있다.
- **자바 6**까지의 용량 증가 계산식

  (oldCapacity * 3) / 2 + 1
- **자바 7** 이후의 용량 증가 계산식

  oldCapacity + (oldCapacity >> 1)
- 계산식이 바뀐 이유는 계산 중에 발생할 수 있는 Overflow를 방지하기 위함이다.
- **최대 용량(MAX_ARRAY_SIZE)** 은 `Integer.MAX_VALUE - 8`로 지정되어있다.
  - 더 큰 용량을 할당하려고 했을 때 만날 수 있는 에러
  
    ```OutOfMemoryError: Requested array size exceeds VM limit```

<br>

### add()

```java
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
```
size와 ArrayList의 크기를 비교해서 같다면 grow()가 진행된다.

<br>

### grow()

```java
private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                       newCapacity(minCapacity));
}

private Object[] grow() {
    return grow(size + 1);
}
```
- 매개 변수로 최소 용량 값을 받았을 때, 기존 배열을 최소 용량으로 계산한 크기로 새로 만들어진 배열에 복사한다.
- 매개 변수가 없을 때, 기존 size에 1을 더한 값을 최소 용량 값으로 넘겨준다.

<br>

### newCapacity(int minCapacity)
```java
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}
```
- 새로운 용량을 계산하는 공식은 `oldCapacity + (oldCapacity >> 1)`로 기존 용량의 1.5배 값이다.
- 만약 새로운 용량이 기존 용량 + 1인 값보다 작을 때 빈 배열의 상태라면 최소 용량을 기본 값인 10으로 지정한다.
- 새로운 용량이 MAX_ARRAY_SIZE보다 클 때 hugeCapacity에 최소 용량 값을 넘겨준다.

<br>

### hugeCapacity(int minCapacity)
```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE)
        ? Integer.MAX_VALUE
        : MAX_ARRAY_SIZE;
}
```
최소 용량이 MAX_ARRAY_SIZE를 넘어갈 경우 새로운 용량으로 Integer.MAX_VALUE 값을 반환하고 그게 아니라면 MAX_ARRAY_SIZE를 반환한다.

<br>

> 즉, 용량이 가득 찼을 경우 기존 용량의 1.5배로 값을 늘린 새로운 배열에 기존 배열을 복사합니다.
> 
> 만약 새로운 용량이 최대 용량(MAX_ARRAY_SIZE)보다 클 때는 최소 용량(기존 용량 + 1)을 기준으로 다시 새로운 용량을 계산하는데
> 
> 최소 용량이 최대 용량보다 클 경우 새로운 용량은 Integer.MAX_VALUE 값이 되고, 작을 경우 최대 용량을 할당합니다.