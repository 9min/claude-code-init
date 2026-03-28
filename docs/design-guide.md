# 디자인 가이드

## UI 코드 컨벤션

### 컴포넌트 구조 원칙

- **단일 책임 원칙**: 하나의 컴포넌트는 하나의 역할만 담당한다.
- **합성 패턴**: 작은 컴포넌트를 조합하여 복잡한 UI를 구성한다.
- **Presentational/Container 분리**: UI 렌더링과 비즈니스 로직을 분리한다.

### Tailwind CSS 사용 규칙

#### 클래스 순서

다음 순서로 Tailwind 클래스를 작성한다:

1. 레이아웃 (`flex`, `grid`, `block`)
2. 포지셔닝 (`relative`, `absolute`, `fixed`)
3. 박스 모델 (`w-`, `h-`, `p-`, `m-`)
4. 타이포그래피 (`text-`, `font-`, `leading-`)
5. 시각적 (`bg-`, `border-`, `rounded-`, `shadow-`)
6. 상태 (`hover:`, `focus:`, `active:`)
7. 반응형 (`sm:`, `md:`, `lg:`)
8. 애니메이션 (`transition-`, `animate-`)

```tsx
// 좋은 예
<button className="flex items-center gap-2 px-4 py-2 text-sm font-medium bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">

// 나쁜 예 (순서 불규칙)
<button className="hover:bg-blue-700 text-sm bg-blue-600 flex rounded-lg px-4 items-center">
```

#### 반복되는 스타일 추출

동일한 스타일 조합이 3회 이상 반복되면 컴포넌트로 추출한다.

```tsx
// 좋은 예: 재사용 컴포넌트로 추출
function Badge({ children, variant }: BadgeProps) {
  const variants = {
    success: "bg-green-100 text-green-800",
    error: "bg-red-100 text-red-800",
    warning: "bg-yellow-100 text-yellow-800",
  };
  return (
    <span className={`inline-flex items-center px-2 py-1 text-xs font-medium rounded-full ${variants[variant]}`}>
      {children}
    </span>
  );
}
```

### 조건부 스타일링

`clsx` 또는 `cn` 유틸리티를 사용한다.

```tsx
import { cn } from "@/utils/cn";

<button
  className={cn(
    "px-4 py-2 rounded-lg font-medium",
    isActive && "bg-blue-600 text-white",
    isDisabled && "opacity-50 cursor-not-allowed"
  )}
/>
```

## 디자인 시스템

### 색상 팔레트

프로젝트별로 커스텀 색상을 정의하되, 기본 구조는 다음을 따른다:

```
색상 체계:
├── primary     # 주요 브랜드 색상 (50~950)
├── secondary   # 보조 색상
├── neutral     # 중립 색상 (gray 계열)
├── success     # 성공 (green 계열)
├── warning     # 경고 (yellow/amber 계열)
├── error       # 에러 (red 계열)
└── info        # 정보 (blue 계열)
```

각 색상은 50, 100, 200, ..., 900, 950 단계로 정의한다.

### 타이포그래피

```
텍스트 크기 체계:
├── xs      # 12px  - 캡션, 보조 텍스트
├── sm      # 14px  - 작은 본문
├── base    # 16px  - 기본 본문
├── lg      # 18px  - 큰 본문
├── xl      # 20px  - 소제목
├── 2xl     # 24px  - 제목
├── 3xl     # 30px  - 큰 제목
└── 4xl     # 36px  - 페이지 제목
```

- 본문 기본 폰트: 시스템 폰트 스택 사용
- 폰트 굵기: `normal (400)`, `medium (500)`, `semibold (600)`, `bold (700)`

### 스페이싱 스케일

Tailwind 기본 스페이싱 스케일(4px 단위)을 따른다.

| 단위 | 크기 | 용도 |
|------|------|------|
| `1` | 4px | 아이콘 간격, 미세 조정 |
| `2` | 8px | 인라인 요소 간격 |
| `3` | 12px | 작은 내부 여백 |
| `4` | 16px | 기본 내부 여백, 요소 간격 |
| `6` | 24px | 카드 내부 여백 |
| `8` | 32px | 섹션 간격 |
| `12` | 48px | 큰 섹션 간격 |
| `16` | 64px | 페이지 여백 |

### 반응형 브레이크포인트

Tailwind 기본 브레이크포인트를 따른다.

| 접두사 | 최소 너비 | 대상 디바이스 |
|--------|----------|-------------|
| (없음) | 0px | 모바일 (기본) |
| `sm` | 640px | 큰 모바일 / 소형 태블릿 |
| `md` | 768px | 태블릿 |
| `lg` | 1024px | 소형 데스크탑 |
| `xl` | 1280px | 데스크탑 |
| `2xl` | 1536px | 대형 데스크탑 |

**모바일 퍼스트** 원칙을 따른다: 기본 스타일은 모바일, 큰 화면은 브레이크포인트로 확장한다.

```tsx
// 좋은 예: 모바일 퍼스트
<div className="flex flex-col gap-4 md:flex-row md:gap-8">

// 나쁜 예: 데스크탑 퍼스트
<div className="flex flex-row gap-8 max-md:flex-col max-md:gap-4">
```

## 다크모드 지원

### Tailwind CSS 다크모드 설정

`tailwind.config.js`에서 `darkMode: "class"`를 설정한다.

```js
// tailwind.config.js
export default {
  darkMode: "class",
  // ...
};
```

### dark: 클래스 사용법

```tsx
<div className="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100">
  <h1 className="text-gray-800 dark:text-gray-200">제목</h1>
  <p className="text-gray-600 dark:text-gray-400">본문</p>
</div>
```

### 테마 토글 훅

```tsx
import { useEffect } from "react";

export function useTheme() {
  const theme = useThemeStore((state) => state.theme);
  const toggleTheme = useThemeStore((state) => state.toggleTheme);

  useEffect(() => {
    document.documentElement.classList.toggle("dark", theme === "dark");
  }, [theme]);

  return { theme, toggleTheme };
}
```

### 라이트/다크 색상 매핑

| 용도 | 라이트 | 다크 |
|------|--------|------|
| 배경 (기본) | `bg-white` | `dark:bg-gray-900` |
| 배경 (카드) | `bg-gray-50` | `dark:bg-gray-800` |
| 텍스트 (기본) | `text-gray-900` | `dark:text-gray-100` |
| 텍스트 (보조) | `text-gray-600` | `dark:text-gray-400` |
| 테두리 | `border-gray-200` | `dark:border-gray-700` |
| 입력 필드 배경 | `bg-white` | `dark:bg-gray-800` |

## 상태별 UI 패턴

### 로딩 상태

| 패턴 | 사용 시점 | 예시 |
|------|----------|------|
| 스켈레톤 | 목록, 카드 등 콘텐츠 영역 | 최초 데이터 로딩 |
| 스피너 | 버튼, 전체 페이지 전환 | 폼 제출, 페이지 이동 |
| 프로그레스 바 | 파일 업로드 등 진행률 표시 | 이미지 업로드 |

```tsx
// 스켈레톤 예제
function TodoListSkeleton() {
  return (
    <div className="space-y-3">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-16 bg-gray-200 dark:bg-gray-700 rounded-lg animate-pulse" />
      ))}
    </div>
  );
}
```

### 빈 상태

데이터가 없을 때 안내 메시지와 액션을 표시한다.

```tsx
interface EmptyStateProps {
  icon: React.ReactNode;
  title: string;
  description: string;
  action?: {
    label: string;
    onClick: () => void;
  };
}

function EmptyState({ icon, title, description, action }: EmptyStateProps) {
  return (
    <div className="flex flex-col items-center justify-center py-12 text-center">
      <div className="mb-4 text-gray-400 dark:text-gray-500">{icon}</div>
      <h3 className="text-lg font-medium text-gray-900 dark:text-gray-100">{title}</h3>
      <p className="mt-1 text-sm text-gray-500 dark:text-gray-400">{description}</p>
      {action && (
        <button
          type="button"
          className="mt-4 px-4 py-2 bg-primary-600 text-white rounded-lg hover:bg-primary-700"
          onClick={action.onClick}
        >
          {action.label}
        </button>
      )}
    </div>
  );
}
```

### 에러 상태

에러 발생 시 사용자에게 상황을 알리고 재시도 옵션을 제공한다.

```tsx
interface ErrorStateProps {
  message: string;
  onRetry?: () => void;
}

function ErrorState({ message, onRetry }: ErrorStateProps) {
  return (
    <div className="flex flex-col items-center justify-center py-12 text-center">
      <div className="mb-4 text-red-500">
        <AlertCircleIcon className="w-12 h-12" />
      </div>
      <h3 className="text-lg font-medium text-gray-900 dark:text-gray-100">
        문제가 발생했습니다
      </h3>
      <p className="mt-1 text-sm text-gray-500 dark:text-gray-400">{message}</p>
      {onRetry && (
        <button
          type="button"
          className="mt-4 px-4 py-2 bg-primary-600 text-white rounded-lg hover:bg-primary-700"
          onClick={onRetry}
        >
          다시 시도
        </button>
      )}
    </div>
  );
}
```

### 토스트/알림

| 유형 | 색상 | 사용 시점 | 자동 닫힘 |
|------|------|----------|----------|
| 성공 | green | 작업 완료 (저장, 삭제) | 3초 |
| 에러 | red | 작업 실패 | 사용자 닫기 |
| 경고 | yellow | 주의 필요 (만료 임박 등) | 5초 |
| 정보 | blue | 안내 메시지 | 3초 |

```ts
interface Toast {
  id: string;
  type: "success" | "error" | "warning" | "info";
  message: string;
  duration?: number; // ms, undefined면 수동 닫기
}
```

## 공통 UI 컴포넌트 가이드

프로젝트 전반에서 반복 사용되는 기본 컴포넌트의 구현 패턴을 정의한다. 아래 예시를 기반으로 프로젝트에 맞게 확장한다.

### Button

```tsx
import { cn } from "@/utils/cn";

type ButtonVariant = "primary" | "secondary" | "ghost" | "danger";
type ButtonSize = "sm" | "md" | "lg";

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: ButtonVariant;
  size?: ButtonSize;
  isLoading?: boolean;
}

const variantStyles: Record<ButtonVariant, string> = {
  primary: "bg-primary-600 text-white hover:bg-primary-700 focus-visible:ring-primary-500",
  secondary: "bg-gray-100 text-gray-900 hover:bg-gray-200 dark:bg-gray-700 dark:text-gray-100 dark:hover:bg-gray-600",
  ghost: "text-gray-700 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-800",
  danger: "bg-red-600 text-white hover:bg-red-700 focus-visible:ring-red-500",
};

const sizeStyles: Record<ButtonSize, string> = {
  sm: "px-3 py-1.5 text-sm",
  md: "px-4 py-2 text-sm",
  lg: "px-6 py-3 text-base",
};

function Button({
  variant = "primary",
  size = "md",
  isLoading,
  disabled,
  className,
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      className={cn(
        "inline-flex items-center justify-center font-medium rounded-lg transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2",
        variantStyles[variant],
        sizeStyles[size],
        (disabled || isLoading) && "opacity-50 cursor-not-allowed",
        className
      )}
      disabled={disabled || isLoading}
      {...props}
    >
      {isLoading && <SpinnerIcon className="mr-2 h-4 w-4 animate-spin" />}
      {children}
    </button>
  );
}
```

### Input

```tsx
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
  hint?: string;
}

function Input({ label, error, hint, id, ...props }: InputProps) {
  const inputId = id || label.replace(/\s+/g, "-").toLowerCase();
  const errorId = `${inputId}-error`;
  const hintId = `${inputId}-hint`;

  return (
    <div>
      <label htmlFor={inputId} className="block text-sm font-medium text-gray-700 dark:text-gray-300">
        {label}
        {props.required && <span className="text-red-500 ml-1" aria-hidden="true">*</span>}
      </label>
      <input
        id={inputId}
        className={cn(
          "mt-1 block w-full px-3 py-2 bg-white dark:bg-gray-800 border rounded-lg text-sm transition-colors",
          "focus:outline-none focus:ring-2 focus:ring-primary-500 focus:border-transparent",
          error
            ? "border-red-500 focus:ring-red-500"
            : "border-gray-300 dark:border-gray-600"
        )}
        aria-required={props.required}
        aria-invalid={!!error}
        aria-describedby={cn(error && errorId, hint && hintId)}
        {...props}
      />
      {hint && !error && (
        <p id={hintId} className="mt-1 text-sm text-gray-500 dark:text-gray-400">{hint}</p>
      )}
      {error && (
        <p id={errorId} className="mt-1 text-sm text-red-600 dark:text-red-400" role="alert">{error}</p>
      )}
    </div>
  );
}
```

### Card

```tsx
interface CardProps {
  children: React.ReactNode;
  className?: string;
  padding?: "sm" | "md" | "lg";
}

const paddingStyles = {
  sm: "p-3",
  md: "p-4 sm:p-6",
  lg: "p-6 sm:p-8",
};

function Card({ children, className, padding = "md" }: CardProps) {
  return (
    <div className={cn(
      "bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-xl shadow-sm",
      paddingStyles[padding],
      className
    )}>
      {children}
    </div>
  );
}
```

## 애니메이션 가이드라인

### 기본 원칙

- **목적이 있는 애니메이션만** 사용한다. 장식적 애니메이션은 지양한다.
- **`prefers-reduced-motion`** 미디어 쿼리를 존중한다.
- 애니메이션 지속 시간은 **150ms ~ 300ms** 범위를 기본으로 한다.

### duration 기준

| 용도 | duration | 예시 |
|------|----------|------|
| 마이크로 인터랙션 | 150ms | 버튼 hover, 색상 변경 |
| 일반 트랜지션 | 200ms | 드롭다운 열기/닫기, 탭 전환 |
| 복잡한 트랜지션 | 300ms | 모달 열기/닫기, 페이지 전환 |
| 강조 애니메이션 | 500ms 이하 | 알림 등장, 토스트 슬라이드 |

### easing 기준

| 용도 | easing | Tailwind 클래스 |
|------|--------|----------------|
| 기본 전환 | `ease-in-out` | `ease-in-out` |
| 등장 | `ease-out` | `ease-out` |
| 퇴장 | `ease-in` | `ease-in` |

### 사용 패턴

```tsx
// 버튼 hover 효과 (150ms)
<button className="transition-colors duration-150 ease-in-out hover:bg-primary-700">

// 드롭다운 열기 (200ms)
<div className={cn(
  "transition-all duration-200 ease-out",
  isOpen ? "opacity-100 translate-y-0" : "opacity-0 -translate-y-1 pointer-events-none"
)}>

// 모달 오버레이 (300ms)
<div className="transition-opacity duration-300 ease-in-out">
```

### 모션 감소 대응

사용자가 시스템 설정에서 모션 감소를 요청한 경우, 애니메이션을 비활성화한다.

```tsx
// Tailwind에서 제공하는 motion-safe/motion-reduce 사용
<div className="motion-safe:animate-bounce motion-reduce:animate-none">

// 전역 CSS에서 일괄 대응
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## 접근성 (a11y) 가이드

### 필수 준수 사항

1. **시맨틱 HTML**: 적절한 HTML 요소를 사용한다. (`button`, `nav`, `main`, `article` 등)
2. **대체 텍스트**: 모든 `img` 태그에 의미 있는 `alt` 속성을 제공한다. 장식용 이미지는 `alt=""`로 비워둔다.
3. **키보드 탐색**: 모든 인터랙티브 요소는 키보드로 접근 가능해야 한다.
4. **포커스 표시**: 포커스 상태가 시각적으로 명확해야 한다. (`focus-visible` 사용)
5. **색상 대비**: 텍스트와 배경의 대비 비율 최소 4.5:1을 유지한다. (WCAG 2.1 AA 기준)
6. **ARIA 속성**: 시맨틱 HTML로 불충분한 경우에만 ARIA를 사용한다.

```tsx
// 좋은 예
<button aria-label="메뉴 닫기" onClick={handleClose}>
  <XIcon aria-hidden="true" />
</button>

// 나쁜 예
<div onClick={handleClose}>
  <XIcon />
</div>
```

### 키보드 탐색 규칙

| 키 | 동작 |
|----|------|
| `Tab` | 다음 포커스 가능한 요소로 이동 |
| `Shift + Tab` | 이전 포커스 가능한 요소로 이동 |
| `Enter` / `Space` | 버튼 클릭, 링크 이동, 체크박스 토글 |
| `Escape` | 모달/드롭다운 닫기 |
| `Arrow Keys` | 탭 목록, 라디오 그룹, 드롭다운 항목 간 이동 |

```tsx
// 모달에서 포커스 트랩 및 Escape 처리
function Modal({ isOpen, onClose, children }: ModalProps) {
  useEffect(() => {
    function handleKeyDown(e: KeyboardEvent) {
      if (e.key === "Escape") onClose();
    }
    if (isOpen) {
      document.addEventListener("keydown", handleKeyDown);
    }
    return () => document.removeEventListener("keydown", handleKeyDown);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div role="dialog" aria-modal="true" aria-labelledby="modal-title">
      {children}
    </div>
  );
}
```

### 포커스 스타일

```tsx
// 프로젝트 전역 포커스 스타일 (tailwind.config.js 또는 global CSS)
// focus-visible을 사용하여 마우스 클릭 시에는 포커스 링을 숨기고,
// 키보드 탐색 시에만 표시한다.
<button className="focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-primary-500 focus-visible:ring-offset-2">
```

### 폼 접근성

- 모든 입력 필드에 연결된 `label`을 제공한다.
- 에러 메시지를 `aria-describedby`로 입력 필드에 연결한다.
- 필수 필드에 `aria-required="true"`를 추가한다.
- 에러 발생 시 `aria-invalid="true"`를 설정하고 `role="alert"`로 에러 메시지를 알린다.

```tsx
<div>
  <label htmlFor="email">이메일</label>
  <input
    id="email"
    type="email"
    aria-required="true"
    aria-invalid={!!error}
    aria-describedby="email-error"
  />
  {error && <p id="email-error" role="alert">{error}</p>}
</div>
```

### 스크린리더 유틸리티

```tsx
// 시각적으로는 숨기되 스크린리더에는 읽히는 유틸리티
// Tailwind의 sr-only 클래스 활용
<span className="sr-only">새 알림 3개</span>

// 동적으로 변하는 콘텐츠를 스크린리더에 알림
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>
```

## 관련 문서

- [프로젝트 구조](project-structure.md)
- [코드 리뷰 체크리스트](code-review-checklist.md)
- [상태 관리 전략](state-management.md)
- [에러 핸들링](error-handling.md)
