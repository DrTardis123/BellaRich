# ANALYSIS-05: 코드 품질, 잠재 리스크 및 개선 제안

> **분석 대상 파일**: `index.html` (2,646줄), `quiz-data.js` (716줄)  
> **작성일**: 2026-08-30  
> **작성자**: Antigravity  

---

## 1. 단일 파일 구조의 한계 및 모듈화 분리 제안

현재 `index.html`은 총 **2,646줄(96KB)** 로 마크업(HTML), 스타일시트(CSS, ~1,320줄), 비즈니스 로직 및 이벤트 핸들러(JS, ~730줄)가 한 파일에 인라인으로 결합되어 있습니다.

### 1.1 구조적 한계와 유지보수 리스크
1. **브라우저 캐싱 불가**:
   * CSS와 JS가 인라인되어 있어 HTML을 새로 불러올 때마다 96KB 전체를 다시 다운로드해야 하므로 초기 로딩 속도 및 대역폭 낭비가 발생합니다.
2. **코드 가독성 및 탐색성 저하**:
   * 스타일 수정, 텍스트 카피 변경, 로직 디버깅이 단일 파일 안에서 뒤섞여 있어 코드 충돌(Git Conflict) 위험이 높습니다.
3. **관심사의 분리(SoC) 원칙 위배**:
   * UI 마크업, 스타일링, 카테고리/문항 렌더링, 데이터 수집, 공유 기능이 하나의 거대한 전역 스코프에 노출되어 있습니다.

### 1.2 권장 디렉터리 및 모듈 분리 구조

```
/
├── index.html                  # 순수 시맨틱 마크업 및 골격 (약 300줄)
├── css/
│   ├── reset.css               # 기본 리셋 및 폰트 설정
│   ├── components.css          # 버튼, 카드, 프로그레스 바 공통 컴포넌트
│   └── screens.css             # intro, category, question, consent, result 화면별 스타일
├── js/
│   ├── config.js               # 백엔드 URL, API 키, 상수 설정 (하드코딩 분리)
│   ├── quiz-data.js            # 카테고리, 문항, 등급 데이터 (기존 유지)
│   ├── app.js                  # 화면 전환 및 문항 렌더링 코어 로직
│   ├── analytics.js            # 세션 관리, Q0 수집, sendData (fetch / sendBeacon)
│   └── share.js                # Web Share API, Kakao SDK, 클립보드 복사
└── assets/
    ├── venn-diagram.png
    └── favicon.svg
```

---

## 2. 하드코딩된 설정값 목록 (Config 분리 대상)

코드베이스 전반에 산재된 하드코딩 값들을 별도의 `config.js` 파일로 추출하여 환경별(개발/스테이징/운영) 관리가 가능하도록 해야 합니다.

| 항목 | 현재 하드코딩된 값 | 소스 코드 위치 | 분리 권장 변수명 |
|---|---|---|---|
| **Apps Script WebApp URL** | `'https://script.google.com/macros/s/AKfycbzhUpGPuWJUssPk1F9hlvOwutqGy9w6Lo2fuygxdvDp4rCTRdteVvOk80ZDVgrDPNKX/exec'` | `index.html:1935` | `CONFIG.APPS_SCRIPT_URL` |
| **Kakao JavaScript Key** | `''` (빈 문자열) | `index.html:2446` | `CONFIG.KAKAO_JS_KEY` |
| **상담 수신 이메일** | `'bellarich7@naver.com'` | `index.html:1694, 1923, 2422, 2585` | `CONFIG.CONTACT_EMAIL` |
| **대표 전화번호** | `'+82-10-5031-0789'` | `index.html:37` | `CONFIG.CONTACT_PHONE` |
| **대표자 / 회사명** | `'벨라리치(BELLARICH) 김자영'` | `index.html:8, 31, 59, 1511, 1687` | `CONFIG.COMPANY_INFO` |
| **유튜브 채널 URL** | `'https://www.youtube.com/@bellarich7'` | `index.html:69, 1692, 1888` | `CONFIG.SNS.YOUTUBE` |
| **네이버 블로그 URL** | `'https://blog.naver.com/bellarich1004'` | `index.html:70, 1691, 1896` | `CONFIG.SNS.BLOG` |
| **인스타그램 URL** | `'https://www.instagram.com/bellarich1004'` | `index.html:71, 1693, 1904` | `CONFIG.SNS.INSTAGRAM` |
| **등급 컷오프 경계값** | `80, 60, 40` | `index.html:2356-2359` | `CONFIG.GRADE_CUTOFFS` |
| **세션 스토리지 키** | `'bellarich_sid'` | `index.html:1940, 1943` | `CONFIG.STORAGE_KEYS.SESSION_ID` |

---

## 3. 웹 접근성 (a11y) 진단: 적용점과 누락점

### 3.1 이미 잘 적용된 부분
* **시맨틱 랜드마크 & 건너뛰기 링크**: `<main id="main-content">` 및 `<a href="#main-content" class="sr-only">본문으로 건너뛰기</a>` 적용 (`index.html:1411, 1428`).
* **키보드 포커스 링**: `:focus-visible`에 명확한 고대비 골드 아웃라인(`outline: 3px solid #b08d4f`) 적용 (`index.html:93-100`).
* **문항 라디오 그룹 명시**: 문항 옵션 컨테이너에 `role="radiogroup"`, 각 선택지 버튼에 `role="radio"`, `aria-checked="true/false"`, `aria-label` 부여 (`index.html:2245-2254`).
* **진행률 바 접근성**: 프로그레스 바에 `role="progressbar"`, `aria-valuemin="0"`, `aria-valuemax="100"`, `aria-valuenow` 동적 갱신 (`index.html:1708, 1732, 2237`).
* **스크린리더 토스트 공지**: 공유 토스트 메시지에 `role="status"`, `aria-live="polite"` 지정 (`index.html:1876`).

### 3.2 누락 및 결함 부분 (개선 필요)
1. **Q0 자산규모 컨테이너의 `role="radiogroup"` 누락 (`index.html:1636-1643`)**:
   * 자식 요소(`.q0-opt`)에는 `role="radio"`가 선언되어 있으나, 부모 컨테이너(`#q0-options`)에 `role="radiogroup"` 및 `aria-label`이 누락되어 스크린리더가 라디오 그룹으로 인지하지 못함.
2. **화면 전환 시 스크린리더 라이브 리전(Live Region) 공지 부재**:
   * `#screen-category` → `#screen-question` 등으로 화면이 바뀔 때 시각적으로는 스크롤이 상단으로 이동하지만(`index.html:2177`), 스크린리더 사용자에게는 "질문 1/6 화면으로 이동했습니다"와 같은 `aria-live` 알림이 없어 화면 전환을 인지하기 어려움.
3. **벤다이어그램 이미지의 대체 텍스트 상세성 부족 (`index.html:1468`)**:
   * `alt="부동산 전문가 · 자산관리 · 법인 전문가 — 세 분야 교집합 융합 모델"`로만 되어 있어, 3대 영역의 구체적인 가치 제안이 시각장애인에게 충분히 전달되지 않음.
4. **포커스 트랩(Focus Trap) 및 자동 포커스 이동 관리 미흡**:
   * 질문이 바뀔 때(`goNext()`, `goBack()`) 포커스가 새로운 질문 제목(`h2#q-text`)으로 자동 이동하지 않아 키보드 사용자가 매번 탭을 여러 번 눌러야 함.

---

## 4. 잠재 버그, 엣지케이스 및 우선순위별 개선 과제 (P0 / P1 / P2)

```
[우선순위 분류 체계]
- P0 (Critical): 즉시 수정 필수 (법적 리스크, 치명적 문법 오류, 데이터 손실)
- P1 (Major): 조속한 수정 권장 (사용자 이탈 유발, UX 단절, 렌더링 결함)
- P2 (Minor): 점진적 개선 (코드 청결도, 캐싱, 상태 지속성)
```

| 우선순위 | 항목 ID | 문제점 및 영향 | 원인 및 소스 위치 | 권장 수정 방안 |
|:---:|:---:|---|---|---|
| **P0** | **BUG-01** | **HTML 최하단 중복 태그 문법 오류**<br>(W3C 표준 위반 및 브라우저 파서 오동작 가능성) | `index.html:2643-2646`에 `</body>\n</html>\n</body>\n</html>` 중복 작성됨 | 중복된 2645~2646줄 `</body></html>` 삭제 |
| **P0** | **LEGAL-01** | **인트로 "이메일 미수집" vs 동의폼 "이메일 필수" 충돌**<br>(개인정보보호법 및 표시광고법 위반 소지) | `index.html:1626` ("수집하지 않음: 이메일") vs `index.html:1769` ("이메일 주소 * 필수") 정면 충돌 | 인트로 문구를 "상세 결과 보고서 수신 시 이메일 수집"으로 정정 및 개인정보처리방침 링크 추가 |
| **P0** | **BUG-02** | **이메일 제출 중복 클릭(Double Submit) 방지 부재**<br>(동일 세션에 대해 Google Sheets 및 메일 다중 발송) | `submitConsent()`(`index.html:2307`) 실행 시 버튼 `disabled` 처리나 로딩 스피너가 없음 | `btn-consent.disabled = true` 처리 및 "발송 중..." 상태 UI 표시 |
| **P1** | **UX-01** | **1번 문항에서 카테고리 재선택 불가 (뒤로가기 단절)**<br>(카테고리 잘못 고른 사용자의 이탈 유발) | `index.html:2256`에서 `current === 0`일 때 `btn-back`이 무조건 비활성화됨 | `current === 0`일 때 `goBack()` 클릭 시 `#screen-category`로 되돌아가도록 분기 구현 |
| **P1** | **UX-02** | **동의 화면(`#screen-consent`)에서 뒤로가기 버튼 부재**<br>(이전 답변 수정 불가) | `index.html:1749-1796` 동의 화면에 이전 단계로 가는 버튼이 아예 없음 | 동의 화면에 "← 이전 질문으로" 버튼 추가하여 6번 문항으로 복귀 허용 |
| **P1** | **BUG-03** | **결과 화면 DOM 렌더링 혼선 및 CTA 버튼 링크 미바인딩**<br>(결과 화면에서 솔루션 카드 숨김 방식의 불안정성) | `showResult()`(`index.html:2395-2398`)에서 JS로 `display: none` 처리하나, HTML 마크업에 남아있고 `buildCtaLink()`가 실제 버튼에 연결 안 됨 | 메일 발송 전용 UI로 마크업을 명확히 정리하거나, 화면에도 요약 CTA를 표시하도록 통일 |
| **P1** | **BUG-04** | **이메일 검증 정규식의 취약성**<br>(`a@b.c` 등 무효한 이메일 입력 통과) | `index.html:2323`의 `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`는 도메인 길이 및 오타를 충분히 거르지 못함 | 정규식 강화 (`/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/`) 및 도메인 검증 |
| **P2** | **CLEAN-01** | **미사용 레거시 10문항 `QUESTIONS` 배열 방치 (Dead Code)**<br>(코드 용량 낭비 및 유지보수 혼란) | `index.html:2063-2165` (102줄 분량)의 `QUESTIONS` 배열이 전혀 사용되지 않음 | 불필요한 `QUESTIONS` 상수 코드 완전 삭제 |
| **P2** | **STATE-01** | **새로고침 시 상태 소실 (State Persistence 부재)**<br>(5번 문항 풀다가 새로고침 시 처음 인트로로 리셋) | 진행 중인 카테고리, 문항 인덱스, 답변이 `sessionStorage`에 백업되지 않음 | `sessionStorage`에 진행 상태를 자동 저장하여 새로고침 시 이어서 풀기 지원 |
| **P2** | **CONFIG-01** | **Kakao SDK JavaScript 키 미등록 상태**<br>(카카오톡 공유 클릭 시 무조건 클립보드 복사 폴백) | `index.html:2446` `const KAKAO_JS_KEY = '';` 빈값으로 방치 | 카카오 개발자 콘솔에서 정식 JS Key 발급받아 등록 |
