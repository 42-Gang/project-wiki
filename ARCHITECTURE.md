# 🏗️ 42-Gang 프로젝트 아키텍처

## 📋 목차
- [개요](#개요)
- [아키텍처 다이어그램](#아키텍처-다이어그램)
- [Backend](#backend)
- [Frontend](#frontend)
- [Infra](#infra)

## 개요

42-Gang은 실시간 탁구 게임 플랫폼으로, **마이크로서비스 아키텍처(MSA)**를 기반으로 설계되었습니다. 각 서비스는 독립적으로 개발, 배포, 확장될 수 있으며, 서로 API를 통해 통신합니다. 이를 통해 높은 확장성, 유지보수성, 그리고 장애 격리를 실현합니다.

### 핵심 설계 원칙
- **서비스 독립성**: 각 마이크로서비스는 독립적인 데이터베이스와 비즈니스 로직을 가짐
- **느슨한 결합**: 서비스 간 API를 통한 통신으로 의존성 최소화
- **수평적 확장**: 트래픽 증가 시 개별 서비스 단위로 확장 가능
- **장애 격리**: 한 서비스의 장애가 전체 시스템에 영향을 주지 않도록 설계

## 아키텍처 다이어그램

```
┌──────────────────────────────────────────────────────────────────┐
│                          Frontend Layer                          │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                    Web Application                         │  │
│  │              (React/TypeScript SPA)                        │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                                 │
                                 │ HTTPS/WSS
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        API Gateway / Load Balancer              │
└─────────────────────────────────────────────────────────────────┘
                                 │
                ┌────────────────┼────────────────┐
                │                │                │
                ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Backend Services                        │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ Auth Server  │  │ User Server  │  │Friend Server │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ Main Game    │  │ Match Game   │  │ Chat Server  │           │
│  │   Server     │  │   Server     │  │              │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                 │
│  ┌──────────────┐                                               │
│  │ File Server  │                                               │
│  └──────────────┘                                               │
└─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Data Layer                               │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │  PostgreSQL  │  │    Redis     │  │ File Storage │           │
│  │  Databases   │  │    Cache     │  │   (S3/Minio) │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Backend

백엔드는 **마이크로서비스 아키텍처**로 구성되어 있으며, 각 서비스는 특정 도메인의 책임을 담당합니다. 모든 백엔드 서비스는 **Fastify** 프레임워크를 기반으로 개발되었습니다.

### 🎮 게임 서버

#### 1. **Main Game Server**
- **저장소**: [main-game-server](https://github.com/42-Gang/main-game-server)
- **역할**: 실시간 탁구 게임 로직 처리
- **주요 기능**:
  - 실시간 게임 물리 엔진 (공, 패들 움직임 계산)
  - WebSocket을 통한 실시간 양방향 통신
  - 게임 상태 관리 및 동기화
  - 게임 결과 기록 및 저장
- **기술 스택**: Fastify, WebSocket, Node.js
- **통신 방식**: WebSocket (실시간), REST API

#### 2. **Match Game Server**
- **저장소**: [match-game-server](https://github.com/42-Gang/match-game-server)
- **역할**: 게임 매칭 및 토너먼트 관리
- **주요 기능**:
  - 1:1 게임 매칭 시스템
  - 토너먼트 생성 및 관리
  - 대진표 자동 생성
  - 매칭 큐 관리
  - Custom 모드 지원
- **기술 스택**: Fastify, Node.js
- **통신 방식**: REST API, WebSocket

### 👥 사용자 관련 서버

#### 3. **Auth Server**
- **저장소**: [auth-server](https://github.com/42-Gang/auth-server)
- **역할**: 인증 및 권한 관리
- **주요 기능**:
  - 42 OAuth 인증
  - JWT 토큰 발급 및 검증
  - 세션 관리
  - 권한 검증 (Authorization)
- **기술 스택**: Fastify, JWT, OAuth 2.0
- **통신 방식**: REST API
- **보안**: HTTPS, JWT, Refresh Token

#### 4. **User Server**
- **저장소**: [user-server](https://github.com/42-Gang/user-server)
- **역할**: 사용자 정보 관리
- **주요 기능**:
  - 사용자 프로필 CRUD
  - 게임 전적 조회
  - 사용자 통계 관리
  - 닉네임 변경
  - 아바타 이미지 관리
- **기술 스택**: Fastify, PostgreSQL
- **통신 방식**: REST API

#### 5. **Friend Server**
- **저장소**: [friend-server](https://github.com/42-Gang/friend-server)
- **역할**: 친구 관계 관리
- **주요 기능**:
  - 친구 추가/삭제
  - 친구 요청 관리
  - 친구 목록 조회
  - 온라인 상태 확인
  - 차단 기능
- **기술 스택**: Fastify, PostgreSQL
- **통신 방식**: REST API

### 💬 커뮤니케이션

#### 6. **Chat Server**
- **저장소**: [chat-server](https://github.com/42-Gang/chat-server)
- **역할**: 실시간 채팅 기능 제공
- **주요 기능**:
  - 1:1 채팅
  - 그룹 채팅
  - 채팅방 관리
  - 메시지 히스토리
  - 실시간 메시지 전송/수신
- **기술 스택**: Fastify, WebSocket, Redis (Pub/Sub)
- **통신 방식**: WebSocket (실시간), REST API

### 📁 파일 관리

#### 7. **File Server**
- **저장소**: [file-server](https://github.com/42-Gang/file-server)
- **역할**: 파일 업로드 및 저장 관리
- **주요 기능**:
  - 이미지 업로드 (프로필 사진, 아바타)
  - 파일 저장 및 관리
  - 이미지 최적화 및 리사이징
  - CDN 연동
- **기술 스택**: Fastify, S3/MinIO
- **통신 방식**: REST API, Multipart Upload

### 📡 서비스 간 통신

- **동기 통신**: REST API (HTTP/HTTPS)
  - 요청-응답 패턴이 필요한 경우
  - 서비스 간 데이터 조회 및 업데이트
  
- **비동기 통신**: WebSocket, Redis Pub/Sub
  - 실시간 이벤트 전달이 필요한 경우
  - 게임 상태 동기화, 채팅 메시지 브로드캐스팅

### 🗄️ 데이터베이스 전략

- **PostgreSQL**: 각 서비스별 독립적인 데이터베이스 인스턴스
  - Auth DB: 사용자 인증 정보
  - User DB: 사용자 프로필 및 통계
  - Game DB: 게임 기록 및 결과
  - Chat DB: 채팅 메시지 히스토리
  - Friend DB: 친구 관계 정보

- **Redis**: 캐시 및 세션 관리
  - 세션 스토어
  - 게임 매칭 큐
  - 실시간 데이터 캐시
  - Pub/Sub 메시징

### 🔒 보안

- **인증 플로우**:
  1. 사용자 → Auth Server (42 OAuth)
  2. Auth Server → JWT 토큰 발급
  3. 클라이언트 → 각 서비스 (JWT 포함)
  4. 각 서비스 → Auth Server (토큰 검증)

- **API 보안**:
  - HTTPS 강제 사용
  - CORS 정책 적용
  - Rate Limiting
  - Input Validation

---

## Frontend

프론트엔드는 **Single Page Application (SPA)** 방식으로 구현되어 있으며, 모던한 웹 기술을 활용하여 사용자 친화적인 인터페이스를 제공합니다.

### 🌐 Web Application

- **저장소**: [web](https://github.com/42-Gang/web)
- **역할**: 사용자 인터페이스 제공
- **기술 스택**:
  - **React**: UI 컴포넌트 기반 개발
  - **TypeScript**: 타입 안정성 확보
  - **React Router**: 클라이언트 사이드 라우팅
  - **WebSocket**: 실시간 통신
  - **Canvas API**: 게임 화면 렌더링
  - **CSS Modules/Styled Components**: 스타일링

### 주요 기능 모듈

#### 1. **인증 모듈**
- 42 OAuth 로그인 연동
- JWT 토큰 관리 (Access/Refresh Token)
- 자동 로그인 및 세션 유지
- 로그아웃 처리

#### 2. **게임 모듈**
- Canvas 기반 실시간 게임 렌더링
- WebSocket을 통한 게임 상태 동기화
- 게임 컨트롤 (키보드 입력 처리)
- 게임 결과 표시

#### 3. **매칭/토너먼트 모듈**
- 1:1 게임 매칭 대기
- 토너먼트 생성 및 참가
- 대진표 시각화
- Custom 모드 설정

#### 4. **사용자 프로필 모듈**
- 프로필 정보 표시 및 수정
- 게임 전적 조회
- 통계 시각화 (승률, 게임 수 등)
- 아바타 이미지 업로드

#### 5. **친구 관리 모듈**
- 친구 목록 조회
- 친구 추가/삭제
- 친구 요청 관리
- 온라인 상태 표시

#### 6. **채팅 모듈**
- 실시간 채팅 UI
- 1:1 및 그룹 채팅 지원
- 메시지 히스토리 로드
- 알림 기능

### 상태 관리

- **전역 상태**: Context API/Redux (선택적)
  - 사용자 인증 상태
  - 현재 사용자 정보
  - WebSocket 연결 상태

- **로컬 상태**: React Hooks (useState, useReducer)
  - UI 컴포넌트 상태
  - 폼 입력 상태

### API 통신

- **REST API**: Axios/Fetch를 통한 HTTP 통신
  - 사용자 정보 조회/수정
  - 친구 관리
  - 게임 기록 조회

- **WebSocket**: 실시간 양방향 통신
  - 게임 플레이
  - 실시간 채팅
  - 알림 수신

### 성능 최적화

- **코드 스플리팅**: React.lazy를 통한 라우트별 번들 분리
- **이미지 최적화**: Lazy Loading, WebP 포맷 사용
- **메모이제이션**: React.memo, useMemo, useCallback 활용
- **Virtual Scrolling**: 긴 목록 렌더링 최적화

---

## Infra

인프라는 **DevOps** 방법론을 기반으로 자동화된 배포 파이프라인과 모니터링 시스템을 구축하여 안정적인 서비스 운영을 지원합니다.

### 🛠️ DevOps 및 인프라

#### 1. **DevOps Infrastructure**
- **저장소**: [devops-infra](https://github.com/42-Gang/devops-infra)
- **역할**: 인프라 설정 및 CI/CD 파이프라인 관리
- **주요 구성**:
  - Docker 컨테이너화
  - Kubernetes 오케스트레이션 (선택적)
  - CI/CD 파이프라인 (GitHub Actions)
  - Infrastructure as Code (IaC)
  - 모니터링 및 로깅 시스템

### 🐳 컨테이너화 및 오케스트레이션

#### Docker
- 모든 백엔드 서비스는 Docker 컨테이너로 패키징
- Multi-stage 빌드를 통한 이미지 최적화
- Docker Compose를 통한 로컬 개발 환경 구성

```yaml
# 예시 구조
services:
  auth-server:
    image: 42gang/auth-server
    ports:
      - "3001:3000"
  user-server:
    image: 42gang/user-server
    ports:
      - "3002:3000"
  # ... 기타 서비스들
```

#### Kubernetes (선택적)
- Pod 단위 서비스 배포
- Service를 통한 내부 통신
- Ingress를 통한 외부 접근 제어
- Horizontal Pod Autoscaler (HPA)를 통한 자동 확장

### 🔄 CI/CD 파이프라인

#### GitHub Actions
```yaml
# 배포 플로우
1. 코드 Push → GitHub Repository
2. GitHub Actions Trigger
3. 테스트 실행 (Unit/Integration Tests)
4. 린팅 및 코드 품질 검사
5. Docker 이미지 빌드
6. 이미지 레지스트리에 Push (Docker Hub/ECR)
7. 배포 환경에 자동 배포 (Dev/Staging/Production)
8. 헬스 체크 및 롤백 (실패 시)
```

#### 배포 전략
- **Rolling Update**: 무중단 배포를 위한 점진적 업데이트
- **Blue-Green Deployment**: 새 버전 검증 후 트래픽 전환
- **Canary Release**: 일부 트래픽으로 먼저 테스트 후 전체 배포

### 📊 모니터링 및 로깅

#### 모니터링 스택
- **Prometheus**: 메트릭 수집 및 저장
  - CPU, 메모리, 네트워크 사용량
  - 애플리케이션 레벨 메트릭 (요청 수, 응답 시간, 에러율)
  - 비즈니스 메트릭 (동시 접속자, 게임 수)

- **Grafana**: 메트릭 시각화 대시보드
  - 실시간 모니터링 대시보드
  - 알림 설정
  - 트렌드 분석

#### 로깅 스택
- **ELK Stack** (Elasticsearch, Logstash, Kibana) 또는 **EFK Stack** (Fluentd)
  - 중앙 집중식 로그 수집
  - 로그 검색 및 분석
  - 에러 추적 및 디버깅

#### APM (Application Performance Monitoring)
- 애플리케이션 성능 추적
- 트랜잭션 추적
- 병목 지점 식별

### 🔐 보안 및 네트워크

#### 네트워크 구성
- **VPC (Virtual Private Cloud)**: 격리된 네트워크 환경
- **서브넷 분리**:
  - Public Subnet: Load Balancer, Bastion Host
  - Private Subnet: Application Servers, Databases
- **Security Groups**: 포트 및 접근 제어
- **NAT Gateway**: Private Subnet의 외부 통신

#### SSL/TLS
- Let's Encrypt를 통한 무료 SSL 인증서
- HTTPS 강제 리다이렉션
- WebSocket Secure (WSS) 연결

#### Secrets 관리
- 환경 변수를 통한 민감 정보 관리
- AWS Secrets Manager / HashiCorp Vault (선택적)
- GitHub Secrets를 통한 CI/CD 자격증명 관리

### 📦 개발 도구 및 보일러플레이트

#### 2. **Fastify Boilerplate**
- **저장소**: [fastify-boilerplate](https://github.com/42-Gang/fastify-boilerplate)
- **역할**: Fastify 기반 서버 개발 템플릿
- **포함 기능**:
  - 프로젝트 구조 템플릿
  - 기본 미들웨어 설정
  - 에러 핸들링
  - 로깅 설정
  - 환경 변수 관리

#### 3. **Fastify Boilerplate v2**
- **저장소**: [fastify-boilerplate2](https://github.com/42-Gang/fastify-boilerplate2)
- **역할**: 개선된 Fastify 보일러플레이트
- **개선 사항**:
  - TypeScript 지원 강화
  - 플러그인 시스템 개선
  - 테스트 환경 구성
  - Docker 및 CI/CD 통합

### 🌍 배포 환경

#### Development (개발)
- 로컬 개발 환경
- Docker Compose 기반
- Hot Reload 지원

#### Staging (스테이징)
- 프로덕션과 동일한 구성
- 배포 전 최종 테스트
- QA 및 통합 테스트 수행

#### Production (프로덕션)
- 실제 서비스 운영 환경
- 고가용성 (HA) 구성
- 자동 스케일링
- 정기 백업 및 재해 복구 계획

### 📈 확장성 전략

#### 수평적 확장 (Horizontal Scaling)
- 서비스 인스턴스 수 증가
- Load Balancer를 통한 트래픽 분산
- Stateless 설계를 통한 확장 용이성

#### 수직적 확장 (Vertical Scaling)
- 서버 리소스 증가 (CPU, 메모리)
- 데이터베이스 성능 업그레이드

#### 데이터베이스 확장
- Read Replica를 통한 읽기 성능 향상
- Sharding을 통한 데이터 분산 (필요시)
- Connection Pool 최적화

### 🔄 백업 및 재해 복구

- **데이터베이스 백업**:
  - 자동 일일 백업
  - Point-in-Time Recovery (PITR)
  - 다중 지역 백업 저장

- **재해 복구 계획**:
  - RTO (Recovery Time Objective) 정의
  - RPO (Recovery Point Objective) 정의
  - 정기적인 복구 훈련

---

## 🔗 관련 문서

- [프로젝트 Wiki](https://github.com/42-Gang/project-wiki/wiki)
- [Notion 문서](https://www.notion.so/clearcat/42-Gang-19e04e9b8dc28027bdd1f4876a997ebb)
- [Figma 디자인](https://www.figma.com/design/whPva7y2FoXdIilDSAfsvA/42-Gang)

---

## 📝 기여 가이드

각 레포지토리의 CONTRIBUTING.md를 참고하여 프로젝트에 기여해주세요.

## 📄 라이센스

각 레포지토리의 LICENSE 파일을 참고해주세요.
