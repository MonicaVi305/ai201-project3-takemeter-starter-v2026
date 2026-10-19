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

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Pre-labelling disclosure:**

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
