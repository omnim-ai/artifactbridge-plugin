# Recipe — thread brief

The `summary` of `artifactbridge_brief_agent_room` (and `briefing.summary` on
`artifactbridge_open_agent_room`) is the first thing a person reads under the
thread title. They may read nothing else. Write it for that person, not for
another agent.

A person opening the thread should know what the work is after the first
paragraph. Optional bullets after the lead hold the scannable facts.

## Structure

    <lead: one or two sentences. The whole high-level message.>

    - **<scannable fact>** <one concrete line>
    - **<scannable fact>** <one concrete line>

The collapsed view shows the lead only, up to five lines. "Show more" reveals
the bullets. If the lead is the whole brief, stop there. Do not add bullets
that restate the lead.

## The lead

One or two sentences. 40 words or fewer. Name the work and why the thread
exists. Do not open with background, process, or who asked. A reader who stops
here must still have the message.

Wrong:

> This thread exists to coordinate the remaining production Stripe work after
> earlier discussion of pricing, secrets, and environment setup.

Right:

> Turn on Stripe live mode for production with placeholder Team prices.

## The bullets

Use a bullet list when there are two or more concrete constraints, numbers,
related tickets, or hard rules. One line per item. 20 words or fewer.

Bold only the words a reader would scan for: amounts, ticket keys, dates, and
hard constraints. Do not bold a whole sentence. Do not bold for tone.

Wrong:

> Stand up Stripe live mode for production using placeholder Team prices
> ($20/mo, $200/yr). Then vault secrets and set GitHub production env
> (AI-2078). Do not wait on AI-1781. Restricted keys are dashboard-only.

Right:

> Turn on Stripe live mode for production with placeholder Team prices.
>
> - **$20/mo** and **$200/yr** placeholder Team prices
> - Vault secrets and set the GitHub production env (**AI-2078**)
> - Do not wait on **AI-1781**
> - Restricted keys stay dashboard-only

## Voice

Write as a sharp human editor. Keep the point. Cut the padding.

- Use short, direct sentences. Active voice. Name the actor when it matters.
- Be concrete. Keep numbers, names, ticket keys, and constraints. Do not
  smooth them into generic importance.
- One term for one concept. Repeat the clear word. Do not cycle synonyms.
- Do not write who created the brief. Do not close with "Created by @name".
- Do not write current status or next actions. The thread status field has
  that job.

Never use: delve, foster, leverage, utilize, facilitate, empower, streamline,
robust, cutting-edge, paradigm, game changer, tapestry, realm, beacon,
multifaceted, meticulous, intricate, paramount, transformative, elevate,
embark, supercharge, harness, "it's worth noting", "at the end of the day",
"in order to", "going forward".

Never open with: "Here's the thing", "Let me be clear", "This thread is about",
"The purpose of this room is". State the work.

Never end with a recap, a slogan, or a line about who wrote it.

## Markdown that is allowed

- A lead paragraph (plain prose).
- A following bullet list (`- `).
- `**bold**` on scannable facts inside those bullets.

Do not use headings, numbered lists, tables, code fences, or links. Caps:
2,000 characters / 20 lines. An over-cap brief is rejected, not truncated.

## When there is only one fact

Write the lead and stop.

> Split the tracker so ordering stays a human decision.

## Checks before you send

- **Stop-after-lead.** Cover the lead. Does the reader know what the work is?
  If no, rewrite the lead. Do not move the message into a bullet.
- **Swap test.** Could this brief sit on a different thread and still read as
  true? If yes, it is too vague.
- **Created-by test.** If the last sentence names an author, a runtime, or
  "Created by", delete that sentence.
- **Scan test.** Read only the bold words. Do they name the numbers, tickets,
  and hard rules? If bold is decorative, remove it.
