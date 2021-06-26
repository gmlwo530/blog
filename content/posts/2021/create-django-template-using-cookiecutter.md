---
title: "Cookiecutter - 지겨운 프로젝트 세팅은 이제 그만하자..!"
date: 2021-06-27T01:00:00+09:00
weight: 1
aliases: []
tags: ["python", "CLI"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: true
---

나는 새로운 프로젝트를 생성할 때마다 똑같은 상황에 부딪히곤 한다. "분명히 예전에 해봤던 세팅들인데?" 라는 생각과 동시에 기억이 흐릿해져, 결국 예전에 했던 프로젝트들을 뒤적이며 프로젝트 세팅에만 에너지를 다 쏟아 정작 비즈니스 로직은 하나도 못 짜고 내일을 기약하는 경험이 한 두번이 아니였다.

고통에 몸 부리치던 와중.. react에서 template이라는 기능을 통해 미리 세팅해놓은 프로젝트를 터미널에서 커맨드 한 줄로 바로 생성했던 것이 기억 났다. 요즘은 django나 fastapi 프로젝트를 주로 하고 있어서, 비슷한게 없나 찾아보다 역시나 개발 세상에는 없는게 없드라..


이번 글은 고통으로부터 나를 구제해줄 [cookiecutter](https://cookiecutter.readthedocs.io/en/1.7.2/README.html)에 대해 소개 해보겠다.

{{< line_break >}}

## cookiecutter란?

cookiecutter는 파이썬으로 만들어진 command line utility다.

(그러므로 장비에 파이썬이 설치되어 있어야한다. 이 툴을 처음 봤을 때는 파이썬으로 만들어진 유틸리티라서 파이썬 프로젝트만 만들 수 있는걸로 착각했는데, 언어와 프레임워크 상관 없이 파일을 생성해주는 개념이기 때문에 원하는 프로젝트를 만들 수 있다. 예를 들어 [go project도 있다.](https://registry.terraform.io/providers/hashicorp/aws/latest/docs))

이 유틸리티가 해주는 작업은 아주 간단하다. 간단한 예시를 통해 알아보자.

{{< line_break >}}

```plain text
├── example
   └── {{cookiecutter.project_name}}
```
일단 위와 같은 구조를 가진 example 디렉토리를 만든다. 이때, `{{cookiecutter.project_name}}`를 기억해두자.

```python
# ~/example/{{cookiecutter.project_name}}/main.py

print("Hello {{ cookiecutter.hello_word }}")
```

그리고 `example` 디렉토리에 문장을 출력해주는 간단한 파이썬 파일을 만들었다.

{{< line_break >}}

같은 디렉토리에 json 파일을 하나 만들자. 이름은 `cookiecutter.json`이여야 한다.

```json
// ~/example/cookicutter.json

{
    "project_name": "my_project",
    "hello_word": "world"
}
```

이렇게 되면 디렉토리 구조가 아래와 같다.

```plain text
example/
├── {{cookiecutter.project_name}}
│   └── main.py
└── cookiecutter.json
```

{{< line_break >}}

이제 example 밖의 디렉토리로 이동해보자.

```bash
$ ls
example
```

cookiecutter 명령어의 인자로 example 디렉토리를 주면 아래와 같이 선택지가 나온다.(`[]` 안에 있는 값은 기본값이다.)

```bash
$ cookiecutter ./example
project_name [my_project]:    
hello_word [world]:
```

다시 ls 명렁어를 실행하면 `my_project` 폴더가 생성된 것을 알 수 있다.

```bash
$ ls
example my_project
```

`my_project` 디렉토리의 구조는 아래와 같다.

```plain text
my_project/
└── main.py
```

그리고 `main.py`를 실행시켜보면 json에 작성한 단어가 출력되는 것을 확인 할 수 있다.

```bash
$ python my_project/main.py
Hello world
```

{{< line_break >}}

`{{ cookiecutter.some_key }}`의 형태로 some_key에 자기가 원하는 단어를 정의하고 `cookiecutter.json`에 해당 키에 맞는 값을 정의 해주면 cookiecutter가 json 파일을 읽고 값으로 대체 해주는게 cookiecutter가 하는 일이다. 간단하면서 강력한 기능이다.

- `{{}}` 안의 키를 대체 해주는 작업은 [Jinja2](https://jinja.palletsprojects.com/en/latest/)를 통해서 한다. 다양한 문법이 궁금하면 Jinja2 패키지를 살펴보는것도 좋을 것 같다.

{{< line_break >}}

## 참고하면 좋은 프로젝트

처음에는 웹 프레임워크를 가지고 예제를 하나 만들어 볼까 했는데, 예제 수준이 어렵지도 않고 해당 웹 프레임워크에 관심이 없는 사람이 있기 때문에 참고하면 좋은 프로젝트 2개를 공유 한다. 

단순히 사용하기 위한 프로젝트를 찾는거라면 [GitHub Search 기능](https://github.com/search?q=cookiecutter&type=Repositories)을 이용해서 찾으면 된다. 아래 프로젝트들은 본인만의 cookiecutter template을 만들 때 참고하면 좋은 프로젝트들이다.(테스트, 프로젝트 구조, hooks 등이 잘 되어 있다.)

- [cookiecutter-django](https://github.com/pydanny/cookiecutter-django): 파이썬 진영에서 제일 많이 쓰이는 웹 프레임워크인 Django cookiecutter

- [cookiecutter-pypackage](https://github.com/audreyfeldroy/cookiecutter-pypackage): 파이썬 패키지를 만들기 위한 cookiecutter

{{< line_break >}}

## 고민했던 포인트들

### 선택에 따른 설정 파일

만약 프로젝트에서 docker 사용을 선택에 맡긴다고 하자.

```json
{
    ...
    "use_docker": "no",
}
```

docker에서만 사용되는 `.dockerignore` 파일이 있다. 이 파일은 선택에 따라 존재하거나 사라져야 한다.

선택에 따른 처리를 해주기 위해서 cookiecutter에서는 [hooks](https://cookiecutter.readthedocs.io/en/latest/advanced/hooks.html)라는 기능을 제공해준다.

```plain text
cookiecutter-something/
├── {{cookiecutter.project_name}}/
├── hooks
│   ├── pre_gen_project.py
│   └── post_gen_project.py
└── cookiecutter.json
```

`cookiecutter.json`과 같은 위치에 `hooks` 디렉토리를 만들고 `pre_gen_project.py`과 `post_gen_project.py` 두 개의 파일을 만든다.(shell script로 만들 수도 있다.)

`pre_gen_project.py`는 프로젝트 생성 전에 실행되고, `post_gen_project.py`는 프로젝트 생성 후에 실행된다.

이제 위의 docker 문제를 해결하려면 `post_gen_project.py` 파일을 생성하면 된다.

```python
# cookiecutter-something/hooks/post_gen_project.py

import os
import shutil


# 선택에 따른 생성보다는 다 만들어 놓고 지우는 방향이 더 편하다.
def remove(filepath):
    if os.path.isfile(filepath):
        os.remove(filepath)
    elif os.path.isdir(filepath):
        shutil.rmtree(filepath)


if "{{ cookiecutter.use_docker }}" == "n":
    remove(os.path.join(os.getcwd(), "{{cookiecutter.project_name}}", ".dockerignore"))
```

### 테스트

cookiecutter template을 만들 때는 테스트 코드가 필요하다. 만약 직접 테스트를 한다면 굉장히 시간이 오래걸린다. 1. template 수정하고 2. cookiecutter로 프로젝트 만들어보고 3. 프로젝트 실행 또는 테스트 해보고 4. 프로젝트를 지우고를 반복해야 한다.

아쉽게도 cookiecutter 공식 문서에는 테스트에 대한 가이드가 자세하게 있지 않다.

그래서 이미 존재하는 cookiecutter template 오픈 소스 프로젝트에서 테스트 하는 방법을 참고하면 좋을 것 같다.

{{< line_break >}}

## 내가 하고 있는 프로젝트

현재 나는 2개의 cookiecutter template을 만들고 있다.

- [gmlwo530/cookiecutter-django](https://github.com/gmlwo530/cookiecutter-django)
- [gmlwo530/cookiecutter-fastapi](https://github.com/gmlwo530/cookiecutter-fastapi)

물론 둘 다 굉장히 스타가 많은 template이 존재한다.(심지어 FastAPI는 FastAPI 개발자가 만든 template이다.)
근데 이걸 만들어보니깐 굉장히 공부가 많이 된다. 그리고 템플릿을 보면 내가 원하는 툴이나 패키지, 라이브러리가 없기도 했다.

물론 급한 경우에는 유명한 템플릿을 바로 사용하는게 맞지만 직접 만들어보는 것도 굉장히 귀중한 자산이 될거라고 생각한다.

{{< line_break >}}

---
**Reference**

- [https://cookiecutter.readthedocs.io/en/latest/](https://cookiecutter.readthedocs.io/en/latest/)