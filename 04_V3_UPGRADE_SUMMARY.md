# Bowerbird PRO V3 업그레이드 완료 요약

> V2 → V3 업그레이드: 6개 Phase, 총 25개 항목 구현 완료
> 완료일: 2026-03-27

---

## Phase 1: 긴급 버그 수정 — 크래시 방지

| 항목 | 수정 파일 | 내용 |
|------|----------|------|
| 1-1 | `js/analysis/statistics.js` | `Math.min(...data)` / `Math.max(...data)` 스택 오버플로 → 루프 기반 min/max 계산으로 교체. 중간 배열 생성도 제거하여 단일 패스 stdDev 계산 |
| 1-2 | `js/main.js`, `js/utils/helpers.js` | csvData 50만행 제한(`MAX_CSV_ROWS`), 초과 시 자동 분할 저장(chunk export), `URL.revokeObjectURL()` 메모리 누수 방지 |

---

## Phase 2: 데이터 처리 핵심 개선

| 항목 | 수정 파일 | 내용 |
|------|----------|------|
| 2-1 | `js/main.js` | `Infinity`, `-Infinity` 값 필터링, 센서 수와 수신 값 개수 불일치 시 경고 로그 |
| 2-2 | `js/main.js` | 첫 데이터 라인 수신 시 콤마 개수로 센서 수 자동 감지, UI 반영 및 알림 |
| 2-3 | `js/analysis/analysis-ui.js`, `js/visualization/chart-handler.js` | `getAllData()` 함수 추가, 분석 UI에 "전체/뷰포트" 토글, 전체 통계와 구간 통계 분리 |
| 2-4 | `js/visualization/chart-handler.js` | `fullDataStore` 별도 저장소, 차트 데이터 10만 포인트 상한 다운샘플링, `getVisibleData()`에 바이너리 서치 적용 |
| 2-5 | `js/main.js`, `js/visualization/chart-handler.js` | `performance.now()` 고해상도 타이머, MCU `TIMESTAMP,` 접두사 파싱, CSV에 경과시간(초) 컬럼 추가 |

---

## Phase 3: 기능 확장 — 데이터 관리

| 항목 | 수정 파일 | 내용 |
|------|----------|------|
| 3-1 | `js/main.js`, `app.html` | CSV 불러오기(Import) 버튼 추가, 헤더에서 센서 이름 자동 추출, Bowerbird CSV 및 일반 CSV 지원 |
| 3-2 | `js/utils/helpers.js`, `js/main.js` | IndexedDB 기반 세션 자동 저장(5초 간격), 앱 시작 시 미완료 세션 복원 다이얼로그 |
| 3-3 | `js/visualization/chart-handler.js`, `app.html` | 차트 PNG 이미지 내보내기 버튼(`chart.toBase64Image()`), 파일명에 날짜/시간 자동 포함 |
| 3-4 | `js/analysis/processing.js`, `js/analysis/analysis-ui.js`, `app.html` | IQR 기반 이상치 감지 함수 `detectOutliers()`, "이상치 제거" 토글, 차트에 빨간색 마커 표시 |

---

## Phase 4: UX 개선

| 항목 | 수정 파일 | 내용 |
|------|----------|------|
| 4-1 | `app.html` | Baud rate 8단계 확장: 4800, 9600, 19200, 38400, 57600, 115200, 230400, 460800 |
| 4-2 | `app.html`, `js/main.js` | 데이터 구분자 선택 드롭다운 (콤마, 탭, 공백, 세미콜론), `processDataLine()`에서 동적 파싱 |
| 4-3 | `js/main.js` | 키보드 단축키: `Space` 일시정지, `Ctrl+S` 시작, `Ctrl+Q` 중지, `Ctrl+E` CSV 내보내기 |
| 4-4 | `js/visualization/chart-handler.js`, `app.html`, `js/main.js` | 센서별 개별 Y축 체크박스, `setIndividualYAxis()` 함수, 축 색상 센서 매칭 |
| 4-5 | `js/visualization/chart-handler.js`, `app.html`, `js/main.js` | Y-min/Y-max 입력 필드, `setYAxisRange()` 함수, 빈 값은 자동 스케일 |
| 4-6 | `app.html`, `css/style.css`, `js/main.js` | 연결 상태 LED (초록/빨강/노랑), 데이터 수신 시 펄스 애니메이션, 샘플링 레이트(pts/s) 실시간 표시 |
| 4-7 | `js/utils/helpers.js`, `js/main.js`, `css/style.css` | `showToast()` 함수 (info/warn/error), `alert()` 호출 전부 교체, 자동 사라짐 애니메이션 |
| 4-8 | `app.html`, `js/utils/helpers.js` | 데이터 로그 검색 입력 필드, 실시간 텍스트 필터링 `filterLogLines()`, 개별 `<div>` 기반 로그 라인 |

---

## Phase 5: 통신 기능 확장

| 항목 | 수정 파일 | 내용 |
|------|----------|------|
| 5-1 | `js/core/serial.js`, `js/core/ble.js`, `app.html`, `js/main.js` | Serial TX (`writable.getWriter()`), BLE TX (`6e400003-...` characteristic), 명령 입력 UI + 전송 버튼 |
| 5-2 | `js/core/ble.js`, `app.html`, `js/main.js` | BLE 커스텀 UUID 입력 필드 (Service UUID, RX UUID), 빈 값이면 기본 Nordic UART UUID 사용 |
| 5-3 | `js/analysis/analysis-ui.js`, `js/analysis/processing.js`, `app.html` | 스무딩 윈도우 슬라이더 (3~51, 홀수), `calculateSavitzkyGolay()` 가변 윈도우 지원 |
| 5-4 | `js/analysis/regression.js`, `js/analysis/analysis-ui.js`, `app.html` | 다항식 차수 선택 드롭다운 (2~5차), `calculatePolynomialRegression()` N차 일반화, 동적 수식 문자열 |

---

## Phase 6: 고급 기능 및 마무리

| 항목 | 수정 파일 | 내용 |
|------|----------|------|
| 6-1 | `js/analysis/classification.js` | k-NN 분류기 구현: `classifyState()`, `setTrainingData()`, `parseTrainingCSV()`, 유클리드 거리 기반 |
| 6-2 | `js/utils/alarm.js` (신규), `app.html`, `js/main.js` | 센서별 상한/하한 알람, 임계값 초과 시 토스트 경고 + 브라우저 Notification API, 알람 이력 로그 |
| 6-3 | `js/main.js` | 로깅 중 센서 이름 변경 허용, 센서 수 변경 잠금 + 토스트 경고 |
| 6-4 | `js/main.js` | 샘플링 레이트 30초 주기 로그 기록, 평균 수신률 계산 |
| 6-5 | `js/main.js` | Serial + BLE 동시 연결 지원, `toggleConnection()`이 선택된 타입만 토글 |
| 6-6 | `translations/ko.json`, `translations/en.json` | 18개+ 번역 키 추가 (구분자, Y축, 내보내기, 알람, 세션 복원 등) |

---

## 수정 파일 전체 목록

| 파일 | 상태 | 수정 Phase |
|------|------|-----------|
| `js/analysis/statistics.js` | 수정 | 1 |
| `js/main.js` | 수정 | 1, 2, 3, 4, 5, 6 |
| `js/utils/helpers.js` | 수정 | 1, 3, 4 |
| `js/visualization/chart-handler.js` | 수정 | 2, 3, 4 |
| `js/analysis/analysis-ui.js` | 수정 | 2, 3, 5 |
| `js/analysis/processing.js` | 수정 | 3, 5 |
| `js/core/serial.js` | 수정 | 5 |
| `js/core/ble.js` | 수정 | 5 |
| `js/analysis/regression.js` | 수정 | 5 |
| `js/analysis/classification.js` | 수정 | 6 |
| `app.html` | 수정 | 3, 4, 5, 6 |
| `css/style.css` | 수정 | 4 |
| `translations/ko.json` | 수정 | 6 |
| `translations/en.json` | 수정 | 6 |
| `js/utils/alarm.js` | **신규** | 6 |

---

## V2 대비 V3 주요 개선 비교

| 영역 | V2 | V3 |
|------|----|----|
| 장시간 로깅 안정성 | 65K 포인트 초과 시 크래시 | 10만 포인트 다운샘플링, 50만행 CSV 분할 |
| 데이터 저장 | CSV 내보내기만 가능 | CSV Import + IndexedDB 세션 자동 저장/복원 |
| 분석 범위 | 뷰포트만 분석 | 전체/뷰포트 토글 선택 |
| 타임스탬프 | 브라우저 Date 기반 | performance.now() + MCU 타임스탬프 파싱 |
| 데이터 검증 | NaN만 체크 | Infinity/센서수 불일치/이상치(IQR) 감지 |
| 통신 방향 | 수신만 가능 | 양방향 (Serial TX + BLE TX) |
| BLE UUID | Nordic UART 고정 | 커스텀 UUID 입력 지원 |
| Baud rate | 2개 (9600, 115200) | 8개 (4800~460800) |
| 구분자 | 콤마 고정 | 콤마/탭/공백/세미콜론 선택 |
| Y축 | 단일 공유 + 자동 | 개별 Y축 + 수동 범위 설정 |
| 에러 알림 | alert() 모달 | 토스트 알림 (자동 사라짐) |
| 키보드 | 없음 | Space/Ctrl+S/Q/E 단축키 |
| 차트 내보내기 | 없음 | PNG 이미지 다운로드 |
| 알람 | 없음 | 센서별 상한/하한 + 브라우저 Notification |
| 분류 | 스텁 | k-NN 분류기 구현 |
| 다중 연결 | 불가 | Serial + BLE 동시 연결 |
| 회귀 분석 | 다항식 2차 고정 | 2~5차 선택 가능 |
| 스무딩 | 윈도우 5 고정 | 3~51 슬라이더 조절 |
| 로그 | 텍스트 누적 | 검색/필터 기능 |

---

*작성일: 2026-03-27*
*기준: Bowerbird PRO V2 → V3 전체 업그레이드*
