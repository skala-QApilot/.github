# QApilot
 
**코드와 문서를 읽고, 테스트를 직접 설계·실행·분석까지 끝내는 All-in-One QA 자동화 플랫폼**
 
---
 
## 프로젝트 배경
 
QApilot은 **코드베이스와 요구사항 문서(PRD/정책/약관 등)를 기반으로 E2E 통합 테스트 시나리오를 자동 생성하고, UI·API·DB를 교차 검증하며, 장애가 발생하면 원인 분석과 해결 방안까지 제시**하는 Agentic AI 기반 QA 자동화 시스템입니다.
 
- 요구사항 정의서 기반 Test Scenario 생성
- 코드베이스, 설계산출물(PRD/정책/약관/api 명세서 등) 기반 TestCase, TestValue 생성
- 자연어로 테스트 시나리오 생성/수정 요청 가능
- UI / API / DB 3-Tier 교차 검증으로 단순 화면 테스트를 넘어선 통합 테스트 수행
- 장애 발생 시 원인 후보와 코드 수정 제안까지 자동 제공
- 요구사항-테스트 추적 매트릭스(RTM)로 요구사항 커버리지 관리
---
 
## 문제 정의 & 목표
 
실제 프로젝트의 통합/인수 테스트는 평균 **3개월**이 소요되며, 다음과 같은 병목이 반복적으로 발생합니다.
 
- 환경 세팅, 데이터 준비, 시나리오 작성, 결함 분석, 재검증, 결과 정리 등 반복 작업
- 코드 한 줄만 수정해도 수십 개의 테스트가 함께 깨짐
- UI·API·DB를 통합적으로 검증할 수 있는 도구의 부재
- 요구사항이 실제로 충족되었는지 추적이 불투명함
QApilot은 코드와 문서를 읽어 **테스트 시나리오를 자동 생성**하고, **UI·API·DB를 교차 검증**하며, **장애 원인 분석과 RTM 기반 요구사항 추적**까지 테스트의 전 과정을 책임지는 시스템을 목표로 합니다.
 
---
 
## ✨ 핵심 기능 / 차별점
 
| 영역 | 설명 |
|---|---|
| 시나리오 자동 생성 | 코드 정적 분석 + PRD/정책/약관 문서를 RAG로 결합해 TS(시나리오)·TC(테스트케이스)·TV(테스트값) 3계층 시나리오를 자동 생성 |
| 자연어 기반 시나리오 제어 | 자연어 요청으로 테스트 시나리오를 새로 생성하거나 기존 시나리오를 수정 |
| 변경 감지 기반 재생성 | Git diff 기반으로 변경된 코드 영향 범위를 분석해 관련 시나리오를 재생성 |
| UI·API·DB 통합 검증 | Playwright로 UI를 실행하면서 동시에 API 호출과 DB 변화를 추적해 3-Tier 교차 검증(Cross-check) 수행 |
| 장애 분류·원인 추론·해결 제안 | 교차 검증에서 불일치가 발견되면 결함 유형(UI/API/DATA/ENV 등)을 분류하고, 코드 위치·근거와 함께 원인 후보(Top-N) 및 수정 코드까지 제안 |
| RTM (요구사항 추적 매트릭스) | 요구사항 ↔ 테스트케이스 매핑을 통해 요구사항 변경의 영향 범위와 커버리지를 추적 |
| HITL (Human-in-the-Loop) | 신뢰도 점수 기반으로 사람의 검토/승인이 필요한 지점에서 HITL 큐를 통해 의사결정 요청 |
| 리포트 & 대시보드 | 테스트 실행 이력, 증적(스크린샷/API 로그/DB 스냅샷), 분석 리포트를 웹 대시보드에서 확인 |
 
**TC 3,000개 기준, 수동 테스트 대비 약 97%의 테스트 공수 절감**을 목표/검증했습니다.
 
---
 
## 시스템 아키텍처
 QApilot은 **웹 대시보드 → 중앙 API 서버(SpringBoot) → Agentic Pipeline(Layer 1~3) → 데이터 스토리지** 로 이어지는 구조이며, 외부의 **테스트 대상 시스템(SUT)** 을 직접 실행·검증합니다.
 
<img width="5944" height="4576" alt="image" src="https://github.com/user-attachments/assets/1a20a6d3-3e40-46eb-99af-af64d441248b" />

 
### Agentic Pipeline (LangGraph 기반, 3 Layer · 8 Agent + 6 Tool)
 
**Layer 0. 전처리**
- **자연어 해석 Agent**: 사용자가 입력한 시나리오 요구사항을 해석해 구조화된 데이터로 변환
- **요구사항 추출 Agent**: PRD 등 설계 산출물을 기반으로 요구사항 리스트 추출
- **코드베이스 스캔 Tool**: AST/IR 파싱을 통해 코드의 종속성 그래프를 생성
**Layer 1. 시나리오 생성**
- **시나리오 생성 Agent**: 전처리 결과를 바탕으로 테스트 시나리오 및 테스트 데이터 생성
  - 도메인 지식 Tool: PRD/정책서를 RAG로 검색해 프롬프트에 주입
  - 요구사항 매핑 Tool: 생성된 시나리오를 요구사항과 매핑(RTM)
- **시나리오-액션 매핑 Agent**: 구조화된 시나리오를 실행 가능한 Step 시퀀스로 변환
- **Playwright 코드 생성 Agent**: UI 액션-API 매핑을 바탕으로 Playwright 테스트 코드 작성
- 생성된 시나리오는 중앙 API 서버를 통해 **HITL(시나리오 검증)** 승인을 거칠 수 있음
**Layer 2. 테스트 실행 및 교차 검증**
- **테스트 Tool**
  - UI 테스트 Tool: Playwright로 UI 실행
  - API 추적 Tool: Playwright 네트워크 리스너로 API 호출 추적
  - DB 테스트 Tool: DB 상태 변화 검증
- **Cross-check Agent**: UI 상태 ↔ API 응답 ↔ DB 상태를 교차 검증
**Layer 3. 장애 분석 및 리포트**
- **원인 추론 Agent**: 원인 후보 도출 및 근거 제시
- **해결 방안 추천 Agent**: 수정 위치 및 방법 제안
- **리포트 생성 모듈**: csv/pdf 형태의 리포트 생성
  
---
### 핵심 시나리오 (S1~S4)
 
- **S1. 설계 산출물 & 코드베이스 기반 테스트 시나리오 및 케이스 자동 생성**
  PRD/정책서와 코드베이스를 함께 분석해 TS/TC/TV 형태의 테스트 시나리오·케이스를 자동 생성
- **S2. UI-API-DB까지 통합 테스트를 자동으로 수행**
  Playwright 기반으로 UI를 조작하면서 동시에 API/DB 상태를 추적해 3-Tier 교차 검증 수행
- **S3. 장애 원인 분석 및 해결 방안 제시**
  Cross-check에서 불일치가 발견되면 원인 후보와 수정 위치·방법을 자동 제안
- **S4. 시나리오-요구사항 매핑 및 요구사항 추적 자동화**
  생성된 시나리오를 요구사항과 매핑해 RTM을 자동으로 구성·갱신
모든 Agent/Tool은 공통 하네스(`BaseAgent` / `BaseTool`) 위에서 동작하며, 로깅·재시도·가드레일·타임아웃·신뢰도 점수가 자동으로 적용됩니다.
 
- 프롬프트 인젝션 / 민감정보 감지 가드레일
- 지수 백오프(Exponential Backoff) 재시도
- asyncio 기반 타임아웃 제어
- trace_id 기반 구조화 로깅
- OpenAI 클라이언트 래퍼 (토큰 카운팅 · 캐시)
- Pydantic 기반 입출력 스키마 검증
  
---
 
## 기술 스택
 
| 영역 | 기술 |
|---|---|
| Agent Framework | LangGraph, OpenAI API (gpt-5.4-mini) |
| AI 서버 | FastAPI |
| 중앙 서버 | Spring Boot |
| 프론트엔드 | Vue 3, Tailwind CSS |
| CLI | Python, Typer |
| 테스트 실행 | Playwright (Python) |
| 코드 분석 | tree-sitter (다중 언어 AST 파싱) |
| 벡터 DB | Qdrant |
| RDB | SQLite (로컬) / PostgreSQL (서버) |
| 배포/인프라 | Docker, Kubernetes |
 
---
 
### 기본 포트 구성
 
| 구성 요소 | 포트 |
|---|---|
| SUT (Frontend / Backend) | 3000 / 8000 |
| QApilot 중앙 서버 (Spring / FastAPI) | 8080 / 8001 |
| Qdrant (Vector DB) | 6333 |
| PostgreSQL | 5432 |
 
---
 
## 📂 Repositories
 
| Repository | 설명 |
|---|---|
| [`system-under-test`](https://github.com/skala-QApilot/system-under-test) | QApilot의 기능 검증/시연을 위한 테스트 대상 시스템(SUT, 통신사 셀프케어 컨셉 서비스) |
 
---
 
## 🏆 성과
 
- TC 3,000개 기준 **수동 테스트 대비 약 97% 테스트 공수 절감** 효과 도출
---
 
## 👥 팀
 
SKALA 3기 AI 1반 3팀 (6인) · SKT NOVA·ICT서비스팀 발제 과제로 진행되었습니다.
 
