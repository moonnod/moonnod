# FoundIt — Campus Lost & Found, Matched Automatically

**Author:** Nishanth Reddy Adidela
**Artifact:** https://claude.ai/artifact/DnmxrchjbA4sDQbypF6Woq

## The idea

Lost and found reports on campus already get posted but just in the wrong place to be found. A student
loses a Hydroflask and posts about it in the campus WhatsApp or Discord group, someone finds it and
posts separately, hours or days later. Both messages scroll past within hours, and nobody connects
them, because nobody reads back through a group chat looking for a match. The usual alternatives like a
campus lost and found office, or a structured app all ask the student to go somewhere new and
file a report and most students never do that. They post where they already are.

FoundIt would read the group chat directly instead of asking anyone to go somewhere to report, it extracts
structured reports from informal chat text like item, distinguishing detail, location, time and then match a
"found" post against open "lost" posts by combining item similarity, location proximity and time
proximity into a ranked list of likely matches. No new app, no form, nothing for the student to adopt.

## Who it is for

Any student in a campus community group chat who's lost or found something. Our first real test would
have been one or two actual campus lost and found chats, pending group admin consent to pull message
history.

## What the NLP is

Its a four staged Structured extraction, a fine tuned token classifier (spaCy NER or DistilBERT) reads each
message, decides whether it's a report at all, and pulls out report type, item, location, and time,
trained on 100–150 hand labeled real messages against a keyword rule baseline. Semantic item matching:
a fine-tuned Sentence-BERT model decides whether a "blue Hydroflask" and a "blue water bottle" are the
same item, trained on (lost, found) pairs from threads where a match was later confirmed. Location and
time normalization: a hand-built campus gazetteer links informal location mentions to canonical
buildings, and a fuzzy time parser converts "this morning" into a real timestamp. A ranking engine with
no new model combines all three into a recall optimized shortlist, since missing a true match is worse
than surfacing an extra candidate.

## Evidence for the problem

Every student who's lost something on campus has scrolled back through a group chat by hand looking for
it, and it's their own campus chat, not a new platform. We can drop a lightweight logger into one
group tonight and have real message data by morning.



## Why we went with it

FoundIt only works with a bot that has standing presence inside the group chat, reading every message
as it arrives is what lets it catch a "lost" and "found" post that are hours apart. WhatsApp
doesn't admit outside bots to ordinary group chats the way Discord or Slack do, so the one piece of
access the whole idea depends on isn't something we're allowed to build. That's not a scope cut or a
harder than expected model, it's the foundation the rest of the pipeline — extraction, matching, the
gazetteer — was going to sit on, gone before any of it could be tested against real messages. The
matching logic itself still holds up, it's a better fit as a feature inside a campus app that already
has legitimate access to student messaging than as a standalone company.