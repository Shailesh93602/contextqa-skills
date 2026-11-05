---
name: cqa-ship-bug
description: Use when one bug ticket should go all the way from report to verified fix — reproduce it with ContextQA, fix the code, open a pull request, run the tests the PR affects (fixing failures, at most 3 rounds), wait for a person to merge, then verify the fix on the ticket. Chains /cqa-bug-repro, /cqa-debug, /cqa-impact and /cqa-verify-fix. Triggers on "ship CON-123", "take CON-123 end to end", "reproduce, fix and verify CON-123", "/cqa-ship-bug".
---

# ContextQA Ship a Bug

One ticket, start to finish. The agent writes the code; ContextQA tests it on a
real browser; a person merges. After every stage, post a one-line status with
how long the stage took (use ContextQA's timestamps where they exist).

## Step 0 — Ready?

If `/cqa-init` has not passed in this session, run it. Also confirm:

- a tracker is connected (`/cqa-integrations`), and the reproduction's
  environment serves the app this repo builds;
- `gh` (or `glab`) is signed in, so a PR can be opened;
- for Step 3, the repo is enrolled for PR impact (`/cqa-impact`).

Stop on anything missing and say the one action that fixes it.

## Step 1 — Reproduce

`/cqa-bug-repro`, the platform run that reports on the ticket. Go on only on
`REPRODUCED`.

- `NEEDS_INFO` → ask the user for the missing facts, `submit_info`, wait again.
- `NOT_REPRODUCED` → stop. Check the environment URL before concluding the bug
  is gone.
- `INTERNAL_ERROR` → stop and report. Do not retry blindly.

## Step 2 — Fix and open the PR

The reproduction is the spec: the failed step and expected vs actual say what
"fixed" means.

1. **Find the cause.** Trace from the screen, route or field named in the
   failed step to the code that produces the wrong value. Name the file and
   line before changing anything. Unsure after reading → say what you checked
   and ask; never ship a guess.
2. **Smallest fix** that turns actual into expected. Follow the repo's own
   conventions. Add or update a test when the repo has tests, and run them.
3. **Don't touch the data ContextQA tests against** while checking — no
   requests that create, edit or delete records in that environment.
4. **PR.** Branch from the up-to-date default branch, e.g.
   `fix/<ticket>-<short-name>`. Commit message and PR title start with the
   ticket key. PR body: the bug in one line, the cause (file:line), the fix, how
   it was checked, and the reproduction's evidence link. Push and open it; report
   the URL.

Never force-push. Never commit secrets or `.env` changes.

## Step 3 — PR impact

`/cqa-impact`, the PR pipeline: trigger the analysis for the new PR, then run
the impacted cases against an environment that serves **the PR's build**.

- A failure → read the evidence. Product regression → fix it on the same
  branch and push; broken test → that is `/cqa-debug`, with the user's OK.
- At most **3 rounds**. Still red after 3 → stop and hand over with all three
  attempts.
- Nothing impacted is not the same as safe. Say which one it is.

## Step 4 — A person merges

Tell the user: PR link, impact verdict, what is left. **Wait** for them to
review, merge and deploy. Never merge yourself, and never approve your own PR.

## Step 5 — Verify

`/cqa-verify-fix` once the user says the fix is live.

## Step 6 — Summary

Ticket; each verdict with its time (reported → reproduced → PR → impact green
→ verified); the PR link; the regression `test_case_id`; credits used (one
reproduction, the impacted runs, one re-check); and how many times a person
had to step in.

## Rules

- Each stage starts only when the one before it has a real result.
- Credits: ask before anything beyond one reproduction, the impacted runs of
  this PR and one re-check.
- Quote ContextQA's own text for verdicts and failures; never fill gaps with
  a guess.
