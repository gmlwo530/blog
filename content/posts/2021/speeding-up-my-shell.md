---
title: "쉘 로딩 시간 개선하기"
date: 2021-03-06T23:00:18+09:00
weight: 1
aliases: []
tags: ["shell", "oh-my-zsh", "zsh"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: true
---

어느 순간부터 터미널의 시작 시간이 오래 걸리는 느낌이 들었다. 터미널을 자주 끄고 키고 하는 습관이 있어서, 시작할 때마다 3~4초동안 검은 화면에 비춰지는 내 얼굴을 봐야 했다. 1분 1초가 아까운데 3~4초의 시간을 내 얼굴을 보는데 사용하는 것은 너무 아깝다. 원인을 찾고 해결해서 속도를 개선하고 싶은 니즈로 구글링을 해봤다. 다행히, 나와 같은 니즈를 가지고 있는 사람의 [잘 정리한 블로그 글](https://blog.mattclemente.com/2020/06/26/oh-my-zsh-slow-to-load.html)을 발견했다. 방법만 알고 싶으면 해당 글을 봐도 좋지만, 나는 shell에 대한 지식이 하나도 없어서 글에 나오는 shell의 개념들을 이번 글에서 정리하며 노트북에서 3~4초 동안 검은 거울이 생기는 문제를 좀 더 짧게 나오도록 해보고자 한다.

글을 시작하기 전에 나의 shell 시작 시간은 어떤지 알아보자.

```zsh
❯ for i in {1..5}; do /usr/bin/time $SHELL -i -c exit; done
```

나는 3.4초에서 3.6초가 나왔다. 느리다! 느리다!! 느리다!!!

## ⚙️ 환경

내가 사용하고 있는 환경은 아래와 같다.

- 맥북 16인치
- zsh & oh-my-zsh
- 기본 터미널
- zsh plugin
  - git
  - zsh-autosuggestions
  - zsh-syntax-highlighting
- virtual environments
  - pyenv

## ⏱ 현재 shell 로딩 시간 측정

내가 지금 느린건 알겠는데, 얼마나 느린지 알아야지 개선 했을 때 빨라졌구나를 알 수 있지 않은가?

측정하는 법은 꽤 간단하다.

```zsh
❯ for i in {1..5}; do /usr/bin/time $SHELL -i -c exit; done
```

- `for in` / `do` / `done`: 꽤 직관적이다. 반복문인데 `do` 키워드로 해야할 일을 명시 해준다는게 인상적이다.
- `;`: for 문을 한 줄로 작성 할 수 있게 해주는 구분자다.
- `{1..5}`: sequence를 만들어준다. 참조한 블로그에서는 `$(seq 1 10)`를 사용하는데, 구식 문법이라고 한다. 게다가 10번까지 체크할 필요도 없이 느려서 5회 측정하도록 변경하였다.
- `/usr/bin/time`: 뒤에 오는 명령어(예제에서는 `$SHELL -i -c`)가 실행되고 끝날 때까지의 시간을 측정해서 통계를 내주는 **명령어**다.
- `$SHELL`: 현재 실행되고 있는 shell을 실행시키는 명령어의 경로를 나타내는 환경 변수다. 내 shell은 zsh를 사용하고 있어서 `echo $SHELL`를 입력하면 `/bin/zsh`가 리턴 된다.
  - `-i`: shell을 강제로 interactive하게 만들어주는 인자다.([좀 더 알아보기](https://unix.stackexchange.com/questions/551654/what-does-zshs-i-option-actually-do))
  - `-c`: 이 인자 뒤로 나오는 첫 번째 인자를 명령어로써 인식하고 실행시킨다.
- `exit`: shell을 종료한다.

{{<line_break>}}

이 명령어를 실행하면 아래와 같이 출력 된다. 개인 터미널의 설정에 따라 추가적인 텍스트가 출력 될 수도 있다. 유심히 봐야 할 부분은 `real`, `user`, `sys` 키워드다.

```zsh
  3.62 real         1.85 user         1.70 sys
  3.52 real         1.88 user         1.65 sys
  3.55 real         1.88 user         1.67 sys
  3.57 real         1.88 user         1.70 sys
  4.07 real         2.13 user         1.92 sys
```

- `real`: 호출 후 시작부터 끝날 때까지 측정 된 시간이다. 이 글에서 우리가 개선하고자 하는 "shell이 시작하는 시간"이라고 생각하시면 된다.
- `user`: 프로세스가 실행 될 때 kernel 외부에서 CPU가 소비한 시간이다.
- `sys`: 프로세스가 실행 될 때 kernel 내부에서 CPU가 소비한 시간이다.
  (`real`, `user`, `sys` 키워드에 대해 좀 더 자세히 알고 싶으면 필자가 [참고한 글](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)을 읽어보시는 것을 추천한다..)

시간이 너무 길다. 개선을 해보자.

## ❗️ 개선을 시작하기 전에

이제 개선을 시작하게 되면 설정 파일들을 변경하고 반영 할 것이다. 아마도 제일 많이 `~/.zshrc` 부분을 수정 할 것이다. 해당 파일의 변경 사항을 변하기 위해서 shell의 session을 다시 시작해야 하는데 `exec zsh` 명령어를 사용하면 된다.

```zsh
❯ exec zsh
```

- 이 부분에서 굉장히 놀랐는데, 평소에 `~/.zshrc` 수정 사항을 반영하기 위해 `source ~/.zshrc` 명령어를 사용했기 때문이다. [oh-my-zsh 공식 문서](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#how-do-i-reload-the-zshrc-file)를 보면 `source ~/.zshrc`를 사용하는 것은 잘못된 방법이라고 말한다.

## ⏱ Plugin 시간 측정

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

중요한 것은 둘 다 밀리 초로 환산해서 계산하고 있다.

{{< line_break >}}

내 맥북에서는 아래와 같이 나왔다

```zsh
❯ exec zsh
15: git
21: zsh-syntax-highlighting
7: zsh-autosuggestions
```

21 밀리초면 0.021초라는 말인데, 플러그인은 속도를 느리게 하는 범인이 아닌듯 하다..

- ❗️ 시간 측정이 끝나면 시간 측정 코드를 꼭 지우도록 하자!

## 🧭 더 확실한 원인을 찾아서

plugin이 치명적인 원인이 아니므로 zsh이 로드 될 때 일어나는 일들의 시간을 측정해야 한다.

zsh은 기가막힌 프로파일링 모듈을 가지고 있는데, 이름하여 [`zsh/zprof`](http://zsh.sourceforge.net/Doc/Release/Zsh-Modules.html#The-zsh_002fzprof-Module)다.

굉장히 편리하게 사용 할 수 있다. 일단 `~/.zshrc` 파일을 열어서 제일 첫 문장에 `zmodload zsh/zprof`를 추가 해준다.

```zsh
zmodload zsh/zprof
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH
# ....
```

잘 저장하고 `exec zsh` 명령어를 사용해서 shell을 다시 실행해준 후 아래와 같은 명령어를 실행하면 프로파일링한 결과를 얻을 수 있다.

```zsh
❯ zprof
num  calls                time                       self            name
-----------------------------------------------------------------------------------
 1)    3         390.53   130.18   39.51%    390.53   130.18   39.51%  _pyenv_virtualenv_hook
 2)    1         159.78   159.78   16.17%    159.78   159.78   16.17%  compdump
 3)    1         350.09   350.09   35.42%    107.25   107.25   10.85%  compinit
 4)  781          83.97     0.11    8.50%     83.97     0.11    8.50%  compdef
 ....
```

- _여기서 `compdump`, `compinit`, `compdef`도 꽤 많은 지분을 가지고 있는데, 잠깐 찾아보니 해당 함수들을 어떻게든 시간 최적화를 한다고 해도 부작용이 일어나지 않을거라는 보장이 없다고 한다. 분하지만 얘네들은 가만히 두자.._

참고한 글의 저자는 프로파일링 정보 중 함수 이름 왼쪽에 있는 퍼센트(%) 값이 속도 개선을 하는데 도움이 된다고 한다. 저 퍼센트(%) 값은 쉘 로드 시간 중에 해당 함수가 로드 된 시간의 지분을 뜻한다. (일리 있는 말이다...!)

범인을 찾았다! 파이썬 버전과 가상 환경 관리를 편하게 해주는 [pyenv](https://github.com/pyenv/pyenv)가 날 불편하게 만들고 있었다...🕵🏻‍♂️

## 🔧 가상 환경 관리 툴과 속도 개선 두 마리 토끼 잡기!

제일 빠르고 편한 방법은 그냥 pyenv를 지우면 된다. 하지만 고작 1~2초의 이득을 보겠다고 세상 편리한 가상 환경 관리 툴을 지우는건 멍청한 짓이다.

이제 참고한 글의 저자는 2가지 방법을 제시한다.

1. Lazy Loading
2. [Caching Eval](https://blog.mattclemente.com/2020/06/26/oh-my-zsh-slow-to-load.html#caching-eval)

개인적인 생각으로 2번째 방법은 caching으로 인해 리소스가 소모되고, 1번 방법보다 시간 최적화가 덜 되어 1번 방법을 선택하기로 했다.(사실 이건 실행하는 환경 관리 툴과 디바이스 별로 다를지도 모른다.) 2번 방법은 참고 글의 링크를 걸었으니, 참고하면 좋을듯 하다.

## 💤 Lazy Loading

이 방법이 해결책이 되는 이유는 간단하다. 쉘이 실행 될 때마다 `pyenv`가 로드 될 필요가 없다는게 포인트다. 안 쓰는데 로드 할 필요가 없지 않은가? 그냥 내가 원할 때 로드해서 사용하면 된다.

참고 글에서는 저자가 `nvm`을 사용하기 때문에, [zsh-nvm](https://github.com/lukechilds/zsh-nvm) 플러그인을 사용해서 해결하는 방법을 보여준다.

슬프게도 `pyenv`에게는 `zsh-nvm` 같이 star가 많은 플러그인이 없다. 그래도 갓갓 개발자분들이 이미 만들어 놓았다. [zsh-pyenv-lazy](https://github.com/davidparsson/zsh-pyenv-lazy)라는 플러그인인데 파일 설치도 간단하고 코드도 직관적이라 star와 last commit 날짜를 신경쓰지 않고 설치했다.

- 저자가 `rbenv`는 lazy load하는 편한 방법을 못 찾았다고 한다. 그래서 2번 방법을 사용하라고 하는데, `zsh-pyenv-lazy`처럼 구현 할 수 있지 않을까 하는 조심스러운 추측을 해본다.(내가 만들어볼까?)

플러그인 설치 후 기존에 입력 해놓은 `pyenv` 실행 코드를 `~/.zshrc`에서 삭제했다.

```zsh
# 혹시 모르니 주석 처리~
# export PYENV_ROOT="$HOME/.pyenv"
# export PATH="$PYENV_ROOT/bin:$PATH"
# if command -v pyenv 1>/dev/null 2>&1; then
#   eval "$(pyenv init -)"
# fi
# eval "$(pyenv virtualenv-init -)"
```

그리고 다시 시간 측정을 하니..!

```zsh
❯ for i in {1..5}; do /usr/bin/time $SHELL -i -c exit; done
        3.24 real         1.77 user         1.44 sys
        2.87 real         1.52 user         1.34 sys
        2.86 real         1.51 user         1.33 sys
        2.87 real         1.51 user         1.34 sys
        2.83 real         1.50 user         1.32 sys
```

대략 0.5~0.8초의 시간을 줄였다.(고작..? 현타가 좀 오는데..?)

뭔가 아직 내가 발견하지 못한 원인들이 있을 것이다. 추후에 다시 한번 연구해봐야겠다.

그래도 0.8초의 이득과 zsh, shell 문법에 대한 지식을 얻었기 때문에 나름 **성공적**이라고 생각하며 글을 마친다.

---

**Reference**

- [https://blog.mattclemente.com/2020/06/26/oh-my-zsh-slow-to-load.html#handling-virtual-environments](https://blog.mattclemente.com/2020/06/26/oh-my-zsh-slow-to-load.html#handling-virtual-environments)

- [https://man7.org/linux/man-pages/man1/time.1.html](https://man7.org/linux/man-pages/man1/time.1.html)
- [https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)
- [https://linux.die.net/man/1/zsh](https://linux.die.net/man/1/zsh)
- [https://unix.stackexchange.com/questions/551654/what-does-zshs-i-option-actually-do](https://unix.stackexchange.com/questions/551654/what-does-zshs-i-option-actually-do)
- [https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#how-do-i-reload-the-zshrc-file](https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#how-do-i-reload-the-zshrc-file)
- [https://man7.org/linux/man-pages/man1/date.1.html](https://man7.org/linux/man-pages/man1/date.1.html)
