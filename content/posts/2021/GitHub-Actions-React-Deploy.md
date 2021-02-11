---
title: "[GitHub Actions] React Deploy"
date: 2021-02-06T14:07:16+09:00
weight: 1
aliases: []
tags: ["CI/CD", "GitHub", "GitHub Actions", "React"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: true
---

React 프로젝트를 GitHub Actions와 GitHub Page를 이용해서 배포해보자.

나는 기존에 [오픈 소스로 개발 중인 프로젝트](https://github.com/gmlwo530/code-process-generator)를 사용했다.

- 이 글은 GitHub Actions를 이해하고 있다는 가정하에 작성한다. 만약 GitHub Actions에 관해 공부해보고 싶다면 [블로그에 정리해놓은 글]({{< relref "/posts/2021/GitHub-Actions-Learning-1.md" >}} "GitHub Actions Learning") 또는 [공식 문서](https://docs.github.com/en/actions/learn-github-actions)를 보자.

# 전체 코드

전체 코드를 먼저 보고 하나씩 의미를 알아 가보자.

<!-- TODO: 완성 코드 수정 -->

```yaml
name: Deploy

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2

      - name: Setup Node
        uses: actions/setup-node@v2.1.2
        with:
          node-version: "12.x"

      - name: Cache dependencies
        uses: actions/cache@v2
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        run: npm install

      - name: Make env file
        run: |
          touch .env
          echo REACT_APP_MODE=prod >> .env
          echo REACT_APP_GA_ID=${{ secrets.REACT_APP_GA_ID }} >> .env

      - name: Build app
        run: npm run build

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build
```

# workflow 이름 및 이벤트 정의

```yaml
name: Deploy

on:
  push:
    branches:
      - master
```

workflow의 이름은 `Deploy`로 정의하고, 이벤트는 `master` 브랜치로 푸시하는 것으로 정의하였다.

# job 정의

```yaml
jobs:
  deploy-job:
    runs-on: ubuntu-18.04
```

job의 이름은 `deploy-job`으로 정의하고, runner의 os는 `ubuntu-18.04`로 정의한다.

# step 정의

## `checkout` actions

```yaml
- uses: actions/checkout@v2
```

이 액션은 GitHub Actions의 workflow가 레포지토리에 접근 할 수 있도록 `$GITHUB_WORKSPACE`에 레포지토리를 복사해 주는 일을 해준다.
예를 들어, `/home/runner/work/my-repo-name/my-repo-name` 경로로 레포지토리를 복사해준다. ([기본 변수 참고](https://docs.github.com/en/actions/reference/environment-variables#default-environment-variables))

## `setup-node` actions

```yaml
- name: Setup Node
  uses: actions/setup-node@v2.1.2
  with:
    node-version: "12.x"
```

이 액션은 노드 환경을 세팅해준다. Actions를 실행하면 아래와 같이 log가 찍힌다.

{{< figure src="/2021/GitHub-Actions-React-Deploy/setup-node-actions-log.png" caption=" " >}}

with으로 사용된 변수들에 대해 알아보자.

- `node-version`: node 버전을 정의해주는 변수다. 이 변수는 옵션이다. (README에서 이 변수가 만약 정의되어 있지 않다면 `PATH`를 사용한다고 되어있는데 이미 정의되어있는 node 버전을 사용하는 것으로 보인다)
- `always-auth`: [npmrc](https://docs.npmjs.com/cli/v6/configuring-npm/npmrc)의 [`always-auth`](https://docs.npmjs.com/cli/v6/using-npm/config#always-auth) 변수의 값을 세팅하는 변수이다.
- `check-latest`: 이 변수의 기본값은 false인데, false로 두면 local에 캐시 된 노드 버전을 사용한다. 만약 true로 세팅하면 캐시 된 노드 버전을 확인하고 최근에 업데이트되지 않았으면, 새로 다운로드한다.
- `token`: [node-versions](https://github.com/actions/node-versions/releases)에 요청할 때 쓰이는 값인데, 이미 기본값으로 `${{ github.token }}`이 세팅되어 있어서 사용자(이 액션을 쓰는 개발자)는 신경 쓸 필요 없다.

## `cache` actions

```yaml
- name: Cache dependencies
  uses: actions/cache@v2
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

이 액션은 의존성과 빌드된 결과를 캐싱하여 workflow의 실행 시간을 향상해준다.

일반적으로 Runner에서 실행되는 Job은 새로운 가상 환경에서 실행되서, 실행될 때마다 의존성을 다운로드 해야 한다. 그렇기 때문에 시간과 비용이 많이 소모된다,

- `path`: 캐싱하고 저장할 파일, 디렉터리를 지정한다. wildcard patterns([@actions/glob](https://docs.github.com/en/actions/guides/caching-dependencies-to-speed-up-workflows#about-caching-workflow-dependencies)에서 더 자세히 알아볼 수 있다.)으로 표현 할 수 있다.

- `key`: 캐시를 저장하고 보관하기 위한 키다.

- `restore-keys`: 키에 대한 캐시가 발생하지 않을 때, 캐시를 복원하기 위해 사용될 키의 순서가 정의된 목록이다.

Node의 npm 의존성 관리 툴에 대한 구성은 [문서](https://github.com/actions/cache/blob/main/examples.md#node---npm)에 잘 정리되어 있다. 각 의존성 툴별로 각자의 방법이 있으니 [문서](https://github.com/actions/cache#implementation-examples)에서 확인하면 된다.

나의 프로젝트에서는 캐싱 되기 전에 의존성 설치가 49s가 걸렸다. 캐싱 된 후에는 41s로 줄었다.(응?)
뭔가 비약적으로 줄진 않았는데, 로그를 보니 캐싱이 되었으니 아무튼 성공이다.

❗️ 캐싱되기 전에 워크플로우를 실행하면 캐싱 관련 로그가 `Cache not found for input keys: ~`로 찍혀 있다. 나는 처음에 액션이 동작 자체가 안되는 것인 줄 알고 헷갈렸는데, 알고 보니 두 번째부터 캐싱 된 것을 잘 가져왔다.(아니 그러면 캐싱 되었다는 메시지를 띄워주는 게 더 좋지 않았을까?)

## Build app

```yaml
- name: Install dependencies
  run: npm install

- name: Make env file
  run: |
    touch .env
    echo REACT_APP_MODE=prod >> .env
    echo REACT_APP_GA_ID=${{ secrets.REACT_APP_GA_ID }} >> .env

- name: Build app
  run: npm run build
```

app을 빌드하기 위해 의존성 들을 설치한다.

`Make env file` step은 아직 좋은 방법을 못 찾은 케이스다. React에 사용할 환경 변수를 `.env`에 정의했는데, 적용하는 방법으로 위와 같이 파일을 만드는 방법밖에 생각을 못 해봤다.(좋은 방법이 있으면 소개 부탁드립니다.)

## Deploy to GitHub Page

```yaml
- name: Deploy
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./build
```

이 액션은 정적 파일들을 GiHub Pages에 배포해준다. 위의 예제는 `./build` 디렉터리를 `gh-pages` 브랜치에 배포하는 의미다.

❗️ 예제에 사용된 `GITHUB_TOKEN`은 GitHub Actions runner가 자동으로 생성해주기 때문에, 유저가 따로 설정할 필요 없다.

# GitHub Actions 실행

이제 push를 하면 Github Actions가 실행되고 배포가 끝난다.

{{< figure src="/2021/GitHub-Actions-React-Deploy/github-actions-deploy-log.png" caption=" " >}}

---

**Reference**

- [actions/checkout](https://github.com/actions/checkout)
- [actions/setup-node](https://github.com/actions/setup-node)
- [actions/cache](https://github.com/actions/cache)
- [peaceiris/actions-gh-pages@v3](https://github.com/peaceiris/actions-gh-pages)
- [About caching workflow dependecies](https://docs.github.com/en/actions/guides/caching-dependencies-to-speed-up-workflows#about-caching-workflow-dependencies)
