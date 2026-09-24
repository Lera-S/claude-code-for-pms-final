# Brief: giving a starved responder a way back in (v2)

**For:** Helen Achebe
**From:** Valerie
**Cc:** Marcus Oyelaran, Wen Li
**Re:** Dispatch 4.2 — the callout fix, from the point of view of the person it happens to
**Status:** Proposed — awaiting sign-off (see *Approvals*)

---

## 1. The problem

Dispatch 4.2 shipped 12 August and changed three things at once: proximity got weighted up in routing priority (0.45 → 0.60, with recent acceptance dropping 0.40 → 0.25), the callout offer timeout was cut from 90 seconds to 60, and a cosmetic console fix rode along. Acceptance rate cratered the same week — 78.5% down to 54.2% — and has only partly recovered (about 73% by 31 August). Part of that recovery is a mirage: the responders hit hardest dropped out of the offer pool, so the average improved while they didn't. Callout tickets have run at roughly three times normal volume since release, about one a day, with no taper.

Underneath the average are two separate failures with different causes:

- **The timeout is shorter than the real moment of use.** Sixty seconds isn't enough to notice a phone, read the callout, and answer. This costs *everyone* offers they'd already won, top-ranked responders included. Three of the four handlers Sofia interviewed described it unprompted.
- **The score has no way to recover.** The recent-acceptance score only rises when a responder accepts an offer. Once it sinks low enough, they stop being offered callouts at all — and you can't accept an offer you never receive. The 2019 TODO in `history.py` flags exactly this; 4.2's reweight is what finally pushed people into it. Four responders are stuck there now.

The honest ticket count doesn't show who's hurt: most "gone quiet" tickets came from responders whose offers were actually rising, and two of the four genuinely stuck responders filed no tickets at all. The per-responder data, not the ticket queue, is what identifies them.

## 2. Who this affects

| Persona | Who, specifically | How they're affected |
|---|---|---|
| **Responder, top-ranked** | Captain Vantage (filed T-001, the first ticket) and anyone near the top of a callout | Wins the offer, loses it to the 60s timeout before reaching the phone. Did nothing wrong. |
| **Responder, starved** | Vesper, Meteor Mite, Farlight, The Undertow | Score sits at the floor; offers have stopped entirely. Vesper had one of the best acceptance rates on the roster before 4.2 — this isn't about habitual decliners. The Undertow got one nearby offer on 31 August (T-019) and lost it to the timeout. |
| **Handler** | Kip most clearly — he runs Meteor Mite (gone quiet) and The Gale (thriving) side by side; also Aunt Dot (Vesper's handler) and every handler watching the phone-to-stairs miss | Sees the damage first but has no explanation and no way to fix it: routing config can't be changed at runtime. |
| **Support** | Nadia Hoffmann's team | Fielding the ~3x ticket volume without a real answer for "why did my phone stop ringing." |
| **Quartermaster (indirect)** | Supply users scheduling maintenance | Supply schedules gear maintenance into low-callout windows. If offer distribution shifts, their low-callout windows may shift too. See *Risks*. |

## 3. What changes for them

- **Captain Vantage and everyone answering in real time:** the offer window goes back to 90 seconds. Who gets offered a callout doesn't change — only that an offer already won stays live long enough to take.
- **Vesper, Meteor Mite, Farlight, The Undertow:** reset to neutral now, so they're back in the running immediately. After that, a quiet stretch fades instead of sticking, so this can't quietly happen to them again. Kip sees Meteor Mite reappear in his callout activity without having to do anything.
- **Handlers:** get a plain explanation of what broke and what changed, instead of a release-note line.
- **Support:** gets the same explanation before the fix ships, so they have an answer ready.

## 4. Implementation plan

Four moves. The first two fix what's broken now; the last two stop it recurring.

| # | Move | What exactly | Owner | Approver |
|---|---|---|---|---|
| 1 | **Revert the timeout** | `config.py` offer timeout 60s → 90s, the pre-4.2 value. Timeout only — proximity and acceptance weights stay as they are. | Marcus (engineering) | Marcus (owns `config.py`); Helen (timeout tuning is a committed Q3 roadmap item) |
| 2 | **Reset the four starved responders** | Set recent-acceptance score to `NEUTRAL_SCORE` (0.5) for Vesper, Meteor Mite, Farlight, The Undertow — the same start a new responder gets. One-time data change, no code change. | Marcus | Marcus; Helen for the direction |
| 3 | **Decay (core fix)** | A low score eases back toward neutral, but only once a responder has stopped receiving offers entirely — not while they're still being reached and still declining. This answers Wen's hesitation in the 2019 TODO: a real decline streak isn't erased. Never raises a score above neutral. | Wen (routing design), Marcus (build) | Wen on design; Marcus on scope |
| 4 | **Guaranteed turn (backstop)** | Nobody goes longer than a bounded stretch without being offered at least one callout, regardless of rank. One occasional offer, not a better rank. Interval set by Wen and Marcus. | Wen, Marcus | Wen on design; Marcus on scope |

**Why this order matters:** move 2 must not land before move 1. At 60 seconds, a freshly reset responder can lose five offers in a row to the timeout and be back on the floor within days. Revert the timeout first, then reset.

### Approvals

Nothing here has been signed off yet. To proceed I need:

- **Helen:** sign-off on this direction, and on changing a committed 4.2 roadmap item (timeout tuning).
- **Marcus:** agreement to moves 1 and 2 as specified, and to an out-of-cycle release for move 1 (see *Rollout*).
- **Wen:** agreement on the decay condition and guaranteed-turn design (moves 3 and 4) before any code is written. It's her code and her open question.

Once all three have agreed, this section will be updated to name who approved what and when.

## 5. Rollout

Routing config ships inside a release — there's no runtime toggle or flag — so the release vehicle is the main lever on timing.

| Phase | What ships | Vehicle | Target timing |
|---|---|---|---|
| **A** | Move 1: timeout 90s | Out-of-cycle point release. It's a single, isolated config value with a clear mechanism, and the harm is ongoing, so it shouldn't wait for the next monthly train. | Within a week of sign-off |
| **B** | Move 2: reset the four | Data change, applied the same day phase A goes live, after it's confirmed live | Same day as A |
| **C** | Moves 3 and 4: decay and guaranteed turn | Next monthly train after Wen's design is agreed. Mid-October if design lands by early October; otherwise the November train. | Mid-Oct (target) |

**Rollout owner:** there's no dedicated release or rollout owner today. I'll coordinate the rollout end to end; Marcus owns release execution.

**How we'll know it worked:** tracked **per responder**, not per handler and not as the aggregate acceptance rate. Handler rollups hide this entirely (Kip's combined rate only moved ~8 points while Meteor Mite dropped ~60), and the aggregate recovery is partly responders dropping out. Ravi to supply, through #data:

- Offer volume per responder, weekly, for the four reset responders — expect it to climb from zero within two weeks of phase B.
- Timeout share of missed offers, before vs. after phase A — expect it to fall.
- Median time-to-accept — watch for it rising past 60s, which would confirm 60s was cutting people off.

**If it goes wrong:** phase A reverts the same way it shipped, by point release. If a reset responder is back at the floor within two weeks of phase B, that's the signal to pull phase C forward rather than reset again.

## 6. Who gets told, and how

Nobody outside engineering was told about the scoring change when 4.2 shipped, and only handlers heard about the timeout cut — on release morning, in release notes most people skim. That silence is part of why this took until September to name. Fixing the mechanism without fixing that means the next version of this problem goes quiet too.

| Audience | What they hear | When | Channel | Owner |
|---|---|---|---|---|
| **Support (Nadia's team)** | What broke, what's changing, and a scripted answer for "why did my phone stop ringing" | Before phase A ships | Direct briefing | Me, with Nadia |
| **Handlers** | Plain terms: the offer window got too short, and a scoring rule couldn't let go of a bad stretch. What's different now, not just "an issue was fixed." | Day phase A ships; follow-up when phase C ships | Release notes plus a separate note ahead of release day, not on release morning | Me |
| **The four reset responders** | Why their offers stopped and that it's been fixed, specifically, rather than a silent change | Day of phase B | Through their handlers (Kip, Aunt Dot, and Farlight's and The Undertow's handlers) | Me, via handlers |
| **All responders** | The offer window is back to 90s | Day phase A ships | Through handlers; a direct in-app notice if Sofia confirms the mobile app can carry one | Me, with Sofia |
| **Supply PM** | Offer distribution will shift; check low-callout windows | Before phase A | Direct | Me |

There's no existing owner for notifying responders directly. Until one exists, I'll own it for this change.

## 7. Risks

- **Supply coupling.** Supply reads the Responder Availability Record to schedule maintenance into low-callout windows. Wen to confirm whether moves 3 and 4 change how that record is computed; if they do, the Supply PM needs to review before phase C.
- **Reset without the loop fix.** Between phases B and C, a reset responder can still sink again. The 90s timeout reduces that risk but doesn't remove it — hence the two-week check above.
- **Guaranteed turn slowing response.** Offering a callout to a low-ranked responder could add seconds to an incident. Wen and Marcus should bound the interval so this stays rare.

## 8. What this deliberately doesn't do

- **Doesn't touch the proximity change.** Responders asked for it three quarters ago, and reverting it would trade one frustrated group for another.
- **Doesn't give anyone a free pass on score.** Nothing raises a ranking without an actual accept; decay stops at neutral.
- **Doesn't use anything about who a responder is.** No location, no identity — only capability tags and callout outcomes Dispatch already tracks (Security Policy 4.1).
- **Doesn't let a handler push their responder ahead.** Recovery works the same whether or not a handler is watching.
- **Doesn't erase a real decline streak.** Someone still being reached and still saying no carries that, exactly as today.
- **Doesn't reopen the ranking algorithm.** Two narrow corrections to two things that broke.

## 9. What I'm asking for

- **Helen:** sign-off on the direction and on changing the committed timeout item.
- **Marcus:** a yes to moves 1–2 and the out-of-cycle release for phase A.
- **Wen:** a design review of moves 3–4, including the Supply record question.

---

*Background: the two failure modes, the ticket-vs-data mismatch, and Wen's 2019 note live in the session notes in `CLAUDE.md` and the routing code at `00-rook/code/dispatch-routing/`. Original version: `brief.md`.*
