# DevLog 로드맵

> 기준 문서: [docs/PRD.md](PRD.md). 총 예상 기간은 **9~14일**입니다.

## 개요

| Phase | 목표 | 예상 기간 | 주요 산출물 |
| --- | --- | --- | --- |
| 1. 프로젝트 초기 설정 | 개발 기반 마련 | 1-2일 | Notion 연동 환경, DevLog 기준 사이트 설정·레이아웃 |
| 2. 공통 모듈 개발 | 재사용 코드 선구현 | 2-3일 | `lib/notion/` 함수, 공통 컴포넌트, 공통 타입 |
| 3. 핵심 기능 개발 | 블로그 기본 기능 | 3-4일 | 글 목록, 글 상세, Notion 콘텐츠 렌더링 (**MVP 완료**) |
| 4. 추가 기능 개발 | 부가 기능 | 2-3일 | 카테고리 필터링, 검색, SEO |
| 5. 최적화 및 배포 | 품질 향상과 출시 | 1-2일 | 성능·반응형 개선, Vercel 배포 |

PRD 기준 MVP는 글 목록, 글 상세, 반응형 디자인입니다. 따라서 Phase 3까지 끝나면 MVP가 완성되고, Phase 4는 "MVP 이후" 기능입니다.

스타터킷에 이미 `SiteHeader`, `SiteFooter`, `Container`, `ThemeProvider`, `error.tsx`, `not-found.tsx`, `loading.tsx`가 있습니다. Phase 1~2의 레이아웃·공통 컴포넌트 작업은 새로 만들기보다 DevLog에 맞게 정리하고 재사용하는 것이 기본입니다.

---

## Phase 1: 프로젝트 초기 설정 (1-2일)

**예상 소요 시간**: 1-2일

**이유**: 견고한 기반 없이는 기능 개발이 어렵습니다.

### 작업

- [ ] Next.js 프로젝트 구조 설정
  - `src/config/site.ts`의 `siteConfig`(이름, 설명)를 DevLog 기준으로 변경
  - `mainNav`, `routeLabels`를 블로그 경로 기준으로 정리 (스타터 데모인 docs·examples·app-shell·login·signup 제거는 선택 사항, 방법은 [architecture.md](architecture.md) 참고)
  - 루트 `layout.tsx`의 metadata title template 확인
- [ ] Notion API 연동 환경 구축
  - `@notionhq/client` 설치
  - Notion 통합(Integration) 생성, 데이터베이스 생성(Title, Category, Tags, Published, Status 필드), 통합에 연결(Connect), 샘플 글 입력
  - 환경 변수 `NOTION_API_KEY`, `NOTION_DATABASE_ID` 정의 (서버 전용, `NEXT_PUBLIC_` 금지) 후 `.env.example`에 키 이름만 추가
  - context7으로 `@notionhq/client` 최신 문서 확인 (데이터베이스 조회 방식이 버전별로 다름)
- [ ] 기본 레이아웃 구조 생성
  - `(marketing)` 라우트 그룹의 헤더·푸터 레이아웃을 DevLog 기준으로 정리
  - `Container`로 페이지 폭 통일, 시맨틱 토큰만 사용

### 완료 기준

- [ ] `@notionhq/client`가 설치되어 있고 `.env.example`에 키 이름이 있다
- [ ] 로컬에서 Notion 데이터베이스를 조회하는 테스트가 성공한다
- [ ] 사이트 이름과 네비게이션이 DevLog 기준으로 표시된다
- [ ] `npm run lint`, `npx tsc --noEmit`, `npm run build`가 통과한다

---

## Phase 2: 공통 모듈 개발 (2-3일)

**예상 소요 시간**: 2-3일

**이유**: 모든 기능에서 재사용되는 코드를 먼저 만들어야 중복을 방지할 수 있습니다.

### 작업

- [ ] Notion API 공통 함수 (`src/lib/notion/`, 서버 컴포넌트에서만 import)
  - `fetchPages`: `발행됨` 상태만 조회, 발행일 최신순 정렬
  - `fetchPageContent`: 페이지 ID로 메타와 본문 블록 조회
  - `'use cache'`, `cacheLife`, `cacheTag`로 캐싱 (`cacheComponents: true`)
- [ ] 공통 컴포넌트
  - Header, Footer: 기존 `SiteHeader`, `SiteFooter`를 DevLog에 맞게 수정
  - Card: 글 카드(`PostCard`)를 `components/common`에 추가
- [ ] 공통 타입 정의
  - `Post`: id, title, category, tags, publishedAt, status
  - `Category`

### 완료 기준

- [ ] `fetchPages`가 `발행됨` 글만 발행일 최신순으로 반환한다
- [ ] `fetchPageContent`가 ID로 글 메타와 본문 블록을 반환한다
- [ ] Notion 코드가 클라이언트 번들에 포함되지 않는다 (API 키 노출 없음)
- [ ] `Post`, `Category` 타입이 정의되어 Notion 응답이 타입으로 변환된다
- [ ] 글 카드가 `components/common`에서 계층 규칙(하위 계층만 import)을 지키며 동작한다

---

## Phase 3: 핵심 기능 개발 (3-4일)

**예상 소요 시간**: 3-4일

**이유**: 블로그의 가장 기본이 되는 기능입니다.

**사전 결정**: 블록 렌더링 방식(자체 렌더러 vs `react-notion-x` 등 라이브러리)을 이 Phase 시작 전에 정합니다. 지원 블록 범위와 React 19 호환성을 비교합니다.

### 작업

- [ ] 블로그 글 목록 페이지
  - `/`에 최근 글 목록 표시 (`PostCard` 사용)
  - 로딩은 Suspense 경계와 스켈레톤, 빈 상태 처리
- [ ] 블로그 글 상세 페이지
  - `/posts/[id]` 라우트, 페이지 ID로 조회
  - `generateMetadata`로 글 제목 지정
  - 없는 글은 `not-found.tsx`로 처리
- [ ] Notion 컨텐츠 렌더링
  - 우선 지원 블록: 제목, 문단, 목록, 코드, 이미지, 인용
  - 미지원 블록은 안전하게 생략

### 완료 기준 (**MVP 완료 지점**)

- [ ] `/`에 `발행됨` 글만 최신순으로 노출된다
- [ ] `/posts/[id]`에서 지원 블록이 정상 렌더링되고, 미지원 블록이 있어도 오류 없이 표시된다
- [ ] 존재하지 않는 ID는 404, API 오류는 `error.tsx`(`retry`)로 처리된다
- [ ] Notion에서 상태를 `발행됨`으로 바꾸면 재검증 주기 안에 반영된다
- [ ] `npm run build`에서 prerender 오류가 없다

---

## Phase 4: 추가 기능 개발 (2-3일)

**예상 소요 시간**: 2-3일

**이유**: 핵심 기능이 완성된 후 부가 기능을 추가합니다.

### 작업

- [ ] 카테고리 필터링
  - `/categories/[category]` 카테고리별 글 목록
  - `mainNav`, `routeLabels`에 경로 반영 (`Breadcrumbs` 라벨)
- [ ] 검색 기능
  - 제목(필요 시 태그) 기준 검색
  - 방식(Notion 쿼리 vs 클라이언트 필터)은 글 수를 보고 결정
- [ ] SEO 최적화
  - 글별 title·description, Open Graph
  - `sitemap`, `robots`

### 완료 기준

- [ ] 카테고리 페이지에서 해당 카테고리 글만 노출된다
- [ ] 검색어를 입력하면 제목에 맞는 글만 표시되고, 결과가 없을 때 안내가 나온다
- [ ] 글 상세의 title, description, Open Graph가 글별로 설정된다
- [ ] `/sitemap.xml`, `/robots.txt`가 정상 응답한다

---

## Phase 5: 최적화 및 배포 (1-2일)

**예상 소요 시간**: 1-2일

**이유**: 기능이 완성된 후 품질을 향상합니다.

### 작업

- [ ] 성능 최적화
  - 캐시·재검증 주기 조정, 요청마다 Notion을 호출하지 않는지 확인
  - 이미지 최적화, Notion 호스팅 이미지 URL 만료 대응
- [ ] 반응형 디자인 개선
  - 모바일·태블릿·데스크톱 점검, 다크 모드 대비 확인
  - 접근성 점검 (시맨틱 마크업, 이미지 `alt`, 키보드 탐색)
- [ ] Vercel 배포
  - Vercel 프로젝트 환경 변수 등록 (`NOTION_API_KEY`, `NOTION_DATABASE_ID`, `NEXT_PUBLIC_SITE_URL`)
  - 배포 후 실제 URL에서 동작 확인

### 완료 기준

- [ ] Playwright로 주요 화면을 세 가지 화면 크기에서 확인했다
- [ ] Notion 이미지가 재검증 주기와 관계없이 깨지지 않는다
- [ ] `npm run lint`, `npx tsc --noEmit`, `npm run build`가 통과한다
- [ ] Vercel 배포 URL에서 글 목록·상세가 표시되고, Notion에서 발행한 글이 반영된다

---

## 공통 완료 기준 (모든 Phase)

- `npm run lint`, `npx tsc --noEmit`, `npm run build` 통과
- 색상은 시맨틱 토큰만 사용, 하드코딩 색 금지
- 폴더 계층 규칙 준수 ([architecture.md](architecture.md))
- 서버 컴포넌트가 기본, `"use client"`는 말단에만 사용
- API 키는 서버에서만 사용

## 리스크와 대응

| 항목 | 일정 영향 | 대응 |
| --- | --- | --- |
| Notion API 요청 속도 제한 | 빌드·개발 중 호출 실패 | 캐싱으로 호출 최소화 (Phase 2) |
| 이미지 URL 만료 | 배포 후 이미지 깨짐 | 재검증 주기 조정 또는 별도 처리 (Phase 5) |
| 블록 렌더러 선택 | 지연 시 Phase 3 전체 지연 | Phase 3 시작 전에 결정 |
| URL 형식 (페이지 ID vs Slug) | 라우트 구조 변경 | Phase 3 전에 `Slug` 필드 추가 여부 결정 |
| 검색 방식 | Phase 4 구현 방식 변경 | 글 수를 보고 결정 |
