# niw.route_candidates Prompt Snapshot

Snapshot date: 2026-05-25

Prompt name: `niw.route_candidates`

Public snapshot version: `v0.1.0`

Source note: adapted from an internal NIW workflow prompt and sanitized for public use.

## Purpose

`niw.route_candidates` is the Strategy Lab route generation task. It designs NIW strategy route candidates for route comparison and primary/backup route selection.

The task consumes:

- `profile_context_json`
- `evidence_inventory_json`
- `existing_pe_titles`
- `existing_routes_json`
- canonical `route_mode_options_json`
- canonical `impact_shape_options_json`
- optional `instruction`

It returns a single JSON object with a `candidates` array.

## Prompt

```text
You are designing NIW route candidates for strategy selection.

Return JSON only that satisfies this JSON Schema:
{output_schema}

HARD RULES
1) Output MUST be a single JSON object with a `candidates` array.
2) Use only the provided profile, evidence inventory, and selected PE titles. Do not invent facts.
3) Generate 2 to 4 route candidates. Make them genuinely distinct.
4) `fit_score`, `proofability_score`, and `risk_score` must be integers from 0 to 100.
5) `risk_score` means risk is higher when the number is higher.
6) `route_mode` MUST be chosen only from the canonical route modes provided below. Do not invent new route_mode values.
7) `impact_shapes` MUST use 1 to 3 values chosen only from the canonical impact shapes provided below. Do not invent new impact_shapes values.
8) If a candidate does not fit an allowed canonical label perfectly, choose the closest allowed label instead of creating a new one.
9) `summary`, `why_primary`, and `next_actions` must be specific to the candidate's current record.
10) `top_risks` should use concise risk codes in uppercase snake_case when possible.
11) When existing routes are provided, every new candidate MUST differ from every existing route in at least one of these dimensions: `route_mode`, `impact_shapes`, or the narrative theme expressed by `title`, `summary`, and `why_primary`.
12) Do NOT submit a paraphrase, renamed duplicate, or same-strategy restatement of an existing route. If the strategic thesis is materially the same, it is invalid even if wording changes.
13) Prefer candidates that open a genuinely different route composition, evidence posture, or impact story instead of cosmetic variation.
14) An `Employer-led Industry Implementation` candidate is valid only when it can be framed as a broader national-interest endeavor advanced through an employer platform, not as ordinary employer-specific job duties or "work for Company X as Y".
15) For any `Employer-led Industry Implementation` candidate, `summary` and `why_primary` must explicitly explain why, on balance, waiving the job-offer / labor-certification framework could be justified despite a possible employer-sponsored PERM path.
16) Ground that employer-led waiver rationale in at least one concrete, Dhanasar-compatible prong-3 rationale: the petitioner's value cannot be adequately captured through PERM's minimum job-requirement framework; even assuming other qualified U.S. workers are available, the United States would still materially benefit from this petitioner's specific contributions; or the endeavor has sufficiently urgent national-interest value.
17) Do not treat employer prestige, company importance, team importance, or job seniority as prong-3 logic. Do not say PERM is impossible unless the provided facts support it. Urgency is one possible rationale, not a required rationale.
18) If the provided facts cannot support a concrete employer-led prong-3 rationale, do not choose `Employer-led Industry Implementation` unless the user explicitly instructs it. If the user explicitly instructs it despite weak facts, lower `fit_score` and `proofability_score`, raise `risk_score`, include `PERM_LIKE_EMPLOYER_DUTIES` in `top_risks`, and make `next_actions` request concrete waiver evidence.

CANONICAL ROUTE MODES
<ROUTE_MODE_OPTIONS>
{route_mode_options_json}
</ROUTE_MODE_OPTIONS>

CANONICAL IMPACT SHAPES
<IMPACT_SHAPE_OPTIONS>
{impact_shape_options_json}
</IMPACT_SHAPE_OPTIONS>

PROFILE CONTEXT
<PROFILE_CONTEXT>
{profile_context_json}
</PROFILE_CONTEXT>

EVIDENCE INVENTORY
<EVIDENCE_INVENTORY>
{evidence_inventory_json}
</EVIDENCE_INVENTORY>

SELECTED PE TITLES
<SELECTED_PE_TITLES>
{existing_pe_titles}
</SELECTED_PE_TITLES>

EXISTING ROUTES
<EXISTING_ROUTES>
{existing_routes_json}
</EXISTING_ROUTES>

OPTIONAL INSTRUCTION
<INSTRUCTION>
{instruction}
</INSTRUCTION>
```

## Canonical Route Modes

```json
[
  "Employer-led Industry Implementation",
  "Independent Expert / Advisory",
  "Founder-led Venture",
  "Research-to-Deployment Translation"
]
```

## Canonical Impact Shapes

```json
[
  "Platform / Infrastructure",
  "Method / Process Innovation",
  "Product / Service Deployment",
  "Ecosystem / Education / Standards / Training",
  "Public / Social Impact"
]
```

## Runtime JSON Schema

This schema is generated by `prompt_json_schema(RouteCandidatesOutput)`.

```json
{
  "$defs": {
    "RouteCandidateItem": {
      "additionalProperties": false,
      "properties": {
        "fit_score": {
          "maximum": 100,
          "minimum": 0,
          "type": "integer"
        },
        "impact_shapes": {
          "items": {
            "enum": [
              "Platform / Infrastructure",
              "Method / Process Innovation",
              "Product / Service Deployment",
              "Ecosystem / Education / Standards / Training",
              "Public / Social Impact"
            ],
            "type": "string"
          },
          "type": "array"
        },
        "next_actions": {
          "items": {
            "type": "string"
          },
          "type": "array"
        },
        "proofability_score": {
          "maximum": 100,
          "minimum": 0,
          "type": "integer"
        },
        "risk_score": {
          "maximum": 100,
          "minimum": 0,
          "type": "integer"
        },
        "route_mode": {
          "enum": [
            "Employer-led Industry Implementation",
            "Independent Expert / Advisory",
            "Founder-led Venture",
            "Research-to-Deployment Translation"
          ],
          "type": "string"
        },
        "summary": {
          "type": "string"
        },
        "title": {
          "type": "string"
        },
        "top_risks": {
          "items": {
            "type": "string"
          },
          "type": "array"
        },
        "why_primary": {
          "type": "string"
        }
      },
      "required": [
        "title",
        "route_mode",
        "summary",
        "fit_score",
        "proofability_score",
        "risk_score",
        "why_primary"
      ],
      "type": "object"
    }
  },
  "additionalProperties": false,
  "properties": {
    "candidates": {
      "items": {
        "$ref": "#/$defs/RouteCandidateItem"
      },
      "type": "array"
    }
  },
  "type": "object"
}
```

## Output Field Contract

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `candidates` | `RouteCandidateItem[]` | optional by schema, required by prompt | Strategy Lab expects one object containing this array. |
| `title` | `string` | yes | Human-readable route title. |
| `route_mode` | enum | yes | Must be one of the canonical route modes. |
| `impact_shapes` | enum array | no | Prompt requires 1 to 3 values, although schema only enforces array item enum. |
| `summary` | `string` | yes | Must be specific to the user's current record. |
| `fit_score` | integer 0-100 | yes | Strategic fit score. |
| `proofability_score` | integer 0-100 | yes | Evidence/proof readiness score. |
| `risk_score` | integer 0-100 | yes | Higher means riskier. |
| `why_primary` | `string` | yes | Why this route could be the primary strategic anchor. |
| `top_risks` | `string[]` | no | Prefer uppercase snake_case risk codes. |
| `next_actions` | `string[]` | no | Concrete actions tied to this route candidate. |
