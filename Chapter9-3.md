# 09. 제네릭스

- 이전 장에서 이미 살펴본 제네릭 설명
- 실체화한 타입 파라미터나 선언 지점 변성 등의 새로운 내용 설명
    - 실체화한 타입 파라미터를 사용하면 인라인 함수 호출에서 타입 인자로 쓰인 구체적인 타입을 실행 시점에 알 수 있다.
    - 선언 지점 변성을 사용하면 기저 타입은 같지만 타입 인자가 다른 두 제네릭 타입 Type<A>와 Type<B>가 있을 때 타입 인자 A와 B의 상위/하위 타입 관계에 따라 두 제네릭 타입의 상위/하위 타입 관계가 어떻게 되는지 지정할 수 있다.

## 제네릭 타입 파라미터

- 제네릭스를 사용하면 타입 파라미터를 받는 타입을 정의할 수 있다.
- 제네릭 타입의 인스턴스를 만들려면 타입 파라미터를 구체적인 타입 인자로 치환해야 한다.
- 타입 파라미터를 사용하면 “이 변수는 문자열을 담는 리스트다” 라고 말할 수 있다. → `List<String>`
- 코틀린 컴파일러는 보통 타입과 마찬가지로 타입 파라미터도 추론할 수 있다.
    - `val authors = listOf(”Dmitry”, “Svetlana”)` → `List<String>` 임을 추론
- 하지만 빈 리스트를 만들 때는 타입 인자를 추론할 근거가 없기 때문에 직접 타입 인자를 명시해야 한다. 변수의 타입을 지정해도 되고 변수를 만드는 함수의 타입 인자를 지정해도 된다.
    - `val readers: MutableList<String> = mutableListOf()`
    - `val readers: mutableListOf<String>()`

**note**

<aside>
💡 자바와 달리 코틀린에서는 제네릭 타입의 타입 인자를 프로그래머가 명시하거나 컴파일러가 추론할 수 있어야 한다.

- 자바는 리스트 원소 타입을 지정하지 않고 List 타입의 변수를 선언할 수도 있다. (1.5에서 제네릭을 도입            했기 때문에 하위 호환성 고려)
- 코틀린은 처음부터 제네릭을 도입했기 때문에 타입 인자를 항상 정의해야 한다.

</aside>

### 제네릭 함수와 프로퍼티

- 리스트를 다루는 함수를 작성한다면 모든 리스트를 다룰 수 있는 함수를 원할 것이다. 이때 제네릭 함수를 작성해야 한다.
- 컬렉션을 다루는 라이브러리 함수는 대부분 제네릭 함수다.
    
    `Example`
    
    ![스크린샷 2021-12-24 오후 2.20.06.png](09%20%E1%84%8C%E1%85%A6%E1%84%82%E1%85%A6%E1%84%85%E1%85%B5%E1%86%A8%E1%84%89%E1%85%B3%20e1dd4459a3fa492384a074baf11439cb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-12-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.20.06.png)
    
    - 이런 함수를 포함할 때 타입 인자를 명시적으로 지정할 수 있으나 대부분 컴파일러가 타입 인자를 추론할 수 있다.
        
        `리스트 9.1 제네릭 함수 호출하기`
        
        ```kotlin
        >>> val letters = ('a'..'z').toList()
        >>> println(letters.slice<Char>(0..2)) //타입 인자를 명시적으로 지정한다.
        [a,b,c]
        
        >>> println(letters.slice(10..13)) // T가 Char라는 사실을 추론한다.
        [k, l, m, n]
        ```
        
    
    `리스트 9.2 제네릭 고차 함수 호출하기`
    
    ```kotlin
    val authors = listOf("Dmitry", "Svetlana")
    val readers = mutalbleListOf<String>(/* ... */)
    
    fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>
    >>> readers.filter {it !in authors}
    ```
    
    - filter를 호출하는 readers의 타입이 List<String>이라는 것을 알고 이후로 T가 String이라는 사실을 추론한다.
    - 클래스나 인터페이스 안에 정의된 메소드, 확장 함수 또는 최상위 함수에서 타입 파라미터를 선언할 수 있다.

**note**

<aside>
💡 확장 프로퍼티만 제네릭하게 만들 수 있다.

일반(확장이 아닌) 프로퍼티는 타입 파라미터를 가질 수 없다.
클래스 프로퍼티에 여러 타입의 값을 저장할 수는 없으므로 제네릭한 일반 프로퍼티는 없다.
일반 프로퍼티를 제네릭하게 정의하면 컴파일러는 오류를 표시한다.

>>> `val <T> x: T = TODO()`
ERROR: type parameter of a property must be used in its receiver type

</aside>

### 제네릭 클래스 선언

- 자바와 마찬가지로 코틀린에서도 타입 파라미터를 넣은 꺽쇠 기호(<>)를 클래스 이름 뒤에 붙이면 클래스를 제네릭하게 만들 수 있다.
- 타입 파라미터를 이름 뒤에 붙이고 나면 클래스 본문 안에서 타입 파라미터를 다른 일반 타입처럼 사용할 수 있다.
- 제네릭 클래스를 확장하는 클래스를 정의하려면 기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해야 한다. → 구체적인 타입 or 타입 파라미터로 받은 타입 가능
    
    ```kotlin
    class stringList: List<String> {
    	override fun get(index: Int): String = ...
    }
    
    // ArrayList의 제네릭 타입 파라미터 T를 List의 타입 인자로 사용한다.
    class ArrayList<T>: List<T> {   
    	override fun get(index: Int): T =...
    }
    ```
    
    - ArrayList 클래스는 자신만의 타입 파라미터 T를 정의하면서 그 T를 기반 클래스의 타입 인자로 사용한다.
        - ArrayList<T>에서 T는 List<T>의 T와 전혀 다른 타입 파라미터이며, 실제로는 T가 아니라 다른 이름을 사용해도 의미에는 차이가 없다.
    - StringList 클래스는 String 타입의 원소만을 포함한다. 따라서 String을 기반 타입의 타입 인자로 지정한다. 하위 클래스에서 상위 클래스에 정의된 함수를 오버라이드하거나 사용하려면 타입 인자 T를 구체적 타입 String으로 치환해야 한다.
        
        ```kotlin
        override fun get(index: Int): String {
        	TODO("Not yet implemented")
        }
        ```
        
- 클래스가 자기 자신을 타입 인자로 참조할 수도 있다.
    
    ```kotlin
    interface Comparable<T> {
    	fun compareTo(other: T): Int
    }
    
    class String: Comparable<String> {
    	override fun compareTo(other: String): Int = /* ... */
    }
    ```
    
    - String 클래스는 제네릭 Comparable 인터페이스를 구현하면서 그 인터페이스의 타입 파라미터 T로 String 자신을 지정한다.

지금까지의 제네릭스는 자바 제네릭스와 비슷하나 이후에는 자바와 다른 점에 대해 설명한다.

### 타입 파라미터 제약

- 타입 파라미터 제약은 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능이다.
- List<Int>나 List<Double>에 sum함수를 적용할 수 있지만 List<String>에는 적용할 수 없다.
- 어떤 타입을 제네릭 타입의 타입 파라미터에 대한 상한으로 지정하면 그 제네릭 타입을 인스턴스화할 때 사용하는 타입 인자는 반드시 그 상한 타입이거나 그 상한 타입의 하위 타입이어야 한다.
- 제약을 가하려면 타입 파라미터 이름 뒤에 콜론(:)을 표시하고 그 뒤에 상한 타입을 적으면 된다.
    
    `fun **<T : Number>** List<T>.sum() : T`
    
- 타입 파라미터 T에 대한 상한을 정하고 나면 T타입의 값을 그 상한 타입의 값으로 취급 가능하다.
    
    ```kotlin
    fun <T: Number> oneHalf(value: T): Double {
        return value.toDouble() / 2.0
    }
    ```
    
- 타입 파라미터에 둘 이상의 제약을 가해야 하는 경우도 있다.
    
    ```kotlin
    fun <T> ensureTrailingPeriod(seq: T)
            where T : CharSequence, T : Appendable {
        if (!seq.endsWith('.')) {
            seq.append('.')
        }
    }
    
    >>> val helloWorld = StringBuilder("Hello World")
    >>> ensureTrailingPeriod(helloWorld)
    >>> println(helloWorld)
    Hello World.
    
    ```
    

### 타입 파라미터를 널이 될 수 없는 타입으로 한정

- 제네릭 클래스나 함수를 정의하고 그 타입을 인스턴스화할 때는 널이 될 수 있는 타입을 포함하는 어떤 타입으로 타입 인자를 지정해도 타입 파라미터를 치환할 수 있다.
- <T: Any>라는 제약은 T 타입이 항상 널이 될 수 없는 타입이 되게 보장한다.
    
    ```kotlin
    class Processor<T> {
        fun process(value: T) {
            value.hashCode()
        }
    }
    
    // 해당 코드 사용 가능, 즉 null이 될 수 있음
    val nullableStringProcessor = Processor<String?>()
    nullableStringProcessor.process(null)
    
    class Processor<T: Any> {
        fun process(value: T) {
            value.hashCode()
        }
    }
    
    // 해당 코드 사용 불가
    val nullableStringProcessor = Processor<String?>()
    nullableStringProcessor.process(null)
    
    // 컴파일 오류
    Error: Type argument is not within its bounds: should be subtype of 'Any'
    ```
    
- 타입 파라미터를 널이 될 수 없는 타입으로 제약하기만 하면 타입 파라미터는 널이 될 수 없다. 즉 Any가 아니더라도 다른 타입을 사용해도 된다.

## 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터

- JVM의 제네릭스는 보통 타입 소거(type erasure)를 사용해 구현된다. 이는 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않다는 뜻이다.
- 이 절에서는 코틀린 타입 소거가 실용적인 면에서 어떤 영향을 끼치는지 살펴보고 함수를 inline으로 선언함으로써 이런 제약을 어떻게 우회하는지 살펴본다. (inline으로 선언 시 타입 인자자 지워지지 않게 함 → **실체화**)

### 실행 시점의 제네릭: 타입 검사와 캐스트

- 자바와 마찬가지로 코틀린 제네릭 타입 인자 정보는 런타임에 지워진다. 즉 인스턴스를 생성할 때 쓰인 타입 인자에 대한 정보를 유지하지 않는다.
    - List<String> 객체를 만들고 그 안에 문자열을 넣더라도 실행 시점에 오직 List로만 볼 수 있다.
    - List 객체가 어떤 타입의 원소를 저장하는지 실행 시점에는 알 수 없다.
        
        ![스크린샷 2021-12-24 오후 4.50.01.png](09%20%E1%84%8C%E1%85%A6%E1%84%82%E1%85%A6%E1%84%85%E1%85%B5%E1%86%A8%E1%84%89%E1%85%B3%20e1dd4459a3fa492384a074baf11439cb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-12-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_4.50.01.png)
        

**타입 소거의 한계**

- 실행 시점에 타입 인자를 검사할 수 없다, 단 List인지 까지는 알아낼 수 있다. (문자열 인지, 다른 객체 인지는 x)
    
    ```kotlin
    if (value is List<String>) {...}
    
    ERROR: Cannot check for instance of erased type
    ```
    
- 저장해야 하는 타입 정보의 크기가 줄어들어서 전반적인 메모리 사용량이 줄어든다는 장점이 있다.
- 리스트인지 맵인지 등의 정보는 스타 프로젝션(start projection)을 사용하면 된다. 타입 파라미터가 2개 이상이면 모든 타입 파라미터에 * 포함
    
    `if (value is List<*>) {...}`
    
- as나 as? 캐스팅에도 제네릭 타입을 사용할 수 있다. 하지만 다른 타입으로 캐스팅해도 캐스팅에 성공한다. 하지만 List<String> 으로 만들경우  String을 Number 사용하려고 하면 ClassCastException이 발생한다.

```kotlin
fun printSum(c: Collection<*>) {
    val intList = c as? List<Int>
        ?: throw IllegalArgumentException("List is expected")
    println(intList.sum())
}
```

- 코틀린 컴파일러는 컴파일 시점에 타입 정보가 주어진 경우에는 is 검사를 수행하게 허용한다.
    
    ```kotlin
    fun printSum(c: Collection<Int>) {
        if (c is List<Int>) {
            println(c.sum())
        }
     }
    ```
    
    - c 컬렉션이 Int 값을 저장한다는 사실이 알려져 있으므로 c가 List<int>인지 검사할 수 있다.

코틀린은 제네릭 함수의 본문에서 그 함수의 타입 인자를 가리킬 수 있는 특별한 기능은 없지만 inline 함수 안에서는 타입 인자를 사용할 수 있다.

### 실체화한 타입 파라미터를 사용한 함수 선언

- 코틀린 제네릭 타입의 타입 인자 정보는 실행 시점에 지워지므로 제네릭 함수가 호출되도 그 함수의 본문에서는 호출 시 쓰인 타입 인자를 알 수 없다.
    
    `fun <T> isA(value: Any) value is T`
    
- 하지만 inline 키워드를 붙이면 컴파일러는 그 함수를 호출한 식을 모두 함수 본문으로 바꾼다. 이로써 실행 시점에 타입이 T의 인스턴스인지를 검사할 수 있다.
    
    ```kotlin
    // reified 키드는 이 타입 파라미터가 실행 시점에 지워지지 않음을 표시한다.
    inline fun <reified T> isA(value: Any) = value is T
    ```
    

`리스트 9.9 filterIsInstance를 간단하게 정리한 버전`

```kotlin
inline fun <reified T> Iterable<*>.filterIsInstance(): List<T> {
    val destination = mutableListOf<T>()
    for (element in this) {
        if (element is T) {
            destination.add(element)
        }
    }
    return destination
}
```

<aside>
💡 **인라인 함수에서만 실체화한 타입 인자를 쓸 수 있는 이유**

컴파일러는 인라인 함수의 본문을 구현한 바이트코드를 그 함수가 호출되는 모든 지점에 삽입한다. 컴파일러는 실체화한 타입 인자를 사용해 인라인 함수를 호출하는 각 부분의 정확한 타입 인자를 알 수 있다.

즉, 아래와 같은 코드를 만들어낸다.
인라인 함수는 자바에서 일반 함수처럼 호출할 수는 없다.

</aside>

```kotlin
for (element in this) {
	if (element is String) {
		destination.add(element)
	}
}
```

### 실체화한 타입 파라미터로 클래스 참조 대신

- java.lang.Class 타입 인자를 파라미터로 받는 API에 대한 코틀린 어댑터를 구축하는 경우 실체화한 타입 파라미터를 자주 사용한다.
- ServiceLoader는 어떤 추상 클래스나 인터페이스를 표현하는 java.lang.Class를 받아서 그 클래스나 인스턴스를 구현한 인스턴스를 반환한다.
    
    `val serviceImpl = ServiceLoader.load(Service::class.java)`
    
- loadService 함수 정의
    
    ```kotlin
    inline fun <reified T> loadService() {
    	return ServiceLoader.load(T::class.java)
    }
    ```
    

### 실체화한 타입 파라미터의 제약

다음과 같은 경우에 실체화한 타입 파라미터를 사용할 수 있다.

- 타입 검사와 캐스팅(is, !is, as, as?)
- 10장에서 설명할 코틀린 리플렉션 API(::class)
- 코틀린 타입에 대응는 java.lang.Class를 얻디(::class.java)
- 다른 함수를 호출할 때 타입 인자로 사용

하지만 다음과 같은 일은 할 수 없다.

- 타입 파라미터 클래스의 인스턴스 생성하기
- 타입 파라미터 클래스의 동반 객체 메소드 호출하기
- 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
- 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 reified로 지정하기

## 변성: 제네릭과 하위 타입

- 변성 개념은 List<String>와 List<Any>와 같이 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념이다.
- 일반적으로 이런 관계가 왜 중요한지 먼저 설명한 다음에 코틀린에서 변성을 어떻게 표시하는지 살펴본다.
- 직접 제네릭 클래스나 함수를 정의하는 경우 변성을 꼭 이해해야 한다.
- 변성을 잘 활용하면 사용에 불편하지 않으면서 타입 안전성을 보장하는 API를 만들 수 있다.

### 변성이 있는 이유: 인자를 함수에 넘기기

- List<Any> 타입의 파라미터를 받는 함수에 List<String>을 넘기면 안전할까?
    - 원소 추가나 변경이 없는 경우에는 List<String>을 List<Any> 대신 넘겨도 안전하다.
    - 하지만 어떤 함수가 리스트의 원소를 추가하거나 변경한다면 타입 불일치가 생길 수 있다.
    - 코틀린에서는 리스트의 변경 가능성에 따라 적절한 인터페이스를 선택하면 안전하지 못한 함수 호출을 막을 수 있다. 함수가 읽기 전용 리스트를 받는다면 더 구체적인 타입의 원소를 갖는 리스트를 그 함수에 넘길 수 있다. 하지만 리스트가 변경 가능하다면 그럴 수 없다.
    
    ```kotlin
    fun main() {
        printContents(listOf("abc", "bac"))
        val strings = mutableListOf("abc", "bac")
        addAnswer(strings)
    }
    
    // 변경 가능 리스트를 받기 위해서는 특정 타입으로 지정해야 한다.
    // list: MutableList<Int>
    fun addAnswer(list: MutableList<Any>) {
        list.add(42)
    }
    ```
    
    - 이후에 List와 MutableList의 변성이 왜 다른지 살펴본다. 그 내용을 이해하기 위해 **타입**과 **하위 타입**이라는 개념을 알아야 한다.

### 클래스, 타입, 하위 타입

- 변수의 타입은 그 변수에 담을 수 있는 값의 집합을 지정한다.
- 클래스와 타입은 같지 않다.
    - 하나의 클래스는 적어도 둘 이상의 타입을 구성할 수 있다. → `var x: String, var x: String?`
    - 제네릭은 더 복잡하다. List는 타입이 아니라 클래스다. 하지만 타입 인자를 치환한 List<Int>, List<String?> List<List<String>> 등은 모두 타입이다.
- 타입 사이의 관계를 논하기 위해 하윕 타입을 알아야 한다. 어떤 타입 A의 값이 필요한 모든 장소에 어떤 타입 B의 값을 넣어도 아무 문제가 없다면 타입 B는 타입 A의 하위 타입이다.
    
    ![스크린샷 2021-12-27 오후 8.10.29.png](09%20%E1%84%8C%E1%85%A6%E1%84%82%E1%85%A6%E1%84%85%E1%85%B5%E1%86%A8%E1%84%89%E1%85%B3%20e1dd4459a3fa492384a074baf11439cb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-12-27_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.10.29.png)
    
- 하위 타입인지는 컴파일러가 변수 대입이나 함수 인자 전달 시 하위 타입 검사를 매번 수행하기 때문에 중요하다.
- **하위 타입**과 **하위 클래스**는 근본적으로 같다. Int 클래스는 Number의 하위 클래스이므로 Int는 Number의 하위 타입이다. 동일하게 어떤 인터페이스를 구현하는 클래스의 타입은 그 인터페이스 타입의 하위 타입이다.
- 널이 될 수 없는 타입은 널이 될 수 있는 타입의 하위 타입이다. 하지만 두 타입 모두 같은 클래스에 해당한다.
    
    ![스크린샷 2021-12-27 오후 8.18.20.png](09%20%E1%84%8C%E1%85%A6%E1%84%82%E1%85%A6%E1%84%85%E1%85%B5%E1%86%A8%E1%84%89%E1%85%B3%20e1dd4459a3fa492384a074baf11439cb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-12-27_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_8.18.20.png)
    
- 제네릭 타입을 다룰 때 하위 클래스와 하위 타입의 차이는 더욱 중요해진다. 제네릭 타입을 인스턴스화할 때 타입 인자가 서로 다른 타입이 들어가면 인스턴스 타입 사이의 하위 타입 관계가 성립하지 않으면 그 제네릭 타입을 **무공변(invariant)**이라고 말한다.
    - 즉 MutableList<A>는 항상 MutableList<B>의 하위 타입이 아니다.
    - `Example` → MutableList<Any> MutableList<String>은 서로 하위 타입이 아니다.
- 하지만 코틀린의 List 인터페이스는 읽기 전용 컬렉션을 표현한다. 즉 A가 B의 하위 타입이면 List<A>는 List<B>의 하위 타입이다. 그런 클래스나 인터페이스를 **공변적(covariant)**이라고 말한다.
- 자바에서는 읽기 전용 타입이 존재하지 않기 때문에 모든 클래스가 무공변이다.

### 공변성: 하위 타입 관계를 유지

- 코틀린에서 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 이름 앞에 out을 넣어야 한다.
    
    ```kotlin
    interface Producer<out T> { // 클래스가 T에 대해 공변적이라고 선언한다.
    	fun produce(): T
    }
    ```
    
- 클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않더라도 그 클래스의 인스턴스를 함수 인자나 반환 값으로 사용할 수 있다.
    
    `리스트 9.11 무공변 컬렉션 역할을 하는 클래스 정의하기`
    
    ```kotlin
    open class Animal {
        fun feed() {}
    }
    
    class Herd<T: Animal> {
        val size: Int get() =  ...
        operator fun get(i: Int): T {
    				...
        }
    }
    
    fun feedAll(animals: Herd<Animal>) {
        for(i in 0 until animals.size) {
            animals[i].feed()
        }
    }
    ```
    
    `리스트 9.12 무공변 컬렉션 역할을 하는 클래스 사용하기`
    
    ```kotlin
    class Cat: Animal() {
        fun cleanLitter() {}
    }
    
    fun takeCareOfCats(cats: Herd<Cat>) {
        for (i in 0 until cats.size) {
            cats[i].cleanLitter()
            // feedAll(cats) 
    				// Error: inferred type is Herd<Cat>, but Herd<Animal> 
    				// was expected 라는 오류가 발생한다.
        }
    }
    ```
    
    - Herd 클래스의 T 타입 파라미터에 대해 아무 변성도 지정하지 않았기 때문에 고양이 무리는 동물 무리의 하위 클래스가 아니다.
        - 명시적으로 타입 캐스팅을 할 수 있긴 하지만 코드가 장황해지고 실수 하기 쉽다. 또한 강제 캐스팅은 올바른 방법이 아니다.
    - 이를 해결하기 위해 Herd를 공변적인 클래스로 만들 수 있다.
        
        `리스트 9.13 공변적 컬렉션 역할을 하는 클래스 사용하기`
        
        ```kotlin
        class Herd<out T: Animal> {}
        fun takeCareOfCats(cats: Herd<Cat>) {
            for (i in 0 until cats.size) {
                cats[i].cleanLitter()
                feedAll(cats) 
        }
        ```
        
- 타입 파라미터를 공변적으로 지정하면 클래스 내부에서 그 파라미터를 사용하는 방법을 제한한다.
- 즉, 타입 안전성을 보장하기 위해 공변적 파라미터는 항상 아웃 위치에만 있어야 한다. 이는 클래스가 T 타입의 값을 생산할 수는 있지만 T 타입의 값을 소비할 수는 없다는 뜻이다.
- 함수 파라미터 타입은 인 위치, 반환 타입은 아웃 위치에 해당한다.
    
    ![스크린샷 2021-12-27 오후 9.05.59.png](09%20%E1%84%8C%E1%85%A6%E1%84%82%E1%85%A6%E1%84%85%E1%85%B5%E1%86%A8%E1%84%89%E1%85%B3%20e1dd4459a3fa492384a074baf11439cb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-12-27_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.05.59.png)
    
- 코틀린의 List는 읽기 전용이다. 따라서 그 안에는 T 타입의 원소를 반환하는 get 메소드는 있지만 리스트에 T 타입의 값을 추가하거나 리스트에 있는 기존 값을 변경하는 메소드는 없다. 따라서 List는 T에 대해 공변적이다.
    
    ```kotlin
    interface List<out E> : Collection<E> {
    	operator fun get(index: Int): T // T는 항상 아웃 위치에 쓰인다.
    
    	// public abstract fun indexOf(element: E): kotlin.Int 이것은 무엇??
    }
    ```
    
- 이런 위치 규칙은 오직 외부에서 볼 수 있는 (public, protected, internal) 클래스 API에만 적용할 수 있다. 비공개(private) 메소드의 파라미터는 인도 아니고 아웃도 아닌 위치다.

### 반공변성: 뒤집힌 하위 타입 관계

- 반공변성은 공변성의 반대라 할 수 있다. 반공변 클래스의 하위 타입 관계는 공변 클래스의 경우와 반대다.
    
    ```kotlin
    interface Comparator<in T> {
    	fun compare(e1: T, e2: T): Int {...}
    }
    ```
    
    - 이 인터페이스의 메소드는 T 타입의 값을 소비하기만 한다. 이는 T가 인 위치에서만 쓰인다는 뜻이다. 따라서 T 앞에는 in 키워드를 붙여야만 한다.

### 사용 지점 변성: 타입이 언급되는 지점에서 변성 지정

### 스타 프로젝션: 타입 인자 대신 * 사용