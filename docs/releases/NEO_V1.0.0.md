# NEO v1.0.0 릴리즈 기록

- 배포일: 2026-08-02
- 상태: GitHub, Docker Hub, AWS 배포 및 검증 완료
- 서비스 주소: `https://3-38-33-156.sslip.io/neo`

## 릴리즈 범위

- NEO Rule Engine, ATMS, CF 기반 판단
- FastAPI 오케스트레이션
- Neo4j Graph RAG 계보 저장 및 조회
- NEMI Vector RAG 근거 검색
- Amazon Bedrock Nova 2 Lite 판단 설명
- Vue 관제 화면 6개

공개 저장소는 운영 화면과 설계·배포 문서를 제공하며, 핵심 규칙 엔진과 Rule KB는 별도로 관리한다.

## Docker Hub 이미지

| 구성요소 | 이미지 |
| --- | --- |
| Vue 관제 화면 | `kimmj6466/neo-operator-frontend:v1.0.0` |
| NEO FastAPI | `kimmj6466/neo-operator-api:v1.0.0` |
| NEMI API | `kimmj6466/neo-nemi-api:v1.0.0` |

AWS EC2는 위 v1.0.0 이미지를 배포 설정에 고정해 사용한다.

## 검증 결과

| 항목 | 결과 |
| --- | --- |
| Python 단위 테스트 | 218개 통과 |
| NEO Vue 프로덕션 빌드 | 통과 |
| 엔진 E2E 스모크 | 통과 |
| FastAPI 스모크 | 통과 |
| Vue 클라이언트 스모크 | 통과 |
| 공개 화면 6개 | 모두 HTTP 200 |
| 공개 실시간 파이프라인 | 완료 |
| NEO 판단 | `requires_operator_ack`, CF `0.85` |
| Neo4j 동기화 | 20노드, 28관계 |
| NEMI 검색 | 문서 2건 |
| Bedrock 설명 | Nova 2 Lite 응답 성공, 1,305ms |
| 배포 직후 오류 로그 | 0건 |

## 배포 안전성

- API 키는 프론트 Nginx의 내부 프록시 계층에서 주입한다.
- FastAPI, Neo4j, NEMI, Qdrant 포트는 EC2 루프백에만 바인딩한다.
- 설명 모델의 역할은 NEO 판단의 읽기 전용 설명으로 제한한다.
- 외부 조치는 운영자 승인 후 송출한다.
