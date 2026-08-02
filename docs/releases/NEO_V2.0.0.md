# NEO v2.0.0 릴리즈 기록

- 배포일: 2026-08-03
- 상태: GitHub, Docker Hub, AWS 배포 및 검증 완료
- 서비스: `https://3-38-33-156.sslip.io/neo`

## 주요 변경

- 감시 화면을 사건, 판단 근거와 현재 조치 중심으로 재구성했습니다.
- 보류와 오탐 처리 전에 운영자 확인과 사유 입력을 거치고, 처리 결과와 감사 ID를 기록하도록 보강했습니다.
- Neo4j 계보의 기본 화면을 전체 그래프로 변경하고, 노드 선택과 판단 경로 전환 뒤에도 그래프 위치와 복귀 흐름을 유지했습니다.
- 예지 및 이상 분석에 단계별 진행 상태, 측정 수치, 허용 범위와 결과 우선순위를 적용했습니다.
- 판단 설명과 예측 요청에 시간초과, 취소와 재시도 상태를 추가했습니다.
- LSTM 요청별 재학습을 제거하고 서비스 기동 시 준비한 모델로 추론하도록 변경했습니다.
- 이력, 상태와 정책 화면의 정보 밀도, 긴 식별자와 상세 확인 흐름을 정리했습니다.

공개 저장소에는 운영 화면과 설계·배포 문서를 담았습니다. NEO 규칙 엔진과 Rule KB는 별도 비공개 저장소에서 관리합니다.

## 검증 결과

- Python 단위 테스트 219개와 NEO Vue 프로덕션 빌드를 통과했습니다.
- 공개 관제, 이력, 예지 및 이상 분석, 계보, 상태와 정책 화면의 HTTP 200 응답을 확인했습니다.
- PC와 모바일 6개 화면에서 가로 넘침과 콘솔 오류가 없음을 확인했습니다.
- 운영자 조치 취소 시 기록이 생성되지 않고, 사유 입력 후 감사 기록 1건이 생성되는 흐름을 확인했습니다.
- 공개 파이프라인은 검증 요청에서 3.15초에 완료됐으며, NEO 판단 `requires_operator_ack`, Neo4j 20노드와 28관계, NEMI 문서 2건을 반환했습니다.
- 배포 후 로그에서 Traceback, uncaught, fatal, panic 패턴이 없음을 확인했습니다.

## 적용 환경

- AWS EC2 Ubuntu에서 Vue, FastAPI, NEMI, Neo4j와 Qdrant를 Docker Compose로 실행합니다.
- 프론트엔드, NEO API와 NEMI API는 Docker Hub의 `v2.0.0` 이미지를 사용합니다.
- Neo4j, Qdrant와 운영자 조치 데이터 볼륨을 유지한 상태로 애플리케이션 컨테이너만 교체했습니다.
- 운영 화면은 [AWS NEO 관제 화면](https://3-38-33-156.sslip.io/neo)에서 확인할 수 있습니다.

## 배포 식별 정보

- `kimmj6466/neo-operator-frontend:v2.0.0` - `sha256:7e0da42b2f7f9773ed083f46be80945f04ec68cdd0e5be697f4bb82ddca37b03`
- `kimmj6466/neo-operator-api:v2.0.0` - `sha256:0a3fb42d56bb7c83d1b2440e6825f172b8672dd330d4aefd158da34d05d19369`
- `kimmj6466/neo-nemi-api:v2.0.0` - `sha256:b24e57e15d7e88d7d3350b7e180df92293e9a15cef3e8815a40aa25e2489b02c`

AWS EC2는 위 이미지 다이제스트를 직접 참조합니다.

## 롤백 기준

- 이전 이미지는 `v1.0.0` 고정 다이제스트 3종입니다.
- 이전 서버 설정은 `~/neo-release-backups/pre-v2.0.0-20260802-201637`에 보관했습니다.
- Neo4j, Qdrant와 운영자 조치 데이터 볼륨은 배포 중 삭제하지 않았습니다.

## 변경 기록

- [v2.0.0 운영 UI 개선](https://github.com/hannip0461/NEO-Intelligent-ITS-Operator/commit/3f9dd11)
- [AWS EC2 배포 기록](https://github.com/hannip0461/NEO-Intelligent-ITS-Operator/blob/main/docs/deployment/AWS_EC2_DEPLOYMENT.md)
