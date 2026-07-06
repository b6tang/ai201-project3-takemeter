## Community: 

I chose r/VALORANT on reddit because it is an active public community with varied, text-based discussion. It is a good classification setting because posts vary substantially in intent: some users want specific help, some show their knowledge/skills about the game, and others mainly share reactions, clips, achievements, or frustrations. This variation makes the three labels meaningful rather than artificial.

## Labels: 

| Label     | Definition      | Primary Drive / Core Intent  |
| :-------- | :-------------- | :-------------------------- |
| **Help-Seeking**   | The author seeks guidance, explanations, recommendations, diagnoses, factual information, or confirmation about their own question, problem, decision, or situation.| **Seeking Guidance:** Driven by a question or problem that needs an answer, solution, or outside feedback.  |
| **Reasoned Analysis** | The author explains, evaluates, or argues a point about Valorant. The post's main claim is supported by at least one concrete reason, game mechanic, comparison, example, or consequence.  | **Knowledge / Insight Sharing:** Driven mainly by explaining or supporting a viewpoint about a game mechanic, balance, strategy, system, meta, or community issue. Emotion may appear, but it is not the main point. |
| **Reaction**          | The author mainly shares a clip, Ace, rank, skin, experience, emotion, meme, or opinion. This includes rants or complaints where reasons mainly justify the author's feelings instead of supporting a broader point about the game. | **Emotional / Experiential Expression:** Driven mainly by venting, bragging, celebrating, complaining, joking, or sharing a personal moment.|

### Label 0: Help-Seeking
Help-Seeking means that the author wants an answer, diagnosis, recommendation, feedback, or confirmation about a specific Valorant-related question, problem, decision, or situation.

- Example: “How to improve aim” asks for a specific Aim Lab and Valorant practice routine.
- Example: “Pistol round on Chamber” asks whether buying Ghost, Bandit, or Sheriff is a legitimate choice and why.

### Label 1: Reasoned Analysis
Reasoned Analysis means that the author explains, evaluates, or argues a broader point about Valorant and supports the main claim with at least one concrete mechanism, comparison, example, or consequence.

- Example: “Valorant's Retake mode is missing the tension CS retake servers have” argues that the mode feels less tense because its 3v3 format creates fewer angles, bodies, and decision points.
- Example: “Quick tip for omen players (or any flashing agent)” recommends waiting before swinging after a flash and explains that this disrupts the opponent’s timing.

### Label 2: Reaction
Reaction means that the author mainly shares an emotion, experience, achievement, highlight, meme, poll, or unsupported opinion. This includes complaints whose reasons mainly justify the author’s frustration instead of supporting a broader point about the game.

- Example: “I FINALLY got Radiant after 5 years” is a personal achievement post.
- Example: “My favourite clutch” shares a memorable gameplay moment rather than seeking advice or making a broader argument.

## Hard edge cases: 
The hardest cases are frustrated posts that contain both a question and a complaint, plus opinion posts that include one or two reasons.

For a frustrated question, I check the comments see whether the author is genuinely seeking an answer, diagnosis, recommendation, feedback, or confirmation about their own situation. If yes, I label it Help-Seeking even when the post contains emotional language or a detailed personal story.

For a complaint or opinion post, I apply the Emotion-Removal Test. I remove the author’s personal frustration, personal history, and agreement-seeking language, then ask whether a concrete claim about a mechanic, system, balance issue, strategy, meta, or community issue remains. I only use Reasoned Analysis when that remaining claim has a concrete reason, comparison, example, mechanism, or consequence.

If a case remains genuinely ambiguous after applying these rules, I label it Reaction.

#### Annotation Decision Rule

1. **Is the author asking for an answer, recommendation, diagnosis, feedback, or confirmation?**  
   Examples include asking how to improve, whether a setting or mechanic works, which option to choose, why something happened, or whether their own decision was correct.
   - Yes → **Help-Seeking**  
   - No → continue.

   General polls, preference prompts, requests for shared experiences, and questions meant mainly to start discussion do not count as Help-Seeking.

1. **Apply the Emotion-Removal Test.**  
   Mentally remove personal feelings, personal story details, and agreement-seeking language, such as:  
   - “I’m so tired of this.”  
   - “I want to uninstall.”  
   - “My teammates are awful.”  
   - “Does anyone else feel this way?”  

   Then ask:
   - Does it make a concrete claim about a mechanic, balance, strategy, system, meta, or community issue?
   - Is that claim supported by at least one reason, comparison, mechanism, example, or consequence? 
   - Yes → **Reasoned Analysis**  
   - No → **Reaction**

#### Tie Rule
Apply this rule only after a post has already been determined not to be Help-Seeking.

If a post remains genuinely ambiguous after applying these rules, label it **Reaction**.

This keeps Reasoned Analysis relatively strict while allowing concise posts that genuinely explain or support a gameplay point.

## Data collection plan: 
I began with a local collection of 400 public r/VALORANT posts. Each retained example contains the post title and body combined into a single `text` field, together with its `label`, Reddit post `id`, `postUrl`, and an optional `notes` field for difficult decisions.

I removed posts with missing, unreadable, or title-only text; posts whose meaning depended on an image or video; recruitment, promotion, or other non-discourse requests; and posts that did not fit a valid label.

After cleaning and re-annotating the dataset under the finalized taxonomy, the current dataset contains 200 labeled posts:
- Help-Seeking: 109 posts (54.5%)
- Reasoned Analysis: 28 posts (14.0%)
- Reaction: 63 posts (31.5%)

No label exceeds 70% of the final dataset. 

I used a stratified 70/15/15 split:

| Split | Examples |
| :--- | ---: |
| Train | 140 |
| Validation | 30 |
| Test | 30 |

The held-out test set contains 17 Help-Seeking posts, 4 Reasoned Analysis posts, and 9 Reaction posts. Because Reasoned Analysis has only four test examples, I will interpret that class’s metrics cautiously.


## Evaluation Metrics

I will use overall accuracy, but accuracy alone is not sufficient because Help-Seeking is the largest class. A model could achieve misleadingly high accuracy by over-predicting Help-Seeking.

I therefore also used:

- Per-class precision, recall, and F1 score, to show how well the model handles each label.
- Macro F1 score, because it weights all three labels equally rather than allowing the majority class to dominate the result.
- A confusion matrix, to identify which categories are most often confused, especially Help-Seeking versus Reaction and Reasoned Analysis versus Reaction.
- A comparison with the Groq zero-shot baseline on the exact same held-out test set, so that the difference between prompt-only classification and fine-tuning is meaningful.

The test split is small, especially for Reasoned Analysis, so I interpret its class-level metric carefully rather than treating one score as conclusive.

## Definition of Success

For this course project, I will consider the fine-tuned model successful if it meets all of the following conditions:

1. Overall test accuracy is at least 70%.
2. Macro F1 is at least 0.55.
3. No label has an F1 score below 0.40.
4. The fine-tuned model matches or exceeds the Groq baseline’s macro F1 score on the same test set.
5. Error analysis indicates that most errors occur in expected boundary cases between labels (especially Reaction vs Reasoned Analysis), rather than systematic misclassification.

For a real community tool, I would require a stronger standard before using the model for any automated decision: at least 75% accuracy, macro F1 of at least 0.65, and F1 of at least 0.50 for every label on a larger and more balanced held-out dataset. Even then, I would use it for routing or analytics rather than automated moderation because these labels involve judgment about user intent.

---
## AI Tool Plan 
I will use ChatGPT to find 5–10 ambiguous Valorant posts that sit near the decision boundaries between:

Help-Seeking vs Reaction
Help-Seeking vs Reasoned Analysis
Reasoned Analysis vs Reaction

### Label stress-testing: 
I used ChatGPT to review the label definitions and selected boundary cases, especially Help-Seeking versus Reaction and Reasoned Analysis versus Reaction. This helped identify that question form alone was too broad for Help-Seeking. I refined the taxonomy to exclude general polls, discussion prompts, and invitations for shared experiences. Synthetic or exploratory examples were not added to the final dataset.


### Annotation assistance: 
ChatGPT was used to discuss taxonomy wording and selected difficult annotation cases. I reviewed the final retained examples against the written decision rule rather than treating an AI suggestion as an automatic final label. The final dataset contains no synthetic examples.

### Failure analysis:

After evaluation, I used ChatGPT to help identify candidate patterns among the fine-tuned model’s wrong predictions. I verified the final conclusions by checking the confusion matrix and manually reviewing all 11 wrong predictions.

The final analysis focuses on the model’s tendency to over-predict Help-Seeking. It correctly identified one Reasoned Analysis post and three Reaction posts, but classified three Reasoned Analysis posts and six Reaction posts as Help-Seeking. It also missed two Help-Seeking posts when their frustrated tone resembled Reaction.

---

