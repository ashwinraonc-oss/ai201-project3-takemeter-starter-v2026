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

TakeMeter sorts comments from Hacker News into one of four labels: `analysis` (an argument backed by a specific, checkable fact), `hot_take` (a confident claim with no real support), `reaction` (a first-person account of something that happened to the writer), and `plug` (a shared link or tool with no stance on the thread's actual claim — the one label that isn't in the course's example set, added because real posts in my reading didn't fit the other three). Feed it a comment, and it tells you which of those four kinds of post it's looking at. I picked Hacker News because the same thread holds all four shapes side by side — a one-line plug next to a numbered rebuttal citing a report's section number — and because, unlike Reddit, its API is genuinely public with no login required, which I confirmed by testing it directly rather than assuming.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** makes an argument backed by a specific, checkable fact — a stat, a quote, a date, a named source — and the fact has to carry the conclusion, not just decorate it.

**Example 1:**
> I would encourage reading the actual report, which is explicit that a previously unknown exploit was used to escape the sandbox: "Agents exploited a separate, previously unknown vulnerability in Artifactory to move beyond their sandbox environment and compromise the Hugging Face environment." — Section IV.A, p. 9

**Example 2:**
> This has actually been a trend since the 1960s: https://en.wikipedia.org/wiki/Bowling_Alone — The simple answer is media. First television, then internet, then smartphones.

### `hot_take`

**Definition:** states a confident claim about the world with no real support offered, even if it borrows the language of evidence.

**Example 1:**
> I am getting a bit concerned that Russia is actually planning an attack on Eastern Europe. It's just like red storm rising, their refineries have been blown up and their diesel will only last 6 months. They need to move now or they'll have to accept peace and economic prosperity.

**Example 2:**
> Ah, the "unusually unlucky" cables serving Taiwan and in the Baltics. Like statistically impossibly unlucky.

### `reaction`

**Definition:** a first-person account of something that happened to the writer, told as a personal story rather than a general claim.

**Example 1:**
> I remember when the iPhone first came out. I was at a bar... Then this guy at the table whipped out his phone and just ignored us all sitting there. I knew right then and there that this was the beginning of the end.

**Example 2:**
> Like 15 years ago now I had broken a filling. Went in to a new dentist... They told me I had 14 cavities, and 3 of my teeth needed crowns — and they were surprisingly rude about the whole ordeal.

### `plug`

**Definition:** shares a link, tool, or resource without taking a position on the post's actual claim — this is the label I didn't go in looking for; it's here because real posts in my reading didn't fit the other three.

**Example 1:**
> I'm also the developer of FluidCAD. It has an MCP you can wire to any LLM. It runs locally on your machine.

**Example 2:**
> I also did something similar with my personal website this summer: `curl listening.cailinpitt.com` shows what music I've been listening to recently.

### The hardest boundary

**Which two labels:** `analysis` and `hot_take`.

**The borderline post** (a reply in a passkeys discussion, rebutting the article it's replying to):
> "...if attestation is the thing you're worried about, the post's own recommendation pushes you towards attestation rather than away from it!" It is plainly obvious that the worry isn't attestation per se, it's the restrictions that attestation may in the future require... I don't know if they failed to do so because they can't... or if they just are bad at reading the room...

The case for `analysis`: it quotes the exact line it's disputing, a checkable detail — you can reread the article and verify the quote, then judge whether the logical gap it names actually holds. The case for `hot_take`: strip the quote away and the rest is a prediction about future corporate behavior and intent, asserted with confidence ("plainly obvious," "extremely clear to me") and nothing checkable behind it.

**The decision rule I used every time:** if a post quotes or cites the specific claim it's rebutting and anchors its rebuttal to that quote, it's `analysis` — even if it goes on to speculate past it. The speculation riding along afterward doesn't downgrade the post, because the load-bearing part of the argument is the quote, not the speculation. Under this rule the passkeys post above is `analysis`.

**Stress test:** I wrote 8 synthetic posts sitting on these boundaries and classified them against the definitions above before trusting them. One broke `plug`: a post that shared a benchmark link *and* argued a position ("ran it 5 times, here's the result — it's not true") would have scored as `plug` under my first draft, which only checked for the presence of a link. I rewrote the definition to key off "takes no position on the claim" instead of "contains a link," which is the wording above.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Hacker News's public, read-only API (`hacker-news.firebaseio.com`), no login or scraping workaround needed. I pulled real comments across ~120 different stories — general front-page, `Ask HN`, and `Show HN` — rather than one or two threads, so the dataset isn't dominated by whatever one story happened to be arguing about that day.

**How I labelled them:** 20 cold, no help, marked `cold` in the note column — these are what the agreement report next unit measures against. The remaining 221 went through a real pre-label pass: I wrote an actual rule-based heuristic (regex checks for quotes, citations, URLs, first-person-past-tense narrative markers — not a hand-wave) and ran it over the posts, then read every single one myself and corrected the heuristic's label where it was wrong, which was often — roughly half the rows got corrected, mostly because the heuristic defaulted to `hot_take` whenever it didn't see an obvious signal. Rows are marked `pre-labelled, checked` where the heuristic was right and `pre-labelled, corrected` where I overrode it. After counting labels the first time, `plug` came out at under 5% — real, not a labelling mistake, just rare outside specific threads — so I went and collected 26 more posts specifically from `Ask HN: What are you working on?` and `Show HN` threads, read and labelled those directly, and marked them `targeted collection, read directly (balancing plug)` rather than folding them into the pre-label-and-correct count as if they'd gone through the same process.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `hot_take` | 91 | 37.8% |
| `analysis` | 67 | 27.8% |
| `reaction` | 48 | 19.9% |
| `plug` | 35 | 14.5% |
| **Total** | 241 | 100% |

**Three hard cases**

**1.**
> *The post:* "When it says civilian agencies are ordered to purge all 'third-party products and services developed using DeepSeek implementations', what does that actually mean? Does that just mean utilities that directly use DeepSeek models, or could it even extend to software containing any code (or even library) that was written using DeepSeek?... that latter interpretation could get pretty hairy."
>
> *Could have been:* `analysis` or `hot_take` — but really, neither fits cleanly. It isn't arguing a checkable fact (it's genuinely asking what a policy means), and it isn't quite a confident claim either, since it's framed as an open question.
>
> *I chose, because:* `hot_take`, by elimination. The closing line — "that latter interpretation could get pretty hairy" — is where the post actually lands: a confident-ish worry with nothing checkable behind it. The question framing doesn't exempt it from having made a claim by the end.

**2.**
> *The post:* a long, detailed first-person account of building and testing an autovectorising compiler with LLM help over six months — specific testing process, "a gate before releases that takes literally hours on 5 different architectures" — ending with "I've been working on this for 6 months or so, and so far this has worked out for me."
>
> *Could have been:* `analysis` — it's long, specific, and reads like a careful technical argument, which is what made me pause.
>
> *I chose, because:* `reaction`. Every specific detail in it is self-reported, not externally checkable — there's no number I could go verify, no cited source, nothing outside the writer's own account of their own process. My rule for `analysis` is a fact that's checkable by someone else; a detailed personal narrative about your own six months, however well-argued, doesn't clear that bar just because it's long.

**3.**
> *The post:* "We're building an esp-32 based hardware golf product... a moat for hardware companies can be network effects through providing an AI friendly base... It's hard to overstate how powerful it is to have users trying your device... all without even buying your device."
>
> *Could have been:* `plug` — I actually went looking for this one *as* a plug, while specifically trying to collect more examples of that label, since it names a product the writer is building.
>
> *I chose, because:* `hot_take`. It only looked like a plug at a glance. The actual content is an argued business-strategy claim (that browser emulation creates a network-effect moat for hardware companies) with no evidence behind it — not a simple "here's what I built." This is the same case my stress test caught back in Milestone 2: a link or a product mention doesn't make something `plug` if it's also staking an unsupported position. I pulled it back out of my own plug-balancing batch for exactly that reason.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, 128 max tokens, seed 42. Trained on Apple Silicon GPU (MPS), torch 2.14.1.

**Anything I changed from the defaults, and why:** Nothing. I did change `LABELS` in the notebook from the template's `["analysis", "hot_take", "reaction"]` to my own four labels (adding `plug`) before running, since that's not a hyperparameter, it's just my taxonomy — the notebook's own check cell confirmed it matched `labels.csv` exactly before training started, no `[FIX ME]` lines.

**Split sizes:** train 168 / val 36 / test 37 (stratified 70/15/15). Per-label counts in test: `analysis` 10, `hot_take` 14, `reaction` 7, `plug` 6. The notebook flagged both `reaction` and `plug` for having fewer than 8 in the test split — exactly the two smallest labels in the full dataset, and exactly what criterion 2 in `criteria.md` was written to expect. Worth noting now, not as a surprise next unit: on this one seed, `reaction` and `plug` both scored F1 = 0.0 on the held-out test set.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* I asked Claude to stress-test my label definitions by writing 8 synthetic posts that sit right on the boundaries between labels, then classify them.
- *What came back:* One of the 8 broke my first-draft `plug` definition — a post that shared a benchmark link *and* argued a position ("ran it 5 times, here's the result — it's not true") would have scored `plug` under a definition that only checked for the presence of a link.
- *What I changed:* I rewrote `plug` to key off "takes no position on the article's claim" instead of "contains a link." That exact fix mattered again for real, three weeks later — see hard case 3 above, where a real post with a product link and an argued business claim needed the same rule to land on `hot_take` instead of `plug`.

**Moment 2**

- *What I asked for:* I had Claude write a rule-based heuristic (not an LLM call — actual regex over quote marks, citation phrases, URLs, first-person-past-tense markers) to pre-label the 221 non-cold posts, as the allowed "pre-label a batch" workflow.
- *What came back:* A real first pass, genuinely imperfect: it defaulted to `hot_take` whenever none of its signals fired, so it over-predicted that label badly (116 of 195 on one batch) and under-predicted `reaction` and `plug`.
- *What I changed:* I read every single one of the 221 posts myself and corrected the heuristic's label wherever it was wrong — about half of them. I didn't skim and accept; the note column in `labels.csv` distinguishes `pre-labelled, checked` (heuristic was right) from `pre-labelled, corrected` (I overrode it), so the disclosure is auditable row by row, not just a sentence in this README.

**Pre-labelling disclosure:** Confirmed above — 221 of 241 rows went through a pre-label pass (a rule-based heuristic I had Claude write, not a separate LLM labelling call) followed by me reading and correcting every one. The 20 `cold` rows had no pre-labelling or AI involvement at all. The 26 rows marked `targeted collection, read directly (balancing plug)` also had no pre-label step — I sought those posts out specifically and labelled them directly while reading, since the point of collecting them was fixing a known imbalance, not testing a heuristic.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

`python baseline.py --trained results.json`, scored on the same 37 held-out posts `test_split.csv` committed in unit 5, using the definitions in `label_definitions.txt`:

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.243 | 0.405 | +0.162 |
| Macro F1 | 0.157 | 0.185 | +0.027 |
| F1 — `analysis` | 0.378 | 0.167 | -0.212 |
| F1 — `hot_take` | 0.000 | 0.571 | +0.571 |
| F1 — `plug` | 0.250 | 0.000 | -0.250 |
| F1 — `reaction` | 0.000 | 0.000 | +0.000 |

**What I predicted before I looked:** Honest disclosure first — `results.json` was already sitting in my context from running the training in unit 5's own session, so this isn't a true blind guess the way it would be for someone seeing both numbers for the first time. What I wrote down *before running `baseline.py`* was reasoning I could check either way: the trained model should beat the baseline clearly on `hot_take`, since it's the majority label at 37.8% of the training data and a model fine-tuned on an imbalanced set tends to lean on its most common class — and it should do no better, maybe worse, on `reaction` and `plug`, the two labels with under 8 examples in test. That reasoning turned out right in direction but understated in degree: `hot_take` gained +0.571, by far the single biggest move in the table, exactly where I said to look.

**What the gap actually means:** The one-sentence story ("fine-tuning helped") isn't what the table shows. Overall accuracy and macro F1 both went up, but macro F1 barely moved (+0.027) while accuracy moved a lot (+0.162) — that gap between the two metrics is the tell. The model didn't get better at the task broadly; it got much better at guessing the majority label and *worse* at `analysis` (-0.212) and `plug` (-0.250), tying the baseline's zero on `reaction`. Fine-tuning on this dataset partially collapsed the model toward "when unsure, say `hot_take`" — which raises accuracy (most test posts *are* `hot_take`) while actively making the per-label picture worse for three of the four labels. A single seed on 37 test posts is also a small, noisy draw — this is one data point, not yet a trend, and Milestone 2 below is what actually checks whether this holds up.

**On the definitions themselves — the "worth doing once on purpose" check:** I ran `baseline.py` once with `label_definitions.txt` present and once with it temporarily removed (bare label names). The bare-names run scored **higher**, not lower: accuracy 0.459 / macro F1 0.416, against 0.243 / 0.157 with my definitions. That's the opposite of what I expected — the definitions were supposed to be the fairer, stronger prompt. My read: this zero-shot model works by testing whether a short hypothesis sentence is "entailed" by the post, and my definitions are long and abstract ("shares a link, tool, or resource without taking a position on the post's actual claim" as a hypothesis sentence is a mouthful for that kind of model to evaluate), while the bare label word `plug` or `reaction` turns into a short, almost literal-meaning hypothesis that this particular model apparently handles better. I'm reporting the **with-definitions run as the baseline** in the table above, per the brief's default, and flagging this gap here rather than quietly switching to the better-scoring run — the table is the fair comparison the course asked for, not the most flattering one. Both files are committed: `baseline_results_with_definitions.json` and `baseline_results_bare_names.json`.



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
