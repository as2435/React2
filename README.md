# 202230209 김태율
# 9월 30일 (5주차)

## Layout and Pages

### 1. Route 방식 비교: Pages Router vs App Router

| 항목 | Page Router | App Router |
| --- | --- | --- |
| **도입 시기** | 초기부터 존재 | Next.js 13부터 도입 |
| **루트 디렉토리** | `pages/` | `app/` |
| **파일 기반** | `pages/about.js` → `/about` | `app/about/page.tsx` → `/about` |
| **특징** | 간단하고 익숙함 (기존 React 방식과 유사) | 더 유연하고 강력한 기능 지원 |
| **대표 기능** | 동적 라우트, `getStaticProps` 등 | 레이아웃 중첩, 서버 컴포넌트, 로딩 UI, 병렬 라우트 등 |
| **추천 여부** | 유지보수 중 (권장 X) | **Next.js 15부터 기본 권장 방식** |

* **Pages Router**
* `export default function Page()` 형식으로 구성
* 각 파일은 하나의 페이지 컴포넌트
* SSR/SSG 함수는 `getStaticProps`, `getServerSideProps` 등으로 처리


* **App Router**
* `page.js`: 각 세그먼트의 페이지
* `layout.js`: 해당 세그먼트 이하의 모든 페이지에 공통 레이아웃 적용
* 추가 기능: `loading.js`, `error.js`, `not-found.js`, route groups, parallel routes 등



### 2. App Router의 강력한 기능들

| 기능 | 설명 |
| --- | --- |
| **중첩 레이아웃** | 여러 레벨의 `layout.js` 파일을 통해 레이아웃을 계층적으로 구성 가능 |
| **서버 컴포넌트 (RSC)** | 서버에서만 렌더링되는 컴포넌트로 성능 최적화 가능 (React Server Component) |
| **로딩 UI** | 페이지 전환 중 보여 줄 `loading.js` 파일 제공 |
| **에러 UI** | 특정 경로에서만 발생하는 에러를 처리할 `error.js` 제공 |
| **병렬 라우팅** | 하나의 경로 안에서 탭 같은 독립적인 뷰를 병렬로 렌더링 가능 |

### 3. 프로젝트 별 추천 방식

| 상황 | 추천 방식 |
| --- | --- |
| **새 프로젝트 시작** | App Router (`app` 디렉토리 기반) |
| **기존 프로젝트 유지보수** | `pages/` 계속 사용 가능하지만, 점차 마이그레이션 필요 |
| **수동 라우팅이 필요한 경우** | React + `react-router-dom` 사용 (Next.js는 자동 라우팅이 기본) |

---

## Linking and Navigating

### Introduction

* Next.js에서 경로(route)는 기본적으로 **서버에서 렌더링** 됨.
* 클라이언트는 새 경로를 표시하기 전 서버의 응답을 기다려야 하는 경우가 많음.
* Next.js에는 **Prefetching, Streaming, Client-side transitions** 기능이 기본 제공되어 네비게이션 속도가 빠르고 반응성이 뛰어남.

### 1. How navigation works (네비게이션 작동 방식)

#### 1-1. Server Rendering (서버 렌더링)

* Next.js에서 레이아웃과 페이지는 기본적으로 **React 서버 컴포넌트**.
* 초기 및 후속 네비게이션 시, 서버 컴포넌트 페이로드는 클라이언트로 전송되기 전 서버에서 생성됨.
* **서버 렌더링의 2가지 유형:**
1. **정적 렌더링 (사전 렌더링):** 빌드 시점이나 재검증 중에 발생, 결과는 캐시(cache)됨. *(재검증: 전체 앱 다시 빌드 없이 캐시 항목 업데이트)*
2. **동적 렌더링:** 클라이언트 요청에 대한 응답으로 요청 시점에 발생.



> **알아두면 좋은 점 (초기 렌더링)**
> * 일반 React 앱(CSR)은 처음 방문 시 빈 HTML + JS 파일을 받아 브라우저가 렌더링함.
> * Next.js는 처음 방문(initial visit) 시 서버가 HTML을 미리 생성해 전달하므로 브라우저가 즉시 뼈대+콘텐츠를 표시 가능 (UX 및 SEO 유리). 이후 React가 **Hydration(하이드레이션)** 과정을 거쳐 상호작용 가능해짐.
> 
> 

#### 1-2. Prefetching (미리 가져오기)

* 사용자가 링크를 클릭하기 전, 백그라운드에서 다음 경로를 로드하는 프로세스.
* `<Link>` 컴포넌트와 연결된 경로를 자동으로 뷰포트에 가져옴 (`<a>` 태그는 프리페칭 안 함).
* **경로에 따른 프리페칭:**
* **정적 경로:** 전체 경로 프리페칭
* **동적 경로:** 건너뛰거나, `loading.tsx`가 있는 경우 부분적 프리페칭 (서버의 불필요한 작업 방지)



#### 1-3. Streaming (스트리밍)

* 서버가 전체 경로 렌더링을 기다리지 않고, 준비되는 즉시 클라이언트에 전송.
* 페이지 일부가 로드 중이더라도 사용자는 더 빨리 콘텐츠를 볼 수 있음 (부분적 미리 가져오기).
* 라우팅 폴더에 `loading.tsx` 생성 시, Next.js가 내부적으로 `page.tsx`를 `<Suspense>`로 자동 래핑함.
* **이점:** 즉각적인 시각적 피드백, 공유 레이아웃 상호작용 유지, 핵심 웹 지표(TTFB, FCP, TTI) 개선.

#### 1-4. Client-side transitions (클라이언트 측 전환)

* 일반적인 서버 렌더링은 페이지 이동 시 전체가 로드되어 상태 삭제, 스크롤 초기화가 발생.
* Next.js는 `<Link>`를 통한 클라이언트 측 전환으로 이를 방지. 공유 레이아웃/UI를 유지하며 콘텐츠만 동적으로 업데이트.

---

## Core Web Vitals (웹 성능 지표)

*웹페이지가 기술적으로 로드되는 순서대로 시간을 측정*

* **TTFB (Time To First Byte):** 네트워크/서버 성능. 길면 서버나 네트워크 연결 문제.
* **FCP (First Contentful Paint):** 사용자가 렌더링 시작을 인지하는 순간 (하얀 화면에서 무언가 처음 뜰 때).
* **TTI (Time To Interactive):** 페이지가 완전히 상호작용 가능한 준비가 완료된 시간. *(최근엔 TBT, INP로 대체되는 추세)*
* **LCP (Largest Contentful Paint):** 뷰포트 내 가장 큰 요소(텍스트, 이미지 등) 표시 시간.
* **FID (First Input Delay):** 첫 상호작용 시도부터 웹페이지가 응답하는 시간.
* **CLS (Cumulative Layout Shift):** 레이아웃 불안정성/이동 빈도 측정. (원인: 치수 없는 이미지/광고/iframe, 동적 콘텐츠 등)

---

## 전환을 느리게 만드는 요인

#### 2-1. 동적 경로 없는 `loading.tsx`

* 동적 경로 이동 시 응답 대기로 인해 지연이 발생할 수 있음. 동적 경로에 `loading.tsx`를 추가하면 부분 프리페칭 및 즉시 로딩 UI 표시가 가능.

#### 2-2. 동적 세그먼트 없는 `generateStaticParams`

* `generateStaticParams`가 누락되면 사전 렌더링되지 않고 런타임에 동적 렌더링으로 대체됨.

**`generateStaticParams`가 없는 경우의 코드 예시:**

```tsx
import { posts } from "../posts";

export default async function Posts({
  params,
}: {
  // 런타임 전달 params는 Promise 형태일 수 있음
  params: Promise<{slug: string}>;
}) {
  const { slug } = await params; // params 해제
  const post = posts.find((p) => p.slug === slug);

  if(!post) {
    return <h1>게시글을 찾을 수 없습니다.</h1> // 404 처리
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}

```

---

---

# 9월 23일 (4주차)

## Link Component

* `<Link>`는 HTML `<a>` 요소를 확장하여 프리페칭과 클라이언트 사이드 네비게이션을 제공하는 React 컴포넌트.

```tsx
import Link from 'next/link'

export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>
}

```

**Props:**

| Prop | Example | Type | Required |
| --- | --- | --- | --- |
| **href** | `href="/dashboard"` | String or Object | **yes** |
| **replace** | `replace={false}` | Boolean | - |
| **scroll** | `scroll={false}` | Boolean | - |
| **prefetch** | `prefetch={false}` | Boolean, "auto", null | - |
| **onNavigate** | `onNavigate={(e) => {}}` | Function | - |
| **transitionTypes** | `transitionTypes={['slide-in']}` | string[] | - |

---

## Layout and Pages (중첩 & 동적 라우트)

### 중첩 라우트 (Nested route)

* 다중 URL 세그먼트로 구성 (`/blog/[slug]` -> `/(Root)` + `blog` + `[slug]`).
* **폴더**는 URL 세그먼트에 매핑되며, **파일**(`page`, `layout`)은 UI를 만듦.

### `[slug]`의 이해 (동적 세그먼트)

* `[slug]`는 특정 페이지를 식별하는 URL의 일부이자 불러올 데이터의 **Key**.
* `[foo]`로 지정했다면 데이터에 `foo` 키가 있어야 함.
* **async / await:** 서버 데이터를 읽어올 때 딜레이 오류 방지를 위해 비동기 처리 필요.
* `{params}` 구조 분해 할당을 통해 Next.js가 전달하는 객체에서 값을 추출함 (`await params`로 Promise 해제).
* *주의:* 데이터가 크다면 `.find`(시간복잡도 O(n)) 대신 DB 쿼리 사용 권장.

### 중첩 레이아웃 (Nesting layouts)

* 폴더 계층 구조에 따라 자식 prop(`children`)을 통해 자식 레이아웃을 감쌈.

### 동적 vs 정적 렌더링 (searchParams 관련)

| 항목 | 동적 렌더링 | 정적 렌더링 |
| --- | --- | --- |
| **예시** | `/product?page=2` (요청 시 생성) | `/about`, `/blog` (미리 생성됨) |
| **장점** | 유연함, 쿼리/요청 기반 응답 가능 | 빠름, 캐싱 가능 |
| **searchParams 사용** | **가능** | 불가능 |

* **searchParams:** URL의 쿼리 문자열 (`?category=shoes&page=2`).
* 이것을 사용하는 순간 요청 시점에만 값을 알 수 있으므로 Next.js는 **동적 렌더링**으로 자동 처리함.

### React vs Next.js 라우팅 방식 차이

| 항목 | React (기본) | Next.js |
| --- | --- | --- |
| **라우팅 방식** | 수동 (직접 설정) | 자동 (폴더/파일 기반) |
| **라우터 도구** | `react-router-dom` 등 외부 라이브러리 필요 | 자체 내장 파일 기반 라우팅 시스템 |
| **정의 방식** | 코드에서 `<Route>`로 직접 정의 | 파일/폴더 이름으로 라우트 자동 매핑 |
| **예시** | `<Route element="{<ABOUT" path="/about"/>}/>` | `app/about/page.tsx` → `/about` 자동 생성 |

---

---

# 9월 16일 (3주차)

## 라우팅 그룹 및 비공개 폴더

| Path | URL Pattern | Notes |
| --- | --- | --- |
| `app/(marketing)/page.tsx` | `/` | URL에서 제외된 라우팅 그룹 |
| `app/(shop)/cart/page.tsx` | `/cart` | `(shop)` 내에서 레이아웃 공유 |
| `app/blog/_components/Post.tsx` | `-` | 라우팅 대상 아님 (UI 유틸리티 안전 공간) |
| `app/blog/data.ts` | `-` | 라우팅 대상 아님 (유틸리티 안전 공간) |

## 병렬 및 가로채기 라우팅 (Parallel & Intercepting)

| Pattern | Meaning | 일반적인 사용 사례 |
| --- | --- | --- |
| `@folder` | 명명된 슬롯 (Named slot) | 사이드바 + 메인 콘텐츠 |
| `(.)folder` | 동일 레벨 가로채기 (Intercept) | 모달에서 형제 라우트 미리보기 |
| `(..)folder` | 한 레벨 위에서 가로채기 | 부모의 자식 라우트를 오버레이로 열기 |
| `(..)(..)folder` | 두 레벨 위에서 가로채기 | 깊게 중첩한 오버레이 |
| `(...)folder` | 루트에서 가로채기 | 현재 뷰에 임의의 라우트 표시 |

## Open Graph Protocol

* 웹사이트 링크를 SNS(카카오톡, 페북, X 등)에 공유할 때 "미리보기"를 생성하는 표준 프로토콜.
* 웹페이지의 메타 태그에 선언하여 사용.

---

## Organizing your project (프로젝트 구성하기)

### Component 계층 구조

특수 파일은 다음과 같은 중첩 계층 구조로 렌더링됨:

```tsx
<Layout>
  <Template>
    <ErrorBoundary fallback={<Error />}>
      <Suspense fallback={<Loading />}>
        <ErrorBoundary fallback={<NotFound />}>
          <Page />
        </ErrorBoundary>
      </Suspense>
    </ErrorBoundary>
  </Template>
</Layout>

```

### Layout vs Template 차이

| 파일 | 특징 | 상태/DOM 유지 | 사용 사례 |
| --- | --- | --- | --- |
| **layout.tsx** | 경로별 공유 레이아웃 | **유지됨 (정적)** | 네비게이션, 사이드바, 공통 레이아웃 |
| **template.tsx** | 매번 새 인스턴스 생성 | **초기화됨 (동적)** | 페이지별로 상태 초기화가 필요한 경우 |

### Colocation (코로케이션) & 파일 구성 규칙

* **코로케이션:** 관련된 파일과 폴더를 기능별로 그룹화하는 것. `app` 디렉토리 내에 파일을 두어도 `page.js`나 `route.js`가 없으면 외부에서 접근 불가하여 안전함.
* **비공개 폴더 (`_folderName`):** 라우팅 시스템에서 완전히 제외됨. UI/라우팅 로직 분리, 내부 파일 구성, 이름 충돌 방지에 유용. (URL로 강제 접근하려면 `%5F` 사용)
* **라우팅 그룹 (`(folderName)`):** URL 경로에는 포함되지 않으며 폴더를 그룹화(목적, 팀별 등)하거나 특정 세그먼트끼리 레이아웃을 공유할 때 사용.
* **`src` 디렉토리:** 애플리케이션 코드(`app`)와 프로젝트 설정 파일(루트 위치)을 분리하기 위해 Next.js에서 권장하는 방식.

---

---

# 9월 9일 (2주차)

## Installation (설치 및 세팅)

### 프로젝트 수동 생성

```bash
# 기본 패키지 설치
pnpm i next@latest react@latest react-dom@latest

```

### package.json 스크립트 추가

* `next dev`: 개발 서버 시작
* `next build`: 프로덕션 빌드
* `next start`: 프로덕션 서버 시작
* `next lint`: ESLint 실행
* *(Turbopack이 기본 번들러. Webpack 사용 시 `next dev --webpack` 실행)*

### 디렉토리 및 파일 생성

* `app` 폴더 생성 후 필수 파일인 `layout.tsx` (html, body 태그 포함) 및 `page.tsx` (홈페이지) 생성.
* **오류 처리 (TypeScript 환경):** TS 환경에서는 타입 정의 패키지가 필요함.
```bash
pnpm add -D @types/react @types/react-dom

```



**일반 설치 vs 개발용 설치(-D)**

| 구분 | 일반 설치 (`pnpm add`) | 개발용 설치 (`pnpm add -D`) |
| --- | --- | --- |
| **등록 위치** | `dependencies` | `devDependencies` |
| **용도** | 실제 서비스 구동에 필수 | 빌드, 테스트, 린팅 등 개발 시에만 필요 |
| **배포 환경** | 빌드 결과물 포함 / 프로덕션 서버 설치 | 빌드/배포 시 제외됨 |
| **대표 예시** | React, Vue, Express, Axios 등 | TypeScript, ESLint, Prettier, Jest 등 |

### 자동 생성되는 항목 (CLI 사용 시)

* `package.json` 스크립트 / `public` 디렉토리
* `tsconfig.json` / `eslint.config.mjs` / Tailwind CSS 설정
* `src` 디렉토리 / `app/layout.tsx`, `app/page.tsx`
* Turbopack 사용 설정 및 import alias (`paths` 자동 생성)

---

## 프로젝트 구조 및 구성

* **route:** 경로 / **routing:** 경로를 찾아가는 과정 / **segment:** 라우팅과 관련된 디렉토리 단위
* **최상위 폴더:** 애플리케이션 코드 및 정적 자산 구성
* **최상위 파일:** 앱 구성, 종속성, 환경 변수 등 정의
* **라우팅 파일:** 레이아웃, 스켈레톤 로딩, 에러, API 경로, 페이지 등 정의

### Next.js의 동적 라우팅 (Dynamic Routing) 3가지

하위 경로(Depth) 허용 범위와 기본 경로 처리 여부에 따라 나뉨.

1. **일반 동적 라우팅**
* 디렉토리 구조: `[slug]`
* 작동 방식: 1개의 특정 경로 세그먼트만 동적으로 매칭.


2. **Catch-all 라우팅**
* 디렉토리 구조: `[...slug]`
* 작동 방식: 하위 무한 Depth의 모든 경로를 하나의 배열로 전달하여 매칭.


3. **Optional Catch-all 라우팅**
* 디렉토리 구조: `[[...slug]]`
* 작동 방식: Catch-all과 동일하나, **동적 파라미터가 없는 기본 경로(예: `/posts`)까지도 매칭**함.
