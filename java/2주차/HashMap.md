# Map

- Map 인터페이스는 Collection 인터페이스와는 다른 저장 방식을 가진다.
- 자바는 `java.util.Map<K,V>` 인터페이스를 제공하며 키/값 모두 반드시 참조형이어야한다.

#### Map<K,V> 인터페이스

- 키를 값에 매핑하는 객체이며 Map에는 중복 키가 포함될 수 없다.
- 키-값 쌍을 자바 7까지는 Entry 클래스를 사용하여 표현했다면 8부터는 Node 클래스를 사용한다.
- 그 구현체로는 HashMap, TreeMap, Hashtable, SortedMap, Collection, Set 등이 있다.

### HashMap

- **hashing**
  - 객체 데이터를 대표적인 정수 값으로 매핑하는 알고리즘 인 해싱의 원리에 따라 작동한다.
  - 키-값 쌍을 저장, 검색하기 위해 버킷의 인덱스를 계산하기 위해 키 객체에 해싱 함수가 적용된다.
- **bucket**
  - HashMap은 내부적으로 bucket을 사용하는데 이 때 요소가 어떤 bucket에 들어가게 될지는 그 요소의 hashCode()값에 의해 결정된다.
- **동기화**
  - 동기화되지 않는다. 여러 스레드가 해시 맵에 동시에 액세스하고 스레드 중 하나 이상이 구조적으로 맵을 수정하는 경우 외부에서 동기화 해야한다.
    - Map m = Collections.synchronizedMap(new HashMap(...));
- **순서**
  - 맵의 순서를 보장하지 않는다.

#### 1-1 HashMap 주요 필드

```
transient Node<K,V>[] table;
    ...
int threshold;
final float loadFactor;
```

- transient Node<K,V>[] table
  - transient로 선언된 이유는 직렬화(serialize)할 때 전체, table 배열 자체를 직렬화하는 것보다 키-값 쌍을 차례로 기록하는 것이 더 효율적이기 때문이다.
- threshold
  - 대략적으로 현재 용량과 부하율의 곱을 의미하며 이 값을 초과하면 Map이 다시 해시하여 
- loadFactor
  - Map 의 용량을 늘릴 시기를 결정하는 척도이다.
  
#### 1-2 HashMap 생성자

```
public HashMap(int initialCapacity, float loadFactor) {
    ...
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

- loadFactor를 매개변수로 주지 않을 경우 DEFAULT_LOAD_FACTOR(0.75)를 주입된다.

#### 1-3 HashMap put 메소드

- 초기화된 HashMap에 put 메소드를 호출하면 내부에서 어떤 동작이 발생하는지 살펴보자.

```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    ...
}

```

- `put` 메소드 호출 시 hash(key) 메소드를 먼저 호출한다.
- `hash(key)` 메소드에서는 해당 key의 Object class에 있는 hashcode() method를 실행하고 리턴 값인 int를 얻어낸 후 16번 오른쪽으로 unsigned right shift한 값과 XOR을 하여 사용할 hash값을 얻는다.
- `putVal(...)` 메소드에서는 해당 키의 해시코드, 키, 값 등등을 통해 node를 생성해 값을 저장한다.
  - 같은 키를 넣으면 원래 HashMap에 있던 키를 치환한다.
  - 이 때 하나의 버킷에 TREEIFY_THRESHOLD에 설정한 개수만큼 키/값 쌍이 모이면 버킷을 TreeNode로 바꾼다. (처음부터 바꾸지않는이유는 TreeNode는 리스트 노드보다 약 2배 커서 그만큼 공간을 더 차지하기 때문이다.)
  - 또한 threshold와 HashMap의 크기를 비교하여 재해시를 할 지 결정한다.

#### 1-3 HashMap 충돌

- 동일하지 않은 어떤 객체 X와 Y가 있을 때, 즉 X.equals(Y)가 '거짓'일 때 X.hashCode() != Y.hashCode()가 같지 않다면, 이때 사용하는 해시 함수는 완전한 해시 함수(perfect hash functions)라고 한다
- HashMap은 기본적으로 각 객체의 hashCode() 메서드가 반환하는 값을 사용하는 데, 결과 자료형은 int다. 32비트 정수 자료형으로는 완전한 자료 해시 함수를 만들 수 없다.
- 이는 해시 함수가 얼마나 해시 충돌을 회피하도록 잘 구현되었느냐에 상관없이 발생할 수 있는 또 다른 종류의 해시 충돌이다.
- 이렇게 해시 충돌이 발생하더라도 키-값 쌍 데이터를 잘 저장하고 조회할 수 있게 하는 방식에는 대표적으로 두 가지가 있는데, 하나는 **Open Addressing**이고, 다른 하나는 **Separate Chaining**이다.

  <img src="https://d2.naver.com/content/images/2015/06/helloworld-831311-4.png" width=600>

  - **Open Addressing**은 데이터를 삽입하려는 해시 버킷이 이미 사용 중인 경우 다른 해시 버킷에 해당 데이터를 삽입하는 방식이다. 
  - **Separate Chaining**에서 각 배열의 인자는 인덱스가 같은 해시 버킷을 연결한 링크드 리스트의 첫 부분(head)이다. 이때 **자바 8 이후부터는 데이터의 개수가 많아지면, Separate Chaining에서 링크드 리스트 대신 트리를 사용**한다.
- Java HashMap에서 사용하는 방식은 Separate Channing이다.

#### 1-3-1 Java 8 HashMap에서의 Separate Chaining

```
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    ...
    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
        treeifyBin(tab, hash);
}
```

- 링크드 리스트를 사용할 것인가 트리를 사용할 것인가에 대한 기준은 하나의 해시 버킷에 할당된 키-값 쌍의 개수이다.
- Java 8 HashMap에서는 상수 형태로 기준을 정하고 있다.
- 즉 하나의 해시 버킷에 8개의 키-값 쌍이 모이면 링크드 리스트를 트리로 변경한다. 만약 해당 버킷에 있는 데이터를 삭제하여 개수가 6개에 이르면 다시 링크드 리스트로 변경한다.

또한 Java 8 HashMap에서는 **Entry 클래스 대신 Node 클래스를 사용**한다.

Node 클래스 자체는 사실상 Java 7의 Entry 클래스와 내용이 같지만, 링크드 리스트 대신 **트리를 사용할 수 있도록 하위 클래스인 TreeNode가 있다**는 것이 Java 7 HashMap과 다르다.

이때 사용하는 트리는 **Red-Black Tree**이다. 트리 순회 시 사용하는 대소 판단 기준은 해시 함수 값이다. 해시 값을 대소 판단 기준으로 사용하면 Total Ordering에 문제가 생기는데, Java 8 HashMap에서는 이를 `tieBreakOrder()` 메서드로 해결한다
> Red-Black Tree는 기본 이진 트리구조에 메타데이터를 부가(노드 컬러링)해서 트리 균형이 한쪽으로 치우치는 현상을 방지한 트리이다.

또한 put 메서드 구현 시 해시 버킷의 인덱스를 가져오는 것이 개선되었다.

#### 1-3-2 해시 버킷 동적 확장

해시 버킷 개수의 기본값은 16이고, 데이터의 개수가 임계점에 이를 때마다 해시 버킷 개수의 크기를 두 배씩 증가시킨다.
버킷의 최대 개수는 230개다. 그런데 이렇게 버킷 개수가 두 배로 증가할 때마다, 모든 키-값 데이터를 읽어 새로운 Separate Chaining을 구성해야 하는 문제가 있다.

```
void resize(int newCapacity) {
    ...
}
```

#### 1-3-2 보조해시함수

- 보조 해시 함수(supplement hash function)의 목적은 '키'의 해시 값을 변형하여, 해시 충돌 가능성을 줄이는 것이다.
- Java 8에서는 이전 자바 버전보다 훨씬 더 단순한 형태의 보조 해시 함수를 사용한다.
  - 상위 16비트 값을 XOR 연산하는 매우 단순한 형태의 보조 해시 함수를 사용한다.

```
static final int hash(Object key) { int h; return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); }
```
