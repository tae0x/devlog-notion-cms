# DevLog

Notion을 CMS로 활용한 개인 개발 블로그입니다. Notion 데이터베이스에서 글을 작성하고 상태를 `발행됨`으로 바꾸면 블로그에 반영되도록 만드는 것이 목표입니다. 별도 관리자 화면이나 코드 수정 없이 Notion만으로 콘텐츠를 관리합니다.

> **진행 상황**: 현재는 Next.js 16 스타터킷 위에 기획(PRD)을 작성한 단계입니다. Notion 연동과 블로그 화면은 아직 구현되지 않았습니다.

## 주요 기능 (계획)

| 단계 | 기능 |
| --- | --- |
| MVP | Notion 데이터베이스에서 글 목록 가져오기 |
| MVP | 글 상세 페이지 |
| MVP | 반응형 디자인 |
| 이후 | 카테고리별 필터링 |
| 이후 | 검색 |

자세한 요구사항, Notion 데이터베이스 구조, 구현 단계는 [docs/PRD-devlog.md](docs/PRD-devlog.md)를 참고하세요.

## 기술 스택

| 구분 | 사용 기술 |
| --- | --- |
| 프레임워크 | Next.js 16 (App Router), React 19 |
| 언어 | TypeScript (strict) |
| CMS | Notion API (`@notionhq/client`, 도입 예정) |
| 스타일 | Tailwind CSS 4, shadcn/ui (`base-nova`, Base UI 기반) |
| 아이콘 | lucide-react |
| 다크모드 | next-themes |
| 폼 / 검증 | react-hook-form, zod, @hookform/resolvers |
| 전역 상태 | zustand |
| 범용 훅 | usehooks-ts |
| 토스트 | sonner |
| 배포 | Vercel |

## 시작하기

```bash
npm install
cp .env.example .env.local   # 필요 시 환경 변수 수정
npm run dev
```

http://localhost:3000 에서 확인할 수 있습니다.

| 스크립트 | 설명 |
| --- | --- |
| `npm run dev` | 개발 서버 실행 |
| `npm run build` | 프로덕션 빌드 |
| `npm run start` | 프로덕션 서버 실행 |
| `npm run lint` | ESLint 검사 |

## 문서

상세 내용은 `docs/` 폴더에 있습니다. 개발 서버에서는 `/docs`, `/examples` 경로에서도 볼 수 있습니다.

| 문서 | 내용 |
| --- | --- |
| [docs/PRD-devlog.md](docs/PRD-devlog.md) | DevLog PRD (프로젝트 개요, 기능, Notion DB 구조, 화면, MVP, 구현 단계) |
| [docs/PRD.md](docs/PRD.md) | Notion CMS 기술 블로그 PRD (초기 기획안) |
| [docs/architecture.md](docs/architecture.md) | 폴더 구조(`src/`), 컴포넌트 계층, 코드 규칙, 문서·예제 삭제 방법 |
| [docs/guide.md](docs/guide.md) | shadcn 컴포넌트, 다크모드·토스트·폼·전역 상태·훅 사용법, 환경 변수 |
