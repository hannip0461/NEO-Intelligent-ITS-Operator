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

핵심 규칙 엔진과 Rule KB는 비공개 자산이며, 공개 저장소에는 화면과 설계·배포 문서만 포함한다.

## Docker Hub 이미지

| 구성요소 | 이미지 | 고정 다이제스트 |
| --- | --- | --- |
| Vue 관제 화면 | `kimmj6466/neo-operator-frontend:v1.0.0` | `sha256:7b36a2fc1fff65ea77e97af65dddf3033853f7126deda1140ec3f64bbb7bc599` |
| NEO FastAPI | `kimmj6466/neo-operator-api:v1.0.0` | `sha256:0eb7b7ff10ead48e14ef45c63a3fe275eefa3312d2c21c335c312f421320bbfc` |
| NEMI API | `kimmj6466/neo-nemi-api:v1.0.0` | `sha256:587045efd529d699c09eb4e0e6cd479d48d1a9238539eab52cb09d3ff6940f99` |

AWS EC2는 위 다이제스트를 직접 참조하므로 Docker Hub 태그가 이후 변경돼도 v1 실행본은 바뀌지 않는다.

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

- 브라우저에는 API 키를 포함하지 않고 프론트 Nginx가 내부 프록시 요청에만 키를 주입한다.
- FastAPI, Neo4j, NEMI, Qdrant 포트는 EC2 루프백에만 바인딩한다.
- 설명 모델은 NEO 판단을 변경할 수 없고 읽기 전용 설명만 제공한다.
- 운영자 승인 전에는 외부 조치를 자동 송출하지 않는다.

## 다음 릴리즈

v2에서는 감시 화면의 정보 위계, 계보 탐색, 로딩·오류 복구, 이력 가독성과 전체 반응형 흐름을 개선한다.
