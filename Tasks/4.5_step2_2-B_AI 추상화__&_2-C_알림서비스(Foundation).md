# FactoryAI — Foundation (AI 추상화 & 알림 서비스) Issues

> **Source**: SRS-002 Rev 2.0 (V0.8) — §4.1.6, §4.2, §10.1  
> **작성일**: 2026-04-19  
> **총 Issue**: 5건 (AI 3건 + NOTI 2건)  
> **프레임워크**: Next.js App Router, Vercel AI SDK

> [!IMPORTANT]
> 본 문서는 모든 AI 기능(E1, E2 등)의 공통 기반이 되는 **AI 추상화 레이어**와, 전역 알림 처리를 담당하는 **알림 서비스** 구현 명세입니다.

---

## AI-001: Vercel AI SDK + Gemini 프로바이더 초기 설정

---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Foundation/AI] AI-001: Vercel AI SDK + Gemini 프로바이더 초기 설정 (추상화 레이어)"
labels: 'feature, backend, ai, priority:must, epic:foundation'
assignees: ''
---

### :dart: Summary
- **기능명**: [AI-001] Vercel AI SDK + Gemini 프로바이더 초기 설정
- **목적**: Vercel AI SDK를 도입하여 특정 AI 모델(Gemini, OpenAI, Claude 등)에 종속되지 않는 추상화 레이어를 구축한다. 환경변수 전환만으로 모델을 교체할 수 있는 유연성을 확보한다.

### :link: References (Spec & Context)
> :bulb: AI 기획 및 개발 가이드
- SRS 문서: §10.1 기술 스택 정의 (Vercel AI SDK)
- 제약사항: CON-08 (Vercel AI SDK 로 종속성 제거)
- ADR: ADR-9 (AI Gateway 패턴 적용)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** 의존성 패키지 설치 (`npm install ai @ai-sdk/google`)
- [ ] **2.** `.env.local` 및 `.env.cloud`에 필수 환경변수 정의:
  ```bash
  GOOGLE_GENERATIVE_AI_API_KEY=<key>
  AI_PROVIDER=google
  AI_MODEL=models/gemini-1.5-flash
  ```
- [ ] **3.** `lib/ai/provider.ts` — 모델 추상화 팩토리 구현:
  ```typescript
  import { createGoogleGenerativeAI } from '@ai-sdk/google';

  // 환경변수에 따라 동적으로 모델 인스턴스 반환
  export function getAIModel() {
    const provider = process.env.AI_PROVIDER || 'google';
    const model = process.env.AI_MODEL || 'models/gemini-1.5-flash';
    
    if (provider === 'google') {
      const google = createGoogleGenerativeAI({ apiKey: process.env.GOOGLE_GENERATIVE_AI_API_KEY });
      return google(model);
    }
    throw new Error(`Unsupported AI provider: ${provider}`);
  }
  ```
- [ ] **4.** `lib/ai/generate.ts` — 공통 래퍼 함수 (generateText, generateObject 모듈화)
- [ ] **5.** 예외 처리: 인증 에러, 타임아웃, 리소스 부족 모의 등 공통 에러 매핑
- [ ] **6.** 연결 테스트용 단순 엔드포인트 작성 및 삭제

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 동적 모델 객체 생성**
- **Given**: 환경변수 `AI_PROVIDER=google`, `AI_MODEL=models/gemini-1.5-flash`가 설정되어 있다
- **When**: `getAIModel()` 함수를 호출한다
- **Then**: 정상적으로 초기화된 `@ai-sdk/google` 모델 인스턴스가 반환된다.

**Scenario 2: 미지원 프로바이더 감지**
- **Given**: 환경변수 `AI_PROVIDER=openai`가 설정되어 있다 (모듈 미설치 상태)
- **When**: `getAIModel()` 함수를 호출한다
- **Then**: `Unsupported AI provider` 에러를 던져 잘못된 구성을 초기에 차단한다.

### :gear: Technical & Non-Functional Constraints
- **종속성 회피**: 비즈니스 로직(Route Handler)에서 직접 `google(...)` 모델을 import 금지. 항상 `getAIModel()` 래퍼를 통해 가져온다.
- **버전**: Vercel AI SDK 최신 3.x 코어 사용

### :checkered_flag: Definition of Done (DoD)
- [ ] Vercel AI SDK 및 provider 패키지 설치 완료
- [ ] 팩토리 패턴 추상화 함수 구현 완료
- [ ] 단위 테스트(Unit Test) 작성 및 통과
- [ ] Linter 경고 0건

### :construction: Dependencies & Blockers
- **Depends on**: None
- **Blocks**: `AI-002`, `E1-CMD-001`, `E1-CMD-002`, `E2-CMD-002`

---

## AI-002: In-memory Queue + Exponential Backoff

---
name: Feature Task
title: "[Foundation/AI] AI-002: In-memory Queue + Exponential Backoff Rate Limiter 구현 (Gemini 15 RPM 제약 우회)"
labels: 'feature, backend, ai, priority:must, epic:foundation'
---

### :dart: Summary
- **기능명**: [AI-002] In-memory Queue + Exponential Backoff Rate Limiter
- **목적**: MVP 제약인 Gemini Free Tier의 가혹한 Rate Limit(15 RPM)을 어플리케이션 레벨에서 조절하여, 사용자의 "데이터 유실(Error)" 경험을 "일시적 대기(Delayed)" 로 전환한다.

### :link: References (Spec & Context)
- 요구사항: REQ-NF-008 (AI API 15 RPM 제한 극복 방안)
- 제약사항: ASM-10 (MVP 환경은 15 RPM 제한 감수)

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `lib/ai/rate-limiter.ts` 구현 (상태 유지를 위한 In-memory Queue 구조)
  ```typescript
  // 토큰 버킷 또는 단순 인터벌 큐 패턴 사용
  // MVP 모델이므로 단일 인스턴스 기준 로컬 메모리 배열로 충분
  ```
- [ ] **2.** 1분 내 최대 허용 호출 횟수 설정 (여유분 감안하여 12~13 RPM으로 자체 제한)
- [ ] **3.** 호출 대기 로직 (Promise 해결 지연 기법) 구현
- [ ] **4.** 429 Too Many Requests 발생 시 Exponential Backoff (지수 백오프) + 재시도 로직 구현
- [ ] **5.** 최대 대기 시간 초과 시 (예: 60초) 예외 처리 (Dead Letter 또는 로깅)
- [ ] **6.** `lib/ai/generate.ts` 래퍼에 Rate Limiter 연동

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 15개 초과 동시 요청 유입 시 큐 대기**
- **Given**: 시스템에 1초 간격으로 20개의 AI 요청이 들어온다
- **When**: AI Generate 래퍼가 실행된다
- **Then**: 처음 12개 요청은 즉각 처리되고, 나머지 8개 요청은 에러 반환 없이 Queue에서 대기하다가 다음 분기에 순차 처리된다.

**Scenario 2: API 단의 429 에러 발생 시 재시도**
- **Given**: 예상치 못한 이유로 Gemini에서 429 Too Many Requests를 반환한다
- **When**: 래퍼가 에러를 캐치한다
- **Then**: 2초 → 4초 → 8초 순서로 Exponential Backoff 대기 후 자동 재시도하여 최종 응답을 반환한다.

### :gear: Technical & Non-Functional Constraints
- **메모리 한계**: 대기 큐 크기가 비정상적으로 커지지 않도록 (예: 최대 50건) 제한
- **성능**: 응답이 지연됨을 클라이언트가 예측할 수 있도록 타임아웃 튜닝

### :checkered_flag: Definition of Done (DoD)
- [ ] In-memory Queue 및 스로틀링 12 RPM으로 테스트 완료
- [ ] 429 에러 자동 복구 재시도 테스트(Mock) 통과
- [ ] 코드 작성 완료 후 SonarQube 무결점 확인

### :construction: Dependencies & Blockers
- **Depends on**: `AI-001` (추상화 팩토리 설정)
- **Blocks**: `AI-003` (UI 대응), AI를 호출하는 전 비즈니스 로직.

---

## AI-003: '처리 중' 진행률 표시기 UI 컴포넌트

---
name: Feature Task
title: "[Foundation/AI] AI-003: 사용자 대면 '처리 중' 진행률 표시기 UI 컴포넌트 구현"
labels: 'feature, frontend, ui, priority:should, epic:foundation'
---

### :dart: Summary
- **기능명**: [AI-003] 사용자 대면 '처리 중' 진행률 표시기 UI
- **목적**: Rate Limit 큐 대기로 인해 발생하는 최대 60초의 시스템 응답 지연을 사용자가 납득할 수 있게 하는 시각적 컴포넌트를 설계한다. 

### :link: References (Spec & Context)
- 요구사항: REQ-NF-008 UI 대응 과제
- 제약사항: 클라이언트(작업자/관리자)가 장애로 오인하지 않아야 함

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `components/ui/ai-processing-indicator.tsx` 파일 생성
- [ ] **2.** 단순 스피너가 아닌 "진행 상태(추정치)" 또는 "스켈레톤(Skeleton)" 애니메이션 구현
- [ ] **3.** 상황별 안내 문구 노출:
  - 0초~5초: "AI가 데이터를 분석 중입니다..."
  - 5초~15초: "심층 모델이 현장 노이즈를 제거하고 있습니다..."
  - 15초 초과 시: "요청량이 많아 안전하게 순차 처리 중입니다. 최대 1분 정도 소요될 수 있습니다." (에러 오인 방지)
- [ ] **4.** 클라이언트 컴포넌트 상태(useState, useEffect 기반 타이머) 연동

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 지연 시간별 다른 텍스트 노출**
- **Given**: AI 작업을 호출하여 대기 모드에 들어간다
- **When**: 15초가 경과한다
- **Then**: "안전하게 순차 처리 중입니다"라는 텍스트로 자연스럽게 전환된다.

### :gear: Technical & Non-Functional Constraints
- **Tailwind CSS** 기반 디자인 시스템 준수

### :checkered_flag: Definition of Done (DoD)
- [ ] 리액트 컴포넌트 구현 및 Storybook 컴포넌트 등록(옵션)
- [ ] 15초 경과 텍스트 전환 동작 확인
- [ ] 컴포넌트 렌더링 성능 최적화(리렌더링 최소화) 보장

### :construction: Dependencies & Blockers
- **Depends on**: `AI-002` (실제 긴 대기 시간 발생 원인)
- **Blocks**: E1, E2 클라이언트 UI (버튼 클릭 후 로딩 상태 통일)

---

## NOTI-001: [Command] 알림 서비스 DB/비즈니스 코어

---
name: Feature Task
title: "[Foundation/Noti] NOTI-001: [Command] 알림 서비스 DB 저장 및 대상 필터링 — POST /api/v1/notifications"
labels: 'feature, backend, core, priority:must, epic:foundation'
---

### :dart: Summary
- **기능명**: [NOTI-001] 알림 서비스 DB 저장 및 발생(Publish) 코어 로직
- **목적**: 시스템 전역에서 발생하는 권한 거부, 데이터 결측, 결재 대기 등을 통합 관리하고 DB(NOTIFICATION 엔터티)에 적재하여, 슬랙/이메일 발송 없이도 앱 내에서 즉각적 안내가 가능한 알림 기반을 확보한다.

### :link: References (Spec & Context)
- SRS: §6.1 API #19, REQ-NF-029(SLA <= 30초)
- 데이터 모델: `DB-017` NOTIFICATION
- API 계약: `API-019` DTO 정의

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `app/api/v1/notifications/route.ts` 구현 (POST)
- [ ] **2.** 요청 페이로드 검증 (API-019 Zod 스키마 활용)
- [ ] **3.** Auth Middleware를 우회하는 내부 서비스 호출 전용 Secret Key 연동 (또는 시스템 내부 Service 모듈 `lib/notifications.ts`로 구현하여 직접 호출)
- [ ] **4.** 로직 (Service):
  - `target_user_id`가 지정된 경우, 해당 1명에게 알림 저장
  - `target_role`만 지정된 경우, `factory_id`와 매치되는 해당 Role의 모든 User 배열을 가져와 `createMany`를 통해 일괄 DB 적재
- [ ] **5.** 에러 처리 및 트랜잭션 래핑

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: Role 기반 일괄 알림 생성**
- **Given**: 특정 공장(`factory_A`)의 CISO 권한자가 2명 존재한다
- **When**: `target_role=CISO`, `factory_id=factory_A`로 알림 생성을 호출한다
- **Then**: DB에 `target_user_id`가 각각 맵핑된 2건의 알림 데이터가 PENDING(is_read=false) 상태로 적재된다.

### :gear: Technical & Non-Functional Constraints
- **성능 보장**: SLA 30초 이내에 다수(예: 100명)에게 인서트 되더라도 블로킹 하지 않게 비동기 배치(insertMany) 최적화

### :checkered_flag: Definition of Done (DoD)
- [ ] Route Handler(또는 Service Layer 함수) 구현 완료
- [ ] Role-Based fan-out (1:N 분할 생성) 기능 통합 테스트 통과
- [ ] API 명세서 최신화

### :construction: Dependencies & Blockers
- **Depends on**: `API-019` (계약), `DB-017` (테이블), `AUTH-002` (역할 체계)
- **Blocks**: `NOTI-002`, `AUTH-004` (접근 차단 알림)

---

## NOTI-002: [Query] 사용자 알림 목록 조회 + 읽음 처리 UI

---
name: Feature Task
title: "[Foundation/Noti] NOTI-002: [Query] 사용자별 미확인 알림 조회 및 일괄 읽음 처리 UI 연동"
labels: 'feature, frontend, query, priority:should, epic:foundation'
---

### :dart: Summary
- **기능명**: [NOTI-002] 사용자 앱 내 알림 헤더 위젯 및 목록 페이지 구현
- **목적**: 사용자가 자신에게 온 알림(결함 감지, 승인 대기, 보안 경고)을 확인하고 읽음(is_read=true) 처리하는 경험을 제공한다.

### :link: References (Spec & Context)
- SRS: REQ-NF-029~031 등 알림을 시각적으로 확인하는 수단 보장

### :white_check_mark: Task Breakdown (실행 계획)
- [ ] **1.** `GET /api/v1/notifications` 쿼리 API 추가 (미확인 알림만 반환하거나 최신 N개 반환)
- [ ] **2.** `PATCH /api/v1/notifications/{id}/read` 읽음 처리 API 구현
- [ ] **3.** 클라이언트: 헤더 종모양(Bell) 아이콘 컴포넌트 작성 (안 읽은 개수 뱃지 노출)
- [ ] **4.** 알림함 오프캔버스(또는 드롭다운) 팝업: 리스트 노출
- [ ] **5.** 알림의 `severity`(INFO/WARNING/CRITICAL)에 따른 색상 하이라이팅
- [ ] **6.** "모두 읽음" UX 처리

### :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 미확인 알림 뱃지 카운트**
- **Given**: 사용자 A에게 안 읽은 `CRITICAL` 알림 1개, `INFO` 알림 2개가 있다
- **When**: 사용자가 대시보드 메인을 로드한다
- **Then**: 종 모양 아이콘 우측 상단에 붉은색 `3` 뱃지가 렌더링된다.

**Scenario 2: 읽음 상태 전환**
- **Given**: 알림 드롭다운을 연 상태이다
- **When**: 특정 알림 컴포넌트를 클릭한다
- **Then**: 클라이언트 상태 카운트가 `2`로 감소하며, 해당 알림은 즉시 흐린 텍스트 처리되고, 백그라운드에서 PATCH 리퀘스트가 전송된다.

### :gear: Technical & Non-Functional Constraints
- **UX 최적화**: SWR 또는 React Query로 주기적 폴링(재요청) 설정하여 실시간 체감을 제공 (MVP에서는 WebSocket 생략 가능, 10~30초 간격 폴링)
- **오버페칭 방지**: 읽음 처리 시 목록 새로고침 대신 Optimistic Update 기법 권장

### :checkered_flag: Definition of Done (DoD)
- [ ] GET 쿼리 및 PATCH 상태변경 API 구현 완료
- [ ] Notification Bell 레이아웃 및 팝오버 컴포넌트 개발
- [ ] Optimistic UI 로직 작성 및 버그 없음 검증

### :construction: Dependencies & Blockers
- **Depends on**: `NOTI-001` (발행 로직)
- **Blocks**: 전반적인 사용자 대시보드 헤더의 최종 완성
