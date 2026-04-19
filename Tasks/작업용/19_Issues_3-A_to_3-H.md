# FactoryAI — Step 3 테스트 태스크 Issues (TEST-E1~HITL)

> **Source**: SRS-002 Rev 2.0 (V0.8) — Step 3. 완료 조건(AC)을 테스트(Test) 태스크로 변환  
> **작성일**: 2026-04-19  
> **총 Issue**: 41건 (E1:7 + E2:5 + E2B:5 + E3:5 + E4:5 + E6:4 + E7:5 + HITL:6)  
> **테스트 전략**: 모든 AC/NAC를 Jest 단위 테스트 + Playwright E2E 테스트로 자동화

> [!IMPORTANT]
> 각 테스트 Issue는 대응하는 **구현 Issue(CMD/QRY/UI)가 완료된 후** 실행합니다.  
> 테스트 코드는 `__tests__/` 디렉토리에, E2E는 `e2e/` 디렉토리에 배치합니다.

---

# 3-A. E1 패시브 로깅 테스트

---

## TEST-E1-001: STT 음성→텍스트 변환 정확도/지연 테스트

---
name: Test Task
title: "[Test/E1] TEST-E1-001: STT 변환 정확도 ≥85% + 응답 ≤5초 검증"
labels: 'test, e2e, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [TEST-E1-001] STT 변환 GWT 테스트
- **목적**: 버튼 녹음 → Gemini STT → 텍스트 변환 → LOG_ENTRY 저장 파이프라인의 정확도와 성능을 검증한다.

### :link: References (Spec & Context)
- SRS: REQ-FUNC-001 Acceptance Criteria
- 구현 Issue: `E1-CMD-001`

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `__tests__/e1/stt-pipeline.test.ts` 작성
- [ ] **2.** 테스트 오디오 픽스처 3종 준비: 명확 음성 / 소음 환경 / 무음
- [ ] **3.** Mock AI 응답 기반 단위 테스트 (순수 로직 검증)
- [ ] **4.** 통합 테스트: 실제 AI 호출 (환경변수 `TEST_WITH_AI=true` 시)
- [ ] **5.** 성능 측정: `performance.now()` 기반 응답 시간 어서션

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 명확한 음성 → 정확한 텍스트**
- **Given**: "온도 180도, 압력 정상" 음성 오디오 파일
- **When**: E1-CMD-001 Server Action을 호출한다
- **Then**: 변환된 텍스트에 "180"과 "압력" 키워드가 포함되고, LOG_ENTRY가 PENDING으로 저장된다 (정확도 ≥85%)

**Scenario 2: 응답 시간 ≤5초**
- **Given**: 정상 오디오 파일
- **When**: Server Action 호출부터 DB 저장 완료까지 소요 시간 측정
- **Then**: p95 ≤ 5,000ms

**Scenario 3: 소음 환경 (경계값)**
- **Given**: 80dB+ 공장 소음이 포함된 오디오
- **When**: STT 처리를 시도한다
- **Then**: 파싱 실패 시 422 에러 반환 (크래시 0건)

### :gear: Technical & Non-Functional Constraints
- **테스트 프레임워크**: Jest + ts-jest
- **AI Mock**: `jest.mock('@/lib/ai/provider')` 사용, 통합 테스트 시 실제 호출
- **픽스처**: `__fixtures__/audio/` 디렉토리에 WAV 파일 3종

### :checkered_flag: Definition of Done (DoD)
- [ ] 3개 시나리오 테스트 작성 + PASS
- [ ] CI 파이프라인에 등록 (`npm test -- --testPathPattern=e1/stt`)
- [ ] 커버리지 ≥ 80% (해당 모듈)

### :construction: Dependencies & Blockers
- **Depends on**: `E1-CMD-001` (구현 완료)

---

## TEST-E1-002: Vision 이미지 파싱 정확도/지연 테스트

---
title: "[Test/E1] TEST-E1-002: Vision 파싱 성공률 ≥85% + 응답 ≤5초"
labels: 'test, e2e, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **목적**: 카메라 촬영 → Gemini Vision → 상태값 JSON → LOG_ENTRY 저장 파이프라인 검증.

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 선명한 계기판 → 정확한 값 추출**
- **Given**: 온도계 '85°C' 이미지
- **When**: E1-CMD-002 Server Action 호출
- **Then**: `{ temperature: 85 }` 추출 + PENDING 저장

**Scenario 2: 흐린/역광 이미지 → Graceful Failure**
- **Given**: 블러 처리된 이미지
- **When**: Server Action 호출
- **Then**: 422 반환 (크래시 0건), 오 데이터 기록 0건

**Scenario 3: 10MB 초과 이미지 → 거부**
- **Given**: 12MB JPEG
- **When**: 업로드 시도
- **Then**: 400 + `FILE_TOO_LARGE` 에러

### :construction: Dependencies & Blockers
- **Depends on**: `E1-CMD-002`

---

## TEST-E1-003: Approve/Reject 반영 속도 + 감사 로그 테스트

---
title: "[Test/E1] TEST-E1-003: 승인/거부 반영 ≤1초 + 감사 로그 + 무승인 발행 0건"
labels: 'test, integration, priority:must, epic:e1-passive-logging'
---

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 승인 반영 ≤1초**
- **Given**: PENDING LOG_ENTRY
- **When**: APPROVED 패치 + 시간 측정
- **Then**: DB 반영 ≤1초 + AUDIT_LOG 자동 기록

**Scenario 2: 미인가 사용자 승인 시도**
- **Given**: VIEWER 역할 사용자
- **When**: 승인 API 호출
- **Then**: 403 차단 + 감사 로그(UNAUTHORIZED_ACCESS) + CISO 알림

**Scenario 3: 인간 승인 없이 외부 발행 0건 (HITL 검증)**
- **Given**: PENDING 상태 LOG_ENTRY
- **When**: 외부 참조(리포트 포함 등) 시도
- **Then**: 100% 차단

### :construction: Dependencies & Blockers
- **Depends on**: `E1-CMD-003`, `AUTH-002`, `AUTH-003`

---

## TEST-E1-004: 결측률 리포트 ≤5% 달성 검증

---
title: "[Test/E1] TEST-E1-004: 결측률 ≤5% 달성 검증"
labels: 'test, query, priority:should, epic:e1-passive-logging'
---

### :test_tube: Acceptance Criteria
- **Given**: 100건 계획, 96건 APPROVED
- **When**: 결측률 API 조회
- **Then**: `missing_rate = 4%`, 목표(5%) 이내 판정

---

## TEST-E1-005: Vision 파싱 실패 → 재촬영 알림 ≤3초

---
title: "[Test/E1] TEST-E1-005: Vision 실패 → '재촬영 요청' 알림 ≤3초 + 오 데이터 0건"
labels: 'test, integration, priority:must, epic:e1-passive-logging'
---

### :test_tube: Acceptance Criteria
- **Given**: 화질 불량 이미지
- **When**: E1-CMD-002 → E1-CMD-004 실행
- **Then**: ①NOTIFICATION 테이블에 WARNING 알림 생성 ②소요시간 ≤3초 ③LOG_ENTRY에 잘못된 데이터 0건

---

## TEST-E1-006: 네트워크 단절 → "연결 끊김" UI 표시

---
title: "[Test/E1] TEST-E1-006: 네트워크 단절 → '연결 끊김' 배너 UI"
labels: 'test, e2e, priority:should, epic:e1-passive-logging'
---

### :test_tube: Acceptance Criteria
- **Given**: Playwright에서 `context.setOffline(true)` 실행
- **When**: 페이지 상태 변경 감지
- **Then**: "네트워크 단절" 배너 가시성 확인 (`data-testid="offline-banner"`)

---

## TEST-E1-007: 동시 5건+ 큐잉 드롭 0건 + 지연 ≤15초

---
title: "[Test/E1] TEST-E1-007: 동시 5건+ 녹음 큐잉 — 드롭 0건, 최대 지연 ≤15초"
labels: 'test, stress, priority:must, epic:e1-passive-logging'
---

### :test_tube: Acceptance Criteria
- **Given**: Mock AI 기반 10건 동시 요청 (`Promise.all`)
- **When**: AI-002 Rate Limiter를 통과한다
- **Then**: ①10건 전부 200 성공 ②유실 0건 ③최대 응답 시간 ≤15초

### :white_check_mark: Task Breakdown
- [ ] 스트레스 테스트 스크립트 (`__tests__/e1/concurrent-queue.stress.ts`)
- [ ] AI-002 Mock (12 RPM 제한 시뮬레이션)
- [ ] 전수 응답 수집 + 통계 리포트 출력

---

# 3-B. E2 감사 리포트 테스트

---

## TEST-E2-001: Lot 병합 + PDF 생성 테스트

---
title: "[Test/E2] TEST-E2-001: Lot 병합 정확도 ≥99% + 클라이언트 PDF 생성"
labels: 'test, integration, priority:must, epic:e2-audit-report'
---

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 정상 병합 + PDF**
- **Given**: 50건 APPROVED LOG_ENTRY (시간순)
- **When**: Lot 병합 → PDF 렌더 실행
- **Then**: ①시간 순서 100% 정확 ②PDF Blob 생성 성공 ③페이지 ≥1

**Scenario 2: 오병합 0건 보장**
- **Given**: 타임스탬프 역전 데이터 포함
- **When**: 병합 시도
- **Then**: 충돌 에러 반환 (자동 병합 차단)

---

## TEST-E2-002: 결측치 감지 ≥95% + 보완 알림 ≤30초

---
title: "[Test/E2] TEST-E2-002: 결측치 목록 + 보완 알림 (감지율 ≥95%, ≤30초)"
labels: 'test, integration, priority:must, epic:e2-audit-report'
---

### :test_tube: Acceptance Criteria
- **Given**: 필수 5항목 중 2항목 누락된 Lot 데이터
- **When**: 감사 리포트 생성 시도
- **Then**: ①422 반환 ②누락 2항목 명시 ③NOTIFICATION 발송 ≤30초

---

## TEST-E2-003: XAI 한국어 설명 PDF 포함 + 누락 0건

---
title: "[Test/E2] TEST-E2-003: XAI 설명 PDF 포함 + 누락 0건"
labels: 'test, integration, priority:must, epic:e2-audit-report'
---

### :test_tube: Acceptance Criteria
- **Given**: `xai_explanation` JSON이 존재하는 AUDIT_REPORT
- **When**: PDF 렌더링 실행
- **Then**: PDF 내 "[인공지능 판단 의견]" 섹션 존재 + 한국어 텍스트 포함

---

## TEST-E2-004: 미지원 포맷 → 안내 + 크래시 0건

---
title: "[Test/E2] TEST-E2-004: 미지원 규제 포맷 에러 핸들링 (≤2초, 크래시 0건)"
labels: 'test, unit, priority:should, epic:e2-audit-report'
---

### :test_tube: Acceptance Criteria
- **Given**: `regulation_type = "ISO_99999"` (미지원)
- **When**: 리포트 생성 요청
- **Then**: 400 + `UNSUPPORTED_FORMAT` + 대안 제시 (≤2초)

---

## TEST-E2-005: 타임스탬프 충돌 → 자동 병합 차단

---
title: "[Test/E2] TEST-E2-005: 타임스탬프 충돌 시 차단 + 목록 표시 (≤5초)"
labels: 'test, unit, priority:must, epic:e2-audit-report'
---

### :test_tube: Acceptance Criteria
- **Given**: 동일 시간에 2건의 상충 데이터
- **When**: 병합 함수 호출
- **Then**: `TIMESTAMP_CONFLICT` + 충돌 Lot 목록 반환 (≤5초)

---

# 3-C. E2-B XAI 이상탐지 테스트

---

## TEST-E2B-001: XAI 한국어 설명 ≤3초 + 구조 검증

---
title: "[Test/E2B] TEST-E2B-001: XAI 한국어 설명 생성 ≤3초 + 구조 검증"
labels: 'test, integration, priority:must, epic:e2b-xai'
---

### :test_tube: Acceptance Criteria
- **Given**: 온도 편차 이상 입력 데이터
- **When**: E2B-CMD-001 호출
- **Then**: ①응답 ≤3초 ②JSON에 `summary`, `highlights`, `recommendations` 존재 ③100% 한국어

### :white_check_mark: Task Breakdown
- [ ] 한국어 검증: 영문 단어 0건 어서션 (정규표현식 `[a-zA-Z]` 매칭 0건)
- [ ] 구조 검증: Zod `.parse()` 통과 여부

---

## TEST-E2B-002: AI 단독 실행 0건 — 무승인 차단 100%

---
title: "[Test/E2B] TEST-E2B-002: AI 단독 실행 0건 — 승인 없이 실행 시 100% 차단"
labels: 'test, security, priority:must, epic:e2b-xai'
---

### :test_tube: Acceptance Criteria
- **Given**: PENDING APPROVAL 상태의 이상 징후 건
- **When**: 승인 없이 조치(STOP/CHANGE) API를 호출한다
- **Then**: ①즉시 차단 ②감사 로그 기록 ③CISO 알림 발송

### :white_check_mark: Task Breakdown
- [ ] HITL-CMD-001, HITL-CMD-003 게이트의 통합 차단 테스트
- [ ] 모든 action_type × 무승인 조합 매트릭스 테스트 (STOP×NULL, CHANGE×NULL, 만료된 approval)

---

## TEST-E2B-003: 판단 이력 전 과정 기록 + 검색 ≤2초

---
title: "[Test/E2B] TEST-E2B-003: AI→인간→결과 이력 누락 0% + 검색 ≤2초"
labels: 'test, integration, priority:must, epic:e2b-xai'
---

### :test_tube: Acceptance Criteria
- **Given**: AI 감지 → XAI 설명 → 품질이사 승인 → 조치 완료된 건
- **When**: Decision Trail API 호출
- **Then**: ①4단계 타임라인 완전 구성 ②누락 0개 ③응답 ≤2초

---

## TEST-E2B-004: XAI 실패 → "설명 불가" 경고 + 수동 요청

---
title: "[Test/E2B] TEST-E2B-004: XAI 실패 → 원본 보존 + 수동 판단 요청"
labels: 'test, integration, priority:must, epic:e2b-xai'
---

### :test_tube: Acceptance Criteria
- **Given**: AI 타임아웃 Mock 설정
- **When**: XAI 설명 생성 시도
- **Then**: ①크래시 0건 ②`MANUAL_REVIEW_REQUIRED` 반환 ③원본 데이터 100% 포함 ④NOTIFICATION 발송

---

## TEST-E2B-005: 30분 무응답 → COO 에스컬레이션

---
title: "[Test/E2B] TEST-E2B-005: 30분 무응답 → COO 에스컬레이션"
labels: 'test, integration, priority:must, epic:e2b-xai'
---

### :test_tube: Acceptance Criteria
- **Given**: PENDING 건이 30분 경과 (타이머 Mock으로 시간 가속)
- **When**: 에스컬레이션 스케줄러 실행
- **Then**: ①COO에게 CRITICAL 알림 ②`escalated_at` 기록 ③중복 에스컬레이션 0건

---

# 3-D. E3 ERP 브릿지 테스트

---

## TEST-E3-001: Read-Only 커넥터 + Write 0건 검증

---
title: "[Test/E3] TEST-E3-001: Read-Only 동기화 + Write 0건 (3중 차단)"
labels: 'test, security, priority:must, epic:e3-erp-bridge'
---

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: Read-Only 정상 동기화**
- **Given**: Mock ERP 65건
- **When**: 동기화 실행
- **Then**: ①내부 테이블 65건 적재 ②ERP 테이블 변경 0건

**Scenario 2: Write 시도 → 3중 차단**
- **Given**: ERP 테이블에 `create()` 호출
- **When**: Prisma Middleware 실행
- **Then**: ①즉시 차단(에러) ②SECURITY_EVENT 발행 ③CISO 알림

**Scenario 3: 미승인 테이블 접근 차단**
- **Given**: `approved_tables`에 없는 테이블
- **When**: 동기화 시도
- **Then**: 400 + `UNAPPROVED_TABLE`

### :white_check_mark: Task Breakdown
- [ ] 단위 테스트: Prisma Middleware Write 인터셉트 검증
- [ ] 통합 테스트: Read → 적재 → ERP 원본 비변경 확인
- [ ] 보안 테스트: 모든 Write 메서드(create/update/delete/createMany/updateMany/deleteMany) 차단 확인

---

## TEST-E3-002: 엑셀 파싱 성공률 ≥95% + ≤30초

---
title: "[Test/E3] TEST-E3-002: 엑셀 파싱 성공률 ≥95% + 응답 ≤30초"
labels: 'test, integration, priority:must, epic:e3-erp-bridge'
---

### :test_tube: Acceptance Criteria
- **Given**: 100행 정상 .xlsx 파일 + EUC-KR .csv 파일
- **When**: E3-CMD-002 파싱
- **Then**: ①파싱 성공 ≥95행 ②한글 깨짐 0건 ③소요 ≤30초

---

## TEST-E3-003: 정합성 불일치율 ≤2%

---
title: "[Test/E3] TEST-E3-003: 정합성 불일치율 ≤2%"
labels: 'test, query, priority:should, epic:e3-erp-bridge'
---

### :test_tube: Acceptance Criteria
- **Given**: 동기화 완료 후 내부 데이터에 1건 인위적 차이 발생
- **When**: 정합성 리포트 조회
- **Then**: 불일치 1건 감지 + 불일치율 계산 정확

---

## TEST-E3-004: 스키마 변경 감지 → 중단 + CIO 알림

---
title: "[Test/E3] TEST-E3-004: 스키마 변경 → 동기화 중단 + CIO 알림 ≤1분"
labels: 'test, security, priority:must, epic:e3-erp-bridge'
---

### :test_tube: Acceptance Criteria
- **Given**: Mock ERP Inventory에 컬럼 추가 (스냅샷과 차이)
- **When**: 동기화 시도
- **Then**: ①동기화 중단 ②적재 0건 ③CIO 알림 ≤1분 ④`SCHEMA_MISMATCH` 상태

---

## TEST-E3-005: 50MB 초과/비표준 파일 거부

---
title: "[Test/E3] TEST-E3-005: 50MB 초과 + 비표준 파일 → 거부 ≤3초 (크래시 0건)"
labels: 'test, unit, priority:should, epic:e3-erp-bridge'
---

### :test_tube: Acceptance Criteria
- **Given**: 55MB .xlsx / .hwp 파일 / 확장자 스푸핑 파일
- **When**: 업로드 시도
- **Then**: 각각 400 에러 + 한국어 에러 메시지 + 서버 크래시 0건

---

# 3-E. E4 ROI 진단 테스트

---

## TEST-E4-001: ROI 계산 정확도 ≥90% + ≤3초

---
title: "[Test/E4] TEST-E4-001: ROI 계산 정확도 ≥90% + 응답 ≤3초"
labels: 'test, unit, priority:must, epic:e4-roi'
---

### :test_tube: Acceptance Criteria
- **Given**: 금속가공, 85명, 매출 50억
- **When**: ROI 계산 API
- **Then**: ①바우처 + 자부담 + Payback 반환 ②벤치마크 대비 ±10% 이내 ③≤3초

### :white_check_mark: Task Breakdown
- [ ] 벤치마크 기대값 픽스처 작성 (금속가공/식품제조 각 3세트)
- [ ] 계산 결과 vs 기대값 편차율 어서션

---

## TEST-E4-002: 적합성 5항목 진단 완전성

---
title: "[Test/E4] TEST-E4-002: 적합성 5항목 진단 — 항목 누락 0건 + ≤5초"
labels: 'test, unit, priority:must, epic:e4-roi'
---

### :test_tube: Acceptance Criteria
- **Given**: 다양한 기업 프로파일 5종
- **When**: 진단 API 호출
- **Then**: ①5항목 × 5프로파일 = 25건 전수 등급(H/M/L) 반환 ②항목 누락 0건

---

## TEST-E4-003: B/A 카드 생성 ≤10초

---
title: "[Test/E4] TEST-E4-003: Before-After 카드 ≤10초"
labels: 'test, unit, priority:should, epic:e4-roi'
---

### :test_tube: Acceptance Criteria
- **Given**: 금속가공 입력
- **When**: B/A 카드 API
- **Then**: ①6개 KPI 카드 데이터 ②`before`/`after`/`improvement` 필드 전부 존재 ③≤10초

---

## TEST-E4-004: 필수 누락 → 차단 + 감사 로그

---
title: "[Test/E4] TEST-E4-004: 필수 항목 누락 → 계산 차단 + 감사 로그"
labels: 'test, unit, priority:should, epic:e4-roi'
---

### :test_tube: Acceptance Criteria
- **Given**: `employee_count` 미입력
- **When**: ROI 계산 요청
- **Then**: ①400 + 누락 필드 ②계산 미실행 ③감사 로그 기록

---

## TEST-E4-005: 비현실적 수치 → 경고

---
title: "[Test/E4] TEST-E4-005: 비현실적 수치(매출 0원) → 경고 + 오진단 0건"
labels: 'test, unit, priority:low, epic:e4-roi'
---

### :test_tube: Acceptance Criteria
- **Given**: `annual_revenue = 0`, `employee_count = 0`
- **When**: ROI 계산 요청
- **Then**: ①`UNREALISTIC_INPUT` 경고 ②계산 결과 미발행 or 명시적 경고 동반

---

# 3-F. E6 보안 테스트

---

## TEST-E6-001: RLS + HTTPS + 전수 감사 로그 검증

---
title: "[Test/E6] TEST-E6-001: Supabase RLS + HTTPS + 전수 감사 로그"
labels: 'test, security, priority:must, epic:e6-security'
---

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: RLS Factory Scope 격리**
- **Given**: `factory_A` 소속 OPERATOR
- **When**: LOG_ENTRY 조회
- **Then**: `factory_A` 데이터만 반환, 타 Factory 0건

**Scenario 2: 전수 감사 로그**
- **Given**: API 3건 호출 (GET, POST, PATCH)
- **When**: AUDIT_LOG 조회
- **Then**: 3건 전수 기록 확인

### :white_check_mark: Task Breakdown
- [ ] RLS 격리 테스트: 2개 Factory × 5역할 = 10 조합 매트릭스
- [ ] 감사 로그 누락 0건 검증 스크립트

---

## TEST-E6-002: RBAC 5역할 전수 매트릭스 테스트

---
title: "[Test/E6] TEST-E6-002: RBAC 5역할 × 19 API 접근 매트릭스"
labels: 'test, security, priority:must, epic:e6-security'
---

### :test_tube: Acceptance Criteria
- **Given**: 5역할(ADMIN/OPERATOR/AUDITOR/VIEWER/CISO) × 19 API 엔드포인트
- **When**: 각 조합으로 API 호출 (95건 테스트)
- **Then**: ACCESS_MATRIX 정의와 100% 일치 (허용→200계열, 차단→403)

### :white_check_mark: Task Breakdown
- [ ] `__tests__/security/rbac-matrix.test.ts` — 자동화 매트릭스 테스트
- [ ] 테스트 데이터: 5역할 Mock 세션 + 19 엔드포인트 URL 배열
- [ ] 매트릭스 결과 리포트 CSV 자동 출력

---

## TEST-E6-003: 미인가 접근 → 차단 + CISO 알림 ≤10초

---
title: "[Test/E6] TEST-E6-003: 미인가 접근 → 차단 + CISO 알림 ≤10초"
labels: 'test, security, priority:must, epic:e6-security'
---

### :test_tube: Acceptance Criteria
- **Given**: VIEWER가 ADMIN 전용 API 호출
- **When**: AUTH-002 미들웨어 실행
- **Then**: ①403 ②AUDIT_LOG 기록 ③CISO NOTIFICATION ≤10초

---

## TEST-E6-004: 보안 체크리스트 자동 생성 검증

---
title: "[Test/E6] TEST-E6-004: 보안 체크리스트 8항목 자동 생성"
labels: 'test, unit, priority:should, epic:e6-security'
---

### :test_tube: Acceptance Criteria
- **Given**: 시스템 정상 가동 상태
- **When**: 체크리스트 생성 API
- **Then**: 8항목 전부 ✅/❌ 판정 + JSON 반환

---

# 3-G. E7 대시보드 테스트

---

## TEST-E7-001: 월말 자동 발행 + 렌더링 ≤5초

---
title: "[Test/E7] TEST-E7-001: 월말 자동 발행 + 4인 대시보드 + 렌더링 ≤5초"
labels: 'test, integration, priority:should, epic:e7-dashboard'
---

### :test_tube: Acceptance Criteria
- **Given**: 당월 로깅 ≥100건
- **When**: 발행 트리거 실행
- **Then**: ①4인 데이터 생성 ②DASHBOARD 테이블 4건 저장 ③UI 렌더링 ≤5초

---

## TEST-E7-002: ROI 누적 리포트 자동 집계 (수동 0건)

---
title: "[Test/E7] TEST-E7-002: 분기 ROI 누적 — 수동 개입 0건"
labels: 'test, query, priority:should, epic:e7-dashboard'
---

### :test_tube: Acceptance Criteria
- **Given**: 3개월 데이터 존재
- **When**: ROI Summary API 조회
- **Then**: 절감액/생성건수 자동 집계 정확 + `manual_input_count = 0`

---

## TEST-E7-003: NPS 응답률 ≥30% 구조 검증

---
title: "[Test/E7] TEST-E7-003: NPS 9~10점 → 1클릭 수집 구조"
labels: 'test, integration, priority:should, epic:e7-dashboard'
---

### :test_tube: Acceptance Criteria
- **Given**: NPS 10점 사용자
- **When**: 설문 발송 → 1클릭 응답
- **Then**: ①NPS 점수 + 레퍼런스 동의 DB 저장 ②응답 플로우 3초 이내

---

## TEST-E7-004: 데이터 부족 → 미발행 + 경고

---
title: "[Test/E7] TEST-E7-004: 당월 <100건 → 미발행 + '데이터 부족' 경고"
labels: 'test, integration, priority:should, epic:e7-dashboard'
---

### :test_tube: Acceptance Criteria
- **Given**: 당월 로깅 50건 (< 100건)
- **When**: 발행 트리거
- **Then**: ①DASHBOARD 미생성 ②COO 경고 알림 발송

---

## TEST-E7-005: 렌더링 지연 → 재시도 3회 → 안내

---
title: "[Test/E7] TEST-E7-005: 렌더링 5초 초과 → 재시도 3회 → '지연 안내'"
labels: 'test, integration, priority:should, epic:e7-dashboard'
---

### :test_tube: Acceptance Criteria
- **Given**: 렌더링 함수가 강제 8초 지연 (Mock)
- **When**: 재시도 3회 실행
- **Then**: ①3회 시도 로그 기록 ②최종 실패 시 "지연 안내" 메시지 반환 ③에러 화면 0건

---

# 3-H. HITL 안전 프로토콜 테스트

---

## TEST-HITL-001: PENDING 외부 발행 차단 ≤1초

---
title: "[Test/HITL] TEST-HITL-001: PENDING 발행 차단 + 알림 ≤10초"
labels: 'test, security, priority:must, epic:hitl-safety'
---

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: PENDING 리포트 다운로드 차단**
- **Given**: `status=PENDING` AUDIT_REPORT
- **When**: PDF 다운로드 시도
- **Then**: ①차단 ≤1초 ②`PUBLICATION_BLOCKED` 이벤트 ③관리자 알림 ≤10초

**Scenario 2: APPROVED 리포트 정상 다운로드**
- **Given**: `status=APPROVED` AUDIT_REPORT
- **When**: PDF 다운로드
- **Then**: 정상 200 반환

---

## TEST-HITL-002: XAI NULL → 리포트 발행 차단

---
title: "[Test/HITL] TEST-HITL-002: xai_explanation=NULL → 발행 차단 + 개발팀 알림"
labels: 'test, security, priority:must, epic:hitl-safety'
---

### :test_tube: Acceptance Criteria
- **Given**: `xai_explanation = null`인 AUDIT_REPORT
- **When**: 승인 시도
- **Then**: ①403 차단 ②"XAI 설명 없는 리포트" 에러 ③개발팀 CRITICAL 알림

---

## TEST-HITL-003: XAI 헬스체크 3회 실패 → 수동 전환 ≤5분

---
title: "[Test/HITL] TEST-HITL-003: 헬스체크 3회 실패 → MANUAL_FALLBACK ≤5분 + 복구 30분"
labels: 'test, integration, priority:must, epic:hitl-safety'
---

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 3회 연속 실패 → 전환**
- **Given**: 헬스체크 3회 연속 false 반환 (Mock)
- **When**: 모드 매니저 업데이트
- **Then**: ①`MANUAL_FALLBACK` 전환 ②COO/OPERATOR 알림 ③감사 로그

**Scenario 2: 복구 → 30분 후 복귀**
- **Given**: `MANUAL_FALLBACK` 상태 + 헬스체크 성공
- **When**: 30분 경과 (타이머 Mock)
- **Then**: `AI_ACTIVE` 복귀 + 복귀 알림

---

## TEST-HITL-004: STOP/CHANGE 무승인 → 즉시 차단

---
title: "[Test/HITL] TEST-HITL-004: STOP/CHANGE + approval_id 없음 → 차단 ≤1초"
labels: 'test, security, priority:must, epic:hitl-safety'
---

### :test_tube: Acceptance Criteria

**매트릭스 테스트** (4건):
| action_type | approval_id | 기대 결과 |
|:---:|:---:|:---|
| STOP | NULL | ❌ 차단 + 감사 로그 + CISO |
| STOP | 유효 | ✅ 통과 |
| CHANGE | NULL | ❌ 차단 + 감사 로그 + CISO |
| CHANGE | 만료 | ❌ 차단 (만료된 승인) |

---

## TEST-HITL-005: 30분 미처리 → COO 에스컬레이션

---
title: "[Test/HITL] TEST-HITL-005: 30분 미처리 PENDING → COO 자동 에스컬레이션"
labels: 'test, integration, priority:must, epic:hitl-safety'
---

### :test_tube: Acceptance Criteria
- **Given**: PENDING 건 (created_at = 35분 전)
- **When**: 에스컬레이션 스캐너 실행
- **Then**: ①COO 알림 발송 ②`escalated_at` 기록 ③25분 경과 건은 미에스컬레이션

---

## TEST-HITL-006: Reject 후 원본 보존 무결성 검증

---
title: "[Test/HITL] TEST-HITL-006: Reject 후 raw_data SHA-256 무결성 + 위반 알림"
labels: 'test, security, priority:should, epic:hitl-safety'
---

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 무결성 정상**
- **Given**: REJECTED LOG_ENTRY + 정상 해시
- **When**: 무결성 검증 스크립트 실행
- **Then**: 위반 0건

**Scenario 2: 변조 감지**
- **Given**: REJECTED LOG_ENTRY의 `raw_data`를 인위 변경
- **When**: 검증 실행
- **Then**: ①`DATA_INTEGRITY_VIOLATION` 감지 ②CISO + ADMIN 알림 ≤10초

---

# 전체 테스트 실행 가이드

## 실행 명령어

```bash
# 전체 테스트 실행
npm test

# Epic별 실행
npm test -- --testPathPattern=e1/
npm test -- --testPathPattern=e2/
npm test -- --testPathPattern=e2b/
npm test -- --testPathPattern=e3/
npm test -- --testPathPattern=e4/
npm test -- --testPathPattern=security/   # E6
npm test -- --testPathPattern=e7/
npm test -- --testPathPattern=hitl/

# 보안 매트릭스 전용
npm test -- --testPathPattern=rbac-matrix

# 스트레스 테스트 (별도 실행)
npm test -- --testPathPattern=stress
```

## 테스트 우선순위 (CI 파이프라인 순서)

| 순서 | 범위 | 건수 | 예상 시간 |
|:---:|:---|:---:|:---:|
| 1 | HITL 안전 프로토콜 | 6건 | ~2h |
| 2 | E6 보안 | 4건 | ~3h |
| 3 | E1 패시브 로깅 | 7건 | ~2.5h |
| 4 | E2 감사 리포트 | 5건 | ~2h |
| 5 | E2B XAI 이상탐지 | 5건 | ~2h |
| 6 | E3 ERP 브릿지 | 5건 | ~2h |
| 7 | E4 ROI 진단 | 5건 | ~1.5h |
| 8 | E7 대시보드 | 5건 | ~1.5h |
| | **총합** | **42건** | **~16.5h** |
