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
| Maximum epochs | 10 |
| Best-checkpoint selection | Highest validation accuracy, loaded automatically at the end of training |
| Learning rate | `2e-5` |
| Train batch size | 16 |
| Evaluation batch size | 32 |
| Weight decay | `0.01` |
| Warmup steps | 5 |

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
| Groq zero-shot baseline | 0.900 | 0.92 |
| Fine-tuned DistilBERT, trained up to 10 epochs | 0.633 | 0.52 |

Fine-tuning produced an accuracy regression of `0.2667` compared with the zero-shot Groq baseline. Macro F1 also decreased from `0.92` to `0.52`.

The fine-tuned model was trained for up to 10 epochs. Because `load_best_model_at_end=True` and validation accuracy was used as the selection metric, final test evaluation used the checkpoint with the highest validation accuracy rather than necessarily the final epoch checkpoint.

### Per-Class Metrics: Groq Zero-Shot Baseline

| Label | Precision | Recall | F1 | Support |
| :--- | ---: | ---: | ---: | ---: |
| Help-Seeking | 0.94 | 0.88 | 0.91 | 17 |
| Reasoned Analysis | 1.00 | 1.00 | 1.00 | 4 |
| Reaction | 0.80 | 0.89 | 0.84 | 9 |
| **Accuracy** |  |  | **0.900** | **30** |
| **Macro Average** | 0.91 | 0.92 | 0.92 | 30 |

The baseline performed strongly on this held-out test set. However, Reasoned Analysis has only four test examples, so its perfect score should be interpreted cautiously.

### Per-Class Metrics: Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
| :--- | ---: | ---: | ---: | ---: |
| Help-Seeking | 0.62 | 0.88 | 0.73 | 17 |
| Reasoned Analysis | 1.00 | 0.25 | 0.40 | 4 |
| Reaction | 0.60 | 0.33 | 0.43 | 9 |
| **Accuracy** |  |  | **0.633** | **30** |
| **Macro Average** | 0.74 | 0.49 | 0.52 | 30 |

The fine-tuned model correctly recognized all three labels at least once, but it still over-predicted Help-Seeking and substantially under-recalled Reasoned Analysis and Reaction.

### Fine-Tuned Confusion Matrix

Rows are true labels. Columns are predicted labels.

| True label \ Predicted label | Help-Seeking | Reasoned Analysis | Reaction |
| :--- | ---: | ---: | ---: |
| Help-Seeking | 15 | 0 | 2 |
| Reasoned Analysis | 3 | 1 | 0 |
| Reaction | 6 | 0 | 3 |

The image version is available in [`confusion_matrix.png`](confusion_matrix.png).

The model correctly classified 15 Help-Seeking posts, 1 Reasoned Analysis post, and 3 Reaction posts, for 19 correct predictions out of 30. Its main error pattern was over-predicting Help-Seeking: 3 Reasoned Analysis posts and 6 Reaction posts were classified as Help-Seeking.

### Error Analysis

#### Error 1: Nostalgic post misclassified as Help-Seeking

**Text:** “Title: Does anyone remember Xirena”  
**Body:** Every few months i just randomly remembers xirena and check on her channel to see if she cameback or if there is any update..
**True label:** Reaction  
**Prediction:** Help-Seeking  
**Confidence:** 0.58

The author is expressing nostalgia and checking whether a former creator has returned or posted an update. Although the title is phrased as a question and the post mentions looking for updates, it does not directly ask the community for advice, diagnosis, recommendation, or a concrete answer.

Under the taxonomy, the core intent is a shared recollection or conversational reaction rather than Help-Seeking. The model likely treated the question-like title and the mention of an “update” as signals that the author wanted information from other users. This shows that the model can over-predict Help-Seeking when a Reaction post contains uncertainty or a question-shaped opening.

#### Error 2: Supported balance claim misclassified as Help-Seeking

**Text:** “Shotguns were handled poorly”  
**Body:** Shotguns definitely needed a nerf and I agree with that. But nerfing the way they’re used was not the solution. A better nerf would’ve been to just increase ...
**True label:** Reasoned Analysis  
**Prediction:** Help-Seeking  
**Confidence:** 0.47

This post makes a detailed balance argument: shotguns needed a nerf, but the chosen nerf was ineffective, and price, ammunition, range, or specific agent interactions would have been better targets. The author supports the claim with gameplay reasoning, comparisons, and proposed alternatives.

The model appears to interpret repeated statements about what should change as a request for guidance. It missed the claim-plus-reason structure that defines Reasoned Analysis, including mechanisms, counterexamples, and alternative recommendations.

#### Error 3: Matchmaking question misclassified as Reaction

**Text:** “Unranked matches are confusing ???”  
**Body:** So genuinely why are unranked people who were gold/plat last act or update being put into MY bronze lobbies its so hard and annoying seeing one guy dro...
**True label:** Help-Seeking  
**Prediction:** Reaction  
**Confidence:** 0.51

Although the post uses a frustrated tone, the author is explicitly asking why players who were previously Gold or Platinum are being placed in Bronze unranked lobbies. The core intent is to obtain an explanation of how the matchmaking system works.

The model likely focused on emotional language such as “hard and annoying” and treated the post as a complaint. This error shows that the model can miss Help-Seeking intent when a direct question is embedded inside a frustrated personal experience.

### Sample Classifications

#### 1. Correct Help-Seeking Prediction

**Post:** “How to improve your aim?”  
“I want to improve my aim overall… What is a good routine and sensitivity to start with?”

**Predicted Label:** Help-Seeking  
**Confidence:** 72.2%  
**Correct:** Yes

#### 2. Correct Reasoned Analysis Prediction

**Post:** “Valorant ranked system is do bad”  
“The ranking system is flawed because of MMR and RR disconnects, inconsistent matchmaking, smurfing, solo queue disadvantages, and weak punishment systems.”

**Predicted Label:** Reasoned Analysis  
**Confidence:** 44.5%  
**Correct:** Yes

#### 3. Correct Reaction Prediction

**Post:** “1v3 clutch after a long time”  
“I was going through my old archive from a few acts ago and saw this. Just wanted to share those small moments that got us hooked on Valorant.”

**Predicted Label:** Reaction  
**Confidence:** 52.2%  
**Correct:** Yes

#### 4. Incorrect Prediction

**Post:** “Does anyone remember Xirena”  
“Every few months I randomly remember Xirena and check her channel to see if she came back or posted an update.”

**Predicted Label:** Help-Seeking  
**Confidence:** 58.0%  
**True Label:** Reaction  
**Correct:** No

## Reflection: Intended Labels vs. Learned Behavior

I intended the classifier to distinguish three discourse intents: posts seeking guidance, posts offering supported analysis, and posts mainly expressing or sharing an experience.

The final fine-tuned model captured part of that intended structure, because it correctly predicted at least one Reasoned Analysis post and three Reaction posts. However, its learned behavior remained strongly biased toward Help-Seeking. It correctly classified 15 of 17 Help-Seeking posts, but it misclassified most Reasoned Analysis and Reaction posts as Help-Seeking.

The main gap is that the taxonomy depends on pragmatic intent and support structure, not only keywords or topic. A post can contain question-like wording without seeking help, and a post can discuss what should change without asking for advice. The model did not reliably learn these distinctions from the available training data, especially for the underrepresented Reasoned Analysis class.

For a future version, I would collect more varied Reasoned Analysis examples, especially short balance arguments, mechanic explanations, and supported critiques. I would also add more examples contrasting discussion questions, nostalgic posts, and gameplay showcases with genuine Help-Seeking posts.

## Spec Reflection

The project specification helped by requiring a written taxonomy and edge-case rules before model training. Writing the distinction between Help-Seeking and general discussion prompts made it possible to identify a concrete model failure later instead of only saying that the model was inaccurate.

My implementation diverged from the starter default by increasing the maximum epoch count from 3 to 10. I first observed that the shorter run was heavily biased toward the majority Help-Seeking label. I kept the learning rate and batch size unchanged, reduced warmup steps from 50 to 5 to better fit the short training run, and used a fixed random seed for reproducibility.

Rather than assuming the final epoch was best, I used `load_best_model_at_end=True` with validation accuracy as the selection metric. The final test evaluation therefore used the checkpoint with the highest validation accuracy from the run trained for up to 10 epochs.

## AI Usage Transparency

### 1. Taxonomy refinement and difficult annotation cases

I used ChatGPT to review the boundaries between Help-Seeking, Reasoned Analysis, and Reaction. It helped surface a problem with treating all question-form posts as Help-Seeking. I revised the taxonomy so that general polls, preference prompts, and invitations for shared experiences are Reaction unless they seek guidance for a concrete personal problem.

ChatGPT also assisted with discussion of selected ambiguous annotation cases. I reviewed the final retained examples against the written taxonomy rather than treating an AI suggestion as an automatic final label.

### 2. Failure-pattern analysis

After evaluation, I provided the final fine-tuned model’s 11 wrong predictions to ChatGPT and asked it to identify possible recurring patterns. It identified a tendency to confuse discussion-style questions and supported analysis with Help-Seeking.

I verified that conclusion using the confusion matrix and direct review of all 11 wrong predictions. The final model correctly predicted one Reasoned Analysis post and three Reaction posts, but it still over-predicted Help-Seeking: three Reasoned Analysis posts and six Reaction posts were classified as Help-Seeking. It also misclassified two Help-Seeking posts as Reaction when frustrated wording dominated the post.

## Repository Contents
```
ai201-project3-takemeter/
├── README.md
├── planning.md
├── datas/
│    └── dataset_200.csv
├── confusion_matrix.png
└── evaluation_results.json
```

## Demo Video
[Watch my demo](https://drive.google.com/file/d/1QlmTSrNwV8kL0aUpltS0hKj1zEV4sakd/view?usp=sharing)