JSON vs. YAML
------------------------

JSON is the right call. Let me explain the tradeoffs clearly, then give you the full schema.

Why JSON over YAML for this use case
YAML problems for deterministic agent consumption:

Implicit type coercion — yes, on, true all parse as boolean; 1.0 and 1 are different types; PROJ-123 is a string but 123 is an integer. Agents get surprising values.
Indentation-sensitive — a single bad indent silently changes structure without a parse error
12+ spec versions, inconsistent multi-document support, anchors/aliases that break naive parsers
Weak validation tooling vs. the mature JSON Schema ecosystem
JSON wins:

One spec, universal parsers, strict syntax — a structural error is always a parse error
JSON Schema validation is mature with tooling in every language
JIRA REST API is JSON — you're already in JSON when reading/writing epics
Native to MongoDB, DynamoDB, Elasticsearch, Postgres JSONB, REST APIs, GraphQL — zero conversion
Claude tool use returns JSON — agent pipelines stay in one format end to end
The one real cost: prose/multiline text is less readable in JSON (\n escapes). The mitigation is keeping prose fields as clean markdown strings — LLMs handle them fine, and a UI renders them properly.

Notes on JSON schema
--------------------------------
Key design decisions worth noting
Structured Gherkin instead of raw strings. Each AC is { id, fr_ref, scenario, given, when, then[], and[] } — individually addressable, directly traceable back to an FR, and consumable by test writers without parsing free text.

fr_ref on every AC. The spec writer requires source anchors; this makes the chain G → FR → AC → Story → Task fully traceable in both directions.

references typed by artifact kind. Figma, GitHub repos, architecture diagrams, and examples are separate arrays. Each downstream agent filters to its type — the architect reads github_repos and architecture_diagrams; the spec writer reads spec_documents; the developer reads examples and github_repos.

problem_statement as a markdown string. This is what orc-test currently injects verbatim into every agent prompt. Keeping it as a string field means zero change to the runtime injection path.

out_of_scope as objects with rationale. A plain string list lets people write "mobile app" and forget why. The epic critic raises a finding when rationale is absent — enforcing it in the schema means the writer agent can't produce a valid document without it.

schema: "orc-test.epic/v1" as a required const field. Any consumer can version-check before parsing, and the orc-test runtime can use it to select the right deserializer when v2 lands.

Epic schema instance example notes
-----------------------------------
A few things worth noting about the choices made in this instance:

Traceability chain is fully wired. Every FR has source pointing to a goal ID (G-1, G-2, G-3) or a named stakeholder decision. Every AC has a fr_ref. Every story outline entry has fr_refs. The spec writer agent can trace Epic Goal → FR → AC → Story → Task without guessing.

NFRs are measurable numbers, not adjectives. ≤ 500ms p95 and 99.95% uptime during market hours — not "fast" or "highly available". The architect can turn these directly into SLOs and Datadog alerts.

Out-of-scope items all have rationale and cross-references. Each excluded item names the JIRA epic where the work lives, so the story writer agent knows not to invent stories for that scope.

Risks include all three required fields. mitigation, trigger, and contingency are all present — the epic critic requires all three or raises a Medium finding.

Dependencies are split internal/external and have blocking: true/false. The architect and planner agents need to know which dependencies gate the work vs. which are nice-to-have timing constraints.

Final Notes
----
Should we have a schema for how this info will display on Jira in a markdown format? So that it is user friendly on Jira, it would persist as a markdown in the jira description, other properties, etc. We would then have a converter for jira epic markdown format -> epic json schema and vice versa.