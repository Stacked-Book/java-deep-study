## IO
- 초기 단계의 자바에서는 I/O를 처리하기 위해 `java.io` 패키지에 있는 클래스를 제공했다.
- 바이트 기반의 데이터를 처리하기 위해 여러 종류의 `Stream`이라는 클래스를 제공한다.
- char 기반의 문자열로만 되어있는 파일은 Reader/Writer를 사용한다.

<br>

### 기존 IO의 문제점

**Java I/O PROCESS**

![image](https://user-images.githubusercontent.com/80027033/199868601-84a80a6d-5fec-4d19-8880-de537ad2dc3e.png)
- 프로세스가 파일을 읽기 위해 커널 명령을 전달한다.
- 커널은 시스템 콜(read)을 사용한다.
- 디스크 컨트롤러가 물리적 디스크로부터 파일을 읽어온다.
- DMA에 저장된 데이터를 다시 커널 버퍼 메모리 영역에 복사한다.
- CPU 인터럽트가 일어나면 CPU는 커널 버퍼 메모리 영역에 복사된 데이터를 프로세스 내부 버퍼 메모리에 복사한다.

<br>

**문제점**

- 커널에서 JVM 내부로 데이터를 복사할 때 CPU 연산을 계속해서 사용하게 되고 이렇게 복사된 데이터들은 GC 대상이 되므로 계속해서 오버헤드가 발생하게 된다. 
- 커널 영역에서 데이터를 복사하는 동안에 프로세스 영역이 Blocking 되면서 전체 처리 속도가 느려진다.

<br>

## NIO(New IO)
- JDK 1.4 부터 도입되어 기존 IO의 **속도** 문제를 해결했다.
- Java 7부터 IO와 NIO 사이의 일관성 없는 클래스 관계를 바로 잡고 비동기 채널 등 네트워크 지원을 강화한 `NIO.2 API`가 추가되었다.
- 스트림 대신 `Channel` 과 `Buffer` 를 사용한다.


```java
public void writeFile(String fileName, String data) throws Exception {
    FileChannel channel = new FileOutputStream(fileName).getChannel();
    byte[] byteData = data.getBytes();
    ByteBuffer buffer = ByteBuffer.wrap(byteData);
    channel.write(buffer);
    channel.close();
}
```

<br>

### IO와 NIO의 차이점을 통해 알아보는 NIO가 IO를 개선한 방법
| 구분 | IO | NIO |
| --- | --- | --- |
| 입출력 방식 | 스트림 | 채널 |
| 버퍼 방식 | 넌버퍼(Non-Buffer) | 버퍼(Buffer) |
| 비동기 방식 | 지원 안 함 | 지원 |
| 블로킹 / 넌블로킹 방식 | 블로킹 방식만 지원 | 블로킹 / 넌블로킹 방식 지원 |

<br>

**1. Channel**

IO에서는 데이터를 읽기 위해서 입력 스트림을 생성하고 출력하기 위해 출력 스트림을 생성해야 했지만 채널에서는 입력과 출력을 위해 별도의 채널을 만들 필요가 없다.
NIO는 채널 기반으로 스트림과 달리 양방향으로 입력과 출력이 가능하며 쓰레드와 버퍼 사이에서 일종의 터널 역할을 해서 입출력 기능을 수행한다.

<br>

**2. Buffer**

기존 IO에서도 Buffer로 한꺼번에 입력받고 출력하기 위해 BufferdInputStream과 같은 확장 클래스를 사용하기도 했는데
NIO에서는 기본적으로 버퍼를 사용해서 입출력을 하기 때문에 효율적으로 처리가 가능하고 데이터의 위치를 이동하면서 필요한 부분을 읽고 쓸 수 있다.

<br>

**3. Blocking / Non-Blocking**

IO는 쓰레드가 Blocking이 되면 인터럽트도 발생시킬 수 없고 빠져나오기 위해서는 스트림을 닫아야만 한다.
하지만 NIO는 Blocking 시 인터럽트로 빠져나올 수 있고, Non-Blocking으로 입출력 작업 준비가 완료된 채널만 선택할 수 있도록 지원해서
작업 중인 쓰레드가 Blocking 되지 않게 한다. 이때 사용하는 것이 `Selector` 이다.

<br>

**Selector**
![image](https://user-images.githubusercontent.com/80027033/199872523-180e3a57-478c-4bcb-b83a-e13fe0c3edde.png)
네트워크 프로그래밍의 효율을 높이기 위해 도입된 것으로 한 개의 쓰레드로 다수의 채널을 처리할 수 있는 기술이다.


Selector에 여러 채널이 등록되면 쓰레드는 셀렉터에 등록된 채널 중에 가용한 채널이 있는지 알 수 있고 이런 방식을 `멀티플렉싱(Multiplexing)`이라고 한다.


<br>

## IO와 NIO를 선택하는 기준
- 비동기적으로 처리할 수 있는 작업이 아니라면 NIO를 사용했을 때 오히려 Non-Blocking 때문에 생기는 추가 동작으로 속도만 더 느려질 수 있다.
- 대용량의 데이터를 처리할 때는 NIO로 모든 입출력에 Buffer를 사용하기 보다는, IO를 선택함으로써 스레드의 개수를 늘려 Blocking으로 처리해야 메모리 낭비를 줄일 수 있고 오버헤드를 줄일 수 있다.


<br>

## 질문
- NIO가 IO의 어떤점을 개선한 것인지? 왜 개선했는지?
> 기존 IO의 속도와 Blocking으로 인한 성능 저하를 개선하기 위해서 여러가지 방식을 도입했습니다.
> 하나의 채널로 다수의 입출력 작업을 수행할 수 있고 버퍼를 사용하여 속도를 개선했으며,
> Non-Blocking 방식을 지원하면서 Selector를 이용하여 가용한 채널에 접근할 수 있습니다.

<br>

- 버퍼를 쓰는 이유와 장점
> 시간을 절약할 수 있고 데이터의 필요한 부분을 읽거나 쓸 수 있기 때문입니다. 데이터를 버퍼 공간에 담아놨다가 공간이 가득차면 데이터를 저장하기 때문에
> 사용하지 않을 때보다 시간을 절약할 수 있고 Buffer의 position을 이용해서 위치를 파악하면 필요한 부분의 데이터를 읽거나 쓸 수 있습니다.
    
<br>

- 파일 다운로드 시 IO/NIO 중 선택할 방식과 이유
> IO를 선택하겠습니다. 파일 다운로드 시에는 데이터를 읽어야만 저장할 수 있으므로 비동기적으로 처리해야 할 작업이 필요하지 않기 때문에 Non-Blocking으로 인한 추가 작업을 수행하지 않고 리소스를 절약하며 기능을 수행할 수 있을 것 같습니다.