# FactoryAI — E2 원클릭 감사 리포트 (Lot Merge + PDF) Issues (E2-CMD-001 ~ E2-UI-003)

> **Source**: SRS-002 Rev 2.0 (V0.8) — E2 감사 리포트 자동화  
> **작성일**: 2026-04-19  
> **총 Issue**: 11건 (Command 7건 + Query 1건 + UI 3건)  
> **목적**: 작업자(클레어 리)가 며칠씩 밤새워 하던 엑셀 취합 작업을 1클릭(≤5초 소요)으로 대체하고, XAI 판단 근거가 포함된 증빙용 PDF 리포트를 산출한다.

> [!IMPORTANT]
> E2 Epic은 **Lot 기반 데이터 무결성 검증**과 **PDF 생성 엔진**이 가장 중요합니다.  
> 결측치 및 충돌 발생 시 강제로 발행되지 않고 작업자에게 **보완을 1차적으로 요구(Fail-Safe)** 해야 합니다.  
> HITL(품질이사) 최종 승인을 거쳐야만 공식 문서로 인정됩니다.

---

## E2-CMD-001: [Command] Lot 시간순 병합 로직 구현

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature/E2] E2-CMD-001: Lot 시간순 병합 로직 (타임스탬프 중복/역전 검증 포함)"
labels: 'feature, backend, core, priority:must, epic:e2-audit-report'
assignees: ''
---

### :dart: Summary
- **기능명**: [E2-CMD-001] Lot 시간순 병합 알고리즘
- **목적**: E1을 통해 수집된 LOG_ENTRY(STT/Vision) 데이터와 부분적인 ERP 데이터를 타임스탬프(`captured_at`) 기준으로 정확하게 하나의 Lot 타임라인으로 병합(Merge)한다.

### :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: REQ-FUNC-009, REQ-FUNC-013
- 데이터 모델: `DB-009` LOT, `DB-007` LOG_ENTRY

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `utils/lot-merger.ts` 병합 순수 함수 작성 (단위 테스트 용이성 확보)
- [ ] **2.** 특정 Lot ID에 속한 모든 `APPROVED` 상태의 LOG_ENTRY 데이터 조회
- [ ] **3.** 타임스탬프 오름차순 정렬
- [ ] **4.** 시간 역전(이후 데이터가 이전 타임스탬프를 가짐) 현상 감지 알고리즘 로직 
- [ ] **5.** 타임스탬프 중복(동일 시간 복수 데이터) 시 최신 갱신 데이터 우선순위 로직 지정

### :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 정상적인 병합**
- **Given**: A시간(+0분), B시간(+10분)의 로그 데이터
- **When**: Lot 병합을 지시한다.
- **Then**: 시간순으로 정렬된 배열 데이터를 반환하며, 오병합(순서 뒤바뀜)이 0건임을 보장한다.

**Scenario 2: 데이터 역전 감지 시 경고**
- **Given**: B시간(+10분) 로그가 A시간(+5분) 로그보다 먼저 입력된 경우
- **When**: 병합을 시도한다.
- **Then**: `TIMESTAMP_CONFLICT` 에러 또는 경고 배열 객체를 반환하여 수동 확인을 유도한다. (E2-CMD-006 블로킹용)

### :gear: Technical & Non-Functional Constraints
- 성능: 1,000건의 로그 배열 병합에 CPU 처리 시간 ≤ 50ms (메모리 O(N log N) 소트 사용)

### :checkered_flag: Definition of Done (DoD)
- [ ] 순수 함수 알고리즘 작성
- [ ] jest 기반 TDD 시나리오 5종 (중복, 역전, 병합, 빈 배열 등) 통과

### :construction: Dependencies & Blockers
- **Depends on**: `DB-009`, `DB-007`
- **Blocks**: `E2-CMD-002`, `E2-CMD-003`, `E2-CMD-006`

---

## E2-CMD-002: [Command] 클라이언트 브라우저 PDF 생성 모듈

---
name: Feature Task
title: "[Feature/E2] E2-CMD-002: 클라이언트 브라우저 PDF 생성 모듈 (@react-pdf/renderer)"
labels: 'feature, frontend, pdf, priority:must, epic:e2-audit-report'
---

### :dart: Summary
- **기능명**: [E2-CMD-002] 브라우저 사이드 PDF 렌더러
- **목적**: 병합이 완료된 JSON 데이터를 규제 양식(테이블, 서명 란 포함)에 맞는 PDF로 클라이언트 브라우저 단에서 즉석 렌더링하고 다운로드 기능을 제공한다. 클라우드 서버 파서 비용 제거(CON-07).

### :link: References (Spec & Context)
- SRS: REQ-FUNC-009, ASM-09, CON-07

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `@react-pdf/renderer` 패키지 설치
- [ ] **2.** `components/pdf/AuditReportDocument.tsx` 템플릿 컴포넌트 마크업
- [ ] **3.** 표 형식, 폰트 임베딩(초기 로딩 지연 방지를 위해 경량 폰트 제공)
- [ ] **4.** 로딩 진행률 및 Blob URL 생성 후 브라우저 Blob 다운로드 호출 트리거 작성

### :checkered_flag: Definition of Done (DoD)
- [ ] 한글 폰트가 깨지지 않고 렌더링 되는지 확인
- [ ] 병합된 JSON 덤프 데이터 주입 시 최대 3장 이상의 페이지 브레이크(Page Break) 정상 작동 확인

### :construction: Dependencies & Blockers
- **Depends on**: `E2-CMD-001`
- **Blocks**: `E2-CMD-004` (XAI 삽입), `E2-CMD-007`

---

## E2-CMD-003: [Command] 결측치 감지 및 보완 알림

---
name: Feature Task
title: "[Feature/E2] E2-CMD-003: 필수 데이터 누락 감지 및 보완 알림 생성 (감지율 ≥95%)"
labels: 'feature, backend, core, priority:must, epic:e2-audit-report'
---

### :dart: Summary
- **기능명**: [E2-CMD-003] 필수 데이터(결측치) 감지 로직
- **목적**: 감사 리포트에 기재되어야 할 필수 항목(예: 온습도, 밸브 압력 등)이 특정 시간대 동안 하나도 기록되지 않은 경우(Missing), 이를 자동 감지하고 관리자에게 채워넣으라는 알림을 발송한다.

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** Regulation 스키마별 `required_fields` 정의
- [ ] **2.** E2-CMD-001 병합 데이터 전체를 스캔하여 필수 항목 유무 체크
- [ ] **3.** 결측 건이 1개라도 있을 경우 PDF 렌더러로 넘어가지 않고 에러(422) 반환
- [ ] **4.** 해당 건에 대해 `NOTI-001` 서비스 호출하여 `[결측치 발생]` 알림 발송

### :test_tube: Acceptance Criteria (BDD/GWT)
- **Given**: 온도 데이터가 2시간 동안 공백인 Lot
- **When**: 감사 리포트 생성을 시도한다
- **Then**: 생성 차단, "온도 데이터 결측치 보완 필요" 명시 객체 반환, 알림 발송 (≤30초 내).

---

## E2-CMD-004: [Command] XAI 판단 근거 PDF 포함

---
name: Feature Task
title: "[Feature/E2] E2-CMD-004: XAI 판단 근거 PDF 한국어 설명 포함"
labels: 'feature, frontend, pdf, priority:must, epic:e2-audit-report'
---

### :dart: Summary
- **기능명**: [E2-CMD-004] 품질이상 관련 XAI 텍스트 PDF 렌더링
- **목적**: E2B-CMD-001 (XAI 모듈)이 생성해준 "왜 이 제품 배치를 의심/정상으로 판단했는지"에 대한 자연어 설명 문구를 최종 감사 리포트 PDF에 결합 반영한다. (품질 이사 승인 시 최종 판단 근거로 작용)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `utils/xai-fetcher.ts` 에서 `AUDIT_REPORT.xai_explanation` 텍스트 로드
- [ ] **2.** `AuditReportDocument.tsx` 내 [인공지능 판단 의견] 블록 신설
- [ ] **3.** 긴 텍스트의 줄바꿈 및 페이지 분할 방어 대응 (Overflow/Wrap)

---

## E2-CMD-005: [Command] 미지원 규제 포맷 요청 차단

---
name: Feature Task
title: "[Feature/E2] E2-CMD-005: 미지원 규제 포맷 요청 시 에러 핸들링 및 대안 제안"
labels: 'feature, backend, priority:low, epic:e2-audit-report'
---

### :dart: Summary
- **명세**: REQ-FUNC-012 대응. 사용자가 시스템이 지원하지 않는 규제 타입(예: ISO_9001 미구현 상태에서 요청)을 API `/audit-reports` DTO에 명시할 경우, 백엔드에서 2초 이내에 400 에러와 함께 `지원 불가 포맷. 현 보유 중인 유사 포맷은 HACCP 입니다.` 형태의 대안을 Object로 리턴한다.
- **DoD**: 크래시 0건 방어. 스키마 Enum으로 엄격한 제한 설정 (Zod 적용).

---

## E2-CMD-006: [Command] Lot 충돌 자동 병합 차단

---
name: Feature Task
title: "[Feature/E2] E2-CMD-006: Lot 충돌 감지 시 자동 병합 차단 + 충돌 객체 반환"
labels: 'feature, backend, priority:must, epic:e2-audit-report'
---

### :dart: Summary
- **명세**: E2-CMD-001에서 도출된 시간 역전/충돌 이슈를 UI로 전달하는 접착 역할을 한다. AI 자동 조정을 맹신하지 않고, 오류 시 무조건 인간(관리자)에게 확정을 넘기는(Fail-Safe) 기능을 구현한다. (REQ-FUNC-013)
- **실행 로직**: Conflict Exception 을 catch하여 `{ conflicting_lots: [{...}, {...}] }` 구조로 렌더링 뷰(UI)에 전달. (응답시간 ≤5초)

---

## E2-CMD-007: [Command] 감사 리포트 HITL 승인 워크플로

---
name: Feature Task
title: "[Feature/E2] E2-CMD-007: 감사 리포트 HITL 승인 워크플로 (PENDING → 승인/반려)"
labels: 'feature, backend, security, priority:must, epic:e2-audit-report'
---

### :dart: Summary
- **기능명**: [E2-CMD-007] 리포트 최종 발행 프로세스 관리
- **목적**: 생성된 `AUDIT_REPORT`가 외부/규제기관에 즉각 전송되지 않고, 반드시 '품질이사'(또는 AUDITOR 권한)의 `APPROVED`를 얻었을 때 비로소 완료 상태가 되는 안전장치를 구축한다.

### :link: References (Spec & Context)
- SRS: §3.4.2 시퀀스, REQ-FUNC-009
- 기반: `HITL-CMD-001` (전역 HITL 프레임워크)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `PATCH /api/v1/audit-reports/{id}/approve` 라우트 구축
- [ ] **2.** AUDITOR, ADMIN 역할만 승인할 수 있도록 RBAC 매트릭스 검사 적용 (`AUTH-002`)
- [ ] **3.** DB 상태 업데이트 (`APPROVED`) 및 `approved_by` 컨텍스트 저장
- [ ] **4.** 만일 `xai_explanation` 필드가 NULL인 경우 승인을 원천 차단 (HITL ② 규칙)
- [ ] **5.** 승인 시 감사 로그 기록 연계 (`AUTH-003`)

### :test_tube: Acceptance Criteria (BDD/GWT)
- **Given**: XAI 설명이 누락된 `AUDIT_REPORT`
- **When**: 관리자가 승인 요청을 보낸다
- **Then**: 403 차단 및 "XAI 설명 없는 리포트는 승인할 수 없습니다" 에러 반환. CISO 이벤트 발송.

---

## E2-QRY-001: [Query] 감사 리포트 목록/상세 조회 API

---
name: Feature Task
title: "[Feature/E2] E2-QRY-001: 감사 리포트 목록 및 상세 이력 조회 (Query)"
labels: 'feature, backend, query, priority:should, epic:e2-audit-report'
---

### :dart: Summary
- **목적**: 기존에 발급/승인되었거나 PENDING 상태인 리포트의 이력을 페이징 처리하여 보여준다.
- **실행**: `GET /api/v1/audit-reports` (전체/필터), `GET /api/v1/audit-reports/{id}` (단건-상세) 구축.
- **제약**: DTO `PaginationMeta` 인터페이스 규격 적용. AUTH 검증 (토큰 보유자만 조회).

---

## E2-UI-001 ~ E2-UI-003: 프론트엔드 리포트 컴포넌트

---
name: Feature Task
title: "[Feature/E2] E2-UI-001: 1클릭 감사 리포트 생성 관리자 UI"
labels: 'feature, frontend, ui, priority:must, epic:e2-audit-report'
---

### :dart: Summary
- **기능명**: [E2-UI-001] "감사 리포트 자동 생성" 메인 뷰
- **목적**: 관리자가 며칠 날짜 구간, 적용 도메인 규제(HACCP, 삼성 등)를 선택하고 [생성] 버튼을 한 번 눌러 리포트 생성을 트리거하는 메인 화면.

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** 폼 컴포넌트 개발: 기간 DatePicker, 규제 타입 SelectBox
- [ ] **2.** 버튼 클릭 시 `POST /api/v1/audit-reports` 호출
- [ ] **3.** 대기 시간 동안 `AI-003` 컴포넌트 활용하여 "데이터 정합성 확인 중..." UI 표시
- [ ] **4.** 성공 시 생성된 PDF Blob을 브라우저에 바로보기(Preview) 삽입 (`iframe` 혹은 `@react-pdf/renderer` 뷰어)

---
name: Feature Task
title: "[Feature/E2] E2-UI-002: 결측치 보완 알림 모달 UI"
labels: 'feature, frontend, ui, priority:must, epic:e2-audit-report'
---

### :dart: Summary
- **목적**: E2-CMD-003에서 422 에러와 결측치 객체를 전달받았을 때 화면에 빨간색 텍스트와 함께 "다음 항목이 기록되지 않았습니다: [온도, PH농도]" 라며 수동 기입 폼을 오픈해주는 Modal UI.
- **DoD**: 사용자가 누락 칸을 치고 <저장> 누르면 해당 페이로드와 함께 RE-TRY API 요청 보장.

---
name: Feature Task
title: "[Feature/E2] E2-UI-003: Lot 충돌 해결 드래그 앤 드롭 UI"
labels: 'feature, frontend, ui, priority:should, epic:e2-audit-report'
---

### :dart: Summary
- **목적**: E2-CMD-006 발동 시, 타임스탬프가 꼬인 2~3개의 데이터 Row를 눈앞에 보여주고, 관리자가 `드래그 앤 드롭(Drag & Drop)` 또는 위/아래 화살표 버튼으로 강제 순서를 지정해주는 테이블 뷰 완성.
- **스택**: `dnd-kit` 또는 단순 버튼 순서 변경 로직 적용.
