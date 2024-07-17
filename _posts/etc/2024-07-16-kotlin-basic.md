---
title: "Kotlin 기본 문법"
excerpt: ""

categories:
  - ETC
tags:
  - [Kotlin]

permalink: /etc/kotlin-basic

toc: true
toc_sticky: true

date: 2024-07-16
last_modified_at: 2024-07-17
---

1. 변수 선언

``` kotlin
val a: Int = 1 // read-only
var b: Int = 2

val c = 3 // read-only
var d = 4
```

- 자바의 경우 Java 10부터 `var` 키워드를 사용하여 지역 변수에 대해서만 타입을 생략할 수 있다.
- 코틀린은 멤버 변수에서도 타입 생략이 가능하다. 타입을 생략하면 컴파일러가 초기화 값에 따라 타입을 추론한다.
- immutable 변수는 val, mutable 변수는 var

2. 함수 선언

``` kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

3. 함수의 리턴 타입 생략
- 코틀린에서 함수의 리턴 타입을 생략할 수 있는 경우는 아래 두 가지가 있다

3-1. 함수가 `Unit` 타입을 반환

``` kotlin
fun printSum(a: Int, b: Int) {
    println("Sum of $a and $b is ${a + b}")
}
```

- 위 함수는 리턴 타입을 명시하지 않았지만, 컴파일러는 해당 함수가 아무 값도 반환하지 않고 Unit 타입을 반환하는 것으로 추론한다.
- Unit은 자바의 void와 유사하지만, 실제로 반환하는 값이 있는 표현이다.
- 자바의 void는 실제 타입이 아니며, 반환 값이 없다는 사실을 나타내기 위한 키워드이다.
- 코틀린의 Unit은 반환 값이 없음을 나타내는 타입이다. 함수는 실제 Unit 객체를 반환할 수 있다.

3-2. 리턴 타입을 컴파일러가 추론할 수 있을 때

``` kotlin
fun sum(a: Int, b: Int) = a + b
```

3. if 표현식

``` kotlin
fun maxOf(a: Int, b: Int) = if (a > b) a else b
```

- if를 statement 뿐만 아니라 expression(표현식)으로 사용 가능

4. when 표현식

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

5. for 루프

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

6. Null Safety

``` kotlin
var a: String = "abc"
var b: String? = "abc"
```

- 코틀린에서 기본적으로 모든 변수는 null 값을 가질 수 없다. 
- null 값을 허용하려면 타입 뒤에 `?`를 붙여야 한다.

``` kotlin
val l = b?.length
```

- null 값이 허용된 변수를 사용할 때는 안전한 호출 연산자(`.?`)를 사용한다

6. 주 생성자(primary constructor)/보조 생성자(secondary constructor)

``` kotlin
class Person(val name: String, var age: Int) {
    
    constructor(name: String) : this(name, 0) {
        // 추가 초기화 코드
    }
}
```

- 주 생성자는 클래스 헤더에 정의되며, 필드를 초기화한다.
- 보조 생성자는 constructor 키워드를 사용하여 정의되며, 주 생성자를 호출하여 초기화한다.

7. 인스턴스 생성

``` kotlin
val person = Person("John", 30)
```

- 인스턴스를 생성할 때 `new` 키워드를 사용하지 않는다.

8. 확장 함수 (Extension Function)

``` kotlin
fun String.addExclamation(): String {
    return this + "!"
}

val message = "Hello"
println(message.addExclamation()) // Hello!
```

9. 싱글톤 객체 선언

``` kotlin
object Database {
    fun connect() {
        println("Connected to database")
    }
}

Database.connect()
```

8. 스마트 캐스트 (Smart Casts)

``` kotlin
fun demo(x: Any) {
    if (x is String) {
        // x는 자동으로 String 타입으로 캐스트된다
    }
}
```

- 타입 체크 후 자동으로 타입 캐스트

9. 데이터 클래스 (Data Classes)

``` kotlin
data class User(val name: String, val age: Int)
```

- 데이터 클래스는 자동으로 equals(), hashCode(), toString(), copy() 메서드를 생성