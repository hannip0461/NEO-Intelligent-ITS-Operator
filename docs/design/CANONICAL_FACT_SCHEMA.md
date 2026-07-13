# Canonical Fact Schema

## 1. Purpose

This document defines the canonical fact schema used between the Detector / Adapter layer and the NEO Reasoning Core.

The schema standardizes raw observations before they are converted into NEO-readable facts. It also preserves the information required for CF, ATMS, priority arbitration, match trace, Neo4j provenance, NEMI evidence retrieval, and LLM explanation.

## 2. Scope

This document covers:

- observation output from Detector / Adapter modules
- canonical fact fields
- NEO-readable fact projection
- confidence and quality fields
- source and provenance fields
- ATMS assumption mapping
- Neo4j and NEMI linkage keys

This document does not define:

- final decision schema
- final priority schema
- recommended action schema
- LLM response schema
- Vue screen layout

Those are handled by the Decision Package and UI/API design documents.

## 3. Core Principle

Detector / Adapter modules must produce observations only.

They must not produce:

```text
final decision
operator priority
unified priority
recommended action
NEO decision
```

Only NEO may create final decision authority values.

## 4. Canonical Fact Shape

Each canonical fact is represented as a structured object before being projected into NEO facts.

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

## 5. Required Fields

| Field | Required | Purpose |
|---|---:|---|
| `fact_id` | Yes | Stable fact identifier |
| `event_id` | Yes | Groups facts belonging to the same event |
| `source.source_id` | Yes | Source identity |
| `source.source_type` | Yes | Source class such as camera, sensor, api, operator |
| `source.source_reliability` | Yes | CF input |
| `timestamp` | Yes | Time of observation |
| `domain` | Yes | Domain such as its, maas, maintenance |
| `asset.asset_id` | Yes | Asset or service identity |
| `observation.observation_type` | Yes | Normalized observation type |
| `observation.symptom` | Yes | Rule-facing symptom key |
| `observation.value` | Yes | Observed value |
| `observation.confidence` | Yes | Detector or adapter confidence |
| `freshness.status` | Yes | fresh, stale, unknown |
| `quality.quality_flags` | Yes | Quality warnings and guard inputs |
| `provenance.raw_ref` | Yes | Link to original source data |
| `provenance.adapter_id` | Yes | Adapter that produced the fact |
| `neo.assumption_id` | Yes | ATMS assumption key |

`provenance.raw_ref` and `provenance.checksum` are preserved as adapter/source
metadata for Decision Package and Neo4j lineage. They are not projected as NEO
reasoning facts, and the current NEO engine does not verify checksum integrity.

## 6. Enumerations

### 6.1 Domain

```text
its
maas
maintenance
time-series
vision
network
operator
```

### 6.2 Source Type

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

### 6.3 Freshness Status

```text
fresh
stale
unknown
```

### 6.4 Quality Flags

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

### 6.5 Fact Group

```text
incident_observation
maintenance_observation
asset_state
source_quality
operator_report
guard_condition
```

## 7. Confidence Policy

Canonical facts must preserve the confidence components needed by NEO CF propagation.

Recommended fields:

```text
observation.confidence
source.source_reliability
freshness.status
quality.quality_flags
neo.cf_hint
```

`neo.cf_hint` may be computed before NEO as a normalized input hint, but it must not be treated as final decision confidence.

Example:

```text
detector confidence = 0.84
source reliability = 0.90
freshness factor = 1.00
cf_hint = 0.756
```

NEO may use this hint in rule-side CF propagation. Final decision confidence belongs in the Decision Package.

## 8. ATMS Assumption Mapping

Each source-backed canonical fact must map to an ATMS assumption.

Assumption id format:

```text
asm_{event_id}_{source_id}_{observation_type}
```

Example:

```text
asm_evt_0001_cctv_17_lane_blocked
asm_evt_0001_public_its_lane_blocked
asm_evt_0001_operator_report_lane_blocked
```

This allows NEO ATMS to explain which assumptions support a derived decision.

Quality or conflict conditions may become nogood candidates.

Examples:

```text
stale_data(cctv_17)
conflicting_source(cctv_17, public_its)
low_confidence(yolo_lane_blocked)
```

For time-series predictive maintenance, a corroboration conflict may invalidate one support set without removing independent support.

Example:

```text
support_pm_timeseries_primary -> invalid
support_pm_timeseries_persistence -> valid
nogood_set -> support_pm_timeseries_primary
conflict_type -> cross_sensor_disagreement
effect -> requires_validation
```

The invalidation result belongs to the NEO Decision Package. The Detector and Adapter only set the source-backed conflict observation.

### 8.1 Time-Series Observation Set

A time-series Detector may emit the following observational facts:

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

These fields describe model output. They must not contain the final NEO risk class, priority, decision, or action.

## 9. NEO Fact Projection

Canonical facts should be projected into simple NEO-readable facts.

Example canonical observation:

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

Projected NEO facts:

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

When a rule needs a direct predicate, the wrapper may project `observation.observation_type` and `observation.value` into a rule-facing NEO fact.

Example:

```text
observation.observation_type = road_status
observation.value = lane_blocked
projected fact = (evt_0001 road_status lane_blocked)
```

This projection remains observational only. It must not create or pass through final decision authority predicates.

## 10. Neo4j Linkage

Canonical facts must provide stable identifiers for graph persistence.

Recommended mapping:

```text
event_id -> Event node
fact_id -> Fact node
source.source_id -> Source node
asset.asset_id -> Asset node
neo.assumption_id -> Assumption node
provenance.raw_ref -> SourceData node or property
```

The graph layer must store and expose provenance paths. It must not create decisions.

## 11. NEMI Linkage

Canonical facts should preserve document retrieval keys.

Recommended keys:

```text
domain
observation.observation_type
observation.symptom
asset.asset_type
asset.location_id
source.source_type
quality.quality_flags
```

NEMI may use these fields with the later NEO Decision Package fields to retrieve relevant runbooks, policies, and past cases.

## 12. Adapter Output Contract

Each adapter must return an array of canonical facts.

```json
{
  "adapter_id": "adapter_yolo_lane",
  "adapter_version": "v1",
  "event_id": "evt_0001",
  "facts": []
}
```

Adapter output must be rejected or marked invalid if it contains final decision fields such as:

```text
neo_decision
operator_priority
unified_priority
recommended_actions
```

## 13. Minimal Valid Fact

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

## 14. Acceptance Criteria

The schema is accepted when:

- every adapter output can be represented as canonical facts
- no adapter output contains final decision authority fields
- each fact has source, time, confidence, freshness, quality, and provenance fields
- each source-backed fact has an ATMS assumption id
- each confidence-bearing fact can produce or carry a CF hint
- each fact can be projected into NEO-readable facts
- each fact can be linked to Neo4j provenance nodes
- each fact carries enough keys for NEMI retrieval

## 15. Next Step

After this schema is fixed, the next design document is:

```text
docs/design/DECISION_PACKAGE_SCHEMA.md
```

The Decision Package defines the standard output produced after NEO reasoning.
