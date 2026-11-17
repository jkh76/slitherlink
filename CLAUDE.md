# CLAUDE.md

슬리더링크(울타리 치기) 웹 게임. `index.html` 한 파일(HTML·CSS·JS)로 동작하고 GitHub Pages로 플레이한다.

## 작업 규칙

- 푸시하기 전에 수정한 HTML을 먼저 사용자에게 보여 주고 확인받는다.
- 푸시할 때마다:
  - `index.html`의 `VERSION`을 0.1씩 올린다 (예: 0.2 → 0.3).
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
  - `makePuzzle(d, seed)`: 구멍 없는 영역을 키워 정답 고리를 만들고, 숫자를 지워 가며 하·중은 추론만으로 풀리는지, 상은 해가 하나인지 확인한다. 시드가 같으면 같은 문제.
- 기록: 난이도별 최단 클리어 시간만 `localStorage`의 `slither_best_<easy|mid|hard>`에 저장.
- `DIFFS`: 난이도별 판 크기, 남길 숫자 비율, 검사 방식.
- 그 아래: 렌더링(SVG, viewBox가 칸 단위), 입력(펜/X 드래그, 오른쪽 클릭), 힌트, 답안보기, 타이머, 클리어 연출, 설정(localStorage).

## 테스트

```bash
sed -n '/CORE-START/,/CORE-END/p' index.html > /tmp/core.js
# /tmp/core.js 끝에 makePuzzle / countSolutions 호출을 붙여 node로 실행
```
상(10×10) 생성은 PC 기준 1~2초.
