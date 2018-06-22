# 자바 제네릭

## 제네릭 사용 이유

자바 5부터 제네릭 타입이 새로 추가됨. 제네릭 타입을 이용함으로써 잘못된 타입이 사용될 수 있는 문제를 컴파일 과정에서 제거할 수 있게 되었으며, 컬렉션, 람다식, 스트림, NIO에서 널리 사용됨.

제네릭은 클래스와 인터페이스, 메서드 정의 시 타입을 파라미터로 사용할 수 있도록 함. 타입 파라미터는 코드 작성 시 구체적인 타입으로 대체되어 다양한 코드를 생성하도록 해줌. 제네릭을 사용하는 코드는 제네릭을 사용하지 않는 코드에 비해 다음과 같은 이점을 가지고 있음.

### 컴파일 시 강타입 체크 가능

자바 컴파일러는 코드에서 잘못 사용된 타입 때문에 발생하는 문제점을 제거하기 위해 제네릭 코드에 대해 강타입 체크를 함. 실행 시 타입 에러가 나는 것보다 컴파일 시 미리 타입을 강하게 체크해서 에러를 방지하는 것이 좋음.

### 타입 변환을 제거

제네릭을 사용하지 않으면 불필요한 타입 변환을 해야 하기 때문에 성능에 악영향을 미침.



## 제네릭 타입(class\<T\>, interface\<T\>)

제네릭 타입은 타입을 파라미터로 가지는 클래스 또는 인터페이스를 의미함. 제네릭 타입은 클래스 또는 인터페이스 이름 뒤에 "\<\>" 부호가 붙고, 사이에 타입 파라미터가 위치함.

```java
public class GenericClass<T> { ... }
public interface GenericInterface<T> { ... }
```

제네릭 타입을 실제 코드에서 사용하려면 타입 파라미터에 구체적인 타입을 지정해야 함.

Object 타입을 사용하면 모든 종류의 자바 객체를 필드에 저장할 수 있지만, 저장할 때 타입 변환이 발생하고 읽어올 때에도 타입 변환이 발생함. 이런 타입 변환은 전체 프로그램 성능에 영향이 있음. 그래서 모든 종류의 객체를 저장하면서 타입 변환이 발생하지 않도록 하는 방법이 필요했고, 제네릭을 이용하게 됨.

제네릭은 클래스를 설계할 때 구체적인 타입을 명시하지 않고 타입 파리미터로 대체했다가 실제 클래스가 사용될 때 구체적인 타입을 지정함으로써 타입 변환을 최소화함.



## 멀티 타입 파라미터(class\<K, V, ...\>, interface\<K, V, ...\>)

제네릭 타입은 두 개 이상의 멀티 타입 파라미터를 사용할 수 있음.

제네릭 타입 변수 선언과 객체 생성을 동시에 할 경우 구체적 타입을 지정하는 코드가 중복되어 복잡해질 수 있는데, 자바 7부터 제네릭 타입 파라미터의 중복 코드를 줄이기 위해 <> 연산자를 제공함. <> 연산자는 타입 파라미터를 유추하여 자동으로 설정해 줌.

```java
GenericClass<String, Integer> example = new GenericClass<String, Integer>();
GenericClass<String, Integer> example = new GenericClass<>(); // 자바 7부터 가능.
```



## 제네릭 메서드(\<T, R\> R method(T t))

제네릭 메서드는 파라미터 타입과 리턴 타입으로 제네릭 타입 파라미터를 갖는 메서드. 

```java
public <타입파라미터, ...> 리턴타입 메서드명(매개변수, ...) { ... }
```

제네릭 메서드는 두 가지 방식으로 호출할 수 있음. 코드 상에서 타입 파라미터의 구체적인 타입을 명시해도 되고 컴파일러가 매개값의 타입을 보고 추정하도록 할 수도 있음.

```java
리턴타입 변수 = <구체적타입> 메서드명(매개값);
리턴타입 변수 = 메서드명(매개값);
```



## 제한된 타입 파라미터(\< T extends 상위타입\>)

타입 파라미터에 지정되는 구체적인 타입을 제한해야 할 필요가 있을 경우 타입 파라미터 뒤에 extends 키워드를 붙이고 상위 타입을 명시하여 제한할 수 있음. 상위 타입은 클래스뿐 아니라 인터페이스도 가능하지만 인터페이스라고 해서 implements 키워드를 사용하지는 않음.

```java
public <T extends 상위타입> 리턴타입 메서드(매개변수, ...) { ... }
```

타입 파라미터를 제한할 경우 지정되는 구체적 타입은 상위 타입 또는 상위 타입의 하위 타입만 가능함. 주의할 점은 메서드 내에서 타입 파라미터 변수로 사용 가능한 것은 상위 타입의 멤버(필드, 메서드)로 제한됨. 하위 타입에만 있는 필드와 메서드는 사용할 수 없음.



## 와일드카드 타입(\<?\>, \<? extends ...\>, \<? super ...\>)

코드에서 `?`를 일반적으로 **와일드카드** 라고 부르는데, 제네릭 타입을 매개값이나 리턴 타입으로 사용 시 구체적 타입 대신 와일드카드를 다음과 같은 3가지 형태로 사용할 수 있음.

- 제네릭타입<?> : Unbounded Wildcards
- 제네릭타입<? extends 상위타입> : Upper Bounded Wildcards
- 제네릭타입<? super 하위타입> : Lower Bounded Wildcards



## 제네릭 타입의 상속과 구현

제네릭 타입도 다른 타입과 같이 상속이 가능함. 또한 자식 제네릭 타입은 부모 제네릭 타입의 타입 파라미터 외의 추가적으로 타입 파라미터를 가질 수 있음.

```java
public class ChildClass<T, M, C> extends ParentClass<T, M> { ... }
```
