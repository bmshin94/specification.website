# specification.website 프로젝트 분석 정리 (한국어)

> 이 문서는 `specification.website` 저장소를 전수조사하여 "무엇을 하는 프로젝트인지, 언제 쓰는지,
> 어떤 도움이 되는지"를 한국어로 정리한 것입니다. 스펙 콘텐츠 자체가 아니라 **저장소에 대한 분석
> 노트**이므로, 스펙 페이지(`src/content/spec/`)나 체인지로그와는 별개의 문서입니다.
>
> 작성일: 2026-09-29

## 링크

| 항목           | 주소                                                           |
| -------------- | -------------------------------------------------------------- |
| 원본 저장소    | https://github.com/jdevalk/specification.website               |
| 이 포크        | https://github.com/bmshin94/specification.website              |
| 라이브 사이트  | https://specification.website                                  |
| MCP 엔드포인트 | https://mcp.specification.website/mcp                          |
| 서버 카드      | https://specification.website/.well-known/mcp/server-card.json |
| 에이전트 카드  | https://specification.website/.well-known/agent-card.json      |
| 라이선스       | 코드 MIT / 콘텐츠 CC BY 4.0                                    |
| 원작자         | Joost de Valk (Yoast SEO 창업자) — https://joost.blog          |

## 1. 이 프로젝트는 무엇인가

"좋은 웹사이트란 무엇인가"를 **플랫폼 중립적으로** 정의한 공개 명세서이자, 그것을 사람과 AI
에이전트 양쪽이 읽을 수 있게 배포하는 **Astro 정적 사이트**입니다.

- 프레임워크도 튜토리얼도 아니고, HTML Living Standard와 같은 의미의 **명세(spec)** 입니다.
- 모든 페이지가 1차 출처(WHATWG, W3C, IETF RFC, IANA, WCAG, schema.org)를 인용합니다.
- 각 항목에는 상태 등급이 붙습니다: `required` / `recommended` / `optional` / `avoid`.

### 카테고리 10개 (총 170 페이지)

| 카테고리        | 페이지 수 | 내용                              |
| --------------- | --------: | --------------------------------- |
| foundations     |        20 | HTML, head, 문서 기본             |
| seo             |        14 | 검색 가시성                       |
| accessibility   |        29 | WCAG 기반 규칙                    |
| security        |        20 | 헤더, 전송, 정책                  |
| well-known      |        12 | `/.well-known/` 표준 경로         |
| agent-readiness |        21 | AI 에이전트가 읽을 수 있는 사이트 |
| performance     |        26 | Core Web Vitals, 캐싱, 폰트       |
| privacy         |         7 | 동의, 신호, 선택 존중             |
| resilience      |         8 | 우아한 실패                       |
| i18n            |        13 | 언어, 로케일, 방향                |

## 2. 폴더 구조 해부

| 경로                                      | 역할                                                                                     |
| ----------------------------------------- | ---------------------------------------------------------------------------------------- |
| `src/content/spec/<카테고리>/<슬러그>.md` | **단일 진실 공급원.** 170개 마크다운. 나머지 표면은 전부 여기서 파생                     |
| `src/content.config.ts`                   | 콘텐츠 컬렉션 스키마. 프론트매터가 틀리면 빌드 실패(의도된 동작)                         |
| `src/lib/site.ts`                         | 사이트 메타데이터 + 카테고리 정의                                                        |
| `src/pages/`                              | HTML 라우트 + `.md` / `.jsonld` / `llms.txt` / `rss.xml` / `sitemap` / `okf/` 엔드포인트 |
| `src/content/changelog/`                  | 62개 변경 이력. **자동 생성이 아니라 손으로 작성**                                       |
| `src/content/considered/`                 | 15개 "일부러 넣지 않은 표준" 기록 — 거절 이유와 되돌릴 조건까지                          |
| `mcp/`                                    | 별도 Cloudflare Worker. MCP 서버 + A2A(JSON-RPC) 엔드포인트                              |
| `functions/`                              | Pages 미들웨어(콘텐츠 협상, 봇 로깅), `/reports` 수집기, `/admin/stats` 대시보드         |
| `public/.well-known/`                     | security.txt, agent-card.json, mcp/server-card.json, agent-skills/, api-catalog 등       |
| `ops/routines/daily-standards-scan.md`    | 매일 도는 AI 에이전트의 작업 지시서 — 표준 스캔 후 draft PR + Slack 요약                 |
| `scripts/`                                | OG 이미지 생성(sharp), SKILL.md sha256 무결성 검사                                       |
| `CLAUDE.md`                               | 약 45KB 분량의 AI 에이전트용 기여 규칙서                                                 |

## 3. 설계 철학 3가지

### ① 하나의 소스, 모든 표면 자동 생성

마크다운 한 개를 고치면 HTML 페이지, `/checklist/`, 카테고리 인덱스, `.md` 엔드포인트,
`llms.txt`, `llms-full.txt`, RSS, 사이트맵, OKF 번들(`okf.tar.gz`), Pagefind 검색 인덱스,
MCP 서버 데이터가 전부 자동으로 갱신됩니다. 파생 표면을 손으로 고치는 것은 버그입니다.

### ② 사이트 자체가 스펙의 실물 예제

`Vary: Accept`를 권장하는 페이지를 쓰면 미들웨어가 실제로 그 헤더를 붙이고, `Server-Timing`
페이지를 쓰면 실제로 `Server-Timing`을 보냅니다. 권장하는 것과 배포하는 것 사이의 괴리는
버그로 취급합니다.

### ③ AI 에이전트도 1급 독자

같은 URL에 `Accept: text/markdown`을 보내면 리다이렉트 없이 마크다운 본문을 200으로 돌려주고,
`Content-Location`과 `Vary: Accept`를 정확히 설정합니다. RFC 9530 `Content-Digest`로 무결성
검증도 가능합니다.

## 4. 어떻게 쓰나 (설치 및 사용)

### 요구 사항

- Node.js 22.12 이상

### 사이트

```bash
git clone https://github.com/bmshin94/specification.website
cd specification.website
npm install          # prepare 스크립트가 .githooks를 연결
npm run dev          # http://localhost:31337
npm run build        # astro build + pagefind + 무결성 검사
npm run preview
npm run check        # astro check
npm run lint         # eslint
npm run format       # prettier --write .
npm run assets       # 아이콘 + OG 이미지 재생성
npm run check:skill  # SKILL.md sha256 무결성 검사
```

### MCP 워커

```bash
cd mcp
npm install
npm run build:data   # 신규 클론이면 필수 (src/data.json 생성)
npm run dev          # wrangler dev, :31338
npm test             # 프로토콜 검증 (의존성 없음, node:assert)
npm run typecheck
```

### 소비하는 방법

```bash
# 마크다운 콘텐츠 협상
curl -H "Accept: text/markdown" https://specification.website/spec/foundations/title/

# LLM용 인덱스
curl https://specification.website/llms.txt

# 전체 지식 번들
curl -O https://specification.website/okf.tar.gz
```

MCP 클라이언트(Claude Code, Cursor 등) 설정 예:

```json
{
  "mcpServers": {
    "spec-website": {
      "type": "http",
      "url": "https://mcp.specification.website/mcp"
    }
  }
}
```

### 스펙 페이지를 추가할 때 (빠뜨리기 쉬운 단계 포함)

1. 기존 `.md` 복사 → 2. 프론트매터 수정 → 3. 정해진 섹션 구조로 본문 작성 →
2. `npm run dev` 확인 → 5. 파생 표면 확인 → 6. **체인지로그 항목 추가** →
3. **`npm run assets` 후 OG 이미지 커밋** → 8. **`npm run sign:skill`** → 9. 커밋/푸시

6, 7, 8단계가 가장 자주 누락됩니다.

## 5. 플러그인인가, 스킬인가, MCP인가

셋 다 아니면서 셋 다를 **제공**합니다. 본체는 "Astro 정적 사이트 + 지식 콘텐츠"이고,
그 위에 다음 인터페이스가 얹혀 있습니다.

| 형태                 | 제공 여부 | 비고                                                    |
| -------------------- | :-------: | ------------------------------------------------------- |
| Claude Code 플러그인 |   아님    | `.claude-plugin/` 없음. 대신 `CLAUDE.md` 규칙서가 있음  |
| Agent Skill          |   제공    | `/.well-known/agent-skills/` 로 HTTP 발견되는 표준 스킬 |
| MCP 서버             |   제공    | 툴 6개 + `audit_url` 프롬프트                           |
| A2A 에이전트         |   제공    | `agent-card.json` + `/a2a/v1` JSON-RPC                  |
| llms.txt / OKF 번들  |   제공    | LLM 인덱스 및 지식 포맷 번들                            |

### MCP 툴 목록

| 툴                                            | 용도                                  |
| --------------------------------------------- | ------------------------------------- |
| `search(query, limit?)`                       | 전체 페이지 전문 검색, 본문 발췌 포함 |
| `list_topics({ category?, status?, limit? })` | 필터링된 목록                         |
| `get_topic({ slug })`                         | 한 페이지의 전체 마크다운 + 출처      |
| `get_checklist({ category?, status? })`       | 감사용 체크리스트                     |
| `get_categories()`                            | 카테고리 10개와 개수                  |
| `get_changes({ since?, type?, limit? })`      | 특정 시점 이후 변경분                 |

## 6. API 토큰이 필요한가

| 상황                           | 토큰 필요 여부                                 |
| ------------------------------ | ---------------------------------------------- |
| 사이트 열람, `.md`, `llms.txt` | 불필요                                         |
| MCP 서버 호출                  | 불필요 (무인증, 무상태, CORS 개방)             |
| A2A 엔드포인트                 | 불필요                                         |
| 로컬 개발 및 빌드              | 불필요                                         |
| 본인 Cloudflare 계정에 배포    | `CLOUDFLARE_API_TOKEN` (Workers Scripts: Edit) |
| `/admin/stats` 대시보드        | `CF_ACCOUNT_ID`, `CF_ANALYTICS_TOKEN`          |

요약하면 **사용자는 무료·무인증, 배포자만 Cloudflare 자격 증명이 필요**합니다.

## 7. 왜 주목받는가

1. 원작자가 Yoast SEO 창업자 Joost de Valk로, 웹 표준·SEO 업계에서 신뢰도가 높습니다.
2. `llms.txt`, MCP, A2A, Agent Skills 등 "AI 에이전트가 사이트를 읽는 방법"이 최대 화두인
   시점에, 그것들을 한 곳에서 **실제로 구현해 보여준** 사례입니다.
3. 권장하는 내용을 자기 서버가 실제로 이행합니다(worked example). Lighthouse CI, 링크 체커,
   보안 스캔이 CI에 걸려 있습니다.
4. 플랫폼 중립성이 규칙으로 강제되어 어떤 스택에서도 쓸 수 있습니다.
5. 모든 페이지가 1차 출처를 인용하고, 죽은 링크는 CI가 잡습니다.
6. `CLAUDE.md`, 매일 도는 표준 스캔 루틴, `/considered/` 기록 등 **AI와 함께 저장소를 운영하는
   방법** 자체가 참고 사례가 됩니다.

## 8. 로컬 에이전트 구축에 주는 도움

- **지식 소스**: MCP를 붙이면 웹 표준 관련 환각이 줄고, 1차 출처 기반으로 답합니다.
- **MCP 구현 교과서**: `mcp/src/index.ts`, `mcp/src/tools.ts`에서 Streamable HTTP 무상태 전송,
  구·신 프로토콜 리비전 동시 지원, JSON Schema 기반 입출력 정의, 빌드타임 데이터 번들링,
  의존성 없는 프로토콜 테스트를 배울 수 있습니다.
- **에이전트 친화 아키텍처 설계도**: 콘텐츠 협상 미들웨어, `/.well-known/` 발견 체계,
  `Link` 헤더 광고, RFC 9530 무결성, 봇 탐지 및 Analytics Engine 로깅.
- **CLAUDE.md 작성법**: "하지 마 + 대신 이렇게" 구조가 일관되어, 다른 프로젝트 규칙서의
  템플릿으로 쓰기 좋습니다.

## 9. React / PHP로 다시 만들 수 있는가

가능합니다. 코드는 MIT, 콘텐츠는 CC BY 4.0이므로 **출처를 밝히면 상업적 재사용도 허용**됩니다.

### React / Next.js 대응

| 원본                       | 대체                                        |
| -------------------------- | ------------------------------------------- |
| Astro                      | Next.js App Router                          |
| Content Collections        | Contentlayer / next-mdx-remote / velite     |
| Pagefind                   | 그대로 사용 가능, 또는 Orama / FlexSearch   |
| Cloudflare Pages Functions | Next.js Middleware / Route Handlers         |
| MCP Worker                 | `@modelcontextprotocol/sdk` + Route Handler |
| Tailwind v4                | 그대로                                      |

### PHP / Laravel 대응

| 원본          | 대체                                              |
| ------------- | ------------------------------------------------- |
| 마크다운 파싱 | `spatie/yaml-front-matter` + `league/commonmark`  |
| 정적 생성     | Laravel + 풀페이지 캐시, 또는 Statamic / Jigsaw   |
| 검색          | Laravel Scout + Meilisearch, 또는 Pagefind 그대로 |
| 콘텐츠 협상   | 미들웨어에서 `Accept` 헤더 분기                   |
| MCP 서버      | `php-mcp/server`, 또는 JSON-RPC 직접 구현         |

워드프레스 플러그인 형태도 유효한 선택지입니다(관리자 화면 체크리스트, 자동 진단,
`security.txt` / `llms.txt` 자동 생성).

**권장 방향**: 동일한 사이트를 복제하기보다, 콘텐츠는 CC BY로 인용하고 원본이 하지 않는
**자동 검사 · 적용 · 증명**을 만드는 쪽이 가치가 큽니다.

## 10. 수익화 아이디어

> 전제: 콘텐츠 라이선스는 CC BY 4.0이므로 저작자 표시(Joost de Valk / specification.website),
> 원문 링크, 변경 여부 명시가 필수입니다. 원본 사이트를 그대로 복제해 광고를 붙이는 방식은
> 가치도 낮고 권장되지 않습니다. **지식이 아니라 적용·자동화·증명을 파는 것**이 핵심입니다.

### 티어 1 — 현실적이고 빠른 것

1. **사이트 자동 감사 SaaS** — URL을 넣으면 170개 항목을 자동 검사해 점수, 리포트, 개선 방법을
   제시. 무료 / $19 / $99 / $499(에이전시) 티어. 판정 근거가 1차 출처라 신뢰 확보가 쉽고,
   기존 도구가 다루지 않는 agent-readiness, well-known, resilience 영역이 차별점.
   스택 예: Next.js + Playwright + 큐 + Postgres + Stripe.
2. **AI 에이전트 준비도(AEO) 점수** — "우리 사이트를 ChatGPT/Claude/Perplexity가 제대로 읽는가"를
   측정. 발견성 / 파싱 가능성 / 신뢰 신호 / 에이전트 인터페이스 4축 점수화. 어떤 AI 봇이 실제로
   방문했는지 로그로 보여주는 기능은 `functions/_shared/bot-detect.ts` 접근을 그대로 응용 가능.
   1회 진단 / 월 모니터링 / 개선 실행 3단 가격.
3. **CI/CD 봇(Spec Guard)** — GitHub App으로 PR마다 위반 항목을 근거와 함께 코멘트. OSS 무료,
   팀 월 정액, 기업 셀프호스팅. 개발 예산으로 결제되고 이탈률이 낮음.
4. **워드프레스 플러그인** — 무료판(진단) + 프로판(자동 수정, 파일 자동 생성) + 에이전시판
   (다중 사이트 대시보드). Yoast가 증명한 시장.

### 티어 2 — 서비스형

5. **감사 컨설팅** — 리포트 단건, 감사 + 실행, 월 리테이너. 제품 개발 없이 즉시 시작 가능하며
   SaaS의 초기 고객과 요구사항 발굴 경로가 됨.
6. **한국어 번역·로컬라이즈** — CC BY로 합법적 번역 가능. 한글 인코딩, KWCAG, 국내 결제·인증 등
   한국 웹 환경 항목을 덧붙이고, 수익은 광고가 아니라 감사 서비스·교육의 유입 경로로 설계.
7. **교육 상품** — 온라인 강의, 기업 워크숍, 유료 뉴스레터(표준 변화 큐레이션은 데일리 스캔
   자동화를 그대로 활용).

### 티어 3 — 장기

8. **인증 배지** — Level A / AA / AAA 등급과 임베드 배지, 연 갱신제. 배지 자체가 백링크이자
   광고가 됨. 다만 중립성 신뢰가 사업의 전부이므로 유료 등급 상향은 금물.
9. **MCP 툴 마켓플레이스 / 에이전트 인프라** — `audit_site(url)` 같은 검사형 툴을 유료 API로
   제공(건당 과금). 에이전트가 직접 호출하는 구조.
10. **기업용 커스텀 스펙 관리** — 사내 표준(디자인 시스템, 브랜드 규칙, 보안 정책)을 동일한
    구조로 관리해주는 엔터프라이즈 제품. 구축비 + 연 유지보수.

### 권장 로드맵

| 기간    | 할 일                                                |
| ------- | ---------------------------------------------------- |
| 1~2개월 | 무료 감사 툴 런칭, 커뮤니티 배포, 이메일 리스트 확보 |
| 2~4개월 | 유료 SaaS 전환 + 컨설팅으로 현금흐름 확보            |
| 4~8개월 | GitHub App, 워드프레스 플러그인으로 채널 확장        |
| 8개월~  | 인증 배지, 엔터프라이즈 커스텀 스펙 관리             |

### 지켜야 할 것

1. CC BY 저작자 표시를 철저히 (출처 링크 + 변경사항 명시).
2. 원본 사이트 복제가 아니라 적용·자동화를 판매.
3. 원본 오픈소스에도 기여(번역, 버그 리포트, 새 페이지 PR) — 신뢰가 곧 영업력.

## 11. 한 줄 결론

`specification.website`는 **웹 표준 지식의 압축 패키지이자, "AI가 읽는 사이트"의 레퍼런스
구현이며, AI와 함께 저장소를 운영하는 방법의 모범 사례**입니다. 그대로 복제하기보다는,
여기서 검증된 구조(단일 소스 파생, 콘텐츠 협상, MCP/A2A 발견 체계)를 자기 제품에 이식하고,
콘텐츠는 CC BY로 인용하면서 **자동 검사와 적용**을 제품화하는 방향이 가장 실용적입니다.
