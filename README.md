# vive-coding-playground

바이브 코딩으로 만드는 작은 프로젝트들을 모아두는 저장소입니다.

떠오른 아이디어를 먼저 글로 적어두고, 그중 만들고 싶은 것을 AI 코딩 에이전트(Claude Code)와 함께 구현합니다. 구현한 프로젝트는 각자 독립된 저장소로 두고, 이 저장소에서는 git submodule로 참조합니다.

## 진행 방식

1. **아이디어 기록**: 아이디어와 요구사항을 [`ideas/`](ideas/)에 문서로 적습니다.
2. **구현**: 만들기로 한 아이디어는 별도 저장소에서 구현합니다. 에이전트가 계획을 세우고 작업을 나눠 진행합니다.
3. **연결**: 완성된 프로젝트를 이 저장소에 서브모듈로 추가합니다.

## 프로젝트

| 프로젝트 | 상태 | 설명 |
|---|---|---|
| [warehouse-tycoon](https://github.com/BranKein/vive-warehouse-tycoon) | 완료 | 창고를 소재로 한 웹 기반 3D 방치형 타이쿤 게임. [플레이하기](https://warehouse-tycoon.yeonhyuk.me) |
| infra (비공개) | 완료 | 프로젝트들을 AWS에 띄우는 Terraform 코드 |
| [museum](https://github.com/BranKein/vive-museum) | 구현 중 | 이 저장소의 프로젝트들을 네 가지 전시관으로 둘러보는 박물관 사이트 |

## 아이디어

| 아이디어 | 상태 | 설명 |
|---|---|---|
| [iKeyboard·iMouse](ideas/ikeyboard-imouse.md) | 아이디어 | 카메라로 책상 위 손동작을 읽어 가상 키보드·마우스 입력으로 바꾸는 실험입니다 |
| [주기적 코딩 문제 생성 및 구독 알림](ideas/periodic-coding-problems.md) | 아이디어 | 코딩·SQL 문제를 주기적으로 만들어 쌓고, 구독자에게 메일로 알립니다 |
| [AWS 배포용 Terraform 인프라](ideas/aws-terraform-infra.md) | 완료 | 다른 프로젝트들을 AWS에 띄우기 위한 Terraform 코드를 한곳에서 관리합니다 |
| [창고 타이쿤 게임](ideas/warehouse-tycoon.md) | 완료 | 창고를 소재로 한 웹 기반 3D 방치형 타이쿤 게임입니다 |
| [프로젝트 박물관 사이트](ideas/project-museum-site.md) | 구현 중 | 프로젝트들을 박물관처럼 전시하고 방문자 아이디어를 받는 사이트입니다 |

전체 목록과 상태 값의 의미는 [ideas/README.md](ideas/README.md)에 있습니다.

## 저장소 구조

```
.
├── ideas/       # 아이디어와 요구사항 문서
├── projects/    # 개별 프로젝트 (git submodule)
├── AGENT.md     # AI 에이전트가 따르는 작업 지침
└── CLAUDE.md    # Claude Code 설정 진입점
```

## 앞으로

프로젝트가 어느 정도 모이면, 이 저장소에 정적 사이트를 만들어 프로젝트들을 한곳에서 둘러볼 수 있게 할 계획입니다.
