# SRS V0.5 AI 작업 지시서

> 본 문서는 AI 에이전트가 `11_SRS_V0.4.md`를 입력으로 받아 `14_SRS_V0.5.md`를 출력하기 위한 **순차적 편집 명령서**입니다.
> 각 TASK는 독립적이며, TASK-01부터 순서대로 실행합니다.

## 전제 조건

- **입력 파일**: `SRS-Drafts/11_SRS_V0.4.md` (1,606줄)
- **출력 파일**: `SRS-Drafts/14_SRS_V0.5.md` (신규 생성)
- **결정사항**:
  - 배포: Vercel → **Cloudflare Pages (무료)**
  - 저장: 이미지/오디오 원본 → **Supabase Storage 저장** (500MB DB + 1GB Storage)
  - AI: Gemini Free Tier + **큐잉(재시도)** 기본. PoC 시 **유료 전환($15/월)**
- **원칙**: V0.4의 모든 REQ ID를 보존한다. 삭제하지 않고 `[PROD-전용]` 또는 `[Phase 2]` 태그를 부여한다.

---

## TASK-01: 파일 복사 및 헤더 수정

**위치**: 1~10줄

**작업**: `11_SRS_V0.4.md`를 `14_SRS_V0.5.md`로 복사한 후 헤더를 수정한다.

**Before**:
```
**Revision**: 1.3 (V0.4 — Tech Stack Transition)  
**Date**: 2026-04-18  
```

**After**:
```
**Revision**: 1.4 (V0.5 — MVP Feasibility Alignment)  
**Date**: 2026-04-18  
```

**Before**:
```
**Base**: SRS V0.3 + 기술 스택 전환 구현 계획서(PLAN-SRS-V04)  
```

**After**:
```
**Base**: SRS V0.4 + MVP 개발목표 적절성 종합 검토 보고서(REVIEW-MVP-001) + AI 작업 지시서(PLAN-SRS-V05)  
```

---

## TASK-02: §1.2.3 Constraints — CON-04, CON-07 수정 + CON-12 추가

**위치**: §1.2.3 Constraints 테이블

### TASK-02-1: CON-04 수정

**Before**:
```
| CON-04 | [MVP] Vercel Git Push 자동 배포. [PROD] USB/내부망 Docker 이미지 배포 | 모드별 | ADR-4 (수정) |
```

**After**:
```
| CON-04 | [MVP] Cloudflare Pages Git Push 자동 배포 (무료). [PROD] USB/내부망 Docker 이미지 배포 | 모드별 | ADR-4 (수정) |
```

### TASK-02-2: CON-07 수정

**Before**:
```
| CON-07 | [MVP] Vercel 서버리스 한도 내 실행. [PROD] 고객사 서버 최소 사양: 16GB RAM, 4core (GPU T4+ 권장) | 모드별 | PRD §5-1 |
```

**After**:
```
| CON-07 | [MVP] Cloudflare Pages Functions 한도 내 실행 (CPU 10ms~50ms/호출, 무료 100,000 호출/일). 장시간 작업(PDF)은 클라이언트 처리. [PROD] 고객사 서버 최소 사양: 16GB RAM, 4core (GPU T4+ 권장) | 모드별 | PRD §5-1 |
```

### TASK-02-3: CON-12 신설 (CON-11 아래에 행 추가)

**추가할 행**:
```
| CON-12 | [MVP] 모든 클라우드 서비스는 무료 티어(Cloudflare Pages Free, Supabase Free 500MB DB + 1GB Storage, Gemini Free Tier 10 RPM)로 운영한다. 유료 전환은 PoC 고객사 확보 시 Gemini 유료($15/월)만 선행 적용 | MVP | ADR-11 |
```

---

## TASK-03: §1.2.4 Assumptions — ASM-01 수정 + ASM-08 추가

**위치**: §1.2.4 Assumptions 테이블

### TASK-03-1: ASM-01 수정

**Before**:
```
| ASM-01 | [MVP] Vercel + Supabase 서비스 가용. [PROD] 고객사에 서버(16GB RAM+, GPU 옵션) 확보 또는 조달 가능 | SW |
```

**After**:
```
| ASM-01 | [MVP] Cloudflare Pages(무료) + Supabase Free(500MB DB, 1GB Storage) + Gemini Free Tier(10 RPM Flash) 서비스 가용. SLA 보장 없음(Best Effort). PoC 시 Gemini유료 전환($15/월). [PROD] 고객사 서버 확보 | SW |
```

### TASK-03-2: ASM-08 신설 (ASM-07 아래에 행 추가)

**추가할 행**:
```
| ASM-08 | [MVP] PoC 고객 1개사, 실 사용자 3~5명 규모. 동시접속 30명은 PROD 전환 시 적용. 이미지/오디오 원본은 Supabase Storage에 저장하며 3개월 PoC 기간 중 1GB 이내 유지 | SW |
```

---

## TASK-04: §3.2 Component Diagram — Vercel → Cloudflare Pages 교체

**위치**: §3.2의 mermaid 다이어그램 내부

**작업**: 다이어그램 내의 모든 `Vercel` 텍스트를 `Cloudflare Pages`로 교체한다.

- `Vercel` → `Cloudflare Pages`
- `Vercel AI SDK` → `Vercel AI SDK` (이것은 npm 패키지명이므로 **변경하지 않는다**)
- 다이어그램 내 `Vercel Git Push` → `Cloudflare Pages Git Push`

---

## TASK-05: §3.3 API Overview — 설명 문구 수정

**위치**: §3.3 API Overview 설명 callout

**Before**:
```
> 모든 API는 Next.js Route Handlers(`/app/api/v1/*/route.ts`)로 구현됩니다. 인증은 NextAuth.js Middleware를 통해 RBAC 5역할로 제어합니다.
```

**After**:
```
> 모든 API는 Next.js Route Handlers(`/app/api/v1/*/route.ts`)로 구현되며, Cloudflare Pages Functions로 배포됩니다. 인증은 NextAuth.js Middleware를 통해 RBAC 5역할로 제어합니다. Gemini API 호출은 큐잉(자동 재시도, 지수 백오프)을 적용하여 Free Tier RPM 한도(10 RPM)를 준수합니다.
```

---

## TASK-06: §4.1.1 E1 REQ-FUNC — 난이도 조정 (5건)

**위치**: §4.1.1 E1 — 무입력 패시브 로깅

### TASK-06-1: REQ-FUNC-001 수정

기존 Requirement 텍스트 앞에 `[MVP]`/`[PROD]` 구분을 삽입한다.

**Before** (Requirement 열만):
```
시스템은 80dB+ 소음 환경에서 작업자의 트리거 워드 음성 발화를 STT 모듈로 감지하여 공정 상태를 텍스트로 변환·로깅해야 한다
```

**After**:
```
[MVP] 시스템은 작업자가 녹음 버튼을 눌러 음성을 녹음하면 Gemini STT API로 공정 상태를 텍스트로 변환·로깅해야 한다. [PROD] 80dB+ 소음 환경에서 트리거 워드 자동 감지로 전환한다
```

Acceptance Criteria도 수정:

**Before** (AC 열만):
```
**Given** 80dB+ 소음 환경에서 작업자가 등록된 트리거 워드를 발화한다 **When** STT 모듈이 트리거 워드를 감지한다 **Then** 공정 상태가 텍스트로 변환·로깅된다 (인식 정확도 ≥90%, 지연 ≤2초)
```

**After**:
```
[MVP] **Given** 작업자가 녹음 버튼을 누른다 **When** 음성이 Gemini STT로 전송된다 **Then** 공정 상태가 텍스트로 변환·로깅된다 (인식 정확도 ≥85%, 지연 ≤5초). [PROD] 트리거 워드 자동 감지 (정확도 ≥90%, 지연 ≤2초)
```

### TASK-06-2: REQ-FUNC-005 수정

**Before** (Requirement 열만):
```
시스템은 비등록 언어(영어·방언) 음성 발화 시 트리거 워드 미감지로 무시 처리하고, 오등록 0건을 보장해야 한다
```

**After**:
```
[MVP] 버튼 녹음 방식이므로 해당 없음(N/A). [PROD] 시스템은 비등록 언어 음성 발화 시 트리거 워드 미감지로 무시 처리하고, 오등록 0건(False Positive ≤2%)을 보장해야 한다
```

### TASK-06-3: REQ-FUNC-007 수정

**Before** (Requirement 열만):
```
시스템은 네트워크 단절 시 로컬 큐에 데이터를 저장하고, 복구 후 ≤5분 내 전량 동기화해야 한다 (유실률 0%)
```

**After**:
```
[PROD-전용] 시스템은 네트워크 단절 시 로컬 큐에 데이터를 저장하고, 복구 후 ≤5분 내 전량 동기화해야 한다 (유실률 0%). [MVP] 클라우드 환경이므로 온라인 전제. 네트워크 단절 시 "연결 끊김" 안내 표시
```

### TASK-06-4: REQ-FUNC-008 수정

**Before** (Requirement 열만):
```
시스템은 동일 시간대 10건+ 동시 음성 입력 시 순차 처리를 완료하고, 큐 드롭 0건을 보장해야 한다
```

**After**:
```
[MVP] 시스템은 동일 시간대 5건+ 동시 버튼 녹음 시 큐잉 후 순차 처리를 완료하고, 큐 드롭 0건을 보장해야 한다 (최대 지연 ≤15초). [PROD] 10건+ 동시 입력, 지연 ≤10초
```

### TASK-06-5: §4.1.4 E3 REQ-FUNC-022 수정

**위치**: §4.1.4 E3

**Before** (Requirement 열만):
```
시스템은 ERP 스키마 변경(컬럼 추가/삭제/타입 변경) 감지 시 동기화를 중단하고 CIO에게 ≤1분 내 알림을 발송해야 한다
```

**After**:
```
[Phase 2] 시스템은 ERP 스키마 변경 감지 시 동기화를 중단하고 CIO에게 ≤1분 내 알림을 발송해야 한다. [MVP] 스키마 변경 자동 감지 미구현. 동기화 오류 발생 시 "동기화 실패" 알림 + 엑셀 업로드 폴백 안내
```

---

## TASK-07: §4.1.6 E6 — Vercel → Cloudflare Pages 교체

**위치**: §4.1.6 E6 보안 패키지 테이블 내

**작업**: 테이블 내 모든 `Vercel Git Push` → `Cloudflare Pages Git Push`로 교체한다.

- REQ-FUNC-029: `[MVP] Supabase RLS + HTTPS` 유지 (Cloudflare도 HTTPS 기본)
- REQ-FUNC-030: `[MVP] Vercel Git Push 자동 배포` → `[MVP] Cloudflare Pages Git Push 자동 배포`
- REQ-FUNC-033: `[MVP] 해당 없음 (Git Push CI/CD 자동 검증)` 유지

---

## TASK-08: §4.2 NFR 이중화 — MVP(Free Tier) / PROD 목표치 분리

**위치**: §4.2 Non-Functional Requirements 전체

### TASK-08-1: §4.2.1 성능 — 7건 수정

각 NFR의 Requirement 열과 목표 열을 MVP/PROD로 분리한다.

| NFR ID | 목표 열 Before | 목표 열 After |
|:---|:---|:---|
| NF-001 | `≤2,000ms` | `[MVP] ≤5,000ms / [PROD] ≤2,000ms` |
| NF-002 | `≤5,000ms` | `[MVP] ≤8,000ms / [PROD] ≤5,000ms` |
| NF-003 | `≤30,000ms` | 유지 (클라이언트 처리이므로 인프라 무관) |
| NF-004 | `≤3,000ms` | 유지 (Cloudflare CDN이 오히려 유리) |
| NF-005 | `≤5분` | 유지 |
| NF-006 | `≤3,000ms` | `[MVP] ≤5,000ms / [PROD] ≤3,000ms` |
| NF-007 | `30명` | `[MVP] 5명 (PoC 1개사) / [PROD] 30명` |
| NF-008 | `0건` (STT 10+Vision 5) | `[MVP] 0건 (STT 5+Vision 3/분, 큐잉 적용) / [PROD] STT 10+Vision 5/분` |
| NF-009 | `500GB/년` | `[MVP] Supabase Free 500MB DB + 1GB Storage (PoC 3개월) / [PROD] 500GB/년` |

Requirement 열에도 동일하게 `[MVP] ... [PROD] ...` 표기를 반영한다.

### TASK-08-2: §4.2.2 가용성 — 5건 수정

| NFR ID | 목표 열 Before | 목표 열 After |
|:---|:---|:---|
| NF-010 | `≥99.5%` | `[MVP] Best Effort (무료 플랜 SLA 없음) / [PROD] ≥99.5%` |
| NF-011 | `≤4시간/월` | 유지 (운영 정책이므로 인프라 무관) |
| NF-012 | `≤3.6시간/월` | `[MVP] Best Effort / [PROD] ≤3.6시간/월` |
| NF-013 | `≥720시간` | `[MVP] Best Effort / [PROD] ≥720시간` |
| NF-014 | `≤2시간` | `[MVP] ≤24시간 (1인 영업일 대응) / [PROD] ≤2시간` |

### TASK-08-3: §4.2.3 신뢰성 — 1건 수정

| NFR ID | 목표 열 Before | 목표 열 After |
|:---|:---|:---|
| NF-018 | `≤1시간` | `[MVP] ≤24시간 (수동 pg_dump 또는 Supabase 대시보드 Export) / [PROD] ≤1시간 (자동 백업)` |

### TASK-08-4: §4.2.4 보안 — Vercel → Cloudflare 교체

테이블 내 모든 `Vercel` 관련 텍스트를 교체한다:
- `Vercel 플랫폼 보안 정책 준수` → `Cloudflare Pages 보안 정책 준수`
- `Vercel 보안 대시보드` → `Cloudflare 보안 대시보드`

---

## TASK-09: §4.3 Sprint 매트릭스 신설

**위치**: §4.2.10 목표 KPI 다음, §5 이전에 새 섹션을 삽입한다.

**삽입할 내용**:

```markdown
### 4.3 MVP Sprint 매트릭스

> 초급 개발자(3개월 SW + 3년 IT + AI 보조)가 구현 가능한 순서로 배치합니다. 각 Sprint는 이전 Sprint의 산출물을 기반으로 합니다.

| Sprint | 기간 | 구현 대상 | 구현 REQ | 난이도 | 선행 조건 | 성공 기준 |
|:---:|:---:|:---|:---|:---:|:---|:---|
| **S0** | 1주 | 프로젝트 세팅: Next.js + Prisma + Supabase + NextAuth + Cloudflare Pages 배포 | — | 🟢 | 없음 | `npm run dev` 로컬 동작 + Cloudflare Pages 배포 성공 |
| **S1** | 2주 | E4 ROI 계산기 + E7 대시보드 기본 UI | FUNC-024~028, FUNC-035~039 | 🟢 | S0 | 바우처 매칭 계산 동작, 대시보드 렌더링 |
| **S2** | 2주 | E6 RBAC 인증 + Prisma 감사 로그 | FUNC-029~032, FUNC-034, NF-022~023 | 🟢🟡 | S0 | 5역할 로그인, 감사 로그 DB 기록 |
| **S3** | 3주 | E3 ERP 브릿지 (엑셀 업로드 우선, Tunnel 후순위) | FUNC-019~021, FUNC-023 | 🟡 | S2 | 엑셀 업로드 → DB 적재 → 정합성 리포트 |
| **S4** | 3주 | E2 감사 리포트 (PDF 생성 + Lot Merge) | FUNC-009~013 | 🟡 | S3 | 1클릭 PDF 다운로드 동작 |
| **S5** | 3주 | E1 패시브 로깅 (버튼 녹음 → Gemini STT + Vision) | FUNC-001~004, FUNC-006, FUNC-008 | 🟡 | S2 | 음성 녹음 → 텍스트 변환 → DB 기록 |
| **S6** | 2주 | E2-B XAI 이상탐지 + HITL 승인 워크플로 | FUNC-014~018 | 🟡 | S4, S5 | XAI 한국어 설명 생성, Approve/Reject |
| **S7** | 1주 | Docker 로컬 모드 (CISO 데모용) | FUNC-029(Demo), NF-020(Demo) | 🟡 | S6 | `docker compose up` → localhost 정상 |

> **총 소요**: ~17주(약 4개월). 72 REQ-FUNC 중 **47건 MVP 구현**, 25건은 `[PROD-전용]` 또는 `[Phase 2]`.

#### MVP 제외 REQ 목록 (PROD/Phase 2 연기)

| REQ ID | 사유 | 연기 대상 |
|:---|:---|:---:|
| FUNC-005 | MVP 버튼 녹음 방식에서 불필요 | PROD |
| FUNC-007 | 오프라인 큐 — Cloudflare 서버리스 불가 | PROD |
| FUNC-022 | ERP 스키마 자동 감지 — DBA 중급 역량 | Phase 2 |
| FUNC-033 | Docker 해시 검증 — MVP에서 Git CI/CD 대체 | PROD |
| NF-035 | 오프라인 모델 업데이트 — PROD 전용 | PROD |
| NF-036 | 스키마 패턴 라이브러리 — Phase 2 | Phase 2 |
| SVC 관련 REQ (FUNC-046~072) | 서비스 운영 REQ — SW 구현 대상 아님 | SVC |
```

---

## TASK-10: §7 ADR — ADR-11 신설

**위치**: ADR-10 다음에 추가

**삽입할 내용**:

```markdown
### ADR-11: Free Tier First + Cloudflare Pages (CON-12)

| 항목 | 내용 |
|:---|:---|
| **결정** | MVP는 Cloudflare Pages(무료) + Supabase Free(500MB) + Gemini Free Tier(10 RPM)로 $0 운영. 이미지/오디오 원본은 Supabase Storage(1GB)에 저장. PoC 고객사 확보 시 Gemini만 유료 전환($15/월) |
| **맥락** | 초기 투자 최소화 + PoC 실패 시 매몰 비용 $0. Vercel Hobby는 비상업 전용이므로 Cloudflare Pages를 대안으로 채택 |
| **근거** | CON-12, 검토 보고서(REVIEW-MVP-001) §3, AI 작업 지시서 Q1 결정 |
| **결과** | NFR을 MVP/PROD 이중 목표로 분리. MVP NFR은 Free Tier 한도 내. 동시접속 30→5명, SLA 99.5%→Best Effort |
| **트레이드오프** | Cloudflare Pages의 Next.js SSR 호환성 제약 → `@opennextjs/cloudflare` 어댑터 사용. Edge Runtime 기반이므로 일부 Node.js API 미지원 가능 |
| **관련 REQ** | NF-007~010, NF-014, NF-018, CON-04, CON-07, CON-12 |
```

---

## TASK-11: §7 Traceability — ADR-11 행 추가

**위치**: §5.3 ADR → REQ 추적표

**추가할 행**:
```
| ADR-11 | Free Tier First + Cloudflare Pages (CON-12) | NF-007~010, NF-014, NF-018 |
```

---

## TASK-12: §8 Risk Register — R16 신설

**위치**: R15 다음

**삽입할 내용**:

```markdown
| R16 | **Gemini Free Tier RPM 초과** → STT/Vision 일시 중단, 큐 지연 증가 | SW | 🟡 | 높 | (1) 요청 큐잉 + 지수 백오프 재시도 기본 적용. (2) Flash-Lite($0.10/1M) 전환으로 비용 최소화. (3) PoC 성공 시 유료 전환($15/월)으로 RPM 제한 해제 | NF-008, ADR-11, FUNC-001~002 |
```

---

## TASK-13: §10 Technology Stack — Cloudflare Pages 전환

**위치**: §10.1 MVP 기술 스택 테이블

### TASK-13-1: 배포 행 수정

**Before**:
```
| 배포 | Vercel | C-TEC-007 | Git Push 자동 배포 |
```

**After**:
```
| 배포 | Cloudflare Pages + `@opennextjs/cloudflare` | C-TEC-007 | Git Push 자동 배포 (무료, 상용 가능) |
```

### TASK-13-2: §10.2 PROD 전환 경로 수정

**Before**:
```
| Vercel | → | Docker Compose | `next build` → Docker 이미지 빌드 |
```

**After**:
```
| Cloudflare Pages | → | Docker Compose | `next build` → Docker 이미지 빌드 |
```

**Before**:
```
| Git Push 배포 | → | USB Docker 배포 | Docker 이미지 + docker-compose.yml 패키지 |
```

**After**:
```
| Cloudflare Pages Git Push | → | USB Docker 배포 | Docker 이미지 + docker-compose.yml 패키지 |
```

### TASK-13-3: §10.3 Docker 로컬 실행 모드 수정

**Before**:
```
| **Cloud** | `vercel dev` / `npm run dev` | Supabase | Gemini API | Vercel | 있음 |
```

**After**:
```
| **Cloud** | `npm run dev` / Cloudflare Pages | Supabase | Gemini API | Cloudflare Pages | 있음 |
```

**Before**:
```
> 환경변수 `.env.cloud` / `.env.local`로 전환. 코드 변경 0줄.
```

**After**:
```
> 환경변수 `.env.cloud` / `.env.local`로 전환. 코드 변경 0줄. Gemini API는 큐잉 미들웨어를 통해 Free Tier RPM(10 RPM) 한도를 자동 관리한다.
```

---

## TASK-14: §11 비용 산정 섹션 신설

**위치**: §10 다음, Footer 이전에 새 섹션을 삽입한다.

**삽입할 내용**:

```markdown
## 11. 비용 산정

### 11.1 MVP 인프라 비용 (3개 시나리오)

| 시나리오 | Cloudflare | Supabase | Gemini | 합계 (USD) | 합계 (KRW) |
|:---|:---:|:---:|:---:|:---:|:---:|
| **A: 개발/테스트** | Free | Free (500MB) | Free (10 RPM) | **$0/월** | **₩0** |
| **B: PoC 고객 1개사** | Free | Free (500MB) | 유료 Flash ~$15 | **~$15/월** | **~₩21,000** |
| **C: PROD 고객사** | $0 (Docker) | $0 (Local PG) | $0 (Ollama) | **$0/월** | **₩0** |

### 11.2 Gemini API 상세 비용 (PoC 기준)

| 용도 | 모델 | 일일 호출 | 토큰 (input/output) | 월 비용 |
|:---|:---|:---:|:---:|:---:|
| STT | Flash | 100건 | 500/200 | ~$0.57 |
| Vision | Flash | 50건 | 2,000/500 | ~$2.79 |
| XAI | Flash | 30건 | 1,000/800 | ~$2.07 |
| ROI/기타 | Flash | 70건 | 500/500 | ~$2.01 |
| **소계** | | **250건/일** | | **~$7.44/월** |

> 실제 비용 ≈ $8~15/월. 개발 기간 중 Free Tier로 $0 가능.

### 11.3 스토리지 사용량 예측 (Supabase Free)

| 데이터 유형 | 단건 크기 | 일일 건수 | 3개월 누적 | 저장 위치 |
|:---|:---:|:---:|:---:|:---|
| LOG_ENTRY (텍스트) | ~1 KB | 200건 | ~18 MB | DB (500MB) |
| 감사 로그 | ~0.5 KB | 500건 | ~22 MB | DB (500MB) |
| 오디오 원본 | ~200 KB | 100건 | ~540 MB | Storage (1GB) |
| 이미지 원본 | ~500 KB | 50건 | ~675 MB | Storage (1GB) |
| **합계** | | | DB ~40MB / Storage ~1.2GB | ⚠️ Storage 초과 주의 |

> [!WARNING]
> 이미지+오디오 원본 전체 저장 시 3개월 내 1GB Storage 초과 가능. **대응**: (1) 해상도 압축(이미지 200KB 이하), (2) 오디오 30초 제한으로 100KB 이하, (3) 초과 시 Supabase Pro($25/월) 전환 또는 오래된 원본 아카이브.
```

---

## TASK-15: Footer 업데이트

**위치**: 파일 마지막 3줄

**Before**:
```
*Document ID: SRS-002 / Revision: 1.3 (V0.4 — Tech Stack Transition) / Date: 2026-04-18 / Owner: 모두연 EGIGAE #5*  
*V0.4 변경: (1) Next.js + Vercel + Supabase + Gemini API 스택 전환, (2) MVP/PROD 이중 모드 제약사항, (3) Docker 로컬 실행 모드, (4) ADR-1/4 수정 + ADR-8~10 신설, (5) Cloudflare Tunnel ERP 연동, (6) §10 Technology Stack 신설*  
*총 요구사항: **72 REQ-FUNC**, **54 REQ-NF**, **14 Entities**, **9 Sequences**, **4 Diagrams**, **10 ADR**, **15 Risks**.*
```

**After**:
```
*Document ID: SRS-002 / Revision: 1.4 (V0.5 — MVP Feasibility Alignment) / Date: 2026-04-18 / Owner: 모두연 EGIGAE #5*  
*V0.5 변경: (1) Vercel → Cloudflare Pages(무료) 배포 전환, (2) NFR MVP/PROD 이중 목표치 분리(NF-001~018), (3) REQ-FUNC 난이도 조정 5건(FUNC-001/005/007/008/022), (4) §4.3 Sprint 매트릭스 신설(S0~S7, 17주), (5) CON-12/ASM-08 Free Tier First 제약 신설, (6) §11 비용 산정 섹션 신설, (7) ADR-11 + R16 추가, (8) 이미지/오디오 원본 Supabase Storage 저장 명시*  
*총 요구사항: **72 REQ-FUNC**(MVP 47건 + PROD/Phase2 25건), **54 REQ-NF**(이중 목표), **14 Entities**, **9 Sequences**, **4 Diagrams**, **11 ADR**, **16 Risks**.*
```

---

## TASK-16: Out-of-Scope 수정

**위치**: §1.2.2 Out-of-Scope 테이블

**Before**:
```
| 퍼블릭 클라우드 배포 | ADR-1에 의한 On-Premise Only |
```

**After**:
```
| 퍼블릭 클라우드 유료 배포 | MVP는 Cloudflare Pages(무료). PROD는 On-Premise Docker (ADR-1) |
```

---

## 실행 순서 요약

```
TASK-01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12 → 13 → 14 → 15 → 16
 헤더    CON   ASM   다이어그램  API   E1/E3   E6    NFR  Sprint  ADR   추적   Risk  Tech  비용  Footer  Scope
```

> 모든 TASK 완료 후, 전체 문서에서 `Vercel`로 남아있는 텍스트를 최종 검색하여 컨텍스트에 따라 `Cloudflare Pages`(배포 플랫폼) 또는 유지(`Vercel AI SDK` — npm 패키지)로 판단한다.
