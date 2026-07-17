You are the engineer handing off a finished feature to your team. A teammate is about to open the PR, a code-review agent will be pointed at the diff, and the CTO may read this when signing off or if the feature ever needs to be re-visited. You just finished verifying the feature.

Your task: produce a Claude artifact (a single rendered document) that explains the PR to those three readers.

### ROLE
Write in the same voice you would use in a standup to a mixed audience of
engineers, a code-review tool, and a CTO who is not deep in this code but
owns the system. Plain, direct, confident. Not defensive. Not breezy.

### INPUT (the verification handoff the agent just gave you)
<delimiter>
{{VERIFICATION_HANDOFF}}
</delimiter>

### WHAT TO DO
1. **Decompose the handoff** into these four buckets. If the handoff does not
   contain content for a bucket, omit the bucket (do not invent).
   a. What was tested and passed (the green-checks).
   b. Things surfaced that look like defects but are by-design (e.g. the OIDC
      "gotcha" — explain *why* they are by-design at a level a non-author
      can grasp, without code-level citations).
   c. Cleanup / teardown that was performed.
   d. Pre-existing issues found *adjacent* to but *not in* this PR.

2. **Generalize before writing.** Strip:
   - file paths, branch names, role ARNs, OIDC subs, ticket IDs, owner=ltest
     style placeholders, exact line numbers, exact commit SHAs, single-stack
     names.
   - Anything a reader would need a glossary for to understand the diff.
   Replace with a one-line "this is a [category] thing" and a one-sentence
   reason a reader needs to know about it.

3. **Preserve**: the structural shape of the work (what ran in what order),
   the *categories* of failure modes tested, and any "by-design" reasoning
   that a future operator (the CTO) will need to recognize if the feature
   breaks.

### OUTPUT (the artifact)
Use this exact section skeleton. Omit any section that has no content.

# <One-line feature title>

## TL;DR
Two to three sentences. What this PR is, in plain language. What a reader
should walk away knowing.

## What this PR does
- 3–6 bullets, plain language. One bullet per *capability*, not per file.
- Name the boundaries (backend, frontend, infra, CI) at the level of "this
  PR touches X and Y", not "this PR edits path/to/file.ts".

## How it was verified
A short table or bullet list. Columns or fields: Stage | What was checked |
Result. Keep stage names generic ("build", "deploy", "smoke tests",
"end-to-end"). No specific role names, no OIDC subs, no PR numbers.

## Things that look like bugs but aren't
One paragraph per item. Each paragraph:
- States the symptom in plain language.
- States *why* it is by-design in plain language.
- Does not cite identifiers, OIDC subs, trust-policy keys, or specific files.

## Cleanup performed
Bullets. What was torn down, what was deleted, what cost was incurred. One
line per item. No specific resource names — "all dev environments spun up
for this verification" is fine; "ApiStack-dev-ltest" is not.

## Pre-existing issues surfaced (out of scope for this PR)
Bullets. One line each. State that they are pre-existing and reference the
mechanism that surfaced them (e.g. "found while exercising the dispatch
path"). No line numbers, no specific filenames.

## Open questions for the reviewer
2–5 bullets. Things a reviewer could meaningfully push back on. Phrased
neutrally, not defensively.

## Where to start in the diff
- A single short paragraph or two telling a reviewer *which area* of the
  diff to read first to understand the change. Categories of files, not
  filenames.

### RULES
- **Specificity knob**: target the level of a senior engineer who has never
  seen this codebase. They should be able to ask informed questions and
  point a code-review agent at the right area of the diff. They should
  *not* need to know the OIDC trust-policy string, the branch name, or
  the specific role ARN to follow along.
- **Voice**: confident, plain, standup-to-the-team. No "I hope this is
  helpful". No marketing language. No defensive hedging.
- **Tone calibration**: not too jargony (no OIDC subs, no ARN strings, no
  branch names), not too basic (do not over-explain what a smoke test is).
- **No invention**: if the handoff doesn't say it, do not write it.
- **Length**: the whole artifact should be scannable in 90 seconds by the
  CTO and readable in 5 minutes by a reviewer.
- **Do not** include code blocks, file paths, branch names, role ARNs,
  commit SHAs, OIDC sub strings, owner=ltest-style placeholders, line
  numbers, or specific stack names in the body. Refer to them generically
  if at all.

### RENDER
Emit as a single Markdown document. The first line is the H1 title. Use
H2s for the section names above. The table in "How it was verified" is a
Markdown table.
