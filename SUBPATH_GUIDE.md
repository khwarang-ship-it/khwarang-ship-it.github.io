# ms.hs.kr 하위 경로 만들기 — 다른 프로젝트용 가이드

다른 프로젝트(레포)에서 `ms.hs.kr/무언가/` 형태의 공개 페이지가 필요할 때 참고하는 문서다.
개인정보 처리방침, 학운위 제출용 공개 문서, 프로젝트 소개 페이지 등이 여기 해당한다.

## 먼저 확인할 것 — 정적인가, 서버가 필요한가

`ms.hs.kr`은 2026-08-10부로 Vercel이 서빙한다(예전엔 GitHub Pages). 그래서:

- **정적 페이지/앱** (순수 HTML·CSS·JS로 빌드 결과가 끝나는 것, 또는 `next export` 같은
  정적 내보내기가 가능한 것) → **방법 A**(이 레포 안에 폴더로 추가). Vercel도 정적
  파일을 그대로 서빙하므로 여전히 제일 간단한 방법이다.
- **서버가 필요한 앱** (로그인/세션, API 라우트, SSR, DB 연결, 쿠키 기반 게이트 등) →
  **방법 C**(Vercel rewrite). `mshs-curriculm`(currifair)이 이 경우였다.
- **방법 B(형제 GitHub Pages 레포 자동 전파)는 더 이상 안 쓴다** — GitHub Pages 시절
  구조였고, DNS가 Vercel을 보는 지금은 동작하지 않는다. 아래 설명은 과거 기록 및
  "왜 이 구조를 버렸는지" 참고용으로 남겨둔다.

## (과거 기록) 방법 A/B가 성립했던 이유 — GitHub Pages 시절

이 레포(`khwarang-ship-it/khwarang-ship-it.github.io`)는 이름이 정확히
`계정이름.github.io`라서 GitHub Pages가 이 레포를 **유저 사이트**로 취급하고, 여기 걸린
커스텀 도메인(`ms.hs.kr`, `CNAME` 파일)이 **같은 계정의 다른 레포에도 자동으로
전파**된다. 다른 레포가 자기만의 커스텀 도메인을 따로 설정하지 않으면, GitHub Pages를
켜는 순간 `ms.hs.kr/그레포이름/`으로 자동으로 뜬다. (자세한 배경은 이 레포의
[`CLAUDE.md`](CLAUDE.md) 참고.) 이전 배경·실제로 겪은 버그들은 `CLAUDE.md`의
"호스팅: GitHub Pages → Vercel 이전 완료" 참고.

## 방법 A — 이 레포 안에 폴더로 추가 (가벼운 정적 페이지 1~2장일 때 권장)

개인정보 처리방침처럼 페이지 한두 장이면 새 레포까지 만들 필요 없다.

```bash
git clone https://github.com/khwarang-ship-it/khwarang-ship-it.github.io.git
cd khwarang-ship-it.github.io
mkdir 프로젝트이름
# 프로젝트이름/index.html 또는 프로젝트이름/privacy.html 등 작성
git add 프로젝트이름
git commit -m "Add 프로젝트이름 공개 페이지"
git push origin main
```

30~60초 후 `https://ms.hs.kr/프로젝트이름/파일명.html`에서 바로 열린다. 배포 확인:

```bash
gh api repos/khwarang-ship-it/khwarang-ship-it.github.io/pages/builds/latest --jq .status
```

기존 예시: [`ms-record-linter/privacy.html`](ms-record-linter/privacy.html) → `https://ms.hs.kr/ms-record-linter/privacy.html`

## 방법 B — 프로젝트 자체 레포로 (그 프로젝트가 진짜 자기 저장소가 필요할 때)

앱 소스코드나 별도 이슈 트래커·커밋 히스토리가 필요한 규모의 프로젝트라면 새 레포를 판다.

1. `khwarang-ship-it` 계정 아래 레포 생성. **레포 이름이 곧 URL 경로**가 된다
   (`gh repo create khwarang-ship-it/프로젝트이름 --public`).
2. 그 레포 루트(또는 원하는 브랜치)에 `index.html` 등 정적 페이지를 둔다.
3. 그 레포 Settings → Pages에서 Source를 켠다 (`gh api -X POST repos/khwarang-ship-it/프로젝트이름/pages -f "source[branch]=main" -f "source[path]=/"`).
4. **그 레포에는 절대 커스텀 도메인/`CNAME`을 설정하지 않는다** — 설정하는 순간 자동
   서브패스 전파가 끊기고 그 레포 혼자 다른 도메인으로 튀어나간다.

배포되면 `https://ms.hs.kr/프로젝트이름/`.

## 방법 C — Vercel rewrite로 붙이기 (서버가 필요한 앱)

로그인/세션, API 라우트, DB 연결 등 서버 실행이 필요한 앱(Next.js 등)은 GitHub Pages에
못 올라간다. 이 레포와 그 앱을 각각 Vercel 프로젝트로 배포하고, 이 레포의 `vercel.json`이
경로를 갈라서 넘겨준다.

1. 앱 레포를 Vercel에 새 프로젝트로 배포한다(private 레포도 무료로 가능). 배포되면 나온
   `https://<프로젝트>.vercel.app` 주소를 적어둔다.
2. Next.js라면 `next.config.mjs`에 `basePath: '/프로젝트이름'`을 추가한다. 이러면
   `next/link`·정적 자산 경로는 자동으로 접두어가 붙지만, **자동으로 안 붙는 지점들을
   직접 찾아 고쳐야 한다**:
   - `next/image`의 리터럴 `src="/foo.png"` (import한 이미지가 아닌 경우)
   - 클라이언트 `fetch("/api/...")` 같은 자체 API 호출
   - 미들웨어/`proxy.ts`에서 `NextResponse.redirect(new URL("/어딘가", request.url))`처럼
     직접 만드는 리다이렉트 — `request.nextUrl.pathname`은 basePath가 이미 제거된
     값이라는 점도 같이 감안해야 한다
   - Server Action을 쓰면 `experimental.serverActions.allowedOrigins`에 `ms.hs.kr`을
     추가해야 한다(배포 도메인과 실접속 도메인이 달라서 origin 검증에 걸림)

   실제 사례: `mshs-curriculm` 레포의 커밋 `Add basePath for serving under
   ms.hs.kr/currifair/ via Vercel rewrite`에 이 항목들을 전부 고친 예시가 있다.
3. 앱 레포도 GitHub 연동으로 Vercel 프로젝트를 만들면(vercel.com → Add New → Project
   → 레포 선택) 이후 `main` push마다 자동 배포된다 — 대시보드를 매번 다시 열 필요 없다.
   이 레포(`khwarang-ship-it.github.io`)는 이미 Vercel에 연결돼 있고 `ms.hs.kr` 도메인도
   붙어 있으니, 새 앱을 붙일 땐 그 앱 레포만 새로 Vercel에 연결하면 된다.
4. 이 레포의 `vercel.json`에 rewrite 규칙을 추가한다 — 와일드카드 규칙만으로는 세그먼트
   0개(경로 자체)를 못 잡으므로 반드시 둘 다 넣는다:
   ```json
   {
     "trailingSlash": false,
     "rewrites": [
       { "source": "/프로젝트이름", "destination": "https://<프로젝트>.vercel.app/프로젝트이름" },
       { "source": "/프로젝트이름/:path*", "destination": "https://<프로젝트>.vercel.app/프로젝트이름/:path*" }
     ]
   }
   ```
   **`"trailingSlash": false`를 빠뜨리지 말 것** — Next.js 앱은 대부분 슬래시 붙은
   경로(`/프로젝트이름/`)를 자체적으로 308 리다이렉트하는데, 이게 rewrite 프록시를
   못 넘어가고 루트 프로젝트에서 `X-Vercel-Error: NOT_FOUND`(플랫폼 레벨 404)로
   튕긴다. currifair 배포 때 실제로 겪은 버그 — Vercel 엣지에서 슬래시를 먼저
   정리하도록 이 설정으로 우회한다.
5. **`next=`류 리다이렉트-후-복귀 쿼리 파라미터에 `basePath`를 수동으로 붙이지 말 것.**
   미들웨어의 `NextResponse.redirect(new URL("/어딘가", ...))`(redirect 대상 자체)는
   `basePath`를 직접 붙여야 하지만, 그 쿼리 파라미터 값이 나중에 `redirect()`나
   `router.push()`(둘 다 `next/navigation`)로 들어간다면 그쪽이 `next/link`처럼
   `basePath`를 스스로 붙인다 — 여기서도 수동으로 붙이면 두 번 겹쳐서
   (`/프로젝트이름/프로젝트이름/...`) 404가 난다. currifair의 게이트→기수선택
   리다이렉트 체인에서 실제로 겪은 버그(`mshs-curriculm`의 `docs/DECISIONS.md` #30).
6. 도메인이 아직 안 붙어 있는 첫 서브패스 프로젝트라면, 루트 포털 Vercel 프로젝트의
   Settings → Domains에 `ms.hs.kr`을 연결하고 가비아 DNS를 Vercel이 알려주는 값으로
   바꾼다. **이 문서에 박힌 값을 쓰지 말고 그때그때 Vercel 대시보드가 보여주는 실제
   값을 쓴다** — 시기/프로젝트마다 달라질 수 있다고 Vercel 문서에 명시돼 있다.

## 반드시 지킬 것

- **실제 코드 저장소(특히 private/민감 정보 있는 레포)를 그대로 Public Pages로 켜지
  않는다.** 공개용 정적 페이지만 담은 폴더(방법 A)나 별도 레포(방법 B)를 쓴다.
- **서브패스 레포에는 `CNAME` 파일을 절대 넣지 않는다.** 자동 전파가 이 조건에
  의존한다.
- **기본값은 `ms.hs.kr` 루트 홈페이지(`index.html`)에 새 서브패스를 링크하지 않는
  것이다.** 루트 페이지는 프로젝트를 자동으로 나열하는 디렉터리가 아니라 학교 신원
  확인용 미니멀 페이지로 유지하기로 했다(`CLAUDE.md`의 "루트 페이지 정책" 참고).
  개인정보 처리방침류는 이 규칙에 예외가 없다 — 항상 비노출.
  프로젝트 소개는 **사용자가 명시적으로 노출을 요청했을 때만** 예외적으로 추가한다
  (현재 예: 선택과목 박람회/currifair). 요청이 없으면 서브패스 URL은 필요한 곳(에듀집
  제출 양식, 학운위 서류 등)에 직접 적어 넣고 홈페이지는 건드리지 않는다.
- 경로가 곧 폴더/레포 이름이므로 이름은 신중하게 정한다(공백·특수문자 금지, 나중에
  바꾸면 예전 URL이 깨짐).

## 관련 문서

- [`CLAUDE.md`](CLAUDE.md) — 이 레포 자체의 구조, 배포, 루트 페이지 정책, 색상 팔레트 출처
- `~/.claude/skills/eduzip/SKILL.md` — 개인정보 처리방침·에듀집 제출 서류를 작성할 때
  이 문서의 방법 A를 기본으로 쓴다
