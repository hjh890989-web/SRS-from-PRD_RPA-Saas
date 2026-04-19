# FactoryAI — E1 패시브 로깅 (STT + Vision) Issues (E1-CMD-001 ~ E1-UI-005)

> **Source**: SRS-002 Rev 2.0 (V0.8) — E1 무입력 패시브 로깅  
> **작성일**: 2026-04-19  
> **총 Issue**: 12건 (Command 5건 + Query 2건 + UI 5건)  
> **목적**: 작업자의 추가 입력 없이 현장 데이터(음성, 이미지)를 수집하여 AI로 자동 구조화(LOG_ENTRY)하고, 인간 승인(HITL) 워크플로를 구축한다.

> [!IMPORTANT]
> E1 Epic은 FactoryAI의 데이터가 유입되는 **가장 핵심적인 진입점(Ingestion Layer)** 입니다.  
> Vercel AI SDK(AI-001)에 강하게 의존하며, 모든 생성 건은 PENDING 상태로 적재되어 HITL 안전망(AUTH-004, APPROVAL)을 거쳐야 합니다.

---

## E1-CMD-001: [Command] 버튼 녹음 → STT 변환 + LOG_ENTRY 저장

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature/E1] E1-CMD-001: 버튼 녹음 → Gemini STT 변환 + LOG_ENTRY 저장 (Server Action)"
labels: 'feature, backend, ai, priority:must, epic:e1-passive-logging'
assignees: ''
---

### :dart: Summary
- **기능명**: [E1-CMD-001] 버튼 녹음 → Gemini STT 변환 + LOG_ENTRY 저장
- **목적**: 작업 현장의 음성(PCM/WAV)을 전달받아 Vercel AI SDK (Gemini)를 통해 텍스트로 변환하고, 파싱된 JSON 결과를 `LOG_ENTRY` 테이블에 `PENDING` 상태로 저장한다.

### :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: REQ-FUNC-001 (음성 로깅)
- 의존성 기반: `AI-001` (AI 팩토리), `API-001` (Request/Response DTO)
- 데이터 모델: `DB-007` LOG_ENTRY

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `actions/logging/process-audio.ts` Server Action 스켈레톤 생성
- [ ] **2.** 오디오 파일 형식을 Zod(또는 Next.js File API)로 검증 (용량 ≤10MB, 오디오 MIME)
- [ ] **3.** Vercel AI SDK의 `generateText` 함수(또는 Gemini Multimodal) 호출하여 음성을 STT 처리
- [ ] **4.** STT 결과를 특정 JSON 스키마 구조 (온도, 압력, 진행상황 등)로 변환 지정
- [ ] **5.** 반환된 JSON 원시 데이터를 기반으로 Prisma `prisma.logEntry.create()` 호출 (`status="PENDING"`, `source_type="STT"`)
- [ ] **6.** 비즈니스 예외 처리 (AI 응답 타임아웃, 포맷 에러)
- [ ] **7.** 반환 DTO 보장: `{ id, status: "PENDING", captured_at }` 반환

### :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 정상적인 텍스트 변환 및 저장**
- **Given**: 라인 넘버와 "온도 180도, 압력 정상"이라고 말한 오디오 파일
- **When**: Server Action을 호출한다.
- **Then**: AI가 텍스트를 구조화({ temp: 180, pressure: "normal" })하고, DB에 `PENDING` 상태의 로그 엔트리가 생성된다.

**Scenario 2: 의미 불명확 오디오**
- **Given**: 공장 소음만 녹음된 오디오 파일
- **When**: Server Action을 호출한다.
- **Then**: AI가 파싱 실패를 반환하며, 클라이언트에게 422 또는 명확한 에러를 전달한다.

### :gear: Technical & Non-Functional Constraints
- 연동: `AI-002`의 Queue/Rate Limiter를 래핑하여 호출
- 타임아웃 방어: Next.js Server Action Timeout Limits 고려 최적화

### :checkered_flag: Definition of Done (DoD)
- [ ] 로직 구현 및 단위 테스트 (Mock AI 응답 기반) 통과 완료
- [ ] `LOG_ENTRY` 테이블에 JSON raw_data 정상 적재 검증
- [ ] MOCK-006 API 레퍼런스와 호환 완료

### :construction: Dependencies & Blockers
- **Depends on**: `AI-001`, `DB-007`, `AUTH-002`
- **Blocks**: `E1-UI-001` (음성 녹음 UI), `E1-CMD-005` (큐잉 순차 처리)

---

## E1-CMD-002: [Command] 카메라 촬영 → Vision 파싱 + LOG_ENTRY 저장

---
name: Feature Task
title: "[Feature/E1] E1-CMD-002: 모바일 카메라 촬영 → Gemini Vision 파싱 + LOG_ENTRY 저장 (Server Action)"
labels: 'feature, backend, ai, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-CMD-002] 모바일 카메라 촬영 → Gemini Vision 파싱 + LOG_ENTRY 저장
- **목적**: 디지털화되지 않은 아날로그 아날로그 계기판/장비 화면 이미지(JPEG/PNG)를 전송받아 Vercel AI SDK(Gemini Multimodal)로 값 객체 추출 후 `LOG_ENTRY` 테이블에 `PENDING` 저장한다.

### :link: References (Spec & Context)
- SRS: REQ-FUNC-002 (Vision 로깅)
- 기반 과제: `AI-001`, `API-001`

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `actions/logging/process-image.ts` Server Action 작성
- [ ] **2.** FormData 처리 및 Zod 유효성 검사 (MIME: image/jpeg, image/png, 용량 한도 확인)
- [ ] **3.** Vercel AI SDK `generateObject` 호출 (Image Parts 파라미터 활용)
- [ ] **4.** JSON Schema를 프롬프트로 주입하여 정형 데이터만 추출 (`{ temperature: number, error_codes: string[] }` 등)
- [ ] **5.** Prisma `logEntry.create` (`status="PENDING"`, `source_type="VISION"`) 처리
- [ ] **6.** 응답 객체 포맷팅 반환

### :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 선명한 계기판 이미지**
- **Given**: 온도계 지침이 '85'를 가리키는 고화질 사진
- **When**: Server Action을 호출한다.
- **Then**: AI가 `85`를 추출하고 DB에 PENDING 저장 후 ID를 반환한다.

### :checkered_flag: Definition of Done (DoD)
- [ ] Vercel AI SDK Multimodal 프롬프팅 구문 정상 적용 보장
- [ ] 단위 테스트 통과 (Mock Image Parts)

### :construction: Dependencies & Blockers
- **Depends on**: `AI-001`
- **Blocks**: `E1-CMD-004` (실패 시 알림), `E1-UI-002`

---

## E1-CMD-003: [Command] LOG_ENTRY Approve/Reject + 감사 로그

---
name: Feature Task
title: "[Feature/E1] E1-CMD-003: LOG_ENTRY Approve/Reject + 감사 로그 기록 (반영 ≤1초)"
labels: 'feature, backend, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-CMD-003] LOG_ENTRY 승인/거부 기능
- **목적**: PENDING 상태의 로그 항목을 관리자(또는 판단 인원)가 검토하고 APPROVED 하거나 REJECTED 할 수 있도록 상태를 바꾸는 Route Handler를 구현한다. 상태 변경과 함께 필수적으로 전수 감사 로그(AUTH-003 연동)가 남아야 한다.

### :link: References (Spec & Context)
- SRS: REQ-FUNC-003 (롤백 및 수정 권한), REQ-NF-023 (감사 로그 10초 이내 알림)
- API: `API-002` (PATCH 승인)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `app/api/v1/log-entries/[id]/approve/route.ts` 구현
- [ ] **2.** 요청 권한(RBAC) 검증 — `AUTH-002` withAuth() 데코레이터 적용
- [ ] **3.** `prisma.logEntry.update`로 상태(`APPROVED` / `REJECTED`) 및 `reviewer_id` 업데이트
- [ ] **4.** 만일 사용자가 AI 결론과 다른 수정된 데이터 페이로드를 전송했다면 `raw_data` 도 덮어쓰기 실시 (데이터 수정)
- [ ] **5.** `AUTH-003` Prisma Middleware가 `UPDATE` 이벤트를 인터셉트해 자동으로 감사 로그 저장함을 테스팅으로 검증

### :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 정상 승인**
- **Given**: `status=PENDING` 로그 ID
- **When**: `/approve` API에 `{ decision: "APPROVED" }` 페이로드를 패치한다
- **Then**: 200 반환, DB상 상태 업데이트 확인

**Scenario 2: 데이터 무단 수정 불가**
- **Given**: VIEWER 권한 사용자
- **When**: API를 호출한다
- **Then**: 403 반환 및 AUTH-004에 따른 CISO 알림 트리거 동작 확인

### :checkered_flag: Definition of Done (DoD)
- [ ] 로직 구현 및 권한 검증 기능 통과
- [ ] 변경 전후(before/after) 감사 로그 자동 기록 확인

### :construction: Dependencies & Blockers
- **Depends on**: `AUTH-002`, `AUTH-003`

---

## E1-CMD-004: [Command] Vision 파싱 실패 시 "재촬영 요청" 알림

---
name: Feature Task
title: "[Feature/E1] E1-CMD-004: Vision 파싱 실패 시 즉각 '재촬영 요청' 알림 발송 (≤3초)"
labels: 'feature, backend, priority:should, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-CMD-004] Vision 파싱 에러 핸들링 및 즉각 알림
- **목적**: E1-CMD-002에서 흔들림, 빛반사 등의 사유로 AI Vision 분석이 실패(Confidence 낮음, 포맷 에러)했을 때, 재촬영을 요구하는 알림(NOTI-001)을 3초 이내에 현장 모바일(OPERATOR)으로 발송한다.

### :link: References (Spec & Context)
- SRS: REQ-FUNC-006 (비전 실패 시나리오)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** E1-CMD-002 로직에 AI 파싱 Exception 캐치 블록 세분화
- [ ] **2.** AI 응답 내에서 거부(refusal) 또는 언파라미터화 실패 조건 감지
- [ ] **3.** 감지 시 `NOTI-001` (Notification DB Service) 호출
- [ ] **4.** 대상자: 해당 이미지를 전송한 `user_id` (세션 기반)
- [ ] **5.** 알림 Payload: `[화질 불량] 계기판 값이 식별되지 않습니다. 재촬영해주세요.`

### :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 흔들린 이미지 분석 시도**
- **Given**: 초점이 맞지 않아 숫자 판별이 불가능한 블러 이미지
- **When**: E1-CMD-002가 이를 `Unidentifiable`로 파싱 실패한다
- **Then**: 클라이언트에 에러 반환(≤3초)과 동시에 해당 사용자 NOTIFICATION Inbox에 Warning 등급 알림 저장

### :construction: Dependencies & Blockers
- **Depends on**: `E1-CMD-002`, `NOTI-001`

---

## E1-CMD-005: [Command] 동시 녹음 5건+ 큐잉 순차 처리

---
name: Feature Task
title: "[Feature/E1] E1-CMD-005: 동시 녹음 발송 시 큐잉 순차 처리 및 무결성 보장 (큐 드롭 0건)"
labels: 'feature, backend, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-CMD-005] 동시성 환경에서의 병목 회피 큐잉 보장
- **목적**: 동시에 여러 작업자 단말에서 STT/Vision 요청이 들어왔을 때 유실 없이 In-Memory Queue (AI-002)에 탑재하고, 15 RPM 한도 이내에서만 순차 처리를 보장한다.

### :link: References (Spec & Context)
- SRS: REQ-FUNC-008, REQ-NF-008

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** E1-CMD-001/002를 `AI-002` 스로틀러로 Wrapping한 구조 통합 테스트
- [ ] **2.** 동시 5건 이상의 Promise.all 스웜프 스트레스 테스트 스크립트 작성
- [ ] **3.** 메모리 누수 방지 점검
- [ ] **4.** (Optional) HTTP 클라이언트(프론트엔드) 타임아웃 제한 연장 대응

### :checkered_flag: Definition of Done (DoD)
- [ ] 모의 스크립트로 동시 10건 요청 시 유실 0건 보장 및 순차적 200 반환 확인

---

## E1-QRY-001: [Query] 일별 결측률 리포트 조회 API

---
name: Feature Task
title: "[Feature/E1] E1-QRY-001: 일별 결측률 리포트 자동 집계 API"
labels: 'feature, backend, query, priority:should, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-QRY-001] 일별 자동 집계 결측률 조회
- **목적**: 월간/일간 전체 계획 대비 누락된(로깅되지 않은 혹은 REJECT된) 로그 건수를 집계하여 백분율(%) 지표를 반환한다. (SVC-1 온보딩 시 결측률 ≤15% 추적용)

### :link: References (Spec & Context)
- SRS: REQ-FUNC-004
- API 계약: `API-003`

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `GET /api/v1/log-entries/missing-rate` Route Handler
- [ ] **2.** Prisma Aggregate 쿼리 작성 (일자 기준 그룹화)
  - 전체 스케줄된(Plan) 건수 가져오기 (WorkOrder 참조 추정)
  - 실제 적재된 `APPROVED` 건수 카운트
  - 수식: `(Plan - APPROVED) / Plan * 100` 계산
- [ ] **3.** JSON Array 응답 (날짜, 계획 건, 달성 건, 누락률)
- [ ] **4.** 캐싱 최적화 (`Next.js revalidate: 3600`) — 대시보드 렌더링 성능 확보

### :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 정상 집계 응답**
- **Given**: 지난주 100건 중 95건 입력, 금주 100건 중 80건 입력 데이터
- **When**: 14일치 결측률 조회를 요청한다.
- **Then**: 200 데이터 배열 반환, 금주 누락률은 20%로 평가된다.

---

## E1-QRY-002: [Query] 롤백/수정 웹 뷰어 (PENDING 목록)

---
name: Feature Task
title: "[Feature/E1] E1-QRY-002: PENDING 권한자 롤백/수정 웹 뷰어용 목록 조회 API"
labels: 'feature, backend, query, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-QRY-002] 롤백/수정 웹 뷰어 화면 데이터 공급 쿼리
- **목적**: HITL 승인이 필요한 PENDING 및 최근 승인된 LOG_ENTRY 데이터를 페이지네이션, 필터, 정렬 기능이 포함된 표 형태로 프론트엔드에 제공한다.

### :link: References (Spec & Context)
- SRS: REQ-FUNC-003, CLI-04
- 연관 API: 유사 Endpoint `API-001`의 GET 메서드 부문

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `GET /api/v1/log-entries` (쿼리 스트링 파싱: `?status=PENDING&page=1&limit=20`)
- [ ] **2.** Prisma `findMany` + `count`로 데이터와 Meta 정보 구성
- [ ] **3.** Response 포맷 `PaginationMeta` 타입 준수 여부 검증
- [ ] **4.** 권한 검증: ADMIN, AUDITOR 만 접근 허용 (`AUTH-002` 연동)

---

## E1-UI-001 ~ E1-UI-005: 프론트엔드 핵심 컴포넌트

---
name: Feature Task
title: "[Feature/E1] E1-UI-001: 버튼 녹음 + 오디오 레코더 + 녹음 상태 UI"
labels: 'feature, frontend, ui, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-UI-001] 녹음/음성 수집 클라이언트 UI
- **목적**: 작업자가 원클릭으로 마이크 레코딩을 시작하고, 웹 오디오 데시벨을 시각화하여 정상 수음 중임을 인지시키며 발송을 완료하게 한다.

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** React MediaRecorder API 기반 Custom Hook 작성 (`useAudioRecorder`)
- [ ] **2.** 녹음 버튼 (Mic 모양 아이콘) 및 펄스(Pulse) 애니메이션 구현
- [ ] **3.** 완료 버튼 클릭 시 `Blob` 데이터를 FormData로 변환 → E1-CMD-001 호출
- [ ] **4.** 로딩 중 `AI-003` 컴포넌트(처리 중 표시) 렌더링 호출
- [ ] **5.** 모바일(터치 스크린) 최적화

---
name: Feature Task
title: "[Feature/E1] E1-UI-002: 카메라 촬영/이미지 업로드 + 파싱 결과 미리보기"
labels: 'feature, frontend, ui, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-UI-002] 모바일 카메라 촬영 통합 UI
- **목적**: 작업용 태블릿 또는 휴대폰에서 직접 `<input type="file" accept="image/*" capture="environment">`를 활용해 즉시 촬영하고 전송한다.

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** 카메라 호출 Input 및 촬영 이미지 임시 Thumbnail 표시 UI
- [ ] **2.** 이미지 최적화 (브라우저 사이드 용량/리사이즈 압축 로직 - 옵션)
- [ ] **3.** E1-CMD-002 호출 및 `AI-003`(처리 표시기) 연동
- [ ] **4.** E1-CMD-004 `재촬영 알림` 수신 시 토스트 경고 및 재촬영 버튼 진입 플로우 제공

---
name: Feature Task
title: "[Feature/E1] E1-UI-003: 롤백/수정 웹 뷰어 (CLI-04)"
labels: 'feature, frontend, ui, priority:must, epic:e1-passive-logging'
---

### :dart: Summary
- **기능명**: [E1-UI-003] 롤백/수정 웹 뷰어 화면
- **목적**: 관리자가 PENDING 건을 리스트업하고, AI 추출 데이터를 검토 후 승인(Approve)하거나 직접 수정 후 반려(Reject)하는 휴먼 승인 워크스페이스(CLI-04).

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** Shadcn/ui `DataTable` 베이스 스켈레톤 구축
- [ ] **2.** E1-QRY-002를 통해 데이터 패칭 및 Pagination 연동
- [ ] **3.** Row 클릭 시 상세 패널(좌측: 원본 Image/Audio, 우측: AI 추출 JSON Edittable Form) 노출
- [ ] **4.** Approve (E1-CMD-003), Reject 버튼에 Mutation 바인딩
- [ ] **5.** 감사 로그 타임라인 뷰 컴포넌트 삽입

---
name: Feature Task
title: "[Feature/E1] E1-UI-004: 결측률 리포트 대시보드"
labels: 'feature, frontend, ui, priority:should, epic:e1-passive-logging'
---
### :dart: Summary
- 목적: E1-QRY-001 API 배열 데이터를 Recharts 기반 시계열 꺾은선(Line) 그래프로 렌더링하고, 목표선(≤15% 가이드)을 빨간 점선으로 시각화.

---
name: Feature Task
title: "[Feature/E1] E1-UI-005: '연결 끊김' 안내 UI"
labels: 'feature, frontend, ui, priority:should, epic:e1-passive-logging'
---
### :dart: Summary
- 목적: REQ-FUNC-007 준수. 클라이언트가 오프라인이 될 때 "네트워크 단절: 연결 복구 후 자동 재전송됩니다" 배너 노출. 로컬 스토리지에 데이터를 백업할 옵셔널한 Hook(`useNetworkSync`) 기초 적용.
