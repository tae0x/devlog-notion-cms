# Development Guidelines

> AI Agent 전용 규칙 문서. 일반 개발 지식은 제외하고 이 프로젝트에만 적용되는 규칙을 명령형으로 정의한다.
> 기준 문서: `docs/PRD.md`, `docs/ROADMAP.md`, `docs/architecture.md`, `CLAUDE.md`

## 프로젝트 개요

- 프로젝트명: **DevLog** (Notion CMS Blog). Notion을 CMS로 쓰는 개인 개발 블로그
- 스택: Next.js 16.4 (App Router) / React 19.3 / TypeScript strict / Tailwind CSS v4 / shadcn/ui (Base UI 기반 `base-nova`) / `@notionhq/client`
- 현재 상태: 스타터킷(`Next.js Starter`)에서 DevLog로 전환 중. `siteConfig.name`, `mainNav`, `routeLabels`는 아직 스타터 값이다
- 진행 기준: `docs/ROADMAP.md`의 Phase 순서(1 초기 설정 → 2 공통 모듈 → 3 핵심 기능 → 4 추가 기능 → 5 최적화·배포)를 따른다. MVP는 Phase 3까지다

## 프로젝트 아키텍처

### 계층 (아래 → 위 방향으로만 import)

`config/site.ts`, `stores/`, `hooks/`, `lib/utils.ts` → `components/ui` → `components/common` → `components/layout` → `components/providers` → `app/**/layout.tsx` → `app/**/page.tsx`

- 상위 계층만 하위 계층을 import 한다. 역방향 import 금지
- 모든 소스는 `src/` 아래에 둔다. 경로 별칭은 `@/*` = `src/*`
- `components/docs/`, `components/examples/`는 `app/(marketing)/docs`, `app/(marketing)/examples`, `app/(app-shell)` 에서만 import 한다

### 신규 코드 배치

| 대상 | 위치 |
| --- | --- |
| Notion 연동 함수·타입 | `src/lib/notion/` (신규 생성) |
| 글 카드 등 재사용 UI (`PostCard`) | `src/components/common/` |
| 글 목록 페이지 | `src/app/(marketing)/page.tsx` (`/`) |
| 글 상세 페이지 | `src/app/(marketing)/posts/[id]/page.tsx` |
| 카테고리 페이지 (Phase 4) | `src/app/(marketing)/categories/[category]/page.tsx` |
| 공통 타입 `Post`, `Category` | `src/lib/notion/` 하위 |
| 노션 블록 렌더러 | `src/components/common/` (Notion 데이터에 의존하는 로직은 `lib/notion/`에 분리) |

- 블로그 라우트는 반드시 `(marketing)` 그룹에 둔다 (SiteHeader/SiteFooter 재사용)
- `(app-shell)`에는 블로그 라우트를 두지 않는다

## 코드 표준

- TypeScript strict. `any`, `@ts-ignore`, `@ts-expect-error` 금지 (불가피하면 사유를 한국어 주석으로 명시)
- 코드 주석·문서는 **한국어**, 변수명·함수명은 **영어**
- ESLint(`eslint-config-next` core-web-vitals + typescript) 위반 금지. 새 규칙 비활성화 주석(`eslint-disable`) 금지
- 클래스 병합: `import { cn } from "cn"` 사용. `@/lib/utils`에서 `cn`을 import 하지 않는다
- 색상: 시맨틱 토큰(`text-muted-foreground`, `bg-muted`, `border` 등)만 사용. `text-gray-500`, `#fff`, `bg-[#...]` 같은 하드코딩 금지
- 페이지 폭: `Container`(`components/common/container.tsx`)로 통일
- 서버 컴포넌트가 기본. `"use client"`는 말단 컴포넌트에만 사용
- zustand 스토어(`stores/`)는 클라이언트 컴포넌트에서만 import 한다
- `usehooks-ts`의 `useLocalStorage`, `useMediaQuery` 등은 `initializeWithValue: false`를 지정한다
- 레이아웃 props 타입은 전역 헬퍼 `LayoutProps<"/경로">` 사용 (`app/layout.tsx` 참고)
- 페이지 `metadata`는 `title`만 지정한다 (루트 title template이 `%s | 사이트명`을 완성). 글 상세는 `generateMetadata`로 글 제목만 채운다

## Next.js 16 규칙

- 코드 작성 전 `node_modules/next/dist/docs/`의 해당 가이드를 읽는다 (학습 데이터와 API가 다름)
- `next.config.ts`에 `cacheComponents: true`, `partialPrefetching: true`가 켜져 있다
- 동적 API(`cookies`, `headers`, `Date.now` 등)는 Suspense 경계 또는 `use cache`로 처리한다
- 에러 바운더리 복구 prop은 `reset`이 아니라 **`retry`** 다 (`src/app/error.tsx`)
- `AGENTS.md`의 `BEGIN:nextjs-agent-rules` 블록은 `next dev`가 자동 생성하므로 편집 금지

## Notion 연동 표준

- `@notionhq/client`는 아직 미설치이고 `NOTION_API_KEY`, `NOTION_DATABASE_ID`도 `.env.example`에 없다. Phase 1에서 `npm install @notionhq/client`로 추가하고 키 이름을 `.env.example`에 넣는다
- 구현 전 context7(`mcp__context7__*`)으로 `@notionhq/client` 최신 문서를 확인한다 (버전별 데이터베이스 조회 방식·데이터 소스 개념이 다름)
- `src/lib/notion/`은 **서버 컴포넌트에서만** import 한다. `"use client"` 파일에서 import 금지
- 환경 변수

| 이름 | 규칙 |
| --- | --- |
| `NOTION_API_KEY` | 서버 전용. `NEXT_PUBLIC_` 접두사 금지 |
| `NOTION_DATABASE_ID` | 서버 전용 |
| `NEXT_PUBLIC_SITE_URL` | 기존 값. `siteConfig.url`에서만 참조 |

- `.env.example`에는 **키 이름만** 추가하고 값은 넣지 않는다. `.env*`(예제 제외)는 커밋 금지
- 데이터베이스 필드: `Title`(title), `Category`(select), `Tags`(multi_select), `Published`(date), `Status`(select: `초안`/`발행됨`). 본문은 속성이 아니라 페이지 블록이다
- `fetchPages`: `Status = 발행됨`만 조회하고 `Published` 내림차순 정렬한다
- `fetchPageContent`: 페이지 ID로 메타와 본문 블록을 조회한다
- 모든 Notion 패칭 함수에 `'use cache'`, `cacheLife`, `cacheTag`를 적용한다. 요청마다 Notion을 호출하지 않는다
- Notion 응답은 `Post`(id, title, category, tags, publishedAt, status)로 변환한 뒤 UI에 전달한다. 원본 응답 객체를 컴포넌트 props로 넘기지 않는다
- 상세 페이지 URL 식별자는 페이지 ID. `Slug` 필드 도입은 사용자 결정 사항이므로 임의로 추가하지 않는다
- 본문 블록 지원 범위: 제목, 문단, 목록, 코드, 이미지, 인용. **미지원 블록은 오류 없이 생략**한다
- 블록 렌더러 방식(자체 구현 vs 라이브러리)은 Phase 3 시작 전 사용자와 합의한다. 임의로 라이브러리를 추가하지 않는다
- Notion 호스팅 이미지 URL은 만료된다. 이미지 처리 변경 시 재검증 주기와 함께 검토한다

## shadcn/ui (Base UI) 표준

- Radix가 아니므로 **`asChild` 사용 금지**. `render` prop을 쓴다

```tsx
// 올바름
<Button render={<Link href="/posts" />} nativeButton={false}>글 목록</Button>

// 금지
<Button asChild><Link href="/posts">글 목록</Link></Button>
```

- `DialogTrigger`, `BreadcrumbLink` 등 트리거·링크 계열 컴포넌트도 `render` 방식이다
- 컴포넌트 추가: `npx shadcn@latest add <name>` 만 사용. 추가 후 생성 파일의 `cn` import가 `"cn"`인지 확인하고 아니면 수정한다
- `src/components/ui/`는 CLI 원본이므로 직접 수정을 최소화한다
- `components.json`의 `tailwind.css`는 `src/app/globals.css`다

## 폼 표준

- 폼은 react-hook-form + `zodResolver` + zod v4 + shadcn `Field` 조합이다
- 기준 예제: `src/app/(marketing)/login/login-form.tsx`, `src/components/examples/signup-form.tsx`
- `SignupForm`이 `app/(marketing)/signup/signup-form.tsx`와 `components/examples/signup-form.tsx` 두 곳에 있다. import 경로를 정확히 지정한다

## 핵심 파일 연동 규칙 (동시 수정 필수)

| 변경 | 함께 수정할 파일 |
| --- | --- |
| 페이지 추가·삭제·경로 변경 | `src/config/site.ts` (`mainNav`, `routeLabels`, 해당 시 `docsNav`·`examplesNav`·`shellNav`) |
| `docsNav` 순서·항목 변경 | `DocPager` 이전/다음 순서가 `docsPages`로 자동 변경됨. `routeLabels`도 확인 |
| 경로 세그먼트 추가 | `routeLabels`에 라벨 추가 (없으면 `Breadcrumbs`에 세그먼트 원문 노출) |
| 사이트 이름·설명 변경 | `siteConfig` (헤더·푸터·metadata가 공유). 루트 `layout.tsx` title template 확인 |
| 새 환경 변수 추가 | `.env.example`(키 이름만), `CLAUDE.md` 환경 변수 절, `docs/PRD.md` 환경 변수 표 |
| 폴더 구조·계층 변경 | `docs/architecture.md`, `CLAUDE.md` 아키텍처 절 |
| 스타터 데모(docs·examples·app-shell) 삭제 | `src/app/(marketing)/docs`, `examples`, `src/app/(app-shell)`, `src/components/docs`, `src/components/examples` 삭제 + `mainNav`, `docsNav`, `examplesNav`, `shellNav`, `routeLabels` 정리. 삭제는 사용자 승인 후에만 수행 |
| ROADMAP 작업 완료 | `docs/ROADMAP.md`의 해당 체크박스 `[ ]` → `[x]` |
| 요구사항 변경 | `docs/PRD.md`와 `docs/ROADMAP.md`를 함께 수정 |

## 워크플로우 표준

- 데이터 흐름: Notion DB → `src/lib/notion/` (캐시된 서버 함수) → 서버 컴포넌트(page) → `components/common`(PostCard 등) → 렌더링
- 로딩: Suspense 경계 + 스켈레톤(`components/ui/skeleton`). 에러: `error.tsx`(`retry`). 없는 글: `not-found.tsx` (`notFound()` 호출)
- 구현 완료 후 검증 명령을 모두 실행한다

```bash
npm run lint
npx tsc --noEmit
npm run build   # cacheComponents prerender 오류 확인
```

- 테스트 러너는 없다. 동작 검증은 `npm run dev` + Playwright MCP로 한다. 산출물 `.playwright-mcp/`는 gitignore 대상이다
- 코드 구현(기능 추가·수정·리팩토링)을 마치면 응답 전에 `code-reviewer` 서브에이전트를 실행하고 지적 사항을 사용자에게 보고한다. 문서·설정만 바뀐 경우는 생략한다

## AI 의사결정 기준

- 요청 범위 밖 코드는 수정하지 않고 제안만 한다
- 불확실하면 추측하지 말고 코드를 확인하거나 사용자에게 질문한다
- 우선순위: 사용자 명시 요청 > `CLAUDE.md` > 이 문서 > `docs/PRD.md`·`docs/ROADMAP.md` > 일반 관례
- 새 UI가 필요할 때: ① `components/ui`에 있는지 확인 → ② 없으면 `npx shadcn@latest add` → ③ 조합이 필요하면 `components/common`에 추가
- 새 라이브러리 추가 전: 기존 의존성(`usehooks-ts`, `zod`, `zustand`, `lucide-react` 등)으로 해결 가능한지 먼저 확인하고, 불가하면 사용자에게 확인한다
- 스타터 데모 코드는 삭제·수정 요청이 있을 때만 건드린다

## `.claude/` 에이전트·명령어·훅 표준

| 파일 | 규칙 |
| --- | --- |
| `.claude/agents/code-reviewer.md` | 코드 구현 직후 반드시 실행. 읽기 전용(Read, Grep, Glob, Bash). 결과를 사용자에게 보고하고 에이전트가 직접 고치게 하지 않는다 |
| `.claude/agents/interior-expert.md` | 인테리어 도메인 콘텐츠를 다룰 때만 사용한다. DevLog 블로그 기능 구현에는 호출하지 않는다 |
| `.claude/commands/git/commit.md` (`/git:commit`) | 사용자가 요청할 때만 실행한다. 이 명령어의 이모지 포맷(`✨ feat: ...`)은 사용자 전역 규칙과 충돌한다. **이모지를 붙이지 않고 `type: 설명` 형식**으로 작성한다 (기존 커밋 이력도 이모지 없음) |
| `.claude/hooks/slack-notify.sh` | 프로젝트 루트 `.env`의 `SLACK_WEBHOOK_URL`, `SLACK_MENTION`을 읽는다. 현재 `settings.local.json`에 훅이 등록되어 있지 않아 비활성 상태다. 이 값을 코드·문서·`.env.example`에 기록하지 않는다 |

- `.env`에 Notion 키와 Slack 값이 함께 들어갈 수 있다. `.env`를 읽거나 출력하지 않는다
- `code-reviewer.md`의 검토 항목은 이 문서의 계층·Next.js·shadcn 규칙과 동일하게 유지한다. 규칙을 바꾸면 두 문서를 함께 수정한다
- `settings.local.json`의 `enabledMcpjsonServers`는 playwright, context7, sequential-thinking, shadcn이다. `.mcp.json`에 서버를 추가하면 여기에도 반영할지 사용자에게 확인한다

## 금지 사항

- 사용자 요청 없이 `git commit`/`git push` 금지
- 커밋 메시지·PR 본문에 `Co-Authored-By` 트레일러, "Generated with Claude Code" 등 Claude 표기 **절대 금지**. 커밋 메시지는 한국어 `type: 설명` 컨벤셔널 형식
- `NOTION_API_KEY` 등 비밀 값을 코드·로그·문서·`.env.example`에 기록하거나 클라이언트 번들에 포함하는 행위 금지
- `"use client"` 파일에서 `src/lib/notion/` import 금지
- `asChild` 사용 금지, 하드코딩 색상 금지, `cn`을 `@/lib/utils`에서 import 금지
- `AGENTS.md`의 자동 생성 블록 직접 편집 금지
- `app/layout.tsx` 외 위치에서 `ThemeProvider`, `TooltipProvider`, `Toaster` 중복 선언 금지
- `<html>`의 `suppressHydrationWarning` 제거 금지
- 계층 역방향 import 금지 (예: `components/ui`가 `components/common`을 import)
- 요청 없이 `components/ui` 원본을 대규모 수정하거나 스타터 데모 폴더를 삭제하는 행위 금지
