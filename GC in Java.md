# GC in Java
## 1. GC(Garbage Collection) 이란?
- 프로그램을 개발하다 보면 유효하지 않은 메모리인 가비지(Garbage)가 발생하게 된다.
- C언어를 사용한다면 free()라는 함수를 통해 직접 메모리를 해제해주어야 하지만 Java나 Kotlin을 이용해 개발을 한다면 개발자가 메모리를 직접 해제해주는 일이 없다.
- 그 이유는 JVM의 가비지 컬렉터가 불필요한 메모리를 알아서 정리해주기 때문이다.

```java
Person person = new Person();
person.setName("choi");
person = null;

// 가비지 발생
person = new Person();
person.setName("lee");
```

- 기존의 choi로 생성된 person 객체는 더 이상 참조를 하지 않기 때문에 가비지가 되었다.
- Java나 Kotlin은 이러한 메모리 누수를 방지하기 위해 가비지 컬렉터(GC)가 주기적으로 검사하여 메모리를 청소해준다.


### Major GC와 Minor GC
- JVM의 Heap 영역은 처음 설계될 때 다음의 2가지를 전제로 설계되었다.
```
대부분의 객체는 금방 접근 불가능한 상태가 된다.
오래된 객체에서 새로운 객체로의 접근은 아주 적게 존재한다.
```

- 즉, 객체는 대부분 일회성이고, 메모리에 오랫동안 남아있는 경우는 드물다는 것이다. 그렇기 때문에 객체의 생존 기간에 따라 물리적인 Heap 영역을 나누게 되었고 Young, Old 총 2가지 영역으로 설계되었다.

<p align="center"><img src="./images/gc_flow.png" width="600"></p>

#### Young 영역
- 새롭게 생성된 객체가 할당(Allocation)되는 영역
- 대부분의 객체가 금방 Unreachable 상태가 되기 때문에 많은 객체가 이 Young 영역에 생성되었다가 사라진다.
- Young 영역에 대한 가비지 컬렉션(GC)를 Minor GC라고 부른다.

#### Old 영역
- Young 영역에서 Reachable 상태를 유지하여 살아남은 객체가 복사되는 영역
- Young 영역보다 크게 할당되며, 영역의 크기가 큰 만큼 가비지는 적게 발생
- Old 영역에 대한 가비지 컬렉션을 Major GC 또는 Full GC라고 부른다.

---
- Old 영역이 Young 영역보다 크게 할당되는 이유는 Young 영역의 수명이 짧은 객체들은 큰 공간을 필요로 하지 않으며 큰 객체들은 Young 영역이 아니라 바로 Old 영역에 할당되기 때문이다.
- 예외적인 상황으로 Old 영역에 이는 객체가 Young 영역의 객체를 참조하는 경우도 있을 수 있다.
- 이러한 경우를 대비하여 Old 영역에는 512 bytes의 덩어리(Chunk)로 되어있는 카드 테이블(Card Table)이 존재한다.

### 카드 테이블(Card Table)
<p align="center"><img src="./images/card_table.png" width="600"></p>

- 카드 테이블에는 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 그에 대한 정보가 표시된다.
- 이 카드 테이블이 도입된 이유는 Young 영역에서 가비지 컬렉션이 실행될 때 모든 Old 영역에 존재하는 객체를 검사하여 참조되는 Young 영역의 객체가 어떤것인지 식별하는 과정은 매우 비효율적이기 때문이다.
- 대신, Young 영역에서 GC가 진행될 때 카드 테이블만 조회하여 GC 대상인지 아닌지를 식별할 수 있다.


## 2. 가비지 컬렉션의 동작 방식
### 과정
- Young 영역과 Old 영역은 서로 다른 메모리 구조로 되어있기 때문에 세부적인 동작 방식은 다르다.
- 하지만 기본적으로 가비지 컬렉션이 동작한다고 하면 다음의 2가지 공통적인 단계를 따른다.
1. Stop The World
2. Mark and Sweep

#### 1) Stop The World
- 가비지 컬렉션을 실행하기 위해 JVM이 애플리케이션의 실행을 멈추는 작업이다.
- GC가 실행될 때는 GC를 실행하는 쓰레드를 제외한 모든 쓰레드들의 작업이 중단되고, GC가 완료되면 작업이 재개된다.
- 당연히 모든 쓰레드들의 작업이 중단되면 애플리케이션이 멈추기 때문에 GC의 성능 개선을 위한 튜닝을 한다고 하면 보통 이 stop-the-world의 시간을 줄이는 작업을 가리킨다.
- 또한 JVM에서도 이러한 문제를 해결하기 위해 다양한 실행 옵션을 제공한다.

#### 2) Mark and Sweep
- Stop The World를 통해 모든 작업을 중단시키면, GC는 스택의 모든 변수 또는 Reachable 객체를 스캔하면서 각각이 어떤 객체를 참조하는지를 탐색한다.
- 그리고 사용되고 있는 메모리를 식별하는데, 이 과정을 Mark라고 한다. 이후에 Mark가 되지 않은 객체들을 메모리에서 제거하는데 이 과정을 Sweep이라고 한다.

### Minor GC 동작 방식
- Minor GC를 정확히 이해하기 위해서는 Young 영역의 구조에 대해 이해해야 한다. Young 영역은 1개의 Eden 영역과 2개의 Survivor 영역, 총 3가지로 나뉜다.
#### Young 영역 구조
- Eden 영역 : 새로 생성된 객체가 할당(Allocation)되는 영역
- Survivor 영역 : 최소 1번의 GC 이상 살아남은 객체가 존재하는 영역
- 객체가 새롭게 생성되면, Young 영역 중에서도 Eden 영역에 할당(Allocation)된다.
- 그리고 Eden 영역이 꽉 차면 Minor GC가 발생하게 되는데, 사용되지 않는 메모리는 해제되고 Eden 영역에 존재하는 객체는 (사용중인) Survivor 영역으로 옮겨지게 된다.
- Survivor 영역은 총 2개이지만 반드시 1개의 영역에만 데이터가 존재해야 한다.

#### 동작 순서
1. 새로 생성된 객체가 Eden 영역에 할당된다.
2. 객체가 새로 생성되어 Eden 영역이 꽉 차게 되고, Minor GC가 실행된다.
    - Eden 영역에서 사용되지 않는 객체의 메모리가 해제된다.
    - Eden 영역에서 살아남은 객체는 1개의 Survivor 영역으로 이동된다.
3. 1~2번의 과정이 반복되다가 Survivor 영역이 가득 차게 되면 Survivor 영역의 살아남은 객체를 다른 Survivor 영역으로 이동시킨다. (1개의 Survivor 영역은 반드시 빈 상태가 된다.)
4. 이러한 과정을 반복하여 계속해서 살아남은 객체는 Old 영역으로 이동(Promotion) 된다.

#### 생존 횟수 카운트
- 객체의 생존 횟수를 카운트하기 위해 Minor GC에서 객체가 살아남은 횟수를 의미하는 age를 Object Header에 기록한다.
- 그리고 Minor GC 때 Object Header에 기록된 age를 보고 Promotion 여부를 결정한다.

#### Eden 영역에 객체를 빠르게 할당하기 위한 기술
##### bump the pointer
-  Eden 영역에 마지막으로 할당된 객체의 주소를 캐싱해두는 것이다.
-  bump the pointer를 통해 새로운 객체를 위해 유효한 메모리를 탐색할 필요 없이 마지막 주소의 다음 주소를 사용하게 함으로써 속도를 높인다.
-  이를 통해 새로운 객체를 할당할 때 객체의 크기가 Eden 영역에 적합한지만 판별하면 되므로 빠르게 메모리 할당을 할 수 있다.
- 싱글 쓰레드 환경이라면 문제없지만 멀티 쓰레드 환경이라면 객체를 Eden 영역에 할당할 때 락(Lock)을 걸어 동기화를 해주어야 한다.
- 이러한 멀티 쓰레드 환경에서의 성능 문제를 해결하기 위해 HotSpot JVM은 추가로 TLABs라는 기술을 도입하게 되었다. 
##### TLABs
- 각각의 쓰레드마다 Eden 영역에 객체를 할당하기 위한 주소를 부여함으로써 동기화 작업 없이 빠르게 메모리를 할당하도록 하는 기술이다.
- 각각의 쓰레드는 자신이 갖는 주소에만 객체를 할당함으로써 동기화 없이 bump the pointer를 통해 빠르게 객체를 할당하도록 한다.

### Major GC 동작 방식
- Young 영역에서 오래 살아남은 객체는 Old 영역으로 Promotion됨을 확인할 수 있었다. 그리고 Major GC는 객체들이 계속 Promotion되어 Old 영역의 메모리가 부족해지면 발생하게 된다.
- Young 영역은 일반적으로 Old 영역보다 크기가 작기 때문에 GC가 보통 0.5초에서 1초 사이에 끝난다. 그렇기 때문에 Minor GC는 애플리케이션에 크게 영향을 주지 않는다.
- 하지만 Old 영역은 Young 영역보다 크며 Young 영역을 참조할 수도 있다. 그렇기 때문에 Major GC는 일반적으로 Minor GC보다 시간이 오래 걸리며 10배 이상의 시간이 소모된다.


## 3. 다양한 가비지 컬렉션 알고리즘
### 개요
- JVM이 메모리를 자동으로 관리해주는 것은 개발자 입장에서 상당한 메리트이다.
- 하지만 문제는 GC를 수행하기 위해 __Stop The World에 의해 애플리케이션이 중지__ 되는 것에 있다.
- Heap의 사이즈가 커지면서 애플리케이션의 지연(Suspend) 현상이 두드러지게 되었고, 이를 막기 위해 다양한 가비지 컬렉션 알고리즘을 지원하고 있다.

### Serial GC
- Young 영역에서는 앞서 설명한 알고리즘(Mark Sweep)대로 수행된다.
- 하지만 Old 영역에서는 Mark Sweep Compact 알고리즘이 사용되는데, 기존의 Mark Sweep에 Compact라는 작업이 추가되었다.
- Compact는 Heap 영역을 정리하기 위한 단계로 객체들이 연속되게 쌓이도록 힙의 가장 앞 부분부터 채워서 객체가 존재하는 부분과 존재하지 않는 부분으로 나누는 것이다.
```java
java -XX:+UseSerialGC -jar Application.java
```
- Serial GC는 서버의 CPU 코어가 1개일 때 사용하기 위해 개발되었으며, 모든 가비지 컬렉션 일을 처리하기 위해 1개의 쓰레드만을 이용한다.
- 그렇기 때문에 CPU의 코어가 여러 개인 운영 서버에서 Serial GC를 사용하는 것은 반드시 피해야 한다.

### Parallel GC
- 기본적인 처리 과정은 Serial GC와 동일하나 여러 개의 쓰레드를 통해 Parallel하게 GC를 수행함으로써 GC의 오버헤드를 상당히 줄여준다.
- Parallel GC는 멀티 프로세서 또는 멀티 쓰레드 머신에서 중간 규모부터 대규모의 데이터를 처리하는 애플리케이션을 위해 고안되었으며, 옵션을 통해 애플리케이션의 최대 지연 시간 또는 GC를 수행할 쓰레드의 개수 등을 설정해줄 수 있다.
```java
java -XX:+UseParallelGC -jar Application.java

// 사용할 쓰레드의 갯수
-XX:ParallelGCThreads=<N>

// 최대 지연 시간
-XX:MaxGCPauseMillis=<N>
```
- Parallel GC가 GC의 오버헤드를 상당히 줄여주었고, Java8까지 기본 가비지 컬렉터로 사용되었다.

### Parallel Old GC
- Parallel Old GC는 JDK5 update6부터 제공한 GC이며, 앞서 설명한 Parallel GC와 Old 영역의 GC 알고리즘만 다르다.
- Parallel Old GC에서는 Mark Sweep Compact가 아닌 Mark Summary Compact이 사용되는데, Summary 단계에서는 앞서 GC를 수행한 영역에 대해 별도로 살아있는 객체를 식별한다는 점에서 조금 더 복잡하다.

1. Mark 단계에서는 Old 영역을 region 별로 나누고 region 별로 살아있는 객체를 식별한다.
2. Summary 단계에서는 region 별 통계정보로 살아있는 객체의 밀도가 높은 부분이 어디까지인지 dense prefix를 정한다. 오랜 기간 참조된 객체는 앞으로 사용할 확률이 높다는 가정 하에 dense prefix를 기준으로 compact 영역을 줄인다.
3. compact 단계에서는 compact 영역을 destination과 source로 나누며 살아있는 객체는 destination으로 이동시키고 참조되지 않는 객체는 제거한다.

### CMS(Concurrent Mark Sweep) GC
- CMS GC는 Parallel GC와 마찬가지로 여러 개의 쓰레드를 이용한다. 하지만 기존의 Serial GC나 Parallel GC와는 다르게 Mark Sweep 알고리즘을 Concurrent하게 수행하게 된다.
<p align="center"><img src="./images/cms_gc.png" width="600"></p>

- 이러한 CMS GC는 애플리케이션의 지연 시간을 최소화하기 위해 고안되었으며 애플리케이션이 구동 중일 때 프로세서의 자원을 공유하여 이용이 가능해야 한다.
- CMS GC가 수행될 때는 자원이 GC를 위해서도 사용되므로 응답이 느려질 수는 있지만 응답이 멈추지는 않는다.
- 하지만 이러한 CMS GC는 다른 GC보다 메모리와 CPU를 더 많이 필요로 하며, Compact 단계를 수행하지 않는다는 단점이 있다.
- 이 때문에 시스템을 장기적으로 운영하다가 조각난 메모리들이 많아 Compact 단계가 수행되면 오히려 Stop The World 시간이 길어지는 문제가 발생할 수 있다.

```java
// deprecated in java9 and finally dropped in java14
java -XX:+UseConcMarkSweepGC -jar Application.java
```
- 만일 GC가 수행되면서 98% 이상의 시간이 CMS GC에 소요되고, 2% 이하의 시간이 Heap의 정리에 사용된다면 CMS GC에 의해 OutOfMemoryError가 던져질 것이다.
- 물론 이를 disable 하는 옵션이 있지만, CMS GC는 Java9 버전부터 deprecated 되었고 결국 Java14 에서는 사용이 중지되었다.

### G1(Garbage First) GC
<p align="center"><img src="./images/g1_gc.png" width="600"></p>

- G1(Garbage First) GC는 장기적으로 많은 문제를 일으킬 수 있는 CMS GC를 대체하기 위해 개발되었고, Java7부터 지원되기 시작했다.
- 기존의 GC 알고리즘에서는 Heap 영역을 물리적으로 Young 영역(Eden 영역 1개와 Survivor 영역 2개)과 Old 영역으로 나누어 사용하였다.
- G1 GC는 Eden 영역에 할당하고 Survivor로 카피하는 등의 과정을 사용하지만 물리적으로 메모리 공간을 나누지 않는다.
 
#### Region이란?
- 대신 Region이라는 개념을 새로 도입하여 Heap을 균등하게 여러 개의 지역으로 나누고 각 지역을 역할과 함께 논리적으로 구분하여(Eden 영역, Survivor 영역, Old 영역) 객체를 할당한다.
- G1 GC에서는 Eden, Survivor, Old 역할과 더해 Humongous와 Available/Unused라는 2가지 역할을 추가하였다.
- Humongous는 Region 크기의 50%를 초과하는 객체를 저장하는 Region을 의미하며, Available/Unused는 사용되지 않는 Region을 의미한다.
- G1 GC의 핵심은 Heap을 동일한 크기의 Region으로 나누고, 가비지가 많은 Region에 대해 우선적으로 GC를 수행하는 것이다.
- 그리고 G1 GC도 다른 가비지 컬렉션과 마찬가지로 2가지 GC(Minor GC, Major GC)로 나누어 수행되는데, 각각에 대해 살펴보도록 하자.

#### 1. Minor GC
- 한 지역에 객체를 할당하다가 해당 지역이 꽉 차면 다른 지역에 객체를 할당하고, Minor GC가 실행된다.
- G1 GC는 각 지역을 추적하고 있기 때문에, 가비지가 가장 많은(Garbage First) 지역을 찾아서 Mark and Sweep를 수행한다.
- Eden 지역에서 GC가 수행되면 살아남은 객체를 식별(Mark)하고, 메모리를 회수(Sweep)한다. 그리고 살아남은 객체를 다른 지역으로 이동시키게 된다.
- 복제되는 지역이 Available/Unused 지역이면 해당 지역은 이제 Survivor 영역이 되고, Eden 영역은 Available/Unused 지역이 된다.

#### 2. Major GC
- 시스템이 계속 운영되다가 객체가 너무 많아 빠르게 메모리를 회수 할 수 없을 때 Major GC(Full GC)가 실행된다.
- 기존의 다른 GC 알고리즘은 모든 Heap의 영역에서 GC가 수행되었으며, 그에 따라 처리 시간이 상당히 오래 걸렸다.
- 하지만 G1 GC는 어느 영역에 가비지가 많은지를 알고 있기 때문에 GC를 수행할 지역을 조합하여 해당 지역에 대해서만 GC를 수행한다. 그리고 이러한 작업은 Concurrent하게 수행되기 때문에 애플리케이션의 지연도 최소화할 수 있는 것이다.
- 물론 G1 GC는 다른 GC 방식에 비해 잦게 호출될 것이다. 하지만 작은 규모의 메모리 정리 작업이고 Concurrent하게 수행되기 때문이 지연이 크지 않으며, 가비지가 많은 지역에 대해 정리를 하므로 훨씬 효율적이다.

```java
java -XX:+UseG1GC -jar Application.java
```
- 이러한 구조의 G1 GC는 당연히 앞의 어떠한 GC 방식보다 처리 속도가 빠르며 큰 메모리 공간에서 멀티 프로레스 기반으로 운영되는 애플리케이션을 위해 고안되었다.
- 또한 G1 GC는 다른 GC 방식의 처리속도를 능가하기 때문에 Java9부터 기본 가비지 컬렉터(Default Garbage Collector)로 사용되게 되었다. 




