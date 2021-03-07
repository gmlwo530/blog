---
title: "Speeding Up My Shell"
date: 2021-03-06T23:00:18+09:00
weight: 1
aliases: []
tags: ["shell", "oh-my-zsh"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: true
hidemeta: false
disableShare: false
comments: true
---

어느 순간부터 터미널의 시작 시간이 오래 걸리는 느낌이 들었다. 터미널을 끄고 키고 자주하는 습관이 있어서, 시작할 때마다 3~4초동안 검은 화면에 비춰지는 내 얼굴을 봐야 했다. 1분 1초가 아까운데 3~4초의 시간을 내 얼굴을 보는데 사용하는 것은 너무 아깝다. 원인을 알고 속도를 개선하고 싶어서 구글링을 해봤는데, 다행이도 나와 같은 니즈를 가지고 있는 사람이 [잘 정리한 블로그 글](https://blog.mattclemente.com/2020/06/26/oh-my-zsh-slow-to-load.html)을 발견했다. 방법만 알고 싶으면 해당 글을 봐도 좋지만, 나는 shell에 대한 지식이 하나도 없어서 방법에 나오는 shell의 개념들을 이번 글에서 정리하며 해결해보고자 한다.

글을 시작하기 전에 나의 shell의 시작 시간은 어떤지 알아보자.

```zsh
$ for i in $(seq 1 10); do /usr/bin/time $SHELL -i -c exit; done
```

나는 3.4초에서 3.6초가 나왔다. 개선 후에는 ,,,

## ⚙️ 환경

내가 사용하고 있는 환경은 아래와 같다.

- 맥북 16인치
- zsh & oh-my-zsh
- 기본 터미널
- zsh plugin
  - git
  - zsh-autosuggestions
  - zsh-syntax-highlighting

## ⏱ 현재 shell 로딩 시간 알아보기

내가 지금 느린건 알겠는데, 얼마나 느린지 알아야지 개선 했을 때 빨라졌구나를 알 수 있지 않은가?

측정하는 법은 꽤 간단하다.

```zsh
$ for i in {1..5}; do /usr/bin/time $SHELL -i -c exit; done
```

- `for in` / `do` / `done`: 꽤 직관적이다. 반복문인데 `do` 키워드로 해야할 일을 명시 해준다는게 인상적이다.
- `;`: for 문을 한 줄로 작성 할 수 있게 해주는 구분자다.
- `{1..5}`: sequence를 만들어준다. 참조한 블로그에서는 `$(seq 1 10)`를 사용하는데, 구식 문법이라고 한다. 게다가 10번까지 체크할 필요도 없이 느려서 5회 측정하도록 변경하였다.
- `/usr/bin/time`: 뒤에 오는 명령어(예제에서는 `$SHELL -i -c`)가 실행되고 끝날 때까지의 시간을 측정해서 통계를 내주는 *명령어*다.
- `$SHELL`: 현재 실행되고 있는 shell을 실행시키는 명령어의 경로를 나타내는 환경 변수입니다. 저는 zsh를 사용하고 있어서 `echo $SHELL`를 입력하면 `/bin/zsh`가 리턴 됩니다.
  - `-i`: shell을 강제로 interactive하게 만들어주는 인자입니다.([좀 더 알아보기](https://unix.stackexchange.com/questions/551654/what-does-zshs-i-option-actually-do))
  - `-c`: 이 인자 뒤로 나오는 첫 번째 인자를 명령어로써 인식하고 실행시킵니다.
- `exit`: shell을 종료합니다.

이 명령어를 실행하면 아래와 같이 출력 됩니다. 개인 터미널의 설정에 따라 추가적인 텍스트가 출력 될 수도 있습니다. 유심히 봐야 할 부분은 `real`, `user`, `sys` 키워드입니다.

```zsh
  3.62 real         1.85 user         1.70 sys
  3.52 real         1.88 user         1.65 sys
  3.55 real         1.88 user         1.67 sys
  3.57 real         1.88 user         1.70 sys
  4.07 real         2.13 user         1.92 sys
```

- `real`: 호출 후 시작부터 끝날 때까지 측정 된 시간입니다. 이 글에서 우리가 개선하고자 하는 "shell이 시작하는 시간"이라고 생각하시면 됩니다.
- `user`: 프로세스가 실행 될 때 kernel 외부에서 CPU가 소비한 시간입니다.
- `sys`: 프로세스가 실행 될 때 kernel 내부에서 CPU가 소비한 시간입니다.
  (위의 키워드에 대해 좀 더 자세히 알고 싶으시면 제가 [참고한 글](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)을 읽어보시는 것을 추천합니다.)

시간이 너무 깁니다. 저 시간이면 코드 한 줄은 족히 쳤을텐데..

## ❗️ 개선을 시작하기 전에

이제 개선을 시작하게 되면 설정 파일들을 변경하고 반영 할 것이다. 아마도 제일 많이 `~/.zshrc` 부분을 수정 할 것이다. 해당 파일의 변경 사항을 변하기 위해서 shell의 session을 다시 시작해야 하는데 `exec zsh` 명령어를 사용하면 된다.

```zsh
$ exec zsh
```

- 이 부분에서 굉장히 놀랐는데, 평소에 `~/.zshrc` 수정 사항을 반영하기 위해 `source ~/.zshrc` 명령어를 사용했기 때문이다. [oh-my-zsh 공식 문서](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#how-do-i-reload-the-zshrc-file)를 보면 `source ~/.zshrc`를 사용하는 것은 잘못된 방법이라고 말한다.

## + Plugin 시간 조사

먼저 `~/.zshrc`에 선언 된 플러그인이 걸리는 시간부터 측정해보자.

`~/.oh-my-zsh/oh-my-zsh.sh` 파일을 열어보면 `~/.zshrc`에 정의 된 플러그인을 로드하는 부분이 있다. 코드는 아래와 같다.

```zsh
# Load all of the plugins that were defined in ~/.zshrc
for plugin ($plugins); do
  if [ -f $ZSH_CUSTOM/plugins/$plugin/$plugin.plugin.zsh ]; then
    source $ZSH_CUSTOM/plugins/$plugin/$plugin.plugin.zsh
  elif [ -f $ZSH/plugins/$plugin/$plugin.plugin.zsh ]; then
    source $ZSH/plugins/$plugin/$plugin.plugin.zsh
  fi
done
```

이제 이 코드에 시간을 측정하는 코드를 넣어보자.

```zsh
# Load all of the plugins that were defined in ~/.zshrc
for plugin ($plugins); do
  timer=$(($(gdate +%s%N)/1000000))
  if [ -f $ZSH_CUSTOM/plugins/$plugin/$plugin.plugin.zsh ]; then
    source $ZSH_CUSTOM/plugins/$plugin/$plugin.plugin.zsh
  elif [ -f $ZSH/plugins/$plugin/$plugin.plugin.zsh ]; then
    source $ZSH/plugins/$plugin/$plugin.plugin.zsh
  fi
  now=$(($(gdate +%s%N)/1000000))
  elapsed=$(($now-$timer))
  echo $elapsed":" $plugin
done
```

추가 된 코드를 살펴 보면 `timer`라는 변수에 plugin 로드 시작 시간을 담아놓고, `now`라는 변수에 plugin 로드 끝 시간을 담아서 그 차이를 보고 걸리는 시간을 측정하는 것이다.

측정하는 과정에 대해 자세히 알아보자

- `$(())`: 계산한 값을 변수에 넣을 때 사용하는 문법이다.
- `gdate`: GNU의 `date` 명령어이다.
  - `+`: `%s`, `%N`등 format을 사용하려면 접두로 붙혀야 한다.
  - `%s`: 1970-01-01 00:00:00 UTC부터의 시간을 초(seconds)로 반환한다.
  - `%N`: 시간을 나노초로 표현한다.

gdate 명령어는 Homebrew로 [coreutils](https://formulae.brew.sh/formula/coreutils)를 설치하면 사용 할 수 있는 명령어다.

만약 설치하기 싫거나 Homebrew를 사용하지 않는다면 기본 내장 된 python을 사용하는 방법도 있다. (이때 python version에 따라 print 문법이 다르다는 것을 주의하자.)

```zsh
timer=$(($(python -c 'from time import time; print(int(round(time() * 1000)))')))

now=$(($(python -c 'from time import time; print(int(round(time() * 1000)))')))
```

- 참고로 python으로 실행해보니 `gdate`보다 좀 더 로드 시간이 길었다. 아마 python 코드 실행 시간이 좀 더 걸린게 아닐까 추측해본다.

{{< line_break >}}

둘 다 밀리 초로 환산해서 계산하고 있다.

내 컴퓨터에서는 아래와 같이 나왔다

```zsh
$ exec zsh
15: git
21: zsh-syntax-highlighting
7: zsh-autosuggestions
```

21 밀리초면 0.021초라는 말인데, 플러그인은 속도를 느리게 하는 범인이 아닌듯 하다..

---

**Reference**

- [https://man7.org/linux/man-pages/man1/time.1.html](https://man7.org/linux/man-pages/man1/time.1.html)
- [https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)
- [https://linux.die.net/man/1/zsh](https://linux.die.net/man/1/zsh)
- [https://unix.stackexchange.com/questions/551654/what-does-zshs-i-option-actually-do](https://unix.stackexchange.com/questions/551654/what-does-zshs-i-option-actually-do)
- [https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#how-do-i-reload-the-zshrc-file](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#how-do-i-reload-the-zshrc-file)
- [https://man7.org/linux/man-pages/man1/date.1.html](https://man7.org/linux/man-pages/man1/date.1.html)
