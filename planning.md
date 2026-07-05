## Community: 

I chose r/VALORANT on reddit because it is an active public community with weekly 9k+ posts. It is a good classification setting because posts vary substantially in intent: some users want specific help, some show their knowledge/skills about the game, and others mainly share reactions, clips, achievements, or frustrations. This variation makes the three labels meaningful rather than artificial.

## Labels: 

| Label     | Definition      | Primary Drive / Core Intent  |
| :-------- | :-------------- | :-------------------------- |
| **Help-Seeking**   | The author wants an answer, diagnosis, recommendation, feedback, or confirmation about a specific Valorant question, problem, decision, or situation.| **Seeking Guidance:** Driven by a question or problem that needs an answer, solution, or outside feedback.  |
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

For a frustrated question, I check the comment see whether the author is genuinely seeking an answer, diagnosis, recommendation, feedback, or confirmation about their own situation. If yes, I label it Help-Seeking even when the post contains emotional language or a detailed personal story.

For a complaint or opinion post, I apply the Emotion-Removal Test. I remove the author’s personal frustration, personal history, and agreement-seeking language, then ask whether a concrete claim about a mechanic, system, balance issue, strategy, meta, or community issue remains. I only use Reasoned Analysis when that remaining claim has a concrete reason, comparison, example, mechanism, or consequence.

General polls, “what do you think?” prompts, and invitations to share experiences are Reaction unless the author is seeking help for a specific personal problem. If a case remains genuinely ambiguous after applying these rules, I label it Reaction.

#### Annotation Decision Rule

1. **Is the author asking for an answer, recommendation, diagnosis, feedback, or confirmation?**  
   - Yes → **Help-Seeking**  
   - No → continue.

2. **Apply the Emotion-Removal Test.**  
   Mentally remove personal feelings, personal story details, and agreement-seeking language, such as:  
   - “I’m so tired of this.”  
   - “I want to uninstall.”  
   - “My teammates are awful.”  
   - “Does anyone else feel this way?”  

   Then ask:Does it make a concrete claim about a mechanic, balance, strategy, system, meta, or community issue?
   Is that claim supported by at least one reason, comparison, mechanism, example, or consequence? 
   - Yes → **Reasoned Analysis**  
   - No → **Reaction**

#### Tie Rule
Apply this rule only after a post has already been determined not to be Help-Seeking.

If a post remains genuinely ambiguous after applying these rules, label it **Reaction**.

This keeps Reasoned Analysis relatively strict while allowing concise posts that genuinely explain or support a gameplay point.

## Data collection plan: 
I began with a local collection of 400 public r/VALORANT posts. Each retained example contains the post title and body combined into a single `text` field, together with its `label`, Reddit post `id`, `postUrl`, and an optional `notes` field for difficult decisions.

I removed posts with missing, unreadable, or title-only text; posts whose meaning depended on an image or video; recruitment, promotion, or other non-discourse requests; and posts that did not fit a valid label.

After cleaning and re-annotating the dataset under the finalized taxonomy, the current dataset contains 223 labeled posts:
- Help-Seeking: 132 posts (59.2%)
- Reasoned Analysis: 28 posts (12.6%)
- Reaction: 63 posts (28.3%)

No label exceeds 70% of the final dataset. I also kept notes for some non-obvious cases so that I can later document at least three difficult annotation decisions in the README.

Reasoned Analysis is the underrepresented label. Rather than deleting useful Help-Seeking examples, the preferred next data-collection step would be to collect more posts that make supported claims about mechanics, balance, maps, ranked systems, agent design, or strategy. For the current time-limited training run, I will use all 253 examples, use a stratified split, and explicitly discuss the class imbalance when interpreting metrics.


## Evaluation Metrics

I will use overall accuracy, but accuracy alone is not sufficient because Help-Seeking is the largest class. A model could achieve misleadingly high accuracy by over-predicting Help-Seeking.

I will therefore also use:

- Per-class precision, recall, and F1 score, to show how well the model handles each label.
- Macro F1 score, because it weights all three labels equally rather than allowing the majority class to dominate the result.
- A confusion matrix, to identify which categories are most often confused, especially Help-Seeking versus Reaction and Reasoned Analysis versus Reaction.
- A comparison with the Groq zero-shot baseline on the exact same held-out test set, so that the difference between prompt-only classification and fine-tuning is meaningful.

The test split is small, especially for Reasoned Analysis, so I will interpret its class-level metric carefully rather than treating one score as conclusive.

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
I will use ChatGPT to generate 5–10 synthetic Valorant posts designed to sit near the decision boundaries between:

Help-Seeking vs Reaction
Help-Seeking vs Reasoned Analysis
Reasoned Analysis vs Reaction

These synthetic examples will only be used to test the clarity of my label definitions and will not be included in the training dataset.
**For label stress-testing**, I will conceptually use ChatGPT to review my label definitions and consider a small number of borderline cases across the three labels above. The goal is to check whether the taxonomy is sufficiently precise for consistent manual annotation. If cases are difficult to classify consistently, I will refine the labeling guidelines before proceeding with large-scale annotation.

**For failure analysis**, , I will not use automated pre-labeling. All training examples will be manually labeled according to the finalized taxonomy to ensure consistency and avoid propagating model bias. 
### Label stress-testing: 
Give the AI your label definitions and edge case description, and ask it to generate 5–10 posts that sit at the boundary between two labels. If it produces posts you can't classify cleanly, your definitions need tightening — do that now, before you annotate 200 examples.
### Annotation assistance: 
ChatGPT assisted with taxonomy refinement and review of difficult annotation cases. I will not treat an AI suggestion as a final label without checking the post against the written decision rule. The final dataset contains no synthetic examples, and the `notes` column labeling decisions. In the README, I will disclose that AI assisted the label-design and ambiguous-case review process.
### Failure analysis: 
Plan to give your list of wrong predictions to an AI tool and ask it to identify patterns before you write up your evaluation. Note what you'll look for and how you'll verify the patterns yourself.

will manually review misclassified test examples after evaluation. I may optionally use ChatGPT to help summarize recurring error patterns, but all conclusions will be verified through direct inspection of the data.

---

