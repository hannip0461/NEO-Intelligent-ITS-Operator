# Canonical Fact 표준 구조

## 1. 목적

이 문서는 감지기(Detector) / 어댑터(Adapter) 계층과 NEO 추론 코어 사이에서 사용하는 Canonical Fact 표준 구조를 정의한다.

이 구조는 원시 관측값을 NEO가 읽을 수 있는 Fact로 변환하기 전에 표준화한다. 또한 CF, ATMS, 우선순위 조정, 규칙 매칭 이력, Neo4j 출처 계보, NEMI 근거 검색과 설명 모델에 필요한 정보를 보존한다.

## 2. 범위

이 문서가 다루는 범위는 다음과 같다.

- 감지기 / 어댑터 모듈의 관측 출력
- Canonical Fact 필드
- NEO가 읽을 수 있는 Fact로의 변환
- 신뢰도와 품질 필드
- 소스와 출처 필드
- ATMS 가정 매핑
- Neo4j와 NEMI 연결 키

다음 항목은 이 문서에서 정의하지 않는다.

- 최종 판단 구조
- 최종 우선순위 구조
- 권고 조치 구조
- 설명 모델 응답 구조
- Vue 화면 배치

이 항목은 Decision Package와 UI/API 설계 문서에서 정의한다.

## 3. 핵심 원칙

감지기와 어댑터 모듈은 관측값만 생성해야 한다.

다음 값은 생성하면 안 된다.

```text
final decision
operator priority
unified priority
recommended action
NEO decision
```

최종 판단 권한에 해당하는 값은 NEO만 생성할 수 있다.

## 4. Canonical Fact 구조

각 Canonical Fact는 NEO Fact로 변환되기 전에 구조화된 객체로 표현한다.

```json
{
  "fact_id": "fact_evt_0001_symptom",
  "event_id": "evt_0001",
  "source": {
    "source_id": "cctv_17",
    "source_type": "camera",
    "source_name": "CCTV 17",
    "source_reliability": 0.9
  },
  "timestamp": "2026-06-10T09:15:30+09:00",
  "domain": "its",
  "asset": {
    "asset_id": "camera_17",
    "asset_type": "cctv",
    "location_id": "road_link_12"
  },
  "observation": {
    "observation_type": "lane_blocked",
    "symptom": "lane_blocked",
    "value": true,
    "severity_score": 8,
    "confidence": 0.84
  },
  "freshness": {
    "status": "fresh",
    "age_seconds": 12,
    "observation_window_seconds": 60
  },
  "quality": {
    "quality_flags": [],
    "is_stale": false,
    "is_low_confidence": false,
    "is_conflicting": false
  },
  "provenance": {
    "raw_ref": "raw://its/cctv_17/20260610T091530",
    "adapter_id": "adapter_yolo_lane",
    "adapter_version": "v1",
    "checksum": "sha256:..."
  },
  "neo": {
    "fact_group": "incident_observation",
    "assumption_id": "asm_evt_0001_cctv_17_lane_blocked",
    "cf_hint": 0.756
  }
}
```

## 5. 필수 필드

| 필드 | 필수 | 목적 |
|---|---:|---|
| `fact_id` | 예 | 안정적인 Fact 식별자 |
| `event_id` | 예 | 같은 사건에 속한 Fact를 묶는 식별자 |
| `source.source_id` | 예 | 입력 소스 식별자 |
| `source.source_type` | 예 | 카메라, 센서, API, 운영자 등의 소스 유형 |
| `source.source_reliability` | 예 | CF 입력값 |
| `timestamp` | 예 | 관측 시각 |
| `domain` | 예 | ITS, MaaS, 유지보수 등의 업무 영역 |
| `asset.asset_id` | 예 | 자산 또는 서비스 식별자 |
| `observation.observation_type` | 예 | 정규화된 관측 유형 |
| `observation.symptom` | 예 | 규칙에서 사용하는 증상 키 |
| `observation.value` | 예 | 관측값 |
| `observation.confidence` | 예 | 감지기 또는 어댑터 신뢰도 |
| `freshness.status` | 예 | `fresh`, `stale`, `unknown` |
| `quality.quality_flags` | 예 | 품질 경고와 조건 검사 입력 |
| `provenance.raw_ref` | 예 | 원본 소스 자료 연결값 |
| `provenance.adapter_id` | 예 | Fact를 생성한 어댑터 |
| `neo.assumption_id` | 예 | ATMS 가정 키 |

`provenance.raw_ref`와 `provenance.checksum`은 Decision Package와 Neo4j 계보에서 사용할 어댑터·소스 메타데이터로 보존한다. 이 값은 NEO 추론 Fact로 변환하지 않으며, 현재 NEO 엔진은 체크섬 무결성을 직접 검증하지 않는다.

## 6. 열거형

### 6.1 업무 영역

```text
its
maas
maintenance
time-series
vision
network
operator
```

### 6.2 소스 유형

```text
camera
ocr
yolo
public_its
maas_log
sensor
network
operator
system
```

### 6.3 최신성 상태

```text
fresh
stale
unknown
```

### 6.4 품질 플래그

```text
low_confidence
stale_data
missing_source
conflicting_source
partial_observation
manual_override
model_uncertain
sensor_conflict
```

### 6.5 Fact 그룹

```text
incident_observation
maintenance_observation
asset_state
source_quality
operator_report
guard_condition
```

## 7. 신뢰도 정책

Canonical Fact는 NEO의 CF 전파에 필요한 신뢰도 구성값을 보존해야 한다.

권장 필드는 다음과 같다.

```text
observation.confidence
source.source_reliability
freshness.status
quality.quality_flags
neo.cf_hint
```

`neo.cf_hint`는 NEO 실행 전에 정규화된 입력 참고값으로 계산할 수 있지만, 최종 판단 신뢰도로 취급하면 안 된다.

예시:

```text
detector confidence = 0.84
source reliability = 0.90
freshness factor = 1.00
cf_hint = 0.756
```

NEO는 규칙의 CF 전파 과정에서 이 참고값을 사용할 수 있다. 최종 판단 신뢰도는 Decision Package에 기록한다.

## 8. ATMS 가정 매핑

소스 근거가 있는 각 Canonical Fact는 ATMS 가정과 연결해야 한다.

가정 식별자 형식:

```text
asm_{event_id}_{source_id}_{observation_type}
```

예시:

```text
asm_evt_0001_cctv_17_lane_blocked
asm_evt_0001_public_its_lane_blocked
asm_evt_0001_operator_report_lane_blocked
```

이를 통해 NEO ATMS는 어떤 가정이 파생 판단을 지지하는지 설명할 수 있다.

품질 문제나 충돌 조건은 nogood 후보가 될 수 있다.

예시:

```text
stale_data(cctv_17)
conflicting_source(cctv_17, public_its)
low_confidence(yolo_lane_blocked)
```

시계열 예지보전에서는 교차 확인 충돌이 독립 근거를 제거하지 않으면서 특정 Support Set만 무효화할 수 있다.

예시:

```text
support_pm_timeseries_primary -> invalid
support_pm_timeseries_persistence -> valid
nogood_set -> support_pm_timeseries_primary
conflict_type -> cross_sensor_disagreement
effect -> requires_validation
```

무효화 결과는 NEO Decision Package에 기록한다. 감지기와 어댑터는 소스 근거가 있는 충돌 관측값만 설정한다.

### 8.1 시계열 관측값 집합

시계열 감지기는 다음 관측 Fact를 출력할 수 있다.

```text
intent = predictive_maintenance
detector_model = numpy_lstm_predictor
anomaly_level = normal | warning | critical
anomaly_ratio = numeric residual ratio
signal_trend = stable | worsening
threshold_breach_count = integer
predicted_issue = no_issue_detected | sensor_degradation
sensor_corroboration = confirmed | conflicting
```

이 필드는 모델 출력을 설명한다. 최종 NEO 위험 등급, 우선순위, 판단 또는 조치를 포함하면 안 된다.

## 9. NEO Fact 변환

Canonical Fact는 NEO가 읽을 수 있는 단순한 Fact로 변환해야 한다.

Canonical 관측값 예시:

```json
{
  "event_id": "evt_0001",
  "source": {
    "source_id": "cctv_17",
    "source_reliability": 0.9
  },
  "domain": "its",
  "observation": {
    "symptom": "lane_blocked",
    "severity_score": 8,
    "confidence": 0.84
  },
  "freshness": {
    "status": "fresh"
  },
  "neo": {
    "assumption_id": "asm_evt_0001_cctv_17_lane_blocked",
    "cf_hint": 0.756
  }
}
```

변환된 NEO Fact:

```lisp
(evt_0001 domain its)
(evt_0001 symptom lane_blocked)
(evt_0001 lane_blocked true)
(evt_0001 severity_score 8)
(evt_0001 source cctv_17)
(evt_0001 source_reliability 0.90)
(evt_0001 detector_confidence 0.84)
(evt_0001 freshness fresh)
(evt_0001 cf_hint 0.756)
(evt_0001 assumption asm_evt_0001_cctv_17_lane_blocked)
```

규칙에 직접 사용할 술어가 필요하면 래퍼는 `observation.observation_type`과 `observation.value`를 규칙용 NEO Fact로 변환할 수 있다.

예시:

```text
observation.observation_type = road_status
observation.value = lane_blocked
projected fact = (evt_0001 road_status lane_blocked)
```

이 변환 결과는 관측 정보로만 유지해야 한다. 최종 판단 권한 술어를 생성하거나 그대로 전달하면 안 된다.

## 10. Neo4j 연결

Canonical Fact는 그래프 저장에 사용할 안정적인 식별자를 제공해야 한다.

권장 매핑:

```text
event_id -> Event node
fact_id -> Fact node
source.source_id -> Source node
asset.asset_id -> Asset node
neo.assumption_id -> Assumption node
provenance.raw_ref -> SourceData node or property
```

그래프 계층은 출처 계보 경로를 저장하고 제공하되 판단을 생성하면 안 된다.

## 11. NEMI 연결

Canonical Fact는 문서 검색 키를 보존해야 한다.

권장 키:

```text
domain
observation.observation_type
observation.symptom
asset.asset_type
asset.location_id
source.source_type
quality.quality_flags
```

NEMI는 이 필드와 이후 생성되는 NEO Decision Package 필드를 함께 사용해 관련 실행 절차, 정책과 과거 사례를 검색할 수 있다.

## 12. 어댑터 출력 계약

각 어댑터는 Canonical Fact 배열을 반환해야 한다.

```json
{
  "adapter_id": "adapter_yolo_lane",
  "adapter_version": "v1",
  "event_id": "evt_0001",
  "facts": []
}
```

어댑터 출력에 다음과 같은 최종 판단 필드가 포함되면 요청을 거부하거나 유효하지 않은 값으로 표시해야 한다.

```text
neo_decision
operator_priority
unified_priority
recommended_actions
```

## 13. 최소 유효 Fact

```json
{
  "fact_id": "fact_evt_0001_001",
  "event_id": "evt_0001",
  "source": {
    "source_id": "public_its_api",
    "source_type": "public_its",
    "source_reliability": 0.85
  },
  "timestamp": "2026-06-10T09:15:30+09:00",
  "domain": "its",
  "asset": {
    "asset_id": "road_link_12",
    "asset_type": "road_link",
    "location_id": "road_link_12"
  },
  "observation": {
    "observation_type": "lane_blocked",
    "symptom": "lane_blocked",
    "value": true,
    "severity_score": 8,
    "confidence": 0.85
  },
  "freshness": {
    "status": "fresh",
    "age_seconds": 30,
    "observation_window_seconds": 300
  },
  "quality": {
    "quality_flags": [],
    "is_stale": false,
    "is_low_confidence": false,
    "is_conflicting": false
  },
  "provenance": {
    "raw_ref": "raw://public_its/event/evt_0001",
    "adapter_id": "adapter_public_its",
    "adapter_version": "v1",
    "checksum": "sha256:..."
  },
  "neo": {
    "fact_group": "incident_observation",
    "assumption_id": "asm_evt_0001_public_its_lane_blocked",
    "cf_hint": 0.7225
  }
}
```

## 14. 수용 기준

다음 조건을 모두 충족하면 이 표준 구조를 사용할 수 있다.

- 모든 어댑터 출력을 Canonical Fact로 표현할 수 있다.
- 어댑터 출력에 최종 판단 권한 필드가 없다.
- 각 Fact에 소스, 시각, 신뢰도, 최신성, 품질과 출처 필드가 있다.
- 소스 근거가 있는 각 Fact에 ATMS 가정 식별자가 있다.
- 신뢰도 값이 있는 각 Fact가 CF 참고값을 생성하거나 전달할 수 있다.
- 각 Fact를 NEO가 읽을 수 있는 Fact로 변환할 수 있다.
- 각 Fact를 Neo4j 출처 노드와 연결할 수 있다.
- 각 Fact가 NEMI 검색에 충분한 키를 포함한다.

## 15. 다음 단계

이 표준 구조를 확정한 뒤 사용할 다음 설계 문서는 다음과 같다.

```text
docs/design/DECISION_PACKAGE_SCHEMA.md
```

Decision Package는 NEO 추론 이후 생성되는 표준 출력을 정의한다.
