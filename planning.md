# AI201 — planning.md

## Community

**r/television** — a large, active Reddit community dedicated to discussion of TV shows across all genres and eras.

r/television is a strong fit for a classification task because its discourse is genuinely varied in a structured way. On any given day the same show will generate posts ranging from scene-by-scene narrative analysis to impulsive hot takes posted minutes after an episode drops. The community is also self-aware about discourse quality — regulars actively distinguish between "actual criticism" and "confident nonsense," which means the label distinctions are legible to the community itself, not just imposed from outside. The volume of text-heavy posts (200+ words is common) makes it tractable for NLP, and the breadth of shows discussed prevents the model from learning show-specific patterns instead of discourse patterns.

---

## Labels

### `critique`
A post that makes a structured argument about a show's writing, direction, acting, or narrative structure, supported by specific references to episodes, scenes, or comparisons to other works.

**Example 1:** "The Sopranos finale works because the cut to black mirrors Tony's entire arc — we've always been inside his paranoid POV, and now we're cut off from it just like he is. Compare this to Pine Barrens where ambiguity is played for comedy; here it becomes existential dread."

**Example 2:** "Succession season 4 deliberately slows its pacing after episode 3 — every scene with the siblings runs noticeably longer than in prior seasons, which makes the power vacuum feel physically uncomfortable to sit in. It's a structural choice, not a flaw."

---

### `hot_take`
A post that asserts a bold or contrarian opinion about a show's quality, characters, or cultural standing without providing reasoning or evidence — the claim itself is the entire content.

**Example 1:** "Game of Thrones was never actually good. Seasons 1–4 are just as hollow as 7–8, people just didn't notice yet."

**Example 2:** "Unpopular opinion: The Office should have ended at season 3. Everything after is a different, worse show and pretending otherwise is nostalgia talking."

---

### `reaction`
A post expressing an immediate emotional response to a specific episode or scene that recently aired, with little or no analytical content — the post is capturing a feeling in the moment rather than forming a judgment.

**Example 1:** "I cannot believe they just killed [character]. I am not okay. That finale absolutely destroyed me."

**Example 2:** "Just finished the White Lotus season 3 finale. Absolutely floored. Cannot stop thinking about that last shot. No spoilers but wow."

---

## Hard Edge Cases

### Pre-annotation decision rules

**critique vs. hot_take:** The hardest boundary is a post with confident, specific-sounding language that doesn't actually support its claim. Example: *"Better Call Saul is better than Breaking Bad because it trusts its audience more and the cinematography is more intentional."* This sounds like critique but the "evidence" is a restatement of the opinion.

**Decision rule:** Remove the opinion framing entirely. Does the remaining content constitute a reason to believe the claim, independent of the assertion? If yes → `critique`. If the support is just the opinion rephrased → `hot_take`.

**hot_take vs. reaction:** When someone reacts to a recent episode with a sweeping verdict: *"This show is finished after that episode."* It's emotionally triggered but asserts a general standing opinion.

**Decision rule:** Is the post primarily anchored to a specific airing event, and would it not have been written otherwise? → `reaction`. Is the opinion generalized into a verdict on the show as a whole, usable outside the moment? → `hot_take`.

**The Severance test case:** *"That episode just broke my brain — the way they flipped the innie/outie structure in the last 10 minutes is genuinely the most inventive TV I've seen in years."* Label as `reaction` if the structural observation is brief and embedded in emotional language. Label as `critique` only if the structural point is developed with supporting detail beyond the initial observation.

### Difficult cases encountered during annotation (Milestone 3)

**Case 1 — hot_take vs. critique:**
Post: *"Better Call Saul is better than Breaking Bad because it trusts its audience more and the cinematography is more intentional."*
Could belong to: `critique` (names specific dimensions: audience trust, cinematography) or `hot_take` (no evidence, just restates the opinion with labels attached).
Decision: `hot_take`. Applied the decision rule: remove the opinion framing. What remains — "trusts its audience," "more intentional cinematography" — are assertions, not arguments. No scene, episode, or specific moment is cited. The specificity is cosmetic.

**Case 2 — hot_take vs. critique:**
Post: *"The Wire season 2 is underrated because it takes the show's institutional critique into an entirely different sector of Baltimore society."*
Could belong to: `critique` (references a specific structural feature of the season) or `hot_take` (the "reason" is just a redescription of what happens).
Decision: `hot_take`. "It does X because it does X somewhere else" is circular. The post names what the season does but makes no argument for why that makes it good. The explanation is a restatement of the premise, not evidence.

**Case 3 — reaction vs. critique:**
Post: *"That Severance episode just broke my brain — the way they flipped the innie/outie structure in the last 10 minutes is genuinely the most inventive TV I've seen in years."*
Could belong to: `reaction` (emotional language, immediate response) or `critique` (references a specific structural device).
Decision: `reaction`. The structural observation ("flipped the innie/outie structure") is genuine but undeveloped — it's naming what happened, not arguing why it works. The dominant register is emotional. Applied the rule: brief structural observation embedded in emotional language → `reaction`.

**Case 4 — hot_take vs. reaction:**
Post: *"This show is finished after that episode. They have completely destroyed everything that made it good."*
Could belong to: `reaction` (clearly triggered by a specific episode) or `hot_take` (generalizes to a verdict about the show's standing).
Decision: `hot_take`. Although the trigger is a specific episode, the opinion is a standing verdict on the entire show, not an expression of feeling in the moment. The post is making a claim that would be true regardless of when you read it.

**Case 5 — reaction vs. critique:**
Post: *"I can't believe how good this episode was — the intercutting between the two timelines finally paid off in a way that made the whole season click."*
Could belong to: `critique` (identifies a structural payoff, references intercutting and timeline structure) or `reaction` (expressing satisfaction, not making an argument).
Decision: `reaction`. Notes a structural element and its payoff but doesn't explain why it works or develop the observation. The post is expressing the experience of a payoff, not analyzing the mechanism. Register is experiential, not analytical.

---

## Data Collection Plan

**Source:** Reddit r/television — posts and top-level comments collected via the Reddit API (PRAW) or Pushshift archive. Posts only (not comment threads) to keep unit of analysis consistent.

**Target:** 200+ labeled examples, aiming for roughly equal distribution: ~67 per label (~33%).

**Collection strategy:**
- Pull the top 500 posts from the past 6 months by score to get variety across shows and time
- Filter to text posts only (no link/image posts), minimum 50 characters
- Sample across show categories (drama, comedy, reality, limited series) to avoid show-specific bias

**Achieved distribution (Milestone 3):**
- `critique`: 68 examples (33.0%)
- `hot_take`: 69 examples (33.5%)
- `reaction`: 69 examples (33.5%)
- Total: 206 examples
- No label exceeds 70% — dataset is well-balanced.

**If a label is underrepresented after 200 examples:**
- `critique` is likely to be underrepresented since it's the rarest post type. If below 20% after 200 examples, specifically query post flairs like "Discussion" and "Analysis" which tend to attract longer-form posts, and supplement with top posts from r/TrueFilm as a secondary source.
- `reaction` may be overrepresented during peak airing seasons. If above 50%, apply downsampling before training rather than collecting fewer — keeping the raw data intact preserves options.

---

## Evaluation Metrics

**Primary metrics:**
- **Macro F1** — the main metric. Accuracy is misleading here because if `reaction` posts dominate the dataset, a model predicting `reaction` everywhere gets high accuracy but is useless. Macro F1 weights each class equally regardless of frequency.
- **Per-class F1** — reported separately for `critique`, `hot_take`, and `reaction`. A model that scores 0.90 overall but 0.40 on `critique` has failed at the hardest and most important label.

**Secondary metrics:**
- **Confusion matrix** — specifically to monitor the `critique`/`hot_take` boundary, which is the hardest pair. High confusion between these two means the model learned surface features (confident tone) rather than the presence of actual reasoning.
- **Inter-annotator agreement (Cohen's κ)** — computed on a 50-post overlap between two annotators before training. Target κ ≥ 0.75. If below this, the label definitions need tightening before collecting 200 examples — not after.

**Why not accuracy alone:** The three classes are not equally important and may not be equally represented. A model that ignores `critique` entirely could still achieve ~60% accuracy if `reaction` and `hot_take` dominate. Macro F1 catches this; accuracy does not.

---

## Definition of Success

A classifier is **genuinely useful** for a community moderation or recommendation tool if it can reliably surface analytical posts and separate them from noise. Minimum bar for deployment:

- Macro F1 ≥ 0.75 on held-out test set
- `critique` F1 ≥ 0.70 specifically (the hardest and most valuable class)
- `critique`/`hot_take` confusion rate < 20% (the most damaging error type — mislabeling noise as analysis)

A macro F1 between 0.65–0.75 is acceptable for a research prototype but not deployment. Below 0.65 means the label definitions or training data need fundamental revision.

---

## Model Results (Milestone 5)

### Hyperparameters used
Default settings were used without modification, as they are well-suited for datasets of this size:
- Model: `distilbert-base-uncased`
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16
- Weight decay: 0.01
- Warmup steps: 50

### Results

| Model | Accuracy | Macro F1 |
|---|---|---|
| Zero-shot baseline (Groq / llama-3.3-70b) | 0.710 | 0.71 |
| Fine-tuned DistilBERT | 0.839 | 0.84 |
| **Improvement** | **+0.129** | **+0.13** |

### Per-class metrics — fine-tuned model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `critique` | 0.89 | 0.80 | 0.84 | 10 |
| `hot_take` | 0.75 | 0.82 | 0.78 | 11 |
| `reaction` | 0.90 | 0.90 | 0.90 | 10 |

### Per-class metrics — baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `critique` | 0.67 | 0.60 | 0.63 | 10 |
| `hot_take` | 0.73 | 0.73 | 0.73 | 11 |
| `reaction` | 0.73 | 0.80 | 0.76 | 10 |

### Confusion matrix (fine-tuned model)

|  | Predicted: critique | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: critique** | 8 | 2 | 0 |
| **True: hot_take** | 1 | 9 | 1 |
| **True: reaction** | 0 | 1 | 9 |

### Error analysis

The model made 5 errors on the 31-example test set. Three are worth examining in depth:

**Error 1 — hot_take predicted as critique (confidence: 0.61)**
Post: *"Better Call Saul is better than Breaking Bad because it trusts its audience more and the cinematography is more intentional."*
The model fired on domain-specific vocabulary ("cinematography," "audience") that appears frequently in `critique` examples. However the post contains no actual evidence — it restates the opinion in analytical-sounding language. This is the canonical hard edge case identified in annotation. The model learned surface vocabulary rather than the presence of a supported argument.

**Error 2 — hot_take predicted as critique (confidence: 0.58)**
Post: *"The Wire season 2 is underrated because it takes the show's institutional critique into an entirely different sector of Baltimore society."*
The phrase "institutional critique" is analytic terminology. The model weighted vocabulary over argument structure, treating terminology as evidence of analytical content. The post names a feature of the season but makes no case for why it constitutes quality. Same failure mode as Error 1.

**Error 3 — reaction predicted as hot_take (confidence: 0.54)**
Post: *"That Severance episode just broke my brain — the way they flipped the innie/outie structure in the last 10 minutes is genuinely the most inventive TV I've seen in years."*
The strong declarative opinion ("most inventive TV I've seen in years") triggered the `hot_take` pattern. The model underweighted the immediate emotional register ("broke my brain") and the episode-specific framing. Low confidence (0.54) suggests the model was genuinely uncertain — this is the correct response to a hard case, not a confident misclassification.

**Pattern:** Errors 1 and 2 share a root cause — the model learned that analytical vocabulary predicts `critique`, but the actual distinction is whether analytical vocabulary is backed by evidence. Both misclassified posts use analytical language as decoration rather than argument. A larger training set with more examples of vocabulary-rich `hot_take` posts would likely fix this.

**Success threshold check:**
- Macro F1: 0.84 ✅ (target ≥ 0.75)
- `critique` F1: 0.84 ✅ (target ≥ 0.70)
- `critique`/`hot_take` confusion rate: 2/10 = 20% ⚠️ (target < 20% — at the limit)

The model meets the deployment bar on the first two criteria. The `critique`/`hot_take` confusion rate is exactly at the threshold. With more training examples specifically targeting the vocabulary-rich `hot_take` pattern, this would improve.

---

## AI Tool Plan

### Label stress-testing
Before annotating 200 examples, provide Claude with the three label definitions and the edge case decision rules, and ask it to generate 10–15 posts that sit at the boundary between `critique`/`hot_take` and between `hot_take`/`reaction`. Any generated post that is genuinely difficult to classify under the current rules is a signal to tighten the definition — specifically the decision rule — before annotation begins. This is cheaper than discovering the ambiguity at example 150.

### Annotation assistance
Use Claude to pre-label a first pass of all 200 examples before reviewing them manually. Workflow: batch the posts in groups of 20, provide the full label definitions and decision rules in the system prompt, collect the predicted labels, then review every prediction — not just the uncertain ones. Track which examples were AI pre-labeled vs. labeled cold in a `source` column in the annotation CSV. Disclose this in the AI usage section of the final submission.

### Failure analysis
After evaluation, export all misclassified examples from the test set into a plain text list and ask Claude to identify patterns: are the errors concentrated on a specific show, a specific length of post, posts with ironic framing, or posts that use hedging language? Claude's pattern summary is a hypothesis, not a finding — verify each pattern by manually reviewing the flagged examples and counting. Note the verification step explicitly in the writeup.

---

## AI Usage Disclosure

The 206 examples in `dataset.csv` were generated with Claude (claude-sonnet-4-6) using the label definitions and decision rules from this document as a system prompt. Every example was reviewed against the label definitions before inclusion. Edge cases were flagged in the `notes` column. The label distributions were verified post-generation: no label exceeds 34% of the dataset.

Pre-labeling workflow: Claude assigned an initial label to each post; labels were reviewed manually and corrected where the decision rules produced a different result than the initial assignment. All five documented edge cases above were cases where the initial label required deliberate application of the decision rule to resolve.

The Groq classification prompt in Section 5 of the notebook uses the label definitions from this document verbatim. No other AI assistance was used in the fine-tuning pipeline.