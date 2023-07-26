# 다이나믹 프록시(Dynamic Proxy)
## 다이나믹 프록시란?
- 다이나믹 프록시란 애플리케이션 실행 도중, 특정 인터페이스들을 [구현하는 클래스 또는 인스턴스]를 자동으로 만드는 기술이다.
  - 다이나믹 프록시는 리플렉션 기술을 통해 만들어진다.
- 스프링 데이터 JPA에서 인터페이스 타입의 인스턴스는 누가 만들어주는 것인가?
  - Spring AOP를 기반으로 동작하며 RepositoryFactorySupport에서 프록시를 생성한다.
  - Spring AOP의 가장 밑단에서는 Proxy 라이브러리를 이용한 리플렉션을 사용하는 것을 볼 수 있다.
 
```java
@Entity
public class Book{
    ...
}
 
@Service
public class BookService {
    @Autowired // 인터페이스에 자동 주입
    BookRepository bookRepository;
}
```
```java
public interface BookRepository extends JpaRepository<Book, Integer>{
}
 
// 스프링 빈으로 등록될 것 같지도 않고, 인터페이스에는 아무런 코드도 없다.
// 하지만 위의 @Autowired는 정상적으로 동작하고, 실제 인스턴스가 주입된다.
// 어노테이션도 없는데 어떻게 리플렉션으로 이 모든 기능을 만들 수 있었을까?
```

## 프록시 패턴
<p align="center"><img src="../images/proxy_pattern.png" width="700"></p>

- Proxy는 '대리인'이라는 의미를 가진다. 개발에서 사용하는 Proxy 객체는 클라이언트-서버 등의 관계에서 중간에 끼어 대리인 역할을 하는 객체라고 생각하면 된다.
- 프록시와 실제 객체가 공유하는 인터페이스가 있고, 클라이언트는 인터페이스를 통해 프록시를 사용한다.
- 클라이언트는 실제 객체를 사용하는 것처럼 동작하지만, 실제로는 프록시를 통해서 접근하게 된다.
- 이렇게 중간에 프록시를 사용함으로써 접근관리(보안), 캐싱(성능), 필터 등의 부가 기능들을 제공할 수 있다.
- 또한 실제 객체는 자신이 해야할 일(단일책임원칙, SRP)만 집중하면서 부가적인 기능(로깅, 트랜잭션 처리등)을 프록시 객체에게 넘겨 객체지향적인 프로그래밍을 할 수 있게 된다.

```java
public interface BookService{ ... }
```
```java
// 클라이언트는 인터페이스를 사용해서 객체를 사용한다.
// 하지만 실제로 호출되는 것은 중간에 있는 프록시 객체이다.
public class BookServiceProxy implements BookService {
    BookService bookService; // 실제 객체를 담는 곳
    
    @Override
    public void rent(Book book){
    	bookService.rent(book); // 보통은 프록시를 호출하면 실제 객체에게 위임하도록 설계한다.
        // 물론 실제 객체를 호출하지 않고 프록시에서 원하는 동작을 하도록 바꿀 수도 있다.
    }
}
```

### 프록시의 문제점
- 기존 프록시의 가장 큰 단점은 부가적인 기능을 추가할 때마다 프록시를 새롭게 만들어야 한다는 것이다.
- 프록시를 하나만 사용한다면 괜찮겠지만, 실제 어플리케이션을 구현할 때에는 정말 많은 프록시들이 중첩되게 된다.
- 그렇다고 단 하나의 프록시에 모든 기능을 합치게 되면 프록시를 사용하여 실제 객체와 분리하는 의미 자체가 없어지게 된다.
  - 그냥 객체가 많아져 코드 관리만 복잡해질 뿐이다.
- 실제 객체가 인터페이스를 구현한다면 프록시 또한 사용하지도 않는 모든 메서드를 @Override 해야한다.
- 프록시가 중첩되면 코드가 복잡해지고, 같은 코드의 중복이 발생한다.
  - 이를 해결하기 위해 나온 것이 리플렉션을 사용해서 런타임에 만드는 동적 프록시, 즉 Dynamic proxy이다.
  - 프록시 객체를 기존 객체를 상속받아 하나하나 직접 만드는게 아니라, 런타임 시점에 [클래스 정보 Object.class]를 이용하여 어노테이션, 메서드에 따라 동적으로 다른 메서드 동작을 실행하도록 프록시를 만들 수 있다. 

## 다이나믹 프록시 특성
<p align="center"><img src="../images/dynamic_proxy_1.png" width="700"></p>

- 실제 객체와 같은 인터페이스를 사용하는 프록시 객체를 만든다.
- 자바 API로는 실제 객체를 바로 상속받아서 조작하는 기능을 제공하지 않는다.
- 프록시 객체에서 해당 객체의 리플렉션 Class<T> (myClass.class)를 조작하고 조작된 Class<T>를 이용해서 런타임에 [프록시 객체 인스턴스]를 만든다.
- 개발자는 리플렉션 Class<T>를 쉽게 조작하기 위해서 Java API에 있는 InvocationHandler 객체의 invoke() 메서드만 오버라이딩하여 조작하고 싶은 내용을 작성하면 된다.
- 이렇게 프록시 객체를 만들면, 특정 메서드나 @어노테이션 값에 따라 동적으로 동작이 달라지는 프록시를 만들 수 있다. => 다이나믹 프록시

<p align="center"><img src="../images/dynamic_proxy_2.png" width="700"></p>

- Proxy 객체의 IH(Invoke Handler)만 개발자가 구현하여 편하게 사용할 수 있는 API를 제공해준다.

```java
public class BookServiceProxy implements BookService {
    // java.lang.reflect.Proxy.newProxyInstance
    // BookService 인터페이스에 프록시 객체 인스턴스를 런타임에 동적으로 등록하는 방법.
    BookService bookService = (BookService) Proxy.newProxyInstance(
            BookService.class.getClassLoader() // 프록시를 만들 ClassLoader
            ,new Class[]{BookService.class} // 프록시를 만들 인터페이스
            ,new InvocationHandler() { //invoke handler, 프록시의 내용. 직접 작성한 구현체
                BookService bookService = new DefaultBookService();
 
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    // 아무런 동작도 하지않고, 단순히 실제 객체에게 위임하는 프록시 코드.
                    // Object invoke = method.invoke(...)등으로 리플렉션 조작가능.
                    // method.invoke(메서드를 호출할 객체 , 메서드에 전달할 파라메타)
                    
                    // 여기에 추가코드를 작성하면 객체의 모든 메서드가 해당 코드를 동작한다.
                    return method.invoke(bookService, args);
                }
            });
            
    /*... 기타 메서드 생략 ...*/
}
```

- 만약 특정 메서드( rent() )만 추가적인 동작을 하도록 만들고 싶다면, 아래와 같이 조건을 만들어주면 된다.
```java
if (method.getName().equals("rent")) {
    System.out.println("aaaa");
    Object invoke = method.invoke(bookService, args);
    System.out.println("bbbb");
    return invoke; // 이제 bookService.rent()는 실행 전후로 aaaa, bbbb를 출력한다.
}
```
- 다만 자바 API에서 제공하는 다이나믹 프록시는 인터페이스에서만 생성할 수 있다.(BookService.class.getClassLoader()).
- 인터페이스를 사용하는게 코드도 깔끔하고 제약사항도 없다. + 아마 자바 코드의 하위 호완성 때문에 제공하지 않는게 아닌가 싶다.
- 실제로 클래스로 생성하려고 하려고 하면 컴파일 오류가 나오게 된다.
- 만약 클래스의 다이나믹 프록시가 필요하다면 다른 방법이 있는데, 이는 아래의 [클래스의 프록시 만들기] 에서 알아보도록 하자.

### 단점
- JDK dynamic proxy로 구현되면 프록시 사용을 위해 Interface 사용이 강제화한다는 단점이 있다.
- Dynamic Proxy는 Invocation Handler를 상속받아서 실체를 구현하게 되는데, 이 과정에서 특정 객체를 통으로 Reflection을 사용하는 방식이라서 성능이 조금 떨어진다.

## Spring AOP와 프록시
<p align="center"><img src="../images/spring_aop_process.png" width="600"></p>

- 하지만 위와 같이 직접 동적 프록시를 만드는 것은 코드도 상당히 복잡해지고 구현하기도 어렵다.
- 그래서 스프링에서는 리플렉션 라이브러리(CGlib)를 이용하여 위와 같은 복잡한 구조를 추상화된 인터페이스로 포장해서 제공해주는데, 이것이 바로 스프링 AOP이다.
- 그래서 스프링 AOP는 프록시 기반으로 동작한다고 말한다. 이는 스프링을 공부하면서 배워보도록 하자.
- 스프링에서 AOP를 어떻게 만들었는가 궁금하면 [토비의 스프링3.1, 6장-AOP]를 읽어보도록 하자.

## 클래스의 프록시 (인터페이스가 없는 경우, 상속 프록시)
- 위에서 말했듯이, 자바 API에서 제공하는 다이나믹 프록시는 인터페이스에서만 생성할 수 있다.
- 클래스 프록시가 필요하다면 직접 리플렉션으로 구현하거나 아래와 같은 리플렉션 라이브러리들을 사용하면 된다.
- 참고로 타겟 객체의 상속을 이용해서 프록시를 만드는 방법을 Subclass 프록시라고도 한다.

<p align="center"><img src="../images/cglib_proxy.png" width="600"></p>

### CGlib ( https://github.com/cglib/cglib/wiki )
- CGlib는 바이트코드를 조작하여 프록시 객체를 생성해주는 코드 생성(Code gen)라이브러리이다.
- 런타임에 동적으로 자바 클래스의 프록시 생성 기능을 제공한다
- 스프링, 하이버네이트가 사용하는 리플렉션 라이브러리 (물론 Java 리플렉션 API도 같이 사용한다)
- 버전 호환성이 좋지 않아서 서로 다른 라이브러리 내부에 내장된 형태로 제공하기도 한다.
- 스프링에서는 객체를 상속해서 프록시를 만들고 CGlib를 이용해 리플렉션을 활용한다.
- 왜 스프링 빈 객체의 생성자는 protected, public 밖에 사용할 수 없는가? 에 대한 답변이 된다.

### JDK API에서 제공하는 동적 프록시와의 차이점 
- JDK 프록시는 인터페이스에 적혀있는 모든 메서드에 프록시를 적용시킨다. 그리고 Class<T>정보로 조건문을 만들어 특정 메서드만 추가동작을 하도록 만들 수 있다.
- CGlib는 MethodInterceptor 객체를 사용하여 특정 메서드만 프록시화 하고, 나머지는 프록시를 거치지 않고 실제 객체 메서드를 호출하도록 만들 수 있다.
- 이게 가능한 이유는 CGlib는 인터페이스가 아닌 타겟 객체를 직접 상속받아 만들기 때문이다.

```java
MethodInterceptor handler = new MethodInterceptor() {
    private final MethodMatcher methodMatcher; // 특정 메서드를 찾는 객체
    BookService bookService = new BookService();
	
    public MethodInterceptor(MemthodMatch m){
    	this.methodMatcher = m;
    }
    
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy
            methodProxy) throws Throwable {    
        if (methodMatcher.matches(method)) {
            // CGlib는 상속된 프록시 객체를 사용하기에 특정메서드만 프록시로 바꿀 수 있음
            // JDK 다이나믹 프록시와 다르게 필요없는 메서드는 프록시를 안거치게 할수있음.
            return methodName.toUpperCase();
        }
        return method.invoke(bookService, objects);
    }
};
```
```java
// CGlib로 조작한 프록시 객체를 삽입
BookService bookService = (BookService) Enhancer.create(BookService.class, handler);
```

### ByteBuddy
- 바이트 코드를 조작을 제공해주는 라이브러리지만 Cglib처럼 런타임(다이나믹) 프록시를 만들 때도 사용한다.
- CGlib와 마찬가지로 상속을 이용한 (subClass) 프록시를 지원한다.
```java
Class<?> dynamicType = new ByteBuddy()
  .subclass(Object.class)
  .method(ElementMatchers.named("toString"))
  .intercept(FixedValue.value("Hello World!"))
  .make()
  .load(getClass().getClassLoader())
  .getLoaded();
 
assertThat(dynamicType.newInstance().toString(), is("Hello World!"));
```

- 다만 리플렉션과 프록시를 사용할 정도의 어플리케이션이라면 비즈니스 규모가 크고 코드가 복잡한 경우가 많다.
  - 그래서 가능하면 클래스를 상속받아서 만드는 것보다는, 가능하면 인터페이스 프록시를 사용해서 만드는 것이 좋다.
- 상속을 사용하지 못하는 경우 프록시를 만들 수 없다. (스프링 빈 private, final 클래스가 안되는 이유)
- 클래스로 프록시를 계속 상속받으면 비즈니스가 커졌을 때 유지보수, 확장이 힘든 코드가 될 수 있다.
- 즉 가능하면 클래스가 아닌 인터페이스의 프록시를 만드는 것이 유연한 코드를 만들기 좋다.

## 다이나믹 프록시 정리 (바이트코드 Weaving)
- 다이나믹 프록시를 사용한다는 말은 런타임에 [인터페이스 프록시 인스턴스] 또는 [클래스의 프록시 인스턴스] 또는 [리플렉션을 이용한 클래스 자체]를 만들어 사용하는 프로그래밍 기법을 사용한다는 말이다.
- 이렇게 원하는 타겟 객체에 프록시를 적용시켜,  동적으로 [프록시된 객체]를 삽입하는 기술을 Runtime Weaving이라고 표현한다.
- 스프링에서는 JDK Weaving, CGlib Weaving을 사용하는데 이는 위에서 설명한 리플렉션, 프록시 내용과 같은 말이다.

### 다이나믹 프록시의 사용처 
#### 스프링 AOP
- AOP는 설명하면 끝도 없으므로, 궁금하다면 [토비의스프링3.1v 6장 AOP]를 읽어보자.

#### 하이버네이트 Lazy Initialization
```java
@Entity
@Getter
public class Book {
    @Id @GeneratedValue
    private Integer id;
 
    private String title;
 
    @OneToMany
    private List<Note> notes;
    // Book을 조회할 때 List<Note>를 같이 가져오진 않는다. 실제 notes를 사용할 때 쿼리를 발생시켜 받아온다.
    // 이는 Fetch 전략을 Lazy로 했기 때문. 그렇다면 notes를 사용하기 전까지는 값은 null 인가?
    // 하지만 실제로 조회해보면 null이 아니다. 그 이유는 다이나믹 프록시 객체를 이용하기 때문이다.
}
```

#### Mockito (진짜처럼 동작하는 테스트용 Mock 객체를 지원하는 프레임워크)
```java
import static org.mockito.Mockito.mock;
 
// 실제 객체를 상속한 다이나믹 프록시를 활용해서 만든 mock 객체.
BookRepository bookRepositoryMock = mock(BookRepository.class);
 
// Mockito 프레임웤은 이렇게 특정 메서드의 반환값을 내가 원하는 객체로 조작 가능 (mocking)
Book testBook = new Book("myBookName");
when(bookRepositoryMock.save(any())).thenReturn(testBook); // 아무거나 넣어도 testBook 반환
 
BookService bookService = new BookService(bookRepositoryMock);
bookService.rent(new Book("custom name"))
book.rent() // Mockito의 다이나믹 프록시 적용 => 무조건 Book("myBookName")이 반환됨
public class BookService {
    BookRepository bookRepository;
 
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
 
    public void rent(Book book) {
        Book savedBook = bookRepository.save(book);
        System.out.println("rent: " + savedBook.getTitle());\
    }
}
```

