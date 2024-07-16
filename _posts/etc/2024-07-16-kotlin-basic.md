---
title: "Kotlin �⺻ ����"
excerpt: ""

categories:
  - ETC
tags:
  - [Kotlin]

permalink: /etc/kotlin-basic

toc: true
toc_sticky: true

date: 2024-07-16
last_modified_at: 2024-07-16
---

1. ���� ����
``` kotlin
val a: Int = 1 // read-only
var b: Int = 2

val c = 3 // read-only
var d = 4
```
- �ڹ��� ��� Java 10���� `var` Ű���带 ����Ͽ� ���� ������ ���ؼ��� Ÿ���� ������ �� �ִ�.
- ��Ʋ���� ��� ���������� Ÿ�� ������ �����ϴ�. Ÿ���� �����ϸ� �����Ϸ��� �ʱ�ȭ ���� ���� Ÿ���� �߷��Ѵ�.
- immutable ������ val, mutable ������ var

2. �Լ� ����
``` kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

3. �Լ��� ���� Ÿ�� ����
- ��Ʋ������ �Լ��� ���� Ÿ���� ������ �� �ִ� ���� �Ʒ� �� ������ �ִ�

3-1. �Լ��� `Unit` Ÿ���� ��ȯ
``` kotlin
fun printSum(a: Int, b: Int) {
    println("Sum of $a and $b is ${a + b}")
}
```
- �� �Լ��� ���� Ÿ���� ������� �ʾ�����, �����Ϸ��� �ش� �Լ��� �ƹ� ���� ��ȯ���� �ʰ� Unit Ÿ���� ��ȯ�ϴ� ������ �߷��Ѵ�.
- Unit�� �ڹ��� void�� ����������, ������ ��ȯ�ϴ� ���� �ִ� ǥ���̴�.

```
�ڹ��� void�� ���� Ÿ���� �ƴϸ�, ��ȯ ���� ���ٴ� ����� ��Ÿ���� ���� Ű�����̴�.
��Ʋ���� Unit�� ��ȯ ���� ������ ��Ÿ���� Ÿ���̴�. �Լ��� ���� Unit ��ü�� ��ȯ�� �� �ִ�.
```

3-2. ���� Ÿ���� �����Ϸ��� �߷��� �� ���� ��
``` kotlin
fun sum(a: Int, b: Int) = a + b
```

3. if ǥ����
``` kotlin
fun maxOf(a: Int, b: Int) = if (a > b) a else b
```
- if�� statement �Ӹ� �ƴ϶� expression(ǥ����)���� ��� ����

4. when ǥ����
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
- �ڹ��� switch ���� ����������, �پ��� ������ ó���� �� ������ ǥ�������� ��� ����

5. for ����
``` kotlin
val items = listOf("apple", "banana", "kiwi")
for (item in items) {
    println(item)
}
```
- �⺻ for ������ for-each ������ ���
``` kotlin
val items = listOf("apple", "banana", "kiwi")
for (index in items.indices) {
    println("Item at $index is ${items[index]}")
}

for ((index, value) in items.withIndex()) {
    println("Item at $index is $value")
}
```
- �ε����� ����� ��쿡�� indices �Ӽ� Ȥ�� withIndex()�� ���

6. Null Safety
``` kotlin
class Person(val name: String, var age: Int) {
    
    constructor(name: String) : this(name, 0) {
        // �߰� �ʱ�ȭ �ڵ�
    }
}
```
- ��Ʋ������ �⺻������ ��� ������ null ���� ���� �� ����. 
- null ���� ����Ϸ��� Ÿ�� �ڿ� `?`�� �ٿ��� �Ѵ�.

6. �� ������(primary constructor)/���� ������(secondary constructor)
``` kotlin
class Person(val name: String, var age: Int) {
    
    constructor(name: String) : this(name, 0) {
        // �߰� �ʱ�ȭ �ڵ�
    }
}
```
- �� �����ڴ� Ŭ���� ����� ���ǵǸ�, �ʵ带 �ʱ�ȭ�Ѵ�.
- ���� �����ڴ� constructor Ű���带 ����Ͽ� ���ǵǸ�, �� �����ڸ� ȣ���Ͽ� �ʱ�ȭ�Ѵ�.

7. �ν��Ͻ� ����
``` kotlin
val person = Person("John", 30)
```
- �ν��Ͻ��� ������ �� `new` Ű���带 ������� �ʴ´�.

6. ������ ��� ����

8. Ȯ�� �Լ��� Ȯ�� ������Ƽ
9. �̱��� ��ü ����
7. ���� �Լ�