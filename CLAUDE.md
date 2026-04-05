# CLAUDE.md - 프로젝트 공통 규칙

## 기술 스택

- **프론트엔드**: Vite + React + TypeScript
- **라우터**: TanStack Router (파일 기반 라우팅)
- **백엔드(BaaS)**: Supabase (PostgreSQL, Auth, Storage, Edge Functions)
- **린트/포매팅**: Biome
- **테스트**: Vitest + Playwright (E2E)
- **배포**: Vercel (프론트엔드), Supabase (백엔드)
- **버전 관리**: Git (GitHub Flow)

## 핵심 규칙

### 언어

- 코드 내 주석, 커밋 메시지, PR 설명 등 모든 문서는 **한국어**로 작성한다.

### 코드 스타일

- Biome 설정을 따른다. (`biome.json` 참조)
- TypeScript strict 모드를 사용한다.
- `any` 타입 사용을 금지한다. 불가피한 경우 `unknown`을 사용하고 타입 가드를 적용한다.
- 함수형 컴포넌트와 훅을 사용한다. 클래스 컴포넌트는 사용하지 않는다.
- 네이밍 컨벤션:
  - 컴포넌트: `PascalCase`
  - 함수/변수: `camelCase`
  - 상수: `UPPER_SNAKE_CASE`
  - 타입/인터페이스: `PascalCase`
  - 파일명: 컴포넌트는 `PascalCase.tsx`, 그 외는 `camelCase.ts`

### 커밋 및 푸시

- **작업 완료 후 커밋/푸시는 직접 요청받았을 때만 진행한다.** 명시적 요청 없이 자동으로 커밋하거나 푸시하지 않는다.

### 브랜치 전략

- GitHub Flow 기반: `main` → `feature/*`, `fix/*`, `hotfix/*`
- **`main` 브랜치에 직접 커밋하거나 푸시하는 것은 절대 금지한다.** 어떤 상황에서도 예외 없이 반드시 feature 브랜치를 생성한 후 PR을 통해 머지한다.
- 작업 시작 전 현재 브랜치를 반드시 확인한다. `main` 브랜치에 있다면 즉시 작업 브랜치로 전환한다.
- `git push --force` 및 프로덕션 브랜치(`main`)로의 직접 푸시는 사용하지 않는다.

### 커밋 컨벤션

- Gitmoji + Conventional Commits 형식
- 예: `✨ feat: 사용자 로그인 기능 추가`
- 예: `🐛 fix: 토큰 만료 시 리다이렉트 오류 수정`

### 테스트 (TDD) — 필수 준수 사항

> **이 섹션의 모든 규칙은 선택이 아닌 필수다. 예외 없이 반드시 따른다.**

#### 절대 금지 사항

- **구현 코드를 먼저 작성하고 나중에 테스트를 추가하는 것은 금지한다.** 이 순서를 위반하면 해당 작업 전체를 TDD 위반으로 간주한다.
- 테스트 없이 기능 코드를 커밋하는 것은 금지한다.
- "나중에 테스트를 추가하겠다"는 접근은 허용하지 않는다.

#### 필수 개발 순서 (이 순서를 반드시 지킨다)

새로운 기능 또는 버그 수정 시, 아래 단계를 **순서대로** 실행한다. 단계를 건너뛰거나 순서를 바꾸지 않는다.

| 단계 | 행동 | 완료 기준 |
|------|------|-----------|
| **1. 테스트 파일 생성** | 구현 파일보다 테스트 파일(`*.test.ts` / `*.test.tsx`)을 먼저 생성한다 | 테스트 파일이 존재한다 |
| **2. Red — 실패하는 테스트 작성** | 요구사항을 검증하는 테스트 케이스를 작성한다. 아직 구현이 없으므로 테스트는 반드시 실패해야 한다 | `vitest run` 실행 시 테스트가 FAIL한다 |
| **3. Green — 최소 구현** | 테스트를 통과시키기 위한 **최소한의** 구현 코드만 작성한다. 불필요한 코드를 미리 작성하지 않는다 | `vitest run` 실행 시 테스트가 PASS한다 |
| **4. Refactor — 코드 개선** | 테스트가 통과하는 상태를 유지하면서 코드 품질을 개선한다 | `vitest run` 실행 시 테스트가 여전히 PASS한다 |
| **5. 반복** | 다음 요구사항에 대해 2~4단계를 반복한다 | 모든 요구사항에 대한 테스트가 존재하고 통과한다 |

#### 자기 검증 체크리스트

작업 완료 시 아래 항목을 모두 확인한다. 하나라도 충족하지 못하면 작업을 완료한 것으로 간주하지 않는다.

- [ ] 구현 코드보다 테스트 코드를 먼저 작성했는가?
- [ ] 테스트가 한 번 이상 FAIL 상태를 거친 후 PASS로 전환되었는가?
- [ ] 모든 공개 함수/컴포넌트에 대응하는 테스트가 존재하는가?
- [ ] `vitest run` 실행 시 모든 테스트가 통과하는가?

#### 예외 허용 범위

아래 경우에 **한하여** TDD 순서를 완화할 수 있다. 그 외에는 예외를 인정하지 않는다.

| 예외 상황 | 허용 범위 | 조건 |
|-----------|-----------|------|
| 설정 파일 변경 | `vite.config.ts`, `biome.json`, `tsconfig.json` 등 | 테스트 대상이 아닌 순수 설정 파일에 한함 |
| 타입 정의만 추가 | `types/` 디렉토리의 인터페이스/타입 선언 | 런타임 로직이 포함되지 않은 경우에 한함 |
| 문서 및 주석 수정 | `.md` 파일, JSDoc 주석 | 코드 동작에 영향이 없는 경우에 한함 |
| 프로토타이핑 | 사용자가 명시적으로 "프로토타입" 또는 "TDD 생략"을 요청한 경우 | 사용자의 명시적 요청이 있을 때만 허용 |

#### 도구 및 파일 규칙

- Vitest를 사용한다.
- 테스트 파일명: `*.test.ts` 또는 `*.test.tsx`
- 상세 가이드: [testing-guide.md](docs/testing-guide.md)를 참조한다.

### Supabase 사용 규칙

- Supabase 클라이언트는 `lib/supabase.ts`에서 단일 인스턴스로 생성하여 사용한다.
- SPA에서는 `createClient`를 사용한다. SSR 도입 시 클라이언트 구분은 [project-structure.md](docs/project-structure.md)를 참조한다.
- 데이터베이스 접근은 반드시 RLS(Row Level Security) 정책을 통해 보호한다.
- 직접 SQL보다 Supabase Client 메서드(`.from().select()` 등)를 우선 사용한다.
- 복잡한 비즈니스 로직은 Edge Functions 또는 Database Functions(RPC)로 처리한다.

### 보안

- **기능 개발 시 보안 검토를 필수로 수행한다.** 구현 완료 후 보안 체크리스트를 점검한다.
- 환경변수로 시크릿을 관리한다. 코드에 하드코딩 금지.
- `SUPABASE_URL`과 `SUPABASE_ANON_KEY`는 클라이언트에 노출 가능하지만, `SERVICE_ROLE_KEY`는 서버 측에서만 사용한다.
- 사용자 입력은 반드시 검증하고 새니타이즈한다.
- OWASP Top 10을 준수한다.
- 모든 테이블에 RLS 정책을 활성화한다.
- 보안 관련 상세 체크리스트는 [security-guide.md](docs/security-guide.md)를 참조한다.

### 에러 처리

- 프론트엔드: Error Boundary + try-catch 패턴
- Supabase: `{ data, error }` 패턴으로 에러를 처리한다. `error`를 항상 확인한다.

## 상세 문서 참조

@docs/git-workflow.md
@docs/commit-convention.md
@docs/maintainability-guide.md

각 항목에 대한 상세 내용은 아래 문서를 참조한다.

| 문서 | 설명 |
|------|------|
| [docs/prd.md](docs/prd.md) | 제품 요구사항 문서 (PRD) |
| [docs/git-workflow.md](docs/git-workflow.md) | Git 워크플로우 및 브랜치 전략 |
| [docs/commit-convention.md](docs/commit-convention.md) | 커밋 메시지 컨벤션 |
| [docs/project-structure.md](docs/project-structure.md) | 프로젝트 폴더 구조 가이드 |
| [docs/lint-config.md](docs/lint-config.md) | Biome 린트/포매팅 설정 |
| [docs/design-guide.md](docs/design-guide.md) | 디자인 가이드 (UI 컨벤션 + 디자인 시스템) |
| [docs/testing-guide.md](docs/testing-guide.md) | 테스트 코드 가이드 |
| [docs/security-guide.md](docs/security-guide.md) | 보안 가이드 |
| [docs/cicd-guide.md](docs/cicd-guide.md) | CI/CD 설정 가이드 |
| [docs/hotfix-guide.md](docs/hotfix-guide.md) | 핫픽스 대응 가이드 |
| [docs/code-review-checklist.md](docs/code-review-checklist.md) | 코드 리뷰 체크리스트 |
| [docs/error-handling.md](docs/error-handling.md) | 에러 핸들링 가이드 |
| [docs/dev-environment.md](docs/dev-environment.md) | 개발 환경 셋업 가이드 |
| [docs/state-management.md](docs/state-management.md) | 상태 관리 전략 |
| [docs/performance-guide.md](docs/performance-guide.md) | 성능 최적화 가이드 |
| [docs/data-modeling.md](docs/data-modeling.md) | 데이터 모델링 가이드 |
| [docs/maintainability-guide.md](docs/maintainability-guide.md) | 유지보수 가이드 (아키텍처·설계 원칙) |

## 신규 개발자 학습 경로

프로젝트에 처음 합류한 개발자는 다음 순서로 문서를 읽는다.

1. **환경 구축**: [dev-environment.md](docs/dev-environment.md)
2. **프로젝트 이해**: [docs/prd.md](docs/prd.md) → [project-structure.md](docs/project-structure.md)
3. **코드 작성 규칙**: [lint-config.md](docs/lint-config.md) → [design-guide.md](docs/design-guide.md) → [state-management.md](docs/state-management.md)
4. **워크플로우**: [git-workflow.md](docs/git-workflow.md) → [commit-convention.md](docs/commit-convention.md) → [testing-guide.md](docs/testing-guide.md)
5. **심화**: [security-guide.md](docs/security-guide.md) → [data-modeling.md](docs/data-modeling.md) → [error-handling.md](docs/error-handling.md)

## 문서 갱신 규칙

- **코드 변경 시**: 관련 문서의 코드 예시나 설정이 달라졌다면 함께 업데이트한다.
- **기술 스택 변경 시**: 이 파일의 기술 스택 섹션과 관련 가이드 문서를 모두 갱신한다.
- **새 컨벤션 합의 시**: 해당 가이드 문서에 반영하고, 필요하면 핵심 규칙도 업데이트한다.
