# ScanOps — 보안 특화 AI 취약점 진단 SaaS

> 부산대학교 2026 AI 해커톤 · 창업트랙(C) 3조 · 팀 **ScanOps**

[![GitHub App](https://img.shields.io/badge/GitHub%20App-설치하기-238636?logo=github)](https://github.com/apps/scanops-security-scanner)

웹 URL 또는 GitHub 코드를 입력하면, **자체 파인튜닝한 1.5B 경량 AI 모델**이 보안 취약점을 자동으로 찾아 CVE·CWE·CVSS 근거와 한국어 수정 가이드를 제공하는 **개발자 친화 보안 진단 SaaS**입니다. 자체 벤치마크 40케이스 기준 **탐지율 100%** 를 달성했습니다.

> **소스코드는 메모리에서만 처리 후 즉시 폐기하고, 결과만 보관(사용자 삭제 전까지·최대 1개월)합니다.** — 코드 프라이버시가 우리의 핵심 약속입니다.

---

## 1. 프로젝트 소개

### 1.1. 개발배경 및 필요성
- 보안 점검은 보통 개발이 끝난 뒤 별도 단계에서 이루어져 발견이 늦고 수정 비용이 큽니다.
- 기존 상용 SAST 도구(예: 스패로우)는 가격이 높고 정적 분석(SAST)에 치우쳐 있으며, 결과가 영어·전문 용어 위주라 비(非)보안 개발자가 곧바로 조치하기 어렵습니다.
- 범용 AI 코딩 에이전트(Claude Code, Codex 등)는 보안에 특화되어 있지 않아 취약점 탐지의 일관성이 떨어집니다.
- "코드를 작성하는 흐름 안에서" 취약점을 짚어주고 **무엇이·왜 문제이며·어떻게 고치는지**까지, 그리고 **고객 코드를 외부에 남기지 않으면서** 알려주는 도구가 필요했습니다.

### 1.2. 개발 목표 및 주요 내용
- 상용 API에 종속되지 않는 **보안 특화 자체 파인튜닝 모델(QLoRA + RAG)** 로 취약점을 진단한다.
- **DAST(웹 URL) / SAST(레포 전체) / GitHub Actions(PR diff)** 세 가지 스캔 방식을 한 플랫폼에서 제공한다.
- 익명 구조에서 **사용자 계정 · 구독 · 사용량 게이트** 기반 SaaS로 전환한다 (모든 스캔이 `user_id`에 귀속).
- **코드 즉시 삭제 정책 + 삭제 증빙 로그**로 프라이버시를 데이터로 증명한다.

### 1.3. 세부내용
**세 가지 스캔 방식**

| 방식 | 입력 | 검사 범위 | 사용량 미터 |
|------|------|-----------|-------------|
| DAST | 웹 URL / 도메인 | 실행 중인 앱 외부 동적 분석 (OWASP ZAP) | 스캔 횟수 |
| SAST | Git 레포 URL | 레포 전체 코드 정적 분석 | 전체 LOC 누적 |
| GitHub Actions | PR 이벤트 | PR diff 파일 전체 (변경 라인엔 위치 댓글, 그 외엔 위치 없이 보고) | 분석 파일 LOC 누적 |


**구독 플랜**

| 플랜 | 가격 | DAST(웹 URL) |  SAST(레포 LOC) & GitHub Actions(LOC) |
|------|------|------|------|
| Free | 0원 | 1회 | ✗ |
| Pro | 19,900원/월 | 월 5회 | 월 10만 줄 |
| Max | 69,000원/월 | 월 30회 | 월 40만 줄 |
| Team | 59,000원/월 | 월 20회 | 월 33만 줄 | 


- **AI 분석 엔진:** NVD CVE 기반 RAG + QLoRA 파인튜닝 LLM이 코드를 입력받아 취약점·CVE·CWE·CVSS·수정 코드를 출력합니다.
- **Adaptive 2-Stage:** ① 파인튜닝 모델 단독 분석 → ② 검증 실패 시 base 모델 + RAG(CVE 컨텍스트) 폴백.
- **AI 분석 라우팅:** `AiAnalyzer` 인터페이스 + `AiRouter` 로 자체 모델(CUSTOM)을 축으로 GPT/Claude/Gemini 폴백 체인을 구성해 가용성·비용을 함께 잡았습니다.

### 1.4. 기존 서비스 대비 차별성
- **코드 프라이버시:** 소스코드는 메모리에서만 처리 후 즉시 폐기, 결과만 보관. 삭제 증빙 로그로 "즉시 폐기" 약속을 데이터로 증명합니다. (무료/Pro URL 스캔은 코드 미전송, GitHub Actions는 고객 인프라 내 스캔 후 결과만 전송하는 **구조적 분리**)
- **SAST + DAST 동시 제공:** 정적 분석에 치우친 기존 상용 도구와 달리 정적·동적 분석을 모두 제공합니다.
- **자체 무료 모델:** 상용 API에 종속되지 않는 파인튜닝 모델(986MB, 자체 서버 무제한 구동)을 보유해 비용·프라이버시 측면에서 자생력이 있습니다.
- **검증된 정확도:** 자체 어댑티브 시스템(QLoRA v4 + RAG)이 **탐지율 100%(40/40), 평균 5.3s** 로 유료 Grok-3 API와 동등 성능을 무료로 달성했습니다.
- **차별화 증명:** 범용 에이전트(Claude Code/Codex)·경쟁사(스패로우, SAST 계열) 대비 같은 입력으로 결과를 나란히 비교하는 벤치마크 데모를 제공합니다.

### 1.5. 사회적가치 도입 계획
- 보안 전문 인력이 부족한 **중소기업·스타트업·1인 개발자**가 저비용으로 PR/배포 단계에서 보안 점검을 받게 합니다.
- 한국어 가이드 제공으로 국내 개발자의 **보안 학습·내재화**를 돕습니다.
- 코드 미보관 원칙으로 진단 과정 자체의 **정보 유출 리스크를 제거**합니다.
- 안전한 코드 문화를 확산해 개인정보 유출·서비스 침해 등 사회적 피해를 예방합니다.

---

## 2. 상세설계

### 2.1. 시스템 구성도
```
사용자 (로그인 · 소유권 인증 · 사용량 체크)
        │  웹 URL / GitHub 레포 / PR
        ▼
 scanops-frontend (대시보드 · Vercel)
        │
        ▼
 scanops-backend (Spring Boot · Railway)
 - 인증/구독/사용량 게이트 (AiRouter)
 - 스캔 오케스트레이션
        │  HTTP POST /analyze(/batch)
        ▼
 분석 코어 (AWS)
 ├─ FastAPI (api_server.py · v4.0.0)
 ├─ Ollama  (qwen2.5-coder-security-v4 · 986MB)
 └─ Qdrant  (CVE 12,251건 벡터 검색)
        │
        ▼
 OWASP ZAP (DAST 동적 스캔 · scanops-infra)
        │
        ▼
 PostgreSQL (스캔·결과·사용량·구독 / 코드 원문 미저장)
```

### 2.2. 사용 기술
| 스택 | 기술 / 버전 | 배포 |
|------|-------------|------|
| Frontend | React 18, TypeScript 5, Vite 6, Tailwind CSS v4, React Router v6, Recharts, FSD 아키텍처 | Vercel |
| Backend | Spring Boot 3.2.5, Java 17, Spring Data JPA, Spring Security(JWT/OAuth), WebClient | Railway |
| AI 분석 API | FastAPI (Python), Ollama 서빙 | AWS |
| AI Model | QLoRA v4 파인튜닝 (Qwen2.5-Coder-1.5B-Instruct), GGUF Q4_K_M(986MB) | HuggingFace Hub → AWS |
| RAG | BAAI/bge-small-en-v1.5 임베딩(384차원) + Qdrant (CVE 12,251건) | AWS |
| Security Engine | OWASP ZAP (`ghcr.io/zaproxy/zaproxy:stable`) | AWS / 로컬 |
| Database | PostgreSQL 15 | Railway |
| 결제 | 토스페이먼츠 / 스트라이프 (예정) | - |
| Infra | Docker Compose (로컬: ZAP + DVWA + PostgreSQL), 하이브리드 배포(분석 코어 AWS / BE·FE Railway) | - |

**활용한 생성형 AI / AI 코딩 도구**
- **Claude Code (Anthropic)** — 멀티 레포 아키텍처 정리, 백엔드·모델 코드 작성, 벤치마크·리팩토링, GitHub Actions 미러링 자동화에 활용.
- **자체 파인튜닝 LLM (핵심)** — Qwen2.5-Coder-1.5B를 QLoRA로 보안 특화 파인튜닝(v4, scratch 재훈련, 학습 데이터 1,000개 / CWE 35종) + NVD CVE RAG.
- **OpenAI GPT / Anthropic Claude / Google Gemini API** — 런타임 AI 분석 폴백 체인(`AiRouter`)의 구성 요소.

---

## 3. 개발결과

### 3.1. 전체시스템 흐름도
```
진입(회원 분기)
  ├─ 미회원 → 회원가입(이메일/GitHub OAuth) → 이메일 인증 → 로그인
  └─ 회원   → 로그인
        │
        ▼
   랜딩 (차별점 · 가격 · 약관/개인정보)
     ├─ 구독/결제
     └─ MyPage (개인정보 · 스캔기록+삭제 · 사용량 미터 · 구독상태)
            │
            ▼
   스캔  ① 로그인  ② 레포/도메인 소유권 인증  ③ 플랜별 사용량 한도 통과
            │
            ▼
   분석  FastAPI → Ollama(Qwen v4) → 검증 실패 시 Qdrant RAG 폴백
            │
            ▼
   결과  취약점 + CVE/CWE + CVSS + 신뢰도 → 대시보드 / PDF / AI 브리핑
        (소스코드는 폐기, 결과만 저장 · 삭제 증빙 로그 기록)
```

### 3.2. 기능설명
- **인증/계정:** 이메일·GitHub OAuth 가입/로그인, 이메일 인증, 약관 동의 기록, JWT 세션, 비밀번호 재설정.
- **MyPage:** 프로필·구독 등급, 이번 달 사용량 미터(DAST 횟수 / Actions·SAST LOC 잔여), 스캔 기록 조회·재조회(보관 1개월), 개별·전체 결과 삭제, 회원 탈퇴.
- **소유권 인증:** DAST 대상 도메인(DNS TXT)·SAST/Actions 대상 레포(repo 파일) 소유권을 검증한 뒤에만 스캔, 검증된 타겟은 재사용.
- **스캔 게이트:** 스캔 전 ① 로그인 ② 소유권 인증 ③ 플랜별 사용량 한도를 통과시킵니다.
- **결과 출력:** 취약점별 CVSS·신뢰도·위치 상세 뷰, CVSS 7.0 이상 우선 필터링, 공유용 PDF, Claude/GPT에 붙여 수정코드를 받는 AI 보안 브리핑(Pro+).
- **GitHub App PR 분석:** [GitHub App](https://github.com/apps/scanops-security-scanner) 설치 후 PR을 올리면 자동 분석, 취약점이 발견되면 해당 코드 라인에 한국어 코멘트가 달립니다.

> 📹 시연 영상: `섹션 5` 참고

### 3.3. 기능명세서
> 기능명세서(Notion): https://lavish-carpet-8f6.notion.site/ScanOps-3730070cb76480ebacf6dc6ee917f99d?pvs=74

### 3.4. 디렉토리 구조
본 제출 레포지토리는 4개 서비스 레포를 동일 이름의 서브폴더로 미러링한 모노 구조입니다. 각 레포는 `main` 브랜치 push 시 GitHub Actions로 자동 동기화됩니다.

```
pnuai-c-03-scanops/
├── scanops-frontend/   React + TS + Vite + Tailwind, FSD 아키텍처 (대시보드 UI)
├── scanops-backend/    Spring Boot 3.2.5 (인증·구독·게이트·스캔 오케스트레이션·AiRouter)
├── scanops-model/      QLoRA v4 + RAG 보안 분석 LLM (FastAPI · Ollama · Qdrant)
├── scanops-infra/      Docker Compose (ZAP + DVWA + PostgreSQL)
└── README.md
```

| 서비스 | 설명 |
|--------|------|
| `scanops-frontend` | 대시보드 UI |
| `scanops-backend` | Spring Boot 백엔드 (인증·구독·게이트) |
| `scanops-model` | AI 분석 서버 (QLoRA v4 + RAG, FastAPI) |
| `scanops-infra` | ZAP + 인프라 구성 |

### 3.5. AI 도구 활용 및 모델 성능
**AI 도구 활용**
- **설계:** Claude Code로 멀티 레포 아키텍처(레포 분리, 의존성 정리, 해커톤 미러링 워크플로, SaaS 전환 설계)를 정리했습니다.
- **개발:** 백엔드 `AiRouter` 폴백 구조, 모델 RAG 파이프라인, 프론트엔드 FSD 구조 코드를 AI 페어 프로그래밍으로 작성했습니다.
- **모델:** QLoRA v4 파인튜닝 데이터 생성·scratch 재훈련·GGUF 양자화·벤치마크 자동화 전 과정을 AI로 가속했습니다.

**모델 (Qwen2.5-Coder-1.5B QLoRA v4)**
- 베이스: Qwen2.5-Coder-1.5B-Instruct (양자화 후 986MB)
- 파인튜닝: QLoRA r=32 / alpha=64, 어텐션+MLP 7개 레이어 타겟, 학습 가능 파라미터 36.9M(2.34%)
- 학습 데이터 1,000개(CWE 35종, CWE Top-25 전수 포함), scratch 재훈련(Catastrophic Forgetting 해결), 최종 손실 0.2897
- v4 신규: 응답에 **CVSS 점수** 포함

**벤치마크 (40케이스)**

| 방법 | 탐지율 | Stage1 비율 | 속도 | 비용 |
|------|--------|-------------|------|------|
| ScanOps v2 | 95% | 75% | 2.7s | 무료 |
| ScanOps v4 (현재) | **100%** | **100%** | 5.3s | 무료 |
| Grok-3 API (비교) | 95% | - | 17.7s | 유료 |
| Grok-3 + RAG (비교) | 100% | - | 5.5s | 유료 |

→ Stage1 100% = RAG 폴백 없이 파인튜닝 모델 단독으로 40케이스 전원 탐지.

---

## 4. 설치 및 사용 방법

### 가장 쉬운 방법 — 웹 / GitHub App
1. 웹에서 회원가입·로그인 후 대상 URL·레포를 등록하고 소유권 인증 → 스캔 실행, 또는
2. [ScanOps GitHub App](https://github.com/apps/scanops-security-scanner) 설치 → 레포 선택 → PR 올리면 자동 분석.

### 로컬 실행 (개발용)
각 서비스 레포의 `README.md`에 상세 가이드가 있습니다.

```bash
# 1) 인프라 (ZAP + DVWA + PostgreSQL)
cd scanops-infra && docker-compose up -d

# 2) 모델/분석 서버 (FastAPI + Ollama + Qdrant)
cd scanops-model && pip install -e .
docker-compose up -d            # Qdrant
scanops scan <파일/디렉토리>     # CLI 분석

# 3) 백엔드 (Spring Boot)
cd scanops-backend && ./gradlew bootRun

# 4) 프론트엔드 (React)
cd scanops-frontend && npm install && npm run dev
```

---

## 5. 소개 및 시연 영상
> 프로젝트 소개 영상을 교육원 메일(swedu@pusan.ac.kr)로 제출 후 부여받은 YouTube URL을 여기에 추가하세요. — _TODO_

---

## 6. 팀 소개
> 창업트랙(C) 3조 · 팀 **ScanOps**

| 이름 | 역할 | 연락처 |
|------|------|--------|
| 김세한 | 팀장 | 010-7722-3694 |
| 전혜은 | 개발 | 010-9155-8528 |
| 이경윤 | 개발 | 010-2012-9376 |
| 최효석 | UX/마케팅 | 010-8974-3098 |

---

## 7. 해커톤 참여 후기
> 팀원별 참여 후기를 작성하세요. — _TODO_
