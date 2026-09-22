# Failure detection for LLM apps, on every conversation

**Author:** Anoop Reddy Kallem
**Artifact:** https://claude.ai/artifact/DsRBK1P8ZmSKygaAktoZDy

## The idea

Teams shipping chatbots and agents log every conversation but read almost none of them. The standard
fix is an LLM judge: a frontier model that reads a trace and says whether it went wrong. It works, but
it costs a few dollars per thousand traces and takes seconds per call, so teams run it on a 1% sample
and hope the failures land in that sample. Rare failures, the ones that show up once in a thousand
conversations, almost never do.

Tripwire is a small classifier that reads every trace instead of a sample. It tags five kinds of
failure: the user got frustrated, the user corrected the bot, the user gave up mid-task, the bot
refused something it should have handled, or the conversation drifted out of the app's scope. It runs
on a CPU in tens of milliseconds, drops in as a two-line SDK or an OpenTelemetry exporter, and feeds a
dashboard that shows failure rates over time and groups flagged traces by what went wrong.

The judge gives you accuracy on a sample. Keyword rules give you coverage with no understanding. We
want judge-level quality on the common cases at full coverage, for roughly 1/500th of the cost.

## Who it is for

Small teams running an LLM app with real traffic and no one whose job is reading logs. Our first users
are the other MSML641 teams, who will each ship a product this term and need to know when it fails in
front of their own users.

## What the NLP is

We distill the judge into a student. A frontier judge with a fixed rubric labels about 20k traces once,
returning a probability per label instead of a yes or no. Two of us hand-label 2k of those, which both
measures where the judge is wrong and becomes the held-out test set that never touches training. A
DeBERTa-v3-small encoder reads the last few turns with role markers and is trained on a blend of the
judge's probabilities and the human labels, so it learns the judge's judgment and our corrections to
it. We then calibrate each label with one temperature parameter, so a score of 0.8 means about 80% of
those traces really are failures, and export to int8 ONNX for CPU inference.

The evaluation compares three systems on the same gold set: keyword rules, the zero-shot judge, and our
student, on macro-F1, latency, and cost per thousand traces. The bar we set is at least 0.85 of the
judge's F1. The error analysis reports F1 by conversation length, language, and app domain, because
frustration detection is likely to misread terse or non-native writing, and we read 50 misclassified
traces by hand.

## Evidence for the problem

Not collected yet. In the first two weeks we will interview at least eight teams running LLM apps and
ask what fraction of their traces anyone reads, how they learn about failures today, and whether they
run a judge on a sample. We will also hand-label 200 traces from a public chat log to measure how often
each failure type actually occurs, which tells us how many a 1% sample would miss.

## Why we went with it

The upfront build would cost too much. Before the student can learn anything, a frontier judge has to 
label around 20k traces, and two of us have to hand-label 2k more for a gold set. Only after that can we
start training and calibrating the student. That is weeks of API spend and annotation time before we
have a single model to evaluate. The bigger problem is that we can't get real user evidence. Tripwire
only proves its value on a company running an LLM product with enough real traffic for rare failures
to show up, and with a team that would act on what it finds. Classmate projects won't have that volume
this term, and no company would hand its users' conversations to a pretend startup, so we could never
show that it works for the customer it was built for.