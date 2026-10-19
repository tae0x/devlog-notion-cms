# PRD: DevLog

## 1. 프로젝트 개요

| 항목 | 내용 |
| --- | --- |
| 프로젝트명 | DevLog |
| 목적 | Notion을 CMS로 활용한 개인 개발 블로그 |
| CMS 선택 이유 | Notion에서 글을 작성하면 자동으로 블로그에 반영된다. 별도 관리자 화면이나 코드 수정 없이 Notion만으로 콘텐츠를 관리할 수 있다 |
| 배포 | Vercel |

### 1.1 목표

- 글 작성과 사이트 개발을 분리한다. 작성자는 Notion만 사용한다.
- Notion에서 상태를 `발행됨`으로 바꾼 글이 재검증 주기 안에 사이트에 반영된다.
- 기존 스타터(레이아웃, 테마, shadcn/ui)를 재사용해 빠르게 MVP를 만든다.

## 2. 주요 기능

| # | 기능 | 설명 | 단계 |
| --- | --- | --- | --- |
| 1 | 글 목록 | Notion 데이터베이스에서 `발행됨` 글을 가져와 발행일 최신순으로 노출 | MVP |
| 2 | 글 상세 | 개별 글의 본문(페이지 콘텐츠)을 상세 페이지에 표시 | MVP |
| 3 | 반응형 디자인 | 모바일·태블릿·데스크톱 대응 (기본 스타일링 포함) | MVP |
| 4 | 카테고리별 필터링 | 카테고리별 글 목록 페이지 | MVP 이후 |
| 5 | 검색 | 제목(필요 시 태그) 기준 검색 | MVP 이후 |

## 3. 기술 스택

| 영역 | 선택 |
| --- | --- |
| Frontend | Next.js 16.4 (App Router), React 19.3, TypeScript |
| CMS | Notion API (`@notionhq/client`) |
| Styling | Tailwind CSS v4, shadcn/ui (Base UI 기반 `base-nova`) |
| Icons | Lucide React |
| Deployment | Vercel |

> 요구사항 초안의 Next.js 15는 현재 프로젝트(`package.json`)의 버전인 **16.4**로 갱신했다. Tailwind도 v4 기준이다.

### 3.1 구현 시 확인할 사항

- Notion API 버전에 따라 데이터베이스 조회 방식(데이터 소스 개념 등)이 다를 수 있다. 구현 전에 context7로 `@notionhq/client` 최신 문서를 확인한다.
- Next.js 16 API는 학습 데이터와 다르므로 코드 작성 전 `node_modules/next/dist/docs/`의 해당 가이드를 읽는다.
- 본문 블록 렌더링 방식(자체 렌더러 vs 기존 라이브러리)은 지원 블록 범위와 React 19 호환성을 비교해 결정한다 (열린 질문).

## 4. Notion 데이터베이스 구조

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| Title | title | 제목 |
| Category | select | 카테고리 |
| Tags | multi_select | 태그 |
| Published | date | 발행일. 목록 정렬 기준 |
| Status | select | 상태. `초안` / `발행됨` — `발행됨`만 사이트에 노출 |
| Content | page content | 본문. DB 속성이 아니라 페이지 본문(블록)으로 작성 |

- Notion 통합(Integration)을 만들고 해당 데이터베이스를 통합에 연결(Connect)해야 API로 읽을 수 있다.
- 상세 페이지 URL 식별자는 페이지 ID를 사용한다. (slug 속성이 없으므로 가독성 있는 URL이 필요하면 `Slug` 필드 추가를 검토 — 열린 질문)

## 5. 화면 구성

| 화면 | 경로(안) | 설명 |
| --- | --- | --- |
| 홈 | `/` | 최근 글 목록 |
| 글 상세 | `/posts/[id]` | 개별 글 내용 표시 |
| 카테고리 | `/categories/[category]` | 카테고리별 글 목록 (MVP 이후) |

- 로딩은 Suspense 경계와 스켈레톤, 오류는 기존 `error.tsx`(`retry` prop), 없는 글은 `not-found.tsx`로 처리한다.
- 모든 화면은 반응형으로 구성한다.

## 6. 아키텍처 반영 사항

`docs/architecture.md`의 계층 규칙을 따른다.

- 라우트는 `src/app/(marketing)/` 하위에 두어 헤더·푸터 레이아웃을 재사용한다.
- Notion 연동 코드는 `src/lib/notion/`에 두고 **서버 컴포넌트에서만** import 한다. API 키가 클라이언트 번들에 들어가면 안 된다.
- 글 카드 등 재사용 UI는 `components/common`, 페이지 폭은 `Container`, 색상은 시맨틱 토큰만 사용한다.
- 경로가 늘어나면 `src/config/site.ts`의 `mainNav`, `routeLabels`를 함께 수정한다.
- 페이지 `metadata`는 `title`만 지정하고, 글 상세는 `generateMetadata`로 글 제목을 채운다.

### 6.1 캐싱과 재검증

- `cacheComponents: true`이므로 Notion 패칭 함수에 `'use cache'`, `cacheLife`, `cacheTag`를 적용한다.
- 기본은 시간 기반 재검증이며, 즉시 반영이 필요하면 `revalidateTag` 호출 경로(웹훅 등)를 추가한다.
- 동적 API를 쓰면 Suspense 처리가 필요하다. `npm run build`의 prerender 오류로 확인한다.

## 7. 환경 변수

| 이름 | 용도 |
| --- | --- |
| `NOTION_API_KEY` | Notion 통합 시크릿 (서버 전용, `NEXT_PUBLIC_` 금지) |
| `NOTION_DATABASE_ID` | 블로그 글 데이터베이스 ID |
| `NEXT_PUBLIC_SITE_URL` | 기존 값. 사이트 URL |

`.env.example`에는 키 이름만 추가하고 값은 커밋하지 않는다. Vercel에는 프로젝트 환경 변수로 등록한다.

## 8. MVP 범위

**포함**
- Notion API 연동 (읽기 전용)
- 글 목록 및 상세 페이지
- 기본 스타일링
- 반응형 디자인

**MVP 이후**
- 카테고리별 필터링(카테고리 페이지)
- 검색
- 태그 필터, RSS, sitemap, 웹훅 기반 즉시 반영

## 9. 구현 단계

1. **Notion API 패키지 설치 및 환경 설정**: `@notionhq/client` 설치, 환경 변수 정의, `.env.example` 갱신
2. **Notion 데이터베이스 생성 및 API 키 설정**: 위 스키마로 DB 생성, 통합 생성·연결, 샘플 글 입력, 로컬·Vercel에 키 등록
3. **글 목록 페이지 구현**: `src/lib/notion/` 패칭 함수(`발행됨` 필터, 발행일 정렬, 캐싱), 홈의 최근 글 목록, 글 카드
4. **글 상세 페이지 구현**: 페이지 ID로 메타·블록 조회, 본문 렌더링(제목, 문단, 목록, 코드, 이미지, 인용 우선), `generateMetadata`, 404 처리
5. **스타일링 및 최적화**: 반응형 점검, 다크 모드 확인, 로딩·에러 상태, 캐시·재검증 주기 조정, `lint`·`tsc`·`build` 통과, Vercel 배포
6. **(이후) 카테고리 필터링 → 검색**: 카테고리 페이지, 검색 UI/쿼리 추가

## 10. 비기능 요구사항

- **성능**: 요청마다 Notion을 호출하지 않고 캐시된 결과를 제공한다.
- **접근성**: 시맨틱 마크업, 이미지 `alt`, 키보드 탐색, 다크 모드 대비.
- **보안**: API 키는 서버에서만 사용한다.

## 11. 리스크와 열린 질문

| 항목 | 내용 | 대응 |
| --- | --- | --- |
| Notion API 제한 | 요청 속도 제한 | 캐싱으로 호출 최소화 |
| 이미지 URL 만료 | Notion 호스팅 파일 URL은 일정 시간 후 만료 | 재검증 주기 조정 또는 이미지 별도 처리 |
| 블록 렌더링 범위 | 미지원 블록 노출 문제 | 지원 블록을 정하고 미지원 블록은 안전하게 생략 |
| URL 형식 | 페이지 ID 기반 URL은 가독성이 낮음 | `Slug` 필드 추가 여부 결정 |
| 검색 방식 | Notion 쿼리 vs 클라이언트 필터 | MVP 이후 글 수를 보고 결정 |
