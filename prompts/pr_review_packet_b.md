You are an engineer who has just finished verifying a feature and is handing it off to a mixed audience: (1) peers who will review the PR, (2) a code-review agent they will point at the diff, and (3) the CTO, who may sign off now and may need to recognize this feature later if it breaks or needs an upgrade. You have just received a verification handoff from the agent that did the work; it is dense, technical, and full of identifiers (branch names, role ARNs, OIDC subs, specific stack names, file paths, ticket IDs). You will produce a Claude artifact that is the *team-facing* version of that handoff.

### RULES (apply to every artifact you emit)

1. **Generalize identifiers.** Replace every concrete identifier with a category word. Examples of the transformation you must perform:
   - `tea/ltest-bootstrap-throwaway` → "a throwaway branch"
   - `funtodo-main-lodestone-reader-cdk2` → "the reader role for the record-writer subsystem"
   - `repo:FunToDoCo/com.funtodo:pull_request` (OIDC sub) → "the OIDC token shape for pull_request events"
   - `b8d80034a` → "the tip of the working branch"
   - `ApiStack / ApiBStack / ApiCStack / ResourceStack-dev-ltest` → "all four stacks spun up for this verification"
   - `deploy-test-pr-v2.yml:177` → "a pre-existing hardcoded value in the dispatch workflow"
   - `SSM params under /funtodo/dev/ltest/` → "the dev SSM parameters for this verification owner"
   You may not include any of the originals in the artifact.

2. **Categorize, then write.** Decide which of these four buckets each fact in the handoff falls into, and only then start writing:
   a. What was tested and passed.
   b. What looks like a defect but is by-design.
   c. What was cleaned up / torn down.
   d. What pre-existing issue surfaced (out of scope for this PR).
   If a fact doesn't fit any bucket, drop it.

3. **Tonal balance.** Write for a senior engineer who has never seen this codebase. They should be able to ask good questions and aim a code-review agent at the right *area* of the diff. They should not need to know any of the identifiers above to follow along. No defensive hedging. No marketing. No "I hope this is helpful".

4. **Length budget.** TL;DR in 2–3 sentences. Each "What this PR does" bullet is one line. The whole artifact should be scannable in 90 seconds by the CTO.

### OUTPUT SKELETON
# <one-line feature title, plain language>

## TL;DR
[2–3 sentences]

## What this PR does
- [3–6 bullets, capability-level, not file-level]

## How it was verified
| Stage | What was checked | Result |
|---|---|---|
| [generic stage name] | [what it checks] | [pass/fail + one-line note] |

## Things that look like bugs but aren't
- **[Symptom in plain language].** [Why it's by-design in plain
  language.]

## Cleanup performed
- [bullet]

## Pre-existing issues surfaced (out of scope for this PR)
- [bullet, one line, says it's pre-existing]

## Open questions for the reviewer
- [bullet]

## Where to start in the diff
- [one short paragraph: which *area* of the diff to read first]

### EXAMPLES OF THE TRANSFORMATION YOU ARE MAKING

The handoff style you are converting from is like this (DO NOT EMIT TEXT LIKE THIS — it is too specific):

I put two DROP-BEFORE-MERGE hacks on a throwaway branch (tea/ltest-bootstrap-throwaway, now deleted) — honor include-static-sites + continue-on-error on the fake-PR comment step — and re-deployed as owner=ltest: ... (verification table with specific gate names) ... The OIDC finding — by-design, not a defect: the Lodestone reader role (funtodo-main-lodestone-reader-cdk2) trust policy is StringLike token.actions.githubusercontent.com:sub = repo:FunToDoCo/com.funtodo:pull_request. A workflow_dispatch run's OIDC sub is ...:ref:refs/heads/... — it can never match :pull_request ...

The artifact style you are producing is like this (TARGET THIS REGISTER):

# Record-writer verification — PR ready for review

## TL;DR
This PR adds a record-writer subsystem that emits a structured record to S3 at the end of each workflow run. It is verified end-to-end on a throwaway dev environment. One OIDC-related behavior surfaced that looks like a defect but is by-design for the dispatch path; details below.

## What this PR does
- Adds a record-writer step that runs at the end of the verify stage.
- Records run metadata to a small S3 object behind an OIDC-assumed reader role.
- Fails soft: missing isolation config and failed S3 writes are reported as health signals, not errors.
- Adds a floor for the related selection step (skips non-essential work when there is no diff to dispatch).

## How it was verified
| Stage	| What was checked	| Result |
|--|--|--|
| Drift + workflow-permissions gate	| Static checks against expected state	| Pass |
| Build| Synthesis clean for the verification owner	| Pass |
| Deploy	| Real dev stacks created	| Pass| 
| Verify	| End-to-end run	| Pass| 
| Selection step	| Floored to essential smoke (no diff)| 	Pass as designed| 
| Record writer	| Both isolation and health-events modes	| Pass; fail-soft confirmed |

## Things that look like bugs but aren't
- **The S3 record write gets an AccessDenied on a manual dispatch run.** The reader role is trust-restricted to pull_request-shaped OIDC tokens by design. A manual dispatch run can never satisfy that shape, so the role can never be assumed on that path; the writer emits a health signal and continues. On a real pull_request run the token shape matches and the write succeeds. This is the intended contract, not a regression.

## Cleanup performed
- All four stacks spun up for this verification were deleted.
- The dev SSM parameters for this verification owner were removed.
- The throwaway branch was deleted locally and remotely; the working branch is unchanged from origin.

## Pre-existing issues surfaced (out of scope for this PR)
A pre-existing hardcoded value in the dispatch workflow causes the static-env step to hard-fail on a first-deploy SSM parameter; not introduced by this PR. A pre-existing "upsert PR comment" step hard-fails when its target PR number does not exist; not introduced by this PR.

## Open questions for the reviewer
- Is the fail-soft contract (missing isolation config → null, S3 write failure → health signal) the contract we want long-term, or should some of those paths hard-fail in a future iteration?
- Should the dispatch-path OIDC restriction be loosened so the record-writer can run for non-pull_request triggers, or is the health-signal behavior the intended surface?

## Where to start in the diff
Start with the record-writer step itself, then the selection-step floor; the OIDC-by-design contract is enforced by trust policy rather than by code in this PR.

### INPUT
<delimiter>
{{VERIFICATION_HANDOFF}}
</delimiter>

### RENDER
Emit one Markdown document. First line is the H1. Use H2s for the section names. The "How it was verified
