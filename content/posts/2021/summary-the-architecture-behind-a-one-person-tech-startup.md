---
title: "[정리] 1인 기술 스타트업의 아키텍처 스택"
date: 2021-04-24T14:34:02+09:00
weight: 1
aliases: []
tags: ["글", "요약", "번역"]
author: ["Maru"]
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
comments: true
---

최근에 [GeekNews](https://news.hada.io/)에서 [재밌는 글](https://news.hada.io/topic?id=4055)을 봤다. SaaS를 운영하는 1인 개발자가 설계한 아키텍처에 대한 설명인데, GeekNews에 공유해주신 분이 요약 해주신 내용만으로도 굉장히 퀄리티 높은 글인 것을 느낄 수 있었다. 그래서 나도 원문을 읽으며 나름대로 정리를 해볼려고 한다.

참고로 본인이 1인 기술 스타트업을 만들 생각이 없어도 개발자라면 한 번쯤 읽어보면 도움이 되는 글이라고 생각한다.

**원문 글: [https://anthonynsimon.com/blog/one-man-saas-architecture/](https://anthonynsimon.com/blog/one-man-saas-architecture/)**


## 서론

- 원문 글의 필자는 독일에서 1인 기업을 운영하고 있고 스트레스 없이 자기 자본으로만 천천히 운영하고 있다고 한다.

- 혼자서 운영하는 만큼 다양한 오픈 소스와 서비스들을 사용하게 되었는데, 이것 없이는 목표를 이룰 수 없었고 이걸 사용함에 있어서 거인의 어깨에 서있는 느낌을 받았다고 한다.(개인적으로 이 표현이 좋았다.)

- *상황에 따라 기술적 선택은 다르다.* 이 글은 본인이 선택한 기술들을 공유하는 것 뿐이니, **진리라고 생각하지 말자.**

- 글쓴이는 AWS에서 Kubernetes를 사용하는데, 이것은 단지 이전 회사와 팀에서 몇 년동안 삽질을 하며 내공을 쌓았기 때문에 쓴 것이다. 이 글을 보는 사람들은 자신에게 익숙한 기술을 사용해서 생산성을 저하시키는 방향을 가지 않도록 하자.


## 프로젝트의 구조

- 글쓴이가 만든 [PanelBear](https://panelbear.com/)라는 서비스로 프로젝트 구조를 설명함.

- django monolith 구조, app DB는 Postgres, analytics data는 ClickHouse, 캐싱은 Redis, 테스크 스케줄링은 Celery를 사용하고 쿠버네티스(EKS)에서 운영

- Monolithic 구조이므로 Django는 Rails나 Laravel 등으로 대체 가능하다.

- 여기서 흥미로운 부분은 autoscaling, ingress, TLS certificates, failover, logging, monitoring 같은 것들이 서로 결합되고 자동화 되는 부분이다.

- 이 세팅은 다양한 프로젝트에서 사용했다. 이 세팅 덕분에 비용을 줄이고 검증을 쉽게 할 수 있었다.(Dockerfile을 작성하고 git push만 하면 된다.)

- 실제로 인프라를 관리하는 시간은 한 달에 0~2시간 뿐이다. 덕분에 피처 개발, CS, 사업의 성장에 집중 할 수 있었다.

- *"Kubernetes makes the simple stuff complex, but it also makes the complex stuff simpler"*


## Automatic DNS, SSL, and Load Balancing

- 첫 번쨰 주제: 클러스터에서 트래픽을 어떻게 받을 것인가?

- 클러스터는 private 네트워크에 있다.

- NLB(AWS L4 Network Load Balancer)로 트래픽을 향하게 하는 Cloudflare proxying이 존재한다. 이 로드 밸런서는 public internet과 private network의 브릿지 역할을 한다.

- 요청이 들어오면 로드밸러스가 쿠버네티스 클러스터 노드 중 하나로 포워드한다.

- [ingress-nginx](https://github.com/kubernetes/ingress-nginx)를 사용해서 쿠버네티스가 어느 서비스로 포워드 할지 정해준다. ingress-nginx는 클러스터의 입구다.

- NGINX는 요청을 일치하는 컨테이너(해당 글에서는 Uvicorn으로 서빙되는 django)로 전달하기 전에 rate-limiting과 traffic shaping 규칙을 적용한다.

- 몇 개의 terraform/kubernetes 파일로 전체 프로젝트에 적용 가능하기 때문에 한 번 설정하고 잊어버릴 수 있다.

- 새로운 프로젝트를 배포 할 때, 필수적인 설정을 20줄만 적어주면 된다.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
 namespace: example
 name: example-api
annotations:
 kubernetes.io/ingress.class: "nginx"
 nginx.ingress.kubernetes.io/limit-rpm: "5000"
 cert-manager.io/cluster-issuer: "letsencrypt-prod"
 external-dns.alpha.kubernetes.io/cloudflare-proxied: "true"
spec:
tls:
- hosts:
   - api.example.com
 secretName: example-api-tls
rules:
- host: api.example.com
 http:
   paths:
     - path: /
       backend:
         serviceName: example-api
         servicePort: http
```


## Automated rollouts and rollbacks

- GitHub의 마스터 브랜치에 프로젝트를 push하면 GitHub Actions의 CI 파이프라인이 실행 된다.

- CI 파이프라인은 아래와 같은 과정을 거친다.
    1. 코드베이스로 체크한다.(Format, Lint, Check types)
    2. Docker-compose로 완벽한 환경을 만들어서 end-to-end test를 거친다.
    3. 1, 2번이 통과되면 새로운 Docker image를 만들어서 [ECR](https://aws.amazon.com/ko/ecr/)로 push 한다.

- ECR로 push 되면 [flux](https://fluxcd.io/)라는 컴포넌트가 현재 클러스터에서 실행되고 있는 이미지와 최근 등록된 이미지를 동기화 시켜준다.

- Flux는 새로운 Docker image가 생겼을 때 자동으로 incremental rollout을 하고 "Infrastructure Monorepo"에 기록한다.


## Let it crash

- 쿠버네티스는 비정상 적인 pod이 고쳐질 동안 트래픽을 정상적인 pod으로 옮기는 것에 뛰어나다.

## Horizontal autoscaling

- 컨테이너들은 CPU와 Memory 사용량에 따라 오토스케일링 된다.

- 노드당 많은 Pods이 클러스터에 생기면 자동적으로 클러스터 용량을 늘리고, 아닐 경우 축소한다.

- 오토스케일링은 아래와 같이 한다
    ```yaml
    apiVersion: autoscaling/v1
    kind: HorizontalPodAutoscaler
    metadata:
    name: panelbear-api
    namespace: panelbear
    spec:
    scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: panelbear-api
    minReplicas: 2
    maxReplicas: 8
    targetCPUUtilizationPercentage: 50
    ```

- PanelBear는 API pod을 CPU 사용량에 따라 2개에서 9개로 복제본을 조정한다.

## Static assets cached by CDN

- Cloudflare를 사용해서 DNS, 그리고 요청에 대한 CDN, DDoS 보호를 함.

- 단순하게 HTTP cache headers를 사용해서 Cloudflare 캐시를 컨트롤 하게 한다.
    ```python
    # Cache this response for 5 minutes
    response["Cache-Control"] = "public, max-age=300"
    ```

- [Whitenoise](https://github.com/evansd/whitenoise)를 사용해서 app container로 부터 정적 파일을 서빙한다.

- 이걸 사용하면 Nginx/Cloudfront/S3에 정적파일을 업로드 할 필요가 없어서 간단하다.

- 지금까지 이런 방식은 문제가 없었고 성능도 뛰어나면서 간단했다.

- 정적 웹사이트로는 NextJS를 사용했다. Cloudfront/S3 또는 Netlify/Vercel을 통해 서빙 할 수도 있지만, 클러스터의 하나의 컨테이너로 실행되게 하고 정적 assets을 Cloudflare에 캐싱하는게 더 쉬웠다.


## Application data caching

- 파이썬에서 제공하는 [LRU(Least Recently Used)](https://docs.python.org/3/library/functools.html#functools.lru_cache) cache를 사용해서 네트워크 콜을 제로로 할 수 있었다.

- 대부분의 endpoints는 인 클러스터 Redis로 캐싱 했다. Redis는 모든 Django instance가 공유 했다. 인스턴스가 재배포 되면 메모리 캐시를 모두 지웠다.

## Per endpoint rate-limiting

- Kubernetes의 nginx-ingress로 전역적인 rate limits를 설정하긴 했지만, 특정 endpoint나 메서드에도 limit을 설정하기 위해 [Django Ratelimit](https://django-ratelimit.readthedocs.io/en/stable/) 라이브러리를 사용했다.

- user의 ip를 각 request에 따라 Redis에 저장해서 조건에 맞게 요청을 제한한다. 아래 예제는 한 유저가 1분에 요청을 5번 넘게 보내면 `HTTP 429 Too Many Requests status` 상태 코드를 반환하는 예제이다
    ```python
    class MySensitiveActionView(RatelimitMixin, LoginRequiredMixin):
        ratelimit_key = "user_or_ip"
        ratelimit_rate = "5/m"
        ratelimit_method = "POST"
        ratelimit_block = True

        def get():
            ...

        def post():
            ...
    ```

## App administration

- Django가 기본으로 제공해주는 admin을 사용합니다.

## Running scheduled jobs

- 고객용 데일리 리포트, 15분 마다 사용 통계 계산, 스태프용 이메일 보내기 등 다양한 예약 작업이 있다.

- Celery workers와 Celery beat를 cluster에서 실행한다. Task queue로는 Redis를 사용한다.

- 만약 예약 된 작업이 정상적으로 작동하지 않으면 [Healthchecks](https://healthchecks.io/)를 통해 slack/sms/email로 알림을 받는다.

- 아래는 예약 된 작업의 상태를 체크하는 모니터링 코드다.
    ```python
    def some_hourly_job():
        # Task logic
        ...

        # Ping monitoring service once task completes
        TaskMonitor(
        name="send_quota_depleted_email",
        expected_schedule=timedelta(hours=1),
        grace_period=timedelta(hours=2),
        ).ping()
    ```

## App configuration

- 쿠버네티스의 configmap을 사용하여 환경 변수를 오버라이딩하고 django의 `settings.py`에 환경 변수를 정의 한다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
 namespace: panelbear
 name: panelbear-webserver-config
data:
 INVITE_ONLY: "True"
 DEFAULT_FROM_EMAIL: "The Panelbear Team <support@panelbear.com>"
 SESSION_COOKIE_SECURE: "True"
 SECURE_HSTS_PRELOAD: "True"
 SECURE_SSL_REDIRECT: "True"
```

```python
INVITE_ONLY = env.str("INVITE_ONLY", default=False)
```

## Keeping secrets

- 쿠버네티스의 [kubeseal](https://github.com/bitnami-labs/sealed-secrets)을 사용해서 키들을 암호화한다. kubeseal은 비대칭 암호화를 한다. 그리고 오직 클러스터만 복호화 키에 접근할 권한을 가지고 있다. 암호화하면 아래와 같이 보인다.
    ```yaml
    apiVersion: bitnami.com/v1alpha1
    kind: SealedSecret
    metadata:
        name: panelbear-secrets
        namespace: panelbear
    spec:
        encryptedData:
            DATABASE_CONN_URL: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEq...
            SESSION_COOKIE_SECRET: oi7ySY1ZA9rO43cGDEq+ygByri4OJBlK......
    ```

- 클러스터 안의 Secret을 보호하기 위해 AWS KMS을 사용한다. 쿠버네티스 클러스터를 생성할 때 세팅한다.

- 과정을 나타내면 아래와 같다.
    1. 암호화 할 환경 변수를 쿠버네티스 manifest에 작성한다.
    2. 커밋하기 전에 암호화를 하고 푸시한다.
    3. 암호화 된 변수는 배포되고, 클러스터는 자동으로 컨테이너를 실행하기 전에 암호화 된 변수를 복호화 한다.


## Relational data: Postgres

- 처음에는 바닐라 postgres 컨테이너를 쿠버네티스 클러스터에서 실행했다.

- 프로젝트가 성장함에 따라 AWS RDS로 이전했다.

- RDS를 사용함으로써 지속적인 보안 업데이트, 암호화 된 백업 같은 장점을 누릴 수 있었다.


## Columnar data: ClickHouse

- 프로젝트의 분석 데이터를 실시간으로 쿼리하고 효율적이게 저장하기 위해 [ClickHouse](https://clickhouse.tech/)를 이용했다.

- 이 서비스는 엄청 빠르고, 데이터 구조를 잘 잡음으로써 압축률만 잘 잡으면 저장 비용이 낮아짐. 이건 수익률을 높임.

- 현재 ClickHouse 인스턴스를 쿠버네티스 클러스터에 호스트해서 사용하고 있다.

- 쿠버네티스 CronJob을 사용해서 주기적으로 효율적인 columnar format으로 백업한 데이터를 S3로 보냄.

- 재난 상황이 일어났을 때 복원 할 수 있게 스크립트를 짜놓음

- 유일하게 써보지 않은 툴이였는데, 문서가 잘 정리되어 있어서 금방 사용하게 됨.


## DNS-based service discovery

- 각 컨테이너끼리 쿠버네티스 내에서 서로 통신은 DNS 레코드 기반으로 이루어진다.
    ```text
    redis://redis.weekend-project.svc.cluster:6379
    ```

- DNS 레코드는 쿠버네티스가 자동으로 관리한다.

- 쿠버네티스는 자동으로 DNS 레코드를 건강한 Pod으로 동기화 한다.

- 위의 내용의 원리를 알고 싶으면, [잘 설명하고 있는 글](https://medium.com/google-cloud/understanding-kubernetes-networking-services-f0cb48e4cc82) 하나를 추천한다.

## Version-controlled infrastructure

- version-controlled를 간단한 커맨드로 하고 싶어서 Docker, Terraform 그리고 Kubernetes manifest를 하나의 레포에 두었다.

- 이 레포는 여러 프로젝트에 대한 인프라를 가지고 있다. 즉, 여러 프로젝트들은 서로 다른 레포에 저장되어 있지만, 각 프로젝트의 인프라에 대한 관리는 이 단일 레포에서 하는 것이다.

- [The Twelve-Factor App](https://12factor.net/)에 대해 알고 있다면 이 구조가 꽤 괜찮아 보일 것이다.

- 이렇게 해서 단순한 커맨드로 인프라로 관리할 수 있게 되었다. 아래는 인프라 단일 레포의 구조를 예시로 나타냈다.
    ```text
    # Cloud resources
    terraform/
        aws/
            rds.tf
            ecr.tf
            eks.tf
            lambda.tf
            s3.tf
            roles.tf
            vpc.tf
        cloudflare/
            projects.tf

    # Kubernetes manifests
    manifests/
        cluster/
            ingress-nginx/
            external-dns/
            certmanager/
            monitoring/

        apps/
            panelbear/
                webserver.yaml
                celery-scheduler.yaml
                celery-workers.yaml
                secrets.encrypted.yaml
                ingress.yaml
                redis.yaml
                clickhouse.yaml
            another-saas/
            my-weekend-project/
            some-ghost-blog/

    # Python scripts for disaster recovery, and CI
    tasks/
    ...

    # In case of a fire, some help for future me
    README.md
    DISASTER.md
    TROUBLESHOOTING.md
    ```


## Terraform for cloud resources

- [Terraform](https://www.terraform.io/)을 사용해서 대부분의 클라우드 리소스를 관리한다. 

- 문서화와 리소스를 추적하고 인프라를 설정하는 것에 도움을 주었다.


## Kubernetes manifests for app deployments

- 쿠버네티스 manifest들은 모든 YAML 파일로 기술되어 있고 인프라 단일 레포에 위치해 있다.

- 두 개의 레포로 분리 했다: `cluster` & `app`

- `cluster` 디렉토리 안에는 nginx-ingress, encryped secrets, premotheus scaper 같은 클러스터 전체 서비스의 설정에 대한 내용이 있다.

- `app` 디렉토리 안에는 프로젝트와 관련된 내용이 있다.

## Subscriptions and Payments

- [Stripe Checkout](https://stripe.com/payments/checkout)으로 결제 기능을 관리한다. 

- 결제 정보에 접근할 필요가 없으므로 프로덕트 개발에 더 집중할 수 있게 됐다.

- 새로운 고객 세션을 만든 뒤 Stripe가 호스트한 페이지로 리다이렉트 시킨 후 webhook을 통해 결제의 결과에 따라 나의 데이터베이스를 업데이트 하기만 하면 된다.

- 물론 webhook과 관련되서 중요한 부분이 있지만, Strip 문서가 잘 다뤄주고 있다.


## Logging

- logging을 하기 위해 특별한 agent 같은 건 사용하지 않음.

- stdout을 통해 찍힌 로그를 쿠버네티스가 자동으로 수집하게 하고, 로그를 rotate 함.

- ElasticSearch/Kibana 같은 자동으로 로그를 모아주는 툴을 쓸 수도 있지만, 지금은 간단하게 유지하고 싶어함.

- 로그를 추적하기 위해 쿠버네티스 CLI 툴인 [stern](https://github.com/wercker/stern)을 사용함.


## Monitoring and alerting

- 처음에는 Prometheus/Grafana를 자체 호스팅해서 클러스터와 애플리케이션 수치를 모니터링 했다.

- 하지만 클러스터가 문제가 생기면 모니터링 툴의 alerting system도 다운되서 좋지 않다고 느꼈다.

- 그래서 [New Relic](https://newrelic.com/)로 툴을 변경했다.

- 클러스터 내의 모든 서비스는 Prometheus integration을 가지고 있어서 자동으로 기록을 New Relic으로 보낼 수 있었다. 그래서 New Relic으로 마이그레이션 할 때 단순히 Prometheus Docker image만 사용하면 됐었다.

- 장고 앱의 수치를 기록하는 방법으로는 [django-prometheus](https://github.com/korfuri/django-prometheus) 라이브러리를 사용했다.


## Error tracking

- Error tracking 툴로는 [Sentry](https://sentry.io/welcome/)를 사용했다.

- 장고 앱에서 샌트리를 사용하는 방법은 간단하다.
    ```python
    SENTRY_DSN = env.str("SENTRY_DSN", default=None)

    # Init Sentry if configured
    if SENTRY_DSN:
        sentry_sdk.init(
            dsn=SENTRY_DSN,
            integrations=[DjangoIntegration(), RedisIntegration(), CeleryIntegration()],
            # Do not send user PII data to Sentry
            # See also inbound rules for special patterns
            send_default_pii=False,
            # Only sample a small amount of performance traces
            traces_sample_rate=env.float("SENTRY_TRACES_SAMPLE_RATE", default=0.008),
        )
    ```

- 또한 Slack `#alerts` 채널을 사용해서 모든 알림(downtime, cron job failures, security alerts, performance regressions, application exceptions, and whatnot.)을 이 채널로 오도록 했다.

## Profiling and other goodies

- [cProfile](https://docs.python.org/3/library/profile.html), [snakeviz](https://jiffyclub.github.io/snakeviz/), [Django debug toolbar](https://django-debug-toolbar.readthedocs.io/en/latest/)를 통해 profiling을 했다.

## 마치면서 (개인적인 생각)

저자도 앞에서 언급한 것과 같이 위에서 소개 된 도구와 방법은 정답이 아니다. 각자의 상황 또는 프로젝트의 특성에 따라 정답은 바뀔 수 있다.

그러나 프로젝트가 진행 될 때 공통적으로 필요한 기능들은 위에서 대부분 언급했고 그에 대한 하나의 선택지를 알려줬다고 생각한다.

이 글을 쓰는 나를 포함해서 글을 읽는 여러분들이, 원문의 저자가 공유해준 경험과 선택지를 참고 삼아 자신만의 정답을 찾았으면 좋겠다.

---

**Reference**

- [https://anthonynsimon.com/blog/one-man-saas-architecture/](https://anthonynsimon.com/blog/one-man-saas-architecture/)