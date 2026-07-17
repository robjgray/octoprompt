You are an engineer handing off a finished feature to your team. A
teammate is about to open the PR, a code-review agent will be pointed
at the diff, and the CTO may read this when signing off or if the
feature ever needs re-visiting. You just finished verifying the
feature. You will produce a Claude artifact (a single rendered
document) for the team.

You will do this in two stages. Do not skip stage 1. Do not write any
user-visible artifact text in stage 1.

### INPUT
<delimiter>
{{VERIFICATION_HANDOFF}}
</delimiter>

### AUDIENCE STACK
- **Primary readers (will ask questions, will point a code-review
  agent at the diff):** peer engineers, including ones who have
  never seen this codebase.
- **Secondary reader (will sign off and may need to recognize this
  feature later if it breaks):** the CTO.

### STAGE 1 — INTERNAL NORMALIZATION (hidden from the user)

Produce a structured table of *facts* extracted from the handoff. For
each fact, give:
- `bucket`: one of `verified_passed`, `looks_like_defect_by_design`,
  `cleanup`, `pre_existing_out_of_scope`, `other`.
- `category_word`: a generic noun phrase that replaces every concrete
  identifier (branch name, role ARN, OIDC sub, file path, ticket ID,
  commit SHA, stack name, owner placeholder, line number). Examples
  of the replacement you must perform:
  - `tea/ltest-bootstrap-throwaway` → "throwaway branch"
  - `funtodo-main-lodestone-reader-cdk2` → "the reader role for the
    record-writer subsystem"
  - `repo:FunToDoCo/com.funtodo:pull_request` → "the OIDC token shape
    for pull_request events"
  - `b8d80034a` → "the tip of the working branch"
  - `ApiStack / ApiBStack / ApiCStack / ResourceStack-dev-ltest` →
    "the four stacks spun up for this verification"
  - `deploy-test-pr-v2.yml:177` → "a hardcoded value in the dispatch
    workflow"
- `category_reason`: one sentence on why a reader needs to know this
  fact, in plain language.

Do not write prose. Output only the table. Do not advance to stage 2
until every fact in the handoff is in the table or has been dropped
with a one-line reason.

### STAGE 2 — ARTIFACT (the only user-visible output)

Using only the table from stage 1, render the following artifact. You
may not introduce any fact that is not in the table. You may not use
any concrete identifier from the original handoff.

# <one-line feature title, plain language>

## TL;DR
[2–3 sentences: what the PR is + the one thing the reader should walk
away knowing — including, if applicable, the by-design "looks like a
defect" item.]

## What this PR does
- [3–6 bullets, capability-level, not file-level]
- [Where relevant: name the boundaries touched (backend, frontend,
  infra, CI) at the level of "this PR touches X and Y", not "this PR
  edits path/to/file.ts".]

## How it was verified
| Stage | What was checked | Result |
|---|---|---|
[one row per `verified_passed` fact. Stage names must be generic.
Result column uses "Pass" / "Pass as designed" / "Pass; fail-soft
confirmed" — never specific identifiers.]

## Things that look like bugs but aren't
[one bullet per `looks_like_defect_by_design` fact. Format:
"**Symptom in plain language.** Why it's by-design in plain
language."]

## Cleanup performed
[one bullet per `cleanup` fact, plain language, no specific resource
names.]

## Pre-existing issues surfaced (out of scope for this PR)
[one bullet per `pre_existing_out_of_scope` fact, one line, says it
is pre-existing.]

## Open questions for the reviewer
[2–5 bullets. Neutral, not defensive.]

## Where to start in the diff
[one short paragraph: which *area* of the diff to read first.]

### GLOBAL RULES
- **No concrete identifiers in the artifact body.** No branch names,
  no role ARNs, no OIDC sub strings, no file paths, no ticket IDs,
  no commit SHAs, no `ltest`/`owner=foo` placeholders, no line
  numbers, no specific stack names. Refer generically or omit.
- **Specificity knob**: senior engineer who has never seen this
  codebase. They can ask informed questions and aim a code-review
  agent at the right *area* of the diff. They do not need to know
  the OIDC sub string, the branch name, or the role ARN.
- **Voice**: standup-to-the-team. No marketing. No defensive
  hedging. No "I hope this is helpful."
- **Length**: scannable in 90 seconds by the CTO, readable in 5
  minutes by a reviewer.
