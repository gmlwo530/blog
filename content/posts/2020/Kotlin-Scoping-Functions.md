---
title: "[Kotlin] 범위 지정 함수"
date: 2019-04-12T14:02:23+09:00
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

Kotlin에서 사용되는 함수 중에 *범위 지정 함수*라는게 있습니다. 이 함수들은 비슷한 모습을 하고 있어서, 약간의 혼란을 불러 일으키는 것 같습니다. 그러므로 이번 글에서는 이 범위 지정 함수들을 정리해보겠습니다.

apply, with, let, also, run은 전달받는 인자와 작동 방식, 결과가 매우 비슷하다. 이 5개는 **범위 지정 함수**라고 명칭한다.

이 함수들은 공통적으로 두가지 구성 요소를 가진다.

- 수신 객체
- [수신 객체 지정 람다](https://kotlinlang.org/docs/reference/lambdas.html#function-literals-with-receiver)

  함수 타입의 receiver이다. 예시 :

  `val sum: Int.(Int) -> Int = { other -> plus(other) }`

<br>

### with의 정의

```Kotlin
  inline fun <T, R> with(receiver: T, block: T.() -> R): R {
      return receiver.block()
  }
```

receiver는 수신 객체이고, block은 수신 객체 지정 람다이다.

수신 객체가 매개 변수 T로 제공된다. 이것을 명시적으로 제공된 수신 객체라고 한다.

수신 객체 지정 람다가 T의 확장함수 형태로 코드 블록 내에 수신 객체가 암시적으로 전달 된다.

최종적인 반환 값 : 람다를 실행한 결과를 반환한다.

- _확장함수란?_

  확장 하려는 대상에 함수를 추가하는 것이다. 예시,

        fun main(args: Array<String>{
        	println("Hello".whatIsLonggerString("Hi"))
        }

        fun String.whatIsLonggerString(x: String) : String {
        	return if(this.length > x.length) this else x
        }

        //결과값 : Hello

        //즉 여기서 "Hello"는 확장 하려는 대상이고, whatIsLonggerString(String)은 확장 함수명이다.

<br>

### also의 정의

```Kotlin
  inline fun <T> T.also(block: (T) -> Unit): T {
      block(this)
      return this
  }
```

T(수신 객체)의 확장함수로 수신 객체가 암시적으로 제공 된다.

수신 객체 지정 람다에 매개 변수 T로 코드 블록 내에 명시적으로 전달 된다.

최종적인 반환 값 : 코드 블록 내에 전달된 수식객체를 그대로 다시 반환 한다.

<br>

### apply의 정의

```Kotlin
  inline fun <T> T.apply(block: T.() -> Unit): T {
      block()
      return this
  }
```

T(수신 객체)의 확장함수로 수신 객체가 암시적으로 제공 된다.

수신 객체 지정 람다가 T의 확장함수 형태로 코드 블록 내에 수신 객체가 암시적으로 전달 된다.

최종적인 반환 값 : 코드 블록 내에 전달된 수신객체를 그대로 다시 반환 한다.
<br>

### let의 정의

```Kotlin
  inline fun <T, R> T.let(block: (T) -> R): R {
      return block(this)
  }
```

T(수신 객체)의 확장함수로 수신 객체가 암시적으로 제공 된다.

수신 객체 지정 람다에 매개 변수 T로 코드 블록 내에 명시적으로 전달 된다.

최종적인 반환 값 : 람다를 실행한 결과를 반환한다.

<br>

### run의 정의

```Kotlin
  inline fun <T, R> T.run(block: T.() -> R): R {
      return block()
  }
```

T(수신 객체)의 확장함수로 수신 객체가 암시적으로 제공 된다.

수신 객체 지정 람다가 T의 확장함수 형태로 코드 블록 내에 수신 객체가 암시적으로 전달 된다.

최종적인 반환 값 : 람다를 실행한 결과를 반환한다.

<br>

### 정리

5개의 범위 지정 함수는 아래의 3가지 차이점 중 1가지가 서로 다르다고 할 수 있다.

- 범위 지정 함수의 호출 시에는 수신 객체가 매개 변수로 명시적으로 전달 되거나 수신 객체의 확장 함수로 암시적 수신 객체로 전달 된다.
- 범위 지정 함수 의 수신 객체 지정 람다 에 전달되는 수신 객체가 명시적 매개 변수 로 전달 되거나 수신 객체의 확장 함수로 암시적 수신 객체로 코드 블록 내부로 전달 된다.
- 범위 지정 함수의 결과로 수신 객체를 그대로 반환하거나 수신 객체 지정 람다 의 실행 결과를 반환한다.

<br>

다음 글에서는 위에서 설명한 5개 함수의 예시를 설명하는 글을 정리하겠습니다.

---

**출처**

- [https://medium.com/@limgyumin/코틀린-의-apply-with-let-also-run-은-언제-사용하는가-4a517292df29](https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%98-apply-with-let-also-run-%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-4a517292df29)
- [https://brunch.co.kr/@mystoryg/19](https://brunch.co.kr/@mystoryg/19)
