# 문서 가이드

이 프로젝트는 AI 에이전트(Claude)가 일관된 품질로 코드를 생성할 수 있도록 **프로젝트 규칙과 컨벤션을 문서화**한 템플릿 저장소입니다.

`CLAUDE.md`가 전체 규칙의 진입점(Entry Point) 역할을 하며, `docs/` 디렉토리의 각 문서가 세부 영역을 담당합니다.

---

## 문서 구조 개요

```
CLAUDE.md              ← 프로젝트 공통 규칙 (진입점)
DOCS_GUIDE.md          ← 이 문서 (문서 역할 안내)
docs/
├── prd.md
├── git-workflow.md
├── commit-convention.md
├── project-structure.md
├── lint-config.md
├── design-guide.md
├── testing-guide.md
├── security-guide.md
├── cicd-guide.md
├── code-review-checklist.md
├── error-handling.md
├── dev-environment.md
├── state-management.md
├── performance-guide.md
├── data-modeling.md
└── maintainability-guide.md
```

---

## 각 문서의 역할

| 문서 | 설명 |
|------|------|
| [CLAUDE.md](CLAUDE.md) | **프로젝트 공통 규칙의 진입점.** 기술 스택, 핵심 규칙(코드 스타일·브랜치 전략·커밋 컨벤션·테스트·보안 등)을 요약하고, 각 상세 문서로의 링크를 제공한다. |
| [docs/prd.md](docs/prd.md) | **제품 요구사항 문서 (PRD).** 서비스의 목적, 타겟 사용자, 핵심 기능 요구사항 등 제품 수준의 명세를 정의한다. |
| [docs/git-workflow.md](docs/git-workflow.md) | **Git 워크플로우 및 브랜치 전략.** GitHub Flow 기반의 브랜치 생성·PR·머지 규칙과 브랜치 보호 규칙을 안내한다. |
| [docs/commit-convention.md](docs/commit-convention.md) | **커밋 메시지 컨벤션.** Gitmoji + Conventional Commits 형식의 커밋 메시지 작성 규칙과 예시를 정의한다. |
| [docs/project-structure.md](docs/project-structure.md) | **프로젝트 폴더 구조 가이드.** Vite + React + Supabase 기반의 디렉토리 구조와 각 폴더의 역할을 설명한다. |
| [docs/lint-config.md](docs/lint-config.md) | **Biome 린트/포매팅 설정.** `biome.json` 설정 내용과 린트·포매팅 규칙을 안내한다. |
| [docs/design-guide.md](docs/design-guide.md) | **디자인 가이드 (UI 컨벤션 + 디자인 시스템).** 컴포넌트 구조 원칙, UI 스타일링 컨벤션, 디자인 토큰 등을 정의한다. |
| [docs/testing-guide.md](docs/testing-guide.md) | **테스트 코드 가이드.** TDD 사이클, Vitest 사용법, 테스트 파일 구조 및 작성 규칙을 안내한다. |
| [docs/security-guide.md](docs/security-guide.md) | **보안 가이드.** OWASP Top 10 체크리스트, 인증/인가, RLS 정책, 입력 검증 등 보안 규칙을 정의한다. |
| [docs/cicd-guide.md](docs/cicd-guide.md) | **CI/CD 설정 가이드.** Vercel 자동 배포, GitHub Actions 워크플로우, 배포 파이프라인 설정을 안내한다. |
| [docs/code-review-checklist.md](docs/code-review-checklist.md) | **코드 리뷰 체크리스트.** PR 리뷰 시 확인해야 할 기능 요구사항, 코드 품질, 보안, 성능 항목을 정리한다. |
| [docs/error-handling.md](docs/error-handling.md) | **에러 핸들링 가이드.** Error Boundary, try-catch, Supabase `{ data, error }` 패턴 등 에러 처리 전략을 정의한다. |
| [docs/dev-environment.md](docs/dev-environment.md) | **개발 환경 셋업 가이드.** 필수 도구, 환경 변수 설정, 로컬 개발 서버 실행 방법을 안내한다. |
| [docs/state-management.md](docs/state-management.md) | **상태 관리 전략.** 서버 상태·클라이언트 상태·UI 상태의 분류와 각각의 관리 도구 및 패턴을 정의한다. |
| [docs/performance-guide.md](docs/performance-guide.md) | **성능 최적화 가이드.** React 렌더링 최적화, 코드 스플리팅, 이미지 최적화 등 성능 개선 기법을 안내한다. |
| [docs/data-modeling.md](docs/data-modeling.md) | **데이터 모델링 가이드.** Supabase/PostgreSQL 기반의 테이블 설계 원칙, 네이밍 규칙, 관계 설정을 정의한다. |
| [docs/maintainability-guide.md](docs/maintainability-guide.md) | **유지보수 가이드 (아키텍처·설계 원칙).** 코드 설계 원칙, 의존성 관리, 리팩토링 기준 등 장기 유지보수 전략을 안내한다. |

---

## 문서 간 관계

```
CLAUDE.md (진입점)
│
├─ 제품 정의
│  └─ prd.md
│
├─ 개발 환경·프로젝트 설정
│  ├─ dev-environment.md
│  ├─ project-structure.md
│  └─ lint-config.md
│
├─ 코드 작성 규칙
│  ├─ design-guide.md
│  ├─ state-management.md
│  ├─ error-handling.md
│  ├─ data-modeling.md
│  └─ maintainability-guide.md
│
├─ 품질 보증
│  ├─ testing-guide.md
│  ├─ security-guide.md
│  ├─ performance-guide.md
│  └─ code-review-checklist.md
│
└─ 워크플로우·배포
   ├─ git-workflow.md
   ├─ commit-convention.md
   └─ cicd-guide.md
```

---

## 사용 방법

1. **새 프로젝트 시작 시**: 이 저장소를 템플릿으로 복사한 뒤, `docs/prd.md`에 제품 요구사항을 작성한다.
2. **AI 에이전트 활용 시**: `CLAUDE.md`가 자동으로 로드되어 모든 규칙이 적용된다. 상세 규칙이 필요할 때 각 문서를 참조한다.
3. **팀 온보딩 시**: 이 문서(`DOCS_GUIDE.md`)를 먼저 읽고 전체 구조를 파악한 뒤, 필요한 문서를 순서대로 확인한다.
