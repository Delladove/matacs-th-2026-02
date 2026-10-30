# AI-Assisted Proposal Triage Tool

The tool ranks contractor proposals against admin-defined criteria. A human makes the final award decision.

## AI vs. Rules

**AI (Gemini)**
- Extracts values from each proposal, along with the sentence that supports each value.
- Optionally writes a one-line reason for each rank.

**Rules (plain code)**
- Evaluate gate criteria (pass/fail).
- Score, weight and rank the proposals.

**Why this split:** Reading messy text is something AI does well and rules do poorly. Scoring must be exact and repeatable. Keeping it in code also means a weight change is recomputed directly, with no further API calls.

## Consistency and Explainability

- **Phase 1 (extraction):** Gemini reads the proposal and returns values.
- **Phase 2 (scoring):** Code applies the rules and computes scores.
- Phase 2 is deterministic, so ranking the same extractions twice gives the same result.
- Extraction stays consistent by using the same model version, the same extraction instructions and the same structured output schema. Results are cached, so proposals are not re-extracted on every run.
- The reviewer sees the ranked list, each proposal's per-criterion scores, and a short reason for its position.

## Human in the Loop

- For every extracted value, the system shows the exact line of text it came from. The reviewer can check whether the text was misinterpreted and whether edge cases were handled.
- The reviewer can edit a value at runtime. The score and ranking update immediately.
- The reviewer makes the final award decision. The ranking is advisory only.

## Changing Criteria

- **Weight change:** No new Gemini calls. Scores are recomputed from stored extractions.
- **New or edited criterion:** Only that criterion is re-extracted for each existing proposal (one small call each), keyed by a bumped criterion version. Old extractions are untouched.
- **Deleted criterion:** It is dropped from scoring, and its old data is kept.
- **Versioning:** Each ranking is tied to the criteria version that produced it, so changes do not break earlier results.

## Risks

| Risk | Guard |
|---|---|
| **Wrong extraction** (misread number, wrong sentence) | Every value comes with a quoted sentence the reviewer can check and correct. |
| **Made-up facts** | The prompt says "never guess, return null". Evidence must be an exact substring of the proposal, and the code rejects any value whose quote is not found in the text. |
| **Cost and latency** | One extraction call per proposal, cached. Weight changes and ranking need no AI calls. Only new criteria trigger small, targeted re-extractions. |

if there is 10days written in proposal it may extract it as 10 weeks.