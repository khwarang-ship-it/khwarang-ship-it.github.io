# CLAUDE.md — khwarang-ship-it.github.io (ms.hs.kr)

## 이 저장소가 하는 일

명석고등학교 도메인 `ms.hs.kr`의 GitHub Pages 소스. 저장소 이름이
**반드시** `khwarang-ship-it.github.io`여야 한다 (유저 사이트 이름). 이 이름일 때만
GitHub Pages의 "커스텀 도메인이 형제 프로젝트 레포에 자동 전파" 기능이 동작해서,
새 프로젝트 레포를 만들고 Pages만 켜면 커스텀 도메인 설정 없이
`ms.hs.kr/레포이름/`으로 자동 노출된다. `mshs-site` 같은 일반 프로젝트 레포 이름으로
바꾸면 이 기능이 깨지므로 그 이름은 다시 쓰지 않는다.

- `index.html` — 루트 랜딩 페이지 (아래 참고)
- `CNAME` — `ms.hs.kr`. 호스팅이 Vercel로 이전 완료된 뒤로는 실제 서빙에 쓰이지
  않는다(도메인은 이제 Vercel Domains 설정이 담당). GitHub Pages 쪽 레거시 설정이라
  지워도 무방하지만, 굳이 지울 이유도 없어 남겨둠.
- `vercel.json` — `/currifair` 서브패스를 currifair의 Vercel 배포로 넘기는 rewrite 설정.
  `"trailingSlash": false`도 여기서 지정 — 이유는 아래 "겪은 버그" 참고. `redirects`에는
  옛 서브도메인 `currifair.ms.hs.kr`을 `ms.hs.kr/currifair/*`로 경로 보존 리다이렉트하는
  규칙도 있다(Host 헤더 조건부 — 아래 "옛 서브도메인 리다이렉트" 참고).
- `assets/logo.png`, `assets/logo.ico` — 명석고 실제 교표. `assets/logo-mark.png`는
  `logo.png`에서 흰 배경을 투명 처리하고 여백을 잘라낸 파생 이미지 (재생성 방법은 아래)
- `ms-record-linter/privacy.html` — 「명석한 생기부」 개인정보 처리방침. 루트 페이지에서
  **의도적으로 링크하지 않음** (아래 정책 참고)

## 배포

빌드 없음. `main`에 push하면 Vercel이 자동 배포한다(GitHub App 연동, 대시보드 조작
불필요). 배포 상태는 Vercel 대시보드에서 확인하거나, 아래처럼 배포된 페이지를 직접
찔러서 확인한다.

```bash
git push origin main
# 30~60초 후 반영 확인:
curl -sI https://ms.hs.kr/ | head -5
```

## 호스팅: GitHub Pages → Vercel 이전 완료 (2026-08-10)

`mshs-curriculm`(선택과목 박람회, "currifair")을 `ms.hs.kr/currifair/`에 붙이기 위해
호스팅 주체를 GitHub Pages에서 Vercel로 옮겼다. `mshs-curriculm`은 로그인 게이트·AI
상담·실시간 데이터 갱신이 있는 Next.js **서버** 앱이라 정적 파일만 서빙하는 GitHub
Pages로는 애초에 붙일 수 없었다(경로 기반 프록시 기능이 없음).

**구조**: 이 레포(루트 포털, 정적)와 `mshs-curriculm`(currifair, Next.js)이 각각 별도
Vercel 프로젝트(둘 다 GitHub 연동, `main` push 시 자동 배포)로 떠 있고, 이 레포의
`vercel.json`이 `/currifair`, `/currifair/:path*` 요청만 currifair의 Vercel 배포
주소(`https://mshs-curriculm.vercel.app`)로 rewrite한다. 그 외 경로는 전부 이 레포가
정적으로 직접 서빙. `ms.hs.kr`/`www.ms.hs.kr`은 가비아 DNS가 GitHub Pages A레코드에서
Vercel 값으로 바뀌어 있고(apex는 www로 308 canonical redirect, Vercel 기본 동작), 루트
포털 Vercel 프로젝트의 Domains에 연결돼 있다.

**실제 배포하면서 터진 버그 2건** (둘 다 수정·배포 완료):
1. **`/currifair/`처럼 트레일링 슬래시가 붙은 모든 경로가 404.** currifair 앱이
   슬래시 붙은 요청을 자체적으로 308 리다이렉트하는데, 그 리다이렉트가 외부 오리진
   rewrite 프록시를 그대로 못 넘어가고 루트 프로젝트에서 `X-Vercel-Error: NOT_FOUND`로
   튕겼다. 이 레포의 `vercel.json`에 `"trailingSlash": false`를 추가해 Vercel 엣지
   단계에서 슬래시를 먼저 정리하도록 해서 해결(이 레포 쪽만 수정, currifair 코드는
   안 건드림).
2. **게이트 PIN 입력 → 학년 선택을 마치면 404.** `mshs-curriculm`의 `proxy.ts`가
   `next=` 리다이렉트 대상 쿼리 파라미터에 `basePath`(`/currifair`)를 직접 붙였는데,
   그 값이 나중에 `redirect()`/`router.push()`(`next/navigation`, `next/link`처럼
   `basePath`를 스스로 붙임)로 들어가면서 게이트→기수선택 단계마다 겹쳐 붙어
   (`/currifair/currifair/currifair/...`) 존재하지 않는 경로에 도달했다.
   `mshs-curriculm` 레포의 `proxy.ts`에서 `next=` 값의 수동 접두를 제거해 해결.
3. **`ms.hs.kr/currifair/` 홈 화면이 게이트 없이 그냥 열림** (세부 메뉴로 들어가야만
   PIN이 뜸). `proxy.ts`의 negative-lookahead 정규식 matcher가 `basePath` 루트
   경로에서 안 걸리는 버그 — Next.js/Vercel 쪽 알려진 이슈군(vercel/next.js#86701,
   #86269)과 일치. matcher를 `/:path*`(전체 매칭)로 단순화하고 제외 로직을 함수
   본문으로 옮겨서 해결.

   위 3건 자세한 내용은 `mshs-curriculm`의 `docs/DECISIONS.md` #30 참고.

새로 서브패스를 붙이고 싶으면 `SUBPATH_GUIDE.md`의 방법 A(이 레포 안 폴더, 정적) 또는
방법 C(Vercel rewrite, 서버 앱)를 쓴다. **방법 B(형제 GitHub Pages 레포 자동 전파)는
DNS가 더 이상 GitHub Pages를 안 보므로 이제 동작하지 않는다.**

## 옛 서브도메인 리다이렉트 (`currifair.ms.hs.kr` → `ms.hs.kr/currifair/*`)

currifair를 처음 검토할 때 서브도메인 방식(`currifair.ms.hs.kr`)도 고려해서 가비아에
CNAME을 미리 걸어뒀던 적이 있다. 최종적으로 경로 방식(`ms.hs.kr/currifair/`)으로 결정한
뒤에도 그 DNS 레코드는 남아있었고, 옛 링크가 안 죽게 리다이렉트로 연결했다.

**Vercel 대시보드의 "Redirect to Another Domain" 도메인 설정은 쓰지 않는다** —
도메인만 받고 경로(`/currifair`)는 못 받아서(슬래시 들어간 값을 넣으면 Save가
비활성화됨) `ms.hs.kr/currifair`처럼 경로가 있는 대상으로는 애초에 설정이 안 된다.
대신 `currifair.ms.hs.kr`을 이 프로젝트에 **"Connect to an environment" (Production)**
로 붙이고, `vercel.json`의 `redirects`에 `has: [{ type: "host", value:
"currifair.ms.hs.kr" }]` 조건부 규칙으로 직접 처리한다 — 이러면 경로까지 그대로
보존해서(`currifair.ms.hs.kr/majors` → `ms.hs.kr/currifair/majors`) 308 리다이렉트된다.
`/`(루트)와 `/:path*`를 따로 나눈 이유는 앞서 겪은 것과 같은 이유(와일드카드가 세그먼트
0개를 못 잡음).

## 루트 페이지(index.html) 정책 — 중요

**루트 페이지는 학교 소개/신원 확인용 미니멀 페이지다. 프로젝트를 자동으로 나열하는
디렉터리가 아니다. 기본값은 "노출 안 함"이고, 노출은 매번 프로젝트별로 사용자가
명시적으로 요청해야만 한다.**

- 새 프로젝트를 추가해도 `index.html`에 카드나 링크를 **자동으로 추가하지 않는다.**
  사용자가 그 프로젝트를 명시적으로 노출해달라고 요청했을 때만 추가한다.
- 각 프로젝트의 개인정보 처리방침 링크는 **원칙적으로 루트 페이지에 노출하지 않는다**
  (필요하면 그 프로젝트의 서브패스 안에서만 존재) — 이건 프로젝트 노출 여부와 별개로
  항상 지키는 규칙이다.
- 현재 노출 중인 프로젝트: **선택과목 박람회(currifair)** — 사용자가 명시적으로
  노출을 요청해서 `.project` 섹션과 nav/footer 링크로 추가함(`https://ms.hs.kr/currifair/`).
  명석한 생기부(ms-record-linter)는 노출 요청이 없어서 계속 비노출 상태.

이전에 "교내 도구 모음" 형태의 벤토 그리드 포털(생기부 프로그램 소개 카드 포함)로
한 번 만들었다가, 사용자가 프로젝트 노출을 원치 않는다고 해서 미니멀 버전으로 교체한
이력이 있다(커밋 `74d5c75` → `3a16865`). 이후 currifair만 예외적으로 노출 요청이 들어옴
— 즉 "미니멀 유지"가 기본값이고 노출은 예외라는 원칙 자체는 안 바뀌었다. 새 프로젝트를
만들 때 "그것도 홈페이지에 보여줄지"는 항상 먼저 물어보고, 답이 없으면 안 보여주는
쪽으로 판단한다.

## 기술 스택 — 의도적 선택

- **빌드 도구 없음, 런타임 JS 의존성 없음.** 순수 `index.html` 하나.
- **Tailwind CDN 안 씀.** Play CDN은 ~120KB 실시간 컴파일 JS로 프로덕션 비권장 +
  FOUC 발생. 글래스모피즘/블롭/틸트 같은 효과는 어차피 커스텀 CSS가 필요해서
  Tailwind를 얹어도 이득이 없음. 손으로 쓴 CSS(`<style>` 내부)만 사용.
- **AOS.js, Lucide JS 번들 안 씀.** 스크롤 reveal은 `IntersectionObserver` 10줄,
  아이콘은 필요한 것만 Lucide SVG path를 인라인 `<svg><use>`로 복사해서 사용.
- **외부 의존성은 폰트 2개만 허용**: Pretendard Variable(본문), Gowun Batang(제목).
  jsdelivr/Google Fonts CDN에서 로드. 한글 프리미엄 룩을 위한 유일한 예외.
- **다크모드**: `prefers-color-scheme` + `:root[data-theme]` 토큰 오버라이드 패턴.
  헤더의 토글 버튼이 `localStorage`에 저장.

## 색상 팔레트 — 출처

`assets/logo.png`에서 실제 픽셀 색상을 샘플링해서 뽑음 (임의로 정한 색 아님):

| 이름 | hex | 용도 |
|---|---|---|
| 남색 (교표) | `#171c61` | 라이트 모드 accent (링크/버튼) |
| 하늘색 (교표) | `#28a7e1` | 히어로 글로우 |
| 연두 (교표) | `#8ec31f` | 히어로 글로우 |
| 주황 (교표) | `#d14e13` | 유일한 포인트 컬러 (`--pop`, 강조 밑줄·경고 아이콘) |

다크모드는 위 색을 밝게 조정한 별도 토큰(`--accent: #5cc2f0` 등)을 씀 — CSS 파일
상단 `:root` / `@media (prefers-color-scheme: dark)` / `:root[data-theme]` 블록 참고.

로고 파생 이미지 재생성이 필요하면 (예: 원본 logo.png가 바뀌었을 때):

```python
from PIL import Image
im = Image.open('assets/logo.png').convert('RGBA')
data = im.getdata()
new = [(r,g,b,0) if (r>248 and g>248 and b>248) else (r,g,b,255) for r,g,b,a in data]
im.putdata(new)
im = im.crop(im.getbbox())
im.resize((1440, int(1440*im.height/im.width))).save('assets/logo-mark.png', optimize=True)
```

## 검증 방법

빌드가 없으니 브라우저에서 직접 확인하면 된다. 로컬 확인 시 `python -m http.server`로
띄우고(파일을 `file://`로 열면 fetch/폰트 프리커넥트가 깨짐), 데스크톱(1280)/태블릿
(768)/모바일(375) 3구간 + 라이트/다크 2테마 + `prefers-reduced-motion` 확인.
