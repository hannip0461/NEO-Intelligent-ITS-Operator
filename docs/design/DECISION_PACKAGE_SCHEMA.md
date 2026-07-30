# Decision Package 표준 구조

## 1. 목적

이 문서는 NEO 추론 이후 생성되는 Decision Package의 표준 구조를 정의한다.

Decision Package는 FastAPI, Vue, Neo4j, NEMI와 설명 모델 계층이 함께 사용하는 단일 구조화 출력이다.

## 2. 핵심 원칙

판단 권한은 NEO에만 있다.

하위 계층은 Decision Package를 저장, 시각화, 설명하고 검색할 수 있다. 다만 다음 값을 생성하거나 덮어쓰면 안 된다.

```text
neo_decision
decision_mode
priority
unified_priority
recommended_actions
decision_confidence
```

## 3. 생성자와 사용 계층

| 역할 | 구성요소 | 책임 |
|---|---|---|
| 생성자 | NEO 추론 코어 / 래퍼 | Decision Package 생성과 정규화 |
| 오케스트레이터 | FastAPI | Decision Package 제공과 보조 데이터 연결 |
| UI 사용 계층 | Vue | 판단, 이력, 그래프, 근거와 설명 답변 표시 |
| 그래프 사용 계층 | Neo4j | 출처 계보 경로 저장과 조회 제공 |
| 근거 사용 계층 | NEMI | 문서 이력과 근거 검색 |
| 설명 사용 계층 | 설명 모델 | 패키지를 변경하지 않고 설명 |

## 4. 최상위 구조

```json
{
  "package_id": "dp_evt_0001_20260610T091530",
  "schema_version": "1.0",
  "created_at": "2026-06-10T09:15:35+09:00",
  "event_id": "evt_0001",
  "decision_authority": "NEO",
  "knowledge_base": {},
  "decision": {},
  "confidence": {},
  "reasoning": {},
  "assumptions": {},
  "conflicts": [],
  "actions": [],
  "source_facts": [],
  "provenance": {},
  "downstream_context": {},
  "warnings": []
}
```

## 5. 필수 최상위 필드

| 필드 | 필수 | 목적 |
|---|---:|---|
| `package_id` | 예 | 안정적인 패키지 식별자 |
| `schema_version` | 예 | Decision Package 구조 버전 |
| `created_at` | 예 | 패키지 생성 시각 |
| `event_id` | 예 | 판단 대상 사건 |
| `decision_authority` | 예 | 반드시 `NEO` |
| `knowledge_base` | 예 | 판단에 사용한 KB 산출물 버전과 해시 |
| `decision` | 예 | 최종 NEO 판단 필드 |
| `confidence` | 예 | CF에서 계산한 신뢰도 필드 |
| `reasoning` | 예 | 규칙과 추론 이력 메타데이터 |
| `assumptions` | 예 | ATMS 지지 근거 메타데이터 |
| `conflicts` | 예 | 충돌과 nogood 메타데이터 |
| `actions` | 예 | NEO 권고 조치 |
| `source_facts` | 예 | NEO가 사용한 Fact |
| `provenance` | 예 | 소스와 그래프 연결 정보 |
| `downstream_context` | 예 | Neo4j, NEMI, 설명 모델과 Vue 연결 키 |
| `warnings` | 예 | 운영자에게 표시할 경고 |

### 5.1 Knowledge Base 고정

`knowledge_base` 객체는 패키지를 생성할 때 사용한 정확한 NEO KB 산출물과 규칙 집합 버전을 식별한다. 같은 입력을 같은 KB 버전과 해시 기준으로 비교할 수 있어 반복 검토가 가능하다.

```json
{
  "kb_version": "neo_demo_integrated_incident_predmaint_metadata",
  "kb_hash_sha256": "9c1c32e9d7baff44678a713b1a0778ee8b88d7d1d4bbe695c556fbad8f6b4db2",
  "rule_set_version": "neo_demo_integrated_incident_predmaint_metadata",
  "generated_at": "2026-06-23T01:47:35+09:00",
  "artifact_name": "neo_demo_integrated_incident_predmaint_metadata.kb",
  "artifact_available": true,
  "pinning_scope": "NEO reasoning KB artifact used by the wrapper for this Decision Package."
}
```

## 6. Decision 객체

`decision` 객체에는 NEO 판단 권한 필드를 기록한다.

```json
{
  "decision_mode": "incident_response",
  "neo_decision": "requires_operator_ack",
  "incident_class": "traffic_lane_blockage",
  "risk_class": null,
  "impact_level": "high",
  "operator_priority": "p1",
  "maintenance_priority": null,
  "unified_priority": "p1",
  "queue_reason": "active lane blockage has immediate traffic impact"
}
```

### 6.1 판단 모드

허용값:

```text
incident_response
predictive_maintenance
hybrid_arbitration
monitor_only
insufficient_evidence
```

### 6.2 우선순위

허용 우선순위:

```text
p1
p2
p3
none
unknown
```

우선순위는 NEO 규칙 출력에서만 가져와야 한다.

## 7. Confidence 객체

`confidence` 객체에는 CF에서 계산한 신뢰도 정보를 기록한다.

```json
{
  "decision_confidence": 0.75,
  "cf_policy": "source_reliability * detector_confidence * freshness_factor",
  "cf_inputs": [
    {
      "fact_id": "fact_evt_0001_001",
      "observation_type": "anomaly_level",
      "assumption_id": "asm_evt_0001_anomaly_level",
      "fact_group": "maintenance_observation",
      "detector_confidence": 0.84,
      "source_reliability": 0.9,
      "freshness_factor": 1.0,
      "cf_hint": 0.756,
      "cf_role": "evidence",
      "include_in_decision_confidence": true
    }
  ],
  "confidence_notes": [
    "decision confidence is derived from NEO-side CF propagation"
  ]
}
```

규칙:

- `decision_confidence`는 0.0 이상 1.0 이하여야 한다.
- 설명 모델은 이 값을 지정하면 안 된다.
- NEMI와 Neo4j는 이 값을 표시할 수 있지만 변경하면 안 된다.
- `cf_role = evidence` 입력은 판단 신뢰도 계산에 참여한다.
- 요청 의도와 같은 `cf_role = context` 입력은 추적을 위해 표시하지만 판단 신뢰도 계산에서는 제외한다.
- 감지기 모델 품질과 같은 `cf_role = model_quality` 입력은 별도로 표시하고 판단 신뢰도 계산에서는 제외한다.

## 8. Reasoning 객체

`reasoning` 객체에는 규칙 매칭과 추론 이력을 기록한다.

```json
{
  "matched_rules": [
    "rule_lane_block_p1",
    "guard_stale_data_check"
  ],
  "candidate_rules": [
    "rule_lane_block_p1",
    "rule_camera_degradation_p2"
  ],
  "failed_conditions": [
    {
      "rule_id": "guard_stale_data_check",
      "condition": "freshness stale",
      "result": false
    }
  ],
  "derived_facts": [
    "(evt_0001 incident_class traffic_lane_blockage)",
    "(evt_0001 operator_priority p1)"
  ],
  "forward_results": [],
  "backward_results": [],
  "match_trace": [
    {
      "step": 1,
      "rule_id": "rule_lane_block_p1",
      "matched_facts": [
        "fact_evt_0001_001",
        "fact_evt_0001_002"
      ],
      "bindings": {
        "?event": "evt_0001",
        "?asset": "road_link_12"
      },
      "result": "matched"
    }
  ]
}
```

규칙:

- `matched_rules`에는 최종 판단에 기여한 규칙을 포함해야 한다.
- `failed_conditions`에는 관련 조건 검사 또는 충돌 조건을 포함해야 한다.
- `match_trace`는 UI 표시와 검토에 사용할 수 있는 구조여야 한다.

## 9. Assumptions 객체

`assumptions` 객체에는 ATMS 지지 근거 메타데이터를 기록한다.

```json
{
  "atms_context": "incident_atms",
  "support_assumptions": [
    "asm_evt_0001_cctv_17_lane_blocked",
    "asm_evt_0001_public_its_lane_blocked"
  ],
  "support_sets": [
    {
      "support_set_id": "support_evt_0001_001",
      "assumptions": [
        "asm_evt_0001_cctv_17_lane_blocked",
        "asm_evt_0001_public_its_lane_blocked"
      ],
      "justification": "rule_lane_block_p1",
      "valid": true
    }
  ],
  "invalid_support_sets": []
}
```

규칙:

- 소스 근거가 있는 Fact는 가정으로 표시해야 한다.
- 어떤 Support Set이 판단을 유효하게 유지하는지 패키지에서 설명해야 한다.
- 유효하지 않은 Support Set도 검토를 위해 보존해야 한다.

## 10. Conflicts 배열

`conflicts` 배열에는 충돌과 nogood 정보를 기록한다.

```json
[
  {
    "conflict_id": "conflict_evt_0001_stale_cctv",
    "conflict_type": "stale_data",
    "affected_assumptions": [
      "asm_evt_0001_cctv_17_lane_blocked"
    ],
    "nogood_set": [
      "asm_evt_0001_cctv_17_lane_blocked"
    ],
    "severity": "warning",
    "effect": "weakens_support",
    "message": "CCTV evidence is stale risk but public ITS evidence still supports the decision."
  }
]
```

허용되는 `effect` 값:

```text
blocks_decision
weakens_support
requires_operator_check
informational
```

## 11. Actions 배열

`actions` 배열에는 NEO가 권고한 조치를 기록한다.

```json
[
  {
    "action_id": "act_evt_0001_001",
    "action_key": "activate_vms_detour",
    "action_label": "Activate VMS detour notice",
    "action_priority": "p1",
    "source_rule": "rule_lane_block_p1",
    "requires_operator_ack": true
  },
  {
    "action_id": "act_evt_0001_002",
    "action_key": "dispatch_field_check",
    "action_label": "Dispatch field inspection",
    "action_priority": "p1",
    "source_rule": "rule_lane_block_p1",
    "requires_operator_ack": true
  }
]
```

규칙:

- 조치는 NEO 출력에서만 가져와야 한다.
- 설명 모델은 조치를 설명할 수 있지만 새로운 조치를 생성하면 안 된다.
- Vue는 조치를 표시하고 운영자 확인을 요청할 수 있지만, 확인 절차는 판단 권한과 분리한다.

## 12. Source Facts 배열

`source_facts` 배열은 판단을 원본 Canonical Fact와 연결한다.

```json
[
  {
    "fact_id": "fact_evt_0001_001",
    "event_id": "evt_0001",
    "source_id": "cctv_17",
    "observation_type": "lane_blocked",
    "confidence": 0.84,
    "assumption_id": "asm_evt_0001_cctv_17_lane_blocked",
    "used_by_rules": [
      "rule_lane_block_p1"
    ]
  }
]
```

## 13. Provenance 객체

`provenance` 객체는 그래프와 소스 연결 정보를 제공한다. 원본 참조값과 체크섬은 추적을 위해 보존하지만, NEO가 규칙 추론 중 원본 데이터의 무결성을 검증했다는 의미는 아니다.

```json
{
  "raw_refs": [
    "raw://its/cctv_17/20260610T091530"
  ],
  "neo4j": {
    "event_node_id": "Event:evt_0001",
    "decision_node_id": "Decision:dp_evt_0001_20260610T091530",
    "path_templates": [
      "Event -> Fact -> Rule -> Decision -> Action",
      "Fact -> Source",
      "Decision -> AssumptionSet",
      "Decision -> ConflictSet"
    ]
  },
  "nemi": {
    "query_keys": [
      "traffic_lane_blockage",
      "activate_vms_detour",
      "road_link_12",
      "rule_lane_block_p1"
    ]
  }
}
```

## 14. Downstream Context 객체

`downstream_context` 객체에는 FastAPI, Vue, Neo4j, NEMI와 설명 모델에 제공할 통제된 문맥을 기록한다.

```json
{
  "vue": {
    "display_cards": [
      "decision",
      "confidence",
      "actions",
      "trace",
      "graph",
      "documents",
      "chatbot"
    ]
  },
  "neo4j": {
    "write_graph": true,
    "visualize_path": true
  },
  "nemi": {
    "retrieve_documents": true,
    "top_k": 3
  },
  "llm": {
    "allow_explanation": true,
    "allow_decision_override": false,
    "allowed_sources": [
      "decision",
      "reasoning",
      "assumptions",
      "conflicts",
      "actions",
      "nemi",
      "neo4j"
    ]
  }
}
```

## 15. Warnings 배열

경고는 운영자에게 표시하지만 판단 권한을 갖지 않는다.

```json
[
  {
    "warning_id": "warn_evt_0001_low_confidence",
    "warning_type": "low_confidence",
    "severity": "warning",
    "message": "One supporting observation has low confidence.",
    "related_fact_ids": [
      "fact_evt_0001_003"
    ]
  }
]
```

## 16. 전체 예시

```json
{
  "package_id": "dp_evt_0001_20260610T091530",
  "schema_version": "1.0",
  "created_at": "2026-06-10T09:15:35+09:00",
  "event_id": "evt_0001",
  "decision_authority": "NEO",
  "decision": {
    "decision_mode": "incident_response",
    "neo_decision": "requires_operator_ack",
    "incident_class": "traffic_lane_blockage",
    "risk_class": null,
    "impact_level": "high",
    "operator_priority": "p1",
    "maintenance_priority": null,
    "unified_priority": "p1",
    "queue_reason": "active lane blockage has immediate traffic impact"
  },
  "confidence": {
    "decision_confidence": 0.75,
    "cf_policy": "source_reliability * detector_confidence * freshness_factor",
    "cf_inputs": [
      {
        "fact_id": "fact_evt_0001_001",
        "observation_type": "anomaly_level",
        "assumption_id": "asm_evt_0001_anomaly_level",
        "fact_group": "maintenance_observation",
        "detector_confidence": 0.84,
        "source_reliability": 0.9,
        "freshness_factor": 1.0,
        "cf_hint": 0.756,
        "cf_role": "evidence",
        "include_in_decision_confidence": true
      }
    ],
    "confidence_notes": [
      "decision confidence is derived from NEO-side CF propagation"
    ]
  },
  "reasoning": {
    "matched_rules": [
      "rule_lane_block_p1"
    ],
    "candidate_rules": [
      "rule_lane_block_p1"
    ],
    "failed_conditions": [],
    "derived_facts": [
      "(evt_0001 incident_class traffic_lane_blockage)",
      "(evt_0001 operator_priority p1)"
    ],
    "forward_results": [],
    "backward_results": [],
    "match_trace": [
      {
        "step": 1,
        "rule_id": "rule_lane_block_p1",
        "matched_facts": [
          "fact_evt_0001_001"
        ],
        "bindings": {
          "?event": "evt_0001"
        },
        "result": "matched"
      }
    ]
  },
  "assumptions": {
    "atms_context": "incident_atms",
    "support_assumptions": [
      "asm_evt_0001_cctv_17_lane_blocked"
    ],
    "support_sets": [
      {
        "support_set_id": "support_evt_0001_001",
        "assumptions": [
          "asm_evt_0001_cctv_17_lane_blocked"
        ],
        "justification": "rule_lane_block_p1",
        "valid": true
      }
    ],
    "invalid_support_sets": []
  },
  "conflicts": [],
  "actions": [
    {
      "action_id": "act_evt_0001_001",
      "action_key": "activate_vms_detour",
      "action_label": "Activate VMS detour notice",
      "action_priority": "p1",
      "source_rule": "rule_lane_block_p1",
      "requires_operator_ack": true
    }
  ],
  "source_facts": [
    {
      "fact_id": "fact_evt_0001_001",
      "event_id": "evt_0001",
      "source_id": "cctv_17",
      "observation_type": "lane_blocked",
      "confidence": 0.84,
      "assumption_id": "asm_evt_0001_cctv_17_lane_blocked",
      "used_by_rules": [
        "rule_lane_block_p1"
      ]
    }
  ],
  "provenance": {
    "raw_refs": [
      "raw://its/cctv_17/20260610T091530"
    ],
    "neo4j": {
      "event_node_id": "Event:evt_0001",
      "decision_node_id": "Decision:dp_evt_0001_20260610T091530",
      "path_templates": [
        "Event -> Fact -> Rule -> Decision -> Action"
      ]
    },
    "nemi": {
      "query_keys": [
        "traffic_lane_blockage",
        "activate_vms_detour",
        "road_link_12",
        "rule_lane_block_p1"
      ]
    }
  },
  "downstream_context": {
    "vue": {
      "display_cards": [
        "decision",
        "confidence",
        "actions",
        "trace",
        "graph",
        "documents",
        "chatbot"
      ]
    },
    "neo4j": {
      "write_graph": true,
      "visualize_path": true
    },
    "nemi": {
      "retrieve_documents": true,
      "top_k": 3
    },
    "llm": {
      "allow_explanation": true,
      "allow_decision_override": false,
      "allowed_sources": [
        "decision",
        "reasoning",
        "assumptions",
        "conflicts",
        "actions",
        "nemi",
        "neo4j"
      ]
    }
  },
  "warnings": []
}
```

## 17. 설명 모델 계약

설명 모델은 다음 작업을 수행할 수 있다.

```text
판단 요약
매칭된 규칙 설명
신뢰도와 경고 조건 설명
Neo4j 그래프 경로 요약
NEMI 문서 요약
패키지와 검색 근거를 사용한 운영자 질문 답변
```

설명 모델은 다음 작업을 수행하면 안 된다.

```text
우선순위 변경
새 조치 생성
neo_decision 변경
근거 없는 내용 생성
충돌 또는 경고 숨김
```

## 18. Neo4j 계약

Neo4j는 다음 노드를 저장하고 조회할 수 있다.

```text
Event
Fact
Rule
Decision
Action
Source
AssumptionSet
ConflictSet
Document
```

Neo4j는 최종 판단 권한 값을 계산하면 안 된다.

## 19. NEMI 계약

NEMI는 다음 자료를 검색할 수 있다.

```text
실행 절차
정책
과거 사례
유지보수 기록
운영자 안내 문서
```

NEMI는 최종 판단 권한 값을 계산하면 안 된다.

## 20. 수용 기준

다음 조건을 모두 충족하면 Decision Package 표준 구조를 사용할 수 있다.

- NEO 판단 권한 필드를 포함한다.
- CF에서 계산한 판단 신뢰도를 포함한다.
- ATMS 지지 가정과 Support Set을 포함한다.
- 필요한 경우 충돌 또는 nogood 메타데이터를 포함한다.
- 매칭된 규칙과 규칙 매칭 이력을 포함한다.
- 원본 Fact와 연결된다.
- Neo4j 그래프 연결 키를 제공한다.
- NEMI 검색 키를 제공한다.
- 통제된 설명 모델 문맥을 제공한다.
- 어떤 하위 계층도 NEO 판단을 덮어쓸 수 없다.

## 21. 다음 단계

이 표준 구조를 확정한 뒤 진행할 다음 설계 단계는 다음과 같다.

```text
NEO KB / 래퍼 / 출력 매핑 설계
```

다음 단계에서는 기존 NEO 엔진을 사용하며 KB Fact, 규칙, 래퍼 매핑과 Decision Package 변환에 집중한다.
