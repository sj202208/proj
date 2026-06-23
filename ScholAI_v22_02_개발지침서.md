# ScholAI — 개발 지침서 v2.2

> **서비스명**: ScholAI | **팀명**: APT | **버전**: v2.2
> **v2.2 보완**: scholar_service LiteratureSource enum 적용 · KCI import enum 적용 · constants import 누락 해소
> **v2.1 보완**: 국회도서관 API fallback 로직 · KCI CSV 컬럼 자동탐지 · source enum 정의
> **v2 주요 변경**: 전문가 패널 4인 · Google Scholar + 국회도서관 + KCI CSV · 통계 R/Python(Tableau 제거)

---

# 목차

1. [기술 스택](#1-기술-스택)
2. [프로젝트 폴더 구조](#2-프로젝트-폴더-구조)
3. [환경 설정](#3-환경-설정)
4. [화면별 개발 명세](#4-화면별-개발-명세)
5. [AI 전문가 패널 시스템 (4인)](#5-ai-전문가-패널-시스템-4인)
6. [외부 API 연동 가이드](#6-외부-api-연동-가이드)
7. [KCI CSV 데이터 처리](#7-kci-csv-데이터-처리)
8. [통계 프로그램 연동 (R/Python)](#8-통계-프로그램-연동-rpython)
9. [백엔드 API 설계](#9-백엔드-api-설계)
10. [데이터베이스 스키마](#10-데이터베이스-스키마)
11. [인증 및 보안](#11-인증-및-보안)
12. [스프린트 계획](#12-스프린트-계획)
13. [코딩 컨벤션](#13-코딩-컨벤션)
14. [환경변수 목록](#14-환경변수-목록)
15. [배포 구성](#15-배포-구성)
16. [테스트 전략](#16-테스트-전략)

---

# 1. 기술 스택

## 프론트엔드

| 항목 | 선택 | 버전 | 이유 |
|------|------|------|------|
| 프레임워크 | Next.js (App Router) | 14.x | SSR + SEO + 파일 기반 라우팅 |
| 언어 | TypeScript | 5.x | 타입 안전성, 협업 적합 |
| 스타일링 | Tailwind CSS | 3.x | 빠른 UI 구현, 일관된 디자인 |
| 상태관리 | Zustand | 4.x | 가볍고 직관적인 전역 상태 |
| 서버 상태 | TanStack Query | 5.x | API 캐싱, 로딩/에러 자동 처리 |
| 폼 관리 | React Hook Form + Zod | 최신 | 유효성 검사 + 폼 상태 |
| 다국어 | next-intl | 3.x | 한국어/영어 전환 |
| UI 컴포넌트 | shadcn/ui | 최신 | 접근성 고려 기본 컴포넌트 |
| Split-View | react-resizable-panels | 최신 | 좌우 패널 크기 조절 |
| 코드 에디터 | Monaco Editor | 최신 | R/Python 코드 편집기 |
| PDF 뷰어 | react-pdf | 최신 | 논문 업로드/뷰어 |

## 백엔드

| 항목 | 선택 | 버전 | 이유 |
|------|------|------|------|
| 프레임워크 | FastAPI (Python) | 0.110+ | 비동기 지원, AI 라이브러리 연동 |
| 언어 | Python | 3.11+ | AI/ML, 데이터 처리 생태계 |
| 인증 | NextAuth.js + JWT | 최신 | 소셜 로그인 + 토큰 인증 |
| 데이터베이스 | PostgreSQL | 16.x | 관계형 + JSONB + 전문검색 |
| ORM | SQLAlchemy 2.0 + Alembic | 최신 | 마이그레이션 관리 |
| 캐싱 | Redis | 7.x | API 응답 캐시, 세션 관리 |
| 작업 큐 | Celery + Redis | 최신 | AI 패널 협의 비동기 처리 |
| CSV 처리 | pandas | 최신 | KCI CSV 파싱 및 DB 임포트 |
| 파일 저장 | AWS S3 | - | 연구모형 이미지 / 논문 파일 |

## AI / 연동

| 항목 | 선택 | 용도 |
|------|------|------|
| LLM | Claude API (claude-3-5-sonnet) | 상담·패널·심사 AI 엔진 |
| 번역 | DeepL API | 영문 논문 번역 |
| Google Scholar | SerpAPI | 해외 논문 검색 |
| 국회도서관 | Open API | 국내 논문 검색 |
| KCI 논문 | CSV → PostgreSQL | 로컬 KCI 논문 DB |
| 통계 R | WebR (WebAssembly) | 브라우저 내 R 실행 |
| 통계 Python | Pyodide | 브라우저 내 Python 실행 |

## 인프라

| 항목 | 선택 | 비고 |
|------|------|------|
| 프론트 호스팅 | Vercel | Next.js 최적화 |
| 백엔드 호스팅 | AWS EC2 (t3.medium 이상) | FastAPI 서버 |
| 컨테이너 | Docker + Docker Compose | 환경 통일 |
| CI/CD | GitHub Actions | 자동 테스트 + 배포 |
| 모니터링 | Sentry + Datadog | 에러 추적 + 성능 |

---

# 2. 프로젝트 폴더 구조

## 프론트엔드 (Next.js)

```
scholai-frontend/
├── app/
│   ├── (auth)/
│   │   └── login/page.tsx                  # 화면1: 간편 로그인 (소셜만, register 없음)
│   ├── (main)/
│   │   ├── layout.tsx                      # 인증 후 공통 레이아웃
│   │   ├── dashboard/page.tsx              # 메인 대시보드
│   │   ├── research-idea/
│   │   │   └── page.tsx                    # 화면2: 내 생각이 연구로 (rq-ai 연동)
│   │   ├── journal-guide/
│   │   │   └── page.tsx                    # 화면3: 학술지 작성 가이드
│   │   ├── literature/
│   │   │   ├── search/page.tsx             # 화면4-A: 선행연구 검색
│   │   │   └── models/page.tsx             # 화면4-B: 연구모형 갤러리
│   │   ├── split-view/
│   │   │   └── page.tsx                    # 화면5: 해외 학술지 Split-View
│   │   ├── statistics/
│   │   │   ├── recommend/page.tsx          # 화면6: 통계분석 추천
│   │   │   └── execute/
│   │   │       ├── page.tsx                # 화면7: 통계 직접 실행
│   │   │       └── components/
│   │   │           ├── RConsole.tsx        # WebR R 콘솔
│   │   │           └── PyConsole.tsx       # Pyodide Python 콘솔
│   │   ├── expert-panel/
│   │   │   └── page.tsx                    # AI 전문가 패널 협의
│   │   └── review-coaching/
│   │       ├── upload/page.tsx             # 화면8-A: 논문 업로드
│   │       ├── feedback/page.tsx           # 화면8-B: 4인 피드백
│   │       └── mock-defense/page.tsx       # 화면8-C: 가상 심사 실습
├── components/
│   ├── ui/                                 # shadcn/ui 기본 컴포넌트
│   ├── common/
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   └── LoadingSpinner.tsx
│   ├── chat/
│   │   ├── ChatInterface.tsx
│   │   ├── MessageBubble.tsx
│   │   └── ExpertPanelChat.tsx
│   ├── literature/
│   │   ├── PaperCard.tsx                   # 피인용 횟수·KCI/SCI 배지 포함
│   │   ├── PaperFilters.tsx
│   │   ├── KciBadge.tsx                    # KCI 등재 배지 컴포넌트
│   │   └── ModelImageGrid.tsx              # 연구모형 4~8개 그리드
│   ├── split-view/
│   │   ├── SplitViewLayout.tsx
│   │   ├── OriginalPanel.tsx
│   │   ├── TranslationPanel.tsx
│   │   └── SummaryPanel.tsx
│   └── statistics/
│       ├── StatMethodCard.tsx
│       ├── ResultInterpreter.tsx           # 통계 결과 자동 해석
│       └── GuideOverlay.tsx                # 초보자 가이드 오버레이
├── lib/
│   ├── api/
│   │   ├── client.ts
│   │   ├── literature.ts                   # Scholar + 국회도서관 + KCI
│   │   ├── translate.ts
│   │   ├── panel.ts
│   │   └── statistics.ts
│   ├── hooks/
│   │   ├── useChat.ts
│   │   ├── useLiterature.ts
│   │   ├── useSplitView.ts
│   │   └── useStatistics.ts
│   ├── stores/
│   │   ├── userStore.ts
│   │   └── researchStore.ts
│   └── utils/
│       ├── formatters.ts
│       └── validators.ts
├── types/
│   ├── user.ts
│   ├── research.ts
│   ├── literature.ts                       # KciPaper, ScholarPaper 타입 포함
│   └── statistics.ts
└── public/
    └── locales/
        ├── ko.json
        └── en.json
```

## 백엔드 (FastAPI)

```
scholai-backend/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── api/v1/
│   │   ├── auth.py
│   │   ├── literature.py                   # Scholar + 국회도서관 + KCI 통합
│   │   ├── translate.py
│   │   ├── expert_panel.py
│   │   ├── statistics.py
│   │   ├── models_gallery.py
│   │   └── review_coaching.py
│   ├── core/
│   │   ├── security.py
│   │   └── dependencies.py
│   ├── services/
│   │   ├── claude_service.py
│   │   ├── scholar_service.py              # Google Scholar (SerpAPI)
│   │   ├── nanet_service.py                # 국회도서관 API
│   │   ├── kci_service.py                  # KCI CSV → DB 검색
│   │   ├── deepl_service.py
│   │   ├── panel_service.py                # 4인 패널 협의
│   │   └── statistics_service.py
│   ├── scripts/
│   │   └── import_kci_csv.py              # KCI CSV DB 임포트 스크립트
│   ├── models/
│   └── schemas/
├── data/
│   └── 한국연구재단_KCI논문정보_20250825.csv  # KCI 원본 데이터
├── alembic/
├── tests/
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

---

# 3. 환경 설정

```bash
# 1. 저장소 클론
git clone https://github.com/apt-team/scholai.git
cd scholai

# 2. 프론트엔드
cd scholai-frontend
npm install
cp .env.example .env.local
npm run dev        # http://localhost:3000

# 3. 백엔드
cd ../scholai-backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env

# 4. KCI CSV 데이터 DB 임포트 (최초 1회)
python app/scripts/import_kci_csv.py --file data/한국연구재단_KCI논문정보_20250825.csv

# 5. 서버 실행
uvicorn app.main:app --reload  # http://localhost:8000

# 6. 도커 전체 실행 (DB + Redis 포함)
docker-compose up -d
```

---

# 4. 화면별 개발 명세

## 화면 1 — 간편 로그인

```
목적: 최소한의 절차로 빠른 접속

구성 요소:
  - Google OAuth 버튼
  - Kakao OAuth 버튼
  - 서비스 소개 한 줄 문구

동작:
  - NextAuth.js 처리 → JWT 발급
  - 로그인 성공 → /dashboard 이동
  - 첫 로그인: 소셜 최초 인증 시 users 테이블 자동 생성
  - 별도 register 페이지 없음

유의사항:
  - 이메일 로그인 미제공 (소셜 로그인 전용)
```

## 화면 2 — 내 생각이 연구로 이어질 수 있을까?

```
목적: 막연한 아이디어 → 연구 가능한 주제로 구체화

구성 요소:
  - 상단: 연구 아이디어 자유 텍스트 입력창
  - 중앙: AI 상담 챗봇 (Claude API 스트리밍)
  - 우측 패널: rq-ai iFrame 임베드
  - 하단: "이 주제로 선행연구 찾기" 버튼 → 화면4 이동

rq-ai 연동:
  - URL: https://rq-ai-620920432884.asia-east1.run.app/
  - iFrame embed + postMessage로 키워드 수신
  - 수신 키워드 → researchStore에 저장 → 화면4 자동 반영

AI 프롬프트:
  - 역할: "연구 주제 구체화를 돕는 친절한 연구 멘토"
  - 연구 가능성 평가 + 구체화 질문 + 유사 선행연구 힌트 제공
  - Claude API streaming 사용 (실시간 타이핑 효과)
```

## 화면 3 — 학술지 무엇이 필요할까?

```
목적: 학술지 구성요소를 단계별로 이해하고 작성 방향 수립

구성 요소:
  - 학술지 5단계 구조 탭/아코디언
    ① 서론: 연구 배경, 목적, 연구 문제
    ② 이론적 배경: 선행연구, 연구모형
    ③ 연구 방법론: 대상, 도구, 절차
    ④ 결과 및 분석
    ⑤ 결론 및 시사점
  - 각 섹션별 AI 멘토 질문/답변 패널
  - 단계별 작성 체크리스트 (로컬 저장)
  - 진행률 표시바

4인 전문가 멘토 역할:
  - 논문 다작 박사: 구조적 조언
  - 논문 심사위원: 심사 관점 예고
  - 논문 지도 교수: 방향성 제시
  - 데이터 분석가: 방법론/분석 조언
```

## 화면 4 — 나의 연구주제와 선행연구 연계

```
목적: 선행연구 검색 + 연구모형 갤러리

[선행연구 검색]
구성 요소:
  - 키워드 태그 입력창 (복수 입력, 엔터로 추가)
  - 검색 소스 선택: Google Scholar / 국회도서관 / KCI DB (복수 선택)
  - 필터: 연도 범위 / 논문 유형 / 언어 / KCI·SCI 등급
  - 결과 카드:
      - 제목 / 저자 / 발행연도
      - 피인용 횟수 (숫자 + 막대 시각화)
      - KCI 등재 배지 (보라색)
      - SCI·SCIE·SSCI 배지 (금색)
      - AI 초록 요약 (200자)
      - 원문 링크 / 저장 버튼
  - 정렬: 피인용 횟수 / 최신순 / 관련도순

검색 처리 순서:
  1. KCI CSV DB 검색 (로컬, 가장 빠름)
  2. 국회도서관 API 검색
  3. Google Scholar 검색 (SerpAPI)
  4. 결과 병합 + 중복 제거 (DOI 또는 제목 기준)
  5. Claude API로 초록 AI 요약 생성
  6. 결과 Redis 1시간 캐싱

[연구모형 갤러리]
구성 요소:
  - CSS Grid 4열 레이아웃 (4~8개 이미지)
  - 각 카드: 연구모형 이미지 + 출처 논문 + 연도
  - 클릭 → 모달 확대
  - 다운로드: 개별 / 전체 ZIP
```

## 화면 5 — 해외 학술지를 한눈에 (Split-View)

```
목적: 영문 논문 원본 + 번역본 + 요약 동시 제공

레이아웃 (PC):
  ┌──────────────┬──────────────┐
  │  원문 (영어)  │ 번역 (한국어) │  ← 50:50 기본, 드래그 조절
  │  (스크롤 동기화)│(스크롤 동기화)│
  └──────────────┴──────────────┘
  │         요약본 (접이식 하단)         │
  └─────────────────────────────────────┘

구성 요소:
  - 파일 업로드: PDF / TXT 드래그&드롭
  - 좌측: 원문 (문단 선택 → 하이라이트)
  - 우측: DeepL 번역 실시간 표시
  - 하단: Claude AI 전체 요약 (접이식)
  - 번역 언어 선택 드롭다운
  - 드래그 핸들 더블클릭 → 50:50 리셋

스크롤 동기화:
  - Intersection Observer API로 현재 문단 감지
  - 좌우 패널 동일 문단 위치 유지

번역 최적화:
  - 문단 단위 lazy 번역 (화면에 보이는 영역만)
  - 번역 완료 문단 세션 내 캐싱

모바일:
  - 탭 전환 방식 (원문 / 번역 / 요약 탭)
```

## 화면 6 — 통계분석 추천

```
목적: 연구 주제에 적합한 통계분석 5~7가지 추천

구성 요소:
  - 연구 주제 + 변수 설명 입력창
  - 결과 카드 5~7개:
      - 분석 방법명 (예: 회귀분석, 구조방정식 등)
      - 적합 이유 (2~3줄)
      - 적용 사례 예시
      - 난이도 배지 (초급 / 중급 / 고급)
      - 추천 도구: R 또는 Python 버튼
      - "직접 실행하기" 버튼 → 화면7 이동 (선택 도구 자동 탭 전환)

AI 프롬프트:
  - 연구 주제, 변수 유형, 연구 유형 컨텍스트 주입
  - 데이터 분석가 패널 역할 반영
  - 각 방법별 R 또는 Python 추천 사유 포함
```

## 화면 7 — 통계 직접 실행 (R / Python)

```
목적: 브라우저에서 R / Python 직접 실행

탭 구성: R | Python

공통 요소:
  - Monaco 코드 에디터 (다크 테마)
  - ▶ 실행 버튼 (cursor: pointer, 클릭형 커서)
  - 결과 출력 패널 (검은 배경)
  - AI 통계 결과 자동 해석 패널 (Claude API)
  - 초보자 모드 토글
  - 데이터 파일 업로드 (CSV / Excel)

초보자 모드 ON:
  - 단계별 가이드 오버레이 (툴팁 + 화살표)
  - 샘플 코드 자동 삽입
  - 각 코드 줄 한국어 설명 사이드 패널

R 탭 (WebR):
  - webr CDN (WebAssembly) 로드
  - 사전 로드 패키지: stats, tidyverse, ggplot2, psych, lavaan, lme4
  - CSV 업로드 → R 데이터프레임 자동 변환
  - 샘플: 회귀분석, 상관분석, t-검정, ANOVA

Python 탭 (Pyodide):
  - Pyodide CDN 로드
  - 사전 로드: pandas, numpy, matplotlib, scipy, sklearn, statsmodels, seaborn
  - CSV 업로드 → DataFrame 자동 변환
  - 샘플: 선형회귀, 클러스터링, 시각화
```

## 화면 8 — 학술지 심사 코칭

```
[논문 업로드 & 4인 피드백]
목적: 완성된 논문을 AI 전문가 4인이 피드백 초안 제공

구성 요소:
  - 논문 업로드: PDF / DOCX 드래그&드롭
  - 업로드 후 분석 진행 애니메이션
  - 4인 전문가 피드백 카드:
      - 논문 다작 박사: 전반적 완성도, 논문 구조
      - 논문 심사위원: 방법론 + 이론적 배경 관점
      - 논문 지도 교수: 종합 코칭, 방향성
      - 데이터 분석가: 통계분석 타당성
  - 강점 / 보완점 섹션별 상세 리스트
  - 피드백 초안 PDF 다운로드

[가상 논문 심사 실습 - 교수 3인]
목적: 실제 심사처럼 질문-답변 연습

진행 순서:
  ① 논문 제목 / 초록 입력 (또는 업로드본 자동 추출)
  ② 가상 교수 A (방법론) 질문 → 연구자 텍스트 답변
  ③ 가상 교수 B (이론적 배경) 질문 → 연구자 답변
  ④ 가상 교수 C (통계 타당성) 질문 → 연구자 답변
  ⑤ 각 답변 후 AI 즉각 피드백 (적절성 + 보완점)
  ⑥ 심사 종료 → 종합 리포트

교수 캐릭터 시스템 프롬프트:
  - 교수 A: "연구 방법론의 엄밀성을 날카롭게 검토하는 심사위원"
  - 교수 B: "이론적 배경의 충분성과 선행연구를 꼼꼼히 확인하는 원로 교수"
  - 교수 C: "통계적 타당성과 결과 해석을 집중 검토하는 심사위원"

결과 리포트:
  - 강점 목록
  - 보완 필요 항목 우선순위
  - 추천 공부 영역
  - PDF 다운로드
```

---

# 5. AI 전문가 패널 시스템 (4인)

## 역할별 시스템 프롬프트

```python
# services/panel_service.py
PANEL_ROLES = {
    "prolific_doctor": """
당신은 논문을 50편 이상 집필한 시니어 연구자입니다.
연구 주제의 타당성, 논문 구조, 선행연구 연결성에 집중하여 의견을 제시합니다.
구체적이고 실용적으로, 200자 이내로 작성하세요.
    """,
    "thesis_reviewer": """
당신은 다년간 학술지 심사 경험이 있는 논문 심사위원입니다.
연구 방법론, 이론적 배경, 표본 설계, 측정 도구의 타당도를 날카롭게 검토합니다.
잠재적 문제점도 함께 지적하세요. 200자 이내로 작성하세요.
    """,
    "advisor_professor": """
당신은 논문 지도 경험이 풍부한 교수입니다.
연구자의 전반적인 방향성을 코칭하고, 격려와 함께 구체적 개선안을 제시합니다.
200자 이내로 작성하세요.
    """,
    "data_analyst": """
당신은 연구 데이터 분석 전문가입니다.
연구 주제에 맞는 통계분석 방법 선택, 데이터 처리 방법, 결과 해석의 적절성을 평가합니다.
R 또는 Python 기반의 구체적 방법을 언급하며 200자 이내로 작성하세요.
    """,
}
```

## 4인 패널 병렬 처리

```python
import asyncio
from anthropic import AsyncAnthropic
import json

client = AsyncAnthropic()

async def get_panel_opinion(role: str, question: str, context: dict) -> dict:
    system = PANEL_ROLES[role]
    user_content = f"""
연구 유형: {context.get('research_type', '학술지')}
연구 주제: {context.get('topic', '미정')}
연구자 질문: {question}
"""
    response = await client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=500,
        system=system,
        messages=[{"role": "user", "content": user_content}]
    )
    return {"role": role, "opinion": response.content[0].text}


async def synthesize_opinions(opinions: list[dict], question: str) -> dict:
    opinions_text = "\n".join([f"[{o['role']}]: {o['opinion']}" for o in opinions])
    prompt = f"""
4인 전문가 패널 의견:
{opinions_text}

위 의견을 종합하여 반드시 아래 JSON 형식으로만 반환하세요:
{{
  "summary": "핵심 통합 의견 (300자 이내)",
  "options": [
    {{"label": "A안", "description": "...", "pros": "...", "cons": "..."}},
    {{"label": "B안", "description": "...", "pros": "...", "cons": "..."}},
    {{"label": "C안", "description": "...", "pros": "...", "cons": "..."}}
  ]
}}
"""
    response = await client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1000,
        messages=[{"role": "user", "content": prompt}]
    )
    return json.loads(response.content[0].text)


async def run_panel(question: str, context: dict) -> dict:
    """4인 패널 병렬 실행"""
    opinions = await asyncio.gather(*[
        get_panel_opinion(role, question, context)
        for role in PANEL_ROLES.keys()
    ])
    result = await synthesize_opinions(list(opinions), question)
    result["individual_opinions"] = list(opinions)
    return result
```

## Claude 스트리밍 (연구 상담 챗봇)

```python
async def stream_chat(message: str, history: list, context: dict):
    system = f"""
당신은 AI 논문 연구를 지도하는 친절한 멘토입니다.
연구 유형: {context.get('research_type', '학술지')}
연구 주제: {context.get('topic', '미정')}
한국어로 응답하고, 연구자의 아이디어를 구체화하도록 질문으로 유도하세요.
"""
    async with client.messages.stream(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1000,
        system=system,
        messages=history + [{"role": "user", "content": message}]
    ) as stream:
        async for text in stream.text_stream:
            yield f"data: {text}\n\n"
```

---

# 6. 외부 API 연동 가이드

## rq-ai 연동

```typescript
// components/research-idea/RqAiEmbed.tsx
export function RqAiEmbed({ onKeywordsReceived }: { onKeywordsReceived: (kw: string[]) => void }) {
  useEffect(() => {
    const handler = (event: MessageEvent) => {
      if (event.origin !== 'https://rq-ai-620920432884.asia-east1.run.app') return;
      if (event.data?.type === 'RESEARCH_KEYWORDS') {
        onKeywordsReceived(event.data.keywords);
      }
    };
    window.addEventListener('message', handler);
    return () => window.removeEventListener('message', handler);
  }, [onKeywordsReceived]);

  return (
    <iframe
      src="https://rq-ai-620920432884.asia-east1.run.app/"
      className="w-full h-[600px] border-0 rounded-lg"
      title="AI 연구 주제 상담"
      allow="clipboard-write"
    />
  );
}
```

## Google Scholar (SerpAPI)

```python
# services/scholar_service.py
import httpx
from app.config import settings
from app.constants import LiteratureSource   # source enum 일관성 유지

SERPAPI_BASE = "https://serpapi.com/search"

async def search_google_scholar(keyword: str, year_from: int = 2015) -> list:
    """
    SerpAPI를 통한 Google Scholar 검색
    ⚠️ SerpAPI는 유료 서비스 (월 100회 무료 플랜 존재)
    대안: scholarly 라이브러리 (비공식, 이용약관 확인 필요)
    """
    params = {
        "engine": "google_scholar",
        "q": keyword,
        "as_ylo": year_from,
        "hl": "ko",
        "num": 20,
        "api_key": settings.SERPAPI_KEY,
    }
    async with httpx.AsyncClient() as client:
        response = await client.get(SERPAPI_BASE, params=params, timeout=15.0)
        response.raise_for_status()
        results = response.json().get("organic_results", [])
        return [_parse_scholar_result(r) for r in results]

def _parse_scholar_result(item: dict) -> dict:
    return {
        "title": item.get("title", ""),
        "authors": item.get("publication_info", {}).get("authors", []),
        "year": item.get("publication_info", {}).get("year"),
        "citation_count": item.get("inline_links", {}).get("cited_by", {}).get("total", 0),
        "abstract": item.get("snippet", ""),
        "url": item.get("link", ""),
        "source": LiteratureSource.SCHOLAR   # enum 사용 — "google_scholar"
    }
```

## 국회도서관 API

> ⚠️ **가용성 주의**: 국회도서관(dl.nanet.go.kr) Open API는 공개 범위가 제한적입니다.
> - **1순위**: 국립중앙도서관 Open API (www.nl.go.kr, 공식 무료 제공) 병행 사용 권장
> - **2순위**: 국회도서관 사서서비스 API (기관 계정 필요 시)
> - API 호출 실패 시 → KCI DB 결과만으로 fallback 처리

```python
# services/nanet_service.py
import httpx
import logging
from app.config import settings

logger = logging.getLogger(__name__)

# 국회도서관 API (공개 범위 제한적)
NANET_BASE = "https://www.nanet.go.kr/openapi/search"

# 국립중앙도서관 API (대안 - 공식 무료 제공)
NLA_BASE = "https://www.nl.go.kr/NL/search/openApi/search.do"


async def search_nanet(keyword: str) -> list:
    """
    국회도서관 Open API 검색
    실패 시 국립중앙도서관 API로 자동 fallback
    """
    try:
        result = await _search_nanet_primary(keyword)
        if result:
            return result
        logger.warning("국회도서관 API 결과 없음 → 국립중앙도서관으로 fallback")
    except Exception as e:
        logger.warning(f"국회도서관 API 실패: {e} → 국립중앙도서관으로 fallback")

    return await _search_nla_fallback(keyword)


async def _search_nanet_primary(keyword: str) -> list:
    """국회도서관 API 호출 (1순위)"""
    params = {
        "key": settings.NANET_API_KEY,
        "query": keyword,
        "startNo": 1,
        "displayCount": 20,
        "mediaCode": "BO",
    }
    async with httpx.AsyncClient() as client:
        response = await client.get(NANET_BASE, params=params, timeout=8.0)
        response.raise_for_status()
        data = response.json()
        items = data.get("result", {}).get("list", [])
        return [_parse_nanet_result(r) for r in items]


async def _search_nla_fallback(keyword: str) -> list:
    """국립중앙도서관 API (fallback - 공식 무료 제공)"""
    params = {
        "key": settings.NLA_API_KEY,       # 국립중앙도서관 별도 키
        "srchTarget": "total",
        "kwd": keyword,
        "pageNum": 1,
        "pageSize": 20,
        "systemType": "오픈API",
    }
    async with httpx.AsyncClient() as client:
        response = await client.get(NLA_BASE, params=params, timeout=8.0)
        response.raise_for_status()
        data = response.json()
        items = data.get("result", [])
        return [_parse_nla_result(r) for r in items]


def _parse_nanet_result(item: dict) -> dict:
    return {
        "title": item.get("titleInfo", ""),
        "authors": item.get("authorInfo", ""),
        "year": item.get("pubYear"),
        "citation_count": 0,
        "abstract": item.get("abstractInfo", ""),
        "url": item.get("titleUrl", ""),
        "source": LiteratureSource.NANET,    # enum 사용
    }


def _parse_nla_result(item: dict) -> dict:
    return {
        "title": item.get("title", ""),
        "authors": item.get("author", ""),
        "year": item.get("pubYear"),
        "citation_count": 0,
        "abstract": item.get("description", ""),
        "url": item.get("url", ""),
        "source": LiteratureSource.NANET,    # 동일 소스 분류로 처리
    }
```

## DeepL 번역

```python
# services/deepl_service.py
import deepl

translator = deepl.Translator(settings.DEEPL_API_KEY)

def translate_paragraph(text: str, target_lang: str = "KO") -> str:
    """문단 단위 번역 (lazy 방식으로 호출)"""
    result = translator.translate_text(text, target_lang=target_lang)
    return result.text

async def summarize_paper(full_text: str) -> str:
    """논문 전체 AI 요약 (Claude API)"""
    response = await client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=600,
        messages=[{
            "role": "user",
            "content": f"다음 논문을 한국어로 300자 이내로 핵심 내용만 요약하세요:\n\n{full_text[:4000]}"
        }]
    )
    return response.content[0].text
```

---

# 7. KCI CSV 데이터 처리

## CSV 임포트 스크립트

```python
# app/scripts/import_kci_csv.py
import pandas as pd
from sqlalchemy import create_engine, text
import argparse
import os
import sys
sys.path.insert(0, ".")
from app.constants import LiteratureSource   # source enum 일관성 유지

def import_kci_csv(csv_path: str, db_url: str):
    """
    한국연구재단 KCI 논문정보 CSV를 PostgreSQL에 임포트
    실행: python import_kci_csv.py --file 한국연구재단_KCI논문정보_20250825.csv
    """
    print(f"CSV 로딩 중: {csv_path}")
    df = pd.read_csv(csv_path, encoding="utf-8-sig")  # BOM 처리

    # 컬럼명 표준화 (CSV 컬럼명에 따라 조정 필요)
    # ── 실제 CSV 컬럼명 자동 탐지 ──
    actual_columns = df.columns.tolist()
    print(f"[KCI CSV] 실제 컬럼 목록: {actual_columns}")

    # 후보 컬럼명 다중 매핑 (KCI CSV 버전마다 컬럼명 다를 수 있음)
    COLUMN_CANDIDATES = {
        "title":          ["논문명", "논문제목", "제목", "Title", "title"],
        "authors":        ["저자", "저자명", "Authors", "author"],
        "year":           ["발행연도", "출판연도", "연도", "Year", "year", "pub_year"],
        "journal":        ["학술지명", "저널명", "학술지", "Journal", "journal"],
        "citation_count": ["피인용수", "피인용횟수", "인용수", "CitationCount", "cited_count"],
        "kci_grade":      ["KCI등재여부", "등재여부", "등재구분", "KCI_grade", "kci"],
        "issn":           ["ISSN", "issn"],
        "doi":            ["DOI", "doi"],
        "abstract":       ["초록", "요약", "Abstract", "abstract"],
        "url":            ["URL", "url", "링크", "link"],
    }

    column_mapping = {}
    unmatched_targets = []
    for target, candidates in COLUMN_CANDIDATES.items():
        matched = next((c for c in candidates if c in actual_columns), None)
        if matched:
            column_mapping[matched] = target
        else:
            unmatched_targets.append(target)

    if unmatched_targets:
        print(f"[KCI CSV] ⚠️ 매핑 실패 컬럼: {unmatched_targets}")
        print(f"[KCI CSV] → 해당 컬럼은 None으로 처리됩니다.")
        print(f"[KCI CSV] → COLUMN_CANDIDATES에 실제 컬럼명을 추가하세요.")

    df = df.rename(columns=column_mapping)

    # 필수 컬럼 없으면 기본값
    for col in ["citation_count", "kci_grade", "doi", "abstract", "url", "issn"]:
        if col not in df.columns:
            df[col] = None

    df["source"] = LiteratureSource.KCI    # enum 사용 — "kci"
    df["citation_count"] = pd.to_numeric(df.get("citation_count", 0), errors="coerce").fillna(0).astype(int)

    engine = create_engine(db_url)
    df.to_sql("kci_papers", engine, if_exists="replace", index=False, chunksize=1000)
    print(f"KCI 논문 {len(df):,}건 임포트 완료")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--file", required=True)
    args = parser.parse_args()
    import_kci_csv(args.file, os.environ["DATABASE_URL"])
```

## Source 상수 Enum 정의

```python
# app/constants.py  ← 전체 서비스에서 공통으로 import
from enum import Enum

class LiteratureSource(str, Enum):
    """선행연구 출처 상수 — 오타/혼용 방지용 enum"""
    KCI     = "kci"           # 한국연구재단 KCI 로컬 DB
    NANET   = "nanet"         # 국회도서관 (또는 국립중앙도서관 fallback)
    SCHOLAR = "google_scholar" # Google Scholar (SerpAPI)

# 사용 예시:
# source = LiteratureSource.KCI          → "kci"
# source = LiteratureSource.NANET        → "nanet"
# source = LiteratureSource.SCHOLAR      → "google_scholar"
# TypeScript 타입과 동일하게 유지:
# source: 'kci' | 'nanet' | 'google_scholar'
```

## KCI DB 검색 서비스

```python
# services/kci_service.py
from sqlalchemy import text
from app.core.dependencies import get_db
from app.constants import LiteratureSource

async def search_kci(keyword: str, year_from: int = 2015, kci_only: bool = False) -> list:
    """
    KCI 로컬 DB 전문검색
    - PostgreSQL 전문검색(to_tsvector) 또는 ILIKE 사용
    """
    query = text("""
        SELECT title, authors, year, journal, citation_count, kci_grade, doi, abstract, url, source
        FROM kci_papers
        WHERE (title ILIKE :kw OR authors ILIKE :kw OR abstract ILIKE :kw)
          AND year >= :year_from
          AND (:kci_only = false OR kci_grade IS NOT NULL)
        ORDER BY citation_count DESC NULLS LAST, year DESC
        LIMIT 30
    """)
    async with get_db() as db:
        result = await db.execute(query, {
            "kw": f"%{keyword}%",
            "year_from": year_from,
            "kci_only": kci_only,
        })
        rows = result.fetchall()
    return [dict(row._mapping) for row in rows]
```

## 통합 선행연구 검색 (병렬 처리)

```python
# services/literature_service.py
async def search_all(keyword: str, sources: list[str], year_from: int) -> list:
    """3개 소스 병렬 검색 후 병합"""
    tasks = []
    if "kci" in sources:
        tasks.append(search_kci(keyword, year_from))
    if "nanet" in sources:
        tasks.append(search_nanet(keyword))
    if "scholar" in sources:
        tasks.append(search_google_scholar(keyword, year_from))

    results = await asyncio.gather(*tasks, return_exceptions=True)

    merged = []
    seen_titles = set()
    for result in results:
        if isinstance(result, Exception):
            continue  # 개별 소스 실패 시 무시
        for paper in result:
            normalized = paper["title"].strip().lower()
            if normalized not in seen_titles:
                seen_titles.add(normalized)
                merged.append(paper)

    # 피인용 횟수 내림차순 정렬
    merged.sort(key=lambda x: x.get("citation_count", 0), reverse=True)
    return merged
```

---

# 8. 통계 프로그램 연동 (R / Python)

## WebR — 브라우저 내 R 실행

```typescript
// components/statistics/RConsole.tsx
'use client';
import { useEffect, useState } from 'react';

const SAMPLE_CODE = `# 선형 회귀분석 예시
data <- read.csv("your_data.csv")
model <- lm(dependent_var ~ independent_var, data = data)
summary(model)
plot(model)`;

export function RConsole({ initialCode = SAMPLE_CODE }: { initialCode?: string }) {
  const [webR, setWebR] = useState<any>(null);
  const [code, setCode] = useState(initialCode);
  const [output, setOutput] = useState('WebR 초기화 중...');
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    const init = async () => {
      try {
        const { WebR } = await import('https://webr.r-wasm.org/latest/webr.mjs' as any);
        const wr = new WebR();
        await wr.init();
        // 사전 패키지 로드
        await wr.evalR('suppressMessages(library(stats))');
        setWebR(wr);
        setIsReady(true);
        setOutput('R 준비 완료. 코드를 실행하세요.');
      } catch (e: any) {
        setOutput(`초기화 실패: ${e.message}`);
      }
    };
    init();
  }, []);

  const runCode = async () => {
    if (!webR || !isReady) return;
    setOutput('실행 중...');
    try {
      const shelter = await webR.evalR(code);
      const result = await shelter.toJs();
      setOutput(String(result));
    } catch (e: any) {
      setOutput(`오류: ${e.message}`);
    }
  };

  return (
    <div className="flex flex-col h-full gap-3">
      <textarea
        value={code}
        onChange={e => setCode(e.target.value)}
        className="flex-1 font-mono text-sm bg-gray-900 text-green-400 p-4 rounded-lg resize-none"
        placeholder="R 코드를 입력하세요..."
        spellCheck={false}
      />
      <button
        onClick={runCode}
        disabled={!isReady}
        style={{ cursor: isReady ? 'pointer' : 'not-allowed' }}
        className="bg-green-600 hover:bg-green-700 disabled:bg-gray-600 text-white px-6 py-2 rounded-lg font-semibold transition-colors"
      >
        ▶ 실행
      </button>
      <div className="h-48 bg-black text-green-300 font-mono text-sm p-4 rounded-lg overflow-auto whitespace-pre-wrap">
        {output}
      </div>
    </div>
  );
}
```

## Pyodide — 브라우저 내 Python 실행

```typescript
// components/statistics/PyConsole.tsx
'use client';
import { useEffect, useState } from 'react';

const SAMPLE_CODE = `import pandas as pd
import numpy as np
from scipy import stats

# 예시: 상관분석
data = pd.DataFrame({'x': [1,2,3,4,5], 'y': [2,4,5,4,5]})
corr, pval = stats.pearsonr(data['x'], data['y'])
print(f"피어슨 상관계수: {corr:.4f}, p값: {pval:.4f}")`;

export function PyConsole({ initialCode = SAMPLE_CODE }: { initialCode?: string }) {
  const [pyodide, setPyodide] = useState<any>(null);
  const [code, setCode] = useState(initialCode);
  const [output, setOutput] = useState('Python 환경 로딩 중...');
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    const init = async () => {
      try {
        const script = document.createElement('script');
        script.src = 'https://cdn.jsdelivr.net/pyodide/v0.25.0/full/pyodide.js';
        document.head.appendChild(script);
        script.onload = async () => {
          const py = await (window as any).loadPyodide({
            indexURL: 'https://cdn.jsdelivr.net/pyodide/v0.25.0/full/'
          });
          await py.loadPackage(['numpy', 'pandas', 'scipy', 'matplotlib', 'statsmodels', 'scikit-learn']);
          setPyodide(py);
          setIsReady(true);
          setOutput('Python 준비 완료. 코드를 실행하세요.');
        };
      } catch (e: any) {
        setOutput(`초기화 실패: ${e.message}`);
      }
    };
    init();
  }, []);

  const runCode = async () => {
    if (!pyodide || !isReady) return;
    setOutput('실행 중...');
    try {
      pyodide.runPython('import sys; from io import StringIO; _stdout = StringIO(); sys.stdout = _stdout');
      await pyodide.runPythonAsync(code);
      const result = pyodide.runPython('sys.stdout = sys.__stdout__; _stdout.getvalue()');
      setOutput(result || '(출력 없음)');
    } catch (e: any) {
      setOutput(`오류: ${e.message}`);
    }
  };

  return (
    <div className="flex flex-col h-full gap-3">
      <textarea
        value={code}
        onChange={e => setCode(e.target.value)}
        className="flex-1 font-mono text-sm bg-gray-900 text-yellow-300 p-4 rounded-lg resize-none"
        placeholder="Python 코드를 입력하세요..."
        spellCheck={false}
      />
      <button
        onClick={runCode}
        disabled={!isReady}
        style={{ cursor: isReady ? 'pointer' : 'not-allowed' }}
        className="bg-blue-600 hover:bg-blue-700 disabled:bg-gray-600 text-white px-6 py-2 rounded-lg font-semibold transition-colors"
      >
        ▶ 실행
      </button>
      <div className="h-48 bg-black text-yellow-200 font-mono text-sm p-4 rounded-lg overflow-auto whitespace-pre-wrap">
        {output}
      </div>
    </div>
  );
}
```

## 통계 결과 자동 해석

```typescript
// 통계 출력 → Claude API → 한국어 해석
const interpretResult = async (statOutput: string, method: string): Promise<string> => {
  const response = await fetch('/api/v1/statistics/interpret', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ output: statOutput, method })
  });
  const data = await response.json();
  return data.data?.interpretation ?? '';
};
```

## 초보자 가이드 오버레이

```typescript
// components/statistics/GuideOverlay.tsx
const GUIDE_STEPS = [
  { target: '#data-upload-btn', message: '① CSV 또는 Excel 데이터를 업로드하세요', position: 'bottom' as const },
  { target: '#code-editor',     message: '② 샘플 코드가 자동 삽입됩니다. 변수명을 맞게 수정하세요', position: 'right' as const },
  { target: '#run-button',      message: '③ ▶ 실행 버튼을 클릭하면 분석이 시작됩니다', position: 'top' as const },
  { target: '#output-panel',    message: '④ 분석 결과가 표시됩니다', position: 'top' as const },
  { target: '#interpret-panel', message: '⑤ AI가 결과를 한국어로 해석해드립니다', position: 'left' as const },
];
```

---

# 9. 백엔드 API 설계

```
# 인증
POST   /api/v1/auth/login                   # JWT 발급 (소셜 로그인)
POST   /api/v1/auth/refresh                 # 토큰 갱신

# 연구 세션
POST   /api/v1/research/session             # 세션 생성
GET    /api/v1/research/session/{id}        # 세션 조회

# 연구 상담 챗봇
POST   /api/v1/chat/message                 # 메시지 전송 (SSE 스트리밍)
GET    /api/v1/chat/history/{session_id}    # 대화 히스토리

# 선행연구 (통합 검색)
POST   /api/v1/literature/search            # 통합 검색 (Scholar + NaNet + KCI)
GET    /api/v1/literature/saved             # 저장 목록
POST   /api/v1/literature/save              # 논문 저장

# 번역 & 요약
POST   /api/v1/translate                    # 문단 번역 (DeepL)
POST   /api/v1/summarize                    # 논문 요약 (Claude)

# 연구모형 갤러리
GET    /api/v1/models/images                # 이미지 목록 (S3)
POST   /api/v1/models/upload                # 이미지 업로드

# AI 전문가 패널 (4인)
POST   /api/v1/panel/consult                # 패널 협의 요청 (Celery 비동기)
GET    /api/v1/panel/result/{task_id}       # 협의 결과 폴링

# 통계
POST   /api/v1/statistics/recommend         # 통계분석 5~7가지 추천
POST   /api/v1/statistics/interpret         # 결과 해석 (Claude)

# 심사 코칭
POST   /api/v1/review/upload                # 논문 파일 업로드 (S3)
POST   /api/v1/review/feedback              # 4인 피드백 요청
POST   /api/v1/defense/start                # 가상 심사 시작
POST   /api/v1/defense/answer               # 답변 + 즉각 피드백
GET    /api/v1/defense/report/{id}          # 심사 리포트 조회
```

---

# 10. 데이터베이스 스키마

```sql
-- 사용자
CREATE TABLE users (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email         VARCHAR(255) UNIQUE NOT NULL,
    name          VARCHAR(100),
    institution   VARCHAR(200),
    provider      VARCHAR(50) NOT NULL,           -- 'google', 'kakao'
    created_at    TIMESTAMPTZ DEFAULT NOW()
);

-- 연구 세션
CREATE TABLE research_sessions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    research_type   VARCHAR(50) DEFAULT 'journal', -- 'journal', 'master', 'doctoral'
    topic           TEXT,
    keywords        JSONB DEFAULT '[]',
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- 상담 대화 히스토리
CREATE TABLE chat_histories (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id  UUID REFERENCES research_sessions(id) ON DELETE CASCADE,
    role        VARCHAR(20) NOT NULL,              -- 'user', 'assistant'
    content     TEXT NOT NULL,
    created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- KCI 논문 (CSV 임포트)
CREATE TABLE kci_papers (
    id             SERIAL PRIMARY KEY,
    title          TEXT NOT NULL,
    authors        TEXT,
    year           INTEGER,
    journal        VARCHAR(500),
    citation_count INTEGER DEFAULT 0,
    kci_grade      VARCHAR(100),
    issn           VARCHAR(20),
    doi            VARCHAR(200),
    abstract       TEXT,
    url            TEXT,
    source         VARCHAR(20) DEFAULT 'kci',
    created_at     TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_kci_title ON kci_papers USING gin(to_tsvector('simple', title));
CREATE INDEX idx_kci_year ON kci_papers(year DESC);
CREATE INDEX idx_kci_citation ON kci_papers(citation_count DESC);

-- 저장된 선행연구
CREATE TABLE saved_literatures (
    id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id          UUID REFERENCES users(id) ON DELETE CASCADE,
    title            TEXT NOT NULL,
    authors          TEXT,
    year             INTEGER,
    journal          VARCHAR(500),
    citation_count   INTEGER DEFAULT 0,
    kci_grade        VARCHAR(100),
    abstract_summary TEXT,
    source_url       TEXT,
    source           VARCHAR(50),                  -- 'kci', 'nanet', 'google_scholar'
    created_at       TIMESTAMPTZ DEFAULT NOW()
);

-- AI 패널 협의 결과
CREATE TABLE panel_results (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id          UUID REFERENCES research_sessions(id),
    user_question       TEXT NOT NULL,
    individual_opinions JSONB,                     -- 4인 각 의견
    summary             TEXT,
    options             JSONB,                     -- A안/B안/C안
    created_at          TIMESTAMPTZ DEFAULT NOW()
);

-- 통계 결과 해석 (재조회 가능하도록 저장)
CREATE TABLE statistics_interpretations (
    id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id     UUID REFERENCES research_sessions(id),
    method         VARCHAR(200),
    stat_output    TEXT,
    interpretation TEXT,
    tool           VARCHAR(20),                    -- 'r', 'python'
    created_at     TIMESTAMPTZ DEFAULT NOW()
);

-- 가상 심사 결과
CREATE TABLE mock_defense_results (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id  UUID REFERENCES research_sessions(id),
    qa_pairs    JSONB,                             -- [{professor, question, answer, feedback}]
    report      JSONB,                             -- {strengths, weaknesses, recommendations}
    created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

---

# 11. 인증 및 보안

```python
# core/security.py
from datetime import datetime, timedelta
from jose import jwt

def create_access_token(user_id: str) -> str:
    payload = {
        "sub": user_id,
        "exp": datetime.utcnow() + timedelta(hours=1),
        "type": "access"
    }
    return jwt.encode(payload, settings.JWT_SECRET_KEY, algorithm="HS256")
```

```
보안 체크리스트
  ✅ register 페이지 없음 (소셜 로그인 자동 생성만)
  ✅ 모든 API JWT 검증 미들웨어 적용
  ✅ Rate Limiting: 분당 60회
  ✅ CORS 허용 도메인 명시 (rq-ai 도메인 포함)
  ✅ S3 버킷 퍼블릭 차단, Signed URL 사용
  ✅ 외부 API 키 서버사이드 전용 관리
  ✅ KCI CSV 파일 서버 내부 전용 보관 (퍼블릭 노출 금지)
  ✅ Google Scholar 스크래핑 시 이용약관 준수
  ✅ HTTPS 강제
```

---

# 12. 스프린트 계획

## Sprint 1 (1~2주) — 기반 세팅
```
[ ] Next.js 14 + TypeScript + Tailwind 초기 설정
[ ] FastAPI 프로젝트 구조 생성
[ ] PostgreSQL + Redis Docker Compose
[ ] NextAuth.js Google + Kakao OAuth (register 없음)
[ ] JWT 미들웨어 + 자동 계정 생성 로직
[ ] DB 마이그레이션 (Alembic)
[ ] KCI CSV DB 임포트 스크립트 작성 및 실행
[ ] GitHub Actions CI/CD
```

## Sprint 2 (3~4주) — P0 핵심 기능
```
[ ] 로그인 화면 (소셜 전용)
[ ] rq-ai iFrame 연동 + postMessage 키워드 수신
[ ] Claude 챗봇 스트리밍 (연구 상담)
[ ] KCI DB 검색 서비스
[ ] 국회도서관 API 연동
[ ] Google Scholar SerpAPI 연동
[ ] 3소스 통합 병렬 검색 + 중복 제거
[ ] 학술지 작성 가이드 페이지 (5단계)
[ ] Redis 캐싱 적용
```

## Sprint 3 (5~6주) — P1 부가 기능
```
[ ] Split-View 번역 (DeepL + 스크롤 동기화 + 요약)
[ ] 연구모형 이미지 갤러리 (S3)
[ ] 통계분석 5~7가지 추천 (Claude API)
[ ] AI 전문가 패널 4인 협의 (asyncio 병렬)
[ ] Celery 작업 큐
```

## Sprint 4 (7~8주) — P2 + 마무리
```
[ ] WebR R 콘솔 + 사전 패키지 로드
[ ] Pyodide Python 콘솔 + 사전 패키지 로드
[ ] 통계 결과 자동 해석 (Claude) + DB 저장
[ ] 초보자 가이드 오버레이
[ ] 논문 업로드 (PDF/DOCX) + S3 저장
[ ] 4인 심사 피드백 초안 생성
[ ] 가상 심사 실습 (교수 3인) + 리포트 PDF
[ ] 전체 QA + 성능 최적화
[ ] Vercel + AWS 프로덕션 배포
```

---

# 13. 코딩 컨벤션

```
커밋: feat: / fix: / docs: / refactor: / test: / chore:
브랜치: main / develop / feature/기능명
PR: 최소 1인 리뷰 후 머지
환경변수: .env 커밋 금지, .env.example 필수
```

```typescript
// PascalCase: 컴포넌트
// camelCase: 훅, 유틸, 변수
// interface + Props 접미사
// any 타입 금지 (unknown 사용)
interface PaperCardProps {
  title: string;
  citationCount: number;
  kciGrade?: string;
  source: 'kci' | 'nanet' | 'google_scholar';
}
```

```python
# snake_case: 파일명, 함수, 변수
# PascalCase: 클래스
# UPPER_SNAKE_CASE: 상수
# 타입 힌트 필수, Docstring 필수
async def search_kci(keyword: str, year_from: int = 2015) -> list[dict]:
    """KCI 로컬 DB 선행연구 검색"""
    pass
```

---

# 14. 환경변수 목록

```env
# AI
ANTHROPIC_API_KEY=sk-ant-...

# 번역
DEEPL_API_KEY=...

# 선행연구
SERPAPI_KEY=...                            # Google Scholar (SerpAPI)
NANET_API_KEY=...                          # 국회도서관 Open API
NLA_API_KEY=...                            # 국립중앙도서관 Open API (fallback용, www.nl.go.kr)

# 인증
NEXTAUTH_SECRET=...
NEXTAUTH_URL=http://localhost:3000
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
KAKAO_CLIENT_ID=...
KAKAO_CLIENT_SECRET=...

# JWT
JWT_SECRET_KEY=...

# DB
DATABASE_URL=postgresql://user:password@localhost:5432/scholai

# Redis
REDIS_URL=redis://localhost:6379

# AWS S3
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_S3_BUCKET=scholai-assets
AWS_REGION=ap-northeast-2

# 외부 연동
RQ_AI_URL=https://rq-ai-620920432884.asia-east1.run.app/

# 앱
ENVIRONMENT=development
ALLOWED_ORIGINS=http://localhost:3000

# KCI CSV (로컬 경로)
KCI_CSV_PATH=data/한국연구재단_KCI논문정보_20250825.csv
```

---

# 15. 배포 구성

```yaml
# docker-compose.yml
version: '3.9'
services:
  api:
    build: ./scholai-backend
    ports: ["8000:8000"]
    env_file: .env
    depends_on: [db, redis]

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: scholai
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes: [pgdata:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine

  worker:
    build: ./scholai-backend
    command: celery -A app.worker worker --loglevel=info
    depends_on: [redis]

  # KCI CSV 초기 임포트 (최초 1회만 실행)
  kci-importer:
    build: ./scholai-backend
    command: python app/scripts/import_kci_csv.py --file data/한국연구재단_KCI논문정보_20250825.csv
    depends_on: [db]
    profiles: ["init"]   # docker-compose --profile init up kci-importer

volumes:
  pgdata:
```

---

# 16. 테스트 전략

## 프론트엔드
```
단위 (Jest + Testing Library):
  - PaperCard: KCI 배지 표시, 피인용 횟수 시각화
  - SplitView: 스크롤 동기화 동작
  - RConsole / PyConsole: 실행 버튼 클릭 시 API 호출

E2E (Playwright):
  - 로그인 → 연구 상담 → 선행연구 검색 → Split-View 전체 흐름
  - 통계 추천 → 직접 실행 화면 이동
  - 논문 업로드 → 피드백 생성
```

## 백엔드 (pytest)
```
단위:
  - kci_service: 키워드 검색, 피인용 정렬
  - scholar_service: SerpAPI 응답 파싱
  - panel_service: 4인 병렬 처리 검증
  - deepl_service: 번역 호출

통합:
  - /api/v1/literature/search: 3소스 병합 검색
  - /api/v1/panel/consult: 4인 협의 결과 구조
  - /api/v1/defense/answer: 피드백 생성

성능:
  - KCI DB 검색: 1초 이내
  - 패널 협의: 10초 이내
  - AI 응답: 3초 이내
  - 번역: 2초 이내
```

---

*문서 버전: v2.2 | 작성일: 2025*
*v2.2 보완: scholar_service enum 적용 · KCI import enum 적용 · constants import 완전 통일*
*v2.1 보완: 국회도서관 fallback → 국립중앙도서관 · KCI CSV 컬럼 자동탐지 · LiteratureSource enum 정의 · NLA_API_KEY 환경변수 추가*
*v2 변경: 전문가 패널 4인 · Scholar/국회도서관/KCI CSV 연동 · R/Python(Tableau 제거) · statistics_interpretations 테이블 추가*
