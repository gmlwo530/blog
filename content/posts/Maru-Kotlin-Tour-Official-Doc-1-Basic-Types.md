---
title: "[Maru의 Kotlin Tour - 공식 문서 편] #1 Basic Types"
date: 2020-02-02T23:23:40+09:00
weight: 1
aliases: []
tags: ["kotlin"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: false
---

해당 문서 첫 문장에 참 친근한 문장이 있다.

> In Kotlin, everything is an object

그래서 모든 변수에서 member function과 properties를 부를 수 있다고 한다.

이 섹션에서는 numbers, characters, booleans, arrays, 그리고 strings을 다룬다.

## Numbers

Kotlin에서 integer는 총 4가지, floating은 총 2가지 타입을 제공해준다.
<br>
<br>

### Integer

- `Byte`
- `Short`
- `Int`
- `Long`

```Kotlin
val one = 1 // Int
val threeBillion = 300000000 // Long
val oneLong = 1L // Long
val oneByte: Byte = 1
```

위의 코드에서 주목해야 할 점은 `threeBillioin` 변수이다.

integer로 초기화 된 변수는 `Int`의 maximum value를 초과 할 수 없다. 만약 초과 했다면, 그 변수의 type은 `Long`이다. 타입을 명시하고 싶으면 값 뒤에 `L`을 붙여주면 된다.
<br>
<br>

### Floating

- `Double`
- `Float`

```Kotlin
val pi = 3.14 // Double
val e = 2.7182818284 // Double
val eFloat = 2.7182818284f // Float, actual value is 2.7182817
```

별 다른 표기가 없으면 compiler는 `Double` type으로 추론한다.
`Float` type으로 명시하고 싶으면 `f` 또는 `F`를 접미어로 붙입니다. `Float` type의 값의 소수점이 6-7자리보다 많으면 반올림을 합니다. (`Double`은 15-16, `Float`은 6-7의 소수점 까지 표현 가능)
<br>
<br>

### Explict conversions

여기서 주의 해야 할 것이 다른 언어와 달리, Kotlin은 `implicit widening conversions`을 Number에서 제공하지 않습니다.

다음 예제 코드를 확인해보세요

```kotlin
fun main() {
  fun printDouble(d: Double) { print(d) }

  val i = 1
  val d = 1.1
  val f = 1.1f

  printDouble(d)
  // printDouble(i) // Error: Type mismatch
  // printDouble(f) // Error: Type mismatch
}
```

이런 상황도 발생한다.

```kotlin
// 가상의 코드다, 실제로 컴파일 되지 않는다.
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(b == a) // Surprise! This prints "false" as Long's equals() checks whether the other is Long as well


/*
---------------------------------------------
*/


val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
val i: Int = b.toInt() // OK: explicitly widened
print(i)
```

그래서 아래와 같은 함수를 사용하면 된다.

- `toByte()`: Byte
- `toShort()`: Short
- `toInt()`: Int
- `toLong()`: Long
- `toFloat()`: Float
- `toDouble()`: Double
- `toChar()`: Char
  <br>
  <br>

### Representation

java platform에서 numbers는 JVM 원시 타입으로 저장되는데, nullable number reference (e.g. Int?)로 인해 아래와 같은 상황이 생긴다.

```kotlin
// '==='은 두 변수의 object type이 같은지도 체크한다.

val a: Int = 10000
println(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
println(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
```

```kotlin
val a: Int = 10000
println(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
println(boxedA == anotherBoxedA) // Prints 'true'
```

<br><br>

### Floating point numbers comparison

- Equality checks: `a == b` and `a != b`
- Comparison operators: `a < b`, `a > b`, `a <= b`, `a >= b`
- Range instantiation and range checks: `a..b`, `x in a..b`, `x !in a..b`

<br><br><br>

## Character

`Char` type으로 표현되고, 숫자로 다뤄지지 않는다.

```kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```

Charcter는 single quote를 사용한다. (`'1'`)

그리고 `Int` number로 explict conversion이 가능하다.

```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```

<br><br><br>

## Boolean

`true`와 `false` 값을 가질 수 있으며, null도 값으로 가질 수 있습니다.

```kotlin
val b: Boolean? = null
b // null
```

<br><br><br>

## Arrays

Kolin에서 Array는 Array Class로 표현한다.

```kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

array를 만들기 위해서는 문서에서 3가지 방법을 제시 해줍니다.

- `arrayOf(1, 2, 3)`
- `arrayOfNulls()`
  ```kotlin
  val a : Array<String?> = arrayOfNulls(3)
  ```
- `Array` constructor
  ```kotlin
    // Creates an Array<String> with values ["0", "1", "4", "9", "16"]
  val asc = Array(5) { i -> (i * i).toString() }
  asc.forEach { println(it) }
  ```

또한, Kotlin에서 Array는 불변이라 `Array<String>`가 `Array<Any>`에 할당되지 않는다.

### Primitive type arrays

kotlin에는 원시 타입의 배열을 표현 할 수 있는 특별한 클래스가 있다.

```Kotlin
// Array of int of size 5 with values [0, 0, 0, 0, 0]
val arr = IntArray(5)

// e.g. initialise the values in the array with a constant
// Array of int of size 5 with values [42, 42, 42, 42, 42]
val arr = IntArray(5) { 42 }

// e.g. initialise the values in the array using a lambda
// Array of int of size 5 with values [0, 1, 2, 3, 4] (values initialised to their index value)
var arr = IntArray(5) { it * 1 }
```

<br><br><br>

## Unsigned integers

부호없는 유형은 Kotlin 1.3 이후에만 사용할 수 있어서, 일단 패스하겠다.

<br><br><br>

## String

`String` 타입으로 표현 되고, 불변이다. String의 요소는 Charcter이고 s[i] 또는 `for-loop`로 접근 할 수 있다.

```kotlin
val c1 : Char = str[i]

for (c in str) {
    println(c)
}
```

`+` operator로 String들을 연결 할 수 있고, 앞에 오는 변수가 string이면 다른 타입의 변수와도 연결 할 수 있다.

```kotlin
val s = "abc" + 1
println(s + "def") // abc1def

val s1= 1 + "abc" // Error!

```

여러 줄을 표현하는 방법은 다음과 같다

```Kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

`trimMargin()` 함수를 이용해서 whitespace를 제거 할 수 있다.

```Kotlin
// By default | is used as margin prefix


/*
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
*/
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """

/*
Tell me and I forget.
Teach me and I remember.
Involve me and I learn.
(Benjamin Franklin)
*/
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

### String templates

`$`를 사용해서 string 안에 변수를 넣을 수 있다.

```kotlin
val i = 10
println("i = $i") // prints "i = 10"

val s = "abc"
println("$s.length is ${s.length}") // prints "abc.length is 3"

val price = """
${'$'}9.99
"""
```

<br><br>

---

**Reference**

- [https://kotlinlang.org/docs/reference/basic-types.html](https://kotlinlang.org/docs/reference/basic-types.html)
- [https://kotlinlang.org/docs/reference/equality.html](https://kotlinlang.org/docs/reference/equality.html)
