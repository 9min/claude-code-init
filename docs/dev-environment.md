# 개발 환경 셋업 가이드

## 사전 요구사항

| 도구 | 최소 버전 | 비고 |
|------|----------|------|
| Node.js | 22+ (.nvmrc 참조) | `import.meta.dirname` 사용을 위해 22 이상 필요 |
| pnpm | >= 9 | `corepack enable`으로 활성화 |
| Docker Desktop | 최신 | Supabase 로컬 실행용 |
| Supabase CLI | 최신 | `npx supabase` 또는 `pnpm add -D supabase`로 설치 |

> Node.js 버전은 프로젝트 루트의 `.nvmrc`가 단일 진실 소스다. `nvm use` 또는 `fnm use`로 자동 전환된다. `.nvmrc` 기본값은 `22`로 설정한다.

## Supabase 로컬 환경

### 초기화 및 실행

```bash
# 최초 1회: Supabase 프로젝트 초기화
npx supabase init

# 로컬 Supabase 서비스 시작 (Docker Desktop이 실행 중이어야 함)
npx supabase start
```

> `pnpm add -D supabase`로 로컬 설치한 경우 `npx supabase` 대신 `pnpm supabase` 또는 `./node_modules/.bin/supabase`를 사용한다. 어느 쪽이든 동일하게 동작하므로, 팀 내에서 한 가지로 통일한다.

시작 후 로컬 서비스 URL:

| 서비스 | URL | 설명 |
|--------|-----|------|
| Studio | http://localhost:54323 | 데이터베이스 관리 UI |
| API | http://localhost:54321 | REST/GraphQL API |
| DB | localhost:54322 | PostgreSQL 직접 접속 |
| Inbucket | http://localhost:54324 | 로컬 이메일 테스트 |

### 중지/리셋

```bash
# 로컬 서비스 중지
npx supabase stop

# 데이터베이스 리셋 (마이그레이션 재적용 + 시드 데이터)
npx supabase db reset
```

## 환경변수 설정

### 파일 역할

| 파일 | 역할 | Git 추적 |
|------|------|----------|
| `.env.example` | 환경변수 키 목록 (값 없음). 팀원 온보딩용 | O |
| `.env.local` | 로컬 개발용 실제 값. 각 개발자가 생성 | X |
| `.env` | 빌드/배포 시 사용. CI/CD에서 주입 | X |

### .env.local 구성 예시

```
VITE_SUPABASE_URL=http://localhost:54321
VITE_SUPABASE_ANON_KEY=eyJ...로컬_anon_key...
```

`npx supabase start` 실행 후 출력되는 `anon key`를 복사하여 사용한다.

### 환경별 관리

| 환경 | SUPABASE_URL | ANON_KEY | 관리 방식 |
|------|-------------|----------|----------|
| 로컬 | localhost:54321 | 로컬 키 | `.env.local` |
| Preview | Supabase 프로젝트 URL | 프로젝트 키 | Vercel 환경변수 |
| Production | Supabase 프로젝트 URL | 프로젝트 키 | Vercel 환경변수 |

## IDE 설정

### VSCode 필수 확장

| 확장 | ID | 용도 |
|------|----|------|
| Biome | `biomejs.biome` | 린트/포매팅 |
| Tailwind CSS IntelliSense | `bradlc.vscode-tailwindcss` | Tailwind 자동완성 |
| ES7+ React Snippets | `dsznajder.es7-react-js-snippets` | React 스니펫 |

### .vscode/settings.json 예제

```json
{
  "editor.defaultFormatter": "biomejs.biome",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "quickfix.biome": "explicit",
    "source.organizeImports.biome": "explicit"
  },
  "tailwindCSS.experimental.classRegex": [
    ["cn\\(([^)]*)\\)", "'([^']*)'"]
  ]
}
```

### .vscode/extensions.json 예제

```json
{
  "recommendations": [
    "biomejs.biome",
    "bradlc.vscode-tailwindcss",
    "dsznajder.es7-react-js-snippets"
  ]
}
```

### 디버깅 설정 (.vscode/launch.json)

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Vite 디버깅",
      "url": "http://localhost:5173",
      "webRoot": "${workspaceFolder}/src"
    }
  ]
}
```

## 로컬 개발 실행

### 실행 순서

```bash
# 1. Supabase 로컬 서비스 시작
npx supabase start

# 2. .env.local 파일 확인 (없으면 .env.example 복사 후 값 입력)
cp .env.example .env.local

# 3. 개발 서버 실행
pnpm run dev
```

### package.json scripts 예제

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "biome check --write .",
    "test": "vitest",
    "test:coverage": "vitest run --coverage",
    "db:start": "npx supabase start",
    "db:stop": "npx supabase stop",
    "db:reset": "npx supabase db reset",
    "db:types": "npx supabase gen types typescript --local > src/types/database.ts",
    "prepare": "husky"
  }
}
```

## Supabase 타입 생성

### 명령어

```bash
npx supabase gen types typescript --local > src/types/database.ts
```

### 타입 생성 시점

- 마이그레이션 파일 추가/변경 후
- `npx supabase db reset` 실행 후
- 테이블 구조가 변경된 경우

### package.json 스크립트 등록

```json
{
  "scripts": {
    "db:types": "npx supabase gen types typescript --local > src/types/database.ts"
  }
}
```

## 트러블슈팅

| 증상 | 원인 | 해결 |
|------|------|------|
| `supabase start` 실패 | Docker Desktop 미실행 | Docker Desktop 실행 후 재시도 |
| 포트 충돌 (54321 등) | 다른 프로세스가 포트 사용 중 | `supabase stop` 후 재시작, 또는 충돌 프로세스 종료 |
| 타입 생성 실패 | 로컬 Supabase 미실행 | `supabase start` 먼저 실행 |
| `.env.local` 미인식 | Vite 환경변수 접두사 누락 | `VITE_` 접두사 확인 |
| 마이그레이션 충돌 | 원격과 로컬 마이그레이션 불일치 | `supabase db reset`으로 로컬 초기화 |
| `supabase start` 느림 | Docker 이미지 최초 다운로드 | 최초 실행 시 정상. 이후 빠름 |

## Git Hooks (husky)

커밋과 푸시 시점에 자동으로 품질 검사를 실행하여, 문제가 있는 코드가 리모트에 올라가는 것을 방지한다.

### 설치

```bash
pnpm add -D husky
pnpm exec husky init
```

`husky init`이 `.husky/` 디렉토리와 `pre-commit` 파일을 자동 생성한다.

### pre-commit hook — 린트 + 타입 검사

커밋할 때마다 린트와 타입 검사를 실행한다. **테스트는 여기서 실행하지 않는다** — 매 커밋마다 테스트를 돌리면 느려서 `--no-verify` 우회가 남용되기 때문이다.

```bash
# .husky/pre-commit
pnpm biome check --write . && pnpm tsc --noEmit
```

### pre-push hook — 전체 테스트 실행

푸시 전 1회만 전체 테스트를 실행한다. 테스트가 실패하면 푸시가 차단된다.

```bash
# .husky/pre-push
pnpm vitest run
```

### hook 구성 요약

| hook | 실행 시점 | 실행 내용 | 목적 |
|------|-----------|-----------|------|
| `pre-commit` | `git commit` | `biome check` + `tsc --noEmit` | 린트/타입 오류가 있는 커밋 방지 |
| `pre-push` | `git push` | `vitest run` | 테스트 실패 코드가 리모트에 올라가는 것 방지 |

### 주의사항

- `--no-verify` 플래그로 hook을 우회하지 않는다. 우회가 필요한 상황이면 근본 원인을 먼저 해결한다.
- CI가 최종 안전망이므로 hook이 모든 것을 잡을 필요는 없다. hook은 "빠른 피드백", CI는 "완전한 검증" 역할을 분담한다.
- E2E 테스트(`playwright`)는 hook에 포함하지 않는다. 실행 시간이 길고 로컬 환경 의존성이 크기 때문에 CI에서만 실행한다.

## AI 토큰 최적화 (프로젝트 규모 확장 시)

프로젝트 문서와 코드가 많아지면 AI 에이전트가 파일을 탐색하는 데 토큰을 과도하게 소비할 수 있다. 아래는 프로젝트 규모가 커졌을 때 검토할 수 있는 토큰 절약 방법이다.

### `.claudeignore` 활용

빌드 아티팩트, 로그, 잠금 파일 등 AI가 읽을 필요 없는 파일을 제외한다.

```
dist/
node_modules/
coverage/
*.log
package-lock.json
```

### 문서 구조화

문서 간 중복 내용을 줄이고, 참조 링크로 대체한다. 같은 규칙이 여러 문서에 기술되면 한쪽만 업데이트되어 내용이 어긋날 수 있다.

### 문서 검색 인덱싱

프로젝트 문서가 30개 이상이거나 전체 분량이 3,000줄을 초과하면, Markdown 검색 엔진(MCP 서버 기반)을 도입하여 AI가 파일을 통째로 읽는 대신 관련 부분만 조회하도록 할 수 있다. 도입 시 팀에서 사용할 도구를 평가하고 MCP 서버 설정을 문서화한다.

## 관련 문서

- [프로젝트 구조](project-structure.md)
- [린트 설정](lint-config.md)
- [CI/CD 가이드](cicd-guide.md)
