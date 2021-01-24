---
title: "[Maru의 Kotlin Tour - 공식 문서 편] #2 Control Flow "
date: 2020-02-02T23:23:53+09:00
weight: 1
aliases: []
tags: ["kotlin"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: true
---

## If Expression

`if`는 expression, 즉 값을 return 하므로 `tenary operator`(condition ? then : else)가 없다고 한다.(난 tenary operator 좋은데..)

그럼 if 표현식을 보자

```kotlin
// Traditional usage
var max = a
if (a < b) max = b

// With else
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}

// As expression
val max = if (a > b) a else b
```

마지막 코드인 `val max = if (a > b) a else b`가 tenary operator와 비슷해보인다.

if문을 이런 식으로 써서 변수에 값을 넣어 줄 수도 있다.

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

<br><br>

## When Expression

when `switch`와 같은 표현입니다

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}

when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}

when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}

when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}

fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}

when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}


//Since, Kotlin 1.3
fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
```

<br><br><br>

## For Loops

`for loop`는 다음과 같이 표현 할 수 있다.

```kotlin
for (item in collection) print(item)

for (item: Int in ints) {
    // ...
}

for (i in 1..3) {
    println(i)
}

(0..10).forEach { }

for (i in 6 downTo 0 step 2) {
    println(i)
}

// Iterate through an array or a list with an index
for (i in array.indices) {
    println(array[i])
}

for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

`for`는 무엇이든 반복자들이 주어지면 iterate를 한다.

<br><br><br>

## While Loops

흔히 쓰는 while 문의 형태를 갖고 있다.

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```

<br>

---

**Reference**

- [https://kotlinlang.org/docs/reference/control-flow.html](https://kotlinlang.org/docs/reference/control-flow.html)
- [https://devhints.io/kotlin](https://devhints.io/kotlin)
