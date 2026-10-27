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

Overall accuracy on the held-out test split is at least 0.60, holding across all three seeds in unit 6.

**Why this target:** I have four labels (`analysis`, `hot_take`, `reaction`, `plug`), so a uniform coin-flip guesser clears about 25%. 0.60 is comfortably more than double that floor, which is the bar for "this learned the taxonomy rather than the label frequencies." I'm not setting it higher than that because 200 posts fine-tuning a small model is not a lot of signal to work with, and a target I can't realistically defend clearing isn't a target, it's a wish.

---

## 2. Per-label performance

Every label has an F1 of at least 0.50 on the held-out test split, holding across all three seeds.

**Why this target:** I hand-classified the 40 real posts I read for the taxonomy under my own four labels before writing this: 11 `analysis`, 12 `hot_take`, 9 `reaction`, 8 `plug`. `plug` is already the smallest at 20% of that sample. Scaled to 200 posts that's roughly 40 examples, which lands close to the "about 8 in the test split" case `criteria.md`'s own guidance warns gets jumpy between seeds. I'm setting the per-label floor lower than the 0.60 overall-accuracy target specifically because of that — `plug` or `reaction` missing this by a point or two on one seed because of a small test slice is a sample-size problem, not a taxonomy problem, and I don't want to confuse the two later.

---

## 3. Balance

No single label makes up more than 40% of `labels.csv`, and none makes up less than 15%.

**Why this target:** this one is grounded in what I actually saw, not a guess. My real 40-post sample split 27.5% / 30% / 22.5% / 20% across the four labels with no manual rebalancing — close to even on its own, because the four labels track real, common ways people argue on Hacker News rather than something I had to go hunting for examples of. 40%/15% gives room for the natural skew to move a bit as I collect the rest of the 200 (some labels are just more common there) without quietly letting one label swallow the dataset the way "good/bad" would.

---

## 4. Consistency

Agreement with the staff labels on the 30-post calibration set (`data/staff_posts.csv`) is at least 24 of 30 (80%).

**Why this target:** this measures something uncomfortable but useful — whether I apply a clearly written rule the same way its authors did, which is the same skill that decided whether I labelled my own 200 consistently. 100% isn't the right target: the README for that exercise says plainly that several of the 30 are chosen because they're genuinely ambiguous and I should be winning some of those arguments, not conceding all of them. 80% leaves room for a handful of defensible, argued disagreements on the hard ones while still requiring me to get the clear-cut majority right.

---

## 5. Confidence

Among predictions where the model's top predicted probability is at least 0.80, accuracy is at least 0.75, holding across all three seeds.

**Why this target:** a confident wrong answer is worse than an unsure wrong answer, because it doesn't ask for a second look. 0.75 sits meaningfully above the 0.60 overall-accuracy floor in criterion 1 — confident predictions should be a real cut above the average case, not just the same accuracy relabelled — but I'm not demanding near-perfect calibration, because with only ~200 training examples I'd expect the model's confidence itself to be somewhat noisy, not perfectly tuned.

> **Revised in unit 6:** Among the third of predictions the model was most confident about, accuracy is at least 0.75, holding across all three seeds.
>
> **Why revised:** the original names an absolute probability cutoff (≥0.80), but the notebook's `confidence_report()` in `takemeter.ipynb` section 5 doesn't expose a per-example probability I can threshold against — it only computes accuracy on the top and bottom thirds by confidence, as an aggregate. I can't produce a verdict on "≥0.80" from what the harness actually measures, only on "the most confident third," which is a genuinely different quantity (a relative rank, not an absolute threshold) — not a looser version of the same thing, a different thing entirely. This is a measurability gap, not a target I found inconvenient: I'm revising to the number I can actually check.

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
