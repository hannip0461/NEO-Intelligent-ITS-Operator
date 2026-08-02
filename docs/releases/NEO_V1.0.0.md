## 주요 변경

- 교통 입력을 Canonical Fact로 정규화하고 NEO Rule Engine, ATMS, CF를 적용해 판단과 대응 가이드를 생성했습니다.
- Neo4j에 Fact, Rule, Decision 관계를 저장하고 NEMI Vector RAG로 SOP, 정책, 사고 이력 근거를 검색하도록 연결했습니다.
- 관제자가 사건 선택, 판단 재실행, 근거 검토, 조치 준비와 감사 이력을 하나의 흐름에서 확인하도록 Vue 화면 6개를 구성했습니다.
- LSTM 잔차, 통계 기준선, AI4I 참조 규칙과 C-MAPSS 참조 RUL을 NEO 판단 흐름에 연결했습니다.
- Amazon Bedrock Nova 2 Lite가 현재 판단과 연결 근거를 한국어로 설명하도록 구성했습니다.
- Docker Compose 기반 실행 환경을 AWS EC2에 배포하고 Nginx, HTTPS와 내부 API 프록시를 적용했습니다.

## 검증 결과

- Python 단위 테스트 218개를 통과했습니다.
- NEO Vue 프로덕션 빌드와 엔진 E2E, FastAPI, Vue 클라이언트 스모크 검증을 통과했습니다.
- 관제, 예지 및 이상 분석, 계보, 이력, 상태, 정책 화면의 HTTP 200 응답을 확인했습니다.
- 공개 파이프라인에서 NEO 판단 `requires_operator_ack`, CF `0.85`, Neo4j 20노드와 28관계, NEMI 문서 2건을 확인했습니다.
- Amazon Bedrock Nova 2 Lite의 한국어 설명 응답을 1,305ms에 확인했습니다.
- 배포 직후 프론트엔드, FastAPI, NEMI 컨테이너의 오류 로그가 0건임을 확인했습니다.

## 적용 환경

- AWS EC2 Ubuntu에서 Vue, FastAPI, NEMI, Neo4j와 Qdrant를 Docker Compose로 실행합니다.
- 호스트 Nginx가 HTTPS 요청을 내부 프론트엔드로 전달하고 API 키는 내부 프록시 계층에서 주입합니다.
- FastAPI, Neo4j, NEMI와 Qdrant 포트는 EC2 루프백에 바인딩합니다.
- 설명 모델은 NEO 판단의 읽기 전용 설명을 담당하고 외부 조치는 운영자 승인 후 송출합니다.
- Vue 관제 화면은 `kimmj6466/neo-operator-frontend:v1.0.0`을 사용합니다.
- NEO FastAPI는 `kimmj6466/neo-operator-api:v1.0.0`을 사용합니다.
- NEMI API는 `kimmj6466/neo-nemi-api:v1.0.0`을 사용합니다.
- 운영 데모는 [AWS NEO 관제 화면](https://3-38-33-156.sslip.io/neo)에서 확인할 수 있습니다.

## 변경 기록

- [v1.0.0 공개 릴리즈 커밋](https://github.com/hannip0461/NEO-Intelligent-ITS-Operator/commit/a20c6a47d54117d93c432d62133529dae65fb42c)
- [AWS EC2 배포 기록](https://github.com/hannip0461/NEO-Intelligent-ITS-Operator/blob/main/docs/deployment/AWS_EC2_DEPLOYMENT.md)
