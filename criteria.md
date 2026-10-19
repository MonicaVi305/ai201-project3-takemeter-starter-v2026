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

## 1. Overall accuracy

Overall accuracy on the held-out test split is at least **0.80**.

**Why this target:**

My 200 labels split 70% `analysis`, 21% `reaction`, 9% `hot_take`. A model
that always predicts `analysis` already scores about 70% accuracy on a
stratified 30-post test split (21 of 30 posts are `analysis`). An accuracy
target only means something if it clears that majority-class floor by a
real margin — 0.80 requires getting several non-`analysis` posts right on
top of the `analysis` ones, which a baseline guesser can't do.

---

## 2. Balance (macro F1)

Macro F1 — the three per-label F1 scores averaged with equal weight,
regardless of label size — on the held-out test split is at least **0.55**.

**Why this target:**

A model that always predicts `analysis` scores roughly 0.82 F1 on
`analysis` and 0 on `reaction` and `hot_take`, for a macro F1 around 0.27.
Accuracy alone can't catch that collapse, because `analysis` is 70% of the
data — macro F1 can, since it weights all three labels equally. Asking for
at least 0.55, roughly double the degenerate baseline, means the model has
to be doing real work on both minority labels, not just leaning on how
rare they are.

---

## 3. Per-label performance — `analysis`

F1 for `analysis` on the held-out test split is at least **0.80**.

**Why this target:**

`analysis` has 140 of my 200 examples, about 21 in the stratified test
split — by far my best-represented label. There's no small-sample excuse
here, so I'm holding it close to what I'd consider a ceiling for this
task.

---

## 4. Per-label performance — `hot_take`

F1 for `hot_take` on the held-out test split is at least **0.35**.

**Why this target:**

`hot_take` is only 18 of my 200 examples, which leaves roughly 3 posts in
a stratified 15% test split. With that few examples, a single right or
wrong call swings F1 by about a third, so a strict target would really be
measuring luck instead of the model. 0.35 is loose enough to survive that
variance but still rules out the model never predicting `hot_take` at
all, which would score 0.

---

## 5. Consistency

Agreement with the staff-labeled 30-post calibration set (via
`agreement.py`) is at least **0.70**.

**Why this target:**

Staff's definitions for these three labels (`data/staff_taxonomy.md`) are
close to the ones I wrote in Milestone 2 — including the same rule that a
checkable fact has to be *load-bearing* to count as `analysis`, not just
present. I expect real alignment because of that. But staff deliberately
picked several of the 30 posts for being genuinely ambiguous, and their
own documentation says a rate "in the 70s is normal and expected" for
this set — so 0.70 is an honest floor based on what they told me to
expect going in, not a stretch target.

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
