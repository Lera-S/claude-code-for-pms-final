# 04 · X-Ray Vision — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly. Last session the numbers showed you what happened: a
ping used to wait ninety seconds and now it waits sixty, people
missed pings they used to catch, and missing one counts the same as
turning one down — so four responders stopped hearing from us
altogether.

At the end of the session, ask Claude Code to save the prompts you
wrote yourself below — not the starter prompt. The closing slide has
the exact prompt to paste. By Module 6 this file is a prompt library
built from your own questions.

---

### 1.

create the illustration or diagram of the user journey covered by this code, so I can understand the story base on the image

### 2.

Has anything in this code changed recently? Walk me through what's different, and why it would matter to a responder.

### 3.

if user was missing the call how drastically their score would change?

### 4.

was is the same missed call penalty before the update?

### 5.

provide the score history for top responder and responder which calls droped to 0

### 6.

do you have a history of their score changing after every call?

### 7.

provide the callout history for captain vantage and vesper for every call after the release
specify call location and the responder location and the result of the call: accepted or missed

### 8.

but the balancing of the scoring logic was changed to give more score for responders closer to the necessary location, so why this data is not tracked?

### 9.

add the travel time minutes for every call received by Captain Vantage and Vesper

### 10.

what available analytics parameters could explain the drop of the calls for some responders?

### 11.

Which file has last session's numbers in it? Read that alongside this code. Does it back up what I found in Lab A? And does that answer Marcus's question above?

### 12.

why change in the closeness made such a bit impact on the ping? since according to the analytics we see that responders were equally missing the calls

### 13.

Find me the part of this code that takes points off somebody when they miss a ping or turn one down. Show it to me and explain it in plain English. Then find me every single thing in this code that puts points back on.

### 14.

Somebody has been quiet for a month. Walk me through, step by step, exactly what would have to happen for them to start getting work again.

### 15.

how this scoring system works? is it in % or just always growing number?

### 16.

if person went to vacation before 4.2 and came back in a month after. Will this person get less chances to get a ping because its score wasn't updated for a month during the vacation?

### 17.

reply like to a 5 year old and shortly how vacation could affect the score

### 18.

but the person will not be able to increase its score while it is in vacation. How it can affect him?

### 19.

how the code protects people which were in a stand by for a long time and come back still having good chances to get the work again?

### 20.

how the new commers can start getting calls if they are starting from 0?
