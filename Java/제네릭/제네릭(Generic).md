# 제네릭 (Generic)
- 기존의 자바에서는 Object 타입을 이용하여 다형성을 구현하였다.
- 이 경우 반환된 Object 타입을 다시 원하는 객체로 변환하는 과정에서 문제가 발생하더라도 컴파일 타임에서 해당 오류를 잡을 수 없었다.
  - 즉, 단순한 타입 실수가 런타임 오류로 넘어가 App이 멈춰버리는 최악의 상황까지 갈 수 있었다.
- 그래서 Java 5부터 Generic 이라는 타입 기능을 제공해준다.
  - 컴파일 시점에 타입을 체크하여 다양한 문제를 방지하고 다형성을 지원하는 코드를 쉽게 만들 수 있게 해준다.

## 제네릭의 장점
- 제네릭의 사용에서 얻는 이점은 컴파일 타임에 강한 타입 체크를 할 수 있게 만들어준다는 것이다.
- 타입을 명시적으로 표시하면서 불필요한 타입 캐스팅이나 런타임에 발생할 수 있는 오류를 예방해준다.


#### 제네릭을 사용한다고 해서 바이트코드가 변하지는 않는다.
- 제네릭은 컴파일 타임에 도와주는 도구이지, 컴파일 이후의 바이트코드에 변화를 주거나 JVM의 성능을 향상시키지는 않는다.
- 컴파일이 완료된 바이트코드를 뜯어보면 제네릭은 java.lang.Object로 서술되어있기에 자바 코드에서 제네릭을 일반 타입으로 바꿔도 바이트코드는 동일하다.
- 즉, 아래의 두 코드는 제네릭을 사용하던, 그냥 List를 사용하던 동일한 바이트 코드를 생성을 한다.



// 만약 List에 다른 자료형이 들어간다해도 컴파일 시점에 확인할 방법이 없다.
// list에 다른 타입을 잘못 넣은 사실을 개발자가 놓친다면 심각한 런타임 버그를 만들 수 있다.
List list = new ArrayList();
list.add("hello");
String str = (String) list.get(0); // 불필요한 Object -> String 캐스팅
// 제네릭은 타입을 명시적으로 나타내고, 컴파일 시점에 체크가 가능하다.
// 필요하다면 < exnteds, super > 등을 이용하여 컴파일 시점에 타입을 강제할 수도 있다.
List<String> list = new ArrayList<String>();
list.add("hello");
String str = list.get(0); // list에 다른 타입이 들어갈 수 없다. 타입이 String으로 강제된다.
제네릭으로 선언된 객체는 컴파일 시점에 타입이 결정된다. 그래서 클래스 제네릭의 경우 Static 멤버나 new 의 경우 인스턴스가 만들어지기 전까지 타입을 알 수없기 때문에 제네릭 타입변수(T)를 사용 할 수 없음을 유의하자. 

# 클래스 제네릭
 public class 클래스명<T> { ... }    
 public interface 인터페이스명<T> { ... }
 
public class Box<T> {
  private T t;
 
  public T get() { return t; }
 
  public void set(T t) { this.t = t }
}
Box<Integer> box = new Box<Integer>();
Box<String> box = new Box<String>();
 
// 아래와 같이 생략하더라도 컴파일러가 위의 코드로 변환해준다.
Box<Integer> box = new Box<>();
Box<String> box = new Box<>();
 

# 제네릭 메서드 
클래스가 아닌 메서드 단위에서도 제네릭 사용이 가능하다. 반환값 앞에 <타입>을 명시해주면 된다.

 public <타입파라미터,...> 리턴타입 메소드명(매개변수,...) { ... }
 
 // 반환값 앞에 타입을 명시해주어야한다.
 public <T> Box<T> boxing(T t) { ... }
// 사용할 때도 마찬가지로 반환 값 앞에 타입을 명시해주면 된다.
Box<Integer> box = <Integer>boxing(100); 
Box<Integer> box = boxing(100); //java 7부터 타입을 생략할 수 있게 되었다.
 

# 멀티타입 파라메타
참고로 제네릭에 여러타입을 사용하는 건 Java 7부터 지원하는 기능이다.

class<K,V,...> { ... }
interface<K,V,...> { ... }
Product<TV, String> product = new Product<Tv,String>();
Product<Tv,String> product = new Product<>(); // 타입을 생략해도 된다. 추론 가능
public class Pair<K, V> {
  private K key;
  private V value;
 
  public Pair(K key, V value) {
    this.key = key;
    this.value = value;
  }
  public void setKey(K key) {
    this.key = key;
  }
 
  public void setValue(V value) {
    this.value = value;
  }
 
  public K getKey() {
    return key;
  }
 
  public V getValue() {
    return value;
  }
}
public class Util {
  public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
    boolean keyCompare, valueCompare;
    keyCompare = p1.getKey().equals(p2.getKey());
    valueCompare = p1.getValue().equals(p2.getValue());
 
    return keyCompare && valueCompare;
  }
}
 

# 제네릭 타입 제한
extends 와 super를 이용하여 타입을 제한 할 수 있다. 참고로 여러 클래스 제약을 동시에 걸고 싶다면 & 연산자를 쓰면 된다.

// 해당 타입 또는 해당 타입의 자식 타입만 사용가능
public <T extends 타입> 리턴타입 메소드(매개변수, ...) { ... }
 
// 해당 타입 또는 해당 타입의 부모 타입만 사용가능
public <T super 타입> 리턴타입 메소드(매개변수, ...) { ... }
 
// 다중상속은 안되지만, 인터페이스의 경우 여러 클래스의 제약을 동시에 거는 경우도 생긴다.
public class Item<T extends Book & List> { }
// 다음과 같은 제네릭이 있을 때
public class Item<T extends Book> { }
 
public class Book { } // 자기자신, Book 클래스 사용가능
public class JavaBook extends Book { } // Book의 자식클래스 사용가능
 
public class Drink { } // 상관없는 클래스. 사용불가능
public <T extends Number> int compare(T t1, T t2) {
    double v1 = t1.doubleValue();
    double v2 = t2.doubleValue();
 
    return Double.compare(v1, v2);
}
 

# 와일드 카드
제네릭으로 만들어진 클래스를 사용하는 코드를 살펴보자.

class Juicer {
    static Juice makeJuice(FruitBox<Fruit> box) {
        String names = "";
        for(Fruit fruit : box.getList()) { names += fruit }
        return new Juice(names);
    }
}
제네릭을 만들 때에는 별 문제가 없을 줄 알았지만, 사용할 때 다음과 같은 문제가 발생한다.

FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
Juicer.makeJuice(fruitBox); // OK
 
FruitBox<Grape> grapeBox = new FruitBox<Grape>();
Juicer.makeJuice(grapeBox); // Err!, FruitBox는 Fruit타입으로 결정되어 변경불가능.
 

이 때 사용해라고 만든 것이 와일드카드( <?> ) 이다. 와일드카드는 어떠한 타입도 받을 수 있는 제네릭인데, 여기에 제네릭 제한을 걸어 내가 원하는 특정 타입을 지정하여 사용 할 수 있다.

<? extends T> // T와 자손들만 가능
<? super T> // T와 조상들만 가능
<?> // 모든 타입 가능. <? extends Object>와 같음. 보통 이렇게 사용하지는 않음
// 와일드 카드를 이용하여 제네릭을 구현하면 다형성을 구현하기 쉬워진다.
static Juice makeJuice(FruitBox<? extends Fruit>) {
    ...
}
 
FruitBox<Grape> grapeBox = new FruitBox<Grape>();
Juicer.makeJuice(grapeBox); // OK!
실제로 여러 자료형을 다루는 Collection의 경우 와일드카드를 이용하여 만든 메서드들을 많이 볼 수 있다.

static <T> void sort(List<T> list, Comparator<? super T> cmp)
