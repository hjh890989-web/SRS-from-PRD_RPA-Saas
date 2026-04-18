# FactoryAI 기술 제약 해소 및 구현 계획서 (SRS V0.7 수정안)

**목적**: `16_SRS_V06.md` 문서 내 "개인 개발 난이도"와 "무료 인프라 제약 vs. 초기 비기능 요구사항(동시 접속 등)" 간의 심각한 물리적 모순을 해결하기 위해, 합리적으로 타협된 기술 아키텍처 및 요구사항 수정 계획을 수립합니다.

---

## 1. 문제 정의: "개인 개발 + 완전 무료 인프라" vs "기업용 B2B 제조 AI"의 모순

현재 SRS V0.7은 **Next.js + Vercel + Supabase + Gemini API (Free Tier)**라는 제로 베이스 무료 스택으로 MVP를 설계했습니다.
하지만 시스템 비기능 요구사항은 이를 상회합니다.

### 1-1. 심각한 기술적 모순점
1. **무료 인프라 제약 vs 요구 트래픽 (단기 해결 불가)**
   - Vercel Free / Cloudflare Pages Free는 함수 실행 시간 제한(10ms~50ms, 최대 10초)이 매우 엄격합니다. (CON-07)
   - PDF 생성 엔진(react-pdf 등)이나 외부의 느린 ERP 연동을 Vercel Function 위에서 실행할 경우 "Timeout Error(504)"를 피할 수 없습니다. (REQ-NF-003, REQ-FUNC-009)
2. **Gemini Free Tier Rate Limit (월 결제 없이 불가능)**
   - Gemini API 무료 구간 한도는 15 RPM(분당 15회) 수준입니다. (CON-12)
   - REQ-NF-008의 "동시 인입 시 큐 드롭 0건 (STT 5+Vision 3/분)"은 큐잉으로 회피한다 해도, XAI 설명과 함께 호출되면 한도를 즉시 초과하여 HTTP 429(Too Many Requests) 에러를 뿜습니다.
3. **Cloudflare Tunnel + 내부망 연동의 개인 벤쳐 개발 소요**
   - Cloudflare Tunnel (EXT-01) 설정, 보안 증명(0 byte 트래픽, REQ-NF-020), 그리고 더존/영림원 로컬 DB 연결은 로컬망 구성/인프라 설정에만 1인이 2주 이상 쏟아야 할 정도로 난이도가 높습니다.
4. **온프레미스 AI 패키징 난이도 (PROD 모드)**
   - MVP 모드와 Docker 기반의 100% 온프레미스 PROD 환경을 "환경 변수 변경 0줄"로 전환하는 것(CON-08, CON-09)은 1인 개발 범위에서 너무 큰 엔지니어링 비용을 발생시킵니다.

---

## 2. 해결 방향성: "Fake It Till You Make It" (모의 데이터 & 비동기 처리 분리)

개인적으로 주어진 자원 내에서 구현할 수 있도록 **MVP 모드**의 제약을 극단적으로 완화하되, 이것이 PoC로서의 제품 코어 가치를 훼손하지 않게 합니다.

### 2.1 개발 난이도 조정을 위한 기술적 타협점 (Tech Compromises)

#### [방안 1] 무거운 연산(PDF 생성 등) 및 외부 데이터 연동(ERP)의 클라이언트/Mock 위임
- **PDF 생성 (REQ-NF-003)**: Next.js 서버에서 하지 않고, **클라이언트 브라우저 (React 컴포넌트)**에서 렌더링하고 `window.print()` 또는 `@react-pdf/renderer`의 브라우저 렌더러를 사용해 클라이언트 자원을 사용합니다. (서버 타임아웃 해결)
- **ERP 연동 (RE-FUNC-019, EXT-01)**: Cloudflare Tunnel을 통한 진짜 더존 iCUBE 연동은 포기하고, **"Dummy ERP DB(또는 Supabase 안의 가상 ERP 테이블)"**를 미리 구성하여 마치 연동된 것처럼 시연(Mocking)합니다. (실제 현장 투입 전까지 난이도 100% 삭감)

#### [방안 2] Gemini 무료 API 한도를 우회하는 극단적 비동기화 및 캐싱
- **API 호출 병목 차단**: STT, Vision, XAI 요청이 동시 다발적으로 일어나는 UI 동작을 배제합니다.
- 배치 처리 및 큐잉 (REQ-FUNC-008): Next.js API Route 내부 호출 시점 전면에, 백그라운드로 작동하는 단순한 In-memory Queue (또는 브라우저 단의 Delay retry 로직)를 구성하여 "초당 0.2건(Free tier 방어선)" 속도로 천천히 소화시킵니다.

#### [방안 3] MVP에서는 PROD 하이브리드 지원 강박 버리기
- "코드를 수정하지 않고 100% 온프레미스로 전환(CON-01P, CON-08)"이라는 조항을 **MVP 스펙에서 제거**합니다. (Phase 2로 완전 이연)
- 개발자는 현재 100% SaaS 환경(클라우드) 구현에만 집중합니다.

---

## 3. SRS 16_SRS_V06.md 수정 세부 계획 (Plan for Revision)

다음 항목들을 수정하여 `17_SRS_V08.md` 버전을 생성합니다.

### 3.1 제약사항 (Constraints & Assumptions) 대폭 완화
- **CON-01P, CON-08, CON-09 등 완화**: "MVP와 PROD의 코드 분리 금지/무변경 전환" 항목을 삭제하거나 "Phase 2 목표"로 수정. 현재는 "클라우드 전용 아키텍처"로 단일화.
- **CON-07 (서버 사양) 수정**: PDF 생성 모듈 등 무거운 작업은 "Next.js 서버 외 클라이언트 브라우저에서 실행(Client-Side Generation)" 하도록 명시.
- **가정(Assumptions)**: "ERP 연동은 시연을 위해 더존/영림원을 모방한 Mock DB 구조를 Supabase 내에 구축하여 테스트한다."를 추가.

### 3.2 외부 인터페이스 (External Systems) 변경
- **EXT-01 & EXT-02**: 연동 방식을 `Cloudflare Tunnel` → `Mock Database Query (시연용 가상 테이블)`로 강등.
- **EXT-09 모델 업데이트**: PROD 모드 관련 "USB 무설치 업데이트" 등의 요구사항을 Out-of-Scope (Phase 2)로 과감히 제외.

### 3.3 비기능 요구사항 (NFR) 현실화
- **REQ-NF-003**: PDF 생성 지연시간을 30초에서 "클라이언트 리소스 의존, 최대 60초 보장 안 함(Best Effort)"으로 하향.
- **REQ-NF-008**: 큐잉 처리 속도를 명시하여 "Gemini Free Tier Rate Limit(10 RPM)에 맞춰 처리 속도 의도적 지연 허용(사용자에겐 '진행률 표시기' 제공)"으로 변경.
- **동시 접속자 목표 강등 (REQ-NF-007)**: "동시 접속 30명 부하 테스트" 조건 삭제. 오직 "1개 기업 PoC 시연 용도(최대 동시 접속 2인)" 기준으로 NFR을 약화.

### 3.4 기능 요구사항 중 고난이도 항목 타협
- **REQ-FUNC-030 & 031 & 033 등 (PROD 보안/배포 패키지)**: ISMS/망분리 자동 생성기능(REQ-FUNC-031), USB 오프라인 모델 업데이트 등은 1인 개발 범위 초과이므로 "Mock 문서 다운로드" 정도로 강등 또는 Phase 2 이연.

---

## 4. 진행 순서
1. 위 계획에 대해 동의/피드백 확인 (OK/No).
2. 피드백 수령 직후, `16_SRS_V06.md` 문서 파일을 템플릿 삼아 수정 사항을 반영한 **`17_SRS_V08.md`** 문서를 코드 블록 교체/추가를 통해 완성.
3. 문서 완성 후 최종 커밋.
