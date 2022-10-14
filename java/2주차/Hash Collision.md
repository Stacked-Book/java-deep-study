## 예상 질문
### 1. 해시 충돌이란 무엇인가요?

해시함수가 서로 다른 두 값을 입력받았을 때 동일한 출력 값을 내는 상황을 말합니다.

<br>

### 2. 해시 충돌이 발생하는 이유는 무엇인가요?

해시 알고리즘은 고정된 길이의 결과 값을 반환하는데 입력 값의 개수가 고정된 길이로 가질 수 있는 결과 값의 개수보다 많아지면 어쩔 수 없이 동일한 값이 발생하게 됩니다.

<br>

### 3. 해시 충돌이 일어날 때 해결할 수 있는 방법은 무엇인가요?

첫 번째로 `Chaining`을 이용해서 동일한 해시값이 출력되면 그 위치에 있던 데이터 값과 포인터로 연결해서 데이터들을 선형 구조로 만들거나, 두 번째로 `Open Addressing` 을 이용해서 충돌이 발생했을 때 다른 빈 자리를 찾아서 데이터를 저장할 수 있습니다.

<br>

### 4. 두 방법 중 하나를 선택하는 기준은 무엇인가요?

어떤 작업을 수행해야 하느냐에 따라 다른데 삭제 작업을 많이 수행해야 하는 경우에는 Chaining을 선택합니다. Open Addressing은 삭제한 후에 재배열이 필요하기 때문입니다.


<br>

## 들어가기 전에

### 해시 충돌(Hash Collision)이란

해시함수가 서로 다른 두 값을 입력받았을 때 동일한 출력 값을 내는 상황을 말한다.

<br>

### 해시 충돌이 발생하는 이유
해시 알고리즘은 고정된 길이의 결과 값을 반환하는데 입력 값의 개수가 고정된 길이로 가질 수 있는 결과 값의 개수보다 많아지면 어쩔 수 없이 동일한 값이 발생하게 된다.

<br>

1. **비둘기집 원리**

    비둘기가 5마리일 때 상자가 4개밖에 존재하지 않는다면 아무리 비둘기를 균등하게 배분하더라도 최소한 한 상자에는 2마리의 비둘기가 들어가게 된다는 원리로
    이 원리에 따라 해시 충돌이 발생할 수 있다.

<br>

2. **Birthday Paradox**
   
    ![birthday](http://wiki.hash.kr/images/5/56/Birthday_paradox.png)
    
    366명의 사람이 모였을 때, 생일이 겹치는 사람이 최소 2명 이상이 된다는 것으로 모든 경우의 수를 넘어가는 값이 주어지면 중복 값이 발생할 수 밖에 없다는 수학적 원리를 설명한 것이다.

<br>

## 충돌 최적화

### 1. Chaining(=Separate Chaining)

충돌이 발생했을 때 기존 값과 새로운 값을 LinkedList로 연결하는 방식
![chaining](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d0/Hash_table_5_0_1_1_1_1_1_LL.svg/1920px-Hash_table_5_0_1_1_1_1_1_LL.svg.png)

**시나리오**

❶ Object A의 해싱 결과 값이 1이 나와서 해당 위치의 버킷에 데이터를 저장한다.

❷ Object B의 해싱 결과 값도 1이 나와 해시 충돌이 발생한다. 

❸ 해당 위치의 버킷에 있던 Object A의 데이터에 Object B의 데이터를 연결한다. 

<br>

즉, 최초로 저장된 데이터 이후의 값들은 Single LinkedList 형태가 되는 것이다.

<br>

**장점**
- 구현이 간단하다.
- 데이터를 얼마나 자주 삽입하거나 삭제할지 알 수 없을 때도 사용 가능하다.

**단점**
- 충돌이 많아지면 조회/삽입/수정이 O(1)이 아닌 O(N)이 된다.
- 해시 테이블의 일부는 사용되지 않기 때문에 공간이 낭비된다.

<br>

### 2. Open Address
충돌이 발생했을 때 비어있는 공간에 데이터를 저장하는 방식
![open_address](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/Hash_table_5_0_1_1_1_1_0_SP.svg/760px-Hash_table_5_0_1_1_1_1_0_SP.svg.png)

❶ 선형 탐색(Linear Probing)

```text
rehash(key) = (n+1)%table-size
```
hash(x)가 해시함수로 계산된 인덱스 값이고 S가 테이블 크기라면 hash(x) % S가 가득차면 (hash(x)+1) % S를 시도한다.
- 해당 인덱스가 아닌 그 다음으로 하나씩 이동하며 비어있는 인덱스를 찾으면 데이터를 저장한다.
- 단점
  - 특정 위치에 데이터가 밀집하는 현상(primary clustering)이 발생한다.
  - 슬롯이 많아질수록 탐색하는데 시간 소요가 길어진다.

<br>

❷ 제곱 탐색(Quadratic Probing)
- 해당 인덱스부터 시작해서 충돌횟수 n의 제곱칸만큼 이동하며 비어있는 인덱스를 찾으면 데이터를 저장한다.
- 단점
  - 처음 인덱스 값이 같으면 이후의 해시값들이 동일하기 때문에 충돌이 반복적으로 발생(secondary clustering)한다.

<br>

❸ 이중 해시(Double Hash)
- 제곱 탐색의 규칙성을 제거해서 clustering을 방지하기 위해 두 개의 해시함수를 사용하는 방식이다.
- 첫 번째 해시함수는 최초의 해시 값을 얻고, 두 번째 해시함수로 해시 충돌이 일어났을 때 탐사 이동 폭을 얻는다.

<br>

### 3. Chaining vs Open Address
|  | Chaining | Open Address |
| --- | --- | --- |
| 구현 | 쉬움 | 계산이 더 필요함 |
| 해시 테이블 공간 | 모두 채워지지 않음(공간 낭비) | 채워질 수 있음(입력이 매핑되지 않더라도 슬롯을 사용할 수 있음) |
| 추가 공간 사용 여부 | 사용 | 사용하지 않음 |
| 민감한 요소 | 해시 함수 또는 load factor에 덜 민감 | clustering과 load factor를 피하기 위해 주의 |
| 사용  | 키가 얼마나 많이 삽입되거나 삭제되는지 알 수 없을 때 주로 사용 | 키의 빈도와 수를 알고 있을 때 사용 |


<br>

## 참고
- [해시충돌](http://wiki.hash.kr/index.php/%ED%95%B4%EC%8B%9C%EC%B6%A9%EB%8F%8C)
- [Hash table](https://en.wikipedia.org/wiki/Hash_table)
- [Hasing | Set 3(Open Addressing)](https://www.geeksforgeeks.org/hashing-set-3-open-addressing/)