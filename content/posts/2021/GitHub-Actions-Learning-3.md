---
title: "[GitHub Action Learning] #3 Workflows에서 변수 사용하기"
date: 2021-01-27T00:10:06+09:00
weight: 1
aliases: []
tags: ["CI/CD", "GitHub", "GitHub Actions"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: true
---

개발을 하다보면 세팅이 불가피한 기능 또는 프로그램을 사용할 때가 있다. 예를 들어 [PostgreSQL](https://www.postgresql.org/) 같은 데이터베이스에 연결 할 때 HOST, PORT, USERNAME, PASSWORD 같은 환경 변수를 세팅해야 할 때가 있다. 또는 job을 이용해서 어플리케이션을 빌드한 다음에 내가 작성한 script를 돌리고 싶을 때도 있다.

이렇게 커스텀하게 GitHub Actions를 다뤄보는 방법에 대해 알아보도록 하자.

{{< line_break >}}

### 환경 변수

일단 각 workflows가 실행 될때 마다 GitHub Actions가 가지고 있는 [기본 환경 변수](https://docs.github.com/en/actions/reference/environment-variables#default-environment-variables)가 제공 된다.

물론 개발자가 원하는 환경 변수를 설정 할 수도 있다. 환경 변수는 step, job, 그리고 workflow 전체의 범위로 사용 할 수 있다. 아래 예제는 step과 job에 환경 변수를 사용한 경우다.

```yaml
jobs:
  weekday_job:
    runs-on: ubuntu-latest
    env:
      DAY_OF_WEEK: Mon
    steps:
      - name: "Hello world when it's Monday"
        if: env.DAY_OF_WEEK == 'Mon'
        run: echo "Hello $FIRST_NAME $middle_name $Last_Name, today is Monday!"
        env:
          FIRST_NAME: Mona
          middle_name: The
          Last_Name: Octocat
```

여기서 `DAY_OF_WEEK` 변수와 `FIRST_NAME` 변수가 사용되는 부분을 보자. `DAY_OF_WEEK` 변수는 `env.DAY_OF_WEEK` 형식으로 작성되었다. 왜 그럴까?

`run` 키로 실행하기 되는 명렁어, 즉 `run` 키로 읽혀지는 환경 변수는 runner 운영체제에서 읽혀진다. job이 runner로 보내진 뒤 변수들이 대체된다.

`env.DAY_OF_WEEK`으로 표현 된 이유는 `if` 키에 있다. `if` 키로 읽히는 변수는 workflow가 runner로 보내지기 전 처리 될 때 대체되어야 하기 때문에 `env`를 사용한 것이다.

{{< line_break >}}

위의 예제는 명령어를 위한 환경 변수였다면, 아래처럼 actions를 위해 환경 변수를 설정할 수도 있다.

```yaml
- name: Create Sentry release
  uses: getsentry/action-release@v1
  env:
    SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
    SENTRY_ORG: ${{ secrets.SENTRY_ORG }}
    SENTRY_PROJECT: ${{ secrets.SENTRY_PROJECT }}
```

{{< line_break >}}

### Secret 환경 변수

위의 코드에서 `${{ secrets.SENTRY_ORG }}` 표현을 확인 할 수 있다. 어플리케이션을 만들다 보면 외부에 노출이 되면 안되는 민감한 정보(API_TOKEN, PASSWORD 등)들이 있다.
GitHub Actions는 이런 정보를 GitHub에 *secrets*로 저장해놓고 workflows에서 환경 변수로 사용할 수 있게 해준다.

- 참고: `${{ <expression> }}`은 변수를 세팅하거나 context에 접근하기 위한 표현식이다. 위의 예제에서는 context에 접근하고 환경 변수를 세팅하기 위해 사용 되었다.([문서](https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions#about-contexts-and-expressions))

secrets 환경 변수는 아래의 사진에 표시 된 레포지토리의 Settings 탭을 들어간 다음에

{{< figure src="/2021/GitHub-Actions-Learning-3/repo-actions-settings.png" caption=" " >}}

왼쪽 사이드 바의 Secrets 항목을 클릭 후 만들면 된다.

{{< line_break >}}

### `with` 키워드

Actions들을 보다보면 `env` 느낌처럼 `with` 키워드를 써서 변수들을 정의 해놓은 모습을 볼 수 있다.

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/hello_world@main
        with:
          first_name: Mona
          middle_name: The
          last_name: Octocat
```

`with`은 actions에 의해 정의된 파라미터들을 나열해주는 키워드이다. 위의 예제에서 `first_name`, `middle_name`, `last_name`은 `actions/hello_world@main`을 위한 파라미터라는 뜻이다. 파라미터들은 대문자로 변경되고 `INPUT_` 접두어가 붙여져서 `INPUT_FIRST_NAME`이라는 환경 변수로 변환 된다.

{{< line_break >}}

### 이번 글을 마치며

현재까지 내용으로 GitHub Actions를 사용 할 수 있을 것 같아, 개인적으로 개발하고 있는 [오픈 소스 프로젝트](https://github.com/gmlwo530/code-process-generator)에 GitHub Actions를 적용할 예정이다. 적용하면서 [Learn GitHub Actions](https://docs.github.com/en/actions/learn-github-actions)를 참고하게 되면 시리즈를 이어나가도록 하겠다.

---

**Reference**

- [Managing complex workflows](https://docs.github.com/en/actions/learn-github-actions/managing-complex-workflows)
- [Environment variables](https://docs.github.com/en/actions/reference/environment-variables)
- [Sentry Release GitHub Actions](https://github.com/marketplace/actions/sentry-release)
- [Creating encrypted secrets for a repository](https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository)
- [GihHub Actions `with` keyword](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepswith)
