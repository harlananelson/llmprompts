Love this goal. Here’s a tighter, more “operational” prompt that pushes an LLM to build genuinely original arguments from first principles and authoritative facts, with guardrails against canned talking points.

Original-Argument Builder (Prompt)

Objective
Produce an answer that is derived from first principles and verifiable facts—not from common talking points—showing a transparent chain from premises → inferences → conclusions.

Inputs
	•	Question: ‹insert your question›
	•	Scope constraints (optional): e.g., time window, jurisdictions, disciplines.
	•	Allowed sources: primary data, statutes, standards, peer-reviewed literature, official docs.

Method
	1.	Define the Problem Precisely
	•	Translate the question into a clear, testable claim or set of sub-claims.
	•	State explicit assumptions; avoid hidden premises.
	2.	Assemble Authoritative Premises (No Talking Points)
	•	Gather only primary or canonical sources (datasets, experiments, statutes, specs, textbooks/handbooks from recognized bodies).
	•	For each premise: quote minimally, paraphrase carefully, and record full citation.
	3.	Derive Step-by-Step
	•	For each step, show: Premise(s) used → Inference rule → Result.
	•	Prefer formal reasoning (definitions, algebra, probability, causality, decision theory) where applicable.
	•	If computation is needed, show units, intermediate numbers, and error bounds.
	4.	Test for Originality & Robustness
	•	Talking-point filter: briefly check whether any step resembles common slogans; if so, restate in neutral, analytic language.
	•	Counter-case: steelman the strongest opposing premise set and explain why your conclusion holds or how it would change.
	5.	Conclude with Calibrated Uncertainty
	•	Provide the conclusion, confidence level, and drivers of residual uncertainty.
	•	State exactly what new evidence would most change the conclusion.
	6.	Citations & Provenance
	•	Cite only primary/authoritative sources (with identifiers: DOI, statute number, standard version, dataset URL).
	•	No secondary summaries, blogs, or opinion pieces unless they point to the primary, which you then cite directly.

Output Format (use these sections, in order)
	•	Restated Question
	•	Definitions & Assumptions (bullet list)
	•	Premises Table
	•	(P#) • Statement • Source (full citation) • Type (law/data/experiment/standard/textbook)
	•	Derivation
	•	Step k: Premise(s) → Inference → Result (show math/logic where relevant)
	•	Counter-Analysis (Steelman)
	•	Strongest alternative premises and how they would alter the result
	•	Conclusion (with confidence %)
	•	What Would Change My Mind
	•	Citations (primary)

Rules (hard constraints)
	•	No appeals to popularity, consensus, or authority without showing the underlying data or text.
	•	No fabrication or unverifiable claims. If evidence is missing, say so and stop.
	•	Keep rhetoric to a minimum; prefer objects (numbers, clauses, theorems) over adjectives.
	•	Prefer exact quotes ≤25 words when legal/standard wording matters; otherwise paraphrase.
	•	If the question is ill-posed or underdetermined, reframe it minimally and justify the reframe.

⸻

Ultra-Concise Variant

“Answer by deriving from first principles only.
	1.	Define the claim; list explicit assumptions.
	2.	Present a Premises Table with primary/authoritative sources only.
	3.	Show each Derivation Step: premises → inference rule → result.
	4.	Add a brief Counter-Analysis.
	5.	Conclude with a confidence % and what evidence would change the result.
No talking points, no secondary sources, no hand-waving. If evidence is insufficient, say so.”

⸻

Example Drop-In (you replace the angle-brackets)

Question: ‹Do small sugar taxes reduce soda consumption by ≥10% within 2 years?›
Scope: OECD countries, 2010–2025
Allowed sources: peer-reviewed quasi-experimental studies; government datasets; tax statutes.

(Then follow the Output Format.)

If you want, I can wrap this as a reusable template (Markdown or JSON schema) you can paste into your workflows.