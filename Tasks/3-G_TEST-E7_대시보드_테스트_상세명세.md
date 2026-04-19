# 3-G. E7 대시보드 테스트 — GitHub Issue 상세명세서

**Source**: SRS-002 Rev 2.0 (V0.8) / `3.0_Full TASKS list.md` §3-G  
**작성일**: 2026-04-19  
**작성자**: Senior Full-Stack Engineer  
**대상 태스크**: TEST-E7-001 ~ TEST-E7-005 (5건)  
**Epic**: E7 — 성과 가시화/리텐션 대시보드

---

## TEST-E7-001

---
name: Test Task
about: SRS 기반의 AC 검증 테스트 태스크 명세
title: "[Test] TEST-E7-001: 월말 자동 발행 → 4인 맞춤 대시보드 생성 검증"
labels: 'test, backend, e7-dashboard, priority:high'
assignees: ''
---

### :dart: Summary
- **기능명**: [TEST-E7-001] 월말 자동 발행 → 4인 맞춤 대시보드 (≤24시간, 렌더링 ≤5초) 테스트
- **목적**: 매월 말 자동 트리거에 의해 4인 페르소나(COO/구매본부장/품질이사/CFO) 각각에 맞춤화된 성과 대시보드 데이터가 ≤24시간 내에 생성되고, 프론트엔드에서 ≤5초 이내에 렌더링되는 것을 검증한다. 이는 고객 리텐션의 핵심 기능이며, 경영진에게 FactoryAI 도입 효과를 가시적으로 전달하는 최종 산출물이다.

### :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: [`SRS_V_1.0.md#REQ-FUNC-035`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/2_SRS_V_1.0.md) — §4.1.7 E7 성과 가시화/리텐션 대시보드
- API 명세: [`SRS_V_1.0.md#6.1`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/2_SRS_V_1.0.md) — §6.1 API #16 `POST /api/v1/dashboards/publish`
- 선행 구현 태스크: [`3.0_Full TASKS list.md#E7-CMD-001`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/3.0_Full%20TASKS%20list.md) — 월말 자동 발행 트리거 + 4인 맞춤 대시보드 데이터 생성
- 데이터 모델: [`SRS_V_1.0.md#6.2.13`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/2_SRS_V_1.0.md) — SUBSCRIPTION 엔터티 (MRR, cumulative_savings)
- NFR: REQ-NF-004 (대시보드 렌더링 p95 ≤5,000ms)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] 월말 자동 트리거 시뮬레이션: fakeTimers로 월말 시점 트리거 실행 확인
- [ ] 4인 페르소나별 대시보드 데이터 생성 검증: COO/구매본부장/품질이사/CFO 각각 독립 데이터셋
- [ ] COO 대시보드 내용 검증: 종합 운영 지표 (생산성, 가동률, 결측률 추이)
- [ ] 품질이사 대시보드 내용 검증: 품질 지표 (불량률, 이상 징후 건수, XAI 판단 이력)
- [ ] CFO 대시보드 내용 검증: 재무 지표 (절감액, ROI, 바우처 현황)
- [ ] 구매본부장 대시보드 내용 검증: 조달/재고 지표 (ERP 동기화 현황, 정합성)
- [ ] 생성 속도 검증: 전체 4인 대시보드 생성 ≤24시간 (시뮬레이션 기준)
- [ ] 렌더링 속도 검증: 프론트엔드 렌더링 ≤5초

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 월말 자동 트리거 → 4인 대시보드 생성 완료**
- **Given**: 2026-04-30 23:59 (월말 시점)에 자동 발행 트리거가 예약되어 있음
- **When**: 트리거가 실행됨
- **Then**: 4인(COO/구매본부장/품질이사/CFO) 각각에 대한 대시보드 데이터가 생성되고, `SUBSCRIPTION` 테이블에 발행 이력이 기록되며, 생성 완료 알림이 각 페르소나에게 발송된다.

**Scenario 2: COO 대시보드 — 운영 지표 포함 확인**
- **Given**: COO 페르소나용 대시보드가 생성됨
- **When**: `GET /api/v1/dashboards?persona_type=COO&period=2026-04`를 조회함
- **Then**: 다음 지표가 포함: ① 월간 로깅 건수 ② 결측률 추이 ③ 생산라인별 가동률 ④ 전월 대비 개선율

**Scenario 3: CFO 대시보드 — 재무 지표 포함 확인**
- **Given**: CFO 페르소나용 대시보드가 생성됨
- **When**: CFO 대시보드를 조회함
- **Then**: 다음 지표가 포함: ① 누적 절감액 ② ROI 달성률 ③ 바우처 집행 현황 ④ Payback 잔여 기간

**Scenario 4: 렌더링 속도 ≤5초 검증**
- **Given**: 생성된 대시보드 데이터가 존재함
- **When**: 프론트엔드에서 대시보드 페이지를 로드함
- **Then**: 초기 렌더링이 ≤5초 내에 완료되고, 진행률 표시기가 표시된다.

### :gear: Technical & Non-Functional Constraints
- **성능**: 대시보드 데이터 생성 ≤24시간, 렌더링 p95 ≤5,000ms
- **페르소나**: 4인 맞춤 (COO/구매본부장/품질이사/CFO) 독립 데이터셋
- **데이터 소스**: LOG_ENTRY(E1) + AUDIT_REPORT(E2) + SUBSCRIPTION(E7) 크로스 집계
- **테스트 도구**: Jest (단위/통합 — 데이터 생성), Playwright (E2E — 렌더링 속도)
- **테스트 위치**:
  - `src/__tests__/e7/test-e7-001-dashboard-publish.test.ts`
  - `e2e/e7/test-e7-001-dashboard-render.spec.ts`

### :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria(Scenario 1~4)를 충족하는 자동화 테스트가 작성되었는가?
- [ ] 4인 페르소나별 데이터 내용 검증이 포함되었는가?
- [ ] 렌더링 ≤5초가 E2E 테스트로 확인되었는가?
- [ ] SonarQube / ESLint 정적 분석 도구 경고가 없는가?

### :construction: Dependencies & Blockers
- **Depends on**: E7-CMD-001 (월말 자동 발행 구현), DB-015 (SUBSCRIPTION 스키마), DB-007 (LOG_ENTRY), DB-010 (AUDIT_REPORT), AUTH-002 (RBAC)
- **Blocks**: TEST-E7-004 (데이터 부족 NAC — 정상 발행 통과 후 착수)

---

## TEST-E7-002

---
name: Test Task
about: SRS 기반의 AC 검증 테스트 태스크 명세
title: "[Test] TEST-E7-002: 분기 말 ROI 누적 리포트 자동 집계 검증"
labels: 'test, backend, e7-dashboard, priority:high'
assignees: ''
---

### :dart: Summary
- **기능명**: [TEST-E7-002] 분기 말 ROI 누적 리포트 자동 집계 (수동 개입 0건) 테스트
- **목적**: 분기별로 ROI 누적 리포트(절감액/생성건수)가 **수동 개입 없이** 완전 자동으로 집계되는 것을 검증한다. 이는 CFO에게 분기 성과를 보고하고, 계속 이용 의사결정을 지원하는 핵심 리포트이다.

### :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: [`SRS_V_1.0.md#REQ-FUNC-036`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/2_SRS_V_1.0.md) — §4.1.7 분기 말 ROI 누적 리포트
- 선행 구현 태스크: [`3.0_Full TASKS list.md#E7-QRY-001`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/3.0_Full%20TASKS%20list.md) — 분기 말 ROI 누적 리포트 자동 집계
- 데이터 소스: DB-015 (SUBSCRIPTION), DB-007 (LOG_ENTRY), DB-010 (AUDIT_REPORT)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] 분기 자동 트리거 시뮬레이션: Q1(3월 말), Q2(6월 말) 시점 트리거 검증
- [ ] 절감액 자동 집계 검증: 분기 내 전체 절감액 합산 정확성
- [ ] 리포트 생성 건수 집계 검증: 분기 내 감사 리포트 발행 건수 정확성
- [ ] 수동 개입 0건 확인: 트리거 → 집계 → 저장 전 과정에서 인간 입력 없음
- [ ] 이전 분기 데이터 비교: 전분기 대비 증감률 자동 산출
- [ ] 집계 정합성: SUM/COUNT 쿼리 결과 vs 리포트 값 일치 검증

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: Q1 분기 말 → ROI 누적 리포트 자동 생성**
- **Given**: 2026-01-01 ~ 2026-03-31 기간 동안 절감액 5,000만원, 감사 리포트 15건이 축적됨
- **When**: 분기 말 자동 집계 트리거가 실행됨
- **Then**: ROI 누적 리포트가 자동 생성되며: `cumulative_savings=50,000,000`, `report_count=15`, `quarter=Q1-2026` 값이 정확하고, **수동 개입 = 0건**.

**Scenario 2: 전분기 대비 증감률 자동 산출**
- **Given**: Q1 절감액=5,000만원, Q2 절감액=6,500만원
- **When**: Q2 누적 리포트가 생성됨
- **Then**: `quarter_growth_rate=+30%`, `cumulative_total=11,500만원`이 자동 산출된다.

**Scenario 3: 데이터 정합성 — DB 직접 집계와 리포트 값 일치**
- **Given**: 리포트가 생성됨
- **When**: DB에서 `SELECT SUM(savings) FROM log_entries WHERE quarter='Q1-2026'`을 직접 실행함
- **Then**: 직접 쿼리 결과와 리포트의 `cumulative_savings` 값이 정확히 일치한다.

### :gear: Technical & Non-Functional Constraints
- **자동화**: 수동 개입 **0건** — 트리거~저장 전 과정 완전 자동
- **정합성**: DB 직접 집계 vs 리포트 값 오차 0원
- **주기**: 분기별 (3/6/9/12월 말) 자동 실행
- **테스트 도구**: Jest (단위/통합), fakeTimers
- **테스트 위치**: `src/__tests__/e7/test-e7-002-quarterly-roi.test.ts`

### :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria(Scenario 1~3)를 충족하는 자동화 테스트가 작성되었는가?
- [ ] 수동 개입 0건이 전 과정 자동화로 확인되었는가?
- [ ] DB 직접 집계 vs 리포트 값 정합성이 검증되었는가?
- [ ] SonarQube / ESLint 정적 분석 도구 경고가 없는가?

### :construction: Dependencies & Blockers
- **Depends on**: E7-QRY-001 (분기 ROI 집계 구현), DB-015 (SUBSCRIPTION), DB-007 (LOG_ENTRY), DB-010 (AUDIT_REPORT)
- **Blocks**: 없음 (독립 집계 테스트)

---

## TEST-E7-003

---
name: Test Task
about: SRS 기반의 AC 검증 테스트 태스크 명세
title: "[Test] TEST-E7-003: NPS 9~10점 감지 시 1클릭 NPS + 레퍼런스 동의 수집 검증"
labels: 'test, backend, frontend, e7-dashboard, priority:medium'
assignees: ''
---

### :dart: Summary
- **기능명**: [TEST-E7-003] NPS 9~10점 → 1클릭 NPS + 동의 수집 (응답률 ≥30% 구조 검증) 테스트
- **목적**: 대시보드 이용 후 NPS(Net Promoter Score) 9~10점(Promoter)을 준 사용자에게 자동으로 1클릭 NPS 설문과 레퍼런스 동의 수집 요청을 발송하고, 응답 수집 구조가 ≥30% 응답률을 달성할 수 있는 UX를 갖추고 있는지 구조적으로 검증한다.

### :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: [`SRS_V_1.0.md#REQ-FUNC-037`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/2_SRS_V_1.0.md) — §4.1.7 NPS 설문 + 레퍼런스 수집
- API 명세: [`SRS_V_1.0.md#6.1`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/2_SRS_V_1.0.md) — §6.1 API #17 `POST /api/v1/nps/survey`
- 선행 구현 태스크: [`3.0_Full TASKS list.md#E7-CMD-004`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/3.0_Full%20TASKS%20list.md) — NPS 설문 발송 + 동의 수집

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] NPS 점수 감지 테스트: 9~10점 Promoter 필터링 정확성
- [ ] 설문 발송 테스트: 대상자에게 1클릭 NPS 설문이 발송됨
- [ ] 1클릭 응답 구조 테스트: 단일 클릭으로 응답 완료되는 UX 흐름
- [ ] 레퍼런스 동의 수집 테스트: NPS 응답 후 레퍼런스 동의 옵션 표시
- [ ] 응답률 구조 검증: 1클릭 UX가 일반 설문 대비 응답 마찰 최소화 확인
- [ ] 비대상자 필터링: NPS 8점 이하 사용자에게는 발송하지 않음

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: NPS 9점 사용자 → 1클릭 설문 + 레퍼런스 동의 발송**
- **Given**: COO가 대시보드에서 NPS 9점을 제출함
- **When**: NPS 점수가 기록됨
- **Then**: 해당 사용자에게 ① 1클릭 NPS 재확인 설문 ② 레퍼런스 동의 수집 요청이 자동 발송된다.

**Scenario 2: 1클릭 응답 완료 구조**
- **Given**: 설문이 발송되어 사용자 화면에 표시됨
- **When**: 사용자가 "추천합니다" 버튼을 1회 클릭함
- **Then**: 응답이 즉시 기록되고, 추가 입력 없이 완료 안내가 표시된다(1클릭 UX).

**Scenario 3: NPS 8점 이하 → 설문 미발송**
- **Given**: 품질이사가 NPS 7점을 제출함
- **When**: NPS 점수가 기록됨
- **Then**: 1클릭 설문이 발송되지 않는다 (Detractor/Passive는 대상 아님).

**Scenario 4: 레퍼런스 동의 수집 + 기록**
- **Given**: NPS 10점 사용자가 1클릭 응답을 완료함
- **When**: "레퍼런스 고객으로 등록합니다" 동의 체크박스를 선택함
- **Then**: DB에 `reference_consent=true`, `consent_date`, `user_id`가 기록된다.

### :gear: Technical & Non-Functional Constraints
- **UX**: 1클릭으로 응답 완료 (추가 페이지/입력 없음)
- **대상**: NPS 9~10점 (Promoter)만 선별
- **응답률 구조**: ≥30% 달성 가능한 마찰 최소화 UX
- **테스트 도구**: Jest (단위/통합), Playwright (E2E — 1클릭 UX)
- **테스트 위치**: `src/__tests__/e7/test-e7-003-nps-survey.test.ts`

### :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria(Scenario 1~4)를 충족하는 자동화 테스트가 작성되었는가?
- [ ] NPS 9~10 필터링 + 8 이하 제외가 모두 검증되었는가?
- [ ] 1클릭 UX 흐름이 E2E 테스트로 확인되었는가?
- [ ] SonarQube / ESLint 정적 분석 도구 경고가 없는가?

### :construction: Dependencies & Blockers
- **Depends on**: E7-CMD-004 (NPS 설문 발송 구현), DB-002 (USER 스키마), AUTH-002 (RBAC)
- **Blocks**: 없음 (독립 테스트)

---

## TEST-E7-004

---
name: Test Task
about: SRS 기반의 NAC 검증 테스트 태스크 명세
title: "[Test] TEST-E7-004: 데이터 부족 시 대시보드 미발행 + 경고 검증"
labels: 'test, backend, e7-dashboard, negative-ac, priority:high'
assignees: ''
---

### :dart: Summary
- **기능명**: [TEST-E7-004] 당월 로깅 <100건 → "데이터 부족" 경고 + 미발행 (오해 유발 대시보드 0건) 테스트
- **목적**: 당월 패시브 로깅 건수가 100건 미만으로 통계적 유의성이 부족할 때, 부정확한 대시보드를 발행하지 않고 "데이터 부족" 경고를 COO에게 발송하여 **오해 유발 대시보드 0건**을 보장하는 **부정 시나리오(NAC)**를 검증한다.

### :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: [`SRS_V_1.0.md#REQ-FUNC-038`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/2_SRS_V_1.0.md) — §4.1.7 데이터 부족 NAC
- 선행 구현 태스크: [`3.0_Full TASKS list.md#E7-CMD-002`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/3.0_Full%20TASKS%20list.md) — 당월 로깅 <100건 시 "데이터 부족" 경고 + 미발행
- 알림 서비스: [`3.0_Full TASKS list.md#NOTI-001`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/3.0_Full%20TASKS%20list.md)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] 데이터 부족 기준치 검증: 당월 LOG_ENTRY 건수 < 100건 시 트리거
- [ ] 대시보드 미발행 검증: 100건 미만일 때 대시보드 데이터 생성이 실행되지 않음
- [ ] "데이터 부족" 경고 알림 검증: COO에게 경고 내용 + 현재 건수 + 최소 필요 건수 포함
- [ ] 경계값 테스트: 99건(미발행), 100건(발행), 101건(발행) 정확한 분기
- [ ] 오해 유발 대시보드 0건 검증: 미발행 시 이전 월 대시보드가 "최신"으로 오인되지 않는 처리

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 당월 로깅 50건 → "데이터 부족" 경고 + 대시보드 미발행**
- **Given**: 2026-04월 LOG_ENTRY 건수 = 50건 (기준치 100건 미만)
- **When**: 월말 자동 발행 트리거가 실행됨
- **Then**: 대시보드 데이터가 생성되지 않고, COO에게 "데이터 부족 — 당월 로깅 50건 (최소 100건 필요). 대시보드가 발행되지 않았습니다." 경고가 발송된다. **오해 유발 대시보드 = 0건**.

**Scenario 2: 경계값 99건 → 미발행**
- **Given**: 당월 LOG_ENTRY 건수 = 99건
- **When**: 트리거가 실행됨
- **Then**: 대시보드 미발행 + 경고 발송.

**Scenario 3: 경계값 100건 → 정상 발행**
- **Given**: 당월 LOG_ENTRY 건수 = 100건 (기준치 정확히 도달)
- **When**: 트리거가 실행됨
- **Then**: 대시보드가 정상 발행된다.

**Scenario 4: 미발행 시 이전 대시보드 "최신 표기" 방지**
- **Given**: 4월 대시보드가 미발행되고, 3월 대시보드가 존재함
- **When**: 사용자가 대시보드 페이지에 접근함
- **Then**: 3월 대시보드에 "최신 데이터가 아닙니다 (4월 미발행)" 표기가 추가되어, 오래된 데이터를 최신으로 오인하지 않도록 한다.

### :gear: Technical & Non-Functional Constraints
- **안전성**: 오해 유발 대시보드 **0건** — 부족한 데이터로 생성 금지
- **기준치**: 당월 LOG_ENTRY ≥100건 (미만 시 미발행)
- **알림**: COO 경고에 현재 건수 + 최소 기준 포함
- **테스트 도구**: Jest (단위/통합), ts-jest
- **테스트 위치**: `src/__tests__/e7/test-e7-004-data-insufficient.test.ts`

### :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria(Scenario 1~4)를 충족하는 자동화 테스트가 작성되었는가?
- [ ] 경계값(99/100/101)이 정확히 테스트되었는가?
- [ ] 미발행 시 이전 대시보드 오인 방지가 확인되었는가?
- [ ] SonarQube / ESLint 정적 분석 도구 경고가 없는가?

### :construction: Dependencies & Blockers
- **Depends on**: E7-CMD-002 (데이터 부족 경고 구현), E7-CMD-001 (자동 발행 트리거), NOTI-001 (알림)
- **Blocks**: 없음 (독립 NAC 테스트)

---

## TEST-E7-005

---
name: Test Task
about: SRS 기반의 NAC 검증 테스트 태스크 명세
title: "[Test] TEST-E7-005: 대시보드 렌더링 지연 시 재시도 + 안내 검증"
labels: 'test, backend, frontend, e7-dashboard, negative-ac, priority:medium'
assignees: ''
---

### :dart: Summary
- **기능명**: [TEST-E7-005] 렌더링 지연 → 재시도 3회 → "지연 안내" (에러 화면 0건) 테스트
- **목적**: 대시보드 렌더링이 5초 이내에 완료되지 않는 경우, 자동으로 3회 재시도하고, 그래도 실패하면 사용자에게 "지연 안내" 메시지를 표시하여 **에러 화면(빈 화면/크래시) 0건**을 보장하는 **부정 시나리오(NAC)**를 검증한다.

### :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: [`SRS_V_1.0.md#REQ-FUNC-039`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/2_SRS_V_1.0.md) — §4.1.7 렌더링 지연 NAC
- 선행 구현 태스크: [`3.0_Full TASKS list.md#E7-CMD-003`](file:///c:/Antigravity_Workspace/SRS%20from%20PRD_RPA%20Saas/Tasks/3.0_Full%20TASKS%20list.md) — 렌더링 5초 미완료 시 재시도 3회 → "지연 안내"

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] 렌더링 지연 시뮬레이션: API 응답을 6초+ 지연시키는 Mock 구성
- [ ] 자동 재시도 3회 검증: 1차 실패 → 2차 → 3차 자동 재시도 동작 확인
- [ ] 재시도 간격 검증: 재시도 간 적절한 간격(1초→2초→4초 Exponential Backoff)
- [ ] "지연 안내" 메시지 표시 검증: 3회 실패 후 사용자에게 안내 메시지 표시
- [ ] 에러 화면 0건 보증: 빈 화면, 크래시, Unhandled Error 표시 없음
- [ ] 큐 대기 안내: 재시도 중 "로딩 중" 진행률 표시기 표시

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 1차 렌더링 실패 → 자동 재시도 → 2차 성공**
- **Given**: 대시보드 데이터 API가 1차에서 6초 응답 (5초 초과), 2차에서 3초 정상 응답
- **When**: 사용자가 대시보드 페이지에 접근함
- **Then**: 1차 타임아웃 후 자동 재시도하여 2차에서 정상 렌더링 완료. 사용자에게는 "로딩 중" 표시만 보임.

**Scenario 2: 3회 연속 실패 → "지연 안내" 표시**
- **Given**: API가 3회 모두 6초+ 응답 (연속 타임아웃)
- **When**: 3차 재시도까지 실패함
- **Then**: "대시보드 생성에 시간이 걸리고 있습니다. 잠시 후 다시 접속해 주세요." 안내 메시지가 표시된다. **에러 화면(빈 화면/크래시) = 0건**.

**Scenario 3: 재시도 중 진행률 표시**
- **Given**: 1차 렌더링이 실패하여 재시도 중임
- **When**: 사용자가 화면을 보고 있음
- **Then**: "로딩 중... (재시도 2/3)" 형태의 진행률 표시기가 표시된다.

**Scenario 4: 정상 렌더링 → 재시도 없음 (대조군)**
- **Given**: API가 3초 내 정상 응답함
- **When**: 대시보드 페이지에 접근함
- **Then**: 1회 요청으로 정상 렌더링 완료. 재시도 로직은 발동하지 않음.

### :gear: Technical & Non-Functional Constraints
- **안정성**: 에러 화면(빈 화면/크래시) **0건** — 항상 안내 메시지 제공
- **재시도**: 최대 3회, Exponential Backoff (1s → 2s → 4s)
- **UX**: 재시도 중 진행률 표시, 최종 실패 시 한국어 안내
- **테스트 도구**: Jest (단위/통합), Playwright (E2E — UI 상태)
- **테스트 위치**:
  - `src/__tests__/e7/test-e7-005-render-retry.test.ts`
  - `e2e/e7/test-e7-005-render-delay.spec.ts`

### :checkered_flag: Definition of Done (DoD)
- [ ] 모든 Acceptance Criteria(Scenario 1~4)를 충족하는 자동화 테스트가 작성되었는가?
- [ ] 1회 성공, 2회 후 성공, 3회 전부 실패 3종 시나리오가 테스트되었는가?
- [ ] 에러 화면 0건이 E2E 테스트로 검증되었는가?
- [ ] SonarQube / ESLint 정적 분석 도구 경고가 없는가?

### :construction: Dependencies & Blockers
- **Depends on**: E7-CMD-003 (렌더링 재시도 구현), E7-CMD-001 (대시보드 데이터 생성)
- **Blocks**: NFR-PERF-004 (대시보드 렌더링 p95 ≤5,000ms 부하 테스트)

---

*본 문서는 SRS-002 Rev 2.0 (V0.8) 기반 E7 대시보드 테스트 태스크 5건(TEST-E7-001 ~ TEST-E7-005)의 GitHub Issue 상세명세서입니다.*  
*모든 테스트는 REQ-FUNC AC/NAC를 기반으로 도출되었으며, MVP = 클라우드 전용 SaaS 범위 내에서 정의되었습니다.*
