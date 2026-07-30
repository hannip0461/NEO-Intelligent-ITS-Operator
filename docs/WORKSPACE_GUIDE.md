# NEO 도로 ITS 판단 시스템 작업공간 안내

## 작업공간 기준

이 문서의 모든 경로는 저장소 최상위 폴더를 기준으로 한다.

이 작업공간은 NEO 기반 도로 ITS 운영 판단 시스템의 개발 기준을 정리한다. 교통 운영 판단을 주 기능으로 다루고, 예지보전 지원은 분리된 장비 분석 영역으로 관리한다.

## 폴더 구성

```text
docs/          요구사항, 설계와 배포 문서
src/           구현 소스
tests/         테스트와 검증 스크립트
outputs/       생성 산출물과 검토 자료
runbooks/      실행 절차와 운영 참고 문서
work/          임시 작업 파일과 보조 스크립트
```

공개 저장소에는 운영 화면, 설계 계약, 작업공간 안내와 AWS 배포 기록만 포함한다. 핵심 규칙 엔진과 Rule KB는 비공개 자산으로 관리한다.

## 기준 문서

```text
docs/design/CANONICAL_FACT_SCHEMA.md
docs/design/DECISION_PACKAGE_SCHEMA.md
docs/deployment/AWS_EC2_DEPLOYMENT.md
```

- `Canonical Fact` 문서는 외부 입력을 NEO가 읽을 수 있는 표준 Fact로 정규화하는 규칙을 정의한다.
- `Decision Package` 문서는 NEO 판단, 근거, 신뢰도, 버전과 연동 정보를 기록하는 규칙을 정의한다.
- AWS 배포 문서는 EC2, Docker Compose, Nginx, HTTPS와 Bedrock Nova 2 Lite 적용 결과를 기록한다.

## 실행 입력

| 입력 | 처리 기준 |
| --- | --- |
| 공공 ITS 원시 자료 | 5분 단위 CSV 재생 결과를 교통 흐름 이상 Fact로 변환 |
| 교통 흐름 자료 파일·API | 국토교통부·ITS 행 데이터를 Canonical Fact와 검토 문맥으로 변환 |
| VMS | 장비·문안 자료를 그래프 메타데이터와 대응 정합성 근거로 사용 |
| TAAS | 사고 다발 구간 자료를 안전 위험 문맥과 운영 가이드로 사용 |

기본 시연은 재현 가능한 예시 또는 재생 입력을 사용한다. 재생 자료를 현재 도로 상태로 해석하지 않는다. 실시간 교통 흐름, VMS 또는 TAAS 상태를 표시하려면 API 인증 정보와 정상 수집 결과가 필요하다.

## 판단 설명 환경

- 로컬 또는 GPU 서버에서는 Ollama의 Gemma4를 사용한다.
- AWS 데모에서는 Amazon Bedrock의 Nova 2 Lite를 사용한다.
- 설명 모델은 현재 Decision Package, Neo4j 판단 관계와 NEMI 문서 근거만 읽는다.
- 설명 모델은 NEO가 정한 판단, 우선순위, 신뢰도와 권고 조치를 변경할 수 없다.

## 개발 순서

```text
요구사항
-> Canonical Fact 표준 구조
-> Decision Package 표준 구조
-> NEO 스키마 검증과 Fact 변환
-> NEO 실행과 출력 매핑
-> FastAPI 오케스트레이션
-> Neo4j / NEMI / 설명 모델 연동
-> Vue 운영 화면
```

## 판단 권한 원칙

```text
NEO가 판단한다.
FastAPI가 연동 흐름을 조율한다.
Vue가 결과를 표시한다.
Neo4j가 관계를 시각화한다.
NEMI가 문서 근거를 검색한다.
설명 모델이 판단을 설명한다.
```

Neo4j, NEMI, 설명 모델과 Vue는 NEO 판단을 생성하거나 변경하지 않는다.
