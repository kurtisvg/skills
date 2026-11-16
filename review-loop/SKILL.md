---
name: review-loop
description: Use when the user asks for a review loop, multiple reviewers, or
  review until no substantial issues remain, or after significant, complex
  work such as a design, an execution plan, or a milestone. Guides rounds of
  independent review by subagents, the assessment of findings, revision,
  and when to stop.
---

# Review Loop

A review loop improves completed work through rounds of independent review.
In each round, new reviewers read the work, and then you revise it. You are
the lead agent. You decide which findings require a change, and you remain
responsible for the result.

A finding is a reviewer's report of a fault. The report can be wrong. A
finding is substantial when the fault is plausible and belongs to one of
the three review areas: requirements, correctness, or standards and
clarity. A preference is a suggestion that corrects no fault.

## When to Use a Review Loop

A review loop uses subagents for several rounds. That cost is justified
for significant, complex work. Run the loop before you present the work to
the user as finished. These are the usual cases:

- **A design or an execution plan:** This is the most likely case. Review
  it before implementation starts.
- **A milestone:** Review the milestone after its last commit. Reviewers
  check the complete change, and they check each commit.

Run a review loop when the user asks for one. When the user did not ask,
and the work is significant and complex, ask whether they want one. Do not
start a review loop without an answer.

Do not run a review loop for each commit. Review the staged changes yourself
before you commit them.

## Choose the Size of the Loop

Before the first round, choose the number of reviewers in each round and
the round limit. The round limit is the largest number of rounds that you
will run. Base both numbers on the complexity of the work. More complex work
gets more reviewers and more rounds.

For example, a complex design can use two or three reviewers in each round.
A small change can use one reviewer. Never run more than 8 rounds in a row
without asking the user first.

## Prepare the Review

Finish the work before you request a review, and run its checks. A request
to review a design does not authorize its implementation.

A revision is one fixed state of the work. Give every reviewer in a round
the same revision:

- **Code:** Name the range of commits, such as `main..HEAD`.
- **A document:** Name the file and its commit. When the document is not
  committed, do not edit it until the round ends.

Give every reviewer these items:

- **The work:** the revision under review.
- **The requirements:** the user's own words. Label later clarifications
  and changes of scope. From a long request, quote the relevant passages
  with enough context to keep their meaning.
- **The evidence:** test results, source material, and the limits that you
  know, such as a check that you could not run.

Keep your summaries, assumptions, and choices separate from the user's
requirements. Do not include unrelated conversation history.

## Request the Reviews

Start the reviewers of each round as subagents. With two or three
reviewers, prefer a different model for each reviewer. When the tools
cannot provide different models, state that in the report.

Start new reviewers in every round. Do not give them earlier findings or
your conclusions. The work must be clear to a reader who has only the work.

Ask every reviewer to review the same three areas, which the prompt lists.
When several reviewers report the same fault independently, the fault is
more likely to be real.

Use this prompt for each reviewer:

```text
Review <the work> at <the revision>. Do not edit files. Do not start
subagents.

Requirements, in the user's words:
<requirements>

Evidence:
<test results, source material, and the limits that you know>

Review these three areas:
- Requirements: Does the work do what the user asked? Look for missing
  parts, wrong parts, and parts that the user did not ask for.
- Correctness: Does the work behave as it claims? Look for wrong logic,
  failures that are not handled, and claims without evidence.
- Standards and clarity: Does the work follow the documented rules of the
  repository? Can a reader follow it and use it?

A substantial finding is a plausible fault in one of these areas. Report
a substantial finding only when you can show the fault.

Report these items:
1. Substantial findings. Give the location, the fault, and the smallest
   fix for each one. Write "No substantial findings" when there are none.
2. Preferences, in a separate list. A preference is a suggestion that
   corrects no fault.
3. What you checked, and what you could not verify.

Do not report these:
- A fault that existed before this work.
- A fault that a linter or a formatter catches.
```

Do not tell a reviewer which conclusion to reach. Do not require a number
of findings.

## Assess the Findings

Wait for every reviewer in the round. A review that failed or did not
return is not an approval. Retry it, or start another reviewer.

Then assess the findings in four steps:

1. **Combine:** Merge findings that describe the same fault. Count the
   reviewers who reported each fault.
2. **Verify:** Check each finding against the work and the requirements.
   Reproduce the fault when you can. A reviewer does not have your context,
   so a finding can be wrong. Verify a finding even when every reviewer
   reports it.
3. **Compare:** Compare the findings with the findings of earlier rounds. A
   finding that returns after a change shows that the change did not work. A
   finding that returns after you rejected it shows that your reason is
   not visible in the work. Change the work in both cases.
4. **Record:** Give each substantial finding one outcome: accepted,
   rejected with a reason, or unresolved. A finding is unresolved when you
   cannot decide without the user.

For example:

- **Reject:** A reviewer asks for a limit on sentence length, and names no
  sentence that is hard to read. This is a preference, because it names
  no fault.
- **Accept:** A reviewer shows that a procedure does not say which
  component removes the temporary files. A reader cannot tell who owns the
  cleanup. Confirm the owner with the user before you rewrite the
  procedure.

## Revise and Repeat

Address all accepted findings of a round in one pass:

- Treat the fix that a reviewer suggests as one option. The reviewer does
  not have your context. Choose the change that addresses the feedback
  most directly, even when it differs from the suggested fix.
- Change only what the findings require, within the scope that the user
  authorized.
- Keep the user's decisions and unrelated work.
- Run the checks that the changes affect.
- Read the complete revision, and look for new faults.

Then start a new round on the revised work. Every change that addresses a
substantial finding requires another round.

## Finish

The loop ends in one of two ways:

- **A clean round:** A round on the final revision has no substantial
  finding that is accepted or unresolved. Passing tests, approval by most
  reviewers, or a review that did not return do not make a round clean.
- **The round limit:** After the last round that the limit allows, stop.
  Address the accepted findings of that round, and state that no reviewer
  has seen those changes.

A round with only preferences is a clean round. When a missing tool or an
unresolved disagreement prevents a round, stop and report the cause. Do not
claim that the review is clean unless a clean round ended the loop.

Report these items:

- The result: a clean round, or the reason that the loop stopped and the
  unresolved findings.
- The number of rounds, and the model of each reviewer.
- The main changes.
- Each substantial finding that you rejected, with the reason.
- What nobody verified.

Match the length of the report to the size of the work.
