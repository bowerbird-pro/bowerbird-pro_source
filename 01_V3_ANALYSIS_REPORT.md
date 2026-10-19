# Bowerbird PRO V2 프로젝트 분석 보고서

## 프로젝트 개요

순수 클라이언트 사이드 웹앱으로, MCU(Arduino/ESP32/Micro:bit 등)로부터 Web Serial/BLE을 통해 센서 데이터를 수신하고, 실시간 차트 시각화 + 통계/회귀 분석 + CSV 내보내기를 제공하는 데이터 로깅 도구이다.

### 기술 스택

| 구분 | 기술 |
|------|------|
| 프론트엔드 | 순수 HTML5 + CSS3 + ES6 JavaScript (프레임워크 없음) |
| 차트 | Chart.js 3.9.1 + chartjs-plugin-zoom 2.0.1 |
| 통신 | Web Serial API, Web Bluetooth API |
| 데이터 저장 | 메모리 (csvData 배열) + CSV 내보내기 |
| 국제화 | 21개 언어 JSON 번역 파일 |
| 백엔드/DB | 없음 (완전 클라이언트 사이드) |

### 프로젝트 구조

```
boewrbirdpro-V2/
├── index.html                    # 랜딩 페이지
├── app.html                      # 메인 애플리케이션
├── information.html              # 사용자 매뉴얼
├── css/
│   └── style.css                 # 통합 스타일시트 (1120줄)
├── js/
│   ├── main.js                   # 진입점 & 앱 오케스트레이션
│   ├── core/
│   │   ├── serial.js             # Web Serial API 핸들러
│   │   └── ble.js                # Web Bluetooth API 핸들러
│   ├── visualization/
│   │   └── chart-handler.js      # Chart.js 통합 (줌/패닝)
│   ├── analysis/
│   │   ├── analysis-ui.js        # 분석 UI 컨트롤러
│   │   ├── regression.js         # 5가지 회귀 분석 모델
│   │   ├── statistics.js         # 기초 통계
│   │   ├── calculus.js           # 미분 & 적분
│   │   ├── processing.js         # 스무딩 필터
│   │   └── classification.js     # ML 분류 (스텁)
│   └── utils/
│       ├── i18n.js               # 국제화 (21개 언어)
│       └── helpers.js            # 로깅, CSV 내보내기, Wake Lock
├── translations/                 # 21개 언어 JSON 파일
└── assets/images/                # 앱 아이콘, 스크린샷, 문서 이미지
```

---

## 1. 기능(Functionality) 측면 부족한 점

### 1-1. 데이터 저장/불러오기 기능 없음

- CSV 내보내기만 존재하며 로깅 중단 시 자동 다운로드만 됨
- CSV/JSON 불러오기(Import) 기능 없음 — 이전에 수집한 데이터를 다시 분석할 방법이 없음
- 세션 저장/복원 불가 — 브라우저를 닫으면 모든 데이터 소실
- localStorage나 IndexedDB를 활용한 자동 저장 기능이 없음

### 1-2. classification.js는 완전한 스텁

```javascript
// classification.js - 10줄, 기능 없음
export function classifyState(features) {
    return "UNKNOWN";
}
```

ML 분류 기능을 광고하지만 실제 구현이 전혀 없음

### 1-3. 데이터 전송(MCU로 명령 보내기) 불가

- Serial/BLE 모두 수신(Read)만 지원, 송신(Write) 기능 없음
- MCU에 설정 변경 명령이나 캘리브레이션 명령을 보낼 수 없음

### 1-4. 다중 연결 미지원

- Serial과 BLE 동시 연결 불가 — 모듈별 싱글턴 패턴으로 하드코딩
- 동일 타입의 여러 기기 동시 연결도 불가

### 1-5. 알람/트리거 시스템 없음

- 센서 값이 임계값을 초과했을 때 경고를 표시하는 기능 없음
- 실험 모니터링 시 중요한 기능임

### 1-6. 차트 이미지 내보내기 없음

- 차트를 PNG/SVG로 저장하는 기능이 없음
- 보고서 작성 시 스크린샷을 찍어야 함

### 1-7. BLE UUID 하드코딩

- `ble.js:6-7` — Nordic UART Service UUID만 지원
- 사용자가 커스텀 BLE 서비스를 사용할 수 없음

---

## 2. 사용자 편리성(UX) 측면 부족한 점

### 2-1. 로깅 설정의 유연성 부족

- Baud rate 선택지 2개(9600, 115200)만 제공 — 19200, 38400, 57600, 230400 등 일반적인 보레이트 미지원
- 센서 개수를 로깅 전에 미리 설정해야 함 — 데이터 형식에서 자동 감지하지 않음
- 데이터 구분자가 콤마(`,`)로 고정 — 탭, 공백, 세미콜론 등 대안 없음

### 2-2. 로깅 중 설정 변경 불가

- 센서 이름, 개수 등 로깅 시작 후 변경하면 차트 데이터 구조가 깨질 위험
- 장시간 실험 중 설정을 바꿔야 할 때 로깅을 중단해야 함

### 2-3. 에러 처리 부족

- `main.js:128-131` — 연결 실패 시 `catch (e) { // Error handled in modules }` — 사용자에게 구체적 피드백 없음
- `main.js:163` — 연결 전 로깅 시도 시 `alert()` 사용 — 모달 알림은 UX에 좋지 않음
- BLE 연결 시 타임아웃 처리 없음

### 2-4. 키보드 단축키 없음

- Start/Pause/Stop 등 빈번한 조작에 키보드 단축키 미지원
- 실험 중 마우스 조작이 불편한 상황에서 사용성 저하

### 2-5. 데이터 로그의 검색/필터 기능 없음

- Console Log와 Data Log가 순수 텍스트 누적 방식
- 특정 시간대, 특정 센서 값만 필터링할 수 없음

### 2-6. 차트 Y축 범위 수동 설정 불가

- Y축이 항상 자동 스케일 — 특정 범위를 고정하고 싶을 때 방법 없음
- 센서 간 스케일이 크게 다를 때 작은 변화를 보기 어려움

### 2-7. 센서별 개별 Y축 미지원

- 모든 센서가 동일한 Y축(y) 공유
- 예: 온도(20~30)와 습도(0~100)를 동시에 볼 때 스케일 문제
- `y1` 축은 미분/적분 등 처리 데이터 전용으로만 사용

### 2-8. 연결 상태 시각적 표시 부족

- 연결 중/끊김/재연결 시도 등의 상태를 실시간 표시하는 인디케이터 없음
- BLE 신호 강도(RSSI) 표시 없음

---

## 3. 데이터의 효과적인 처리 측면 부족한 점

### 3-1. Math.min/max 스프레드 연산자 — 스택 오버플로 위험

```javascript
// statistics.js:18-19
const min = Math.min(...data);
const max = Math.max(...data);
```

- 스프레드 연산자는 데이터를 함수 인자로 전개함
- 데이터 포인트 약 65,000개 초과 시 `Maximum call stack size exceeded` 발생
- 장시간 로깅 시 확실히 이 한계를 넘음

### 3-2. 차트 데이터 메모리 관리 불완전

- `chart-handler.js:380-383` — SHIFTING 모드에서만 오래된 데이터 제거
- EXPANDING 모드나 ALL 모드에서는 데이터가 무한 누적 — 장시간 로깅 시 메모리 부족
- 분석 시 `getVisibleData()`가 전체 데이터를 선형 필터링 — 바이너리 서치 미사용

### 3-3. csvData 배열 무한 성장

```javascript
// main.js:155
csvData.push([currentTime.toLocaleDateString(getLanguage()), timeWithMs, ...values]);
```

- 모든 수신 데이터가 메모리의 `csvData` 배열에 누적
- 고속 샘플링(115200bps)에서 수시간 로깅하면 수백만 행 → 브라우저 메모리 고갈

### 3-4. 분석이 뷰포트 데이터에만 적용

- `analysis-ui.js:170` — `getVisibleData()`로 화면에 보이는 범위만 분석
- 줌/패닝하면 통계값이 계속 바뀜
- 전체 데이터에 대한 분석이 불가하며, 사용자가 전체 통계와 구간 통계를 구분해서 볼 수 없음

### 3-5. 시간 동기화 문제

```javascript
// main.js:142
const currentTime = new Date();
```

- MCU의 타임스탬프가 아닌 브라우저 수신 시각을 기록
- 직렬 통신 지연, 버퍼링, JavaScript 이벤트 루프 지연으로 실제 측정 시각과 차이 발생
- 고주파 샘플링에서 시간 정확도가 심각하게 떨어짐

### 3-6. 이상치(Outlier) 처리 없음

- 통신 노이즈나 센서 오류로 인한 이상치를 감지하거나 제거하는 기능 없음
- 이상치가 회귀 분석과 통계에 큰 영향을 미침

### 3-7. Smoothing 윈도우 크기 고정

```javascript
// processing.js — MA window=5, SG window=5 고정
smoothed = calculateMovingAverage(dataset.data, 5);
```

- 사용자가 윈도우 크기를 조절할 수 없음
- 노이즈 수준이나 샘플링 속도에 따라 최적 윈도우가 다른데 대응 불가

### 3-8. 회귀 분석 다항식 차수 고정

```javascript
// regression.js:20
return calculatePolynomialRegression(validPoints, 2); // 항상 2차
```

- 다항식 회귀가 2차로 고정 — 3차, 4차 등 선택 불가

### 3-9. 데이터 검증 불충분

```javascript
// main.js:151-152
const values = dataPart.split(',').map(v => parseFloat(v.trim()));
if (values.some(isNaN)) return logMessage('logDataWarning', { line });
```

- `Infinity`, `-Infinity`, 극단적 값에 대한 검증 없음
- 센서 수와 수신 값 개수 불일치 시 조용히 일부만 기록하거나 에러 없이 무시

### 3-10. 샘플링 레이트 정보 부재

- 실제 수신 데이터의 샘플링 레이트를 계산하거나 표시하지 않음
- 미분(dy/dx)의 정확도가 불균일한 시간 간격에 의존하지만 보정하지 않음

---

## 4. 개선 우선순위 요약

| 우선순위 | 영역 | 이슈 | 관련 파일 |
|---------|------|------|-----------|
| **긴급** | 데이터 | `Math.min(...data)` 스택 오버플로 — 장시간 로깅 시 크래시 | `statistics.js:18-19` |
| **긴급** | 데이터 | csvData 무한 성장 — 메모리 고갈 | `main.js:155` |
| **높음** | 기능 | 데이터 Import / 세션 저장 기능 없음 | 전체 |
| **높음** | 데이터 | 전체 데이터 분석 불가 (뷰포트만 분석) | `analysis-ui.js:170` |
| **높음** | UX | 센서 수 자동 감지 미지원 | `main.js:141-158` |
| **높음** | 데이터 | 브라우저 시각 기반 타임스탬프 부정확 | `main.js:142` |
| **중간** | 기능 | MCU로 데이터 송신 불가 | `serial.js`, `ble.js` |
| **중간** | UX | 센서별 개별 Y축 미지원 | `chart-handler.js` |
| **중간** | 데이터 | 이상치 감지/제거 없음 | `main.js:150-158` |
| **중간** | UX | Baud rate 선택지 부족 (2개만 제공) | `app.html` |
| **낮음** | 기능 | `classification.js` 미구현 스텁 | `classification.js` |
| **낮음** | UX | 키보드 단축키, 차트 이미지 내보내기 없음 | 전체 |

---

*분석일: 2026-03-27*
*분석 대상: boewrbirdpro-V2 전체 소스코드*
