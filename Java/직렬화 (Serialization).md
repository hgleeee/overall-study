# 직렬화 (Serialization)

## 정의
- 직렬화란 데이터를 바이트 단위로 변환하여, 파일 또는 네트워크를 통해 송수신(I/O) 가능하도록 만드는 것을 의미한다.
- 직렬화 기술은 마샬링(marshalling)과 원격 프로시저 호출(remote procedure call)과 비슷한 기술이긴 하다.
  - 마샬링은 파라메타를 바이트로 전달하는 작업이라면, 직렬화는 마샬링을 이용하여 [객체나 구조적인 데이터]를 바이트 스트림으로 전환하여 입출력하는 작업이다.
- 웹 환경(객체, 구조화된 데이터), 네트워크 패킷 통신, 분산 컴퓨팅, 데이터베이스에서 구조화 데이터 저장 등 직렬화는 다양한 분야에서 쓰이는 기술이다.
  - 직렬화 성능에 따라 CPU, 메모리, 네트워크 비용이 크게 달라진다.

## 직렬화 데이터 방식
- 네트워크 상에서 데이터를 주고받을 때, Binary 단위로 직접 받아도 되지만 보통은 아래와 같은 직렬화 데이터 구조를 사용한다.

### Json (JavaScript Object Notation)
- 단순한 key-value 텍스트 형식으로 사람, 기계가 모두 읽을 수 있는 직렬화 데이터 형식이다.
### XML (eXtensible HTML)
- 태그를 사용하는 텍스트 형식이다. Json보다 복잡하지만 schema를 적용할 수 있고 무결성 검사가 가능하다.
### YAML (YAML Ain't HTML)
- XML에 보다 사람이 읽기 편하도록 만들어진 마크업 언어이다. 문법이 단순하고 가독성이 높다.

## 직렬화를 사용하기 전 자바
- 자바에서는 정수, 문자열, 바이트 단위의 입출력만 제공했었다.
- 그래서 복잡한 객체를 파일로 저장 <-> 복원하거나 네트워크로 전송하기 위해서는 객체의 멤버 변수를 일정한 형식(네트워크의 경우 패킷 단위)으로 변환하여 전송해야 했다.

## 객체 직렬화
- 자바에서는 Serializable 인터페이스를 통해 직렬화를 제공한다.
- 객체 직렬화는 객체의 내용(멤버 변수의 내용)을 자바 입출력에서 자동적으로 바이트 단위로 변환하여 저장 <-> 복원해주고 네트워크로 전송할 수 있도록 해주는 기능이다.
- 즉, 개발자 입장에서는 복잡한 객체를 사용하더라도 손쉽게 파일로 저장하거나 네트워크로 전송할 수 있다.
- 또한 자바에서 제공하므로 운영체제가 달라도 개발자가 신경쓸 것은 없다.
- 객체 직렬화가 진행될 때 멤버 변수가 레퍼런스를 가지고 있는 경우 (단, 해당 객체도 Serializable 인터페이스를 구현한 경우) 에도 자동으로 해당 객체를 불러와 함께 직렬화해버린다.
  - 이런 식으로 사용되는 레퍼런스 객체들을 전부 연속적으로 직렬화하여 네트워크에 손쉽게 객체를 전달할 수 있게 된다.
- 참고로 externalization 인터페이스를 이용하여 직접 직렬화 코드를 만들 수도 있지만, 사용할 일은 거의 없다.

## 객체 전송의 단계
- 객체를 분해하여 전송하기 위해서는 직렬화(Serialization)되어야 한다.
- 참고로 여기에서 마샬링(Marshalling)이란 데이터를 바이트 덩어리로 만들어, I/O 스트림에 보낼 수 있는 형태로 바꾸는 변환 작업을 의미한다.

```
(1) 직렬화된 객체를 바이트 단위로 분해한다. (marshalling)
(2) 직렬화되어 분해된 데이터를 순서에 따라 전송한다.
(3) 전송받은 데이터를 원래대로 복구한다. (unmarshalling) 
```

## 자바 직렬화 방법
- 자바 직렬화를 위해서 type이 primitive type(기본형 타입)이거나 java.io.Serializable 인터페이스를 상속받아야 한다.
- Serializable의 선언부를 보면 다음과 같이 비어있는 것을 볼 수 있는데, 다른 특별한 기능 없이 Serializable 인터페이스를 구현한 객체는 직렬화 가능하다는 걸 알 수 있는 용도로만 사용된다.

```java
package java.io;
﻿public interface Serializable {}
```

- 직렬화 테스트를 위해 Serializable 인터페이스를 구현한 User 클래스를 선언해보자.

```java
package net.happykoo.model;
﻿
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class User implements Serializable {
    private String name;
    private int age;
    private String email;
    
    @Override
    public String toString() {
        return String.format("User name: %s, age: %s, email: %s", name, age, email);
    }
}
```

- 직렬화를 위해서 java.io.ByteArrayOutputStream, java.io.ObjectOutputStream 을 이용하면 된다.

```java
package net.happykoo.test;

@Slf4j
public class SerializeTest {
    @Test
    @DisplayName("Serialize Test")
    public void SerializeTest() {
        User user = User.builder()
                .name("Happykoo")
                .age(30)
                .email("rudals4549@gmail.com")
                .build();

        String serializedUserBase64;

        try(ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            try(ObjectOutputStream oos = new ObjectOutputStream(baos)) {
                oos.writeObject(user);
                // 직렬화(byte array)
                byte[] serializedUser = baos.toByteArray();
                // byte array를 base64로 변환
                serializedUserBase64 = Base64.getEncoder().encodeToString(serializedUser);
                log.debug("serializedUserBase64 >> {}", serializedUserBase64);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
```bash
//결과
serializedUserBase64 >> rO0ABXNyABduZXQuaGFwcHlrb28ubW9kZWwuVXNlcqLhsk0TYPUEAgADSQADYWdlTAAFZW1haWx0ABJMamF2YS9sYW5nL1N0cmluZztMAARuYW1lcQB+AAF4cAAAAB50ABRydWRhbHM0NTQ5QGdtYWlsLmNvbXQACEhhcHB5a29
```

- 이번에는 위에서 직렬화하여 Base64 값으로 변환한 값을 다시 원래 객체로 역직렬화 해보자.
- 역직렬화 시에는 java.io.ByteArrayInputStream, java.io.ObjectInputStream 을 사용하자.
- 단, 역직렬화를 하기 위해서는 직렬화 대상이 된 객체의 클래스가 클래스 path에 존재해야 하며, import 되어 있어야 한다. (직렬화와 역직렬화를 진행하는 시스템이 서로 다를 수 있음)

```java
package net.happykoo.test;

@Slf4j
public class SerializeTest {
    ...

    @Test
    @DisplayName("Deserialize Test")
    public void DeserializeTest() {
        String serializedUserBase64 = "rO0ABXNyABduZXQuaGFwcHlrb28ubW9kZWwuVXNlcqLhsk0TYPUEAgADSQADYWdlTAAFZW1haWx0ABJMamF2YS9sYW5nL1N0cmluZztMAARuYW1lcQB+AAF4cAAAAB50ABRydWRhbHM0NTQ5QGdtYWlsLmNvbXQACEhhcHB5a29v";
        byte[] serializedUser = Base64.getDecoder().decode(serializedUserBase64);

        try(ByteArrayInputStream bais = new ByteArrayInputStream(serializedUser)) {
            try(ObjectInputStream ois = new ObjectInputStream(bais)) {
                // 역직렬화(byte array -> object)
                Object objectUser = ois.readObject();
                User user = (User) objectUser;
                log.debug(user.toString());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
```bash
//결과
User name: Happykoo, age: 30, email: rudals4549@gmail.com
```

- 제대로 역직렬화 되었음을 확인할 수 있다.

### transient
- 비밀번호와 같이 직렬화하는 대상에서 제외하고 싶은 항목이 있을 수 있다.
- 이 때, transient 키워드를 이용하면, 그 property는 직렬화 대상에서 제외된다.

```java
public class User implements Serializable {
    private String name;
    private int age;
    //직렬화에서 제외
    private transient String email;

    ...
}
```
- 직렬화 후 역직렬화 하게 되면, 해당 제외된 property는 null 값이 들어간다.

## 언제, 왜 사용하는가?
- 데이터를 직렬화하는 방법에는 자바 직렬화뿐만 아니라 CSV, XML, JSON으로의 직렬화도 존재한다.
  - 표 형태의 다량의 데이터를 저장하거나 전송할 때 CSV를 많이 이용한다. (CSV로 직렬화 시 Apache Commons CSV, opencsv 등의 라이브러리를 많이 이용) 
  - 네트워크에서 구조적인 데이터를 송수신 할 때(API 시스템), 대부분 JSON을 많이 사용한다. (JSON으로 직렬화 시 Jackson, GSON 등의 라이브러리를 많이 이용)
- 그렇다면, 위의 CSV, JSON으로의 직렬화를 사용하면 되는데, 굳이 자바 직렬화는 왜 사용하는 것일까?
- 자바 직렬화는 같은 자바 시스템에서의 데이터 전송(다른 JVM 환경의 시스템으로의 전송), 저장에 최적화되어 있다.
- 데이터 구조가 복잡하더라도 직렬화의 기본 조건만 지키면, 바로 직렬화와 역직렬화가 가능하다.
- 무엇보다 데이터 타입이 자동으로 맞춰지기 때문에, 역직렬화 시 바로 기존 객체처럼 사용 가능하다.
  - 개발자가 일일이 타입을 맞춰 줄(형변환) 필요가 없으므로 상당히 편리하다.
  - 물론, JSON, CSV 로의 직렬화, 역직렬화도 Jackson 등의 라이브러리를 이용하면 편리하지만, 외부 라이브러리를 사용하지 않고 자바가 기본으로 제공하는 라이브러리만 사용한다는 전제.

### 자바 직렬화의 사용 (JSON으로 직렬화하여도 사용 가능)
#### 서블릿 세션(Servlet Session)
- 서블릿 기반의 WAS(톰캣, 웹 로직)들은 대부분 세션의 자바 직렬화를 지원한다.
- 단순히 메모리 위에서 운용하는 세션이 아닌(서버 재시작 시 초기화) 클러스터링 환경에서 세션 공유를 위해 파일로 저장하거나 DB에 저장할 때 세션 객체를 자바 직렬화를 사용하여 저장할 수 있다.

#### 캐시(Cache)
- 실시간으로 반복적으로 DB에서 조회하는 데이터가 아니라면, 리소스 절약을 위해 주로 캐시(메모리 DB) 라이브러리 시스템(Ehcache, Redis ...)에 데이터를 저장해 놓고 사용하게 된다.
- 이 때 데이터를 자바 직렬화하여 저장할 수 있다.

#### 자바 RMI(Remote Method Invocation)
- 사실 자바 직렬화는 자바 RMI에서 많이 사용했다고 한다. (지금은 잘 사용하지 않는..)
- RMI(Remote Method Invocation)은 쉽게 말하면 원격 시스템(외부 시스템)에서 원격에 있는 시스템 메서드를 로컬 시스템의 메서드인 것처럼 호출하는 것을 말한다.
  - 메서드의 파라미터로 객체를 전달할 때, 그 객체를 자동으로 직렬화시켜 전달한다.
- 그렇기에 파라미터로 전달되는 데이터는 serializable 인터페이스를 구현해야 한다.
- 데이터를 전달받은 원격 시스템에서는 직렬화된 데이터를 역직렬화하여 사용하게 된다.

## 주의사항
### 구조 변경
- 객체를 자바 직렬화한 후 데이터(클래스) 구조가 변경될 때의 문제이다.
- 다음과 같이 User 클래스의 객체를 직렬화했다고 가정하자.

```java
public class User implements Serializable {
    private String name;
    private int age;
    private String email;

    ...
}
```

```java
User user = User.builder()
        .name("Happykoo")
        .age(30)
        .email("rudals4549@gmail.com")
        .build();
```

```bash
rO0ABXNyABduZXQuaGFwcHlrb28ubW9kZWwuVXNlcqLhsk0TYPUEAgADSQADYWdlTAAFZW1haWx0ABJMamF2YS9sYW5nL1N0cmluZztMAARuYW1lcQB+AAF4cAAAAB50ABRydWRhbHM0NTQ5QGdtYWlsLmNvbXQACEhhcHB5a29v
```

- 이 때, 만약 User 클래스에 address 프로퍼티가 추가되었다고 했을 때, 기존의 직렬화된 데이터를 역직렬화하면 어떤 일이 발생할까?

```java
public class User implements Serializable {
    private String name;
    private int age;
    private String email;
    //추가
    private String address; 
    ...
}
```
```bash
java.io.InvalidClassException: net.happykoo.model.User; local class incompatible: stream classdesc serialVersionUID = -6709885925697981180, local class serialVersionUID = -5519332639680234859
```

- 에러를 확인하면 serialVersionUID가 달라서 발생한 에러이다.
  - serialVersionUID를 어디에도 선언한 적이 없는데, 어떻게 된 일일까?
- 직렬화 시 serialVersionUID는 직접 기술하지 않아도 내부적으로 추가되며, 클래스 구조 정보를 이용하여 생성된 해쉬값을 이용한다.
  - 그리고 serialVersionUID를 비교하여 다르다면, 역직렬화가 불가능하다. (클래스 구조가 변경되면 serialVersionUID도 변경됨)
  - 클래스 구조가 변경되었을 때, 역직렬화 시 에러를 발생시켜야 하는 시스템이 아니라면, serialVersionUID를 개발자가 직접 관리할 수도 있다.
- 다음과 같이 User 클래스에 serialVersionUID를 추가해주자. (다시 address property 제거)

```java
public class User implements Serializable {
    //serialVersionUID 추가
    private static final long serialVersionUID = 1L;
    private String name;
    private int age;
    private String email;
    ...
}
```

- 그리고 아까 객체를 다시 직렬화한 후 User 클래스에 address property를 추가하여 역직렬화 해보면, 제대로 역직렬화됨을 알 수 있다. (address는 null로 셋팅 됨)
- 이처럼 자바 직렬화 / 역직렬화는 클래스 구조 변경에 민감한데, 경우의 수를 정리해보면 다음과 같다. (serialVersionUID를 설정했다고 가정)

```
1. 직렬화 후 property를 추가, 삭제하거나 이름을 변경했을 때는 에러가 발생하지 않고 단순히 값이 삭제되거나 null로 셋팅된다.
2. 직렬화 후 property의 타입을 변경하면 에러가 발생한다. (직렬화는 타입에 엄격함)
```

- 위와 같은 특성 때문에 자바 직렬화를 자주 변경되는 클래스의 객체에 사용하는 것을 지양해야 한다.
- 또한, 외부(DB, Cache, NoSQL 서버)에 장기간 저장될 정보에 자바 직렬화를 사용하는 것도 피해야한다.
  - 데이터가 저장되어 있는 동안 데이터 구조가 변경된다면, 그 직렬화된 데이터는 쓰레기(Garbage)가 되어 버릴 가능성이 높기 때문이다.
- 개발자가 직접 컨트롤이 불가능한 외부 객체(serialVersionUID 관리 불가능)의 직렬화 또한 지양해야 한다.
  - 아까 봤듯이 serialVersionUID가 변경되면 역직렬화 시 에러가 발생하기 때문이다.

### 용량
- 자바 직렬화 시에는 기본적으로 타입에 대한 정보 등 클래스에 대한 메타정보가 포함되어 있기 때문에 상대적으로 다른 포맷(JSON, CSV)에 비해 용량이 크다.
- 이를 확인하기 위해 아까 위에서 테스트한 User를 각각 자바 직렬화, JSON 으로 변경하여 size를 측정해 보자.

```java
try(ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
  try(ObjectOutputStream oos = new ObjectOutputStream(baos)) {
    oos.writeObject(user);
    byte[] serializedUser = baos.toByteArray();
    serializedUserBase64 = Base64.getEncoder().encodeToString(serializedUser);

    ObjectMapper objectMapper = new ObjectMapper();
    String userJson = objectMapper.writeValueAsString(user);
    log.debug("serializedUserBase64 >> {}", serializedUserBase64.length());
    log.debug("json >> {}", userJson.length());
  }
} catch (IOException e) {
  e.printStackTrace();
}
```
```bash
serializedUserBase64 >> 196
json >> 74
```

- 자바 직렬화 시, 약 2배정도 용량을 차지하는 것을 알 수 있다.
- 만약, 클래스 내부에 또 다른 클래스나 리스트를 가지고 있다면, 용량 차이는 몇 배 더 커질 수도 있다.
- 따라서, 메모리 캐시 서버(Redis, Ehcache ...) 처럼 데이터의 크기가 성능에 크게 영향을 미치는 곳에는 자바 직렬화가 아닌 다른 포맷(JSON)으로 변환하여 저장하는 것을 고려해야 한다.
- 특히 Spring Framework 에서 기본적으로 지원하는 캐시 모듈(Spring Data Redis, Spring Session..)은 개발자가 데이터 형식에 신경쓰지 않고 빠르게 개발하게 하기 위해 디폴트로 자바 직렬화 형태를 제공하기 때문에, 잘 판단해야 한다.



