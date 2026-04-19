# FactoryAI — Step 4 NFR + SVC + DevOps Issues

> **Source**: SRS-002 Rev 2.0 (V0.8) — Step 4. 비기능 제약(NFR) + SVC + DevOps  
> **작성일**: 2026-04-19  
> **총 Issue**: 50건 (PERF:8 + AVAIL/REL:6 + SEC:5 + MON:6 + SCALE:3 + SVC:14 + INIT:4 + 기타:4)

---

# 4-A. 성능 (Performance) — 8건

---

## NFR-PERF-001: STT p95 ≤5,000ms 성능 테스트

---
title: "[NFR/Perf] NFR-PERF-001: STT p95 ≤5,000ms 성능 테스트 (2건/분 Throttle)"
labels: 'nfr, performance, test, priority:must, epic:infra'
---

### :dart: Summary
- **기능명**: [NFR-PERF-001] STT 성능 벤치마크
- **목적**: AI-002 Rate Limiter(2건/분)를 적용한 환경에서 STT(음성→텍스트) 파이프라인의 p95 응답 시간이 5,000ms 이하를 유지하는지 반복 측정한다.

### :link: References
- SRS: REQ-NF-001

### :white_check_mark: Task Breakdown
- [ ] **1.** `scripts/perf/stt-benchmark.ts` 작성:
  ```typescript
  // 20건 순차 요청 → 응답 시간 배열 수집 → p95 계산
  const results: number[] = [];
  for (let i = 0; i < 20; i++) {
    const start = performance.now();
    await callSttAction(testAudioFixture);
    results.push(performance.now() - start);
  }
  const p95 = percentile(results, 95);
  assert(p95 <= 5000, `STT p95 ${p95}ms > 5000ms`);
  ```
- [ ] **2.** Throttle 시뮬레이션: AI-002의 12 RPM 제한 적용 상태에서 실행
- [ ] **3.** CI 리포트 출력: min / avg / p50 / p95 / max 통계

### :test_tube: Acceptance Criteria
- **Given**: 20건 테스트 오디오 → 2건/분 Throttle
- **Then**: p95 ≤ 5,000ms

### :construction: Dependencies & Blockers
- **Depends on**: `E1-CMD-001`

---

## NFR-PERF-002: Vision p95 ≤8,000ms 성능 테스트

---
title: "[NFR/Perf] NFR-PERF-002: Vision p95 ≤8,000ms 성능 테스트"
labels: 'nfr, performance, test, priority:must, epic:infra'
---

### :dart: Summary
- **목적**: Vision(이미지→JSON) 파이프라인의 p95 응답 시간 ≤8,000ms 검증. STT 대비 이미지 처리가 무거우므로 한도가 넓다.
- **실행**: NFR-PERF-001과 동일 구조. 테스트 이미지 픽스처 20건 사용.

### :construction: Dependencies
- **Depends on**: `E1-CMD-002`

---

## NFR-PERF-003: 클라이언트 PDF 렌더링 시간 테스트

---
title: "[NFR/Perf] NFR-PERF-003: 클라이언트 PDF 생성 (100 Lot) 브라우저 렌더링 시간"
labels: 'nfr, performance, test, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: `@react-pdf/renderer`로 100개 Lot 데이터를 포함한 PDF를 브라우저에서 생성할 때 렌더링 시간을 측정한다. 목표: 10초 이내 Blob 생성.

### :white_check_mark: Task Breakdown
- [ ] Playwright 브라우저 테스트: PDF 생성 버튼 클릭 → Blob URL 생성까지 경과 시간 측정
- [ ] 100 Lot 픽스처 데이터 준비

### :construction: Dependencies
- **Depends on**: `E2-CMD-002`

---

## NFR-PERF-004: 대시보드 렌더링 동시 3명 부하 테스트

---
title: "[NFR/Perf] NFR-PERF-004: 대시보드 렌더링 p95 ≤5,000ms (동시접속 3명)"
labels: 'nfr, performance, test, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: 3명이 동시에 성과 대시보드를 로딩할 때 p95 렌더링 시간 ≤5초 보장.
- **실행**: `k6` 또는 커스텀 스크립트로 3 concurrent sessions 시뮬레이션.

### :construction: Dependencies
- **Depends on**: `E7-UI-001`

---

## NFR-PERF-005: XAI 설명 생성 p95 ≤8,000ms

---
title: "[NFR/Perf] NFR-PERF-005: XAI 설명 생성 p95 ≤8,000ms"
labels: 'nfr, performance, test, priority:must, epic:infra'
---

### :dart: Summary
- **목적**: XAI 한국어 설명 생성(E2B-CMD-001) + AI-002 큐 대기 포함 응답 시간 벤치마크.
- **실행**: 20건 이상 징후 데이터 순차 투입 → p95 계산.

### :construction: Dependencies
- **Depends on**: `E2B-CMD-001`

---

## NFR-PERF-006: 복합 부하 테스트 (동시 3명)

---
title: "[NFR/Perf] NFR-PERF-006: 동시접속 3명 복합 부하 — 전체 p95 유지 검증"
labels: 'nfr, performance, test, priority:must, epic:infra'
---

### :dart: Summary
- **목적**: STT + Vision + XAI + 대시보드를 3명이 동시에 사용하는 복합 시나리오에서 개별 p95 기준이 모두 유지되는지 검증.

### :white_check_mark: Task Breakdown
- [ ] 복합 시나리오 스크립트:
  - User A: STT 3건 연속
  - User B: Vision 2건 + XAI 1건
  - User C: 대시보드 로딩 5회
- [ ] 결과 매트릭스 리포트 출력

### :construction: Dependencies
- **Depends on**: `NFR-PERF-001` ~ `NFR-PERF-005` 전체

---

## NFR-PERF-007: Gemini Throttle 큐 드롭 0건 통합 테스트

---
title: "[NFR/Perf] NFR-PERF-007: Gemini Throttle 큐 드롭 0건 + 최대 대기 60초 + 진행률 UI"
labels: 'nfr, performance, test, priority:must, epic:infra'
---

### :dart: Summary
- **목적**: AI-002 Rate Limiter + AI-003 UI가 통합된 상태에서 15 RPM 초과 요청 시 ①큐 드롭 0건 ②최대 대기 60초 ③진행률 텍스트 전환 검증.

### :white_check_mark: Task Breakdown
- [ ] Mock AI (강제 3초 지연) + 20건 동시 요청
- [ ] 프론트엔드 Playwright: 15초 경과 시 텍스트 전환 어서션
- [ ] 유실 건수 = `total_requests - successful_responses` = 0 검증

---

## NFR-PERF-008: Supabase Free 용량 모니터링

---
title: "[NFR/Perf] NFR-PERF-008: Supabase Free 500MB DB + 1GB Storage 용량 모니터링"
labels: 'nfr, monitoring, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: Supabase Free Tier 용량 한도에 도달하기 전에 사전 알림을 발송한다.

### :white_check_mark: Task Breakdown
- [ ] **1.** `lib/monitoring/capacity-checker.ts`:
  ```typescript
  // Supabase Management API 또는 pg 쿼리로 사용량 조회
  const dbSize = await prisma.$queryRaw`SELECT pg_database_size(current_database())`;
  const usagePercent = (dbSize / (500 * 1024 * 1024)) * 100;
  if (usagePercent > 80) {
    await notifyAdmin('DB_CAPACITY_WARNING', `사용량 ${usagePercent.toFixed(1)}%`);
  }
  ```
- [ ] **2.** 임계치: DB 80% (400MB), Storage 80% (800MB) 시 ADMIN 알림
- [ ] **3.** 일간 1회 체크 스케줄 (setInterval or Cron)

---

# 4-B. 가용성 & 신뢰성 — 6건

---

## NFR-AVAIL-001: 가용성 모니터링 대시보드

---
title: "[NFR/Avail] NFR-AVAIL-001: Cloudflare/Supabase 상태 페이지 모니터링"
labels: 'nfr, monitoring, priority:low, epic:infra'
---

### :dart: Summary
- **목적**: 외부 인프라(Vercel, Supabase)의 상태 페이지(https://status.vercel.com 등)를 주기적으로 체크하여 가용성 대시보드에 반영.
- **MVP**: Best Effort — 상태 페이지 URL 링크 + 수동 체크 가이드 문서.

---

## NFR-AVAIL-002: 유지보수 사전 72시간 고지 프로세스

---
title: "[NFR/Avail] NFR-AVAIL-002: 유지보수 사전 72시간 고지 + 일정 로그"
labels: 'nfr, process, priority:low, epic:infra'
---

### :dart: Summary
- **목적**: 계획 유지보수 시 72시간 전 NOTI-001을 통해 전 사용자에게 알림 발송. 유지보수 일정을 DB에 로그 기록.
- **실행**: `POST /api/v1/maintenance/schedule` → 알림 발송 + 로그.

---

## NFR-REL-001: pg_dump 백업 프로세스 문서화

---
title: "[NFR/Rel] NFR-REL-001: 수동 pg_dump 백업 프로세스 문서화 (RPO ≤24시간)"
labels: 'nfr, documentation, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: Supabase 제공 자동 백업(Pro 이상) 대신 Free Tier에서 수동 `pg_dump`로 일일 백업하는 절차를 문서화한다.
- **산출물**: `docs/backup-procedure.md` — 명령어, 복원 절차, RPO 24시간 보장 체크리스트.

---

## NFR-REL-002: STT 오인식률 ≤10% 정기 테스트

---
title: "[NFR/Rel] NFR-REL-002: STT 오인식률 ≤10% 정기 정확도 자동화 스크립트"
labels: 'nfr, test, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: STT 변환 결과를 정답 레이블과 비교하여 오인식률(Word Error Rate)을 자동 측정한다.
- **실행**: 10종 오디오 픽스처 + 정답 JSON → WER 계산 → ≤10% 어서션.

---

## NFR-REL-003: Vision 파싱 실패율 ≤15% 정기 테스트

---
title: "[NFR/Rel] NFR-REL-003: Vision 파싱 실패율 ≤15% 정기 정확도 스크립트"
labels: 'nfr, test, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: 10종 이미지 픽스처 + 기대 JSON → 파싱 성공률 ≥85% 어서션.

---

## NFR-REL-004: 감사 리포트 불일치율 ≤1%

---
title: "[NFR/Rel] NFR-REL-004: 감사 리포트 불일치율 ≤1% 정합성 검증"
labels: 'nfr, test, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: Lot 병합 결과가 원본 LOG_ENTRY와 100% 일치하는지 자동 검증. 불일치율 목표 ≤1%.

---

# 4-C. 보안 (Security) — 5건

---

## NFR-SEC-001: HTTPS 전구간 암호화 검증

---
title: "[NFR/Sec] NFR-SEC-001: HTTPS 전구간 암호화 + SSL 인증서 검증"
labels: 'nfr, security, priority:must, epic:security'
---

### :dart: Summary
- **목적**: Vercel 배포 환경에서 모든 트래픽이 HTTPS로 제공되는지 검증. HTTP → HTTPS 리다이렉트 확인 + TLS 1.2+ 확인.

### :white_check_mark: Task Breakdown
- [ ] `curl -I http://factoryai.vercel.app` → 301/308 리다이렉트 확인
- [ ] `openssl s_client` 또는 SSL Labs API로 인증서 검증 스크립트
- [ ] CI에 추가: 배포 후 자동 HTTPS 검증

---

## NFR-SEC-002: AI 모델 환경변수 전환 테스트

---
title: "[NFR/Sec] NFR-SEC-002: Vercel AI SDK 모델 교체 — 환경변수 전환 테스트"
labels: 'nfr, security, test, priority:should, epic:security'
---

### :dart: Summary
- **목적**: `AI_PROVIDER`/`AI_MODEL` 환경변수만 바꿔서 모델을 전환할 때 코드 변경 0건 보장.
- **실행**: `.env.test`에서 `AI_PROVIDER=google→mock` 전환 → `getAIModel()` 정상 동작 확인.

---

## NFR-SEC-003: 감사 로그 누락률 0% + 알림 ≤10초

---
title: "[NFR/Sec] NFR-SEC-003: Prisma 감사 로그 누락률 0% + 이상 알림 ≤10초"
labels: 'nfr, security, test, priority:must, epic:security'
---

### :dart: Summary
- **목적**: AUTH-003 Prisma Middleware가 100% 무결하게 감사 로그를 기록하는지 통합 검증.

### :white_check_mark: Task Breakdown
- [ ] 100건 CRUD 작업 실행 → AUDIT_LOG 100건 확인 (누락 0건)
- [ ] 보안 이벤트 발생 → CISO 알림 ≤10초 (타이밍 어서션)

---

## NFR-SEC-004: 분기 보안 감사 프로세스 문서화

---
title: "[NFR/Sec] NFR-SEC-004: 분기 1회 내부 보안 감사 프로세스 + 템플릿"
labels: 'nfr, documentation, priority:low, epic:security'
---

### :dart: Summary
- **산출물**: `docs/security-audit-template.md` — 분기별 체크 항목(RLS, 암호화, 접근 로그, 키 로테이션 등) 템플릿.

---

## NFR-SEC-005: ERP 연결 문자열 암호화 저장 검증

---
title: "[NFR/Sec] NFR-SEC-005: ERP_CONNECTION.connection_string 암호화 저장"
labels: 'nfr, security, priority:should, epic:security'
---

### :dart: Summary
- **목적**: DB-011의 `connection_string` 필드가 평문이 아닌 AES-256 암호화된 상태로 저장되는지 검증.
- **실행**: DB 직접 조회 → Base64/AES 패턴 매칭 → 평문 불가 어서션.

---

# 4-D. 운영/모니터링 — 6건

---

## NFR-MON-001: 결측률 >10% 시 COO 알림

---
title: "[NFR/Mon] NFR-MON-001: 결측률 >10% → COO 알림 ≤30초"
labels: 'nfr, monitoring, priority:must, epic:infra'
---

### :dart: Summary
- **목적**: E1-QRY-001의 결측률 데이터를 모니터링하여 10% 초과 시 COO에게 30초 이내 알림 발송.
- **실행**: 결측률 API 주기적 폴링 (5분) → 임계치 초과 → NOTI-001 COO 알림.

---

## NFR-MON-002: 보안 이벤트 → CISO 알림

---
title: "[NFR/Mon] NFR-MON-002: 보안 이벤트 → CISO 알림 ≤30초"
labels: 'nfr, monitoring, security, priority:must, epic:infra'
---

### :dart: Summary
- **목적**: AUTH-004의 `securityEventEmitter`에서 발행된 이벤트를 소비하여 CISO에게 ≤30초 알림.
- **실행**: 이벤트 리스너 → NOTI-001(target_role=CISO, severity=CRITICAL) 호출.

---

## NFR-MON-003: 이상 감지 → 품질이사 알림

---
title: "[NFR/Mon] NFR-MON-003: 이상 감지 → 품질이사 알림 ≤30초"
labels: 'nfr, monitoring, priority:must, epic:infra'
---

### :dart: Summary
- **목적**: E2B-CMD-001에서 이상 징후 감지 시 품질이사(AUDITOR)에게 ≤30초 알림.

---

## NFR-MON-004: 센서 HW 연결 끊김 → 알림 ≤1분

---
title: "[NFR/Mon] NFR-MON-004: 센서 연결 끊김 감지 → ≤1분 알림"
labels: 'nfr, monitoring, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: DATA_SOURCE 테이블의 `status` 필드를 모니터링하여 DISCONNECTED 전환 감지 시 ≤1분 내 OPERATOR 알림.
- **실행**: 60초 간격 스캐너 → `status !== 'CONNECTED'` → NOTI-001.

---

## NFR-MON-005: 디스크 용량 <20% → 알림 ≤1분

---
title: "[NFR/Mon] NFR-MON-005: 디스크(DB) 용량 <20% 여유 시 알림"
labels: 'nfr, monitoring, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: NFR-PERF-008의 용량 모니터링 결과를 기반으로, 여유 용량이 20% 미만일 때 ≤1분 ADMIN 알림.

---

## NFR-MON-006: 월간 가동률 자동 리포트

---
title: "[NFR/Mon] NFR-MON-006: 월간 가동률 리포트 자동 생성"
labels: 'nfr, monitoring, priority:low, epic:infra'
---

### :dart: Summary
- **목적**: 월말에 API 가동률(uptime %), 평균 응답 시간, 에러율을 자동 집계하여 리포트를 생성한다.
- **MVP**: Vercel Analytics 데이터 기반 간이 리포트.

---

# 4-E. 확장성 & 유지보수성 — 3건

---

## NFR-SCALE-001: 3개 생산라인 동시 지원 검증

---
title: "[NFR/Scale] NFR-SCALE-001: 고객사당 3개 생산라인 동시 운영 테스트"
labels: 'nfr, scalability, test, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: MOCK-001의 3개 PRODUCTION_LINE 데이터로, 동시 로깅/조회/리포트 생성이 충돌 없이 동작하는지 검증.
- **실행**: 3 라인 × 5건 동시 STT → 15건 전수 처리 + 라인별 격리 확인.

---

## NFR-SCALE-002: 플러그인 아키텍처 설계 검증

---
title: "[NFR/Scale] NFR-SCALE-002: Phase 2/3 모듈 추가 — 아키텍처 변경 0건 검증"
labels: 'nfr, architecture, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: Phase 2(온프레미스 전환) 또는 Phase 3(멀티 테넌트)에서 **기존 코드 변경 없이** 새 모듈을 추가할 수 있는 아키텍처 유연성을 검증한다.
- **실행**: 가상 모듈 `plugins/sample-module/` → 핵심 코드 변경 0건 + 환경변수만으로 활성화.

---

## NFR-MAINT-001: ERP 스키마 변형 패턴 라이브러리

---
title: "[NFR/Maint] NFR-MAINT-001: ERP 스키마 변형 패턴 데이터 구조 + 초기 적재"
labels: 'nfr, maintenance, priority:should, epic:infra'
---

### :dart: Summary
- **목적**: 더존, 영림원 등 다양한 ERP 스키마 변형 사례를 라이브러리로 관리한다. E3-CMD-004의 스키마 변경 감지 로직에 활용.
- **산출물**: `data/erp-schema-patterns/` 디렉토리 + 더존 iCUBE / Smart A / 영림원 K-System 초기 패턴 JSON.

---

# 4-F. SVC 서비스 운영 시스템 지원 태스크 — 14건

---

## SVC-SYS-001: [Command] ONBOARDING_PROJECT 상태 관리 CRUD

---
title: "[Feature/SVC] SVC-SYS-001: 온보딩 프로젝트 CRUD (4단계 상태 관리)"
labels: 'feature, backend, priority:should, epic:svc-onboarding'
---

### :dart: Summary
- **목적**: 고객 온보딩 프로젝트를 4단계(SURVEY→INSTALL→ACCOMPANY→COMPLETE)로 관리하는 CRUD API.

### :white_check_mark: Task Breakdown
- [ ] `POST /api/v1/onboarding` — 프로젝트 생성
- [ ] `PATCH /api/v1/onboarding/{id}/advance` — 다음 단계 전환
- [ ] 상태 전환 규칙: SURVEY→INSTALL (체크리스트 100% 완료 시만)
- [ ] 감사 로그 (AUTH-003) 연동

---

## SVC-SYS-002: [Query] 온보딩 체크리스트 현황 조회

---
title: "[Feature/SVC] SVC-SYS-002: 온보딩 체크리스트 진행률 조회"
labels: 'feature, backend, query, priority:should, epic:svc-onboarding'
---

### :dart: Summary
- **실행**: `GET /api/v1/onboarding/{id}/checklist` → 항목별 완료 상태 + 진행률%.

---

## SVC-SYS-003: [UI] 온보딩 프로젝트 관리 대시보드

---
title: "[Feature/SVC] SVC-SYS-003: 온보딩 관리 대시보드 UI"
labels: 'feature, frontend, ui, priority:should, epic:svc-onboarding'
---

### :dart: Summary
- **목적**: 온보딩 4단계 진행 현황 스텝퍼(Stepper) UI + 체크리스트 뷰.

---

## SVC-SYS-004: [Command] VOUCHER_PROJECT 6단계 상태 관리

---
title: "[Feature/SVC] SVC-SYS-004: 바우처 프로젝트 CRUD (6단계 상태 관리)"
labels: 'feature, backend, priority:should, epic:svc-voucher'
---

### :dart: Summary
- **목적**: 바우처 프로젝트를 DRAFT→SUBMITTED→REVIEWING→APPROVED→EXECUTING→CLOSED 6단계로 관리.
- **실행**: CRUD + 상태 전환 API + 자부담액/일정 관리 필드.

---

## SVC-SYS-005: [Query] 바우처 프로젝트 현황 조회

---
title: "[Feature/SVC] SVC-SYS-005: 바우처 프로젝트 현황/이력 조회"
labels: 'feature, backend, query, priority:should, epic:svc-voucher'
---

---

## SVC-SYS-006: [UI] 바우처 프로젝트 관리 대시보드

---
title: "[Feature/SVC] SVC-SYS-006: 바우처 관리 대시보드 (신청→정산 플로우 시각화)"
labels: 'feature, frontend, ui, priority:should, epic:svc-voucher'
---

### :dart: Summary
- **목적**: 6단계 플로우를 Kanban 보드 또는 프로세스 바 형태로 시각화.

---

## SVC-SYS-007: [Command] SECURITY_REVIEW 상태 관리

---
title: "[Feature/SVC] SVC-SYS-007: 보안 심의 상태 관리 CRUD"
labels: 'feature, backend, priority:should, epic:svc-security-review'
---

### :dart: Summary
- **목적**: CISO 보안 심의를 PENDING→CONDITIONAL→APPROVED/REJECTED 상태로 관리.

---

## SVC-SYS-008: [Command] 보안 심의 문서 자동 생성

---
title: "[Feature/SVC] SVC-SYS-008: 보안 심의 문서 3종 자동 생성"
labels: 'feature, backend, priority:should, epic:svc-security-review'
---

### :dart: Summary
- **목적**: 망분리 설계서 + ISMS 확인서 + 데이터 흐름도 3종을 E6-CMD-002 체크리스트 데이터 기반으로 자동 생성.

### :white_check_mark: Task Breakdown
- [ ] 템플릿 3종 마크다운/PDF 구현
- [ ] 시스템 현황 자동 주입 (RLS 상태, 감사 로그 건수, RBAC 설정 등)
- [ ] PDF 다운로드 API

---

## SVC-SYS-009: [UI] 보안 심의 관리 페이지

---
title: "[Feature/SVC] SVC-SYS-009: 보안 심의 관리 UI"
labels: 'feature, frontend, ui, priority:should, epic:svc-security-review'
---

### :dart: Summary
- **목적**: 문서 준비 현황(✅/❌), 심의 결과, 보완 요청 트래킹.

---

## SVC-SYS-010: [Command] 성과 보고서 자동 생성

---
title: "[Feature/SVC] SVC-SYS-010: 사후관리 성과 보고서 자동 생성 + 정부 포맷 변환"
labels: 'feature, backend, priority:should, epic:svc-post-management'
---

### :dart: Summary
- **목적**: E7-QRY-001 성과 데이터를 정부 제출용 양식으로 자동 변환. **고객 투입 0시간**.

### :white_check_mark: Task Breakdown
- [ ] 성과 데이터 집계 (절감액, 생성건수, ROI)
- [ ] 정부 양식 템플릿 매핑 + PDF 생성
- [ ] 제출 이력 DB 저장

---

## SVC-SYS-011: [Command] 정부 양식 버전 자동 검증

---
title: "[Feature/SVC] SVC-SYS-011: 정부 양식 버전 불일치 → 제출 차단"
labels: 'feature, backend, priority:should, epic:svc-post-management'
---

### :dart: Summary
- **목적**: 정부 양식 버전이 변경된 경우 구버전 양식으로 제출하는 것을 차단 + 관리자 알림.

---

## SVC-SYS-012: [Command] 장애 접수 + 1H 에스컬레이션

---
title: "[Feature/SVC] SVC-SYS-012: 장애 접수 + 1시간 미해결 → 자동 출동 에스컬레이션"
labels: 'feature, backend, priority:should, epic:svc-incident'
---

### :dart: Summary
- **목적**: 장애 접수 → 1차 원격 진단 → 1시간 내 미해결 시 자동으로 현장 출동 에스컬레이션.
- **실행**: `setTimeout(60 * 60 * 1000)` → 미해결 시 ADMIN 긴급 알림.

---

## SVC-SYS-013: [Command] 장애 보고서 자동 생성

---
title: "[Feature/SVC] SVC-SYS-013: 장애 보고서 템플릿 자동 생성"
labels: 'feature, backend, priority:should, epic:svc-incident'
---

### :dart: Summary
- **목적**: 장애 해결 후 원인 분석 + 재발 방지 대책 템플릿을 자동 생성. 수동 작성 최소화.

---

## SVC-SYS-014: [Command] 장애 재발 감지 → RCA 트리거

---
title: "[Feature/SVC] SVC-SYS-014: 동일 원인 90일 내 재발 → RCA 심화 리포트 자동 트리거"
labels: 'feature, backend, priority:should, epic:svc-incident'
---

### :dart: Summary
- **목적**: 동일 `root_cause` 분류의 장애가 90일 이내 재발하면 자동으로 RCA(Root Cause Analysis) 심화 리포트를 트리거.
- **실행**: 장애 생성 시 과거 90일 동일 원인 검색 → 발견 시 RCA 알림 + 템플릿 자동 생성.

---

# 4-G. 프로젝트 초기 설정 & DevOps — 4건

---

## INIT-001: Next.js 15 프로젝트 초기화

---
title: "[Init] INIT-001: Next.js 15 (App Router) + Tailwind CSS + shadcn/ui 초기 설정"
labels: 'init, foundation, priority:must, epic:foundation'
---

### :dart: Summary
- **기능명**: [INIT-001] 프로젝트 뼈대 생성
- **목적**: 전체 프로젝트의 기초가 되는 Next.js 15 App Router 프로젝트를 초기화한다.

### :white_check_mark: Task Breakdown
- [ ] **1.** `npx -y create-next-app@latest ./ --ts --eslint --tailwind --app --src-dir --import-alias "@/*"`
- [ ] **2.** shadcn/ui 초기화: `npx shadcn@latest init`
- [ ] **3.** 필수 컴포넌트 설치: `button`, `input`, `card`, `table`, `dialog`, `toast`, `badge`
- [ ] **4.** 디렉토리 구조 생성:
  ```
  src/
  ├── app/           # App Router pages
  ├── components/    # UI 컴포넌트
  ├── lib/           # 비즈니스 로직 (ai/, erp/, hitl/, etc.)
  ├── actions/       # Server Actions
  ├── types/         # TypeScript 타입/인터페이스
  └── utils/         # 유틸리티 함수
  ```
- [ ] **5.** 한글 폰트 설정 (Noto Sans KR 또는 Pretendard)
- [ ] **6.** 기본 `metadata` SEO 설정

### :test_tube: Acceptance Criteria
- **Given**: INIT-001 실행
- **Then**: `npm run dev` → localhost:3000 정상 렌더링 + shadcn/ui 버튼 표시

### :construction: Dependencies & Blockers
- **Depends on**: None (**Good First Issue**)
- **Blocks**: `INIT-002`, `INIT-003`, `INIT-004`, `DB-001` (모든 후속 작업)

---

## INIT-002: Vercel 배포 + CI/CD

---
title: "[Init] INIT-002: Vercel Free 배포 + Git Push 자동 배포 파이프라인"
labels: 'init, devops, priority:must, epic:foundation'
---

### :dart: Summary
- **목적**: Git Push 시 자동 배포되는 CI/CD 파이프라인 구성.

### :white_check_mark: Task Breakdown
- [ ] Vercel 프로젝트 연결 (`vercel link`)
- [ ] 환경변수 설정 (Vercel Dashboard)
- [ ] Preview Deployment (PR) + Production Deployment (main merge)
- [ ] 자동 롤백 설정 확인

---

## INIT-003: 환경변수 관리 체계

---
title: "[Init] INIT-003: 환경변수 관리 (.env.local / .env.cloud) + README"
labels: 'init, documentation, priority:should, epic:foundation'
---

### :dart: Summary
- **목적**: 개발(local)과 MVP(cloud) 환경을 환경변수만으로 전환하는 체계를 확립한다.

### :white_check_mark: Task Breakdown
- [ ] `.env.local` 템플릿:
  ```bash
  DATABASE_URL="file:./dev.db"
  NEXTAUTH_SECRET="dev-secret-32-bytes"
  AI_PROVIDER=google
  AI_MODEL=models/gemini-1.5-flash
  GOOGLE_GENERATIVE_AI_API_KEY=<key>
  MOCK_API=true
  ```
- [ ] `.env.cloud` 템플릿:
  ```bash
  DATABASE_URL="postgresql://..."
  NEXTAUTH_SECRET=<production-secret>
  MOCK_API=false
  ```
- [ ] `.env.example` + `.gitignore` 업데이트
- [ ] `docs/env-guide.md` 문서 작성

---

## INIT-004: 공통 레이아웃 + 네비게이션 + 라우팅

---
title: "[Init] INIT-004: 공통 레이아웃 + 4개 클라이언트 앱 라우팅 구조"
labels: 'init, frontend, priority:must, epic:foundation'
---

### :dart: Summary
- **목적**: CLI-01~04 네 개의 클라이언트 앱을 하나의 Next.js 앱 내에서 라우팅으로 분리.

### :white_check_mark: Task Breakdown
- [ ] **1.** 공통 RootLayout (`app/layout.tsx`):
  - 사이드바 네비게이션
  - 헤더 (로고 + 알림 벨 + 사용자 아바타)
  - 스켈레톤 로딩
- [ ] **2.** 라우팅 구조:
  ```
  /dashboard          → CLI-01 관리자 대시보드
  /dashboard/xai      → E2B XAI 대시보드
  /dashboard/erp      → E3 ERP 관리
  /dashboard/security → CLI-03 CISO 보안 콘솔
  /dashboard/performance → E7 성과 대시보드
  /roi-calculator     → CLI-02 ROI 계산기 (공개)
  /rollback           → CLI-04 롤백/수정 뷰어
  ```
- [ ] **3.** 역할 기반 네비메뉴 표시/숨김 (AUTH-002 연동 준비)
- [ ] **4.** 반응형 사이드바 (모바일 햄버거 메뉴)

### :test_tube: Acceptance Criteria
- **Given**: INIT-004 완료
- **Then**: 6개 경로 접근 시 각각 다른 페이지 렌더링 + 사이드바 활성 메뉴 하이라이팅.

### :construction: Dependencies & Blockers
- **Depends on**: `INIT-001`
- **Blocks**: 모든 UI 태스크의 공통 프레임워크

---

# 전체 실행 순서 + 총 통계

## 권장 실행 순서

| Phase | 범위 | 건수 | 예상 |
|:---:|:---|:---:|:---:|
| **0** | INIT-001~004 (프로젝트 초기화) | 4건 | 4h |
| **1** | NFR-SEC-001, NFR-AVAIL-001 (Good First Issues) | 2건 | 1h |
| **2** | NFR-SEC-002~005 (보안 검증) | 4건 | 3h |
| **3** | NFR-PERF-001~008 (성능 벤치마크) | 8건 | 6h |
| **4** | NFR-AVAIL/REL (가용성/신뢰성) | 5건 | 3h |
| **5** | NFR-MON-001~006 (모니터링) | 6건 | 4h |
| **6** | NFR-SCALE/MAINT (확장성) | 3건 | 2h |
| **7** | SVC-SYS-001~014 (서비스 운영) | 14건 | 14h |
| | **총합** | **46건** | **~37h** |

## 전체 프로젝트 Issue 최종 현황

| # | 파일명 | 범위 | 건수 |
|:---:|:---|:---|:---:|
| 9 | `9_Issues_DB-001_to_DB-017.md` | DB 스키마 | 17 |
| 10 | `10_Issues_API-001_to_API-019.md` | API 계약 | 19 |
| 11 | `11_Issues_MOCK-001_to_MOCK-010.md` | Mock 데이터 | 10 |
| 12 | `12_Issues_AUTH-001_to_AUTH-004.md` | 인증/인가 | 4 |
| 13 | `13_Issues_Foundation_AI_NOTI.md` | AI + 알림 | 5 |
| 14 | `14_Issues_E1-CMD_to_E1-UI.md` | E1 패시브 로깅 | 12 |
| 15 | `15_Issues_E2-CMD_to_E2-UI.md` | E2 감사 리포트 | 11 |
| 16 | `16_Issues_E2B-CMD_to_E2B-UI.md` | E2B XAI | 8 |
| 17 | `17_Issues_E3-CMD_to_E3-UI.md` | E3 ERP 브릿지 | 9 |
| 18 | `18_Issues_E4_E6_E7_HITL.md` | E4+E6+E7+HITL | 29 |
| 19 | `19_Issues_TEST-E1_to_TEST-HITL.md` | 테스트 | 42 |
| **20** | **`20_Issues_NFR_SVC_INIT.md`** | **NFR+SVC+DevOps** | **46** |
| | | **총합** | **212건** |
