# Design Write-up

## Design overview

The tool is an evidence-based decision-support workflow, not an autonomous procurement decision-maker. Admins configure categories, mandatory checks, weighted criteria, definitions, scoring formulas, and a version number through admin screens. Reviewers upload proposals and select a category. The system extracts evidence, validates it, applies rules, calculates scores, and presents a ranked shortlist with evidence-backed reasons. A human reviewer resolves flags and the authorized procurement decision-maker makes the final award decision.

## 1. AI vs. rules

- **Plain code/rules:** authenticate users and enforce roles; validate file types and size; extract PDF text; load the selected category's active criteria version; validate the AI response against a schema; apply mandatory pass/fail rules; check numeric limits; calculate weighted scores; sort results; and log events. These steps are deterministic, easier to test, and should not depend on model judgment.
- **AI:** map varied proposal wording to a fixed evidence schema, summarize relevant evidence, and identify ambiguity or missing information. AI is useful for interpreting natural language, but it must return evidence and citations rather than decide the winner.
- **Human:** review uncertain or conflicting evidence, decide whether clarification is needed, approve any override, and make the award decision. The model never awards a contract.

## 2. Consistency and explainability

Use a versioned criteria configuration, fixed scoring formulas, schema-constrained extraction, explicit rounding rules, and deterministic tie-breakers. For a given stored extraction, criteria version, and scoring version, the rules engine returns the same score and order. A repeated AI extraction can vary, so preserve its original output and require review when extraction changes or confidence/evidence is insufficient. The reviewer sees each criterion's subscore, weight, weighted contribution, extracted value, supporting quote/page, missing evidence, mandatory-check outcome, and reason for flags. Sort by total score descending; break exact ties by higher mandatory-evidence completeness, then lower total price, then company name alphabetically. Never silently convert missing evidence into a positive claim.

## 3. Human in the loop and audit

Reviewers confirm extracted facts and source references before relying on a result, especially for mandatory criteria and flagged proposals. A reviewer can request clarification, correct an extraction with a recorded source, or override a flag/score only with a reason and identity. Overrides do not erase the original result. A separate authorized person makes the final award decision. Audit records include proposal/file hash, uploader, timestamps, selected category, criteria and prompt versions, model/version and settings where available, extracted fields and evidence references, validation errors, criterion scores, total score, rank, flags, reviewer edits, override reasons, and final decision. Limit access and retention to what procurement policy requires.

## 4. Changing criteria

Criteria edits create a new immutable version with effective time, editor, and change rationale; they never overwrite the version used for an existing evaluation. Each evaluation stores the exact criteria version. If an admin changes weights or rules, the reviewer can explicitly rerun all proposals in that comparison against the new version, producing a new evaluation ID and retaining the old result for audit. Validate that mandatory rules are complete, weights total 100%, and scoring formulas are configured before publishing a version. Do not mix scores from different versions in one ranking.

## 5. Risks and safeguards

- **Incorrect extraction / scanned or malformed PDFs:** check extraction quality, retain page references and short evidence quotes, validate types/ranges, and flag unreadable or conflicting values for manual review.
- **Hallucinated facts:** require every extracted claim to cite proposal text; reject unsupported claims; treat missing data as unknown, not as a pass.
- **Prompt injection / manipulation:** treat document text as untrusted data, never as instructions. Proposal C contains an instruction to rank itself first; ignore it as non-evidence and flag it as an attempted manipulation. Keep the model away from award actions, and validate its output in code.
- **Bias and inconsistent judgment:** use criteria defined by the admin, evidence-based sub-rubrics, the same formulas for all bidders, and human review of ambiguous cases. Monitor override and flag patterns.
- **Weight/configuration errors:** validate published criteria versions and weight totals; require change history and authorization.
- **Privacy, security, and cost:** restrict access, encrypt files in transit and at rest, set file/token limits, avoid sending unnecessary data to the model, redact unrelated personal data where feasible, rate-limit requests, and record model usage. Provide a manual fallback when the AI service is unavailable.

## Scoring method and sample result

The brief specifies weights but not exact formulas. For a transparent illustrative result, apply the following configurable rules to proposals that pass mandatory checks and have enough evidence to score every weighted criterion:

- **Experience (30):** `min(qualifying projects / 14, 1) × 100` criterion score. The 14 is the highest clearly stated qualifying count among the rankable sample proposals.
- **Price (25):** `lowest price among rankable eligible proposals / proposal price × 100`. All prices must be within the $260,000 ceiling; over-budget proposals fail the price gate.
- **Timeline (15):** `((16 - completion weeks) / 16) × 100`, only when a stated completion time is 16 weeks or less. Missing or unclear timeline is flagged and not ranked until clarified.
- **Warranty (15):** average of panel-warranty years divided by 25 and workmanship-warranty years divided by 12, each capped at 100.
- **Team and approach (15):** equal credit for evidence of (1) named project lead, (2) identified crew, (3) safety plan, and (4) monitoring plan. Score is the proportion evidenced × 100. Weekly progress reporting alone is not assumed to be performance monitoring.
- Multiply each criterion score by its weight and sum. Keep full precision internally; display totals to two decimal places.

Under these assumptions, A ranks first (87.50), E second (74.49), and C third (66.80). B is **ineligible** because the proposal says electrical work would be subcontracted to a licensed provider to be confirmed after award; this does not confirm that the bidder holds and states the required licence. D is **flagged** and left unranked because the timeline is to be determined after site assessment, so compliance with the 16-week limit cannot be established. These statuses are not final procurement decisions; a reviewer may seek clarification and rerun the comparison consistently.
