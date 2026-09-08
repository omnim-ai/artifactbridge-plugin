# Recipe — proposal summary

The `summary` argument of `artifactbridge_propose_document_patch` (including a
revision passed with `revises_review_request_id`) is posted as the proposal's
first comment. A reviewer reads it before anything else and may read nothing
else.

Write the summary so a reviewer understands the proposal from the first two
lines and can stop there. Use this structure exactly.

## Structure

    <lead: one or two sentences>

    **Changes**
    - <one line per change>

    **New in this version**
    - <one line per addition>

    **Open questions**
    - <one line per decision waiting on the reader>

    Based on <the source>.

## The lead

One or two sentences, 35 words or fewer. Name the area the changes touch and
what was wrong with it. Do not lead with the biggest single change — lead with
what the set of changes is about.

Example:

> The first-run plan assumed too much about invited members. This version fixes
> what an invited member can do and what they see, and removes the limits that
> shaped the old answer.

## The bullets

One line per change. 20 words or fewer. Each line says what is different now
and why. Write full sentences, not fragments.

Example:

> - An invited member can connect an agent. The old plan treated that as the
>   owner's job.

## The headings

Include a heading only when it has content.

- **Changes** — corrections and rewrites to what already existed.
- **New in this version** — sections or ideas that were added, not corrected.
- **Open questions** — decisions genuinely waiting on the reader. Never park a
  decision here that the proposal already made.
- **Based on** — the source, as a plain last line: whose comments, which
  earlier version. Never put the source in the lead.

## When there is only one change

Write the lead sentence and stop. No headings, no bullets. Use this form:

    <verb, past tense> <the thing that changed> so that <what is true now>[, not <what was true before>]

> Rewrote the first-run plan so that an invited member can connect an agent
> too, not just the owner.

## Rules that apply everywhere

1. Name the change, not the work you did. Say what is now different for a
   person who reads the document.
2. Give a reason, not a source. "the old plan treated it as the owner's job" is
   a reason. "seven comments on version 2" is a source; it belongs on the last
   line.
3. Name the real subject: "the first-run plan", never "the document", "the
   section", or "several parts".
4. Do not repeat the document title.
5. Use past tense or present tense, never the imperative. "Removed the limit",
   not "Remove the limit" — the imperative reads as an order to the reader.
6. Never use: revised, updated, refined, adjusted, improved, addressed,
   various, several, some, changes, version, revision, feedback, "per your
   comments", "as requested".
7. Never make a count the subject. "Four corrections" and "+219 −119" measure
   work, not meaning.
8. No file names, no ticket numbers, no internal names for things the reader
   has not seen in the product.
9. Never use a verb that sounds like an explanation but names no mechanism:
   shaped, informed, drove, unlocked, enabled, reflects, aligns with,
   underpins, feeds into. Say what actually happens instead.

   Wrong: "drops the limit that shaped the old answer."
   Right: "drops the rule that the design must not need new backend work."

## Three checks before you send

Run all three on the lead, word by word. Run the first two on every bullet.

- **Which-one test.** Read the lead and stop at every noun. Does any noun make
  the reader ask "which one?" — "one new idea", "a third sighting", "several
  improvements", "the constraint", "a new section"? If yes, replace that noun
  with the thing itself. This is the check that fails most often.
- **Swap test.** Could this summary sit on a different proposal and still make
  sense? If yes, it is too vague. Rewrite it.
- **Decision test.** After the lead alone, does the reviewer know enough to
  lean toward Accept or Revise? If no, rewrite it.

Worked failure:

> Wrong: "Adds what the Make call produced: one new product idea, and a third
> sighting of the human-orchestration idea."

Two counts, no subject. The reader asks "which idea?" and must open the
document — which is the exact cost the summary exists to remove.

> Right: "The Make call adds one way to prove AI work pays off — tie every
> initiative to a department KPI — and a third confirmation that humans should
> orchestrate agents rather than hand off to them."

## Other item types

Questions carry the asker's own words. Never rewrite them.

## Example 1 — the same proposal, written twice

This is the most important example. Both versions carry the same facts. Nothing
is dropped from the first to the second.

**Wrong — 158 words in one paragraph:**

> Revision from Michaela's seven comments on version 2. Four substantive
> corrections: (1) an invited member does connect an agent — the "may not be
> their job" and "may never run a coding agent" assumptions are removed, and
> connecting becomes a shared step made obvious to both people; (2) an invited
> member arrives to a partial view filtered by role, not "everything"; (3) the
> "without new backend work" constraint is deleted everywhere — the design
> chooses the right signal and the backend follows; (4) Case 1 is designed first
> while both cases hold equal priority. Adds the requested empty-state audit as
> a new section, which corrects the "21 empty states" figure: only 3 of the 21
> are genuine day-one states, and a second empty-state component family was
> missed by that count. Expands the vocabulary layer into intro messages per
> area. Raises one new open decision: whether to limit other actions until an
> agent is connected.

Five faults, in the order the reader meets them:

1. It opens with the source, so the first thing read is the least useful.
2. It counts the work: "four substantive corrections".
3. It numbers items inside a paragraph, so nothing can be scanned.
4. It gives all six changes the same weight, so no subject stands out.
5. It buries the open question in the last clause, where a reader who stops
   early never sees it.

**Right — 31 words, then six lines:**

> The first-run plan assumed too much about invited members. This version fixes
> what an invited member can do and what they see, and removes the limits that
> shaped the old answer.
>
> **Changes**
> - An invited member can connect an agent. The old plan treated that as the
>   owner's job.
> - An invited member sees the part of the product their role covers, not
>   everything.
> - The "no new backend work" limit is gone, so the design picks the right
>   signal and the backend follows.
> - The empty-workspace case gets designed first, because nobody is present to
>   explain it. Both cases still block launch.
> - The wording layer becomes a short intro message for each area of the
>   product.
>
> **New in this version**
> - An audit of the empty states: only 3 of the 21 are day-one states, and a
>   second set was missed.
>
> **Open questions**
> - Should other actions wait until an agent is connected?
>
> Based on your seven comments on version 2.

## Example 2 — a proposal with one change

A rewrite of a public pricing page.

> Emptied the pricing page of plans and limits, because nothing in the product
> enforces those numbers yet.

No headings. No bullets. One change needs one sentence.

## Example 3 — a proposal with one change and one open question

> Moved the agent's answer under the question it answers, so a reader no longer
> scrolls to match them.
>
> **Open questions**
> - Should an unanswered question stay at the top of the list?

## Example 4 — line by line, what to write instead

**The lead**

Wrong:

> Revision from Michaela's seven comments on version 2. Four substantive
> corrections: (1) an invited member does connect an agent — the "may not be
> their job" assumption is removed; (2) an invited member arrives to a partial
> view filtered by role...

Right:

> The first-run plan assumed too much about invited members. This version fixes
> what an invited member can do and what they see.

The wrong version opens with the source, counts the work, and numbers the
changes inside a paragraph. The reader must finish it to learn the subject.

**A vague lead that fits any proposal**

Wrong: "Improves the onboarding document based on review feedback."

Right: "The first-run plan assumed too much about invited members."

The wrong version passes no swap test. Paste it on any other proposal and it
still reads as true.

**A bullet that names the work instead of the change**

Wrong: "Updated section 3 per feedback."

Right: "An invited member sees the part of the product their role covers, not
everything."

**A bullet with no real subject**

Wrong: "Removed the constraint."

Right: "The 'no new backend work' limit is gone, so the design picks the right
signal and the backend follows."

**A bullet in the imperative**

Wrong: "Fix the empty states."

Right: "Adds an audit of the empty states: only 3 of the 21 are day-one
states."

The imperative reads as an instruction to the reviewer. It is not one.

**An open question that asks nothing**

Wrong: "There are some open items to discuss."

Right: "Should other actions wait until an agent is connected?"

**A source in the wrong place**

Wrong: "Following your seven comments, this version fixes what an invited
member can do."

Right: the lead states what changed. The last line says "Based on your seven
comments on version 2."
