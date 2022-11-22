## Heap Space in Java
![memory structure](https://k.kakaocdn.net/dn/bjJAy6/btrNYuyvBYY/kVHl2yypgTh6zMekcK6YiK/img.jpg)

GC가 설계될 때 고려해야 하는 두 가지의 가설이 있다. 

첫 번째는 대부분의 객체는 금방 unreachable 상태가 된다는 것이고 두 번째는 오래된 객체에서 새로운 객체를 참조하는 경우는 드물다는 것이다.
이러한 두 조건을 `Weak Generational Hypothesis`라고 한다. 

두 가설에 따르면 객체는 수명에 따른 그룹화를 할 수 있으며 실제로 Heap Memory는 `Young Generation`과 `Old Generation`으로 나뉜다.
**두 영역으로 나눔으로써 GC를 자주 수행해야 하는지에 따라 효율적인 메모리 관리를 할 수 있으며 GC 수행 시 모든 영역을 스캔하지 않도록 하여 수행 시간을 줄일 수 있다.**

<br>

### Young Generation 
모든 새 객체가 할당되고 Aging되는 곳으로 가득 차면 `Minor GC`가 수행된다.
- `Eden` : 새로운 객체가 할당되는 공간
- `Survivor1 / Survivor2` : Eden 영역에서 수행된 GC에 의해 살아남은 객체들이 이동되는 공간

<br>

**✔️ Survivor**

Survivor 영역에서는 `Mark-and-Copy(Copying) 알고리즘`을 사용한다. 
이 알고리즘은 기본적으로 객체를 덮어쓰지 않고 복사가 이루어지기 때문에 더 빠르고 효율적인 작업이 가능한데 이때 별도의 두 공간(Survivor1, Survivor 2)이 필요하다. 
하나는 활성 상태인 공간이고 다른 하나는 비활성 상태인 공간으로 GC가 수행되고 난 후에 두 공간의 역할이 전환된다. 

Copy 작업 시 Compact를 수행한 것 같이 메모리를 압축하기 때문에 `메모리 단편화` 문제를 해결할 수 있으며 Eden 영역에서는 이 두 영역을 둠으로써 새로운 객체만을 위한 공간을 유지할 수 있다.

<br>


### Old Generation(Tenured Generation)
- 여러 차례의 Minor GC 이후에도 살아남은 객체가 저장되는 곳으로 Young Generation에서 설정된 임계값을 넘어선 Age 값을 가진 객체가 이 곳으로 이동된다.
- Young Generation 영역보다 크게 할당되어 상대적으로 GC가 적게 발생하나 시간이 오래 걸린다.
- `Major GC(Full GC)`가 수행된다.

<br>

### Promotion process
1. 새로운 객체는 Eden에 할당된다.
2. Eden이 가득 차면 GC가 수행된다.
3. GC에서 살아남은 객체는 Survival 1으로 이동한다.
4. Survival 1이 가득차면 GC가 수행된다.
5. GC에서 살아남은 객체는 Age 값이 증가하며 Survival 2로 이동한다.
6. Survival 1은 비활성화되고 Survival 2가 활성화된다.
7. Survival 2가 가득차면 GC가 수행된다.
8. GC에서 살아남은 객체는 Age 값이 증가하며 Survival 1으로 이동한다.
9. Survival 2는 비활성화되고 Survival 1이 활성화된다.
10. 위 내용이 반복되어 계속해서 살아남은 객체는 Age 값이 증가한다.
11. Age 값이 MaxTenuringThreshold 설정을 초과한 객체는 old Generation 영역으로 옮겨진다.

<br>

**✔️ Age**

Survivor 영역에서 객체가 살아남은 횟수를 의미하며 Age 값이 임계값에 다다르면 Promotion 여부가 결정된다.
HotSpot JVM의 경우 기본 임계값은 31이며 객체 헤더에 Age를 기록하는 부분은 6bit이다.

<br>

## GC Algorithm

- Reference Counting Algorithm
  - 객체마다 참조 횟수를 관리하여 0이 되면 GC를 수행하는 알고리즘
- Mark-and-Sweep Algorithm
  - 살아남아야 할 객체에 Marking을 하고 Marking되지 않은 객체는 지우는 알고리즘
- Mark-and-Compact Algorithm
  - Mark-and-Sweep 알고리즘 방식 이후에 남은 빈자리에 객체들을 적재하여 분산된 메모리를 압축하는 알고리즘
- Copying Algorithm
  - 메모리 단편화를 방지하기 위한 알고리즘
- Generational Algorithm
  - Weak Generational Hypothesis를 바탕으로 만들어진 알고리즘

<br>

### Mark-and-Sweep Algorithm
- `Mark` : 먼저 Root Space로부터 그래프 순회를 통해 연결된 객체들을 찾아내서 사용 중인 메모리와 사용하지 않는 메모리를 식별한다.
- `Sweep` : 참조되지 않는 객체를 제거하고 빈 공간에 대한 포인터를 남긴다.

<br>

## STW(Stop-The-World)
`Stop-The-World`란 GC Thread가 실행 중일 때 다른 Thread가 중지되면서 결론적으로 애플리케이션 실행이 중단되는 것을 말한다.
그래서 STW 시간을 줄이기 위한 GC 튜닝이 필요하며 이를 통한 JVM 최적화로 STW 시간이 허용할 수 있는 최소한의 시간으로 발생하게 만들어야 한다.

<br>

## 질문
- STW(Stop-The-World)가 발생하는 이유와 발생 시 문제점
> GC가 수행될 때 GC를 수행하는 Thread를 제외한 나머지 Thread가 작업을 중지하기 때문에 발생합니다.
> 만약 서비스가 중단된다면 사용자가 요청한 작업들이 Time Out이 발생하게 되고 이 작업들로 또 다른 Full GC가 발생한다면
> 장애가 장시간으로 지속될 수 있습니다.


<br>

- Survivor 영역이 2개인 이유
> 메모리 단편화 문제를 해결하면서 Eden 공간을 새로운 객체만을 위한 영역으로 두어 메모리를 효율적으로 관리하기 위해서입니다.
> Eden 영역에서 살아남은 객체들은 계속해서 GC가 수행되면서 메모리 단편화 문제가 생기고 이 문제를 효율적으로 처리하기 위해 Copying 알고리즘을 사용하는데
> 이때 두 개의 별도의 공간이 필요합니다.


<br>

- 트래픽이 몰리는 이벤트가 예정되어 있을 시 Young Gen / Old Gen 의 비율을 어떻게 설정할 것인가?
> 
> [GC 튜닝](https://velog.io/@jsj3282/19.-GC-%ED%8A%9C%EB%8B%9D%EC%9D%84-%ED%95%AD%EC%83%81-%ED%95%A0-%ED%95%84%EC%9A%94%EB%8A%94-%EC%97%86%EB%8B%A4)


