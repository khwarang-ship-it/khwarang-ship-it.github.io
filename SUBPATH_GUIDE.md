# ms.hs.kr 하위 경로 만들기 — 다른 프로젝트용 가이드

다른 프로젝트(레포)에서 `ms.hs.kr/무언가/` 형태의 공개 페이지가 필요할 때 참고하는 문서다.
개인정보 처리방침, 학운위 제출용 공개 문서, 프로젝트 소개 페이지 등이 여기 해당한다.

## 왜 이게 되는가

이 레포(`khwarang-ship-it/khwarang-ship-it.github.io`)는 이름이 정확히
`계정이름.github.io`라서 GitHub Pages가 이 레포를 **유저 사이트**로 취급하고, 여기 걸린
커스텀 도메인(`ms.hs.kr`, `CNAME` 파일)이 **같은 계정의 다른 레포에도 자동으로
전파**된다. 다른 레포가 자기만의 커스텀 도메인을 따로 설정하지 않으면, GitHub Pages를
켜는 순간 `ms.hs.kr/그레포이름/`으로 자동으로 뜬다. (자세한 배경은 이 레포의
[`CLAUDE.md`](CLAUDE.md) 참고.)

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

## 반드시 지킬 것

- **실제 코드 저장소(특히 private/민감 정보 있는 레포)를 그대로 Public Pages로 켜지
  않는다.** 공개용 정적 페이지만 담은 폴더(방법 A)나 별도 레포(방법 B)를 쓴다.
- **서브패스 레포에는 `CNAME` 파일을 절대 넣지 않는다.** 자동 전파가 이 조건에
  의존한다.
- **`ms.hs.kr` 루트 홈페이지(`index.html`)는 여기서 만든 서브패스를 링크하지
  않는다.** 이건 실수가 아니라 의도된 정책이다 — 루트 페이지는 프로젝트 디렉터리가
  아니라 학교 신원 확인용 미니멀 페이지로 유지하기로 했다(`CLAUDE.md`의 "루트 페이지
  정책" 참고). 이 서브패스 URL이 필요한 곳(에듀집 제출 양식, 학운위 서류 등)에는
  주소를 직접 적어 넣는다 — 홈페이지에 카드나 메뉴를 추가하지 않는다.
- 경로가 곧 폴더/레포 이름이므로 이름은 신중하게 정한다(공백·특수문자 금지, 나중에
  바꾸면 예전 URL이 깨짐).

## 관련 문서

- [`CLAUDE.md`](CLAUDE.md) — 이 레포 자체의 구조, 배포, 루트 페이지 정책, 색상 팔레트 출처
- `~/.claude/skills/eduzip/SKILL.md` — 개인정보 처리방침·에듀집 제출 서류를 작성할 때
  이 문서의 방법 A를 기본으로 쓴다
