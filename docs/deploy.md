# 배포 구조

하위 프로젝트를 웹에 띄우는 공통 방식이다. 처음 정리한 것은 `warehouse-tycoon`(2026-10-15)이고, 다른 프로젝트도 같은 구조를 쓴다. 배포를 만들거나 고칠 때만 이 문서를 읽는다.

## 한눈에

```
프로젝트 저장소                                   vive-infra (비공개, Terraform)
──────────────                                   ─────────────────────────────
v* 태그 push
  └ deploy.yml
      1. 테스트, 빌드
      2. deploy/site.json ──복사──▶ sites/<이름>/site.auto.tfvars.json
           ├ 바뀜   → main에 커밋·push ─┐
           └ 그대로 → <이름>/<태그> 태그 push ─┤
                                              └▶ apply.yml: 그 사이트만 terraform apply
      3. infra apply가 끝날 때까지 기다림 ◀──────────┘
      4. dist/를 S3에 올림
      5. CloudFront 캐시 비움
```

- 정적 사이트 하나 = S3 버킷 + CloudFront + ACM 인증서 + Cloudflare DNS(`<subdomain>.yeonhyuk.me`) + 업로드 전용 IAM 사용자. 모듈은 vive-infra의 `modules/static-site`.
- 인프라 변경은 언제나 vive-infra의 GitHub Actions에서 돈다. 로컬에서 apply하지 않는다(vive-infra의 `bootstrap/`만 예외. 처음 한 번 로컬에서 했다).
- 배포는 프로젝트 저장소에 `v*` 태그를 push하면 시작한다. 그 밖의 push는 배포하지 않는다.

## 저장소별 파일

| 저장소 | 파일 | 하는 일 |
|---|---|---|
| 프로젝트 | `.github/workflows/deploy.yml` | 위 1~5단계. 기준본은 `projects/warehouse-tycoon/.github/workflows/deploy.yml` |
| 프로젝트 | `deploy/site.json` | 프로젝트가 인프라에 바라는 값만. 지금은 `subdomain`, `price_class` |
| vive-infra | `sites/<이름>/` | 사이트 하나의 Terraform. `site.auto.tfvars.json`은 프로젝트 워크플로가 덮어쓴다 |
| vive-infra | `.github/workflows/plan.yml` | PR이면 바뀐 사이트를 plan(읽기 전용 역할) |
| vive-infra | `.github/workflows/apply.yml` | main push면 바뀐 사이트를, `<이름>/v*` 태그면 그 사이트를 apply. `modules/`나 `.github/`가 바뀌면 전부 |

## GitHub 설정

| 저장소 | Secrets | Variables |
|---|---|---|
| vive-infra | `CLOUDFLARE_API_TOKEN` (`yeonhyuk.me` DNS 편집만) | `AWS_PLAN_ROLE_ARN`, `AWS_APPLY_ROLE_ARN`, `TF_STATE_BUCKET` (bootstrap 출력) |
| 각 프로젝트 | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` (IAM 사용자 `deploy-<이름>`의 키), `INFRA_REPO_TOKEN` | `AWS_REGION`(`ap-northeast-2`), `S3_BUCKET`, `CLOUDFRONT_DISTRIBUTION_ID` (사이트 apply 출력) |

`INFRA_REPO_TOKEN`은 fine-grained PAT다. 대상 저장소는 vive-infra 하나, 권한은 Contents: Read and write, Actions: Read. 프로젝트마다 따로 발급해도 되고 같은 토큰을 써도 된다.

## 새 프로젝트에 배포 붙이기

에이전트가 할 일:

1. vive-infra에서 `sites/warehouse-tycoon`을 `sites/<이름>`으로 복사하고 바꾼다: `main.tf`의 `name`, `variables.tf`의 기본값, `versions.tf`의 backend `key`(`sites/<이름>.tfstate`)와 `Stack` 태그.
2. 프로젝트 저장소에 `deploy/site.json`을 만들고, 기준본 `deploy.yml`을 복사해 `env.SITE`를 바꾼다. 테스트·빌드 명령과 빌드 폴더(`dist/`)가 다르면 그 단계도 맞춘다.
3. 프로젝트 README에 배포 방법(태그 push)을 적는다.
4. vive-infra 변경은 브랜치에 올리고 PR을 만들어 plan 결과까지 확인한다. **main 병합은 실제 AWS·DNS를 만들기 때문에 사용자가 한다.**

사용자가 할 일 (에이전트는 하지 않는다):

1. vive-infra PR 병합 → apply가 끝나면 실행 요약에서 `bucket`, `distribution_id`를 확인한다.
2. IAM 콘솔에서 `deploy-<이름>`의 액세스 키 발급.
3. 프로젝트 저장소에 위 표의 Secrets·Variables 입력.
4. 첫 태그 push(또는 에이전트에게 맡김).

## 운영 규칙

- 태그는 `vMAJOR.MINOR.PATCH`. 사용자가 버전을 정해 배포를 요청하면 에이전트가 태그를 push하고, 워크플로가 성공했는지와 사이트가 새 빌드를 내보내는지(첫 화면의 `assets/index-*.js` 이름) 확인한다.
- 같은 프로젝트의 배포는 한 번에 하나(`concurrency: deploy`). vive-infra의 apply도 사이트마다 하나씩 돈다.
- 업로드는 해시가 붙은 `assets/`를 먼저(1년 캐시), `index.html`을 마지막에(캐시 안 함) 올린다. 그래서 CloudFront는 `/`와 `/index.html`만 비운다.
- 정적 사이트가 아닌 것(서버, DB 등)이 필요해지면 vive-infra에 새 모듈을 만들고 이 문서에 추가한다.
