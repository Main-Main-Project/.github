# ⚖️ 법률 RAG 챗봇 프로젝트

## 1. 프로젝트 소개

본 프로젝트는 사용자가 업로드한 법률 문서를 기반으로 질문에 답변하는 **법률 문서 기반 RAG 챗봇 서비스**입니다.

사용자는 계약서, 판결문, 법률 자료 등의 문서를 업로드할 수 있으며, 시스템은 문서 내용을 분석한 뒤 사용자의 질문에 대해 관련 근거를 검색하고 답변을 생성합니다.

단순히 LLM에게 질문을 전달하는 구조가 아니라,
**사용자 문서 + 공용 법률 지식 DB + 출처 기반 답변 검증**을 함께 사용하는 구조를 목표로 합니다.

---

## 2. 핵심 목표

* 법률 문서 업로드 및 사용자별 문서 관리
* JWT 기반 사용자 인증
* 법률 관련 문서 여부 판별
* 문서 텍스트 추출 및 청킹
* Pinecone 기반 벡터 검색
* 공용 법률 DB와 개인 문서 DB 분리
* RAG 기반 답변 생성
* 답변 출처 표시
* 대화 이력 및 로그 저장
* Docker 기반 프론트엔드/백엔드 분리 운영
* 향후 문서 전용 Repository 구성

---

## 3. Repository 구성

현재 프로젝트는 프론트엔드와 백엔드를 분리하여 관리합니다.

| Repository | Description | Status |
| --- | --- | --- |
| [Front-end](https://github.com/Main-Main-Project/Front-end) | 사용자 화면 UI | 완료
| [Back-end](https://github.com/Main-Main-Project/Back-end) | FastAPI 기반 API 서버 | 완료
| [Docker](https://github.com/Main-Main-Project/Docker) | Docker 배포 | 완료
| Document | 프로젝트 발표 자료 pdf 및 시연 영상 | 완료 |
```

### Front-end Repository

사용자 화면을 담당합니다.

주요 역할은 다음과 같습니다.

* 로그인 / 회원가입 화면
* 문서 업로드 화면
* 챗봇 질문 입력 화면
* 답변 및 출처 표시
* 파일 처리 상태 표시
* 사용자별 문서 목록 조회

### Back-end Repository

API 서버와 AI 처리 흐름을 담당합니다.

주요 역할은 다음과 같습니다.

* JWT 인증 처리
* 파일 업로드 API
* 파일 확장자 및 용량 검사
* SHA-256 기반 중복 파일 검사
* 텍스트 추출
* 법률 문서 거름망
* S3 원본 파일 저장
* 텍스트 정제 및 청킹
* 임베딩 생성
* Pinecone 저장 및 검색
* LLM 답변 생성
* 대화 이력 및 로그 저장

### Document Repository

추후 프로젝트 문서를 별도 Repository로 분리하여 관리할 예정입니다.

포함 예정 문서는 다음과 같습니다.

* API 명세서
* ERD 다이어그램
* 전체 서비스 흐름도
* 화면 설계서
* DB 설계 문서
* 함수 명세서
* 배포 문서
* 트러블슈팅 기록

---

## 4. 전체 서비스 구조

```text
사용자
  ↓
프론트엔드
  ↓
백엔드 FastAPI 서버
  ↓
PostgreSQL / S3 / Pinecone / LLM
```

시연 단계에서는 다음과 같은 구조를 사용합니다.

```text
Front-end
  ↓
Cloudflare Tunnel
  ↓
FastAPI Server
  ↓
PostgreSQL
S3
Pinecone
Ollama 또는 Colab LLM
```

향후 고도화 단계에서는 FastAPI 서버를 Docker 이미지로 패키징하고, 클라우드 서버에 배포하는 구조로 확장할 예정입니다.

---

## 5. 주요 기술 스택

### Front-end

* React
* TypeScript
* Vite
* Docker

### Back-end

* Python
* FastAPI
* JWT Authentication
* Docker

### Database / Storage

* PostgreSQL
* AWS S3
* Pinecone

### AI / RAG

* Embedding Model
* Pinecone Vector Search
* Ollama 또는 Colab 기반 LLM
* 법률 문서 기반 RAG 구조

---

## 6. 파일 업로드 처리 흐름

사용자가 법률 문서를 업로드하면 다음 흐름으로 처리됩니다.

```text
1. 사용자 로그인 / JWT 인증
2. 파일 업로드
3. 파일 확장자 및 용량 검사
4. SHA-256 해시 생성
5. PostgreSQL에서 user_id + file_hash 기준 중복 체크
6. 중복 파일이면 업로드 차단
7. 파일 상태 UPLOADED 생성
8. 텍스트 추출
9. 법률 관련 문서 여부 판별
10. 법률 문서가 아니면 REJECTED 처리
11. 법률 문서이면 백그라운드 작업 등록
12. 사용자에게 "처리 중" 응답 반환
```

백그라운드에서는 다음 작업을 수행합니다.

```text
1. S3 원본 파일 저장
2. 추출 텍스트 재사용 또는 추가 추출
3. 전체 텍스트 정제
4. 조항 / 문단 단위 청킹
5. 임베딩 생성
6. Pinecone 개인DB 저장
7. PostgreSQL 파일 메타데이터 갱신
8. 파일 상태 READY 처리
9. 오류 발생 시 FAILED 처리 및 에러 로그 저장
```

---

## 7. 질문 답변 처리 흐름

사용자가 챗봇에 질문을 입력하면 다음 흐름으로 처리됩니다.

```text
1. 사용자 질문 입력
2. JWT 인증
3. session_id 확인
4. 질문 임베딩
5. 공용 Pinecone 검색
6. 개인 Pinecone 검색
7. 검색 결과 병합
8. 유사도 임계값 판별
9. 법률 관련 질문이 아니면 차단
10. Top-K 문서 추출
11. PostgreSQL에서 이전 대화 이력 조회
12. 프롬프트 조립
13. LLM 호출
14. 답변 생성
15. 출처 기반 할루시네이션 검증
16. 답변 반환
17. 대화 이력 저장
18. 로그 저장
```

---

## 8. RAG 구조

본 프로젝트의 RAG 구조는 다음 두 가지 벡터DB를 함께 사용합니다.

### 공용 Pinecone

공용 법률 지식 기반입니다.

예시 데이터는 다음과 같습니다.

* 법령
* 판례
* 행정해석
* 법률 기준 문서
* 법률 상담 예시

### 개인 Pinecone

사용자가 업로드한 문서를 저장하는 공간입니다.

개인 Pinecone에는 반드시 다음 메타데이터를 함께 저장합니다.

```json
{
  "user_id": "user_001",
  "file_id": "file_001",
  "chunk_id": "chunk_001",
  "page": 3,
  "source_type": "user_document",
  "filename": "contract.pdf"
}
```

개인 문서 검색 시에는 반드시 `user_id` 필터를 적용하여
다른 사용자의 문서가 검색되지 않도록 합니다.

---

## 9. ERD

DB 구조는 ERD 다이어그램으로 관리합니다.

현재 ERD는 ERDCloud를 기준으로 작성하며, 추후 Document Repository에 이미지 형태로 저장할 예정입니다.

### ERD 이미지 삽입 위치

<img width="2270" height="1692" alt="ERD" src="https://github.com/user-attachments/assets/0a06ae7b-2ba7-4c1c-ad0b-122b614fb5f1" />



ERD에는 다음과 같은 주요 테이블이 포함될 예정입니다.

* users
* logger
* chat_sessions
* messages
* message_sources
* documents
* document_chunks
* document_status_events

---

## 10. 주요 상태값

파일 처리 상태는 다음과 같이 관리합니다.

| 상태값             | 설명                |
| --------------- | ----------------- |
| UPLOADED        | 파일 업로드 요청 완료      |
| DUPLICATED      | 중복 파일             |
| REJECTED        | 법률 관련 파일이 아니어서 거절 |
| TEXT_EXTRACTING | 텍스트 추출 중          |
| EMBEDDING       | 임베딩 및 벡터DB 저장 중   |
| READY           | 처리 완료             |
| FAILED          | 처리 실패             |

프론트엔드는 이 상태값을 기반으로 사용자에게 현재 처리 상태를 표시합니다.

예시:

```text
파일 분석 중입니다.
벡터DB 저장 중입니다.
처리가 완료되었습니다.
법률 관련 파일이 아닙니다.
```

---

## 11. API 문서

API 명세서는 Notion에서 관리합니다.

주요 API는 다음과 같습니다.

### Auth

* 회원가입
* 로그인
* JWT 토큰 검증
* 사용자 정보 조회

### Documents

* 파일 업로드
* 파일 목록 조회
* 파일 처리 상태 조회
* 파일 삭제

### Chat

* 질문 요청
* 답변 반환
* 대화 이력 조회
* 세션 생성 및 조회

### Logs

* 파일 처리 로그 저장
* RAG 검색 로그 저장
* LLM 응답 로그 저장
* 오류 로그 저장

---

## 12. Docker 실행 구조

프론트엔드와 백엔드는 각각 Docker 기반으로 실행할 수 있도록 구성합니다.

### Front-end

```bash
docker build -t legal-rag-frontend .
docker run -p 5173:5173 legal-rag-frontend
```

### Back-end

```bash
docker build -t legal-rag-backend .
docker run -p 8000:8000 legal-rag-backend
```

### Docker Compose 예시

```yaml
services:
  frontend:
    build:
      context: ./Front-end
      dockerfile: Dockerfile
    ports:
      - "5173:5173"

  backend:
    build:
      context: ./Back-end
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    env_file:
      - .env
```

---

## 13. 환경 변수 예시

백엔드에서는 다음과 같은 환경 변수를 사용합니다.

```env
SECRET_KEY=
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60

DATABASE_URL=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=
S3_BUCKET_NAME=

PINECONE_API_KEY=
PINECONE_INDEX_NAME=

LLM_BASE_URL=
EMBEDDING_MODEL_NAME=
```

실제 `.env` 파일은 Git에 올리지 않고, `.env.example` 파일만 공유합니다.

---

## 14. 파일 삭제 흐름

사용자가 업로드한 파일을 삭제할 경우 다음 저장소를 함께 정리합니다.

```text
1. 사용자 삭제 요청
2. JWT 인증
3. 파일 소유자 확인
4. S3 원본 파일 삭제
5. Pinecone 개인DB에서 file_id 기준 벡터 삭제
6. PostgreSQL 파일 메타데이터 삭제 또는 deleted_at 처리
7. 삭제 완료 응답
```

가장 중요한 부분은 권한 확인입니다.

```text
file.user_id == current_user.id
```

현재 로그인한 사용자가 파일의 소유자가 아니라면 삭제를 거부합니다.

---

## 15. 보안 설계

본 프로젝트는 사용자별 문서와 대화 이력을 분리해야 하므로 인증과 권한 검사가 중요합니다.

주요 보안 기준은 다음과 같습니다.

* 모든 파일 업로드 요청은 JWT 인증 후 처리
* 모든 질문 요청은 JWT 인증 후 처리
* 개인 Pinecone 검색 시 user_id 필터 적용
* 파일 삭제 시 파일 소유자 검증
* `.env` 파일 Git 업로드 금지
* S3 파일 경로 직접 노출 금지
* 로그에는 민감정보 저장 최소화

---

## 16. 프로젝트 문서 관리 계획

추후 Document Repository를 생성하여 문서를 관리할 예정입니다.

---

## 17. 프로젝트 완성 기준

본 프로젝트의 완성 기준은 다음과 같습니다.

* 사용자가 로그인할 수 있다.
* 사용자가 법률 문서를 업로드할 수 있다.
* 법률 관련 문서만 처리된다.
* 중복 파일은 차단된다.
* 파일 처리 상태가 관리된다.
* 업로드 문서가 Pinecone에 저장된다.
* 사용자가 문서 기반 질문을 할 수 있다.
* 답변에 출처가 표시된다.
* 대화 이력이 저장된다.
* 파일 삭제 시 S3, Pinecone, PostgreSQL 데이터가 함께 정리된다.
* 프론트엔드와 백엔드가 Docker 기반으로 실행된다.

---

## 18. 최종 목표

이 프로젝트는 단순한 챗봇 구현이 아니라, 실제 서비스 구조에 가까운 법률 RAG 시스템 구현을 목표로 합니다.

핵심은 다음과 같습니다.

* 사용자별 문서 분리
* 법률 도메인 검증
* 벡터DB 기반 검색
* 출처 기반 답변
* 로그 기반 품질 개선
* Docker 기반 배포 구조
* 포트폴리오 확장 가능한 설계

이를 통해 팀 프로젝트 설명, API 설계, DB 설계, 함수 명세서, 배포 문서, 포트폴리오 정리까지 자연스럽게 연결할 수 있습니다.
