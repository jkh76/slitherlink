# CLAUDE.md

슬리더링크(울타리 치기) 웹 게임. `index.html` 한 파일(HTML·CSS·JS)로 동작하고 GitHub Pages로 플레이한다.

## HTML 파일 이름 규칙

- 수정본이나 새 버전을 만들어 경훈에게 보여 줄 때는 파일 이름을 `슬리더링크.html`로 한다.
- 깃허브에 푸시할 때만 같은 내용을 `index.html`로 올린다. 저장소에는 `슬리더링크.html`을 따로 올리지 않는다.

## 작업 규칙

- 푸시하기 전에 수정한 HTML을 먼저 사용자에게 보여 주고 확인받는다.
- 푸시할 때마다:
  - `index.html`의 `VERSION`을 0.1씩 올린다 (예: 1.0 → 1.1). 사용자가 버전을 지정하면 그 값을 쓴다.
  - `BUILD_DATE`를 푸시하는 날짜(`yyyy-mm-dd`, 한국 시간)로 바꾼다. 화면 맨 아래 가운데에 `v버전 · 날짜`로 표시된다.
  - claude.ai 아티팩트도 같은 HTML과 아이콘 파일로 업데이트한다 (https://claude.ai/artifact/A1Ef3Q4BobGwFwwCWmAbAQ).
  - README.md의 버전 표에 한 줄 추가한다.
- 외부 라이브러리·음원 파일 없이 단일 HTML을 유지한다(아이콘 PNG/ICO만 따로 둔다). 효과음은 Web Audio로 합성한다.
- UI 문구는 한국어.

## 코드 구조 (`index.html`의 `<script>`)

- `// CORE-START` ~ `// CORE-END`: DOM과 무관한 퍼즐 로직. node로 따로 떼어 테스트할 수 있다.
  - `makeGeom(W,H)`: 변·점·칸 인덱스. 변 상태 배열은 `1` 선, `-1` X, `0` 미정.
  - `propagate`: 숫자 규칙, 점의 0·2개 규칙, 고리 조기 닫힘 금지 규칙으로 확정되는 변을 채운다.
  - `verify`: 숫자 일치 + 하나의 닫힌 고리인지 검사 (클리어 판정).
  - `countSolutions` / `isUnique`: 백트래킹으로 해의 개수 확인.
  - `makePuzzle(d, seed)`: 구멍 없는 영역을 키워 정답 고리를 만들고, 숫자를 지워 가며 하는 추론만으로 풀리는지, 상은 해가 하나인지 확인한다. 시드가 같으면 같은 문제.
- 기록: `localStorage`의 `slither_best_<key>`(최단 시간, 초)와 `slither_clears_<key>`(클리어 횟수). key는 `easy`(5×5 하) `5h`(5×5 상) `mid`(7×7 하) `7h`(7×7 상) `10e`(10×10 하) `hard`(10×10 상). 답안보기로 끝낸 판은 저장하지 않음.
- 기기 간 기록 공유 (v1.8): Firebase Realtime Database를 **SDK 없이 REST API**로 호출한다 (`FB` 상수에 API 키·DB 주소, 둘 다 공개돼도 되는 값).
  - 익명 로그인(Identity Toolkit `accounts:signUp`) 토큰을 `slither_fb_auth`에 보관하고 만료되면 refresh token으로 갱신.
  - 데이터: `sync/<8자리 코드>` = `{best:{key:초}, clears:{key:횟수}, updated}`. 코드 문자는 I·O·0·1을 뺀 32자.
  - 클리어하면 `queueSync` → `slither_sync_q`에 쌓고 `flushSync`로 전송(횟수는 서버 increment, 최고 기록은 더 짧을 때만). 오프라인이면 다음에 보냄.
  - 앱 시작·기록 화면 열 때 `syncNow`로 받아와 반영(최고 기록은 짧은 쪽, 횟수는 서버 값).
  - 코드 입력(`joinCode`) 시 이 기기 기록을 합침: 최고 기록은 짧은 쪽, 횟수는 더함(같은 코드 재연결이면 이미 올린 만큼 뺌).
  - 보안 규칙(Firebase 콘솔): `sync/$code`만 로그인한 사용자가 읽기·쓰기, 코드 형식·값 형식 검사, 다른 필드 금지.
  - claude.ai 미리보기·아티팩트 창은 외부 연결이 막혀 공유가 안 될 수 있음 → 실제 확인은 Vercel 주소에서.
- `DIFFS`: 난이도 6개(판 크기 × 하·상). `d = 크기순번*2 + (하 0 / 상 1)`. 하는 `prop`(추론만으로 풀림, 숫자 많음), 상은 `search`(해가 하나, 숫자 적음). `ratio`는 남길 숫자 비율의 목표치.
- 그 아래: 렌더링(SVG, viewBox가 칸 단위), 입력(펜/X 드래그, 오른쪽 클릭), 힌트, 답안보기, 타이머, 클리어 연출, 설정(localStorage).

## 테스트

```bash
sed -n '/CORE-START/,/CORE-END/p' index.html > /tmp/core.js
# /tmp/core.js 끝에 makePuzzle / countSolutions 호출을 붙여 node로 실행
```
상(10×10) 생성은 PC 기준 1~2초.
