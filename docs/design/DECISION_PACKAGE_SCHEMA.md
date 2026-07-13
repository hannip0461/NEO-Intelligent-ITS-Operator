# Decision Package Schema

## 1. Purpose

This document defines the standard Decision Package produced after NEO reasoning.

The Decision Package is the single structured output shared by FastAPI, Vue, Neo4j, NEMI, and LLM explanation layers.

## 2. Core Principle

NEO is the only decision authority.

The Decision Package may be stored, visualized, explained, and searched by downstream layers, but downstream layers must not create or override:

```text
neo_decision
decision_mode
priority
unified_priority
recommended_actions
decision_confidence
```

## 3. Producer and Consumers

| Role | Component | Responsibility |
|---|---|---|
| Producer | NEO Reasoning Core / wrapper | Produce and normalize the Decision Package |
| Orchestrator | FastAPI | Serve the Decision Package and enrich it with support data |
| UI consumer | Vue | Display decision, trace, graph, evidence, and chatbot answer |
| Graph consumer | Neo4j | Store and expose provenance paths |
| Evidence consumer | NEMI | Retrieve document history and evidence |
| Explanation consumer | LLM | Explain the package without changing it |

## 4. Top-Level Shape

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

## 5. Required Top-Level Fields

| Field | Required | Purpose |
|---|---:|---|
| `package_id` | Yes | Stable package identifier |
| `schema_version` | Yes | Decision Package schema version |
| `created_at` | Yes | Package creation time |
| `event_id` | Yes | Event being judged |
| `decision_authority` | Yes | Must be `NEO` |
| `knowledge_base` | Yes | KB artifact version and hash used for this decision |
| `decision` | Yes | Final NEO decision fields |
| `confidence` | Yes | CF-derived confidence fields |
| `reasoning` | Yes | Rule and trace metadata |
| `assumptions` | Yes | ATMS support metadata |
| `conflicts` | Yes | Conflict and nogood metadata |
| `actions` | Yes | NEO-recommended actions |
| `source_facts` | Yes | Facts used by NEO |
| `provenance` | Yes | Source and graph linkage |
| `downstream_context` | Yes | Keys for Neo4j, NEMI, LLM, Vue |
| `warnings` | Yes | Operator-facing warnings |

### 5.1 Knowledge Base Pinning

The `knowledge_base` object identifies the exact NEO KB artifact and rule-set
version used to create the package. It supports repeatable review: the same
input can be compared against the same KB version and hash.

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

## 6. Decision Object

The `decision` object contains NEO authority fields.

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

### 6.1 Decision Mode

Allowed values:

```text
incident_response
predictive_maintenance
hybrid_arbitration
monitor_only
insufficient_evidence
```

### 6.2 Priority

Allowed priority values:

```text
p1
p2
p3
none
unknown
```

Priority must come from NEO rule output.

## 7. Confidence Object

The `confidence` object carries CF-derived confidence information.

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

Rules:

- `decision_confidence` must be between 0.0 and 1.0.
- LLM must not assign this value.
- NEMI and Neo4j may display this value but must not modify it.
- `cf_role = evidence` inputs participate in decision confidence scoring.
- `cf_role = context` inputs, such as request intent, are displayed for traceability but excluded from decision confidence scoring.
- `cf_role = model_quality` inputs, such as detector model quality, are displayed separately and excluded from decision confidence scoring.

## 8. Reasoning Object

The `reasoning` object records rule matching and inference trace.

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

Rules:

- `matched_rules` must include the rules that contributed to the final decision.
- `failed_conditions` should include relevant guard or conflict conditions.
- `match_trace` should be structured enough for UI and review use.

## 9. Assumptions Object

The `assumptions` object records ATMS support metadata.

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

Rules:

- Source-backed facts should appear as assumptions.
- The package should explain which support set keeps the decision valid.
- Invalid support sets should be preserved for review.

## 10. Conflicts Array

The `conflicts` array records conflict and nogood information.

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

Allowed `effect` values:

```text
blocks_decision
weakens_support
requires_operator_check
informational
```

## 11. Actions Array

The `actions` array contains NEO-recommended actions.

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

Rules:

- Actions must come from NEO output.
- LLM may explain actions but must not create new actions.
- Vue may display and request operator acknowledgement, but acknowledgement workflow is separate from decision authority.

## 12. Source Facts Array

The `source_facts` array links the decision back to canonical facts.

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

## 13. Provenance Object

The `provenance` object provides graph and source linkage.
Raw references and checksums are preserved for traceability. They do not imply
that NEO verified raw-data integrity during rule reasoning.

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

## 14. Downstream Context Object

The `downstream_context` object contains controlled context for FastAPI, Vue, Neo4j, NEMI, and LLM.

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

## 15. Warnings Array

Warnings are operator-facing but non-authoritative.

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

## 16. Complete Example

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

## 17. LLM Contract

The LLM may:

```text
summarize the decision
explain matched rules
explain confidence and warning conditions
summarize Neo4j graph paths
summarize NEMI documents
answer user questions using the package and retrieved evidence
```

The LLM must not:

```text
change priority
create new actions
change neo_decision
invent unsupported evidence
hide conflicts or warnings
```

## 18. Neo4j Contract

Neo4j may store and expose:

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

Neo4j must not compute final decision authority.

## 19. NEMI Contract

NEMI may retrieve:

```text
runbooks
policies
past cases
maintenance notes
operator guidance documents
```

NEMI must not compute final decision authority.

## 20. Acceptance Criteria

The Decision Package schema is accepted when:

- it contains NEO decision authority fields
- it contains CF-derived decision confidence
- it contains ATMS support assumptions and support sets
- it contains conflict or nogood metadata when applicable
- it contains matched rules and match trace
- it links back to source facts
- it provides Neo4j graph linkage keys
- it provides NEMI retrieval keys
- it provides controlled LLM context
- no downstream layer is allowed to override NEO decisions

## 21. Next Step

After this schema is fixed, the next design step is:

```text
NEO KB / wrapper / output mapping design
```

This next step uses the existing NEO engine and focuses on KB facts, rules, wrapper mapping, and Decision Package conversion.
