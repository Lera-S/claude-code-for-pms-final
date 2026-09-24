---
name: review-checklist
description: Review a product brief against Valerie's standard pre-circulation checklist — problem before fix, user personas, agreed implementation plan, rollout/notification plan. Use when asked to review, check, or sanity-check a brief, one-pager, or proposal before it goes further.
argument-hint: <path to brief>
---

# Brief review checklist

Review the brief at `$ARGUMENTS`. If no path was given, ask which brief to review
(don't guess). Read the whole brief before judging anything.

Check it against these four criteria, in this order, every time. Don't add,
drop, or reweight criteria.

## The four checks

1. **Problem before fix.** The brief explains the problem — what is happening,
   to whom, and the evidence — *before* it proposes any solution. Fail if the
   fix appears first, or if the problem is only implied by the fix.
2. **User personas.** The brief names the specific users the work affects and
   how each is affected (e.g. for Rook: responders, handlers, quartermasters —
   and which subset, not just "users"). Fail if personas are generic or
   missing, or if an affected group is mentioned only in passing.
3. **Agreed implementation plan.** The brief states the implementation plan and
   makes clear it is *agreed* — who signed off, or who still needs to. Fail if
   the plan is a menu of options with no chosen path, or if ownership/approval
   is absent.
4. **Rollout or notification plan.** The brief covers how the change ships and
   who gets told — release vehicle, timing, and how affected users learn about
   it. Fail if it stops at "ship it" with no rollout or communication step.

## Grading

For each check give one verdict:

- **Pass** — clearly and fully covered.
- **Partial** — present but thin, buried, or missing a piece.
- **Missing** — not addressed.

Every verdict must cite where in the brief it comes from (section heading or a
short quote under 15 words). For Partial or Missing, say in one line what would
make it pass.

## Output format

```
Brief review: <file name>

| # | Check                        | Verdict | Evidence |
|---|------------------------------|---------|----------|
| 1 | Problem before fix           |         |          |
| 2 | User personas                |         |          |
| 3 | Agreed implementation plan   |         |          |
| 4 | Rollout / notification plan  |         |          |

Overall: <Ready to go further | Needs work — N of 4 gaps>

To fix:
- <one line per Partial/Missing check>
```

Keep it to the table, the overall line, and the fix list. Don't rewrite the
brief or edit the file unless asked.
