---
team: MoonNod
session: 04
date: 2026-09-22
members:
  - name: Rayhan Patel
    github: Rayhanpatel
    hat: Product
  - name: Anoop Kallem
    github: kallemanoop
    hat: Engineering
  - name: Kavya Jalihalli Matadha
    github: kavyajalihallimatadha
    hat: Data&Eval
  - name: Nishanth Reddy Adidela
    github: nishanthadidela
    hat: Users&Research
north_star:
  metric: task success rate (a resident searches and finds the item or room, or gets an alert that later matches)
  value: not measured, nobody outside the team has used it
  previous: n/a (first report)
---

## The idea

Every apartment complex in College Park runs its own WhatsApp groups, several of them, and you can only see the
ones you are already in. So a desk being given away in the building next door, or a room coming free in the
complex across the street, is invisible to you and there is no way to ask. Between us we are in seven of these
groups, which is a small fraction of the ones that exist.

**MoonNod is a campus marketplace for both halves of moving: the second-hand things people are selling, and the
rooms and apartments coming up for sublease.** You post an item by photographing it. You post a room or a
sublease the same way, with the dates it is free. You search your own building first, then the buildings around
you, for either. If nothing matches, you ask to be told when it appears.

Facebook Marketplace shows you the whole city and assumes you have a car. A group chat shows you one building,
and only if you already live there. We want the reach of the first with the precision of the second.

This was one of four ideas we brought this week, one per member. We killed two of them, and the other two turned
out to be the same system. The reasoning is below.

## Shipped this week

**A working v0 is deployed and you can use it: https://cr-api-657377045179.us-east4.run.app/pilot/** You pick
your building, search what is nearby, and a photo becomes a draft listing with a source on every field. 223 API
tests and 139 parser tests, in the product repository. Sign-in is by invite code while it is a pilot. **Armin and Aadesh, the code is
`terps-3juyss`**, and any browser you open becomes its own seller account. The product code lives in a
separate repository and is deployed from there, so the evidence for this section is the live URL rather than pull
requests here.

**The NLP in it, so it is on the record.** A rules parser turns one free-text post into separate items with an
evidence span behind every field, a campus lexicon expands queries so *naarkali* finds a chair and *study table*
finds a desk, retrieval is Postgres full-text search over those fields, and a verifier checks every generated
claim against its cited source and deletes the ones it cannot support. The hosted model drafts; our own code
decides what survives.

**Four idea write-ups, each committed by its own author.**

| Member | Idea | Committed |
|---|---|---|
| Rayhan | A marketplace for second-hand things on campus | `research/ideas/rayhan-campus-resale.md` |
| Kavya | GradMatch: show which rooms are going to be free, and when | `research/ideas/kavya-gradmatch.md` |
| Anoop | Tripwire: find the conversations where a chatbot failed its user | `research/ideas/anoop-tripwire.md` |
| Nishanth | FoundIt: match "lost my keys" posts to "found keys" posts in group chats | `research/ideas/nishanth-foundit.md` |

**We chose between them by measuring, not by arguing.** Kavya's spec pulls seven fields out of a housing post, so
we ran those seven fields against the parser already running in the product. The result is the metric below, and
it is why her idea and Rayhan's became one product instead of two.

## User evidence

Following the TA's guidance, this week this is a plan rather than a test.

- **Who they are.** Graduate students at Graduate Hills and Graduate Gardens who are moving in or out.
- **What they do now.** Scroll the few groups they happen to be in and search each one separately. In a
  53-message sample from those groups, 21% of messages were reposts, 10 of 21 listings stated no price, and 9 of
  15 "wanted" posts had already been answered by a listing nobody found.
- **How we get feedback.** Ten invite codes, one per person. Five residents, three search tasks each, success and
  time recorded. The raw log goes into `evidence/` the day it is run.
- Nobody outside the team has used it. One of us published a real listing and deleted it again, but that is our
  own team using our own product, so we are not counting it.

> An interview, a reaction to a mockup, or feedback on this report does not count.
> See the user evidence standard in `project/guidelines.md`.

## Metrics snapshot

| What we measured | This week | Last week |
|---|---|---|
| Housing fields our parser extracts, of Kavya's seven | **3 of 7** | n/a |
| Search precision@5 on 31 items, keyword only | 1.00 | n/a |
| Search precision@5 on 31 items, keyword and vector fused | 0.32 | n/a |
| Cost to serve one listing draft | ~$0.018 | n/a |

Of Kavya's seven fields our parser gets space type, location and price. It gets nothing for available-from,
available-until, utilities and furnished. Measured on the housing posts in our 53-message real sample plus the
synthetic gold set. The same run found three price bugs, now failing tests: `"900/month"` returns no price because
the pattern needs a `$` or a currency word, and `"$850 monthly"` is stored as a one-off price, so a monthly rent
is recorded as a sale price.

**Is this the same model running in the product? No.** Our evaluation harness scores its own BM25, while the
product ranks with Postgres full-text search plus lexicon expansion. Next week the harness calls the shipped
endpoint, so the number describes what a user actually touches.

## What did not work

- **Tripwire, dropped.** If you run a chatbot you cannot read every conversation, so you never find out which
  ones went badly. Tripwire reads all of them cheaply and flags the failures. This was the hardest to turn down,
  and our first reason was wrong: we said we had no data and no budget, but WildChat and LMSYS-Chat-1M are public
  and free, and labelling 20,000 conversations with a cheap model costs tens of dollars. The real problem is the
  user. Tripwire is for small teams running production chatbots, and we cannot get five of those to sit with us in
  College Park. So we are keeping the method and dropping the product: its teacher-student approach is what we
  plan to use on our own extraction problem.
- **FoundIt, dropped.** People post "lost my keys" and "found keys" in campus group chats and the two posts never
  find each other, so a bot sitting in the group would match them. WhatsApp does not admit outside bots to
  ordinary group chats, so the one thing that makes it work is not something we are allowed to build. Good as a
  feature inside a campus app later, not a company on its own.
- **Our evaluation scores the wrong system.** The harness ranks with its own BM25 while the product ranks with
  Postgres full-text search, so every retrieval number we have describes a ranker nobody is running.
- **Our own advice was wrong, and only using it showed us.** Our capture guidance told sellers to photograph the
  label first, so the first real listing went out with a cardboard box as its main picture and the item third.
- **Search got worse when we made it cleverer.** Fusing vector results into keyword search cut precision@5 from
  1.00 to 0.32, so the vector path ships off. The cause is structural rather than a tuning detail: availability
  and freshness are plain multipliers that vary up to 1000x while the relevance term varies at most 4.26x, so the
  ranking is really "newest thing nearby, with relevance as a tiebreaker".
- **And a flaw in that measurement, which is ours.** Relevance there is judged lexically, meaning an item counts
  as relevant if its title contains the query word. That definition favours the keyword path we declared the
  winner. On 3,029 synthetic items both paths scored 1.00, so this is a small-corpus result, and a small corpus
  is exactly what this product starts with.

## Challenges / blockers

- Recruiting five residents who will actually sit down with us in a week when everyone is mid-semester.
- We have no issue board yet. Work this week was tracked in conversation, which does not scale past four people
  and is not what the guidelines ask for. Issues and a board go up with next week's build.
- The iOS build cannot go on a phone yet, so the pilot stays a web page for now.

## Next week's goal

**One website where a person can post a second-hand item or a room, and search across buildings for either.**

Four things, in order:

1. **Items and rooms in one search.** Filter by building, and for rooms, by when they are free.
2. **Teach the parser the four fields it misses.** Available-from, available-until, utilities and furnished, with
   dates normalised against the UMD calendar. This is the measurement above turned into work.
3. **Make dates searchable at all.** Today the room form writes dates into a free-text JSON blob, not into real
   date columns, so no date filter is even possible. They move into typed columns.
4. **Five residents use it**, three search tasks each, logged the same day.

The question we expect to learn something about: whether one person wants items and rooms in one place, or
whether a sublease and a desk in the same feed make each other harder to find.

## Individual contributions

- **Rayhan (Product):** deployed v0, ran the seven-field measurement against the parser, turned the three price
  bugs into failing tests, evaluation harness, and the negative results above.
- **Kavya (Data&Eval):** GradMatch, including per-field F1, a double-annotation agreement target and an ablation
  against a regex baseline. This is the spec we measured against.
- **Anoop (Engineering):** Tripwire, including the labelling cost and data requirements that decided it.
- **Nishanth (Users&Research):** FoundIt, and the competitor search that found the live product ruling it out.

## Lean canvas changes

Two of our ideas became one. They stay two ideas with two authors. What changed is the measurement: **Kavya's
seven fields are our room form, field for field**, and the four our parser cannot extract are exactly the gap her
work fills. Selling the desk and handing over the room are the same person in the same week of the year.

- **New NLP work this creates:** pulling dates out of a post and normalising them against the UMD calendar. Hard,
  measurable per field, and now our strongest component.
- **Go to market:** one building first, not a campus, with supply seeded by residents pasting in their own old
  posts. Supply and demand are out of phase, because people give furniture away in May and December and arrive
  needing it in August and January. That is why the alert and the forward-looking timeline are the mechanism and
  not a convenience.
- **Cost to serve:** about $0.018 per listing draft on Gemini under our cloud credit, $0.09 for the whole measured session. Search is Postgres and
  is effectively free.
- **The risk.** Kavya's own thesis predicts the data may not exist. If people post when the lease is already
  ending, the corpus holds almost no forward-looking dates. Our answer is that the form is the data source and
  the NLP lowers the cost of filling it in: paste your old post, we fill the fields, you correct them, and every
  correction is a labelled example.
- **Decided this week: we are not choosing between furniture and housing.** MoonNod carries both as first-class
  listings in one marketplace. It is the same person in the same week, the reach argument is identical for both,
  and the person selling a desk in December is usually the person leaving the room.
- **Still open, and it is our pivot-or-persevere statement on Oct 6:** whether the AI listing assistant stays in
  the product at all. Five residents settle it.
