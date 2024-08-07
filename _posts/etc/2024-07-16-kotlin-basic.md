---
title: "Kotlin 기본 문법"
excerpt: "Java와 차이나는 문법 위주로 정리"

categories:
  - ETC
tags:
  - [Kotlin]

permalink: /etc/kotlin-basic

toc: true
toc_sticky: true

date: 2024-07-16
last_modified_at: 2024-07-21
---

# 1. 타입
## 1-1. 기본 타입

| 타입 종류     |	타입 이름|	설명|	예시 코드|
|-----------|---|---|---|
| 정수 타입     |	Byte|	8비트 정수|	val byteValue: Byte = 1|
|           |Short|	16비트 정수|	val shortValue: Short = 2|
|           |Int|	32비트 정수|	val intValue: Int = 3|
|           |Long|	64비트 정수|	val longValue: Long = 4L|
| 부동 소수점 타입 |	Float|	32비트 부동 소수점|	val floatValue: Float = 1.23F|
|           |Double|	64비트 부동 소수점|	val doubleValue: Double = 4.56|
| 문자 타입     |	Char|	단일 문자|	val charValue: Char = 'A'|
| 문자열 타입    |	String|	문자열|	val stringValue: String = "Hello"|
| 논리 타입     |	Boolean|	참 또는 거짓|	val booleanValue: Boolean = true|
| 배열 타입     |	Array|	배열|	val arrayValue: Array<Int> = arrayOf(1, 2, 3)|
| 컬렉션 타입    |	List|	읽기 전용 리스트|	val list: List<Int> = listOf(1, 2, 3)|
|           |MutableList|	읽기/쓰기 가능한 리스트|	val mutableList: MutableList<Int> = mutableListOf(4, 5, 6)|
|           |Set|	읽기 전용 집합|	val set: Set<Int> = setOf(1, 2, 3)|
|           |MutableSet|	읽기/쓰기 가능한 집합|	val mutableSet: MutableSet<Int> = mutableSetOf(4, 5, 6)|
|           |Map|	읽기 전용 맵|	val map: Map<String, Int> = mapOf("one" to 1, "two" to 2)|
|           |MutableMap|	읽기/쓰기 가능한 맵|	val mutableMap: MutableMap<String, Int> = mutableMapOf("three" to 3, "four" to 4)|

## 1-2. Any

``` kotlin
val anyValue: Any = "Hello"
```

- 최상위 타입, 모든 타입의 루트

## 1-3. Unit

``` kotlin
fun printMessage() : Unit { println("Hello") }
```

- 반환값이 없는 함수의 반환 타입
- void 반환과 유사하지만, Unit 반환은 실제 Unit 객체가 반환된다.

## 1-4. Nothing

``` kotlin
fun fail(message: String): Nothing { throw IllegalArgumentException(message) }
```

- 함수가 정상적으로 끝나지 않음을 나타내는 타입

<br>

# 2. 변수 선언

``` kotlin
val a: Int = 1 // read-only
var b: Int = 2

val c = 3 // read-only
var d = 4
```

- 자바의 경우 Java 10부터 `var` 키워드를 사용하여 지역 변수에 대해서만 타입을 생략할 수 있다.
- 코틀린은 멤버 변수에서도 타입 생략이 가능하다. 타입을 생략하면 컴파일러가 초기화 값에 따라 타입을 추론한다.
- immutable 변수는 val, mutable 변수는 var

<br>

# 3. 문자열 템플릿

``` kotlin
println("Sum of $a and $b is ${a + b}")
```

- `$변수명`: 변수의 값을 삽입
- `${표현식}`: 표현식의 결과 값을 삽입

<br>

# 4. 함수
## 4-1. 기본 함수 정의

``` kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

## 4-2. 함수의 리턴 타입 생략
- 코틀린에서 함수의 리턴 타입을 생략할 수 있는 경우는 아래 두 가지가 있다

### 4-2-1. 함수가 `Unit` 타입을 반환하는 경우

``` kotlin
fun printSum(a: Int, b: Int) {
    println("Sum of $a and $b is ${a + b}")
}
```

- 위 함수는 리턴 타입을 명시하지 않았지만, 컴파일러는 해당 함수가 아무 값도 반환하지 않고 Unit 타입을 반환하는 것으로 추론한다.
- Unit은 자바의 void와 유사하지만, 실제로 반환하는 값이 있는 표현이다.
- 자바의 void는 실제 타입이 아니며, 반환 값이 없다는 사실을 나타내기 위한 키워드이다.
- 코틀린의 Unit은 반환 값이 없음을 나타내는 타입이다. 함수는 실제 Unit 객체를 반환할 수 있다.

### 4-2-2. 리턴 타입을 컴파일러가 추론할 수 있는 경우

``` kotlin
fun sum(a: Int, b: Int) = a + b
```

## 4-5. 함수 타입

``` kotlin
// 함수 타입 정의
val sum: (Int, Int) -> Int = { a, b -> a + b }
val greet: () -> String = { "Hello" } // 파라미터가 없는 경우
val printNumber: (Int) -> Unit = { number -> println(number) } // 반환값이 없는 경우

// 함수 타입 사용 (1)
val multiply: (Int, Int) -> Int = { x, y -> x * y }
println(multiply(4, 2))

// 함수 타입 사용 (2)
fun add(a: Int, b: Int): Int = a + b

val sum: (Int, Int) -> Int = ::add
println(sum(3, 5))

// 함수 타입 사용 (3)
typealias Operation = (Int, Int) -> Int

val subtract: Operation = { a, b -> a - b }
println(subtract(10, 4))
```

- 코틀린에서는 자바와 다르게 함수형 인터페이스가 필요 없으며, 위와 같이 간단하게 함수 타입을 사용할 수 있다.
- `(파라미터 타입1, 파라미터 타입2, ...) -> 반환 타입`으로 정의된다

<br>

# 5. Null Safety

``` kotlin
var a: String = "abc"
var b: String? = "abc"
```

- 코틀린에서 기본적으로 모든 변수는 null 값을 가질 수 없다.
- null 값을 허용하려면 타입 뒤에 `?`를 붙여야 한다.

<br>

# 6. 스마트 캐스트 (Smart Casts)

``` kotlin
fun demo(x: Any) {
    if (x is String) {
        // x는 자동으로 String 타입으로 캐스트된다
    }
}
```

- 변수의 타입을 확인한 후, 해당 타입으로 캐스팅해주는 기능
- if 블록 내부에서는 캐스팅된 타입으로 동작하고, 바깥에서는 Any 타입으로 동작한다.

``` kotlin
val l = b?.length
```

- null 값이 허용된 변수를 사용할 때는 안전한 호출 연산자(`.?`)를 사용한다
- 안전한 호출 연산자(`.?`)는 변수의 값이 null이 아닌 경우에만 해당 속성이나 매서드에 접근하고, null인 경우에는 null을 반환한다.

<br>

# 7. if 표현식

``` kotlin
fun maxOf(a: Int, b: Int) = if (a > b) a else b
```

- if를 statement 뿐만 아니라 expression(표현식)으로 사용 가능

<br>

# 8. when 표현식

``` kotlin
fun describe(obj: Any): String =
    when (obj) {
        1          -> "One"
        "Hello"    -> "Greeting"
        is Long    -> "Long"
        !is String -> "Not a string"
        else       -> "Unknown"
    }
```

- 자바의 switch 문과 유사하지만, 다양한 조건을 처리할 수 있으며 표현식으로 사용 가능

<br>

# 9. for 루프

``` kotlin
val items = listOf("apple", "banana", "kiwi")
for (item in items) {
    println(item)
}
```

- 기본 for 루프는 for-each 문으로 사용

``` kotlin
val items = listOf("apple", "banana", "kiwi")
for (index in items.indices) {
    println("Item at $index is ${items[index]}")
}

for ((index, value) in items.withIndex()) {
    println("Item at $index is $value")
}
```

- 인덱스를 사용할 경우에는 indices 속성 혹은 withIndex()를 사용

<br>

# 10. 접근 제어자
- public: 접근 제어자 생략 시 기본으로 설정
- internal: 같은 모듈 내에서만 접근 가능
- protected: 서브클래스 내에서만 접근 가능
- private: 해당 선언을 포함하는 클래스 내에서만 접근 가능

<br>

# 11. 주 생성자(primary constructor)/보조 생성자(secondary constructor)

``` kotlin
class Person(val name: String, var age: Int) {
    
    constructor(name: String) : this(name, 0) {
        // 추가 초기화 코드
    }
}
```

- 주 생성자는 클래스 선언 시 생성자에 멤버 변수를 포함할 수 있으며 클래스 헤더에 정의된다.
- 보조 생성자는 constructor 키워드를 사용하여 정의되며, 주 생성자를 호출하여 초기화한다.

``` kotlin
class Person(val name: String = "Unknown", var age: Int = 0)
```

- 생성자에서 기본값을 설정할 수 있다. 기본값을 설정하면 객체 생성 시 인자를 생략할 수 있다.

<br>

# 12. 인스턴스 생성

``` kotlin
val person = Person("John", 30)
```

- 인스턴스를 생성할 때 `new` 키워드를 사용하지 않는다.

<br>

# 13. 확장 함수 (Extension Function)

``` kotlin
fun String.addExclamation(): String {
    return this + "!"
}

val message = "Hello"
println(message.addExclamation()) // Hello!
```

- `클래스명.함수명` 형태로 선언한다.
- 해당 클래스의 인스턴스에서 마치 원래 클래스에 정의된 멤버 함수처럼 호출할 수 있다.
- 확장 함수가 멤버 함수와 같은 이름으로 정의된 경우, 멤버 함수가 우선한다.
- 확장 함수는 클래스의 private 멤버에 접근할 수 없다.
- 확장 함수는 인스턴스 메서드처럼 동작하며, 클래스의 정적(static) 메서드에 대한 확장은 지원되지 않는다.
- 동일한 확장 함수를 여러 파일에서 정의하는 경우, 사용 시점의 네임스페이스에 따라 결정된다.
- 확장 함수는 컴파일 시 어떻게 변환될까?

``` kotlin
fun String.isPalindrome(): Boolean {
    return this == this.reversed()
}
```

- 위 처럼 정의된 확장함수가 있을 때, 컴파일러는 컴파일 시 isPalindrome 확장 함수를 아래와 같은 정적 메서드로 변환한다.

``` kotlin
public static boolean isPalindrome(String str) {
    return str.equals(new StringBuilder(str).reverse().toString());
}
```

- 그리고 확장 함수의 호출은 컴파일 시 아래와 같이 변환된다.

``` kotlin
isPalindrome(str) // str.isPalindrome()
```

<br>

# 14. 확장 프로퍼티 (Extension Property)

``` kotlin
val String.lastChar: Char
    get() = this[this.length - 1]
    
println("Kotlin".lastChar) // n
```

``` kotlin
var <T> MutableList<T>.lastElement: T
    get() = this[this.size - 1]
    set(value) {
        this[this.size - 1] = value
}
```

- 확장 프로퍼티는 상태를 저장할 수 없고, 단지 계산된 값을 반환하는 용도로만 사용될 수 있다.
- 확장 프로퍼티는 `val` 또는 `var`로 선언할 수 있으며, getter와 setter를 정의할 수 있다.
- 확장 함수와 확장 프로퍼티는 컴파일 타임에 정적으로 처리되며, 실제 클래스의 메서드나 프로퍼티로 추가되는 건 아니다.

<br>

# 15. 싱글톤 객체 선언

``` kotlin
object Database {
    fun connect() {
        println("Connected to database")
    }
}

Database.connect()
```

- object 키워드를 사용하여 정의된 객체는 단 하나의 인스턴스만 존재한다. 이를 통해 싱글톤 패턴을 객체 선언으로 구현할 수 있다.
- 인스턴스를 생성할 필요 없이, 정의된 객체를 직접 사용할 수 있다.
- 생성자가 없으며, 객체 초기화는 객체 블록 내에서 수행된다.
- 초기화 시점은 첫 번째 접근 시이다.
- 기본적으로 스레드 안전한 성질을 가진다.

<br>

# 16. 데이터 클래스 (Data Classes)

``` kotlin
data class User(val name: String, val age: Int)
```

- 데이터 클래스는 자동으로 equals(), hashCode(), toString(), copy() 메서드를 생성

<br>

# 17. 람다식(Lambda Expression)

``` kotlin
val sum = { x: Int, y: Int -> x + y }
println(sum(1, 2))

fun operate(x: Int, y: Int, op: (Int, Int) -> Int): Int {
    return op(x, y)
}

val result = operate(3, 4, sum)
println(result)
```

- 람다 표현식은 중괄호 `{}`로 감싸야 한다.

``` kotlin
names.forEach{ println(it) }
```

- 파라미터가 하나일 때는 `it` 키워드를 사용할 수 있다.

<br>

# 18. 부모 클래스의 기본 생성자 호출
- 클래스 상속 시 부모 클래스의 기본 생성자를 호출해야 한다.
- 자바와 다르게 코틀린에서는 부모 클래스의 생성자를 명시적으로 호출해주어야 부모 클래스의 초기화가 제대로 이루어진다.

``` kotlin
open class Parent(val name: String)
class Child(name: String) : Parent(name)
```

- 위 코드에서 Child 클래스는 Parent 클래스의 생성자를 호출하면서 name 매개변수를 전달한다. 이는 상속 구조에서 부모 클래스의 초기화를 보장한다.

<br>

# 19. open

``` kotlin
open class Parent(val name: String)
```

- 코틀린에서는 클래스와 메서드가 기본적으로 상속이 불가능하게 설계되어 있다.
- open 키워드를 사용해야 해당 클래스나 메서드를 상속할 수 있게 된다.

<br>

# 20. by
- by 키워드는 위임(Delegation) 패턴을 구현하는 데 사용
- 위임 프로퍼티(Delegated Properties):
  - 프로퍼티의 게터와 세터를 직접 구현하지 않고, 위임 객체(delegate)에게 맡깁니다.
- 인터페이스 위임(Interface Delegation):
  - 클래스가 특정 인터페이스의 구현을 다른 객체에게 위임할 수 있습니다.