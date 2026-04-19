# FactoryAI — E4, E6, E7, HITL Epic Issues

> **Source**: SRS-002 Rev 2.0 (V0.8)  
> **작성일**: 2026-04-19  
> **총 Issue**: 30건 (E4: 8건 + E6: 7건 + E7: 8건 + HITL: 6건 + Query 1건)

---

# Part A — E4: CFO용 ROI 진단/결재기

> 8건 (Command 5건 + UI 3건)  
> **목적**: CFO(이재무)가 "Payback 18개월 이내" 투자 근거를 즉시 확인하고, 바우처 자부담 최소화를 시뮬레이션할 수 있는 독립형 ROI 계산기.

> [!IMPORTANT]
> E4는 **외부 의존성 없이 독립 실행 가능** (Good First Issues).  
> 인증 없이도 사용 가능한 공개 페이지(`/roi-calculator`)로 설계한다.

---

## E4-CMD-001: [Command] ROI 계산 엔진

---
name: Feature Task
title: "[Feature/E4] E4-CMD-001: ROI 계산 엔진 — 바우처 매칭 + 자부담 + 회수액 산출 (≤3초)"
labels: 'feature, backend, priority:must, epic:e4-roi'
assignees: ''
---

### :dart: Summary
- **기능명**: [E4-CMD-001] ROI 계산 엔진
- **목적**: 기업 규모(직원 수, 매출, 업종)를 입력하면 정부 바우처 매칭, 자부담 금액, 연간 절감액, Payback 기간을 ≤3초 내에 산출한다.

### :link: References (Spec & Context)
- SRS: REQ-FUNC-024
- API: API-009 (`POST /api/v1/roi/calculate`)
- 벤치마크 데이터: `MOCK-005` (금속가공/식품제조 JSON)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `lib/roi/roi-engine.ts` — 순수 함수 계산 엔진:
  ```typescript
  interface RoiInput {
    industry: 'METAL_PROCESSING' | 'FOOD_MANUFACTURING';
    employee_count: number;
    annual_revenue: number;      // 원
    current_missing_rate: number; // % (현재 결측률)
    erp_type: string;
  }
  
  interface RoiResult {
    voucher_amount: number;      // 바우처 지원금
    self_payment: number;        // 자부담 금액
    annual_savings: number;      // 연간 절감액
    payback_months: number;      // 회수 기간 (월)
    roi_percentage: number;      // ROI %
    breakdown: {
      labor_savings: number;
      quality_savings: number;
      audit_savings: number;
    };
  }
  ```
- [ ] **2.** 바우처 매칭 로직 (정부 바우처 프로그램별 조건):
  ```typescript
  function matchVoucher(input: RoiInput): VoucherMatch {
    // 스마트공장 바우처: 직원 50~300명, 매출 80억 이하
    // AI 바우처: 직원 제한 없음, 기술 도입 목적
    // 데이터 바우처: 데이터 수집/분석 목적
  }
  ```
- [ ] **3.** 절감액 산출: MOCK-005 벤치마크 데이터 기반
  ```typescript
  const benchmark = loadBenchmark(input.industry);
  const laborSavings = benchmark.cost_savings.annual_labor_savings * scaleFactor;
  const qualitySavings = benchmark.cost_savings.annual_quality_savings * scaleFactor;
  ```
- [ ] **4.** Payback 계산: `self_payment / (annual_savings / 12)`
- [ ] **5.** `POST /api/v1/roi/calculate` Route Handler (API-009 DTO 준수)

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 금속가공 85명 → ROI 산출**
- **Given**: 금속가공, 85명, 매출 50억
- **When**: ROI 계산을 요청한다
- **Then**: ≤3초 내 바우처 매칭 + 자부담 + Payback ≤18개월 결과 반환

**Scenario 2: 정확도 ≥90%**
- **Given**: 벤치마크 데이터 기반 기대 결과가 있다
- **When**: 동일 입력으로 계산한다
- **Then**: 결과가 기대값 대비 ±10% 이내

### :gear: Technical & Non-Functional Constraints
- **성능**: p95 ≤ 3초
- **정확도**: ≥ 90% (벤치마크 대비)
- **인증**: ROI 계산은 비인증 접근 허용 (공개 API)

### :construction: Dependencies & Blockers
- **Depends on**: `MOCK-005` (벤치마크 JSON)
- **Blocks**: `E4-CMD-004`, `E4-CMD-005`, `E4-UI-001`

---

## E4-CMD-002: [Command] 바우처 적합성 5항목 진단

---
title: "[Feature/E4] E4-CMD-002: 바우처 적합성 5항목 진단 (성공 확률% + 리스크 등급)"
labels: 'feature, backend, priority:must, epic:e4-roi'
---

### :dart: Summary
- **목적**: 기업이 바우처 선정될 확률을 5개 항목으로 진단하고, 항목별 High/Mid/Low 리스크 등급 + 종합 성공 확률(%)을 제공한다.

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** 진단 5항목 정의:
  | # | 항목 | 평가 기준 | 배점 |
  |:---:|:---|:---|:---:|
  | 1 | 기업 규모 적합성 | 직원 수 50~300명 = High, 그 외 = Low | 20 |
  | 2 | 매출 규모 | ≤80억 = High, 80~200 = Mid, >200 = Low | 20 |
  | 3 | 도입 목적 명확성 | 생산성/품질 = High, 기타 = Mid | 20 |
  | 4 | 기존 IT 인프라 | ERP 보유 = High, 미보유 = Mid | 20 |
  | 5 | 업종 우대 여부 | 제조업 = High, 서비스 = Low | 20 |
- [ ] **2.** 종합 점수 → 성공 확률 매핑: 80~100 = 90%, 60~79 = 70%, <60 = 50%
- [ ] **3.** `POST /api/v1/roi/voucher-fit` Route Handler (API-010)

### :test_tube: Acceptance Criteria (BDD/GWT)
- **Given**: 금속가공, 85명, 매출 50억, ERP 보유
- **When**: 적합성 진단
- **Then**: 5항목 모두 High, 성공 확률 90%

### :construction: Dependencies & Blockers
- **Depends on**: `AUTH-002` (선택적 인증)
- **Blocks**: `E4-UI-002`

---

## E4-CMD-003: [Command] Before-After 카드 생성

---
title: "[Feature/E4] E4-CMD-003: Before-After 카드 생성 (동종 업종 데이터 기반, ≤10초)"
labels: 'feature, backend, priority:should, epic:e4-roi'
---

### :dart: Summary
- **목적**: MOCK-005 벤치마크 → 동종 업종의 도입 전/후 핵심 KPI 비교 카드 데이터를 생성한다.

### :white_check_mark: Task Breakdown
- [ ] `POST /api/v1/roi/ba-card` Route Handler (API-011)
- [ ] 벤치마크 metrics를 카드 포맷으로 변환:
  ```json
  { "metric": "결측률", "before": "42.5%", "after": "3.8%", "improvement": "91% 개선" }
  ```

### :construction: Dependencies & Blockers
- **Depends on**: `MOCK-005`
- **Blocks**: `E4-UI-003`

---

## E4-CMD-004: [Command] 필수 입력 검증 + 계산 차단

---
title: "[Feature/E4] E4-CMD-004: 필수 입력 누락 시 하이라이팅 + 계산 차단"
labels: 'feature, backend, priority:should, epic:e4-roi'
---

### :dart: Summary
- **목적**: REQ-FUNC-027. `industry`, `employee_count`, `annual_revenue` 중 하나라도 누락 시 400 에러 + 누락 필드 배열 반환. 감사 로그(AUTH-003) 기록.

### :test_tube: Acceptance Criteria
- **Given**: `employee_count` 미입력
- **When**: ROI 계산 요청
- **Then**: 400 + `{ missing_fields: ["employee_count"], message: "직원 수를 입력해 주세요" }`

---

## E4-CMD-005: [Command] 비현실적 수치 검증

---
title: "[Feature/E4] E4-CMD-005: 비현실적 수치 경고 (매출 0원/직원 0명)"
labels: 'feature, backend, priority:low, epic:e4-roi'
---

### :dart: Summary
- **목적**: REQ-FUNC-028. `employee_count=0`, `annual_revenue=0`, 또는 직원 1명에 매출 1조 같은 비현실적 조합을 감지하여 경고 + 재확인 요청. 이상 패턴 로그 기록.

---

## E4-UI-001 ~ E4-UI-003: ROI 프론트엔드

---
title: "[Feature/E4] E4-UI-001: ROI 웹 계산기 (CLI-02) 메인 페이지"
labels: 'feature, frontend, ui, priority:must, epic:e4-roi'
---

### :dart: Summary — E4-UI-001
- **목적**: `/roi-calculator` 공개 페이지. 기업 규모 입력 → 바우처 매칭 → Payback 시뮬레이션 시각화.
- **실행**:
  - [ ] 입력 폼: 업종 SelectBox, 직원 수 NumberInput, 매출 NumberInput
  - [ ] 결과 영역: 바우처 금액, 자부담, 연간 절감액, Payback 기간 카드
  - [ ] Payback 시뮬레이션: Recharts AreaChart (월별 누적 절감액 vs 자부담 교차점)
  - [ ] 누락 필드 하이라이팅 (E4-CMD-004 연동)

---
### :dart: Summary — E4-UI-002
- **목적**: 바우처 적합성 진단 결과 — 5항목 리스크 카드 + 성공 확률 게이지 (RadialBarChart)

---
### :dart: Summary — E4-UI-003
- **목적**: Before-After 카드 시각화 — 도입 전/후 비교 카드 그리드 (6개 KPI × 2열)

---

# Part B — E6: 보안 패키지

> 7건 (Command 3건 + Query 2건 + UI 2건)  
> **목적**: CISO(최보안)의 "단독 거부권"을 기술적으로 보장. RBAC, 감사 로그, 네트워크 모니터링의 통합 보안 콘솔.

---

## E6-CMD-001: [Command] Supabase RLS 정책 설정

---
title: "[Feature/E6] E6-CMD-001: Supabase RLS 정책 — 역할별 데이터 접근 범위 제한"
labels: 'feature, backend, security, priority:must, epic:e6-security'
---

### :dart: Summary
- **기능명**: [E6-CMD-001] Row Level Security 정책 설정
- **목적**: Supabase PostgreSQL의 RLS를 활용하여, 각 역할(ADMIN/OPERATOR/AUDITOR/VIEWER/CISO)이 자기 `factory_id` 소속 데이터만 접근할 수 있도록 DB 레벨에서 격리한다.

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** 모든 주요 테이블에 RLS 활성화:
  ```sql
  ALTER TABLE "LogEntry" ENABLE ROW LEVEL SECURITY;
  ALTER TABLE "Lot" ENABLE ROW LEVEL SECURITY;
  ALTER TABLE "AuditReport" ENABLE ROW LEVEL SECURITY;
  -- ... (15개 테이블 전체)
  ```
- [ ] **2.** Factory Scope 정책 (공통):
  ```sql
  CREATE POLICY "factory_scope" ON "LogEntry"
    FOR ALL
    USING (factory_id = current_setting('app.factory_id')::uuid);
  ```
- [ ] **3.** CISO 전용 확장 정책 (감사 로그 전체 조회):
  ```sql
  CREATE POLICY "ciso_audit_full" ON "AuditLog"
    FOR SELECT
    USING (current_setting('app.user_role') = 'CISO');
  ```
- [ ] **4.** Prisma 연결 시 `app.factory_id`, `app.user_role` 세팅 주입:
  ```typescript
  await prisma.$executeRaw`SELECT set_config('app.factory_id', ${factoryId}, true)`;
  await prisma.$executeRaw`SELECT set_config('app.user_role', ${role}, true)`;
  ```
- [ ] **5.** Mock ERP 테이블 Write 차단 RLS (E3-CMD-001 연동)

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: Factory Scope 격리**
- **Given**: `factory_A` 소속 OPERATOR
- **When**: `SELECT * FROM "LogEntry"` 실행
- **Then**: `factory_A`의 데이터만 반환, `factory_B` 데이터 0건

**Scenario 2: CISO 감사 로그 전체 조회**
- **Given**: CISO 역할 사용자
- **When**: `SELECT * FROM "AuditLog"` 실행
- **Then**: 전 공장의 감사 로그 전체 반환

### :construction: Dependencies & Blockers
- **Depends on**: `DB-001` ~ `DB-015`, `AUTH-002`
- **Blocks**: `E6-CMD-002`, `E6-QRY-001`

---

## E6-CMD-002: [Command] 보안 체크리스트 자동 생성

---
title: "[Feature/E6] E6-CMD-002: 클라우드 보안 체크리스트 자동 생성"
labels: 'feature, backend, priority:should, epic:e6-security'
---

### :dart: Summary
- **목적**: HTTPS, RLS, 감사 로그, RBAC, 암호화 등 현재 시스템 보안 상태를 자동 수집하여 체크리스트 리포트를 생성한다. CISO가 경영진/원청사에 제출하는 증빙용.

### :white_check_mark: Task Breakdown
- [ ] 체크항목 8개 자동 검증:
  | # | 항목 | 검증 방법 | 상태 |
  |:---:|:---|:---|:---:|
  | 1 | HTTPS 적용 | `req.headers.get('x-forwarded-proto')` | ✅/❌ |
  | 2 | RLS 활성화 | `pg_tables` 조회 | ✅/❌ |
  | 3 | 감사 로그 활성 | AUDIT_LOG 최근 24h 건수 | ✅/❌ |
  | 4 | RBAC 5역할 | ACCESS_MATRIX 키 수 | ✅/❌ |
  | 5 | 비밀번호 암호화 | bcrypt 해시 패턴 검증 | ✅/❌ |
  | 6 | JWT 만료 설정 | `NEXTAUTH_SECRET` 존재 | ✅/❌ |
  | 7 | ERP Write 차단 | Prisma MW 등록 여부 | ✅/❌ |
  | 8 | 환경변수 보호 | `.env.local` 미노출 | ✅/❌ |
- [ ] 결과를 JSON + PDF(선택) 형태로 반환

---

## E6-CMD-003: [Command] 이상 접근 감지 → CISO 알림

---
title: "[Feature/E6] E6-CMD-003: 이상 접근 감지 → 감사 로그 + CISO 알림 ≤10초"
labels: 'feature, backend, security, priority:must, epic:e6-security'
---

### :dart: Summary
- **목적**: AUTH-004에서 구축한 보안 이벤트 파이프라인을 E6 보안 콘솔 차원에서 통합한다. 이상 접근(비인가, Factory Scope 위반, Write 시도 등) 감지 시 CISO 알림을 발송하고, 보안 콘솔에서 실시간 확인할 수 있게 한다. (AUTH-004의 E6 측 소비자.)
- **실행**: AUTH-004의 `securityEventEmitter`를 구독하여, E6-UI-001 대시보드에 실시간 반영.

---

## E6-QRY-001: [Query] 감사 로그 조회 API

---
title: "[Feature/E6] E6-QRY-001: 감사 로그 전수 조회 (필터: factory, date, event_type)"
labels: 'feature, backend, query, priority:must, epic:e6-security'
---

### :dart: Summary
- **목적**: AUDIT_LOG 테이블의 전수 기록을 CISO가 필터링하여 조회. factory_id, date_range, event_type, user_id 기반 검색.
- **실행**: `GET /api/v1/audit-logs` (API-018 DTO 준수) + Pagination.
- **제약**: CISO + ADMIN 역할만 접근 가능.

---

## E6-QRY-002: [Query] 네트워크 모니터링 상태 조회

---
title: "[Feature/E6] E6-QRY-002: 네트워크 모니터링 — 외부 트래픽 바이트 수 (참고 메트릭)"
labels: 'feature, backend, query, priority:low, epic:e6-security'
---

### :dart: Summary
- **목적**: MVP에서는 실제 네트워크 모니터링 대신, Vercel Analytics 또는 Supabase 대시보드에서 수집 가능한 참고 메트릭(24h 트래픽 바이트 수, API 호출 수)을 표시한다.
- **실행**: `GET /api/v1/security/network-status` (API-012). MVP는 하드코딩 또는 Vercel API 기반.

---

## E6-UI-001 ~ E6-UI-002: 보안 콘솔 UI

---
title: "[Feature/E6] E6-UI-001: CISO 보안 콘솔 (CLI-03)"
labels: 'feature, frontend, ui, priority:must, epic:e6-security'
---

### :dart: Summary — E6-UI-001
- **목적**: CISO 전용 보안 관리 대시보드. 감사 로그 검색, RBAC 현황, 보안 이벤트 실시간 피드, 네트워크 참고 메트릭.
- **실행**:
  - [ ] 페이지: `/dashboard/security` (CISO + ADMIN 전용)
  - [ ] **감사 로그 탐색기**: DataTable + 필터 (날짜/이벤트/사용자) + 검색
  - [ ] **보안 이벤트 실시간 피드**: 최신 10건 타임라인 (30초 폴링)
  - [ ] **RBAC 현황 카드**: 역할별 사용자 수, 최근 접근 거부 건수
  - [ ] **네트워크 메트릭**: 24h 트래픽 게이지 (참고용)

### :dart: Summary — E6-UI-002
- **목적**: 보안 체크리스트 결과 표시 (8항목 ✅/❌) + PDF 다운로드 버튼

---

# Part C — E7: 성과 가시화/리텐션 대시보드

> 8건 (Command 4건 + Query 2건 + UI 2건)  
> **목적**: 월말 자동 발행 4인 맞춤 대시보드로 고객 성과를 가시화하고, NPS를 통한 레퍼런스 확보.

---

## E7-CMD-001: [Command] 월말 자동 대시보드 발행

---
title: "[Feature/E7] E7-CMD-001: 월말 자동 발행 — 4인 맞춤 대시보드 데이터 생성"
labels: 'feature, backend, priority:should, epic:e7-dashboard'
---

### :dart: Summary
- **기능명**: [E7-CMD-001] 성과 대시보드 자동 발행
- **목적**: 월말 마감 시 4인(COO/구매본부장/품질이사/CFO) 페르소나별 맞춤 데이터를 자동 집계하여 대시보드를 발행한다.

### :white_check_mark: Task Breakdown
- [ ] **1.** `POST /api/v1/dashboards/publish` Route Handler (API-016)
- [ ] **2.** 4인 페르소나별 데이터 집계:
  | 페르소나 | 핵심 지표 |
  |:---|:---|
  | COO | 이번 달 로깅 건수, 결측률, 라인별 가동률 |
  | 구매본부장 | 감사 리포트 발행 수, 규제 준수율, 미승인 건 |
  | 품질이사 | 이상 감지 건수, XAI 판단 정확도, 불량률 추이 |
  | CFO | 절감액 누적, 바우처 소진율, ROI 달성률 |
- [ ] **3.** `DASHBOARD` 테이블(DB-015)에 JSON 데이터 저장
- [ ] **4.** 발행 완료 알림 (NOTI-001): 4인 각각에게 맞춤 알림 발송

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 정상 발행**
- **Given**: 당월 로깅 건수 ≥100건
- **When**: 월말 발행 트리거 실행
- **Then**: 4인 맞춤 대시보드 데이터 생성 + 알림 발송

**Scenario 2: 데이터 부족 시 미발행 (E7-CMD-002 연동)**
- **Given**: 당월 로깅 건수 < 100건
- **When**: 발행 트리거 실행
- **Then**: 대시보드 미발행 + "데이터 부족" 경고 + COO 알림

### :construction: Dependencies & Blockers
- **Depends on**: `DB-015`, `DB-007`, `DB-010`, `AUTH-002`
- **Blocks**: `E7-CMD-002`, `E7-CMD-003`, `E7-QRY-002`, `E7-UI-001`

---

## E7-CMD-002: [Command] "데이터 부족" 경고 + 미발행

---
title: "[Feature/E7] E7-CMD-002: 당월 로깅 <100건 — 데이터 부족 경고 + 대시보드 미발행"
labels: 'feature, backend, priority:should, epic:e7-dashboard'
---

### :dart: Summary
- **목적**: REQ-FUNC-038. 충분한 데이터 없이 대시보드를 만들면 오해를 유발. 100건 미만 시 미발행 + COO 알림.
- **실행**: E7-CMD-001 진입 시 `SELECT COUNT(*) FROM "LogEntry" WHERE captured_at >= 이번달 1일` 체크.

---

## E7-CMD-003: [Command] 대시보드 렌더링 지연 대응

---
title: "[Feature/E7] E7-CMD-003: 대시보드 렌더링 5초 미완료 → 재시도 3회 → 지연 안내"
labels: 'feature, backend, priority:should, epic:e7-dashboard'
---

### :dart: Summary
- **목적**: REQ-FUNC-039. 서버 부하 시 렌더링 지연 발생 → `retry(3, backoff)` → 최종 실패 시 "지연 안내" 메시지 + 큐 대기.
- **실행**: `lib/dashboard/render-with-retry.ts` — 1회, 2초, 4초 백오프 후 최종 큐잉.

---

## E7-CMD-004: [Command] NPS 설문 발송

---
title: "[Feature/E7] E7-CMD-004: NPS 9~10점 감지 시 1클릭 NPS + 레퍼런스 동의 수집"
labels: 'feature, backend, priority:should, epic:e7-dashboard'
---

### :dart: Summary
- **목적**: REQ-FUNC-037. NPS 9~10점 고객에게 레퍼런스 동의 요청을 자동 발송. 응답률 ≥30% 목표.
- **실행**:
  - [ ] `POST /api/v1/nps/survey` Route Handler (API-017)
  - [ ] NPS 점수 감지: DASHBOARD 발행 후 설문 자동 발송
  - [ ] 1클릭 응답: "추천 의사 있음 + 레퍼런스 허용" 단일 버튼
  - [ ] 결과 DB 저장 + 응답률 집계

---

## E7-QRY-001 ~ E7-QRY-002: 조회 API

---

### :dart: Summary — E7-QRY-001
- **목적**: 분기 말 ROI 누적 리포트 — 절감액/생성건수 자동 집계 (수동 개입 0건). `GET /api/v1/dashboards/roi-summary`

### :dart: Summary — E7-QRY-002
- **목적**: 대시보드 이력/상세 조회 (persona_type 필터). `GET /api/v1/dashboards?persona=COO`

---

## E7-UI-001 ~ E7-UI-002: 대시보드 UI

---
title: "[Feature/E7] E7-UI-001: 성과 대시보드 페이지 (4인 탭)"
labels: 'feature, frontend, ui, priority:should, epic:e7-dashboard'
---

### :dart: Summary — E7-UI-001
- **목적**: `/dashboard/performance` — 4인 페르소나별 탭 전환 + 핵심 KPI 카드 + 추이 차트 (Recharts). 렌더링 ≤ 5초. 지연 시 AI-003 프로그레스 표시기 연동.
- **실행**:
  - [ ] 탭 4개: COO | 구매본부장 | 품질이사 | CFO
  - [ ] KPI 서머리 카드 (4~6개)
  - [ ] 월별 추이 LineChart
  - [ ] "데이터 부족" 경고 배너 (E7-CMD-002 연동)

### :dart: Summary — E7-UI-002
- **목적**: NPS 설문 UI — 1~10 스케일 클릭 + 레퍼런스 동의 체크박스 + 제출 1클릭.

---

# Part D — HITL: 공통 안전 프로토콜

> 6건 (Command 5건 + Query 1건)  
> **목적**: "AI 단독 실행 0건" 원칙을 시스템 전체에서 기술적으로 강제하는 공통 안전 프레임워크.

> [!IMPORTANT]
> HITL은 E1, E2, E2B, E6 전반에 걸쳐 적용되는 **횡단 관심사(Cross-Cutting Concern)** 입니다.  
> 모든 외부 발행(감사 리포트, 대시보드 등)은 HITL 게이트를 반드시 통과해야 합니다.

---

## HITL-CMD-001: [Command] PENDING 외부 발행 자동 차단

---
title: "[Feature/HITL] HITL-CMD-001: APPROVAL status=PENDING 외부 발행 자동 차단 + 알림 ≤10초"
labels: 'feature, backend, security, priority:must, epic:hitl-safety'
---

### :dart: Summary
- **기능명**: [HITL-CMD-001] PENDING 상태 외부 발행 차단 (Fail-Close)
- **목적**: `APPROVAL` 테이블에서 `status=PENDING`인 항목이 외부로 발행되는 것을 시스템 레벨에서 원천 차단한다. AUTH-003 감사 로그 + CISO 알림 ≤10초.

### :link: References (Spec & Context)
- SRS: REQ-FUNC-040

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `lib/hitl/publication-guard.ts` — 발행 게이트:
  ```typescript
  export async function assertApproved(entityType: string, entityId: string): Promise<void> {
    const approval = await prisma.approval.findFirst({
      where: { entity_type: entityType, entity_id: entityId },
      orderBy: { created_at: 'desc' },
    });
    
    if (!approval || approval.status === 'PENDING') {
      // 보안 이벤트
      securityEventEmitter.emit('security', {
        type: 'PUBLICATION_BLOCKED',
        severity: 'CRITICAL',
        resource: `${entityType}/${entityId}`,
      });
      throw new PublicationBlockedError(`미승인 항목 발행 차단: ${entityType}/${entityId}`);
    }
    
    if (approval.status === 'REJECTED') {
      throw new PublicationBlockedError(`거절된 항목 발행 차단: ${entityType}/${entityId}`);
    }
  }
  ```
- [ ] **2.** 발행 체크포인트 삽입:
  - 감사 리포트 PDF 생성 전 → `assertApproved('AUDIT_REPORT', reportId)`
  - 대시보드 외부 공유 전 → `assertApproved('DASHBOARD', dashboardId)`
- [ ] **3.** 차단 시 관리자 알림 ≤10초 (AUTH-004 보안 이벤트 연동)

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: PENDING 리포트 발행 시도**
- **Given**: `status=PENDING`인 AUDIT_REPORT
- **When**: PDF 다운로드 또는 외부 전송을 시도한다
- **Then**: 즉시 차단 + `PUBLICATION_BLOCKED` 이벤트 + 관리자 알림 ≤10초

**Scenario 2: APPROVED 리포트 정상 발행**
- **Given**: `status=APPROVED`인 AUDIT_REPORT
- **When**: PDF 다운로드를 시도한다
- **Then**: 정상 다운로드 허용

### :construction: Dependencies & Blockers
- **Depends on**: `DB-008` (APPROVAL), `NOTI-001`
- **Blocks**: `E2-CMD-007` (감사 리포트 승인), `E7-CMD-001` (대시보드 발행)

---

## HITL-CMD-002: [Command] XAI NULL 시 리포트 발행 차단

---
title: "[Feature/HITL] HITL-CMD-002: xai_explanation=NULL → 리포트 발행 절대 차단"
labels: 'feature, backend, security, priority:must, epic:hitl-safety'
---

### :dart: Summary
- **목적**: REQ-FUNC-041. `AUDIT_REPORT.xai_explanation`이 NULL인 경우, 어떤 상황에서도 리포트가 발행되지 않도록 DB + 애플리케이션 이중 차단.

### :white_check_mark: Task Breakdown
- [ ] **1.** DB 레벨: `AUDIT_REPORT.xai_explanation NOT NULL` 제약 조건 (DB-010에서 이미 설정)
- [ ] **2.** 애플리케이션 레벨: `assertApproved()` 내 추가 검증:
  ```typescript
  if (entityType === 'AUDIT_REPORT') {
    const report = await prisma.auditReport.findUnique({ where: { id: entityId } });
    if (!report?.xai_explanation) {
      // 개발팀 알림 (시스템 버그 의심)
      throw new HitlViolationError('XAI 설명 없는 리포트 발행 시도 — 개발팀 확인 필요');
    }
  }
  ```
- [ ] **3.** 개발팀 알림: severity=CRITICAL, NOTI-001 → ADMIN

---

## HITL-CMD-003: [Command] API 게이트웨이 승인 검증

---
title: "[Feature/HITL] HITL-CMD-003: action_type ∈ {STOP, CHANGE} — approval_id 필수 검증"
labels: 'feature, backend, security, priority:must, epic:hitl-safety'
---

### :dart: Summary
- **목적**: REQ-FUNC-043. 공정 중지(STOP) 또는 설정 변경(CHANGE) 같은 고위험 액션 API 호출 시, 반드시 유효한 `approval_id`가 포함되어야 한다. 무승인 요청 → 즉시 차단 + 감사 로그 ≤1초.

### :white_check_mark: Task Breakdown
- [ ] **1.** `lib/hitl/action-guard.ts`:
  ```typescript
  export async function assertActionApproved(
    actionType: 'STOP' | 'CHANGE',
    approvalId: string | undefined
  ): Promise<void> {
    if (!approvalId) {
      throw new HitlViolationError(`${actionType} 액션에 approval_id 누락`);
    }
    const approval = await prisma.approval.findUnique({ where: { id: approvalId } });
    if (!approval || approval.status !== 'APPROVED') {
      throw new HitlViolationError(`무효한 approval: ${approvalId}`);
    }
  }
  ```
- [ ] **2.** Route Handler 데코레이터 통합: `withAuth()` + `withActionGuard()` 체이닝

### :test_tube: Acceptance Criteria
- **Given**: `action_type=STOP`, `approval_id` 미포함
- **When**: API 호출
- **Then**: 즉시 차단 + 감사 로그 ≤1초 + CISO 자동 통보

---

## HITL-CMD-004: [Command] 30분 미처리 PENDING 에스컬레이션

---
title: "[Feature/HITL] HITL-CMD-004: 30분 미처리 PENDING → COO 자동 에스컬레이션"
labels: 'feature, backend, priority:must, epic:hitl-safety'
---

### :dart: Summary
- **목적**: REQ-FUNC-044. APPROVAL status=PENDING이 30분 경과 → COO에게 에스컬레이션 알림 자동 발송. (E2B-CMD-004와 유사하나, 이것은 전체 HITL 시스템 대상 범용 스케줄러.)

### :white_check_mark: Task Breakdown
- [ ] **1.** `lib/hitl/escalation-cron.ts` — 30분 주기 미처리 건 스캔:
  ```typescript
  // 30분 이상 PENDING인 모든 APPROVAL 조회
  const staleApprovals = await prisma.approval.findMany({
    where: {
      status: 'PENDING',
      created_at: { lt: subMinutes(new Date(), 30) },
      escalated_at: null, // 미에스컬레이션 건만
    },
  });
  
  for (const approval of staleApprovals) {
    await escalateToCOO(approval);
    await prisma.approval.update({
      where: { id: approval.id },
      data: { escalated_at: new Date() },
    });
  }
  ```
- [ ] **2.** MVP: `setInterval(30 * 60 * 1000)` 기반. Phase 2: Vercel Cron.
- [ ] **3.** 중복 에스컬레이션 방지 (`escalated_at` NOT NULL 체크)

---

## HITL-CMD-005: [Command] Reject 후 원본 데이터 무결성 검증

---
title: "[Feature/HITL] HITL-CMD-005: LOG_ENTRY Reject 후 원본 데이터 보존 일일 검증"
labels: 'feature, backend, priority:should, epic:hitl-safety'
---

### :dart: Summary
- **목적**: REQ-FUNC-045. LOG_ENTRY가 REJECTED 처리되어도 원본 `raw_data`가 100% 보존되어야 한다. 일일 1회 무결성 검증(해시 비교)을 수행하고 위반 시 ≤10초 알림.

### :white_check_mark: Task Breakdown
- [ ] **1.** `lib/hitl/integrity-checker.ts` — 일일 검증 스크립트:
  ```typescript
  // REJECTED 건의 raw_data가 생성 시점 해시와 일치하는지 확인
  const rejected = await prisma.logEntry.findMany({ where: { status: 'REJECTED' } });
  for (const entry of rejected) {
    const currentHash = sha256(JSON.stringify(entry.raw_data));
    if (currentHash !== entry.raw_data_hash) {
      // 무결성 위반 감지
      await notifyCritical('DATA_INTEGRITY_VIOLATION', entry.id);
    }
  }
  ```
- [ ] **2.** `LOG_ENTRY` 생성 시 `raw_data_hash` 필드에 SHA-256 해시 저장
- [ ] **3.** 위반 시 CISO + ADMIN 알림 (severity=CRITICAL)

---

## HITL-QRY-001: [Query] PENDING 승인 건 목록 조회

---
title: "[Feature/HITL] HITL-QRY-001: PENDING 승인 건 목록 조회 (age_minutes, 에스컬레이션 상태)"
labels: 'feature, backend, query, priority:must, epic:hitl-safety'
---

### :dart: Summary
- **목적**: 현재 처리 대기 중인 PENDING 건 목록을 경과 시간(age_minutes) 기준으로 정렬하여 조회한다. 에스컬레이션 여부도 포함.
- **실행**: `GET /api/v1/approvals/pending` (API-015 DTO 준수)
- **응답 예시**:
  ```json
  {
    "items": [
      {
        "id": "...",
        "entity_type": "AUDIT_REPORT",
        "entity_id": "...",
        "age_minutes": 45,
        "is_escalated": true,
        "assigned_to": { "name": "차품질", "role": "AUDITOR" }
      }
    ],
    "total_pending": 3,
    "escalated_count": 1
  }
  ```

### :construction: Dependencies & Blockers
- **Depends on**: `DB-008`, `AUTH-002`
- **Blocks**: `E2B-UI-001` (XAI 대시보드 PENDING 뱃지)

---

## 전체 실행 순서 요약

### E4 — ROI 진단기 (~14h)
| 순서 | Task | 예상 |
|:---:|:---|:---:|
| 1 | E4-CMD-001 ROI 엔진 | 3h |
| 2 | E4-CMD-002 적합성 진단 | 2h |
| 3 | E4-CMD-003 B/A 카드 | 1.5h |
| 4 | E4-CMD-004/005 검증 | 1.5h |
| 5 | E4-UI-001~003 | 6h |

### E6 — 보안 패키지 (~14h)
| 순서 | Task | 예상 |
|:---:|:---|:---:|
| 1 | E6-CMD-001 RLS | 4h |
| 2 | E6-CMD-002 체크리스트 | 2h |
| 3 | E6-CMD-003 이상 감지 | 1.5h |
| 4 | E6-QRY-001/002 | 2.5h |
| 5 | E6-UI-001/002 | 4h |

### E7 — 성과 대시보드 (~14h)
| 순서 | Task | 예상 |
|:---:|:---|:---:|
| 1 | E7-CMD-001 발행 엔진 | 3h |
| 2 | E7-CMD-002 데이터 부족 | 1h |
| 3 | E7-CMD-003 재시도 | 1h |
| 4 | E7-CMD-004 NPS | 2h |
| 5 | E7-QRY-001/002 | 2h |
| 6 | E7-UI-001/002 | 5h |

### HITL — 안전 프로토콜 (~10h)
| 순서 | Task | 예상 |
|:---:|:---|:---:|
| 1 | HITL-CMD-001 발행 차단 | 2h |
| 2 | HITL-CMD-002 XAI NULL 차단 | 1h |
| 3 | HITL-CMD-003 액션 가드 | 2h |
| 4 | HITL-CMD-004 에스컬레이션 | 1.5h |
| 5 | HITL-CMD-005 무결성 검증 | 2h |
| 6 | HITL-QRY-001 PENDING 조회 | 1.5h |

**4개 Epic 총 예상: ~52h**
