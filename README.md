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

Both models scored on the same 33 held-out posts in `test_split.csv` (seed 42). The baseline
is zero-shot `facebook/bart-large-mnli` using my label definitions from
`label_definitions.txt`; the trained model is the fine-tuned DistilBERT from `results.json`.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.273 | 0.424 | +0.152 |
| Macro F1 | 0.191 | 0.149 | -0.042 |
| F1 — `analysis` | 0.389 | 0.000 | -0.389 |
| F1 — `experience_report` | 0.250 | 0.000 | -0.250 |
| F1 — `help_request` | 0.125 | 0.596 | +0.471 |
| F1 — `suggestion` | 0.000 | 0.000 | +0.000 |

**What my definitions were worth.** I also ran the baseline with bare label names only
(no definitions), written to `baseline_results_bare.json`:

| Measure | With definitions | Bare label names |
|---|---|---|
| Overall accuracy | 0.273 | 0.364 |
| Macro F1 | 0.191 | 0.337 |
| F1 — `analysis` | 0.389 | 0.182 |
| F1 — `experience_report` | 0.250 | 0.500 |
| F1 — `help_request` | 0.125 | 0.400 |
| F1 — `suggestion` | 0.000 | 0.267 |

The table above uses the with-definitions run as my baseline, as the brief asks. The bare-names
run is better by 0.091 accuracy, so my definitions were worth about -0.09: pasted into the
zero-shot template ("This post is {definition}."), the long definitions made the baseline
worse, not better. On 33 posts that gap is about three posts, so I treat it as a hint rather
than a firm result.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

I predict the trained model will beat the baseline on `help_request`, the largest class
(14 of the 33 test posts), because fine-tuning on 149 examples teaches it that majority
pattern, while the zero-shot baseline will do no better than the trained model on the three
smaller labels (`analysis`, `suggestion`, `experience_report`), where 5 to 7 test posts each
leave the scores very noisy.

*Disclosure:* I had already glanced at `results.json` before writing this (it showed the
trained model mostly predicting `help_request`, accuracy 0.424), and I had seen the baseline
numbers (accuracy 0.273) from the first run. So this prediction is informed by partial
knowledge of both results rather than made fully blind.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

My prediction was half right. The trained model did beat the baseline on `help_request`
(F1 0.596 vs 0.125), but I was wrong that the baseline would do no better on the small labels:
it beat the trained model on `analysis` (0.389 vs 0.000) and `experience_report` (0.250 vs
0.000), and both models scored 0.000 on `suggestion`.

The trained model's higher accuracy (0.424 vs 0.273) is mostly an artifact of class balance:
`help_request` is 14 of 33 test posts (0.424 exactly), so a model that always answered
`help_request` would score the same accuracy. Its macro F1 (0.149) is actually below the
baseline's (0.191). So on this split fine-tuning has not yet learned three of the four labels,
and I can't claim it added anything beyond predicting the majority class. Milestone 2's three
seeds will show whether this is stable.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Seeds 42, 7 and 2024, all on CPU (`device: cpu`, torch 2.14.1+cpu), 33 test posts per seed.
Source: `results_three_seeds_before.json` (`RUN_LABEL = "before"`). Verdicts are filled in under
Verdicts and Diagnoses.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | >= 0.65 on every seed | 0.424 | 0.424 | 0.424 | (see Verdicts) |
| 2. Every label F1 | >= 0.60 on every seed | analysis 0.000, help_request 0.596, suggestion 0.000, experience_report 0.000 | same | same | (see Verdicts) |
| 3. `analysis` called `suggestion` | <= 30% | 0 of 7 (0%) | 0 of 7 (0%) | 0 of 7 (0%) | (see Verdicts) |
| 4. Label balance in `labels.csv` | each label 15%-50% | help_request 43.0%, analysis 22.0%, experience_report 22.0%, suggestion 13.1% | same (data, not seed) | same | (see Verdicts) |
| 5. Confidence: top half accuracy | >= 0.80 on every seed | not saved (top third: 0.818) | not saved (top third: 0.818) | not saved (top third: 0.818) | (see Verdicts) |

Full spread table from section 6:

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.424 | 0.424 | 0.424 | 0.000 |
| Macro F1 | 0.149 | 0.149 | 0.149 | 0.000 |
| F1 - `analysis` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 - `help_request` | 0.596 | 0.596 | 0.596 | 0.000 |
| F1 - `suggestion` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 - `experience_report` | 0.000 | 0.000 | 0.000 | 0.000 |

**A zero spread is not stability.** The notebook prints "Stable enough to trust", but identical
numbers on three different seeds mean the model predicted `help_request` for every test post
every time (see the matrix). A constant predictor has no variance to measure, so these three
runs show the model learned nothing beyond the majority class, not that the result is reliable.
Training loss only moved from about 1.35 to 1.25 over 3 epochs, and validation accuracy stayed
at 0.4375 after every epoch on every seed.

### Confusion matrix

Identical on all three seeds. Rows are true labels, columns are the model's guesses.

| true \ predicted | analysis | help_request | suggestion | experience_report |
|---|---|---|---|---|
| **analysis** | 0 | 7 | 0 | 0 |
| **help_request** | 0 | 14 | 0 | 0 |
| **suggestion** | 0 | 5 | 0 | 0 |
| **experience_report** | 0 | 7 | 0 | 0 |

**My biggest off-diagonal number, and what it means:**
The biggest off-diagonal cell is 7, in two places: 7 real `analysis` posts and 7 real
`experience_report` posts were both called `help_request`, and nothing went the other way,
because the model put all 33 posts in the `help_request` column. It means the model did not
learn any boundary with `help_request`, in either direction; it collapsed to the majority label.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

Criteria are the five in `criteria.md`, unchanged. Run Log — Before above has the numbers.

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | >= 0.65 on every seed | **MISSED** | 0.424 on all three seeds, 0.226 below target. |
| 2 | Every label F1 | >= 0.60 on every seed | **MISSED** | Only `help_request` is close (0.596, still under 0.60); `analysis`, `suggestion` and `experience_report` are 0.000 on all three seeds. |
| 3 | `analysis` called `suggestion` | <= 30% on every seed | **MET** (hollow) | 0 of 7 `analysis` posts were called `suggestion` on every seed, so the number passes. But the model never predicted `suggestion` at all, so this says nothing about whether it can tell the two apart. |
| 4 | Label balance in `labels.csv` | each label 15%-50% | **MISSED** | `suggestion` is 28 of 214 rows = 13.1%, under the 15% floor. `help_request` is 43.0%, under the 50% cap. |
| 5 | Confident half >= 0.80 accurate | >= 0.80 on every seed | **MISSED** | The saved results hold the top third, not the top half: 0.818, 0.727, 0.636. For seed 2024, 4 of its 11 most confident posts were wrong, so its top 17 (half of 33) has at most 13 correct = 0.765 whatever the other 6 are. One seed below 0.80 is enough to miss. |

**Diagnoses**

**Criteria 1 and 2 (accuracy 0.424, three labels at F1 0.000): the model has not learned the
task, it predicts the majority label.** How I know: the confusion matrix has all 33 test
posts in the `help_request` column on all three seeds, and accuracy 0.424 is exactly
14/33, the share of `help_request`. Training loss only fell from about 1.35 to 1.25, and
validation accuracy sat at 0.4375 after every epoch on every seed (also exactly the
`help_request` share), so training barely moved the model. With 149 training posts, batch
size 16 and 3 epochs it took only about 28 optimizer steps at learning rate 2e-5, which is
a very short run. So the leading cause is training that is too short for this data, with a
task that may also be out of reach from about 200 posts: the model did not even separate
`analysis` (33 training posts) from `help_request`.

**Criterion 3 (met only because the model never says `suggestion`):** not a real pass. It
tells me nothing about the `analysis` / `suggestion` boundary, which is still untested.

**Criterion 4 (`suggestion` at 13.1%):** too few examples for one label. I labelled 28
`suggestion` posts, 19 of them in training and 5 in test, so even a good model has little to
learn from or be scored on for that label. This one is about my data, not the model.

**Criterion 5 (confidence):** the model's confidence is nearly flat: mean confidence is
0.44 when right and 0.42 when wrong. The model was almost equally unsure about everything,
because it only ever picks the one label. The more confident third being more accurate
than the least confident third (0.82 vs 0.18 on seed 42) mostly reflects which posts
happen to be `help_request`, not calibrated confidence.

**Evidence I do not have yet:** the Agreement Report against the staff set. It is the one
instrument that can separate a labelling problem from a model problem, and I have not got
the staff labels yet, so I cannot say whether my own labelling was inconsistent on any
boundary. Everything above rests on the confusion matrix and training curves only. I will
revisit the diagnoses of criteria 1 and 2 once the agreement check is run.

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

**What I changed:** One training setting, the learning rate: 2e-5 -> 5e-5. Epochs stay at 3,
batch size at 16, max length at 128, and the data, labels and seeds (42, 7, 2024) are
untouched. `RUN_LABEL = "after"`, so the numbers go to `results_three_seeds_after.json` and
`results_three_seeds_before.json` is not overwritten. I picked the learning rate because
the training loss barely moved, which a larger step size attacks directly, and because a
longer run (more epochs) would have taken much more time on CPU.

**Which diagnosis pointed at it:** Verdicts and Diagnoses for criteria 1 and 2: the model
predicted `help_request` for all 33 test posts on every seed, training loss only fell from
about 1.35 to 1.25, and validation accuracy stayed at 0.4375 after every epoch. At batch
size 16 that is about 28 optimizer steps in total, so the model likely stopped before it
learned anything beyond the majority label. This is the one diagnosis I have evidence for
without the agreement report, which is why I picked it. It does not fix criterion 4
(`suggestion` at 13.1% of the data); more training will not add examples.

**Prediction, written before the run:** I expect accuracy and macro F1 to rise above the
majority-class floor (0.424 / 0.149) and at least one more label to get a non-zero F1.
I do not expect criteria 1 and 2 to be met on all three seeds, and I think `suggestion`
(19 training posts) will stay the weakest label.

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
