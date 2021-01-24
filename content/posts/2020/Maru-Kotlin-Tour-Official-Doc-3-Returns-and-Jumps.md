---
title: "[Maru의 Kotlin Tour - 공식 문서 편] #3 Returns and Jumps"
date: 2020-02-05T00:28:44+09:00
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

kotlin에는 3가지 jump 표현식이 있다.

- `return` : 제일 가까운 enclosing function으로 부터 return 합니다.
- `break` : 제일 가까운 enclosing loop를 종료합니다.
- `continue` : 제일 가까운 enclosing loop의 다음 단계(step)를 진행합니다.

## Break and Continue Labels

Kotlin의 어떤 표현이든 `label`을 붙일 수 있다. `@`을 붙여서 라벨을 만들 수 있는데, 아래와 같이 쓰인다.

```kotlin
loop@ for (i in 1..100){
  // ...
}
```

이제, `break`에 `label`을 붙이면 아래와 같이 쓸 수 있는데

```kotlin
loop@ for (i in 1..100){
  for (j in 1..100){
    if (...) break@loop
  }
}
```

위의 코드 같은 경우는 `break@loop`에서 `loop@`로 점프 한 후 바로 다음 코드가 실행 된다고 한다. `continue` 같은 경우는 loop의 다음 반복자가 실행 된다고 한다.

뭔가 바로 느낌이 오지 않아 label이 없는 것과 있는 것을 비교하기 위해 예제를 만들었다.([코드](https://github.com/gmlwo530/kotlin-tour/blob/master/offical-doc/break_label.kt))

```kotlin
// break_label.kt

fun main() {
  println("Without Label Start")
  for (i in 1..3){
    for (j in 1..5){
      if (j==4) break
      print("$j ")
    }
  }
  println("\nWithout Label Done")
  println("--------------")

  println("With Label Start")
  loop@ for (i in 1..3){
    for (j in 1..5){
      if (j==4) break@loop
      print("$j ")
    }
  }
  println("\nWith Label Done")
}
```

```text
Without Label Start
1 2 3 1 2 3 1 2 3
Without Label Done
--------------
With Label Start
1 2 3
With Label Done
```

출력 된 숫자를 보면 바로 label이 break와 쓰였을 때 어떤 역할을 하는지 알 수 있다.

비슷한 코드로 continue도 알아보겠다.([코드](https://github.com/gmlwo530/kotlin-tour/blob/master/offical-doc/continue_label.kt))

```kotlin
// continue_label.kt

fun main() {
  println("Without Label Start")
  for (i in 1..3){
    for (j in 1..5){
      if (j==4) continue
      print("$j ")
    }
  }
  println("Without Label Done")
  println("--------------")

  println("With Label Start")
  loop@ for (i in 1..3){
    for (j in 1..5){
      if (j==4) continue@loop
      print("$j ")
    }
  }
  println("\nWith Label Done")
}
```

```text
Without Label Start
1 2 3 5 1 2 3 5 1 2 3 5
Without Label Done
--------------
With Label Start
1 2 3 1 2 3 1 2 3
With Label Done
```

잘 쓰면 좋은 표현인거 같다.

## Return at Labels

`return`은 함수를 리턴 시켜준다. 아래 코드를 보자

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // non-local return directly to the caller of foo()
        print(it)
    }
    println("this point is unreachable")
}
```

간단하다. `println("this point is unreachable")` 줄은 실행되지 않는 것을 확인 할 수 있다.

위의 코드와 같이 `return`은 가장 가까운 enclosing function인 `foo()`로 부터 리턴 시켜준다.

만약 lambda expression에서 리턴 시켜주고 싶으면 `label`을 사용하면 된다.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with explicit label")
}

// 위의 라벨을 두개 쓰지 않고 하나만 쓰고 싶으면 라벨 이름을 함수 이름으로 하면 된다.
/*
  fun foo() {
      listOf(1, 2, 3, 4, 5).forEach {
          if (it == 3) return@forEach
          print(it)
      }
      print(" done with explicit label")
  }
*/

// Result : 1245 done with explicit label
```

- 문서에 보면 이런 문장이 있다. `Note that such non-local returns are supported only for lambda expressions passed to inline functions.` 여기서 non-local returns은 이후 문서에서 자세히 다뤄보겠다. 간단하게만 보자면 non-local return은 lambda가 inline function에 들어가게 되면 사용 할 수 있다.

- 처음에 이 코드를 보고 contiune는 안되나 해서 해봤더니, `The label '@lit' does not denote a loop` 오류가 났다. 위의 코드는 lambda expression이지 for-loop가 아니라서 `break`와 `continue`를 쓸 수 없는 것이다. 번외로 아래 코드는 같은 역할을 한다.

  ```kotlin
  arrayOf(1, 2, 3).forEach label@ {
    print(it)
    return@label
  }

  label@ for (i in arrayOf(1, 2, 3)) {
    print(i)
    continue@label
  }
  ```

- `lit@`, `@lit`처럼 두 라벨을 만드는 것을 explict label이라고 한다.
- `@forEach`처럼 하나의 라벨을 만드는 것을 implict label이라고 한다.

위에서는 lambda expression일 때를 다뤘는데, 만약 anonymous functino이 오게 된다면 그냥 `return`만 쓰면 된다. 이때 return을 local return이라고 한다.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // local return to the caller of the anonymous fun, i.e. the forEach loop
        print(value)
    })
    print(" done with anonymous function")
}
```

<br>

위의 예제들은 `continue`의 역할을 했다. `break` 역할을 하는 직접적인 방법은 없지만, nesting lambda와 non-local return을 이용하면 같은 역할을 하는 코드를 만들 수 있다.

```kotlin
fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}
```

return 할 때 원하는 값을 넘겨 줄 수도 있다.

```kotlin
return@a 1
```

<br><br>
라벨이란 개념이 다른 언어에서는 못 봤던 개념이였다. 그래서 아직은 사용에 미숙하지만, 코딩을 할 때 잘 쓰면 좋은 기능일거 같다.

<br><br>
**Reference**

- [https://kotlinlang.org/docs/reference/returns.html](https://kotlinlang.org/docs/reference/returns.html)

- [https://stackoverflow.com/questions/45728687/the-label-does-not-denote-a-loop-in-foreach](https://stackoverflow.com/questions/45728687/the-label-does-not-denote-a-loop-in-foreach)
