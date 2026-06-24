# r/television Discourse Classifier

## Project Overview
This project is an NLP text classification model designed to categorize discourse on the r/television subreddit. Discourse on TV shows varies wildly, from rigorous scene-by-scene narrative analysis to impulsive, unsubstantiated opinions, and visceral emotional reactions to recent episodes. This classifier reliably surfaces analytical posts and separates them from noise by classifying text into three distinct labels:

* **`critique`**: A structured argument about a show's writing, direction, acting, or narrative structure, supported by specific references or evidence.
* **`hot_take`**: A bold or contrarian opinion without provided reasoning or evidence — the claim itself is the entire content.
* **`reaction`**: An immediate emotional response to a specific episode/scene with little or no analytical content.

---

## Evaluation Report

### Overall Metrics
The fine-tuned model (DistilBERT) showed significant improvement over the zero-shot baseline (Groq / llama-3.3-70b), comfortably clearing the deployment threshold of a 0.75 Macro F1 score. 

| Model | Accuracy | Macro F1 |
|---|---|---|
| Zero-shot baseline (Groq / llama-3.3-70b) | 0.710 | 0.71 |
| Fine-tuned DistilBERT | **0.839** | **0.84** |
| **Improvement** | **+0.129** | **+0.13** |

### Per-Class Metrics

**Fine-Tuned Model (DistilBERT)**
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `critique` | 0.89 | 0.80 | 0.84 | 10 |
| `hot_take` | 0.75 | 0.82 | 0.78 | 11 |
| `reaction` | 0.90 | 0.90 | 0.90 | 10 |

**Baseline Model (Zero-Shot LLM)**
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `critique` | 0.67 | 0.60 | 0.63 | 10 |
| `hot_take` | 0.73 | 0.73 | 0.73 | 11 |
| `reaction` | 0.73 | 0.80 | 0.76 | 10 |

### Confusion Matrix (Fine-Tuned Model)

| True \ Predicted | Predicted: critique | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: critique** | 8 | 2 | 0 |
| **True: hot_take** | 1 | 9 | 1 |
| **True: reaction** | 0 | 1 | 9 |

---

### Error Analysis: Where the Model Failed

The model made 5 errors on the 31-example test set. By clustering the misclassified examples (with AI assistance to surface linguistic patterns), specific failure modes became clear:

**Error 1**
* **Text:** *"Better Call Saul is better than Breaking Bad because it trusts its audience more and the cinematography is more intentional."*
* **True Label:** `hot_take` | **Predicted:** `critique` (Conf: 0.61)
* **Analysis:** * **Which labels are confused?** `hot_take` misclassified as `critique`.
    * **Why is the boundary hard?** The post adopts an analytical tone and uses domain-specific vocabulary ("cinematography," "audience trust") but provides zero structural evidence to support its claims. It masquerades as analysis.
    * **Is this a labeling or data problem?** A data problem. The dataset likely doesn't have enough examples of "fake deep" `hot_takes` that use big words without actual argument structures.
    * **How to fix:** Augment the training data with more `hot_take` examples that utilize analytical terminology to force the model to look for structural evidence rather than just keywords.

**Error 2**
* **Text:** *"The Wire season 2 is underrated because it takes the show's institutional critique into an entirely different sector of Baltimore society."*
* **True Label:** `hot_take` | **Predicted:** `critique` (Conf: 0.58)
* **Analysis:** * **Which labels are confused?** `hot_take` misclassified as `critique`.
    * **Why is the boundary hard?** Similar to Error 1, it uses academic/critical vocabulary ("institutional critique", "sector of society"). The model sees these words and assumes analysis, completely missing that the statement is circular and just redescribes the premise of the season without arguing *why* it makes it good.
    * **Is this a labeling or data problem?** Data distribution problem. The model has overfit to vocabulary as a proxy for reasoning.
    * **How to fix:** Introduce explicitly mapped contrasting pairs in the training data (e.g., a `critique` using the phrase "institutional critique" with evidence, and a `hot_take` using the exact same phrase without evidence).

**Error 3**
* **Text:** *"That Severance episode just broke my brain — the way they flipped the innie/outie structure in the last 10 minutes is genuinely the most inventive TV I've seen in years."*
* **True Label:** `reaction` | **Predicted:** `hot_take` (Conf: 0.54)
* **Analysis:** * **Which labels are confused?** `reaction` misclassified as `hot_take`.
    * **Why is the boundary hard?** This post has dual signals. "Broke my brain" is an immediate emotional reaction to a recent viewing, but "most inventive TV I've seen in years" is a sweeping, generalized verdict. 
    * **Is this a labeling or data problem?** This is a boundary ambiguity problem. The prompt definitions prioritize the emotional anchor, but the model prioritized the declarative claim. The low confidence (0.54) indicates the model itself was torn.
    * **How to fix:** Tighten the definition and add more examples of `reaction` posts that contain hyperbolic verdicts to teach the model that an emotional trigger overrides a standing opinion in this context.

---

## Sample Classifications

Below are sample classifications from the fine-tuned model demonstrating its capabilities across the labels:

| Text Post | Predicted Label | Confidence |
| :--- | :--- | :--- |
| "Succession season 4 deliberately slows its pacing after episode 3. The scenes with the siblings run noticeably longer, mirroring the physical discomfort of the power vacuum." | `critique` | 0.94 |
| "Game of Thrones was never actually good. Seasons 1-4 are just as hollow as 7-8, people just didn't notice yet." | `hot_take` | 0.89 |
| "I am literally hyperventilating after that White Lotus finale. NO SPOILERS BUT WOW. I am not okay." | `reaction` | 0.97 |
| *"The Leftovers is a masterpiece because of its thematic resonance."* | `critique` | 0.62 |

**Prediction Reasoning (Correct Example):** In the first example, the model correctly predicts `critique` with high confidence (0.94) because it successfully identifies both the specific narrative claim ("slows its pacing after episode 3") and the supporting textual evidence provided by the user ("scenes with the siblings run noticeably longer"), distinguishing actual analysis from a mere opinion.

---

## Model Reflection: Captured vs. Intended

**Intended:** The model was intended to capture the *presence of a structural argument*—to distinguish between a claim supported by specific evidence (`critique`) and a claim standing on its own (`hot_take`). 
**Captured:** While the model performs well generally, the error analysis reveals it partially overfit to **surface vocabulary and tone**. It learned that words like "cinematography," "pacing," and "institutional" are highly correlated with `critique`. Consequently, when a `hot_take` utilizes "smart-sounding" vocabulary to mask an unsubstantiated opinion, the model's decision boundary fails, proving it is currently better at identifying analytical *diction* than analytical *reasoning*. 

---

## Spec Reflection

* **How the Spec Helped:** Writing out explicit "Decision Rules" in `planning.md` for edge cases (e.g., removing opinion framing to see if evidence remains) was invaluable. It made the annotation process significantly faster and ensured high consistency when the AI pre-labeled the data.
* **How Implementation Diverged:** The original Data Collection Plan stated we would pull raw posts from Reddit using the PRAW API or Pushshift archive. However, to ensure perfectly balanced classes and highly specific edge cases, I pivoted to generating the 206 examples synthetically using Claude (guided by the strict definitions and decision rules). This divergence allowed for a cleaner dataset but means the model is trained on simulated Reddit discourse rather than organic web scrapes.

---

## AI Usage Disclosure

AI tools were leveraged strategically throughout this pipeline to aid in generation and analysis:

1.  **Dataset Generation & Pre-Labeling:** I used Claude (claude-sonnet-4-6) to generate the entire 200+ example dataset. I provided Claude with the label definitions and edge-case decision rules from `planning.md` as the system prompt. It assigned an initial label to each post. *Human override:* I manually reviewed every post and its assigned label, correcting the label on several edge-case posts where Claude failed to properly apply the "remove the opinion framing" decision rule for `critique` vs `hot_take`.
2.  **Failure Analysis Pattern Matching:** After running the test set, I exported the 5 misclassified examples and fed them back to Claude, prompting it to "identify common themes or linguistic patterns linking the errors." Claude surfaced the insight that errors 1 and 2 both contained analytical vocabulary (e.g., "cinematography") without structural evidence. *Human override:* I verified this hypothesis by manually re-reading the text and constructing the final analysis in this README based on that verified pattern.