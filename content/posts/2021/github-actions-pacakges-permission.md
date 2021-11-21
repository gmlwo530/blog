---
title: "Permission error when push to GitHub Pacakges by GitHub Actions"
date: 2021-11-21T16:47:01+09:00
weight: 1
aliases: []
tags: ["CI/CD", "GitHub", "GitHub Actions", "GitHub Packages", "Permission"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: true
---

이 글은 나처럼 공식 문서를 전체적으로 파악하지 않은 상태에서 GitHub Actions으로 GitHub Packages에 push 하는 사람들의 삽질을 방지하기 위한 글이다.

---

어떤 테스크를 해결하기 위해 도커라이징 한 파일을 GitHub Packages에 올릴 일이 생겼다.

매번 이미지를 빌드 할 때 마다 푸시하기 귀찮았고, 이미 GitHub Actions을 CI 용으로 쓰고 있기 때문에 GitHub Actions로 docker 이미지를 푸시 하는 것을 구현하였다.

GitHub Actions에서 docker 레지스트리에 로그인하고 빌드한 뒤 푸시하는 예제는 [공식 문서](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images#publishing-images-to-github-packages)에 아주 잘 나와 있어 구현하기 편했다.

예제를 비슷하게 따라 친 다음 workflow를 올렸는데, `Build and push Docker image` 단계에서 계속 아래와 같은 오류가 발생 했다.

```bash
Error: buildx call failed with: error: denied: permission_denied: write_package
```

바로 yaml 파일의 permission이 선언 되어 있는 부분에 대해서 찾아보았다. 이 부분은 `GITHUB_TOKEN` 관련 된 권한이고, [문서](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)에 나온 내용이 맞다면 잘못 된 부분은 없었다.

```yaml
jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
```

결국, 20~30분 동안 삽질하다가 원인과 해결법을 알려줄 [문서](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#upgrading-a-workflow-that-accesses-ghcrio)를 찾았다. 해결법은 레포지토리의 설정 탭에서 GitHub Actions의 workflow에 권한을 주기만 할 정도로 간단 했다. 스크린샷과 자세한 방법은 앞에 링크 걸어 놓은 페이지를 확인하면 된다.

(아니 세팅이 필요하면 예제가 나와 있는 [Publishing images to GitHub Packages 문서](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images#publishing-images-to-github-packages)에 링크라도 걸어줘야 하는거 아닌가?)

---

**Reference**

- [Publishing images to GitHub Packages](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images#publishing-images-to-github-packages)
- [Permission for GITHUB TOKEN](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)
- [Upgrading a workflow that accesses ghcrio](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#upgrading-a-workflow-that-accesses-ghcrio)
