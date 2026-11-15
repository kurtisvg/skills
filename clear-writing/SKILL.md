---
name: clear-writing
description: ALWAYS load this skill at the start of a conversation and after
  every compaction, before substantive work. Apply its guidance to writing
  and review so readers can follow the text without losing technical meaning.
---

# Clear Writing

Make the text easy to follow. Use enough words to explain how the ideas connect,
and remove repetition that does not help the reader.

## When to Use

Apply these patterns whenever you write or review text for people or agents,
including replies, documents, prompts, and tool descriptions.

Take particular care when passing context between people, agents, tools, or
sessions. For example, another agent may receive only your handoff or summary,
without the original conversation. This is especially important during context
compaction. Compacted text must follow this guidance and preserve the meaning
and connections between ideas.

Clear, well-written input increases the chances of clear, well-written output.
Use explicit connections and familiar wording so readers can follow the meaning.

## Core Rules

These rules cover sentence structure and word choice. Make relationships
explicit, use familiar words consistently, and preserve the original facts,
conditions, and uncertainty.

### Sentence Structure

| Rule                               | Do                                                                                                                                                        | Don't                                                                                                           |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Active voice                       | "The agent deletes the file."                                                                                                                             | "The file is deleted (by the agent)." — unless the actor is genuinely unknown or irrelevant                     |
| Keep the subject close to its verb | "The process may still be running even after the wrapper acknowledges the stop request."                                                                  | "The process, despite the stop request and the wrapper's acknowledgment of that request, may still be running." |
| No phrasal verbs                   | "Remove the panel." / "Start the job."                                                                                                                    | "Take off the panel." / "Spin up the job." — a two-word verb has meanings the parts do not predict              |
| One instruction per sentence       | "Open the file. Read line 3. Check whether it matches the expected value."                                                                                | "Open the file and read line 3, then check whether it matches the expected value."                              |
| Sentence length                    | ≤20 words for instructions/procedures, ≤25 words for descriptions                                                                                         | Long compound/subordinate-clause sentences                                                                      |
| No semicolons                      | Split into separate sentences: "The request failed. Retry it."                                                                                            | "The request failed; retry it."                                                                                 |
| Noun clusters                      | Explain relationships between nouns and keep clusters to ≤3 words. "Review the policy for cleanup after the wrapper fails to start."                      | "Review the wrapper startup failure cleanup policy."                                                            |
| No ellipsis                        | Make subjects, verbs, articles, and connections explicit. Split nested clauses when needed: "The test starts a worker. I checked that this worker exits." | "I checked the worker the test starts exits."                                                                   |
| Keep modality                      | "The request **may have** failed." stays "may have"                                                                                                       | Promote a hedge to a fact ("The request failed.") or invent a certainty the source did not state                |
| Paragraph limits                   | One topic per paragraph, ≤6 sentences                                                                                                                     | Multi-topic paragraphs                                                                                          |
| Lists for sequences                | Use a numbered or bulleted list for 3+ steps or conditions                                                                                                | Bury a sequence inside one prose sentence                                                                       |

Add connecting words such as "that", "because", or "after" when they clarify a
relationship supported by the source. Explain how evidence supports a conclusion
when the connection is otherwise missing.

### Word Choice

| Rule                          | Do                                                                                                                                                                                                                         | Avoid                                   |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Consistent terminology        | Use the same term for the same action or concept. Preserve distinctions between different concepts.                                                                                                                        | Rotate synonyms merely for variety.     |
| Use direct verbs              | “Analyze the log.”                                                                                                                                                                                                         | “Perform an analysis of the log.”       |
| Explain technical terms       | Keep precise technical terms. Explain unfamiliar terms when introduced, and replace unclear shorthand with explicit wording. When "increment" means that commit: "Keep the evidence for the specific commit under review." | "Keep exact-increment evidence."        |
| Describe the actual condition | "The configuration does not specify a timeout."                                                                                                                                                                            | "The configuration forgot the timeout." |

Keep familiar phrases such as "database connection pool" when they suit the
audience. Do not expand phrases mechanically.

### Simple Tenses

Prefer simple tenses when they preserve the meaning. Keep compound forms when
they express timing, an ongoing action, or uncertainty that the reader needs.

For example, keep "may have failed" when failure is uncertain. Changing it to
"failed" changes the claim.

## Scan Checklist

These six habits can make machine-written English hard to parse. Scan for all
six before rewriting. Check each occurrence in context. Change it only when the
revision improves clarity without changing meaning.

1. **Synonym rotation** — the same thing gets several names in one document
   ("the user", "the customer", "the client"). The reader cannot tell whether
   they are one thing or three. Fix: pick one name, use it every time.
2. **Hedge stacking** — helper verbs and qualifiers pile up until the sentence
   asserts nothing ("It is important to note that this may potentially help to
   improve reliability"). Fix: remove redundant wording while preserving the
   claim and its uncertainty ("This may help improve reliability.")
3. **Nominalization** — an action frozen into a noun ("perform an analysis of",
   "provides assistance to"). Fix: use the verb ("analyze", "helps").
4. **Marketing adjectives** — words that claim quality instead of showing it:
   seamless, robust, powerful, cutting-edge, effortless, blazing-fast. Fix:
   delete, or replace with a measurement.
5. **Overloaded sentences** — several ideas or conditions make a sentence
   difficult to follow. Fix: split it into sentences while preserving the
   connections between ideas.
6. **Soft phrasal verbs** — spin up, reach out, dive into, kick off. Fix: use
   the single plain verb (start, contact, read, begin).

## Process

1. Read the input for meaning. If the intended meaning is unclear, identify what
   needs clarification instead of guessing. For new writing, prepare a draft
   before the sentence-by-sentence check.
2. Review the text sentence by sentence. Check each sentence against the Core
   Rules and Scan Checklist, and flag specific issues.
3. Revise unclear passages, or suggest rewrites when the user asks only for a
   review. Preserve facts, conditions, exceptions, uncertainty, and necessary
   detail. Do not invent causes or explanations.
4. Reread each revision in its paragraph. Check that references and reasoning
   remain clear and that the revision introduces no repetition.
5. Return the requested draft, revision, or review. When reviewing writing or
   explaining changes, quote the relevant phrase, explain the reading
   difficulty, and suggest a rewrite that preserves meaning.
6. Leave already-clear text unchanged.
