# 에메랄드 드래곤 한국어판 — 배포 페이지

GitHub Pages 에 그대로 올리면 되는 정적 페이지다.
**신고·집계만 PHP 라 별도 서버로 간다** → `../server/설치.md`

```
web-patcher/
├── index.html        탭 5개 (프로젝트 · 릴리스 · 버그 신고 · 집계 · 한글패치 목록)
├── patcher.html      웹 패처 (판별 / 적용 / 제보)
├── style.css
├── links.json        ← **서버 주소는 여기 한 곳에서만 관리한다**
├── index.pre-redesign.html   이전 소개 페이지 백업
├── style.pre-redesign.css    이전 스타일 백업
├── patch/
│   ├── manifest.json          버전 · 원본 후보 · 결과 해시
│   ├── emdr-ko-v0.6-rev1.bin.gz
│   └── emdr-ko-v0.6-rev0.bin.gz
├── xdelta/
│   ├── emdr-ko-v0.6-rev1.xdelta
│   ├── emdr-ko-v0.6-rev0.xdelta
│   └── CHECKSUMS.txt
├── syscard/
│   └── syscard3_KO.pce
└── img/              스크린샷 · 보존 자료 썸네일 · 공유 이미지
```

전체 5 MB 남짓이라 Pages 용량 제한에 여유가 있다.

---

## 탭 구조

`index.html` 한 파일 안에 `<main data-panel="...">` 다섯 개가 들어 있고
`[data-tab]` 단추가 `hidden` 을 토글한다. 페이지를 나누지 않은 이유는
공통 헤더·CSS·집계 코드를 한 벌만 두려는 것이다.

주소로 바로 열 수 있다 — `index.html#release`, `index.html#board`, `index.html#stats`.
패처 페이지의 상단 메뉴가 이 주소를 쓴다.

**신고·집계는 그 탭을 열 때만 서버를 부른다.** 소개만 보고 갈 사람에게 요청을 만들지 않는다.

---

## links.json — 서버를 켜고 끄는 곳

```json
"board":     { "url": "" },
"stats_api": ""
```

**비어 있으면 그 탭이 "준비 중" 으로 뜨고 외부 요청이 0 이다.**
주소를 채우고 이 파일만 다시 올리면 즉시 반영된다. HTML 은 안 건드려도 된다.

`version` 값이 페이지 표시 버전과 집계 버전을 정한다 — 여기만 고치면 된다.

---

## 이미지 교체

`img/` 의 파일명을 그대로 두고 내용만 바꾸면 된다.

| 파일 | 쓰이는 곳 | 상태 |
|---|---|---|
| `shot-title.png` | 히어로 CRT · 파비콘 · 목록 탭 썸네일 | ✅ 실제 한글 타이틀 (1024², 4× 확대) |
| `bg-crystal.jpg` | 히어로 배경 패럴렉스 · 패처 헤더 | ✅ 타이틀 화면 하단 무늬에서 뽑음 |
| `shot-dialog.png` | 스크린샷 격자 (큰 칸) | ⬜ 플레이스홀더 |
| `shot-battle.png` | 스크린샷 격자 | ⬜ 플레이스홀더 |
| `shot-menu.png` | 스크린샷 격자 | ⬜ 플레이스홀더 |
| `og.png` | 공유 미리보기 | ✅ 1200×630 |

- 남은 3장은 가로 **512 px 이상** 권장 (원본 화면이 256 px 이므로 2배가 깔끔하다)
- 세 장의 **가로세로 비를 맞추면** 격자가 고르게 보인다
- `image-rendering: pixelated` 가 걸려 있어 확대해도 픽셀이 뭉개지지 않는다

### 타이틀·배경을 다시 만들려면

원본은 `emerald-title-screen.png`(256², 실기 캡처)다. 그것만 갈아 끼우고 아래를 다시 돌리면 된다.

```bash
cd img
# 타이틀 — 픽셀을 살려야 하므로 반드시 point 필터
magick emerald-title-screen.png -filter point -resize 400% -strip PNG8:shot-title.png
# 배경 — 글자가 없는 하단 26px 띠만 잘라 늘리고 어둡게
magick emerald-title-screen.png -crop 256x26+0+228 +repage \
  -resize 1600x900! -blur 0x10 -modulate 46,62,112 \
  -fill '#06120d' -colorize 42% -quality 80 bg-crystal.jpg
```

`-crop`의 `+0+228` 은 **크레딧 글자 아래**를 집는 값이다. 캡처가 바뀌면 이 숫자도 바뀐다 —
자른 뒤 글자가 섞이지 않았는지 눈으로 볼 것.

### 그 밖

`archive-front.jpg` · `archive-back.jpg` · `archive-map.jpg` 는 패키지·지도 스캔이고
저해상도(192 px) + 출처 링크 + 크레딧 문구를 붙여 뒀다.

`video-cover.jpg` 는 **어느 파일에서도 참조하지 않는다** (영상을 iframe 으로 바꾸며 남은 것).
`emerald-title-screen.png` 는 화면에 직접 안 나오지만 위 두 이미지의 **원본**이니 지우지 말 것.

---

## 왜 웹 패처가 xdelta 를 안 쓰나

브라우저에서 xdelta 를 풀려면 wasm 디코더가 필요하다. 그런데 이 패치는
**바뀌는 섹터가 4% 남짓**이라, "어느 섹터를 무엇으로 바꿔라" 목록만 있으면
순수 JS 로 끝난다. 의존성 하나를 통째로 덜었다.

xdelta 는 xdelta 대로 따로 배포한다 — 그쪽을 선호하는 사용자가 있다.

### 패치 형식

```
patch/*.bin.gz    [4B 섹터번호][2048B 데이터] 를 이어 붙인 것 (gzip)
```

섹터 번호는 **cooked 데이터 트랙 기준**(0 = 데이터 트랙 첫 섹터)이다.
패처는 raw 이미지의 `(3596 + n) * 2352 + 16` 자리에 쓰고 EDC·ECC 를 다시 만든다.

---

## 리비전 두 종을 모두 지원한다

| | 바뀌는 섹터 | xdelta |
|---|---:|---:|
| Rev 1 | 1,775 | 495 KB |
| 초판 | 2,122 | 712 KB |

두 판은 데이터 트랙에서만 다르고 그것도 1.2%(580 섹터)뿐이다.
오디오 트랙은 **완전히 같다** [측정].

어느 쪽에서 출발하든 결과물은 `de72f2ae…` 로 **바이트까지 같다** — 왕복 검증을 마쳤다.

> emdrtools(영문 패치)가 Rev 1 전용인 것은 **빌드할 때** 이야기다.
> 이미 만들어진 결과물을 **적용**하는 것은 별개다.

---

## 집계가 무엇을 보내나

패처는 **CRC32 를 보내지 않는다.** 그건 파일을 특정하는 값이고, 이 페이지가
"파일이 서버로 가지 않는다" 고 말할 수 있는 근거를 스스로 깎는다.
나가는 것은 판별 결과(`rev0`/`rev1`/`unknown`)와 성공 여부뿐이다.

모르는 덤프를 알려 주는 경로는 패처의 **복사 단추** 하나다 — 사람이 직접 고른다.

그 정보를 받으면 그 덤프를 구해 `make_webpatch.py` 의 `SOURCES` 에 한 줄 더하고
다시 돌리면 된다.

---

## 다시 만들기

```bash
# 1. 한국어판 raw 이미지
python3 build/tools/cooked2raw.py --build \
    "original/…/Emerald Dragon (Japan)(Rev 1).bin" \
    build/cd-img/emdr_02_build.iso  emdr_ko.img

# 2. xdelta (원본마다)
xdelta3 -e -9 -S djw -f -s "<원본>.bin" emdr_ko.img  out.xdelta

# 3. 웹 패처용 섹터 패치 + manifest
python3 build/tools/make_webpatch.py
```

### 반드시 확인할 것

- **xdelta 왕복** — 적용 결과가 `emdr_ko.img` 와 sha256 이 같은가
- **패처의 EDC/ECC** — 원본 섹터로 다시 계산해 바이트가 같은가
  (`fixSector` 를 떼어내 Node 로 돌리면 된다. LBA 3596 / 10000 / 30000 / 52000 / 52674 로 쟀다)

크기만 보고 넘기지 마라. 크기는 검증이 아니다.

---

## 음성 여지

`links.json` 과 `manifest.json` 의 `voice` 가 지금 `null` 이다.
집계 쪽도 `MODES` 에 `voice` 자리를 비워 뒀다.
음성 패치가 생기면 같은 형식으로 채우면 되고, 페이지 구조는 그대로 둬도 된다.

---

## 로컬에서 보기

```bash
python3 -m http.server 8080
```

`file://` 로 열면 `links.json` · `manifest.json` 읽기가 CORS 에 막힌다.
