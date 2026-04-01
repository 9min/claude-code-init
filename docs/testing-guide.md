# 테스트 코드 가이드

## TDD 사이클

**Red → Green → Refactor** 순서를 반드시 지킨다.

1. **Red**: 실패하는 테스트를 먼저 작성한다.
2. **Green**: 테스트를 통과하는 최소한의 구현 코드를 작성한다.
3. **Refactor**: 테스트가 통과하는 상태를 유지하면서 코드를 개선한다.

| 대상 | TDD 적용 |
|------|---------|
| 유틸리티 함수 | 필수 |
| 서비스 함수 | 필수 |
| 커스텀 훅 | 필수 |
| 컴포넌트 | 권장 |
| E2E 시나리오 | 핵심 플로우에 한해 적용 |

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
        functions: 70,
        branches: 70,
        statements: 70,
      },
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

| 항목 | 최소 목표 |
|------|----------|
| 전체 | 70% |
| services / utils | 90% |
| 컴포넌트 | 60% |

```bash
npx vitest run --coverage
```

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
