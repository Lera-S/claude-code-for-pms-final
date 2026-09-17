# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Read and write only within this directory.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

Notes for helping Valerie, PM for **Rook Dispatch**. Source: `00-rook/company/`.

### The company

Rook Industries builds coordination and provisioning software for the
"protective-response sector." Customers are independently-operating masked
**responders** and the **handlers** / **quartermasters** who support them.
Rook does not employ responders. Founded 2014, HQ at Site Aleph (remote);
offices in Berlin, Singapore, Cornwall. ~241 staff, mostly remote.
Subscription revenue, priced per active responder.

**Confidentiality (hard rule):** responder cover identities are never stored
in production. Rook holds only capability tags, availability windows, callout
history — never a mapping to a legal identity, contractually. Do not design
anything that assumes we can identify a responder. See Security Policy 4.1.

### The two products

- **Rook Dispatch** (our product) — responder coordination: availability,
  proximity, callout routing, acceptance. Handlers use a **web console**;
  responders use **native mobile**. Current release **4.2**.
- **Rook Supply** — gear provisioning: requisitions, maintenance, failure
  reports. Handlers + quartermasters.

Monthly release train; point releases numbered 4.x. Routing config ships
_in_ the release — handlers can't tune it at runtime. Support has 3 tiers;
tickets filed from inside an active callout skip the queue.

**Dispatch ↔ Supply coupling:** Dispatch writes the **Responder Availability
Record**; Supply reads it (never writes) to schedule gear maintenance into
low-callout windows. Changes to how Dispatch computes that record flow
straight into Supply's scheduling with no change on their side — factor this
into any routing/availability work.

### Core Dispatch flow

Incident enters console → Dispatch ranks available responders into a
**routing priority** order → **callout offer** sent to top responder's mobile
→ they accept / decline / time out → on timeout or decline it moves to the
next → acceptance marks them engaged and assigns the incident.

**Routing priority inputs:** proximity (travel-time estimate), current
availability, capability match, recent acceptance history. Declining or
timing out lowers the recent-acceptance component, which lowers priority on
later callouts until it recovers.

**Metrics:** **callout acceptance rate** (headline; share of offers accepted,
reported weekly in aggregate), **time-to-accept** (median secs offer→accept),
**coverage gap** (incidents where no available responder had the required
capability tags — "nobody could go," distinct from low acceptance).

### People

| Person | Role | Notes |
|---|---|---|
| Helen Achebe | Director of Product (Dispatch & Supply), Chicago | Valerie's manager; owns roadmap & commitments. "Gives room." |
| Priya Raghunathan | Prior Dispatch PM, departed 21 Aug 2026 | Sole Dispatch PM for 14 months. Left a handover doc. |
| Marcus Oyelaran | Engineering Manager, Dispatch, Chicago | First stop for anything uncertain; blunt, reliable. |
| Wen Li | Staff Engineer, routing, Berlin | Built the ranking logic. No good written doc exists — talk to her. Was on PTO 14–24 Aug. |
| Sofia Marino | Product Designer, Dispatch, Chicago | Console + phone app. |
| Nadia Hoffmann | Support Lead (both surfaces), Berlin | Sees complaint volume first; worth a standing 15 min. |
| Ravi Menon | Data Analyst (both surfaces), Singapore | Owns the weekly "how often are responders answering" numbers. Requests go through **#data**. |

### Where things stand (as of ~Sept 2026)

- **4.2 shipped 12 Aug 2026** and is the live issue. Three things moved at
  once: (1) **"change to who gets pinged"** — proximity weighted up vs. recent
  acceptance history (a 3-quarter-old ask from responders over wide
  geographies); (2) **callout offer timeout cut 90s → 60s**; (3) console
  filter persistence (cosmetic).
- **Since 4.2: acceptance rate down, callout tickets ~3x normal.** Two ticket
  themes, split ~2/3 : 1/3 — "my phone never goes off anymore" (unexplained)
  vs. "phone buzzes but the offer's gone before I can answer" (explained by
  the shorter timeout).
- Open question from Marcus: does the "who gets pinged" change also apply to
  responders who've been declining jobs, or only everyone else? Config
  doesn't distinguish — unclear if that was a decision or a side effect.
- **Priya's read** (in handover): mostly seasonal (August is always soft),
  expects recovery in September; look there before pulling the routing change
  apart. Strongly against letting this become a "revert 4.2" conversation —
  the change was asked for; reverting just angers a different group.
- Team plan: give the new PM a week to form their own view, then regroup on
  the 4.2 picture. Nadia is preparing a ticket breakdown.

### Roadmap (Q3 2026, owner Helen; committed items are locked, changes via Product)

- Dispatch 4.2 (committed): change to who gets pinged; **Availability
  Confidence** (confidence score next to a responder's stated availability;
  driven by support escalations); ping timeout tuning.
- Supply 4.3 (committed): requisition approval chains.
- Q4 (exploring): Supply handler phone app; **shared cover between
  responders** (aka **mutual aid** — cross-region cover when the primary is
  unavailable; not supported today).

### Carry-over / to-do from Priya's handover

- Some items were squeezed out of 4.2 — Helen conversation needed on which
  are still Q3 commitments and which quietly aren't. Hasn't happened.
- **No written description of how Dispatch decides who gets pinged exists.**
  Priya asks the new PM to write it.
- Expect noise tickets from the filter-persistence change; don't let it eat
  the first month.

### What we worked out (session 2 · 8 Sep 2026)

- **The routing code exists** at `00-rook/code/dispatch-routing/` (owner: Wen
  Li) — it's the de facto spec since no written one exists. Post-4.2 values in
  `config.py`: proximity weight 0.45 → **0.60**, recent-acceptance weight 0.40
  → **0.25**, offer timeout 90s → **60s**, decline penalty (0.12) > acceptance
  credit (0.08), 45-min proximity horizon (zeroes the score past it).
  `history.py` carries a **2019 TODO**: the recent-acceptance score never
  decays — a starved responder can't recover rank on their own.
- **4.2 has two independent failure modes**, not one: (a) proximity reweight +
  no-decay loop → a few far / low-score responders get **starved** of offers;
  (b) the 60s timeout hits **everyone regardless of rank** — top responders
  lose reachable callouts too (Captain Vantage, a routing "winner," filed the
  first ticket, T-001). The timeout is *not* part of the feedback loop.
- **Data vs. tickets contradict each other.** `00-rook/data/callout-history.csv`
  (16 responders, weekly, ends 8/31, unattributed — likely Marcus's "rough"
  pull) shows only **4** truly starved: Farlight, Meteor Mite, The Undertow,
  Vesper. Several responders with "gone quiet" tickets (Nightwell, Ironvale,
  Stormwrack, Sgt. Falkirk, Cindermark) actually show pings *rising*. Ticket
  volume ≠ real impact; two of the genuinely starved filed no tickets.
- **Aggregate acceptance:** ~77% pre-4.2 → ~54% release week → recovering to
  ~73% by 8/31 — but partly a composition effect (starved responders dropped
  out of the offer pool). Distribution stays broken.
- **No authoritative metrics exist in `00-rook`:** Ravi's official weekly
  acceptance report isn't here; no year-over-year data to test "August is
  always soft"; no decline-vs-timeout split, time-to-accept, or coverage-gap
  numbers. The one YoY check anyone did (Renata Kovač, T-014) found this year
  *worse* than last.
- **Sofia's interviews** (`00-rook/feedback/interviews/`, 2–5 Sep, 4 handlers,
  about the console redesign) — 3 of 4 unprompted described offers expiring
  before a responder can physically reach the phone ("phone to stairs"). They
  **postdate 4.2** and did not inform it; the roadmap driver was "Internal."
- **Current lean:** 4.2 is the cause, not seasonality (Priya's read doesn't
  fit the data shape). The **timeout cut (60→90) is the clean candidate for a
  temporary partial revert** — isolated, clear mechanism, broad impact. The
  proximity reweight is entangled with the loop; Priya's warning that
  reverting it "trades one angry group for another" has merit *there*.
- **To revert anything:** config ships in a release — no runtime toggle, no
  flag. Needs an out-of-cycle point release or the next monthly train
  (~mid-Sep). `config.py` says don't touch without **Marcus**; a change to a
  committed roadmap item goes through **Helen**; loop **Wen** on interactions.
  There is no dedicated release/rollout owner and no owner for notifying
  responders (only handlers got the 4.2 timeout change, via release notes on
  release morning).
- **Planned next step:** one-pager to Helen + Marcus proposing a timeout-only
  revert, with Nadia's ticket-theme split and Ravi's real acceptance numbers
  gathered first.

### What we worked out (session 3 · 10 Sep 2026)

- **Read both feedback folders in full.** Interviews
  (`00-rook/feedback/interviews/`, 4 handlers, Sofia's console-redesign study
  2–5 Sep): Ambrose/Captain Vantage, Aunt Dot/Vesper, Halloran/Sgt. Bulwark,
  Kip/Meteor Mite + The Gale. Tickets (`00-rook/feedback/tickets/`, T-001–T-025):
  **all post-release** (13 Aug–5 Sep), ~1/day, not tapering.
- **The interviews carry debt that never became a ticket.** 3 of 4 describe the
  timeout "phone-to-stairs" miss unprompted; 2 of 4 (Dot, Kip) describe
  feast-or-famine, only Kip frames it as one pattern. UX gripes, none ticketed:
  no dark mode, status text too small, no handler-side callout alert,
  filter-persistence distrust, buried capability legend, per-responder alert
  sounds. Halloran's interview is mostly **Supply** — requisition approval queue
  too slow + priority field inert, field failure reports unacknowledged, catalog
  search — an early signal for 4.3.
- **Ticket shape:** "gone quiet" 14/25 · pure-timeout 4/25 (all before 27 Aug,
  then stop) · "starved *then* lost the rare offer" 5/25 (all 24 Aug+, still
  rising) · intermittent 2/25. Filed by 22 people (12 handlers, 10 responders),
  only ~12 distinct responders; every responder-filed ticket has a matching
  handler ticket; only Captain Vantage (T-001) and Farlight (T-018) are
  handler-only. Cited silence lengthens week over week (6 days → ~2 wks →
  ~1 month) — the starvation is persistent, not self-correcting.
- **Nothing has shipped since 4.2** — no hotfix, point release, or config change;
  no engineering bug opened; tickets sit in the normal support queue. Earliest
  fix vehicle is the mid-Sep monthly train.
- **Interviews vs tickets diverge, and neither cleanly IDs the victims.** Timeout
  is loud in interviews but only 4/25 tickets and fading; starvation dominates
  tickets (19/25 incl. compound) but is a vague aside in interviews; "is my
  account broken / can't see my own status" is ~12/25 tickets, near-absent from
  interviews. Tickets are noisy with false positives (Nightwell, Ironvale,
  Stormwrack, Falkirk, Cindermark all show pings *rising* in the data);
  interviews caught 2 genuine victims (Meteor Mite, Vesper) who filed no tickets.
- **The "text/sound change → missed pings" theory is dead:** 4.2 changed no font
  size and no notification sound. Missed callouts = timeout + starvation only;
  the honest framing is an interaction effect (4.2 raised the cost of
  pre-existing friction).
- **Draft conclusion for the team:** "the trouble with 4.2 is that it broke the
  callout in two independent ways and shipped both without telling anyone" —
  (1) 60s timeout shorter than the real moment of use; (2) proximity reweight
  silently redistributed who gets offered callouts; (3) neither change
  acknowledged, at ship time or at runtime.
- **Still to rule out:** the 4.2 defect fix "duplicate push notification on
  re-offer" — confirm it isn't suppressing legitimate offers ("phone never goes
  off" is only partly explained by starvation). Also T-002 implies Corporal
  Ashgrove's quiet started ~8 Aug (pre-4.2) — check against real data.

### What we worked out (session 4 · 15 Sep 2026)

- **Handler-level rollups can hide the starvation loop entirely.** Kip is the
  one handler running two responders — Meteor Mite (starved, −60pp) and The
  Gale (thriving, +ping volume). Pooled at the handler level his combined rate
  only drops ~8pp, camouflaged in the middle of the pack. Any metric organized
  by handler instead of by responder will miss this.
- **Cross-checked all 25 tickets against the CSV directly.** Of 20 "gone
  quiet"-type tickets, only ~5 (Farlight, The Undertow ×3, partial matches for
  Corporal Ashgrove/Halfmoon) actually correspond to a real decline. The rest
  (Nightwell, Stormwrack, Ironvale, Sgt. Falkirk, The Longcast, most of
  Cindermark's) were filed in weeks that responder's own ping volume was flat
  or *rising* — ticket volume is not a usable proxy for who's actually hurt
  here. Vesper and Meteor Mite, two of the four worst-hit, filed **zero**
  tickets; they only surface as asides in Sofia's design interviews.
- **Timeline:** the pooled acceptance rate craters in the release week itself
  (78.5%→54.2%, week of 8/10) and the starved four are already dipping that
  same week. The first ticket (T-001, timeout) lands the very next day: 13
  Aug. The first "quiet"-type ticket doesn't land until 18 Aug — six days
  later, and it's a false positive (Nightwell, whose volume was rising).
  Tickets then arrive at a steady ~1/day through 5 Sep with no spike at the
  real peak (week of 8/17) and no taper as the pool partly recovers.
- **Corrected a wrong hypothesis — don't repeat it:** pre-4.2 acceptance rate
  is *not* a "buffer" that protected the top responders. Vesper had the
  2nd-highest pre-release rate of all 16 responders (81.9%) and still became
  the most starved (→6.7%); Captain Vantage had the lowest pre-release rate of
  the top 4 (73.4%) and thrived. The CSV can't reconstruct the actual
  `recent_acceptance` score (it's sequence-dependent, not a rolling ratio), so
  this can't be tested cleanly from weekly totals — treat "who was protected
  and why" as still open.
- **No geolocation/region data exists anywhere in this repo** for responders —
  `availability.py`'s `travel_time_minutes()` is a stub, and the CSV has no
  location field, consistent with the cover-identity confidentiality rule. One
  indirect signal (Kip's interview) has Meteor Mite and The Gale in the *same
  city* with opposite outcomes, which argues against simple regional
  clustering as the explanation.
- **Read the routing code end to end** (`history.py`, `routing.py`,
  `offer.py`) to answer "how does a starved responder recover": there's no
  built-in path. The score only rises on an *accepted* offer, but
  `offer.py`'s `dispatch()` won't reach them unless everyone ranked above them
  on that call also declines/times out — a structural catch-22 the 2019 TODO
  never resolved. The one way out is a specific incident close enough that
  proximity alone (weighted 0.60) carries them despite a floor-level score —
  which is what happened to The Undertow on 31 Aug (T-019): his one offer that
  week arrived, then was lost to the 60s timeout before he could answer.
- **Built an 8-slide briefing deck** for Helen (title, what-shipped, weekly
  chart, two-failure-modes, redistribution chart, tickets-vs-data mismatch,
  timeline, recommendation/asks) — sent to Valerie, not committed to the repo.
  Recommendation in it: timeout-only partial revert now; proximity reweight
  fix deferred, decided separately with Wen.

### What we worked out (session 5 · 17 Sep 2026)

- **Marcus's open question is answered: the reweight is not targeted.**
  `config.py`'s weights are global constants with no branching on a
  responder's history — `routing.py`'s `score()` applies the identical
  formula to everyone. Confirmed against real data too: Vesper had one of
  the best pre-release acceptance rates in the roster (~82%, not a habitual
  decliner) and still became the most starved — proof this is a proximity
  effect, not something aimed at people who'd been turning jobs down.
- **Why ping volume, not just accept rate, is the metric that craters:**
  `offer.py`'s `dispatch()` stops at the first accept — everyone ranked
  below that point never receives an offer at all for that incident, not a
  decline or timeout, just never reached. Explains why a starved
  responder's per-offer miss rate can look ordinary while their total
  offer count collapses to near zero — volume and behavior are separate
  axes.
- **The penalty math itself didn't change in 4.2** — `DECLINE_PENALTY`
  (0.12) and `ACCEPTANCE_CREDIT` (0.08) are unchanged since before the
  release; only the weights and timeout moved. From neutral (0.5), 5
  consecutive misses wipes a responder to the score floor; climbing back
  takes ~7 consecutive accepts, and — per T-019, The Undertow, 31 Aug —
  each one independently needs a nearby incident, a high-enough rank to be
  reached, and a fast-enough answer. He got the one rare nearby offer and
  still lost it to the 60s timeout.
- **Newcomers start at `NEUTRAL_SCORE` (0.5), not 0** — better off by
  default than a starved veteran. There is no onboarding boost, comeback
  bonus, or protection anywhere in the code for responders returning from
  a long standby — the system just doesn't penalize time spent properly
  marked unavailable; it doesn't help them get back in either.
- **No per-call log or location data exists anywhere accessible to us** —
  reconfirmed while probing this further; `travel_time_minutes()` is a
  stub. A real per-call diagnosis would need to come from Wen or whoever
  owns the production event log, not this repo.

### Vocabulary quick-reference

**Responder** (independent field operator, not staff) · **Handler** (manages a
responder/small group; the actual product user) · **Quartermaster** (Supply:
owns equipment stock + approvals) · **Cover identity** (responder's public
persona; no legal-identity mapping held) · **Callout** (request for a
responder to attend an incident; unit of work) · **Callout offer** (a callout
shown to one responder, awaiting accept/decline) · **Decline** vs **timeout**
(both pass the callout on, but distinct in the data) · **Capability tag**
(competency on a responder record, matched to incident needs: flight,
structural-entry, hazmat-tolerant, cold-weather, aquatic, crowd-management,
de-escalation) · **Routing priority** (the ranking score) · **Responder
Availability Record** (shared availability record, written by Dispatch, read
by Supply) · **Mutual aid** (cross-region cover; not built) · **Requisition**
(handler's equipment request → quartermaster) · **Field failure report**
(gear-failed-in-use account; can pull maintenance forward) · **Service
interval** (maintenance cadence for an item).
