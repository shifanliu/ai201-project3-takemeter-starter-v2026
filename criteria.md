# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy is at least 0.65

On the held-out test split, overall accuracy is at least 0.65 on every one of
the three seeds (not just on average).

**Why this target:** I have four labels, so a coin-flip guesser gets about 0.25,
and always guessing my biggest label (probably `help_request`) would score
roughly 0.35-0.40. 0.65 is clearly above both, and it's where I'd honestly call
the classifier useful for sorting r/rpg posts. I didn't go higher because with
only ~200 posts and a boundary I already know is fuzzy, 0.80 would be a guess
about a model I haven't seen.

---

## 2. No label has an F1 below 0.60

Each of the four labels (`analysis`, `help_request`, `suggestion`,
`experience_report`) has a test-set F1 of at least 0.60 on every seed.

**Why this target:** Accuracy can hide a label the model never learned. With
~200 posts and a 15% test split, my smallest label will have only about 6-10
test examples, so its F1 will swing between seeds. That is why the floor is
set slightly below my overall accuracy target of 0.65: 0.60 says "the model
has learned something clearly useful for every label, including the smallest"
without demanding stability the data can't give. 0.50 felt too loose, since it
would pass a label the model gets wrong about half the time.

---

## 3. `analysis` posts are called `suggestion` at most 30% of the time

In the confusion matrix on the test split, at most 30% of true `analysis` posts
are predicted as `suggestion`, on every seed.

**Why this target:** This is the boundary my taxonomy is least sure about. Posts
like "I argue X, so every GM should do Y" sit between the two, and my decision
rule resolves them by the closing call to action. If the model can't keep these
apart even 7 times in 10, the label definitions (not the training run) are the
likely problem. 30% allows for a genuinely ambiguous minority of posts.

---

## 4. No label is above 50% or below 15% of `labels.csv`

Of the rows in `labels.csv`, every label makes up between 15% and 50%
(for 200 rows: between 30 and 100 posts each).

**Why this target:** The course cap is 70%, but with four labels a 70% label
would mean the model can score 0.70 accuracy by guessing one word. The 15%
floor guarantees roughly 4-5 test examples at the very least, and I expect more.
I expect `help_request` to be my biggest label because r/rpg is full of
"which game should I use?" posts, so 50% is a deliberate guard against it
swamping the rest.

---

## 5. The more confident half of predictions is at least 0.80 accurate

On the test split, I sort the posts by the model's top predicted probability
and take the more confident half (rounded up). Accuracy on that half is at
least 0.80 on every seed.

**Why this target:** A confident prediction should be right more often than an
unsure one, otherwise the confidence score is useless for deciding which
posts to trust. Using "the top half" instead of a fixed probability cutoff
means the check always has enough posts (about 15 of ~30), even if the model
is rarely very confident. 0.80 is 0.15 above my overall target of 0.65, so
it says the model's confidence carries real information. I expect the
`help_request` posts, which are mostly phrased as questions, to dominate the
confident half.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
