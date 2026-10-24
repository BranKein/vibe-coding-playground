# 배포 구조

하위 프로젝트를 웹에 띄우는 공통 방식이다. 처음 정리한 것은 `warehouse-tycoon`(2026-10-15)이고, 다른 프로젝트도 같은 구조를 쓴다. 배포를 만들거나 고칠 때만 이 문서를 읽는다.

## 한눈에

```
프로젝트 저장소                                   vibe-infra (비공개, Terraform)
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

- 정적 사이트 하나 = S3 버킷 + CloudFront + ACM 인증서 + Cloudflare DNS(`<subdomain>.yeonhyuk.me`) + 업로드 전용 IAM 사용자. 모듈은 vibe-infra의 `modules/static-site`.
- 인프라 변경은 언제나 vibe-infra의 GitHub Actions에서 돈다. 로컬에서 apply하지 않는다(vibe-infra의 `bootstrap/`만 예외. 처음 한 번 로컬에서 했다).
- 배포는 프로젝트 저장소에 `v*` 태그를 push하면 시작한다. 그 밖의 push는 배포하지 않는다.
- 배포가 끝나면 박물관(vibe-museum)에 `repository_dispatch`(`exhibit-updated`)를 보내 전시를 다시 빌드하게 한다. 박물관에 전시하려면 프로젝트에 `deploy/exhibit.json`을 둔다(형식은 vibe-museum README).

## 저장소별 파일

| 저장소 | 파일 | 하는 일 |
|---|---|---|
| 프로젝트 | `.github/workflows/deploy.yml` | 위 1~5단계. 기준본은 `projects/warehouse-tycoon/.github/workflows/deploy.yml` |
| 프로젝트 | `deploy/site.json` | 프로젝트가 인프라에 바라는 값만. 지금은 `subdomain`, `price_class` |
| 프로젝트 | `deploy/exhibit.json` | 박물관 전시 정보(제목·설명 한영, 기술 스택, 썸네일, PC 권장). 선택 |
| vibe-infra | `sites/<이름>/` | 사이트 하나의 Terraform. `site.auto.tfvars.json`은 프로젝트 워크플로가 덮어쓴다 |
| vibe-infra | `.github/workflows/plan.yml` | PR이면 바뀐 사이트를 plan(읽기 전용 역할) |
| vibe-infra | `.github/workflows/apply.yml` | main push면 바뀐 사이트를, `<이름>/v*` 태그면 그 사이트를 apply. `modules/`나 `.github/`가 바뀌면 전부 |

## GitHub 설정

| 저장소 | Secrets | Variables |
|---|---|---|
| vibe-infra | `CLOUDFLARE_API_TOKEN` (`yeonhyuk.me` DNS 편집만) | `AWS_PLAN_ROLE_ARN`, `AWS_APPLY_ROLE_ARN`, `TF_STATE_BUCKET` (bootstrap 출력) |
| 각 프로젝트 | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` (IAM 사용자 `deploy-<이름>`의 키), `INFRA_REPO_TOKEN` | `AWS_REGION`(`ap-northeast-2`), `S3_BUCKET`, `CLOUDFRONT_DISTRIBUTION_ID` (사이트 apply 출력), `GOOGLE_SITE_VERIFICATION`·`NAVER_SITE_VERIFICATION` (검색엔진 소유 확인, 선택) |

`INFRA_REPO_TOKEN`은 fine-grained PAT다. 대상 저장소는 vibe-infra 하나, 권한은 Contents: Read and write, Actions: Read. 프로젝트마다 따로 발급해도 되고 같은 토큰을 써도 된다.

박물관에 신호를 보내는 단계는 `MUSEUM_DISPATCH_TOKEN`(없으면 `INFRA_REPO_TOKEN`)을 쓴다. 그 토큰의 대상 저장소에 vibe-museum을 더하고 Contents: Read and write를 주면 된다. 권한이 없으면 경고만 남기고 배포는 성공으로 끝난다.

## 새 프로젝트에 배포 붙이기

에이전트가 할 일:

1. vibe-infra에서 `sites/warehouse-tycoon`을 `sites/<이름>`으로 복사하고 바꾼다: `main.tf`의 `name`, `variables.tf`의 기본값, `versions.tf`의 backend `key`(`sites/<이름>.tfstate`)와 `Stack` 태그.
2. 프로젝트 저장소에 `deploy/site.json`을 만들고, 기준본 `deploy.yml`을 복사해 `env.SITE`를 바꾼다. 테스트·빌드 명령과 빌드 폴더(`dist/`)가 다르면 그 단계도 맞춘다.
3. 프로젝트 README에 배포 방법(태그 push)을 적는다.
   - 공개하는 웹 프로젝트면 [seo.md](seo.md)의 체크리스트도 함께 적용한다.
4. vibe-infra 변경은 브랜치에 올리고 PR을 만들어 plan 결과까지 확인한다. **main 병합은 실제 AWS·DNS를 만들기 때문에 사용자가 한다.**

사용자가 할 일 (에이전트는 하지 않는다):

1. vibe-infra PR 병합 → apply가 끝나면 실행 요약에서 `bucket`, `distribution_id`를 확인한다.
2. IAM 콘솔에서 `deploy-<이름>`의 액세스 키 발급.
3. 프로젝트 저장소에 위 표의 Secrets·Variables 입력.
4. 첫 태그 push(또는 에이전트에게 맡김).

## 검색 노출 (SEO)

공개하는 웹 프로젝트는 배포를 붙일 때 [seo.md](seo.md)대로 검색 노출도 붙인다. 소유 확인 값은 위 Variables 표의 `GOOGLE_SITE_VERIFICATION`, `NAVER_SITE_VERIFICATION`.

## 운영 규칙

- 태그는 `vMAJOR.MINOR.PATCH`. 사용자가 버전을 정해 배포를 요청하면 에이전트가 태그를 push하고, 워크플로가 성공했는지와 사이트가 새 빌드를 내보내는지(첫 화면의 `assets/index-*.js` 이름. `warehouse-tycoon`은 언어 페이지라 `assets/main-*.js`) 확인한다.
- 같은 프로젝트의 배포는 한 번에 하나(`concurrency: deploy`). vibe-infra의 apply도 사이트마다 하나씩 돈다.
- 업로드는 해시가 붙은 `assets/`를 먼저(1년 캐시), `index.html`을 마지막에(캐시 안 함) 올린다. 그래서 CloudFront는 `/`와 `/index.html`만 비운다.
- 정적 사이트가 아닌 것(서버, DB 등)이 필요해지면 vibe-infra에 새 모듈을 만들고 이 문서에 추가한다.

## 서버리스 스택 (2026-10-16)

박물관의 방문자 아이디어처럼 정적 사이트가 아닌 자원은 vibe-infra의 `sites/<이름>/`에 같은 방식(PR plan → main apply)으로 둔다. 첫 예가 `sites/vibe-museum-ideas`(API Gateway + Lambda + DynamoDB + SES).

- 함수 코드는 프로젝트 저장소에 두고, Terraform은 자리표시 코드로 함수를 만든 뒤 코드는 건드리지 않는다(`ignore_changes`). 프로젝트의 배포 워크플로가 `aws lambda update-function-code`로 올린다. 배포 사용자에게 그 함수만 고칠 권한을 준다.
- 비밀 값은 SSM SecureString 파라미터로 만들고 값은 `unset`으로 둔다. 실제 값은 사용자가 직접 넣는다.
- 이름은 `vibe-`로 시작한다(2026-10-24 전에 만든 것은 `vive-`). apply 역할은 `vibe-*`, `vive-*` 이름의 DynamoDB, Lambda, 역할, SSM 파라미터, 로그 그룹, EventBridge 규칙과 API Gateway, SES 신원만 다룰 수 있다.
- apply 역할의 권한을 넓히는 것은 `bootstrap/` 변경이라 사용자가 로컬에서 apply한다. 그 전에는 해당 스택 PR을 병합하지 않는다.
