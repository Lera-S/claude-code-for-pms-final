# Brief: giving a starved responder a way back in

**For:** Helen Achebe
**From:** Valerie
**Re:** Dispatch 4.2 — the callout fix, from the point of view of the person it happens to

---

## What happened

Dispatch 4.2 shipped 12 August and changed three things at once: proximity got weighted up in routing priority, the callout offer timeout was cut from 90 seconds to 60, and a cosmetic console fix rode along with them. Acceptance rate cratered the same week — 78.5% down to 54.2% — and has only partly recovered since; callout tickets ran at roughly triple normal volume, without tapering, from release through early September. Underneath that average are two separate, unrelated failures: a shorter timeout that costs *everyone* a live offer they'd already won, and a scoring rule that lets a handful of responders' acceptance scores sink to the floor with no way back — because the only way to earn score back is accepting an offer, and a low enough score means you stop being asked at all.

Helen asked to see this in terms of who feels it, not what config value moves — so that's how the rest of it is written. Engineering says the score could be reset this afternoon and called done; this brief is about why that isn't the same as fixed. The two failures need different fixes, and this brief keeps them apart rather than folding them into one "revert."

## Who this is for

Two people, not one.

The first is **Captain Vantage** — or anyone ranked near the top of a callout who still loses it. He's fast, well-matched, close by, and gets offered the job. He just doesn't have 60 seconds to notice his phone, read the callout, and answer before it's already gone to someone else. He didn't do anything wrong. He's not the problem this brief spends most of its time on, but he's the one who filed the first ticket, and he deserves an answer that isn't about anyone's score.

The second is **Vesper**, or **Meteor Mite**, or anyone like them: a responder who, through an unlucky stretch, a few honest declines, or just geography, has dropped low enough on **routing priority** that they've effectively stopped getting **callout offers** at all. Not "declining more" — not getting asked. Vesper had one of the best acceptance rates on the whole roster before 4.2 and still ended up here. There is currently no way for a responder in this position to work their way back — the routing math only rewards accepting, and you can't accept an offer you never receive. **Kip** is the handler who notices this most clearly: he runs both Meteor Mite, who's gone quiet, and The Gale, who's thriving, side by side, and the contrast is exactly what he sees every week.

## What changes for them

For Captain Vantage and everyone answering offers in real time: the window to respond goes back to a length that matches how long it actually takes to notice a phone and answer it. Nothing about *who* gets offered a callout changes for him — only that the offer he's already won stays live long enough to take.

For Vesper and Meteor Mite: right now, a bad stretch is permanent, because the only way to earn score back is accepting an offer — and they've dropped too low to ever be offered one. The fix closes that loop directly: a quiet stretch fades instead of sticking, so getting back into the running doesn't depend on catching the one rare offer proximity alone can carry through. In practice: Meteor Mite starts showing up in Kip's callout activity again, a little more each week, instead of sitting at zero — Kip doesn't have to do anything to cause it, he just watches it happen. Vesper's score comes to reflect her actual reliability again, not a two-month-old rough patch she never had a way to shake.

## Suggested solution

Three moves, in order of how far each one reaches.

**Reset the four who are already stuck.** Vesper, Meteor Mite, Farlight, and The Undertow all have a recent-acceptance score sitting at the floor and have effectively stopped receiving offers. Whatever fixes the loop going forward, none of them should have to wait on a slow climb back — reset their score to neutral now, the same starting point a brand-new responder gets, and they're back in the running immediately.

**Core fix: let a quiet stretch fade instead of stick.** This is the exact question Wen left sitting next to the code in 2019 — should a low score ease back toward neutral on its own after enough time, or should it stay until someone earns their way out? Her own hesitation was fair: if a responder is still being reached and still saying no, decay shouldn't quietly erase that. So the fix ties the fade to the right condition — a score only eases back once a responder has stopped receiving offers entirely, not while they're still being asked and still declining. That's the distinction that makes it safe to finally build the thing she was right to wonder about. For Vesper and Meteor Mite, it means the system stops requiring an accept they can never get the chance to make — a quiet stretch heals on its own, the same as it eventually would for anyone, with no handler having to notice or step in.

**Backup: guarantee everyone a turn.** Decay is a fade, not an instant floor, so it needs a backstop under it: a rule that nobody goes more than some bounded stretch — the exact interval is Marcus and Wen's call — without being surfaced at least once, regardless of rank. It's not a boost and it doesn't skip anyone in line; it just guarantees the clock the fade depends on actually keeps moving for everyone, not only the people already near the top.

## How people find out

Nobody outside engineering was told about the scoring change when 4.2 shipped, and only handlers heard about the timeout cut — on release morning, in release notes most people skim. That silence is part of why this took until September to even name correctly, and fixing the mechanism without fixing that would just mean the next version of this problem goes quiet again too.

When this ships, both handlers and responders hear, in plain terms, what actually happened: the offer window got too short for how long it really takes to answer a phone, and a scoring rule meant to reward reliability had no way to let go of a bad stretch. Say what's different now, not just that "an issue was fixed" — including the plain fact that a quiet stretch now fades on its own instead of sticking, so people have a real reason to expect their phone to ring again rather than take it on faith. The four responders getting reset today hear why, specifically, rather than noticing a silent change the same way they noticed the silence in the first place. Nadia's team gets the same explanation, so when a responder calls in asking why their phone stopped ringing, support has a real answer instead of a guess.

## What this deliberately doesn't do

- **It doesn't touch the proximity change itself.** Weighting travel-time more heavily was something responders asked for, three quarters ago, because working wide geographies was getting them offered jobs they'd never reasonably reach. That ask was legitimate and stays in place. This fix is about giving a low-scoring responder a path back — it isn't a reason to undo why they scored low in the first place.
- **It doesn't give anyone a free pass on score.** Nothing here raises a responder's ranking without an actual accept — decay only unfreezes a stuck score back toward neutral, and the backup guarantee only means an occasional turn, not a better rank once they're in it. A responder who's actually been declining still has to earn their way up the same way as today; they're just no longer locked out of the chance to try.
- **It doesn't use anything about who a responder actually is.** No location, no identity, no history beyond what Dispatch already tracks as capability tags and callout outcomes. Nothing here needs — or is allowed to use — a mapping back to a real person.
- **It doesn't let a handler push their own responder ahead of anyone else, and it doesn't need one to.** Recovery happens the same way for every responder whether or not a handler happens to be watching closely — Kip doesn't request anything, he just sees it happen.
- **It doesn't erase a real decline streak for someone who's still being reached.** The fade only starts once a responder has stopped receiving offers entirely — someone still being asked and still saying no keeps carrying that, exactly as today. That's the line that answers Wen's original hesitation about building this at all.
- **It doesn't touch the timeout and the starvation loop with the same lever.** They have different causes and different people affected; treating them as one "4.2 problem" with one fix is exactly the framing that let the starvation loop go unnoticed for as long as it did.
- **It isn't a reopening of the ranking algorithm.** The backup guarantee bypasses rank for one occasional offer, the same way a handler's own judgment might if they noticed in time — it doesn't touch how proximity, capability, and acceptance are weighted against each other for everyone else. This is a narrow, specific correction to two things that broke, not a mandate to redesign how routing priority works.

## What I'm asking for

Sign-off on this direction, not the specific numbers — those still need Marcus and Wen. If the shape of it is right, next stop is Marcus for scoping and Wen for the routing design itself: it's her code, and this is the answer to the question she left open in it.

---

*Background this brief draws on — the two independent failure modes, the ticket-vs-data mismatch, Wen's 2019 note that recent-acceptance never decays — lives in the session notes in `CLAUDE.md` and the routing code at `00-rook/code/dispatch-routing/`, for anyone building the deck from this.*
