# 루트 도메인 소개 사이트

- 상태: 구현 중
- 작성일: 2026-10-20
- 주소: (예정) https://yeonhyuk.me
- 저장소: https://github.com/BranKein/vive-home (`projects/home`)
- 전시 제목: 소개 페이지
- 전시 설명: yeonhyuk.me 첫 화면. 만든 사람과 프로젝트들을 소개하고, 개인정보처리방침과 연락처를 둔다.
- 영어 제목: Home
- 영어 설명: The front page of yeonhyuk.me, introducing the maker and the projects, with the privacy policy and contact.

## 개요

지금 `yeonhyuk.me` 루트 주소에는 아무 사이트도 없다. 애드센스는 하위 도메인을 따로 등록할 수 없어 `yeonhyuk.me`로 등록·심사받아야 하고, `ads.txt`도 루트에 있어야 한다. 그래서 루트에 작은 소개 사이트를 둔다. 박물관·창고 타이쿤 광고 계획은 이 사이트가 먼저 떠야 진행된다.

## 요구사항

사용자가 말한 내용:

- (2026-10-20) 박물관과 창고 타이쿤 둘 다에 애드센스 광고를 넣는다. 박물관 3D 전시관은 제외(에이전트 제안을 사용자가 수락).
- (2026-10-20) 루트 주소에는 간단한 소개 페이지를 둔다.
- (2026-10-20) 광고 자리는 자동 광고가 아니라 직접 정한다.

에이전트 제안:

- 내용: 본인 소개, 프로젝트 링크(박물관과 같은 원천: `.gitmodules`, `deploy/exhibit.json`), 개인정보처리방침(쿠키·광고·분석 고지), 연락처. 한영 병기.
- 파일: `index.html`, `privacy.html`, `ads.txt`, `robots.txt`, `sitemap.xml`. SEO는 [docs/seo.md](../docs/seo.md)대로.
- 인프라: vive-infra `modules/static-site`에 루트(apex) 도메인 지원을 더하고 `sites/home`을 만든다.
- 이 사이트 자체에는 광고를 넣지 않는다.

## 미정 사항

- 애드센스 게시자 ID (`ca-pub-…`, 사용자가 가입 뒤 전달)

## 메모

- (2026-10-20) 사용자 결정: 파스텔 벤토 디자인(프로필 사진 색), 기술 뱃지 없음, 이름 김연혁 · Yeonhyuk Kim, 공개 메일 yeonhyuk.public@gmail.com, 한 줄 소개 "백엔드부터 인프라까지, 만들고 띄우는 일을 좋아하는 개발자입니다." 자세한 결정은 저장소 PLAN.md.

- 애드센스 전체 진행 순서: 사용자 저장소 생성·애드센스 가입 → 루트 사이트 → 두 사이트 광고 칸 → 광고 단위 생성·심사 신청(사용자).
- 광고 칸은 게시자 ID와 단위 ID가 빌드 환경 변수에 있을 때만 들어가게 한다(소유 확인 태그와 같은 방식).
