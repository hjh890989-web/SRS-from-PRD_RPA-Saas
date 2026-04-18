# SRS V0.3 기술 스택 전환 영향도 분석 보고서

**검토 대상**: `08_SRS_V0.3.md` (SRS-002, Rev 1.2)  
**검토 목적**: 제안 기술 스택(Next.js Fullstack + Vercel + Supabase + Gemini API) 전환 시 SRS 기능적 커버리지 및 아키텍처 충돌 분석  
**검토일**: 2026-04-18  
**검토자**: AI Requirements Engineer  

---

## 제안 기술 스택

```
[Assumptions & Constraints]

(시스템 내부 - 단일 통합 프레임워크)
C-TEC-001: 모든 서비스는 Next.js (App Router) 기반의 단일 풀스택 프레임워크로 구현한다.
C-TEC-002: 서버 측 로직은 Next.js의 Server Actions 또는 Route Handlers를 사용한다.
C-TEC-003: 데이터베이스는 Prisma + 로컬 SQLite(개발) / Supabase PostgreSQL(배포)을 사용한다.
C-TEC-004: UI 및 스타일링은 Tailwind CSS와 shadcn/ui를 사용한다.

(시스템 외부 - 연결 및 AI 통합)
C-TEC-005: LLM 오케스트레이션은 Vercel AI SDK를 사용하여 Next.js 내부에서 구현한다.
C-TEC-006: LLM 호출은 Google Gemini API를 기본으로 사용한다.
C-TEC-007: 배포 및 인프라 관리는 Vercel 플랫폼으로 단일화한다.
```

---

## 1. 핵심 충돌 분석

### 🔴 Critical: 보안 아키텍처 충돌

| SRS/PRD 원칙 | 현재 SRS | 제안 스택 | 충돌 |
|:---|:---|:---|:---:|
| **ADR-1** On-Premise Only | 외부 트래픽 0 byte | Vercel(클라우드) + Supabase(클라우드) | 🔴 |
| **CON-01** 외부 API 호출 0건 | 폐쇄망 전용 | C-TEC-006 Gemini API (외부 호출) | 🔴 |
| **REQ-NF-020** 외부 트래픽 0 byte | 24h 네트워크 캡처 검증 | 모든 요청이 Vercel → Supabase → Gemini 경유 | 🔴 |
| **REQ-NF-021** 100% 온프레미스 AI | On-Prem Whisper/Vision/LLM | Vercel AI SDK + Gemini (클라우드 AI) | 🔴 |
| **ADR-4** USB 오프라인 배포 | 에어갭, 해시 검증 | C-TEC-007 Git Push 자동 배포 | 🔴 |
| **CISO 공략 전략** | "데이터 한 바이트도 외부 유출 없음" | 고객 데이터가 Vercel/Supabase/Google에 전송 | 🔴 |

> **핵심**: 현재 PRD의 **전체 영업 전략이 "CISO를 설득하는 100% 온프레미스"에 기반**하고 있습니다. 클라우드 스택으로 전환하면 CISO 관문(DMU 의사결정의 최종 관문)을 통과할 수 없으며, 이는 PRD가 정의한 **P5(보안 정책 장벽: SaaS AI 제안 거절률 100%)**와 직접 충돌합니다.

---

## 2. 모듈별 기술 커버리지 분석

### ✅ 잘 매핑되는 영역 (UI/비즈니스 로직)

| SRS 모듈 | 기존 | 신규 스택 매핑 | 커버리지 |
|:---|:---|:---|:---:|
| **CLI-01~04** (4개 웹 클라이언트) | React 웹앱 | Next.js App Router + shadcn/ui + Tailwind | ✅ **100%** |
| **E4 ROI 진단·결재기** | 웹 계산기 | Next.js Page + Server Actions | ✅ **100%** |
| **E7 성과 대시보드** | 웹 대시보드 | Next.js + shadcn/ui 차트 컴포넌트 | ✅ **95%** |
| **E2 감사 리포트** (PDF) | PDF 엔진 | Server Actions + react-pdf/puppeteer | ✅ **90%** |
| **RBAC 인증** | API Gateway | NextAuth.js + Middleware | ✅ **90%** |
| **Data Model** (14 엔터티) | PostgreSQL | Prisma Schema → SQLite(dev) / Supabase(prod) | ✅ **100%** |
| **API Endpoints** (19개) | REST API | Next.js Route Handlers (`/api/v1/*`) | ✅ **100%** |

### ✅ 매핑 상세: 기술 스택 ↔ SRS 매핑 테이블

| C-TEC | 제안 기술 | SRS 대체 대상 | 적용 REQ |
|:---|:---|:---|:---|
| C-TEC-001 | Next.js (App Router) | 관리자 웹 대시보드(CLI-01), ROI 계산기(CLI-02), CISO 콘솔(CLI-03), 롤백 뷰어(CLI-04) | REQ-FUNC-003, 024~028, 032, 035~039 |
| C-TEC-002 | Server Actions / Route Handlers | API Gateway + Core Service Layer | 전체 19개 API Endpoint |
| C-TEC-003 | Prisma + SQLite / Supabase | PostgreSQL + 14개 엔터티 스키마 | §6.2 전체 Data Model |
| C-TEC-004 | Tailwind CSS + shadcn/ui | UI 컴포넌트 + 디자인 시스템 | 대시보드 렌더링 ≤5초 (REQ-NF-004) |
| C-TEC-005 | Vercel AI SDK | On-Prem AI 엔진 (STT, Vision, XAI) | REQ-FUNC-001~002, 014~018, 042 |
| C-TEC-006 | Google Gemini API | Whisper STT + Vision 파서 + XAI LLM | EXT-03, EXT-04, EXT-05 대체 |
| C-TEC-007 | Vercel 배포 | Docker On-Prem + USB 업데이트 | REQ-FUNC-029~030, REQ-NF-035 대체 |

### ⚠️ 부분 커버되는 영역 (AI/연동)

| SRS 모듈 | 기존 | 신규 스택 매핑 | 커버리지 | 제한 사항 |
|:---|:---|:---|:---:|:---|
| **E1 패시브 로깅 (STT)** | On-Prem Whisper | Gemini API 음성 인식 via Vercel AI SDK | ⚠️ **70%** | 80dB+ 소음 환경 파인튜닝 불가, 트리거 워드 커스텀 제한 |
| **E1 패시브 로깅 (Vision)** | On-Prem Vision Parser | Gemini API 멀티모달 비전 | ⚠️ **75%** | 계기판 특화 파싱 정확도 미검증 |
| **E2-B XAI 이상탐지** | On-Prem LLM | Gemini API via Vercel AI SDK | ⚠️ **80%** | 한국어 XAI 설명 품질은 양호하나 지연시간 변동 |
| **E3 ERP 브릿지** | Read-Only DB 커넥터 | Prisma로 FactoryAI DB만 관리 + 엑셀 임포트 | ⚠️ **60%** | 고객사 ERP DB 직접 연결은 클라우드→온프레미스 VPN 필요 |

### ❌ 커버 불가능한 영역

| SRS 모듈 | 영향 REQ | 사유 |
|:---|:---|:---|
| **E6 온프레미스 보안 패키지** | REQ-FUNC-029~034 (6건) | Vercel/Supabase 자체가 클라우드. "외부 트래픽 0 byte" 불가 |
| **USB 오프라인 모델 업데이트** | REQ-FUNC-030, 033 (2건) | Git Push 배포와 호환 불가 |
| **24h 네트워크 모니터링** | REQ-NF-020~021 (2건) | 클라우드 환경에서 "외부 0 byte" 검증 불가 |
| **로컬 큐 (네트워크 단절 대비)** | REQ-FUNC-007 (1건) | Vercel Edge 환경에서 로컬 큐 운영 불가 |
| **SVC-3 보안 심의 동행** | REQ-FUNC-053~054 (2건) | "100% 폐쇄망" 근거 소멸 → 심의 통과 논리 무효화 |
| **ISMS 확인서 자동 생성** | REQ-NF-024 (1건) | 망분리 설계서의 전제 조건 소멸 |
| **Docker CVE 스캔** | REQ-NF-026 (1건) | Vercel 서버리스 환경에서 Docker 미사용 |

---

## 3. REQ 영향도 정량 분석

### 3.1 REQ-FUNC 영향 (72건)

| 상태 | 건수 | 비율 | 상세 |
|:---|:---:|:---:|:---|
| ✅ 완전 커버 | 42건 | 58% | E4, E7, SVC-1~5, HITL 일부 |
| ⚠️ 부분 커버 (기능은 가능, 성능/정확도 차이) | 13건 | 18% | E1(STT/Vision), E2-B(XAI), E3(ERP) |
| ❌ 커버 불가 / 전면 재작성 필요 | 17건 | 24% | E6(보안), 네트워크 단절 대비, USB 배포 |

### 3.2 REQ-NF 영향 (54건)

| 상태 | 건수 | 비율 | 상세 |
|:---|:---:|:---:|:---|
| ✅ 완전 커버 | 35건 | 65% | 성능, 비용, SLA, 목표 KPI |
| ⚠️ 부분 커버 (측정 방法 변경 필요) | 7건 | 13% | 가용성(Vercel SLA 의존), 신뢰성 |
| ❌ 커버 불가 / 전면 재작성 필요 | 12건 | 22% | 보안(외부 트래픽·RBAC·감사 로그) |

### 3.3 종합 커버리지

```
┌─────────────────────────────────────────────┐
│   기술 스택 전환 시 기능적 커버리지          │
│                                             │
│  UI/웹 클라이언트  ████████████████████ 100% │
│  API/데이터 모델   ████████████████████ 100% │
│  비즈니스 로직     ██████████████████░░  90% │
│  AI 기능           ██████████████░░░░░░  75% │
│  ERP 연동          ████████████░░░░░░░░  60% │
│  보안/온프레미스   ██░░░░░░░░░░░░░░░░░░  10% │
│                                             │
│  ▸ 비보안 영역: ~90% 커버                   │
│  ▸ 보안 영역: ~10% 커버 (근본적 충돌)       │
│  ▸ 종합: ~77% (29건 REQ 미충족)             │
└─────────────────────────────────────────────┘
```

---

## 4. ADR 영향도

| ADR | 현재 결정 | 제안 스택 호환 | 필요 조치 |
|:---|:---|:---:|:---|
| ADR-1 | On-Premise Only | ❌ | 폐기 or "Demo Mode" 분리 |
| ADR-2 | Read-Only ERP | ⚠️ | 엑셀 임포트만 지원, DB 직접 연결 불가 |
| ADR-3 | HITL 4대 원칙 | ✅ | 그대로 유지 가능 |
| ADR-4 | USB 오프라인 배포 | ❌ | 폐기 → Git Push 배포로 전환 |
| ADR-5 | Zero-Touch UX | ⚠️ | Gemini 멀티모달로 대체 가능하나 정확도 차이 |
| ADR-6 | 바우처 번들링 | ✅ | 그대로 유지 가능 |
| ADR-7 | E5 Won't (Phase 2) | ✅ | 그대로 유지 |

---

## 5. 권장 접근 방식

### 🎯 Option A: "PoC 데모용 MVP" + "프로덕션 온프레미스" 2-Track

| Track | 스택 | 목적 | 대상 |
|:---|:---|:---|:---|
| **Track 1: PoC Demo** | Next.js + Supabase + Gemini (제안 스택) | 영업 데모, 투자자 발표, 기능 검증 | 내부팀, CFO, COO |
| **Track 2: Production** | Docker On-Prem (현행 SRS) | 실제 고객 배포, CISO 심의 통과 | 고객사 현장 |

→ SRS에 **"Deployment Mode: DEMO / PRODUCTION"** 구분을 추가하고, 제안 스택은 DEMO 모드로 명시

**장점**: 빠른 기능 검증 + 프로덕션 보안 유지  
**단점**: 2벌 코드베이스 유지 부담

### 🎯 Option B: SRS 아키텍처 전면 전환 (클라우드 우선)

PRD의 **P5(보안 정책 장벽)과 ADR-1(On-Prem Only)을 폐기**하고, 클라우드 SaaS 모델로 전환.

**필요 작업**:
- CISO 영업 전략 전면 재설계
- "SaaS AI 제안 거절률 100%" 문제 해결 방안 필요 (예: SOC2/ISO27001 인증)
- 보안 관련 REQ 29건 전면 재작성
- ADR-1, ADR-4 폐기 + 신규 ADR 수립

**장점**: 단일 코드베이스, 빠른 배포  
**단점**: PRD 핵심 차별점 소멸, CISO 관문 해결 방안 필요

### 🎯 Option C: 하이브리드 (가장 현실적)

| 레이어 | 스택 | 배치 |
|:---|:---|:---|
| **프론트엔드 + API** | Next.js + Tailwind + shadcn/ui | 고객 내부 Docker 배포 |
| **데이터베이스** | Prisma + PostgreSQL (로컬 인스턴스) | 고객 서버 내 |
| **AI 엔진** | 로컬 Ollama/vLLM (Gemini 호환 API) | 고객 GPU 서버 |
| **배포** | Docker Compose (Git Push 아님) | USB/내부망 |

→ Next.js + Prisma + shadcn/ui는 그대로 사용하되, **Vercel/Supabase/Gemini API를 로컬 대체제로 교체**

**장점**: 제안 스택의 DX(개발자 경험) 유지 + 온프레미스 보안 유지  
**단점**: 로컬 LLM 성능/비용 관리 필요

---

## 6. SRS 수정 시 변경 범위 예측

### Option A (2-Track) 선택 시

| 수정 대상 | 변경 내용 | 영향 규모 |
|:---|:---|:---:|
| §1.2.3 Constraints | CON-08 추가: "DEMO 모드에서는 클라우드 배포 허용" | 소 |
| §3.1 External Systems | DEMO/PROD 컬럼 분리 | 중 |
| §3.2 Component Diagram | DEMO 아키텍처 다이어그램 추가 | 중 |
| §7 ADR | ADR-8 "Dual-Track Deployment" 추가 | 소 |
| 신규 섹션 | §10 Technology Stack 섹션 신설 | 중 |

### Option B (전면 전환) 선택 시

| 수정 대상 | 변경 내용 | 영향 규모 |
|:---|:---|:---:|
| ADR-1, ADR-4 | 폐기 + 신규 ADR 수립 | 대 |
| CON-01~04 | 전면 재작성 | 대 |
| REQ-FUNC-029~034 | 전면 재작성 (클라우드 보안 모델) | 대 |
| REQ-NF-020~026 | 전면 재작성 | 대 |
| §9 Business Context | CISO 공략 전략 재설계 | 대 |
| **합계** | ~29건 REQ + ADR 2건 + CON 4건 | **대규모** |

### Option C (하이브리드) 선택 시

| 수정 대상 | 변경 내용 | 영향 규모 |
|:---|:---|:---:|
| §3.2 Component Diagram | Next.js + Prisma + Docker 기반으로 재작성 | 중 |
| §3.1 External Systems | EXT-03~05 로컬 AI 대체제로 변경 | 중 |
| §7 ADR | ADR-8 "Next.js Fullstack Monolith" 추가 | 소 |
| 신규 섹션 | §10 Technology Stack 섹션 신설 | 중 |
| REQ-NF-001~006 | 성능 목표 재검증 (로컬 LLM 기준) | 소 |

---

## 7. 최종 판정

| 질문 | 답변 |
|:---|:---|
| **제안 스택으로 기능 구현이 가능한가?** | **비보안 영역 90%+ 가능**, 보안 영역은 근본적 충돌 |
| **SRS를 그대로 충족하는가?** | ❌ **보안/온프레미스 관련 29건 REQ 미충족** |
| **MVP 데모용으로는 적합한가?** | ✅ **UI/비즈니스 로직 검증에 매우 적합** |
| **실제 고객 배포에 적합한가?** | ❌ **CISO 심의 통과 불가** (PRD P5 미해결) |
| **가장 현실적인 접근은?** | **Option A (2-Track)** 또는 **Option C (하이브리드)** |

---

## 부록: 제안 스택 ↔ SRS REQ 전수 매핑

### A. E1 패시브 로깅 (REQ-FUNC-001~008)

| REQ | 현재 구현 | 제안 스택 매핑 | 커버 |
|:---|:---|:---|:---:|
| 001 (STT 트리거 워드) | On-Prem Whisper 파인튜닝 | Gemini API 음성 인식 + 프롬프트 기반 트리거 감지 | ⚠️ |
| 002 (Vision 파싱) | On-Prem Vision 파서 | Gemini API 멀티모달 비전 | ⚠️ |
| 003 (롤백/수정 뷰어) | React 웹앱 | Next.js + shadcn/ui DataTable | ✅ |
| 004 (결측률 리포트) | PostgreSQL 집계 | Prisma 집계 쿼리 + shadcn/ui 차트 | ✅ |
| 005 (비등록 언어 무시) | STT 필터링 | Gemini 프롬프트 기반 필터 | ⚠️ |
| 006 (렌즈 오염 알림) | Vision 파서 에러 핸들링 | Gemini Vision API 에러 응답 처리 | ⚠️ |
| 007 (네트워크 단절 로컬 큐) | 로컬 메시지 큐 | ❌ Vercel Edge에서 로컬 큐 불가 | ❌ |
| 008 (동시 음성 10건+) | STT 큐 순차 처리 | Gemini API Rate Limit 의존 | ⚠️ |

### B. E6 보안 패키지 (REQ-FUNC-029~034)

| REQ | 현재 구현 | 제안 스택 매핑 | 커버 |
|:---|:---|:---|:---:|
| 029 (외부 호출 0건) | 24h 네트워크 캡처 | ❌ Vercel/Gemini API 자체가 외부 호출 | ❌ |
| 030 (USB 오프라인 업데이트) | USB 물리 배포 | ❌ Git Push 배포 | ❌ |
| 031 (ISMS 확인서 자동 생성) | 온프레미스 기준 | ❌ 클라우드 기준 ISMS 재작성 필요 | ❌ |
| 032 (RBAC 5역할 + 감사 로그) | API Gateway | NextAuth.js + Middleware + Prisma 로그 | ✅ |
| 033 (USB 해시 검증) | 해시 검증 모듈 | ❌ USB 배포 자체 불가 | ❌ |
| 034 (미인가 접근 차단) | RBAC 검증 | NextAuth.js Middleware | ✅ |

---

*본 검토 보고서는 SRS V0.3 (08_SRS_V0.3.md)과 제안 기술 스택(C-TEC-001~007)을 교차 대조하여 작성되었습니다.*  
*검토일: 2026-04-18 / 검토자: AI Requirements Engineer*
