# 01 · Origin Story — prompts

**Context:** Rook Industries makes software for superheroes and the
people who handle them.

There are two products. Rook Dispatch is the one that gets a
superhero to where they're needed when there's an emergency — it
works out who is close enough and free enough to help, and gets hold
of them. Rook Supply keeps a responder's equipment serviceable and
accounted for, so a handler is never guessing whether the gear will
hold.

You joined two weeks ago as PM on Rook Dispatch. Release 4.2 shipped
on 12 August, shortly before you arrived. Responders have stopped
answering their phones the way they used to, and complaints have
gone up sharply. You were not in the room for any of it.

At the end of the session, ask Claude Code to save the prompts you
wrote yourself below — not the starter prompt. The closing slide has
the exact prompt to paste. By Module 6 this file is a prompt library
built from your own questions.

---

### 1.

summarize 3 tickets which are part of this release and probably causing issues. Specify the goal and coverage for each

### 2.

which out of those 3 tickets affecting my product Dispatch directly?

### 3.

what is a current issue in production?

### 4.

What's contradictory or missing, not just in the company documents, but across everything in 00-rook?

### 5.

how do we know that numbers are decreasing? what was the benchmark data?

### 6.

was anything fixed or deployed after the release 4.2 that acceptance started to recover?

### 7.

were only specific people affected or there were received complaints even from people which were not part of the feedback loop?

### 8.

60 second timeout is a planned change from 90 to 60, so why it is considered as issue? Were stakeholders notified about the timeout adjustment?

### 9.

how the rollout of the optimization (from 90 to 60 seconds timeout cut) happened?

### 10.

who is responsible for rollouts and stakeholder feedback?

### 11.

should I consult with Priya regarding the temporarily rollback?

### 12.

is it change of 90 to 60 seconds cut justified by the Sofia's research?

### 13.

should I clarify my decision regarding temporality revert of the timeout with someone or I can just approach devops engineer to revert?

### 14.

is Ping timeout tuning related to any of the company goals? is there any ETA for this item?
