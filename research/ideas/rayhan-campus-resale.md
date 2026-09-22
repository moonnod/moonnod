# A marketplace for second-hand things and rooms on campus

**Author:** Rayhan Patel
**Artifact:** https://claude.ai/artifact/Jhh6FWSPo397ykXVSXcRio

## The idea

Every apartment complex in College Park runs its own WhatsApp groups, several of them, and you can
only see the ones you are already in. So a desk being given away in the building next door, or a room
coming free in the complex across the street, is invisible to you and there is no way to ask.

MoonNod is a campus marketplace for both halves of moving: the second-hand things people are selling,
and the rooms and apartments coming up for sublease. You post an item by photographing it. You post a
room the same way, with the dates it is free. You search your own building first, then the buildings
around you, for either. If nothing matches, you ask to be told when it appears.

Facebook Marketplace shows you the whole city and assumes you have a car. A group chat shows you one
building, and only if you already live there. We want the reach of the first with the precision of the
second.

## Who it is for

Graduate students moving in and out of housing in College Park. The same person is a seller in
December and a buyer in January, which is why one product serves both.

## What the NLP is

A rules parser turns one free-text post into separate items, with an evidence span behind every field,
so nothing is stored that the post did not say. A campus lexicon expands queries, so *naarkali* finds
a chair and *study table* finds a desk. Retrieval is Postgres full-text search over those fields. A
verifier checks every generated claim against its cited source and deletes the ones it cannot support.

The next piece is temporal: pulling available-from and available-until out of a post and normalising
them against the UMD calendar. Our parser currently extracts neither.

## Evidence for the problem

From a 53-message sample of one group: 21% of messages were reposts of the same item, 10 of 21
listings stated no price at all, and 9 of 15 "wanted" posts had already been answered by a listing
nobody found.

## Why we went with it

The users are two buildings away and we can hand them an invite code this week. A v0 is already
deployed. And Kavya's seven housing fields map onto our room form field for field, which is why her
idea and this one became one product instead of two.
