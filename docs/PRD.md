# PRD: Notion CMS 기술 블로그

> 프로젝트명은 가칭입니다. 확정되면 이 문서와 `src/config/site.ts`의 사이트 이름을 함께 수정합니다.

## 1. 프로젝트 개요

| 항목 | 내용 |
| --- | --- |
| 프로젝트명 | Notion 기술 블로그 (가칭) |
| 목적 | Notion을 CMS로 활용해 기술 글을 작성·발행하는 개인 기술 블로그 구축 |
| CMS 선택 이유 | Notion API를 활용하여 비개발자도 콘텐츠 관리 가능. 별도 관리자 화면 없이 Notion에서 쓰고 발행 상태만 바꾸면 사이트에 반영된다 |
| 대상 사용자 | **작성자**: 코드 수정 없이 Notion에서 글을 관리하는 사람 / **독자**: 기술 글을 읽고 탐색하는 방문자 |

### 1.1 배경과 목표

- 글 작성과 사이트 개발을 분리한다. 작성자는 Notion만 사용한다.
- 기존 스타터(Next.js + shadcn/ui)의 레이아웃, 테마, 컴포넌트를 재사용해 빠르게 MVP를 만든다.
- 성공 기준: Notion에서 `Published`로 바꾼 글이 설정한 재검증 주기 안에 사이트에 노출된다.

## 2. 주요 기능

| # | 기능 | 설명 | 우선순위 |
| --- | --- | --- | --- |
| 1 | 글 목록 | `Published` 상태의 글을 발행일 최신순으로 카드 형태로 노출 | MVP |
| 2 | 글 상세 | Notion 페이지 본문(블록)을 `/blog/[slug]`에 렌더링 | MVP |
| 3 | 태그·카테고리 필터 | 목록에서 태그/카테고리로 글을 좁혀 본다 | MVP |
| 4 | 다크 모드 | 기존 `ThemeProvider`(next-themes) 재사용 | MVP (기존 기능) |
| 5 | SEO | 글별 `metadata`, `sitemap`, `robots` | MVP |
| 6 | 검색 | 제목·요약 기준 검색 | 이후 |
| 7 | 페이지네이션 | 글이 많아질 때 커서 기반 더 보기 | 이후 |
| 8 | RSS | `/feed.xml` 제공 | 이후 |

## 3. 기술 스택

| 영역 | 선택 |
| --- | --- |
| Frontend | Next.js 16.4 (App Router), React 19.3, TypeScript |
| CMS | Notion API (`@notionhq/client`) |
| Styling | Tailwind CSS v4, shadcn/ui (Base UI 기반 `base-nova`) |
| Icons | Lucide React |
| 폼/검증 | react-hook-form + zod v4 (필요 시) |

> 템플릿 초안에는 Next.js 15로 적혀 있었으나 실제 프로젝트가 16.4이므로 16.4를 기준으로 한다.

### 3.1 구현 시 확인할 사항

- Notion API는 버전별로 데이터베이스 쿼리 방식(데이터 소스 개념 등)이 다를 수 있다. 구현 전에 context7로 최신 문서를 확인한다.
- Notion 블록 렌더링은 자체 렌더러 vs 기존 라이브러리(`react-notion-x` 등) 중 선택해야 한다. 지원 블록 범위, 번들 크기, React 19/Next 16 호환성을 비교해 결정한다 (열린 질문).

## 4. Notion 데이터베이스 구조

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| Title | title | 글 제목 |
| Slug | rich_text | URL 식별자 (`/blog/[slug]`). 영문 소문자·하이픈, 중복 불가 |
| Status | select | `Draft` / `Published`. `Published`만 사이트에 노출 |
| PublishedAt | date | 발행일. 목록 정렬 기준 |
| Tags | multi_select | 기술 태그 (예: nextjs, typescript) |
| Category | select | 대분류 (예: 프론트엔드, 백엔드, 회고) |
| Summary | rich_text | 목록 카드와 meta description에 쓰는 요약 |
| Cover | files | 대표 이미지 (선택) |

- 본문은 페이지 콘텐츠(블록)에 작성한다.
- Notion 통합(Integration)을 만들고 해당 DB를 통합에 공유(Connect)해야 API로 읽을 수 있다.

## 5. 화면 구성

| 경로 | 화면 | 설명 |
| --- | --- | --- |
| `/` | 홈 | 소개와 최신 글 3~6개 |
| `/blog` | 글 목록 | 카드 목록, 태그·카테고리 필터 (`searchParams`) |
| `/blog/[slug]` | 글 상세 | 제목, 메타(발행일·태그), 본문, 이전/다음 글 |
| 404 / 에러 | 안내 화면 | 기존 `not-found.tsx`, `error.tsx` 재사용 (`retry` prop 사용) |

로딩 상태는 Suspense 경계와 `loading.tsx` 스켈레톤으로 처리한다.

## 6. 아키텍처 반영 사항

기존 계층 규칙(`docs/architecture.md`)을 따른다.

- 라우트: `src/app/(marketing)/blog/` 하위에 배치해 헤더·푸터 레이아웃을 재사용한다.
- Notion 연동 코드는 `src/lib/notion/`에 두고 **서버 컴포넌트에서만** import 한다. API 키가 클라이언트 번들에 들어가면 안 된다.
- UI는 `components/ui`(shadcn)를 조합한 `components/common`(예: 글 카드, 태그 배지)과 페이지 전용 컴포넌트로 나눈다. 폭은 `Container`로 통일한다.
- 색상은 시맨틱 토큰만 사용한다.
- 페이지 `metadata`는 `title`만 지정하고, 글 상세는 `generateMetadata`로 제목·요약을 채운다.
- `src/config/site.ts`의 `mainNav`, `routeLabels`에 `/blog`를 추가한다 (`Breadcrumbs`가 라벨을 참조).

### 6.1 캐싱과 재검증

- `next.config.ts`에서 `cacheComponents: true`이므로 Notion 패칭 함수에 `'use cache'`, `cacheLife`, `cacheTag`를 사용한다.
- 반영 방식: 기본은 시간 기반 재검증. 즉시 반영이 필요하면 Notion 변경 → 웹훅/수동 호출 → `revalidateTag` 경로를 추가한다.
- `generateStaticParams`로 발행된 글의 slug를 사전 렌더링하고, 새 글은 요청 시점에 생성되게 한다. 동적 API 사용 시 Suspense 처리가 필요하다 (빌드의 prerender 오류로 확인).
- 코드 작성 전 `node_modules/next/dist/docs/`의 캐싱 가이드를 읽는다.

## 7. 환경 변수

| 이름 | 용도 |
| --- | --- |
| `NOTION_API_KEY` | Notion 통합 시크릿 (서버 전용, `NEXT_PUBLIC_` 접두사 금지) |
| `NOTION_DATABASE_ID` | 블로그 글 DB ID |
| `NEXT_PUBLIC_SITE_URL` | 기존 값. 사이트 URL, `metadataBase` 기준 |

구현 시 `.env.example`에 키 이름만 추가한다 (값은 커밋하지 않는다).

## 8. MVP 범위

**포함**
- Notion DB 연동 (읽기 전용)
- 글 목록 (`Published`만, 최신순) / 글 상세 (제목·메타·본문)
- 태그·카테고리 필터
- 글별 metadata, sitemap
- 로딩·에러·404 처리
- 시간 기반 재검증

**제외 (이후 단계)**
- 검색, 페이지네이션, RSS
- 댓글, 조회수, 뉴스레터
- 웹훅 기반 즉시 반영
- 다국어

## 9. 구현 단계

1. **준비**: Notion 통합 생성, DB 스키마 작성 및 공유, 샘플 글 입력, 환경 변수 설정
2. **데이터 계층**: `@notionhq/client` 설치, `lib/notion/` 클라이언트·타입·패칭 함수(목록, slug 조회, 블록 조회)와 캐싱
3. **목록 페이지**: `/blog` 라우트, 글 카드 컴포넌트, 로딩·빈 상태, `site.ts` 네비게이션 갱신
4. **상세 페이지**: `/blog/[slug]`, 블록 렌더링(제목, 문단, 목록, 코드, 이미지, 인용 등 우선 지원), `generateStaticParams`, `generateMetadata`
5. **필터**: 태그·카테고리 필터 UI와 쿼리 연동
6. **홈·SEO·캐싱 정리**: 홈 최신 글, sitemap/robots, 재검증 주기 조정
7. **QA·배포**: 빌드 오류(prerender) 확인, `lint`·`tsc`, Playwright로 주요 화면 점검, 배포

## 10. 비기능 요구사항

- **성능**: 목록·상세는 사전 렌더링/캐시로 제공하고 요청마다 Notion을 호출하지 않는다.
- **접근성**: 시맨틱 마크업, 이미지 `alt`, 키보드 탐색, 다크 모드 대비 확보.
- **SEO**: 글별 title/description, Open Graph, sitemap.
- **보안**: API 키는 서버에서만 사용, 렌더링 시 사용자 입력을 신뢰하지 않는다.

## 11. 리스크와 열린 질문

| 항목 | 내용 | 대응 |
| --- | --- | --- |
| Notion API 제한 | 요청 속도 제한(평균 초당 3회 수준) | 캐싱으로 호출 최소화, 빌드 시 병렬 호출 제어 |
| 이미지 URL 만료 | Notion 호스팅 파일 URL은 일정 시간 후 만료 | 재검증 주기를 짧게 하거나 이미지를 별도 저장소/프록시로 처리 |
| 블록 렌더링 범위 | 지원하지 않는 블록 노출 문제 | 지원 블록 목록을 정하고 미지원 블록은 안전하게 생략 |
| 렌더러 선택 | 자체 구현 vs 라이브러리 | 3.1의 기준으로 구현 4단계 전에 결정 |
| Slug 중복·누락 | 작성자 실수로 라우팅 충돌 | 조회 시 검증, 누락 글은 목록에서 제외 |
| 프로젝트명 | 가칭 상태 | 확정 후 문서와 사이트 설정 갱신 |
