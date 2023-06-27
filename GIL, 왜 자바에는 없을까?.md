# GIL, 왜 자바에는 없을까?
## GIL이란?
- [GIL(Global Interpreter Lock)](https://github.com/hgleeee/overall-study/blob/main/GIL(Global%20Interpreter%20Lock).md), 해당 링크에서 알 수 있다.

## Cpython은 왜 GIL을 선택했는가?

### 배경
- Cpython에서는 오직 하나의 스레드만이 인터프리터에 대한 점유권을 획득한다. 때문에 멀티 스레드 환경에서 서로 Lock에 대한 점유권을 획득하는 과정에서 Bottleneck과 같은 성능 저하가 일어날 수 있다.
- 이러한 성능 이슈에도 불구하고 Cpython이 언어 스펙 측면에서 GIL이라는 제약을 가져간 이유가 무엇일까?
- 이를 이해하기 위해서는 Cpython이 메모리를 어떻게 관리하는지 대한 이해가 필요하다.

### Cpython은 어떻게 메모리를 관리할까?
- Cpython에서는 메모리를 관리하는 방법으로 __Reference Counting__ 을 이용한다.
- Refrence Counting은 오브젝트마다 자신을 참조하고 있는 변수의 수를 저장한다. 이 수가 0이 되었을 때 Garbage로 판단하고 메모리 해제를 진행한다.
- 이러한 방식의 Reference Counting은 Garbage를 메모리에 쌓아놓지 않고 바로 해제할 수 있다는 점에서 메모리 공간 측면의 이점이 있다.
- 반면에 참조가 해제되거나 추가될 때마다 실시간으로 Reference Count가 동기화되어야 한다.

### Reference Count 업데이트(동기화 동반) 방법
- Reference Count를 업데이트하는 작업은 Atomic한 연산이 되어야 한다. 그렇지 않다면 여러 스레드에서 동시에 접근해 원치 않는 결과가 나올 수 있다.
- Reference Count 업데이트 연산의 Atomic을 보장하기 위해서는 매 연산마다 Lock을 걸어줘야 하는데, 이는 성능에 상당한 영향을 미치며 최악의 경우에는 Dead Lock을 야기시킬 수 있다.
- 때문에 Cpython에서는 인터프리터 전체에 Lock을 걸어 Atomic을 보장한다.

## Java에는 왜 GIL이 없는가?
- Java는 __Mark and Sweep(GC)__ 으로 메모리를 관리한다.
- 객체 참조에 대한 수를 저장하지 않고, 루트부터 하나씩 참조를 따라가면서 찾은 객체가 살아있음을 Marking 한 후 전체 힙 객체들을 순회하면서 마킹되지 않은 객체들을 제거하는 방식이다.
- Mark and Sweep에서는 메모리가 일정 이상 찼을 때 위에서 언급한 컬렉팅 작업이 시작된다. 때문에 오브젝트의 참조가 변경 될 때마다 Atomic한 연산이 수행될 필요가 없다.
- 이것이 Java에서 GIL라는 용어를 찾아보기 힘든 이유이다. 대신에 참조를 확인하는 mark 과정에서 모든 스레드가 일시적으로 중단시켜 GC의 atomic을 보장한다.
- 이 시기를 stop the world라 부르는데 이 stop the world를 줄이기 위해 여러 GC 알고리즘이 발전하고 있다.

## Mark & Sweep  vs.  Reference Counting
- Mark and Sweep은 여러 스레드가 인터프리터로 접근하는 것을 막지 않는다. 그렇다면 Mark and Sweep이 무조건 Referecne Counting보다 좋은가? 그건 아니다. 모든 것은 트레이드 오프다.
- Mark and Sweep은 일반적인 상황에서 스레드의 동시적인 접근이 가능하다는 장점이 있으나, Garbage를 곧바로 회수하는 것이 아닌 메모리에 쌓아놓기 때문에 메모리 공간 측면에서 단점이 있다. 또한 객체가 소멸되는 시점을 예측하기 힘들다.
- 즉 객체의 소멸시점이 예측이 되어야 하는 경우, 또는 메모리 공간 사용의 제약이 있는 경우는 Reference Counting이 유리하다.

## Cpython은 왜 Reference Counting을?
- Cpython이 Reference Counting을 채택한 배경에는 C로 구현된 언어인만큼 C의 영향이 큰 것으로 보인다.





