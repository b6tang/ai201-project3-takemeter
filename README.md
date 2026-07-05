# TakeMeter: Classifying r/VALORANT Posts

## Project Overview

TakeMeter is a text-classification project for public r/VALORANT posts. The goal is to classify each post into one of three discourse-intent labels:

- Help-Seeking
- Reasoned Analysis
- Reaction

I compared a zero-shot Groq baseline with a fine-tuned `distilbert-base-uncased` classifier trained on a labeled dataset of 200 public r/VALORANT posts.

## Community and Dataset

I chose r/VALORANT because it contains varied public discussion about gameplay, agent and weapon choices, balance, ranked systems, clips, achievements, frustrations, and community discussion.

Each retained example combines the Reddit post title and body into one `text` field. The final CSV also contains the post `label`, Reddit post `id`, `postUrl`, and optional `notes` for difficult annotation cases.

I removed posts with missing, unreadable, or title-only text; posts requiring an image or video to understand; recruitment, promotion, or other non-discourse requests; and posts that did not fit a valid label.

### Final Dataset Distribution

| Label | Count | Percentage |
| :--- | ---: | ---: |
| Help-Seeking | 109 | 54.5% |
| Reasoned Analysis | 28 | 14.0% |
| Reaction | 63 | 31.5% |
| **Total** | **200** | **100.0%** |

No label exceeds 70% of the dataset.

### Data Split

I used a stratified 70/15/15 split.

| Split | Examples |
| :--- | ---: |
| Train | 140 |
| Validation | 30 |
| Test | 30 |

The held-out test set contained 17 Help-Seeking posts, 4 Reasoned Analysis posts, and 9 Reaction posts.

## Label Taxonomy

| Label | Definition |
| :--- | :--- |
| **Help-Seeking** | The author seeks an answer, diagnosis, recommendation, feedback, or confirmation that would resolve their own specific Valorant question, problem, decision, uncertainty, or situation. General polls, preference prompts, and invitations for others to share experiences are not Help-Seeking. |
| **Reasoned Analysis** | The author explains, evaluates, or argues a point about Valorant. The main claim is supported by at least one concrete reason, game mechanic, comparison, example, or consequence. |
| **Reaction** | The author mainly shares a clip, Ace, rank, skin, experience, emotion, meme, poll, discussion prompt, or unsupported opinion. This includes rants whose reasons mainly justify the author’s feelings rather than supporting a broader game-related claim. |

### Label Examples

| Label | Example | Why |
| :--- | :--- | :--- |
| Help-Seeking | “How to improve aim” | The author seeks a concrete practice routine. |
| Help-Seeking | “Pistol round on Chamber” | The author seeks advice about a specific weapon-buying decision. |
| Reasoned Analysis | “Shotguns were handled poorly” | The post makes a balance claim and gives a concrete alternative nerf. |
| Reasoned Analysis | “Increasing one ult orb and thinking it’s a nerf” | The post argues that the change has limited impact because the ultimate can still be earned quickly. |
| Reaction | “Not my first ace, but the one I’m really proud of” | The main purpose is sharing a personal gameplay achievement. |
| Reaction | “Is there a player card you’re chasing after?” | The post invites general discussion instead of seeking personal guidance. |


## Data Collection and Annotation Process

I started with a local collection of 400 publicly available r/VALORANT posts. Each retained example combines the post title and body into one `text` field.

I excluded posts with missing, unreadable, or title-only text; posts whose meaning required an image or video; recruitment, promotion, or other non-discourse requests; and posts that did not fit one of the three labels.

Each retained post was labeled using the taxonomy and decision rule above. Difficult or borderline cases were recorded in the `notes` field and reviewed before the final 200-example dataset was created.

## Hard Annotation Cases

### 1. Question-form discussion versus Help-Seeking

**Post:** “Is there a player card you’re chasing after?”

**Final label:** Reaction

This was difficult because the title is phrased as a question. However, the author was inviting people to share their own preferences rather than asking for help resolving a personal problem. Under the taxonomy, this is a discussion prompt and therefore Reaction.

### 2. Short balance opinion versus Reasoned Analysis

**Post:** “Increasing one ult orb and thinking it’s a nerf”

**Final label:** Reasoned Analysis

This post is short and informal, which makes it resemble an unsupported opinion. I labeled it Reasoned Analysis because it supports its balance claim with a specific gameplay consequence: the ultimate can still be obtained in roughly two rounds.

### 3. Frustrated rank complaint versus Reaction

**Post:** “What’s wrong with rank-reset?”

**Final label:** Reaction

The post includes a question and a complaint about rank reset. I labeled it Reaction because its main purpose is expressing frustration about the author’s personal experience, rather than seeking a concrete diagnosis or presenting a supported broader argument about the ranking system.


## Modeling Approach

I fine-tuned `distilbert-base-uncased` in Google Colab using a T4 GPU.

Final training settings:

| Setting | Value |
| :--- | :--- |
| Base model | `distilbert-base-uncased` |
| Training platform | Google Colab T4 GPU |
| Epochs | 6 |
| Learning rate | `2e-5` |
| Train batch size | 16 |
| Evaluation batch size | 32 |
| Weight decay | `0.01` |
| Warmup steps | 50 |

## Zero-Shot Baseline
The baseline used Groq's `llama-3.3-70b-versatile` model with no task-specific training. I gave the model the label definitions, representative examples, and an instruction to output only one valid label.

The baseline was run once on each of the same 30 held-out test examples used for fine-tuned evaluation. All 30 responses were parseable.

### Baseline Prompt
```
SYSTEM_PROMPT = """
You are classifying public posts from r/VALORANT.
Assign each post to exactly one label.

Help-Seeking: The author wants an answer, diagnosis, recommendation, feedback, or confirmation about their own specific Valorant question, problem, decision, or situation.
Example: "Should I main Omen or Clove?"

Reasoned Analysis: The author explains, evaluates, or argues a broader point about Valorant, and supports the main claim with at least one concrete reason, game mechanic, comparison, example, or consequence.
Example: "One extra ult orb is not really a nerf because you can still get your ultimate every two rounds."

Reaction: The author mainly shares an emotion, experience, clip, achievement, meme, poll, personal opinion, or unsupported complaint; reasons that only justify personal frustration do not make a post Reasoned Analysis.
Example: "Finally hit Ascendant after grinding for two years!"

Respond with ONLY the label name.
Do not explain your reasoning.

Valid labels:

Help-Seeking
Reasoned Analysis
Reaction
"""
```

## Evaluation Results

Both models were evaluated on the same 30-example held-out test set.

### Overall Comparison

| Model | Accuracy | Macro F1 |
| :--- | ---: | ---: |
| Groq zero-shot baseline | 0.867 | 0.89 |
| Fine-tuned DistilBERT, 6 epochs | 0.567 | 0.36 |

Fine-tuning produced an accuracy regression of `0.300` compared with the zero-shot Groq baseline.

### Per-Class Metrics: Groq Zero-Shot Baseline

| Label | Precision | Recall | F1 | Support |
| :--- | ---: | ---: | ---: | ---: |
| Help-Seeking | 0.88 | 0.88 | 0.88 | 17 |
| Reasoned Analysis | 1.00 | 1.00 | 1.00 | 4 |
| Reaction | 0.78 | 0.78 | 0.78 | 9 |
| **Accuracy** |  |  | **0.867** | **30** |
| **Macro Average** | 0.89 | 0.89 | 0.89 | 30 |

The baseline performed strongly across this test set. However, Reasoned Analysis has only four test examples, so its perfect F1 score should not be treated as conclusive.

### Per-Class Metrics: Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
| :--- | ---: | ---: | ---: | ---: |
| Help-Seeking | 0.61 | 0.82 | 0.70 | 17 |
| Reasoned Analysis | 0.00 | 0.00 | 0.00 | 4 |
| Reaction | 0.43 | 0.33 | 0.38 | 9 |
| **Accuracy** |  |  | **0.567** | **30** |
| **Macro Average** | 0.35 | 0.39 | 0.36 | 30 |

### Fine-Tuned Confusion Matrix

Rows are true labels. Columns are predictions.

| True label \ Predicted label | Help-Seeking | Reasoned Analysis | Reaction |
| :--- | ---: | ---: | ---: |
| Help-Seeking | 14 | 0 | 3 |
| Reasoned Analysis | 3 | 0 | 1 |
| Reaction | 6 | 0 | 3 |

The image version is available in [`confusion_matrix.png`](confusion_matrix.png).

### Error Analysis

#### Error 1: Discussion question misclassified as Help-Seeking

**Text:** “Is there a player card you’re chasing after?”  
**True label:** Reaction  
**Prediction:** Help-Seeking  
**Confidence:** 0.48

This is a general discussion prompt. The author is curious about what other players are collecting, not seeking guidance about a personal problem. The fine-tuned model appears to over-weight question wording and phrases such as “anyone else,” even though the written taxonomy explicitly excludes general discussion prompts from Help-Seeking.

#### Error 2: Supported balance claim misclassified as Help-Seeking

**Text:** “Shotguns were handled poorly”  
**True label:** Reasoned Analysis  
**Prediction:** Help-Seeking  
**Confidence:** 0.46

This post makes a clear balance argument: shotguns needed a nerf, but the specific nerf was wrong, and a different adjustment would have been better. The model failed to recognize the claim-plus-reason structure that defines Reasoned Analysis. This suggests that it did not learn to reliably detect support relationships such as mechanism, alternative, and gameplay consequence.

#### Error 3: Clear question misclassified as Reaction

**Text:** “How do youtubers like dacoit find the clips they use?”  
**True label:** Help-Seeking  
**Prediction:** Reaction  
**Confidence:** 0.48

This is a genuine request for information. Unlike the discussion-style questions above, the author wants a concrete explanation of how creators obtain clips. The model predicted Reaction, suggesting that after six epochs it began assigning some questions to Reaction without learning the distinction between a personal information request and a casual discussion prompt.

### Systematic Error Pattern

The fine-tuned model did not predict Reasoned Analysis for any of the 30 held-out test examples. Its confusion matrix shows that all four Reasoned Analysis posts were classified as either Help-Seeking or Reaction.

The 3-epoch run predicted every test post as Help-Seeking. Increasing training to 6 epochs changed the model’s behavior: it correctly identified three Reaction posts, and macro F1 increased from 0.24 to 0.36. However, overall accuracy remained 0.567 because three Help-Seeking posts were then classified as Reaction.

These results show that six epochs alone did not solve the main problem. The model still did not learn the Reasoned Analysis boundary on this test set. A likely next step would be to collect more varied Reasoned Analysis examples, especially short balance arguments, mechanic explanations, and supported critiques.

### Sample Classifications

The following per-example predictions were preserved from the final 6-epoch evaluation output.

| Post | Predicted Label | Confidence | Correct? | Notes |
| :--- | :--- | ---: | :--- | :--- |
| “Does anyone remember Xirena” | Help-Seeking | 0.50 | No | Nostalgic personal-sharing post incorrectly classified as Help-Seeking. |
| “Is there a player card you’re chasing after?” | Help-Seeking | 0.48 | No | General discussion prompt incorrectly classified as Help-Seeking. |
| “Shotguns were handled poorly” | Help-Seeking | 0.46 | No | Supported balance claim incorrectly classified as Help-Seeking. |

> **Documentation limitation:** The final confusion matrix confirms 17 correct predictions in the 30-example test set. However, after the final evaluation, Colab GPU access became unavailable and the saved checkpoint was no longer accessible. I could not recover a per-item correct prediction and confidence without rerunning the model, so I have not reconstructed or fabricated one from aggregate results.

## Reflection: Intended Labels vs. Learned Behavior

I intended the classifier to distinguish three discourse intents: posts seeking guidance, posts offering supported analysis, and posts mainly expressing or sharing an experience.

The fine-tuned model did not learn that intended structure. Its strongest learned behavior was a broad Help-Seeking prior. In the 3-epoch run, it predicted every test example as Help-Seeking. The 6-epoch run began predicting some Reaction posts, but it still never predicted Reasoned Analysis.

The main gap is that the taxonomy depends on pragmatic intent and support structure, not only keywords or topic. A post can contain a question without seeking help, and a short informal post can contain a real balance argument. The model did not reliably learn these distinctions from the available training data, especially for the underrepresented Reasoned Analysis class.

For a future version, I would collect more varied Reasoned Analysis examples, especially short balance arguments, mechanic explanations, and supported critiques. I would also add more examples contrasting discussion questions with genuine Help-Seeking posts.

## Spec Reflection

The project specification helped by requiring a written taxonomy and edge-case rules before model training. Writing the distinction between Help-Seeking and general discussion prompts made it possible to identify a concrete model failure later instead of only saying that the model was inaccurate.

My implementation diverged from the starter default by increasing the epoch count from 3 to 6. I made this change after the 3-epoch run collapsed to the majority Help-Seeking label and validation loss was still decreasing. I kept the learning rate and batch size unchanged so that the follow-up experiment changed only one training decision.

## AI Usage Transparency

### 1. Taxonomy refinement and difficult annotation cases

I used ChatGPT to review the boundaries between Help-Seeking, Reasoned Analysis, and Reaction. It helped surface a problem with treating all question-form posts as Help-Seeking. I revised the taxonomy so that general polls, preference prompts, and invitations for shared experiences are Reaction unless they seek guidance for a concrete personal problem.

ChatGPT also assisted with discussion of selected ambiguous annotation cases. I reviewed the final retained examples against the written taxonomy rather than treating an AI suggestion as an automatic final label.

### 2. Failure-pattern analysis

After evaluation, I provided the fine-tuned model’s wrong predictions to ChatGPT and asked it to identify possible recurring patterns. It identified a tendency to confuse discussion-style questions and supported analysis with Help-Seeking.

I verified that conclusion using the confusion matrix and direct review of all 13 wrong predictions. I narrowed the final claim to what the evidence supported: the model never predicted Reasoned Analysis and over-predicted Help-Seeking, while the 6-epoch run introduced some Reaction predictions without improving overall accuracy.

## Repository Contents
ai201-project3-takemeter/
├── README.md
├── planning.md
├── datas/
│    └── dataset_200.csv
├── confusion_matrix.png
└── evaluation_results.json

## Demo Video

[Add demo video link here]