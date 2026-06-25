# TakeMeter: r/TrueCrime Discourse Quality Classifier

## Community

I chose **r/TrueCrime** because it is one of the largest true crime communities on Reddit (2M+ members), with consistently active, text-heavy discussion. The discourse quality varies enormously: some posts compile primary sources from court documents and trial testimony, others advance elaborate theories based on podcast rumors, and others are purely emotional reactions. This variation makes it an ideal community for a classification task — the distinction between evidence-based discussion and speculation is one that regular members actively enforce through upvotes, comments, and moderation.

## Labels

| Label | Definition |
|-------|-----------|
| **evidence_based** | The post presents specific, verifiable facts from primary sources (court documents, police records, trial testimony, medical records) and cites where those facts come from. |
| **informed_opinion** | The post offers a reasoned interpretation or argument about a case, but the evidence is secondary, incomplete, or the conclusion goes beyond what the cited facts strictly support. |
| **speculation** | The post advances a theory about a case with little to no supporting evidence, or treats unverified claims (podcast rumors, "a friend said," internet theories) as plausible. |
| **reaction** | The post is primarily an emotional response to a case or event — shock, sympathy, outrage, or personal connection — with little to no factual or analytical content. |

### Examples

**evidence_based**
&gt; *"In the Menendez trial, Dr. Kerry English testified that Erik's throat injury at age 7 was 'consistent with oral copulation in children.' The medical record from Princeton Medical Center dated the day after the ER visit states: 'Hurt posterior pharynx, uvula, and soft palate. Healing well.' This was entered into evidence during both trials."*

&gt; *"The Belize Ripper case involved five victims between 1998-2000: Sherilee Nicholas (13), Jackie Malic (12), Erica Wills (8), Noemi Hernandez (14), and Jay Blades (9, missing). FBI and Scotland Yard both assisted Belize police. No charges were ever filed and the murders stopped in 2000."*

**informed_opinion**
&gt; *"I think the Menendez brothers' abuse claims are credible given the volume of corroborating evidence, but I also understand why the second jury convicted them — Judge Weisberg excluded much of the abuse evidence from the guilt phase, so jurors never heard the full context before rendering a verdict."*

&gt; *"While Darrell Brooks was clearly guilty based on the video evidence, the prosecution's decision to charge him with 76 counts rather than consolidating them made the trial unnecessarily complex and may have confused the jury on some lesser charges."*

**speculation**
&gt; *"What if the Belize Ripper was actually a foreign medical student who left the country in 2000? The precision of the mutilation suggests medical training, and the timing lines up with when most international students would have graduated. Has anyone looked into this?"*

&gt; *"I have a theory that Lyle Menendez orchestrated the whole thing and manipulated Erik into going along with it. Lyle was always the dominant one, and Erik's letter to Andy sounds like someone who was coerced, not someone with agency."*

**reaction**
&gt; *"I just watched the Menendez documentary and I am absolutely devastated for those boys. No child should ever go through what they did. The system failed them in every possible way."*

&gt; *"The Belize Ripper case makes my blood boil. Five little girls and nobody was ever charged. Those families deserve closure and they'll never get it."*

## Hard Edge Cases

### Edge Case 1: Evidence used to justify emotional conclusion
&gt; *"The Menendez brothers had so much evidence of abuse — medical records, cousin testimony, expert evaluations, even Jose's own behavior with Menudo members. But they still got life without parole. Our justice system is broken."*

**Handling:** If the post *uses* evidence to support an analytical claim (even one with emotional weight), label it `informed_opinion`. If the evidence is merely *mentioned* as context for an emotional outpouring, label it `reaction`. In this case, the evidence is used to justify the emotional conclusion, so → **informed_opinion**.

### Edge Case 2: Inference presented as probable conclusion
&gt; *"The police never found the murder weapon in the Belize Ripper case, but the precision of the mutilation suggests someone with medical training. FBI profilers at the time noted the 'surgical' nature of the wounds. Given that Belize only had one medical school in the late 1990s and it had a small graduating class, I think law enforcement should have cross-referenced alumni records with anyone who left the country around 2000. It seems like an obvious lead that was never pursued."*

**Handling:** If the post treats an inference as a *conclusion worth considering* and acknowledges its limitations, label it `informed_opinion`. If the post treats an inference as *probable or obvious* and presents it as if the evidence strongly supports it, label it `speculation`. The phrases "I think" and "it seems like" frame it as an opinion, but "obvious lead that was never pursued" overstates the certainty → **speculation**.

**If the post instead said:** *"One angle that doesn't seem to have been explored: with only one medical school in Belize at the time, alumni records could potentially narrow the suspect pool, though of course many people with medical training weren't from Belize. Has anyone seen reporting on whether this was pursued?"* — that would be **informed_opinion**, because it frames the inference as a question, not a conclusion.

## Data Collection Plan

**Source:** r/TrueCrime posts and comments collected via Reddit's API (PRAW) or manual collection from the subreddit's front page and search results.

**Target distribution:** 50 examples per label minimum (200 total). If a label is underrepresented after 200 examples, I will:
1. Use targeted search terms to find more examples (e.g., search "court transcript" or "trial testimony" for evidence_based; search "my theory" or "what if" for speculation)
2. If still underrepresented, collect additional examples until the smallest label reaches at least 40 examples (20% of total)

**Collection method:** I will collect 30-40 examples first to verify the labels apply cleanly, then commit to the full 200. I will collect across multiple threads and time periods to avoid bias toward specific cases.

## Evaluation Metrics

- **Overall accuracy:** Baseline metric for comparing models.
- **Per-class F1 score:** More informative than accuracy for imbalanced labels. F1 balances precision and recall, which matters because misclassifying a rare label (e.g., evidence_based) is more consequential than misclassifying a common one.
- **Macro-averaged F1:** To ensure the model performs well across all labels, not just the majority class.
- **Confusion matrix:** To identify systematic misclassifications (e.g., does the model consistently confuse speculation with informed_opinion?).

I will not rely on accuracy alone because the label distribution may not be perfectly balanced, and accuracy masks poor performance on minority classes.

## Definition of Success

A genuinely useful classifier for r/TrueCrime would achieve:
- **Overall accuracy ≥ 70%** on the test set
- **Per-class F1 ≥ 0.60** for all four labels
- **Macro F1 ≥ 0.65**

I would accept this as "good enough" for deployment in a real community tool because:
- True crime discourse is inherently subjective, and even human annotators disagree on borderline cases
- A 70% accuracy with balanced per-class performance would meaningfully help moderators or users filter content by quality
- The baseline (zero-shot LLM) will provide a concrete benchmark — if fine-tuning achieves a 15+ percentage point improvement over zero-shot, the effort is justified

## AI Tool Plan

### Label Stress-Testing
I will give an LLM (Claude or GPT-4) my label definitions and edge case descriptions, and ask it to generate 5-10 posts that sit at the boundary between two labels. If it produces posts I can't classify cleanly, I will tighten my definitions before annotating 200 examples.

### Annotation Assistance
I will **not** use an LLM to pre-label examples. I will label all 200+ examples myself to ensure consistency and to build an intuitive sense of where the boundaries are. This also avoids disclosure issues around AI-assisted annotation.

### Failure Analysis
After training, I will give my list of wrong predictions to an LLM and ask it to identify systematic patterns (e.g., "the model consistently misclassifies short posts as reaction" or "it confuses speculation with informed_opinion when the post cites one source but draws a wild conclusion"). I will verify these patterns manually by re-reading the misclassified examples before writing my evaluation report.