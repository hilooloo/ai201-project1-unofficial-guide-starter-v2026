# The Unofficial Guide

Jiyoung Kim Torres - campus_life

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->
This project is an unofficial retrieval-augmented generation (RAG) guide built on the `campus_life` corpus, which contains 88 student-authored posts covering campus housing, course workloads, exam formats, dining options, and administrative deadlines. The system answers candid student questions—such as late-night dining options, dorm mold conditions, laundry machine fees, and course grading policies—using verified student advice rather than generic catalog descriptions. It indexes documents into sentence-bounded paragraph chunks, filters out-of-scope inquiries using an embedding-distance relevance gate, and grounds its responses strictly within retrieved chunks with cited sources.

## Chunking Strategy

**Chunk size:** 450
**Overlap:** 0

<!-- What about YOUR documents made you pick these numbers? Short posts and
     long sectioned guides don't want the same chunking, and "800 seemed
     reasonable" earns nothing. Point at something you noticed when you read
     the documents in Milestone 1.

     If you changed your mind partway through, say so and say why. That's worth
     more than pretending you got it right first time.

     Milestone 3. -->

The starter chunker produced 88 chunks from 88 documents because its fixed 800-character window never divided anything in `campus_life` (where documents average only ~317 characters). Leaving whole multi-topic posts intact produced broad, diluted embeddings. Furthermore, a naive split on double newlines caused short headings (e.g., "On the add/drop deadline") to break off into uninformative fragments.

I implemented a paragraph-oriented chunker that inspects double newlines and automatically merges short header snippets (< 80 characters) into the subsequent paragraph to preserve topical context. For paragraphs exceeding 500 characters, it splits strictly at sentence boundaries (`.`, `?`, `!`) buffering up to 450 characters with zero character overlap. This preserves self-contained thoughts and keeps specific figures, rules, and costs intact.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.

**Chunk 2** — source: `course_cs_340_workload.txt#0` — produced by: `chunker.py::split_documents`

Workload for CS 340 Databases

People keep asking so: 6 hours a week early, 15 in the last three weeks when the project lands. That's real time, not optimistic time.

**Chunk 3** — source: `course_stat_150_exams.txt#0` — produced by: ``

STAT 150 Applied Statistics — assessment

Three equally weighted midterms, no final. No curve, but the lowest midterm is dropped.

**Chunk 4** — source: `source: `housing_aldridge_hall.txt#0` — produced by: `chunker.py::split_documents`

Aldridge Hall — what it's actually like

I lived here my sophomore year. Built 1968, renovated 2019. Rooms are doubles with a shared bathroom per floor.

**Chunk 5** — source: `housing_morrow_house_laundry.txt#1` — produced by: `chunker.py::split_documents`

Best time to do laundry here is Tuesday or Wednesday morning. Sunday after 6pm you will wait.

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** How much does it cost to do laundry in Morrow House?

**Answer:**

In Morrow House, laundry costs $1.50 for a wash and $1.25 for a dryer, and can be paid with coin or card. 

Source: `housing_morrow_house.txt` (also mentioned in `housing_morrow_house_laundry.txt`).

**My relevance cutoff:**

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

| Question | In corpus? | Best distance |
|---|---|---|
| How bad is the damp or mold in Morrow House? | Yes | 0.4671 |
| Who teaches CS 101 and what do students think of them? | Yes | 0.4938 |
| What is the best place to eat on campus late at night? | Yes | 0.4201 |
| What should I know about taking exams in STAT 150? | Yes | 0.4692 |
| How much does it cost to do laundry in Morrow House? | Yes | 0.2045 |
| Who won the 1994 FIFA World Cup? | No | 0.8506 |
| How do I replace the alternator on a 2012 Honda Civic? | No | 0.8946 |
| What is the capital city of Australia? | No | 0.8232 |
| How does CRISPR-Cas9 gene editing work? | No | 0.8172 |
| What are the rules of cricket? | No | 0.7599 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**
When developing the custom chunker in Milestone 3, I prompted GitHub Copilot to write a paragraph-based chunker that would split on double newlines (`\n\n`) and keep chunks under 450 characters. Copilot initially returned a naive split that caused short document titles (such as "On the add/drop deadline" at 25 characters) to become isolated, zero-context chunk fragments. I refined the prompt and code logic to explicitly inspect paragraph lengths and merge any leading heading shorter than 80 characters into the subsequent paragraph before splitting long paragraphs at sentence boundaries.
**2.**
During Milestone 4, I used AI to analyze the separation gap between the 5 in-scope questions (distances ranging 0.205–0.494) and the 5 out-of-scope questions (distances ranging 0.760–0.895). The AI suggested evaluating whether the starter's default threshold of 0.60 remained robust against edge cases. Based on this analysis, I confirmed that 0.60 sits comfortably in the ~0.26 margin between both groups, preventing false rejections of specific campus queries while strictly rejecting non-corpus topics before prompt construction.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
