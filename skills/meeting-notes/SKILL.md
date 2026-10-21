---
name: meeting-notes
description: Turn a meeting transcript or rough notes into notes with decisions, risks, and owner-tagged action items, or into formal minutes. Use when the user shares a transcript or call notes, or asks for meeting notes, action items, or minutes.
license: MIT
metadata:
  version: "1.0.0"
---

# Meeting Notes

Two leading words control every step:
- **Source-bound**: every name, date, decision, reason, owner, due date, and priority comes from the source. When the source does not give a value, write a marker (`Unassigned`, `No date`, `Not stated`) and add a clarifying question. Tag a value you suggest `(proposed)`.
- **Anchor**: the place in the source that shows an item: the timestamp when the source has one, else the speaker and a short quote. Each decision and action item carries one, unless the user asks to leave anchors out.

## Step 1: Gather Inputs

Ask once, in one message, for the items the request does not give:

- **Source**: pasted text or a file path, and its type: transcript, human notes, or AI summary (Zoom, Meet, Teams, Otter, and others). An AI summary is a set of claims, not the record: ask for the transcript when the user has it.
- **Meeting**: title, date, attendees and their handles
- **Meeting type**: general, standup, client call, project review, formal minutes (board, committee, council), or other
- **Output style**: terse bullets or narrative; detail level (executive, standard, detailed); focus (comprehensive, action items only, decisions only); action-item format (checkboxes or table); redaction rules

Done when you have the source and know its type. For each other item the user does not give, use the default: the date from the source or `No date`, the type the content shows, terse bullets, standard detail, comprehensive focus, checkboxes.

## Step 2: Normalize the Source

1. Keep speaker labels and timestamps: Steps 3 and 4 use them for owners and anchors. Correct a label the source shows is wrong. Mark a line with no clear speaker `Speaker unknown`.
2. Remove filler words. Keep quoted statements word for word.
3. Apply the user's redaction rules. Then flag these items, also when there are no rules:
   - health, religion, politics, union membership, sex life, ethnic origin
   - remarks about a person's performance or conduct
   - parts the speakers call "off the record"
   - passwords, keys, account numbers

   Keep each flagged item out of the notes. List it under Clarifying Questions without the sensitive detail, and ask the user to keep or remove it.

Done when each line keeps the speaker and timestamp the source gave it, each sensitive item is flagged, and each quote is word for word.

## Step 3: Extract

For a long source, work in parts: by agenda topic, or every 15 minutes of timestamps. Finish one part before you start the next.

Sort each statement into one class:

| Class | Cue phrases |
|-------|-------------|
| **Decision** | "we decided", "agreed", "confirmed", "going forward we will", "everyone agreed" |
| **Action item** | "I will", "action:", a name plus a verb ("Maria will fix"), a request that someone accepts ("Sure", "OK") or the meeting assigns |
| **Proposed action** | a request or suggestion with no acceptance: "can you", "let's", "someone should" |
| **Open question** | a question with no answer, "not sure", "TBD", "need to check" |
| **Risk or blocker** | "blocked", "at risk", "depends on", "if X slips" |
| **Parking lot** | a topic moved to a later meeting |
| **Discussion point** | everything else that matters to the topic |

The source-bound test for the two strong classes is agreement:
- A **decision** needs agreement in the source. "Sarah thinks we need two more weeks" stays a discussion point. A topic with no agreement goes under Open Questions with the status `Disagreed` or `Needs discussion`.
- An **action item** needs an owner who accepts it or a meeting that assigns it. A proposed action goes under Open Questions, with a question about who owns it.

For each decision, capture the Decisions fields of the template (reason, options rejected, dissent, replaces) that the source states.

Done when each statement that matters has one class, in every part of the source.

## Step 4: Resolve Action Items

Write each action item as task, owner, and due date.

- **Task**: start with a verb and name the output ("Send the Q3 budget to finance"). Keep the meaning of the request, and add only measures the source gives.
- **Owner**: one named person. A first-person commitment ("I'll do it") goes to the speaker. Ask a clarifying question when:
  - the source names a team or role: write it, and ask "Who in [team] owns this?"
  - it names two or more people and no lead: write all names, and ask who owns it
  - it names no one: write `Unassigned`
  - the owner is not in the attendee list
- **Due date**: the stated date. Convert a relative date ("Friday") to a calendar date only when the meeting date is known, and show both: `Friday (2026-03-20)`. With no stated date, write `No date`. Suggest a date only when the user asks, tagged `(proposed)`.
- **Priority**: only when the source states it or the user asks for it.

Done when every action item has one owner or a clarifying question about it, and a due date or `No date`.

## Step 5: Write the Notes

Read the one template file for the meeting type from Step 1, and use its format:

| Meeting type | Template |
|--------------|----------|
| Standup | `references/standup.md` |
| Client call | `references/client-call.md` |
| Project review | `references/project-review.md` |
| Formal minutes | `references/formal-minutes.md` |
| Any other type | `references/general.md` |

Rules for the content:
- **AI-summary source**: add `**Source:** AI summary, not checked against a transcript` to the header.
- **Reader**: write for a reader who was absent. Expand each acronym once.
- **Attribution**: in Key Points, name a speaker only when the role matters to the point or the person disagrees.
- **Empty sections**: write `None`. Leave out Parking Lot when it is empty.
- **Style**: apply the output style from Step 1: a table for action items when the user asks for one, prose for a narrative style.

Done when every class from Step 3 is in its section and each decision and action item has an anchor.

## Step 6: Check

- Each name has one spelling and one form in all sections.
- Each item traces to a statement in the source, and each anchor shows its claim. Remove each inference, such as a cause the source does not state.
- Each part of the source has its decisions and action items in the notes. Check the middle parts of a long source again.
- Each marker (`Unassigned`, `No date`, `Not stated`), each sensitive flag, and each ambiguous pronoun or acronym has a line under Clarifying Questions.
- Each remark about a person states what they said or did, in words the person can read: notes can become an official record or be disclosed.

Done when all five checks pass. The notes are a draft: tell the user to answer the clarifying questions before they send it.

## Optional Extras

Add these when the user asks:
- **Timeline**: the main moments with their timestamps.
- **Share blurb**: 2–3 sentences for Slack or email, above the full notes.
