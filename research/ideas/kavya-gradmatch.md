# A housing availability layer for Graduate Hills and Gardens
 
**Author:** Kavya  
**Artifact:** https://claude.ai/artifact/GhC3x1FqLofW7JEkxuqi98
 
## The idea
 
Graduate Hills and Gardens runs on WhatsApp groups - one per building, and you can only see the ones you were invited into. A room opening in December two buildings away is invisible to you. Listings are posted the day someone leaves, not three months earlier when you needed to know.
 
GradMatch extracts structured availability from posts across WhatsApp groups, Reddit, and the UMD off-campus housing database and builds a single forward-looking timeline. You search in plain language. The gap detector tells you which windows have no listing at all before you find out by arriving and scrambling.
 
## Who it is for
 
Graduate students moving into or out of Graduate Hills and Gardens - the same person is a poster in December and a seeker in January. New admits arriving for the first time are the critical case: they need a space confirmed before they arrive and are currently searching blind across groups they are not in.
 
## What the NLP is
 
A spaCy NER model fine-tuned on annotated GH posts extracts seven fields from every listing - space type, location, available-from, available-until, price, utilities, furnished - with an evidence span behind each field so nothing is stored the post did not say.
 
A temporal normalization layer converts what people write - *Dec 15ish*, *Spring semester start*, *after I defend* - into actual calendar dates anchored against the UMD academic calendar. Existing date-extraction tools handle formal dates well but break on the informal language grad students use, which is where our model does the real work.
 
A BERT slot-filling model trained on queries from real residents converts a plain-language search into the same structured fields, so matching is field-to-field rather than keyword-to-keyword.
 
Interval algebra over all extracted date ranges surfaces windows with no listing coverage per space type. This is the novel output no existing platform produces.
 
## Evidence for the problem
 
We sampled two Graduate Hills WhatsApp groups over one week. Most availability posts had no end date. Several seekers were looking for something already posted in a group they were not in. January listings routinely appeared in December, after people had already found something else.
 
## Why we went with it
 
Every graduate student in College Park goes through this at least once - arriving without a place, or leaving with a room nobody found in time. The problem is immediate and the users are next door, which means we can put a form in a group chat tonight and have real data by morning.
 
The NLP tasks are grounded. Named entity extraction and temporal normalization have established baselines, published datasets, and clear metrics - so we can show improvement week by week rather than hoping something works at the end. The forward-looking timeline and gap detection are things no existing platform has built for this context, which gives us a genuine contribution to evaluate rather than reproducing something that already exists.
 
