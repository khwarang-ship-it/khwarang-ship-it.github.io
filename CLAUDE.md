# CLAUDE.md — khwarang-ship-it.github.io (ms.hs.kr)

## 이 저장소가 하는 일

명석고등학교 도메인 `ms.hs.kr`의 GitHub Pages 소스. 저장소 이름이
**반드시** `khwarang-ship-it.github.io`여야 한다 (유저 사이트 이름). 이 이름일 때만
GitHub Pages의 "커스텀 도메인이 형제 프로젝트 레포에 자동 전파" 기능이 동작해서,
새 프로젝트 레포를 만들고 Pages만 켜면 커스텀 도메인 설정 없이
`ms.hs.kr/레포이름/`으로 자동 노출된다. `mshs-site` 같은 일반 프로젝트 레포 이름으로
바꾸면 이 기능이 깨지므로 그 이름은 다시 쓰지 않는다.

- `index.html` — 루트 랜딩 페이지 (아래 참고)
- `CNAME` — `ms.hs.kr` (절대 지우지 말 것 — 지우면 커스텀 도메인이 끊김)
- `assets/logo.png`, `assets/logo.ico` — 명석고 실제 교표. `assets/logo-mark.png`는
  `logo.png`에서 흰 배경을 투명 처리하고 여백을 잘라낸 파생 이미지 (재생성 방법은 아래)
- `ms-record-linter/privacy.html` — 「명석한 생기부」 개인정보 처리방침. 루트 페이지에서
  **의도적으로 링크하지 않음** (아래 정책 참고)

## 배포

빌드 없음. `main`에 push하면 GitHub Pages가 그대로 서빙한다.

```bash
git push origin main
# 배포 확인 (보통 30~60초):
gh api repos/khwarang-ship-it/khwarang-ship-it.github.io/pages/builds/latest --jq .status
```

## 루트 페이지(index.html) 정책 — 중요

**루트 페이지는 학교 소개/신원 확인용 미니멀 페이지다. 프로젝트 디렉터리가 아니다.**

- 이 도메인 아래에 어떤 프로젝트가 있는지, 무엇을 하는지 **루트 페이지에 나열하지 않는다.**
- 각 프로젝트의 개인정보 처리방침 링크도 **루트 페이지에 노출하지 않는다.**
  (필요하면 그 프로젝트의 서브패스 안에서만 존재)
- 새 프로젝트를 추가해도 `index.html`에 카드나 링크를 자동으로 추가하지 않는다.
  (사용자가 명시적으로 다시 요청하기 전까지)

이전에 "교내 도구 모음" 형태의 벤토 그리드 포털(생기부 프로그램 소개 카드 포함)로
한 번 만들었다가, 사용자가 프로젝트 노출을 원치 않는다고 해서 지금의 미니멀 버전으로
교체한 이력이 있다 (커밋 `74d5c75` → `3a16865`). 다시 "포털/허브"로 되돌리자는 요청이
없는 한 이 미니멀 구조를 유지한다.

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
