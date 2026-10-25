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

This classifier sorts public IMDb user movie reviews into three categories
based on how the reviewer backs up their opinion: do they argue it with a
specific, checkable detail (`analysis`), state it as a flat verdict with
nothing behind it (`hot_take`), or center the post on their own in-the-moment
experience of watching (`reaction`)? The 200 reviews cover 200 different
films, so no single plot or release dominates the set, and spoilers for one
movie don't make the rest unreadable.

**Candidate distinctions noticed while reading (Milestone 1):**
- Some reviews cite a specific, checkable detail — an actor's name used to
  make a point, a director's other films, a particular scene, a numeric
  score — while others just assert a verdict and move on.
- Some reviews are about the reviewer's own experience in the moment
  (walked out, cried, rewatched it twice) rather than about the film as an
  object to be judged.
- Longer reviews tend to walk through plot and performance point by point;
  one- or two-line reviews tend to be a single blunt judgment.
- Some reviews argue against an anticipated disagreement ("I know everyone
  loved this, but...") while others just state a feeling and stop there.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** Backs its judgment of the film with at least one specific,
checkable detail that's actually used to support the claim, not just named
in passing.

**Example 1:**
> "...I did admire the acting. Jane Fonda and Robert De Niro are great in
> this movie... there is a great scene where he has to take his father to
> a home for elderly people because he can't care for him anymore that
> will break your heart." (imdb-train-001)

**Example 2:**
> "The wonderful music of George Gerschwin complements the great dancing
> by Gene Kelly and Leslie Caron. 'An American in Paris' is a humorous,
> light-hearted, loving film well worth watching. 8/10" (imdb-train-003)

### `hot_take`

**Definition:** States a confident verdict on the film without using any
detail to argue for it — names, scenes, or numbers can still appear, but
they decorate the opinion instead of supporting it.

**Example 1:**
> "This is probably the worst excuse for television programming since,
> oh, I don't know, WHATS HAPPENING NOW?... This show is TOXIC!"
> (imdb-train-022)

**Example 2:**
> "Lots of singing and dancing in this one, especially by Gene Kelly...
> Musical lovers only." (imdb-train-028)

### `reaction`

**Definition:** Centers on the reviewer's own in-the-moment experience of
watching — what they felt or did — rather than building a case about the
film itself.

**Example 1:**
> "Any movie that has nude scenes of Karen Allen and I'm still so bored I
> walk out, that is a stinker!... I walked out." (imdb-train-005)

**Example 2:**
> "I watched it on Saturday and I still cannot get it out of my head.
> AMAZING and sad all at the same time... Make sure you have plenty of
> Kleenex..." (imdb-train-008)

### The hardest boundary

**Which two labels:** `analysis` and `hot_take`

**The decision rule I used every time:**
<!-- e.g. "If the post names a specific checkable fact, it's `analysis`, even
     if the tone is heated." -->

A named actor, director credit, or numeric score only counts toward
`analysis` if the post *uses* it to support the claim. If the fact is just
dropped into a rant with nothing connecting it to a reason, it's still
`hot_take` — even though it looks checkable on its own.

The post that made me write this rule down: imdb-train-049 names three real
actors (Coburn, Hackman, Bergen) and gives a literal score ("4 Horses/10")
— the same ingredients as the `analysis` examples above — but neither does
any work. They're just listed on the way to "Boring, tedious, and very
tiresome waste of celluloid." Nothing in the post explains *why* those
actors or that score support the verdict, so it stayed `hot_take`.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public IMDb user movie reviews — real reviews
ordinary users posted under a film's IMDb page, covering 200 different films
so no single plot or release dominates the set.

**How I labelled them:** I labelled the first 20 posts (`imdb-train-001`
through `imdb-train-020`) cold — no help, no tools, just my own definitions
and decision rule from Milestone 2. Those rows are marked `cold` in the
`note` column. For the remaining 180, I gave an AI tool my label
definitions and decision rule and had it suggest a label for each post,
then I read every single one and corrected any label that didn't actually
match my taxonomy before keeping it. Those rows are marked
`pre-labelled, corrected` in the `note` column.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `analysis` | 140 | 70.0% |
| `reaction` | 42 | 21.0% |
| `hot_take` | 18 | 9.0% |
| **Total** | 200 | 100% |

`analysis` sits right at the 70% ceiling. It isn't over the line, but it's
close enough that I'd add more `hot_take` and `reaction` examples first if
I were collecting again — the per-label `hot_take` target and the macro-F1
balance criterion in `criteria.md` are both set as loosely as they are
partly because of this.

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* `imdb-train-049` — "Ohhh the brutality, ohhhhh the dying
> breed... in light of Coburn/Hackman/Bergen's presence... 4 Horses/10-all
> deader'n hell."
>
> *Could have been:* `analysis` or `hot_take`. It names three real actors
> and gives a literal numeric score — the same ingredients my `analysis`
> examples use.
>
> *I chose `hot_take`, because:* neither the actors' names nor the score
> are used to argue anything — they're dropped into a rant on the way to
> "boring, tedious, and very tiresome." This is the post that made me
> write the "the fact has to do work" rule down.

**2.**
> *The post:* `imdb-train-038` — "Barney seems to be utterly boring and
> uninspired 'content-wise'... I was never bored seeing Alejandro
> Jodorowsky's movies, while Drawing Restraint 9 was an utter
> disappointment... I want more Jodorowsky!!!"
>
> *Could have been:* `reaction` or `hot_take`. It's written in the first
> person and reports the writer's own experience across several films
> ("I was never bored... I am afraid...").
>
> *I chose `hot_take`, because:* it isn't reporting one immediate
> experience — it's a general claim about the director's whole body of
> work being empty and pretentious. Staff's own rule 2 (`reaction` vs.
> `hot_take`) calls this exactly: a post about a claim, not about what
> just happened to the writer.

**3.**
> *The post:* `imdb-train-056` — "I am an avid movie goer and have seen
> previews for this movie for months... Then I went and saw it, I was
> sunk down in my seat almost asleep... Maybe even check out The Kings of
> Comedy again."
>
> *Could have been:* `analysis` or `reaction`. It names a specific
> comparison film ("The Kings of Comedy"), which looks like the kind of
> checkable detail `analysis` needs.
>
> *I chose `reaction`, because:* the comparison is a throwaway aside, not
> the reason for the verdict — the whole post is actually about the
> writer's own experience sitting in the theater (hyped up, then nearly
> falling asleep). The named film doesn't do any argumentative work, so
> the same rule that moved `imdb-train-049` to `hot_take` keeps this one
> out of `analysis` too.

**What the pattern across these three tells me:** all three hard cases
turn on the same test — does the checkable detail (a name, a score, a
comparison) actually carry the argument, or is it just along for the
ride? That's the rule from Milestone 2's hardest boundary, and it ended up
doing more work than I expected across labels I didn't originally write it
for.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42 

**Anything I changed from the defaults, and why:** Nothing — ran the
notebook's defaults as given.

**Split sizes:** Stratified 70/15/15 → train 139, val 31, test 30.
Per-label counts in the test split: `analysis` 21, `reaction` 6,
`hot_take` 3. Both `reaction` and `hot_take` are under the ~8-example
mark the notebook flags — `hot_take` especially, at only 3 posts, where
getting a single one right or wrong swings its F1 by a third. That's
exactly why Milestone 3's `hot_take` target (≥ 0.35) was set loose going
in, rather than as a reaction to seeing this number.

**What I trained on:** CPU (`torch 2.14.1+cpu`), no GPU/MPS available.
Recorded in `results.json` as `device`/`device_name`/`torch_version` —
noting it here too since different hardware can produce different numbers
from the same seed.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* With my three draft label definitions (close to the
  assignment's own worked example at that point), I asked for the
  strongest case for each label on a post that felt borderline, and to
  surface any real post from my 200 that sat ambiguously between
  `analysis` and `hot_take` rather than a made-up one.
- *What came back:* `imdb-train-049` — a review that names three real
  actors (Coburn, Hackman, Bergen) and gives a literal numeric score
  ("4 Horses/10"), the same surface ingredients as my `analysis`
  examples, but spends the whole post ranting rather than arguing from
  either.
- *What I changed:* Rewrote the `analysis` and `hot_take` definitions so
  neither is a copy of the assignment's example wording anymore, and
  added the explicit decision rule — a checkable detail only counts
  toward `analysis` if the post *uses* it to support the claim, not just
  names it. That rule is what Milestone 2's "hardest boundary" section
  and two of the three hard cases in "The Dataset" are built on.

**Moment 2**

- *What I asked for:* Pre-labels for the 180 non-cold posts, given my
  label definitions and the decision rule above, as a first pass to read
  and correct rather than to keep as-is.
- *What came back:* A suggested label for each of the 180 posts.
- *What I changed:* Read every single one against my own definitions and
  corrected any that didn't actually hold up — most often cases where a
  post had just enough of a "fact" in it (a name, a rating) to look like
  `analysis` on a skim but wasn't actually arguing from it, the same
  failure mode as `imdb-train-049` above. I kept the 20 cold-labeled
  posts (`imdb-train-001` through `-020`) completely separate from this
  process — no AI involvement there at all, by design.

**Pre-labelling disclosure:** Yes. 20 of the 200 rows were labeled cold
(no tools, no help) and are marked `cold` in `labels.csv`'s `note`
column. The other 180 were AI-pre-labeled against my definitions and
decision rule, then individually read and corrected before being kept —
marked `pre-labelled, corrected` in the same column. This is also
disclosed in "The Dataset" section above.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.633 | 0.700 | +0.067 |
| Macro F1 | 0.322 | 0.275 | -0.047 |
| F1 — `analysis` | 0.766 | 0.824 | +0.058 |
| F1 — `hot_take` | 0.000 | 0.000 | +0.000 |
| F1 — `reaction` | 0.200 | 0.000 | -0.200 |

**Which baseline this is:** the table above is the run **with** my
definitions from `label_definitions.txt` (`baseline_results.json`). I also ran
it once on purpose with bare label names only (`baseline_results_bare.json`),
on the same 30 held-out posts:

| Baseline prompt | Accuracy | Macro F1 |
|---|---|---|
| Bare label names | 0.133 | 0.104 |
| My definitions | 0.633 | 0.322 |
| **Gap (what the definitions were worth)** | **+0.500** | **+0.218** |

The bare-names run is not the baseline I'm reporting. Its predictions were 21
`reaction`, 8 `hot_take`, 1 `analysis`, on a held-out set that is 21/30
`analysis` — the bare names mean almost nothing to the model. The with-definitions
run predicted 26 `analysis` and 4 `reaction`, never `hot_take`, so much of its
gain is leaning toward the majority class rather than discriminating between
labels. Both baselines are below the 0.70 accuracy that always guessing
`analysis` would score.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
     The prediction: on my balanced file, answering analysis every time is right only 10 of 30 times. So both models would drop to about 0.33 accuracy, roughly one in three. Their good-looking scores now come mostly from the held-out set being mostly analysis, not from the models telling the labels apart.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

Fine-tuning added almost nothing. The trained model is +0.067 on accuracy
(0.700 vs 0.633) but -0.047 on macro F1 (0.275 vs 0.322), and it loses the
only minority-label signal the baseline had (`reaction` F1 0.200 → 0.000).
Its accuracy is exactly the 0.70 you get from always guessing `analysis`
(21 of 30 test posts), because that is what it does: the confusion matrix
shows every test post called `analysis`. The baseline at least tried to tell
labels apart and the trained model did not, so on this data the zero-shot
baseline is the more useful of the two. My prediction above assumed a
balanced file; my real test set is 21/30 `analysis`, which is why both
models land near or above the 0.70 floor instead of 0.33.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.80 | 0.700 | 0.700 | 0.700 | MISSED |
| 2. Macro F1 | ≥ 0.55 | 0.275 | 0.275 | 0.275 | MISSED |
| 3. `analysis` F1 | ≥ 0.80 | 0.824 | 0.824 | 0.824 | MET |
| 4. `hot_take` F1 | ≥ 0.35 | 0.000 | 0.000 | 0.000 | MISSED |
| 5. Agreement with staff set | ≥ 0.70 | 1.000 | 1.000 | 1.000 | MET (one comparison, not per-seed) |

Spread across seeds is 0.000 on every measure. Run on CPU (torch 2.14.1+cpu), 3 epochs, learning rate 2e-5, batch size 16, max length 128, 30 test posts per seed. Raw numbers are in `results_three_seeds_before.json`.

**What the three seeds show:** All three seeds gave accuracy 0.700 and macro F1 0.275. Validation accuracy was 0.710 and macro F1 0.277 at every epoch of every seed, so the model never improved during training. The model is predicting `analysis` for every post. `analysis` is 70% of my 200 labels (140 posts) and `hot_take` is only 18, so the model collapsed onto the majority class. `hot_take` and `reaction` both scored F1 0.000, and `analysis` F1 is 0.824 only because it is the one label the model ever guesses.

The zero spread is not a sign of a stable, good result. The seeds change which posts land in the test split, but the model gives the same answer on every split. The 0.700 accuracy is the share of `analysis` posts in the test set (21 of 30), not evidence that the model can tell the labels apart.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 21 | 0 | 0 |
| **hot_take** | 3 | 0 | 0 |
| **reaction** | 6 | 0 | 0 |

(Seed 42, 30 test posts. Seeds 7 and 2024 produced the identical matrix.)

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

6 real `reaction` posts were called `analysis` (and all 3 `hot_take` posts were too, with nothing going the other way): the model never learned the `reaction` or `hot_take` boundaries and labeled every test post `analysis`, so the 0.70 accuracy is just the `analysis` share of the test set (21 of 30).


---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.80 | MISSED | 0.700 on all three seeds, below 0.80 every time. It is also only the majority-class floor, not skill. |
| 2 | Macro F1 | ≥ 0.55 | MISSED | 0.275 on all three seeds, about half the target and equal to the always-`analysis` degenerate score. |
| 3 | `analysis` F1 | ≥ 0.80 | MET | 0.824 on all three seeds. Holds across seeds, but it is the one label the model ever predicts, so this is not a sign of a good model. |
| 4 | `hot_take` F1 | ≥ 0.35 | MISSED | 0.000 on all three seeds. The model never predicted `hot_take`. |
| 5 | Agreement with staff set | ≥ 0.70 | MET | 30 / 30 = 100% against the staff labels, see the Agreement Report below. |

**Diagnoses**

**Criterion 1, overall accuracy (0.700, target 0.80). Cause: too few examples, so the model collapsed onto the majority label.**
0.700 is exactly the share of `analysis` in the test split (21 of 30), and the
confusion matrix has all 30 test posts in the `analysis` column. So the
accuracy is the "always guess `analysis`" floor, not skill. `analysis` is 140
of 200 labels (70%), so the model can reach 0.70 without learning anything
else.

**Criterion 2, macro F1 (0.275, target 0.55). Cause: the same collapse.**
Macro F1 averages the three labels equally: (0.824 + 0 + 0) / 3 = 0.275.
It is the degenerate always-`analysis` score, which my own criteria.md
predicted before training. Evidence it is the data and training and not bad
luck: validation accuracy (0.710) and macro F1 (0.277) did not move across
any of the 3 epochs in any seed, and all three seeds, which use three
different splits, gave identical test numbers. With 3 epochs, learning rate
2e-5 and 70% of the data in one label, the model never left the cheapest
answer.

**Criterion 4, `hot_take` F1 (0.000, target 0.35). Cause: too few `hot_take` examples, made worse by a tiny test split.**
`hot_take` is 18 of 200 labels (9%), about 12 posts in each training split,
against about 97 `analysis`. The model predicted `hot_take` zero times out of
30, so the 0.000 is not noise: even a loose 0.35 needs at least one correct
`hot_take` prediction. The test split holds only 3 `hot_take` posts, so any
working model would still have a jumpy F1 here, but that explains how noisy
the number would be, not a score of exactly 0 on every seed.

**What I ruled out.** *A genuinely hard label pair:* the off-diagonal cells
are all in the `analysis` column and none between `hot_take` and `reaction`,
so there is no specific pair the model is confusing; it isn't separating
anything. *A task the model can't reach from 200 examples:* I can't rule this
out yet. The model did not train usefully on this imbalanced data, so I
can't say what it would do on balanced data. The Milestone 5 re-run tests it.
*An inconsistently applied boundary in my own labels:* see below.

**Labelling vs. model:** The agreement report points at the model and the
data balance, not my labelling. I matched staff on 30 / 30 posts under their
written rules, so I applied a rule consistently when it was written down.
That makes "a boundary I applied inconsistently" an unlikely cause of the
zero F1 on `hot_take` and `reaction`. Because I had no disagreements, I
can't point to a label pair I split differently from staff, so I can't tie
any miss to a specific pair. The limit on this evidence is that the staff
set is balanced and mine is not, and 100% is above the 70s staff expected,
so it rules out sloppy application of a rule but does not prove all 200
labels are right.

**Checking my verdicts the other way.** *Criterion 3 (`analysis` F1 0.824,
MET):* the strongest case for MISSED is that the model only reaches 0.824
because it guesses `analysis` every time, so it isn't really performing well.
That is a true caveat, but the criterion as written is a number on one
label, 0.824 clears 0.80 on all three seeds, and the weakness belongs to
criteria 2 and 4, so I'm keeping MET. *Criterion 5 (agreement 1.000, MET):*
the case against is that it is one comparison and not three seeds, and that
100% is above what staff expected. I kept MET because the rate is
well above 0.70 and no part of it is a close call.

**Criterion changes:** none. All five criteria were measurable and I missed
three of them, so nothing is being revised and the original targets stand.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->
**Agreement rate:** 30 / 30 = 100% (`agreement_results.json`)

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

There were no disagreements, so there is nothing to adjudicate and no posts to
sort into piles (a), (b) or (c).

**What the pattern in my disagreements tells me:**

I matched staff on all 30 posts, so there is no pattern of disagreement to
read. Two honest caveats. First, 100% is higher than the "in the 70s" staff
said is normal for this set, which was built with several ambiguous posts, so
I can't claim the set was easy or that every call was obvious; I can only say
I applied the written rules the same way staff did on these 30. Second, the
staff set is balanced (10 of each label) while my own 200 labels are 70%
`analysis`, so this shows I can apply a written rule consistently, not that
every one of my 200 labels was right. It is evidence against a labelling
problem, not proof of its absence.


---

## The Improvement

**What I changed:** One training setting: epochs, from 3 to 10. Nothing
else moved: same data (`labels.csv`, 200 rows, no relabelling, no new posts),
same model, learning rate 2e-5, batch size 16, max length 128, same three
seeds (42, 7, 2024), same CPU. `RUN_LABEL` was set to `"after"`, so the
results are in `results_three_seeds_after.json` and the before numbers in
`results_three_seeds_before.json` were not overwritten.

**Which diagnosis pointed at it:** The diagnosis for criteria 1, 2 and 4 said
the model collapsed onto `analysis`, and the evidence was that validation
accuracy (0.710) and macro F1 (0.277) did not move at all across the 3
epochs. That points at training having stopped before the model left the
cheapest answer, so more epochs was the one listed fix that tests it. My
main diagnosis was also too few `hot_take` examples (18). I did not add
examples, because I can't collect and label 30 real posts in this run, so
that part of the diagnosis is untested.

### Run Log — Before and After

| Criterion | Target | Before: 42 / 7 / 2024 | After: 42 / 7 / 2024 | Verdict before | Verdict after |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.80 | 0.700 / 0.700 / 0.700 | 0.800 / 0.833 / 0.767 | MISSED | MISSED |
| 2. Macro F1 | ≥ 0.55 | 0.275 / 0.275 / 0.275 | 0.535 / 0.559 / 0.493 | MISSED | MISSED |
| 3. `analysis` F1 | ≥ 0.80 | 0.824 / 0.824 / 0.824 | 0.900 / 0.909 / 0.864 | MET | MET |
| 4. `hot_take` F1 | ≥ 0.35 | 0.000 / 0.000 / 0.000 | 0.000 / 0.000 / 0.000 | MISSED | MISSED |
| 5. Agreement with staff set | ≥ 0.70 | 1.000 | 1.000 | MET | MET |

Criterion 5 does not depend on training, so it is the same number. Seed 2024
is what keeps criterion 1 a miss (0.767), and 0.800 on seed 42 only just
reaches the target.

The `reaction` F1, which is not one of my five criteria, went from 0.000 in
all three seeds to 0.706 / 0.769 / 0.615.

**After confusion matrix (seed 42, 30 test posts; seeds 7 and 2024 differ by
1 to 2 posts per cell):**

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 18 | 0 | 3 |
| **hot_take** | 1 | 0 | 2 |
| **reaction** | 0 | 0 | 6 |

**Did it help, and how do I know:** Yes, partly. More epochs fixed the
collapse onto `analysis` but not the `hot_take` problem. How I know:
- Accuracy rose from 0.700 to 0.767 to 0.833 and macro F1 from 0.275 to
  0.493 to 0.559, on every seed. The model now predicts `reaction`: all 6
  real `reaction` posts were caught on seed 42.
- The validation scores moved this time. On seed 42 they stayed flat for 3
  epochs (as before) and then rose at epoch 4 to 5, so the earlier flat line
  was the model not having trained long enough, not a ceiling.
- `hot_take` is still 0.000 on all three seeds and the model predicted it
  zero times. With about 12 `hot_take` training posts, 10 epochs wasn't
  enough to learn it, which fits the too-few-examples diagnosis rather than
  the not-enough-training one.
- Verdicts did not change: criteria 1, 2 and 4 are still MISSED, so by my own
  targets the improvement did not get me over the line. Macro F1 is closest
  (0.559 on one seed, 0.493 on another, against 0.55).

Caution: the test split has 30 posts, so one post is 3.3 points of accuracy
and the three seeds differ by up to 6.6 points (0.767 to 0.833). The jump
from 0.700 is bigger than that spread, which is why I call it real, but I
would not trust the exact numbers to a point.

---

## What's Still Broken

**Criterion 1, overall accuracy (0.800 / 0.833 / 0.767 vs 0.80):** Missed on
seed 2024. I'd try a few more epochs and the class-weighting fix below. I
stopped at 10 epochs because the milestone allows one change and the gain
had mostly levelled off by epoch 5 to 7 in the logs.

**Criterion 2, macro F1 (0.535 / 0.559 / 0.493 vs 0.55):** Missed on two of
three seeds. The gap is almost entirely `hot_take` at 0.000, which pulls the
average down by a third regardless of the other two labels.

**Criterion 4, `hot_take` F1 (0.000 on all seeds vs 0.35):** Still missed.
What I'd do: collect 30 or more real `hot_take` posts labelled under the same
definition, and add class weights so the loss counts a missed `hot_take` more
than a missed `analysis`. I stopped because I only changed one setting this
run, and because new labelled data wasn't something I could add in this run.

**The gap between what I meant my labels to capture and what the model
learned:** I meant `hot_take` to be a distinct kind of post, but after 10
epochs the model still calls every real `hot_take` post `analysis` or
`reaction` (the matrix row is 1 / 0 / 2 on seed 42), so it never learned my
`hot_take` boundary. With only 18 examples, I can't tell from this whether
the boundary is hard to learn or just under-sampled.

---

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
