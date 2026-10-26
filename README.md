# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

**Community:** r/rpg — a public Reddit community for discussing tabletop roleplaying games.
**URL:** https://www.reddit.com/r/rpg/

This project classifies public r/rpg posts by what the post is mainly trying to do.
Each post is sorted into one of four labels: `analysis` (argues or evaluates a viewpoint
with reasons or examples), `help_request` (asks for help, advice or recommendations),
`suggestion` (recommends an approach, tool or game), or `experience_report` (recounts
personal play or campaign experience). The model is a fine-tuned DistilBERT.

### Candidate distinctions (from reading r/rpg)

1. Some posts provide concrete examples, while others express general opinions without examples.
2. Some posts explain reasons or outcomes, while others simply express likes or dislikes.
3. Some help-seeking posts describe attempted solutions and their results, while others only ask a question.



---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** Primarily presents or evaluates a viewpoint, supporting it with explicit reasoning, concrete examples, or facts.

**Example 1:**
> "The fact that I'm dumb doesn't mean NPC's are dumb: Importance of declaring intent" — argues that the GM can't be expected to catch a player's hidden master plan, using the polymorph-sword, glyphs-of-warding and devil-wordplay scenarios as support, and concludes players should declare intent.

**Example 2:**
> "Can you think of an innovative roleplaying game rule, mechanic, or procedure that expanded your notions of what a good RPG could be?" — the author uses concrete examples of differently named skills in *Unknown Armies* to explain how a skill system can express both a character's abilities and their personality.

### `help_request`

**Definition:** Primarily asks others for help solving a problem, offering advice, or recommending options.

**Example 1:**
> "Mass Effect RPG System" — "I am looking for a system to run in the setting… Anyone with experience running any of these games… could you tell me why you think X system works well with the setting?"

**Example 2:**
> "Resources/books for designing heists" — "I suck at running heists… Are there any GM books that talk about that?"

### `suggestion`

**Definition:** Primarily recommends an approach, tool, or game to readers.

**Example 1:**
> "HyperspaceD6 V3 has been released!" — announces the full release, lists what's new, and says "if you've been missing D6, give it a look."

**Example 2:**
> "If you haven't tried RPGtrader.com yet, you really should" — recommends the site and describes its filters (by system, genre, setting, player count), encouraging readers to use it.

### `experience_report`

**Definition:** Primarily recounts personal gameplay experiences or campaign progress, rather than arguing a point, seeking help, or offering advice.

**Example 1:**
> "We broke into a guy's house for a good reason. The town did not see it that way" — a narrative of what the party did, how the town reacted, and how it changed the rest of the session.

**Example 2:**
> "A Dangerous Thing to Know" — the author recounts being given a book at age 11, how its RPG chapter shaped their start in the hobby, and how they rediscovered it recently.

### The hardest boundary

**Which two labels:** `suggestion` vs `experience_report` (with `analysis` as a close third).
Borderline post: "Who gets inconvenienced if your character never comes home?" — a GM
describes a question they now ask players at character creation (a recommendation),
tells how it worked in their own small-town campaign (an experience), and argues why
tragic backstories feel disconnected (an argument).

**The decision rule I used every time:** Label by what the post asks the reader to *do
or believe* at the end. If it tells readers to adopt a practice, tool or game, it's
`suggestion`, even when most of the text is a personal anecdote. If it only recounts what
happened with no recommendation, it's `experience_report`. If the central point is a claim
about *why* something is true (not "do X"), it's `analysis`. Posts whose main sentence is
a question to the community are `help_request` regardless of how much backstory they have.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public posts from r/rpg, downloaded as JSON from
Reddit's public listing and search endpoints (no login): the top posts of the year,
and searches by post flair (Game Suggestion, Discussion, Actual Play, Game Master,
Table Troubles, Resources/Tools, Basic Questions). I kept the title plus the whole
body as one text. I left out link-only posts, news and announcements, crowdfunding and
self-promotion, AMAs, obituaries, repeated episode-announcement posts, and posts that
only ask a general discussion question or state a one-line opinion, because none of
my four labels fit them. 214 posts were kept; `labels.csv` is one unsplit file.

**How I labelled them:** I did **not** label any posts cold. All 214 rows were
pre-labelled by Claude from my four definitions and my decision rule, with borderline
cases flagged, and I then reviewed them. Their `note` column says `pre-labelled`.
Because no rows are cold, I have no unaided baseline of my own labelling.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `help_request` | 92 | 43% |
| `analysis` | 47 | 22% |
| `experience_report` | 47 | 22% |
| `suggestion` | 28 | 13% |
| **Total** | 214 | 100% |

No label is above 70% (or my own 50% cap). `suggestion` is the thinnest label at 13%, just
under my own 15% floor in criteria 4; I did not relabel borderline posts to hide this.

**Three hard cases**

**1.**
> *The post:* "Who gets inconvenienced if your character never comes home?" — a GM
> describes a question they now ask players at character creation, tells how it played
> out in a small-town campaign, and argues that tragic backstories feel disconnected.
>
> *Could have been:* `suggestion` / `experience_report` (and `analysis`)
>
> *I chose* `suggestion`, because the post is built around "try asking this question" and
> the anecdote is there to support that recommendation. If it had only told the story,
> it would be `experience_report`.

**2.**
> *The post:* "The fact that I'm dumb doesn't mean NPC's are dumb: Importance of
> declaring intent" — argues that a GM can't be expected to catch a player's hidden master
> plan, using several examples, and ends with "if you have a master plan then declare
> what you're going for."
>
> *Could have been:* `analysis` / `suggestion`
>
> *I chose* `analysis`, because the main body is an argument about *why* the GM can't catch
> hidden plans; the closing request follows from it. This is not consistent with how I
> treated posts that end "you should do X" (which I called `suggestion`), and it is the
> weakest point of my decision rule.

**3.**
> *The post:* "Please, players, find the time to play" — a GM's rant about players
> being disrespectful of their time, ending with "players need to do BETTER".
>
> *Could have been:* `suggestion` / `analysis` / `experience_report`
>
> *I chose* `suggestion`, because the post tells a group of readers what to do, even though
> it is mostly venting from personal experience. A reasonable reader could call it
> `experience_report`; I expect the model to find this kind of post hard.

**A gap in the taxonomy.** Two kinds of common r/rpg posts fit none of my four labels:
one-line opinions with no reasoning ("I am tired of 5e") and pure discussion questions
that are not asking for help ("What's your favourite RPG nobody knows about?"). I left
them out of the dataset, so the model has never seen them and will force them into one
of the four labels.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`, fine-tuned for 4-way classification.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128 tokens, seed 42
(split seed and training seed). Trained on CPU (torch 2.14.1+cpu); `results.json` records
`device: cpu`.

**Anything I changed from the defaults, and why:** Nothing. I used all the defaults.
Posts longer than 128 tokens are truncated, and many of my posts are much longer than that
(some over 1,000 words), so the model only sees the title and the opening of long posts.

**Split sizes:** train 149 / val 32 / test 33 (stratified 70/15/15 of 214 rows).

| Label | train | val | test |
|---|---|---|---|
| `analysis` | 33 | 7 | 7 |
| `help_request` | 64 | 14 | 14 |
| `suggestion` | 19 | 4 | 5 |
| `experience_report` | 33 | 7 | 7 |

Three labels have fewer than 8 test examples (`analysis` 7, `suggestion` 5,
`experience_report` 7), so their F1 will move a lot between seeds.


---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1 — stress-testing my labels**

- *What I asked for:* Claude to write eight posts sitting on the boundaries of my four label definitions.
- *What came back:* Eight short posts plus an answer key. Most were classifiable from my
  decision rule, but number 8 (argues a cause, then says "every GM should insist on it")
  could not be separated cleanly between `analysis` and `suggestion`.
- *What I changed:* I kept the rule (label by what the post asks the reader to do or
  believe at the end) but wrote down that `analysis` vs `suggestion` is its weakest
  boundary, and in this README the NPC post is the clearest case where I did not apply
  it consistently. *(Fill in anything you changed in the definitions after this.)*

**Pre-labelling disclosure:** Claude pre-labelled all 214 rows of `labels.csv` from my four
definitions and my decision rule, including the 20 posts I had meant to label cold. No row is
a cold label. Claude also did the filtering of the downloaded Reddit JSON into posts with
text and chose which posts to leave out. I reviewed the labels afterwards. *(Edit this
sentence to say exactly what you checked and how many labels you changed.)*

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
