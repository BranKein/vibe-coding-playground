# 검색 노출 (SEO)

웹으로 공개하는 하위 프로젝트는 모두 이 문서대로 검색 노출을 붙인다. 2026-10-20에 박물관과 `warehouse-tycoon`에 처음 적용했고, 사용자가 앞으로의 웹 프로젝트에도 같은 방식을 쓰라고 했다. 웹 프로젝트를 새로 구현하거나 배포를 붙일 때 이 문서를 읽는다.

대상 검색엔진은 Google과 네이버다.

## 원칙

- **언어마다 주소를 따로 둔다(경로 방식).** `/ko/`, `/en/`처럼 같은 출처 안의 경로로 나누고, 각 페이지의 제목·설명·본문은 그 언어로만 쓴다. 서로 hreflang으로 가리키고 `x-default`는 `/en/`. 그러면 검색 결과에 사용자 언어에 맞는 쪽만 한 언어로 나온다. 하위 도메인으로 나누지 않는다. 출처가 달라지면 로컬 저장이 따로 놀고 인증서·DNS·OAuth·서치 콘솔 등록이 늘어난다. 루트 `/`는 마지막으로 고른 언어(없으면 브라우저 언어)로 보내기만 하고 `noindex`. 사용자가 2026-10-24에 정했고 `warehouse-tycoon`에 처음 적용했다.
  - 아직 한 주소에 한영을 병기한 사이트(박물관, 홈)는 그대로 두되, 새로 만들거나 손볼 때 이 방식으로 옮긴다.
  - S3 + CloudFront에서는 루트에만 `index.html`이 채워지므로, 배포 때 언어 페이지를 S3 키 `ko/`, `ko`에도 올린다(`warehouse-tycoon`의 `deploy.yml` 참고).
- **검색엔진이 읽을 실제 글자를 HTML에 둔다.** 화면을 JS나 canvas로만 그리면 검색엔진에는 빈 페이지로 보인다. 숨김 텍스트(`display:none`, 화면 밖 배치)는 감점되므로 쓰지 않는다. 대신:
  - 게임·canvas 앱: 첫 화면이 뜨기 전 보이는 로딩·소개 블록(제목, 한영 소개, "PC 권장" 등)을 `index.html`에 두고 앱이 시작되면 지운다.
  - JS로 목록을 그리는 페이지: 빌드 때 같은 컨테이너에 목록을 미리 채워 두고, JS 렌더가 덮어쓰게 한다.
- **문구는 `deploy/exhibit.json`의 한영 제목·설명을 재사용한다.** 박물관 전시 정보와 검색 문구를 한곳에서 관리한다.

## 체크리스트

각 페이지 `<head>`:

- [ ] `<html lang>`: 그 페이지의 언어
- [ ] `<title>`: 그 언어로 `이름 — 한 줄 소개` (병기 사이트는 `한국어 이름 · English Name — 한 줄 소개`)
- [ ] `<meta name="description">`: 그 언어로 (병기 사이트는 한국어 설명 뒤에 영어 설명)
- [ ] `<link rel="canonical">`: 그 페이지의 절대 주소. 같은 내용을 여러 주소로 보여 주면 대표 주소 하나를 가리킨다
- [ ] `<link rel="alternate" hreflang>`: 언어 페이지마다 `ko`, `en`, `x-default`(→ `/en/`). 사이트맵에도 `xhtml:link`로 같은 것을
- [ ] OG 태그: `og:type`, `og:site_name`, `og:title`, `og:description`, `og:url`, `og:image`(절대 주소, 보통 `deploy/thumbnail.jpg`를 복사), `twitter:card`(`summary_large_image`)
- [ ] JSON-LD 구조화 데이터: 게임은 `VideoGame`, 일반 사이트는 `WebSite`(목록이 있으면 `ItemList`). JSON 안의 `<`는 이스케이프한다
- [ ] 소유 확인 메타 태그: 빌드 환경 변수 `GOOGLE_SITE_VERIFICATION`, `NAVER_SITE_VERIFICATION`이 있을 때만 `google-site-verification`, `naver-site-verification` 메타를 넣는다

사이트 루트:

- [ ] `robots.txt`: 전체 허용 + `Sitemap: https://<subdomain>.yeonhyuk.me/sitemap.xml`
- [ ] `sitemap.xml`: 검색에 나오길 바라는 페이지만. 리다이렉트용 페이지는 뺀다

그 밖:

- [ ] 검색에 나오면 안 되는 페이지(공유 미리보기용, 리다이렉트용)에는 `<meta name="robots" content="noindex">`
- [ ] `deploy.yml`의 빌드 단계 env에 `GOOGLE_SITE_VERIFICATION: ${{ vars.GOOGLE_SITE_VERIFICATION }}`, `NAVER_SITE_VERIFICATION: ${{ vars.NAVER_SITE_VERIFICATION }}`
- [ ] 위 태그들이 빌드 결과에 들어가는지 확인하는 테스트
- [ ] 프로젝트 README에 "검색 노출" 절(환경 변수, 생성되는 파일), `PLAN.md`에 결정 사항

## 참고 구현

| 형태 | 저장소 | 볼 곳 |
|---|---|---|
| Vite 언어별 페이지(게임) | `projects/warehouse-tycoon` | `ko/index.html`·`en/index.html`(태그, 로딩 블록), `index.html`(언어로 보내는 루트), `src/i18n`의 `page` 옵션(언어 버튼이 주소를 옮김), `vite.config.ts`(소유 확인 태그 플러그인 `verificationTags`), `public/robots.txt`·`sitemap.xml`, `tests/app/seo.test.ts` |
| 빌드 스크립트가 있는 여러 페이지 | `projects/museum` | `scripts/build.mjs`(페이지별 head, 미리 채운 목록, JSON-LD, robots·sitemap 생성), `scripts/lib.mjs`의 `ogTags()`, `tests/seo.test.mjs` |

## 사용자가 할 일

에이전트는 하지 않는다. 구현을 마치면 사용자에게 아래를 한국어로 안내한다.

1. **Google Search Console**: "URL 접두어" 속성으로 `https://<subdomain>.yeonhyuk.me/` 추가 → HTML 태그 방식의 `content` 값을 프로젝트 저장소 Variables `GOOGLE_SITE_VERIFICATION`에 입력.
2. **네이버 서치어드바이저**: 사이트 등록 → HTML 태그의 `content` 값을 `NAVER_SITE_VERIFICATION`에 입력.
3. 태그 push로 배포(요청하면 에이전트가 한다).
4. 배포 뒤 두 곳에서 소유 확인을 누르고 `sitemap.xml` 제출. 색인까지는 며칠~몇 주 걸린다.

`yeonhyuk.me` 도메인 속성 하나로 묶는 방법도 있지만 Cloudflare DNS TXT 레코드가 필요해 vibe-infra를 거쳐야 하므로 쓰지 않는다.
