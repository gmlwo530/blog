---
title: "[Maru의 Kotlin Tour - 공식 문서 편] #0 Intro"
date: 2020-02-02T17:30:14+09:00
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

들어가기 앞서 코틀린의 정의와 Command Line Compiler를 설치하는 법을 알아보겠습니다.

### 코틀린의 정의

코틀린(Kotlin)은 JVM에서 동작하는 프로그래밍 언어이다. 2011년 7월, 젯브레인사가 공개하였다.

캇린으로 읽어야 한다. (_캇린으로 읽어야 하는 것은 처음 알았습니다;;_)
<br><br>

### Install Command Line Compiler

Kotlin으로 작성한 파일을 실행하기 위해서는 몇 가지 방법을 [Tutorial Getting Start](https://kotlinlang.org/docs/tutorials/)에서 제안합니다. 저는 여기서 **Working with the Command Line Compiler** 방법을 선택 했습니다.

설치 방법도 여러가지 입니다. 저는 OS X를 사용하기 때문에 HomeBrew를 이용해서 설치하겠습니다.

- 다른 방법들은 [문서](https://kotlinlang.org/docs/tutorials/command-line.html)를 참고 해주세요

```bash
$ brew update
$ brew install kotlin
```

<br><br>

### 실행 해보기

잘 설치 되었는지 확인을 위해 가볍게 `hello.kt` 파일을 만들고 실행 해봅시다. ([코드](https://github.com/gmlwo530/kotlin-tour/blob/master/offical-doc/hello.kt))

```kotlin
// hello.kt

fun main(args: Array<String>) {
    println("Hello, World!")
}
```

이제 컴파일을 하면,

```bash
$ kotlinc hello.kt -include-runtime -d hello.jar
```

`hello.kt`가 있는 디렉토리에 `hello.jar`가 생기게 됩니다.

그리고 다음과 같이 어플리케이션을 실행시키면 kotlin-compiler가 제대로 설치 된 것을 확인 할 수 있습니다.

```bash
$ java -jar hello.jar
```

### Kotlin Compiler Option

컴파일을 하는 커맨드 라인을 보면 옵션이 있습니다. 이 옵션에 대해 알아보겠습니다.

```bash
$ kotlinc hello.kt -include-runtime -d hello.jar
```

- `-d` : 생성 할 클래스 파일의 위치를 설정해주는 옵션입니다. directory 또는 .jar 파일이 인자가 될 수 있습니다. (한번 씩 해보세요!)

  ```bash
  $ kotlinc hello.kt -include-runtime -d test/hello.jar

  $ kotlinc hello.kt -include-runtime -d hello.jar

  $ kotlinc hello.kt -include-runtime -d test/
  ```

- `-include-runtime` : 생성 할 jar 파일에 *kotlin runtime library*를 포함 시키는 옵션입니다.
  - kotlin runtime library를 포함 시키는 이유를 간단하게 얘기 하자면, kotlin runtime이 존재하지 않는 유저에게도 어플리케이션이 돌아가게 하기 위해서 입니다. 자세한 이유는 [StackOverflow의 Greg Kopff의 답변](https://stackoverflow.com/questions/30565036/why-kotlin-needs-to-bundle-its-runtime-after-compiled)을 참고해주세요.

<br><br>

**Reference**

- [Kotlin 위키피디아](<https://ko.wikipedia.org/wiki/%EC%BD%94%ED%8B%80%EB%A6%B0_(%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D_%EC%96%B8%EC%96%B4)>)
- [https://kotlinlang.org/docs/tutorials/command-line.html](https://kotlinlang.org/docs/tutorials/command-line.html)
- [https://stackoverflow.com/questions/30565036/why-kotlin-needs-to-bundle-its-runtime-after-compiled](https://stackoverflow.com/questions/30565036/why-kotlin-needs-to-bundle-its-runtime-after-compiled)
