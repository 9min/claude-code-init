# 테스트 코드 가이드

## TDD 사이클 — 필수 준수 사항

> **이 섹션의 모든 규칙은 선택이 아닌 필수다. 예외 없이 반드시 따른다.**
> CLAUDE.md의 "테스트 (TDD)" 섹션과 동일한 규칙이다. 어느 문서를 참조하든 동일하게 적용된다.

### 절대 금지 사항

- **구현 코드를 먼저 작성하고 나중에 테스트를 추가하는 것은 금지한다.**
- 테스트 없이 기능 코드를 커밋하는 것은 금지한다.
- "나중에 테스트를 추가하겠다"는 접근은 허용하지 않는다.

### 필수 개발 순서

**Red → Green → Refactor** 순서를 반드시 지킨다. 단계를 건너뛰거나 순서를 바꾸지 않는다.

| 단계 | 행동 | 완료 기준 |
|------|------|-----------|
| **1. 테스트 파일 생성** | 구현 파일보다 테스트 파일을 먼저 생성한다 | 테스트 파일이 존재한다 |
| **2. Red** | 실패하는 테스트를 먼저 작성한다 | `vitest run` 실행 시 FAIL |
| **3. Green** | 테스트를 통과하는 최소한의 구현 코드를 작성한다 | `vitest run` 실행 시 PASS |
| **4. Refactor** | 테스트가 통과하는 상태를 유지하면서 코드를 개선한다 | `vitest run` 실행 시 여전히 PASS |
| **5. 반복** | 다음 요구사항에 대해 2~4단계를 반복한다 | 모든 요구사항에 대한 테스트가 존재하고 통과한다 |

### 위반 패턴 (이렇게 하면 안 된다)

| 위반 패턴 | 설명 |
|-----------|------|
| 구현 파일과 테스트 파일을 동시에 작성 | 테스트가 FAIL 상태를 거치지 않았으므로 TDD가 아님 |
| 구현을 완성한 뒤 테스트를 맞춤 작성 | 테스트가 구현을 검증하는 것이 아니라 구현을 추인하는 것 |
| "테스트는 나중에 추가" 주석 남기기 | 금지된 접근 방식 |
| 테스트 실행 없이 작업 완료 선언 | 자기 검증 체크리스트 미충족 |

### TDD 적용 범위

| 대상 | TDD 적용 |
|------|---------|
| 유틸리티 함수 | 필수 |
| 서비스 함수 | 필수 |
| 커스텀 훅 | 필수 |
| 컴포넌트 | 권장 |
| E2E 시나리오 | 핵심 플로우에 한해 적용 |

### 예외 허용 범위

| 예외 상황 | 조건 |
|-----------|------|
| 설정 파일 변경 | 테스트 대상이 아닌 순수 설정 파일에 한함 |
| 타입 정의만 추가 | 런타임 로직이 포함되지 않은 경우에 한함 |
| 문서 및 주석 수정 | 코드 동작에 영향이 없는 경우에 한함 |
| 프로토타이핑 | 사용자가 명시적으로 "프로토타입" 또는 "TDD 생략"을 요청한 경우에만 허용 |

## 테스트 도구

- **단위/통합**: Vitest + @testing-library/react
- **E2E**: Playwright (`@playwright/test`)

## vitest.config.ts 설정

```bash
pnpm add -D @vitest/coverage-v8 @testing-library/react @testing-library/jest-dom jsdom
```

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import { resolve } from "node:path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./tests/setup.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov"],
      exclude: [
        "node_modules/",
        "tests/",
        "e2e/",
        "**/*.d.ts",
        "**/*.config.*",
        "src/types/database.ts",
        "src/routeTree.gen.ts",
      ],
      thresholds: {
        lines: 70,
        functions: 75,
        branches: 70,
        statements: 70,
      },
      // 레이어별 개별 임계값은 아래 "커버리지 목표" 섹션 참조
    },
  },
  resolve: {
    alias: {
      "@": resolve(import.meta.dirname, "src"),
    },
  },
});
```

```ts
// tests/setup.ts
import "@testing-library/jest-dom";
```

## 파일 위치 및 네이밍

```
프로젝트-루트/
├── src/
│   ├── components/LoginForm.tsx
│   └── services/authService.ts
├── tests/
│   ├── components/LoginForm.test.tsx     # 단위/통합: *.test.ts(x)
│   └── services/authService.test.ts
└── e2e/
    └── auth.e2e.spec.ts                  # E2E: *.e2e.spec.ts
```

## 테스트 작성 규칙

### AAA 패턴

```ts
it("사용자 이름을 업데이트한다", async () => {
  // Arrange
  const user = createTestUser({ name: "기존이름" });
  // Act
  const result = await updateUserName(user.id, "새이름");
  // Assert
  expect(result.name).toBe("새이름");
});
```

### describe/it 네이밍

```ts
describe("AuthService", () => {
  describe("login", () => {
    it("올바른 자격 증명으로 토큰을 반환한다", () => {});
    it("잘못된 비밀번호로 에러를 던진다", () => {});
  });
});
```

### 테스트 격리

- 각 테스트는 독립적으로 실행 가능해야 한다.
- `beforeEach`에서 상태를 초기화한다.
- 테스트 간 상태를 공유하지 않는다.

## 커버리지 목표

### 전체 임계값 (vitest.config.ts에 설정)

| 항목 | 최소 목표 |
|------|----------|
| lines | 70% |
| functions | 75% |
| branches | 70% |
| statements | 70% |

### 레이어별 임계값

프로젝트 레이어마다 테스트 난이도와 중요도가 다르므로, 아래 기준을 적용한다.

| 레이어 | 대상 경로 | 최소 | 권장 | 근거 |
|--------|-----------|------|------|------|
| 서비스 | `src/services/**` | 85% | 90%+ | 핵심 비즈니스 로직. 순수 함수 비율이 높아 테스트가 용이하고, 미검증 시 런타임 에러 위험이 크다 |
| 유틸리티 | `src/utils/**` | 85% | 90%+ | 입출력이 명확한 순수 함수. 테스트 작성이 가장 쉽고 ROI가 높다 |
| 커스텀 훅 | `src/hooks/**` | 75% | 80% | 비즈니스 로직이 집중되는 레이어. 상태 관리와 사이드이펙트를 검증해야 한다 |
| 컴포넌트 | `src/components/**` | 50% | 60% | UI 렌더링 위주. 단위 테스트보다 E2E 테스트로 보완하는 것이 효율적이다 |
| 스토어 | `src/stores/**` | 75% | 80% | 전역 상태 변경 로직. 상태 전이가 정확한지 검증이 필요하다 |

### branches 커버리지에 대한 주의

`branches` 커버리지가 낮으면 `if/else`, `try/catch`, 삼항 연산자의 실패 경로가 미검증 상태로 남는다. 이는 프로덕션 런타임 에러의 주요 원인이므로, 특히 다음 경우에 branches 커버리지를 우선 확인한다.

- Supabase `{ data, error }` 패턴의 에러 분기
- 인증 상태별 분기 (로그인/비로그인)
- 폼 유효성 검증 분기

### 커버리지 확인 방법

```bash
# 전체 커버리지 리포트
npx vitest run --coverage

# 특정 파일/폴더만 확인
npx vitest run --coverage src/services/
```

커버리지 숫자보다 중요한 것은 **"커버되지 않은 부분이 어디인가"**다. 리포트에서 미검증 라인(빨간 줄)을 직접 확인하고, 해당 경로가 프로덕션에서 실행될 가능성이 있는지 판단한다.

## Supabase 모킹

### 클라이언트 모킹

```ts
// tests/helpers/supabaseMock.ts
import { vi } from "vitest";
import { supabase } from "@/lib/supabase";

export function mockSupabaseSelect<T>(data: T[], error: null | { message: string } = null) {
  const mockChain = {
    select: vi.fn().mockReturnThis(),
    eq: vi.fn().mockReturnThis(),
    order: vi.fn().mockReturnThis(),
    range: vi.fn().mockReturnThis(),
    single: vi.fn().mockResolvedValue({ data: data[0] ?? null, error }),
    then: vi.fn((resolve) => resolve({ data, error })),
  };
  Object.assign(mockChain.select, mockChain);
  vi.mocked(supabase.from).mockReturnValue(mockChain as never);
  return mockChain;
}

// 인증 상태 모킹
export function mockAuthenticated(user = { id: "user-1", email: "test@example.com" }) {
  vi.mocked(supabase.auth.getSession).mockResolvedValue({
    data: { session: { user, access_token: "token", refresh_token: "refresh" } as never },
    error: null,
  });
}

export function mockUnauthenticated() {
  vi.mocked(supabase.auth.getSession).mockResolvedValue({
    data: { session: null },
    error: null,
  });
}
```

전체 mock 설정:

```ts
vi.mock("@/lib/supabase", () => ({
  supabase: {
    from: vi.fn(() => ({
      select: vi.fn().mockReturnThis(),
      insert: vi.fn().mockReturnThis(),
      update: vi.fn().mockReturnThis(),
      delete: vi.fn().mockReturnThis(),
      eq: vi.fn().mockReturnThis(),
      order: vi.fn().mockReturnThis(),
      single: vi.fn(),
    })),
    auth: {
      getSession: vi.fn(),
      signInWithPassword: vi.fn(),
      signOut: vi.fn(),
      onAuthStateChange: vi.fn(() => ({
        data: { subscription: { unsubscribe: vi.fn() } },
      })),
    },
  },
}));
```

### 테스트 데이터 팩토리

```ts
// tests/factories/todoFactory.ts
import type { Todo } from "@/types/todo";

export function createTestTodo(overrides: Partial<Todo> = {}): Todo {
  return {
    id: crypto.randomUUID(),
    user_id: "test-user-id",
    title: "테스트 할 일",
    is_completed: false,
    created_at: new Date().toISOString(),
    updated_at: new Date().toISOString(),
    ...overrides,
  };
}
```

### RLS 정책 테스트

RLS 테스트는 실제 Supabase 로컬 인스턴스(`npx supabase start`)가 실행 중인 상태에서 수행한다.

```bash
pnpm add -D jose
```

```ts
// tests/rls/todos.rls.test.ts
import { createClient } from "@supabase/supabase-js";
import { SignJWT } from "jose";

const SUPABASE_URL = process.env.SUPABASE_URL ?? "http://localhost:54321";
const SERVICE_ROLE_KEY = process.env.SUPABASE_SERVICE_ROLE_KEY ?? "";

// 로컬 JWT 시크릿: supabase/config.toml의 [auth] jwt_secret 값
// 또는 `npx supabase status` 출력에서 확인
const JWT_SECRET = process.env.SUPABASE_JWT_SECRET
  ?? "super-secret-jwt-token-with-at-least-32-characters-long";

async function generateTestJwt(userId: string): Promise<string> {
  const secret = new TextEncoder().encode(JWT_SECRET);
  return new SignJWT({ sub: userId, role: "authenticated" })
    .setProtectedHeader({ alg: "HS256" })
    .setIssuedAt()
    .setExpirationTime("1h")
    .sign(secret);
}

function createAuthenticatedClient(token: string) {
  return createClient(SUPABASE_URL, SERVICE_ROLE_KEY, {
    global: { headers: { Authorization: `Bearer ${token}` } },
  });
}

describe("todos RLS 정책", () => {
  it("본인의 할 일만 조회할 수 있다", async () => {
    const token = await generateTestJwt("user-1");
    const client = createAuthenticatedClient(token);
    const { data } = await client.from("todos").select("*");
    for (const todo of data ?? []) {
      expect(todo.user_id).toBe("user-1");
    }
  });
});
```

## TanStack Router 컴포넌트 테스트

라우터 context(세션, 인증 상태 등)에 의존하는 컴포넌트를 테스트할 때는 `createMemoryHistory`와 `createRouter`로 테스트용 라우터를 설정한다.

```ts
pnpm add -D @tanstack/react-router
```

```tsx
// tests/helpers/routerTestUtils.tsx
import { createMemoryHistory, createRouter, RouterProvider } from "@tanstack/react-router";
import { render } from "@testing-library/react";
import { routeTree } from "@/routeTree.gen";

interface RenderWithRouterOptions {
  initialPath?: string;
  context?: { session: unknown };
}

export function renderWithRouter(
  initialPath = "/",
  context: RenderWithRouterOptions["context"] = { session: null }
) {
  const history = createMemoryHistory({ initialEntries: [initialPath] });
  const router = createRouter({
    routeTree,
    history,
    context,
  });

  return render(<RouterProvider router={router} />);
}
```

```tsx
// 사용 예시: tests/routes/dashboard.test.tsx
import { renderWithRouter } from "@/tests/helpers/routerTestUtils";
import { screen } from "@testing-library/react";

describe("Dashboard 라우트", () => {
  it("인증된 사용자에게 대시보드를 표시한다", async () => {
    renderWithRouter("/dashboard", { session: { user: { id: "user-1" } } });
    expect(await screen.findByText("대시보드")).toBeInTheDocument();
  });

  it("미인증 사용자를 로그인 페이지로 리다이렉트한다", async () => {
    renderWithRouter("/dashboard", { session: null });
    expect(await screen.findByText("로그인")).toBeInTheDocument();
  });
});
```

## E2E 테스트

### 설치

```bash
pnpm add -D @playwright/test
npx playwright install chromium
```

### playwright.config.ts

```ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  use: { baseURL: "http://localhost:4173" },
  webServer: {
    command: "pnpm run build && npx vite preview",
    port: 4173,
    reuseExistingServer: !process.env.CI,
  },
  projects: [{ name: "chromium", use: { ...devices["Desktop Chrome"] } }],
});
```

### 실행 명령어

```bash
npx playwright test          # 헤드리스
npx playwright test --ui     # UI 모드 (디버깅)
```

## 관련 문서

- [프로젝트 구조](project-structure.md)
- [CI/CD 가이드](cicd-guide.md)
- [보안 가이드](security-guide.md)
