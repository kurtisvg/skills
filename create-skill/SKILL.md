---
name: create-skill
description: Use when creating, updating, or reviewing an agent skill. Guides
  the skill's scope, description, structure, and verification.
---

# Create Skill

## About Skills

Skills are packages that extend an agent's capabilities with specialized
knowledge, workflows, and tools. Think of them as playbooks for specific
domains or tasks.

Every skill contains `SKILL.md` and may include supporting resources:

```text
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter — used to decide when to activate the skill
│   │   ├── name
│   │   └── description
│   └── Markdown instructions — read after activation
├── scripts/ (optional)    — executable code
├── references/ (optional) — information loaded as needed
└── assets/ (optional)     — files used in the output
```

### Name and Description

The agent reads only the name and description when it decides whether to
activate a skill. The body is read after activation, so it cannot affect
that decision.

- **Name:** Use lowercase words separated by hyphens. Match the directory
  name. Name the task or subject, such as `fill-pdf-forms`.
- **Trigger:** Start the description with the requests that should activate
  the skill. Use the words a user would use in those requests.
- **Role:** Then state what the skill guides. This helps the agent choose
  between skills with similar triggers.

**Before:**

```yaml
---
name: fill-pdf-forms
description: Helps with PDFs.
---
```

**After:**

```yaml
---
name: fill-pdf-forms
description: Use when a user asks to fill a PDF form. Guides completing the
  form using the information they provide.
---
```

The revised description gives the agent a trigger and a clear role for the skill.

### Progressive Disclosure

Skills use **progressive disclosure** to load information when it becomes relevant:

1. **Name and description:** The agent uses these to decide whether to activate
   the skill.
2. **SKILL.md body:** The agent reads the shared instructions after activating
   the skill.
3. **Supporting references (optional):** These contain information useful for
   particular tasks. The agent reads them only when relevant to the request.

Keep `SKILL.md` under 500 lines and 5,000 tokens. Keep information needed on
every use in the `SKILL.md` body. When more content is necessary, move details
needed only for particular tasks into `references/` or a similar directory.

Link each reference directly from `SKILL.md`, with a clear condition for
reading it. Avoid chains that require opening one reference to discover
another. For references longer than 100 lines, include a table of contents
at the top.

**Before:**

```markdown
See references/ for details.
```

**After:**

```markdown
If the API returns an error response, read [API errors](references/api-errors.md).
```

## Core Principles

### Keep Skills Focused

Focus each skill on one task or a cohesive workflow with a clear outcome.
Define the requests that should trigger it. State what successful use should
produce.

Keep related steps together when they contribute to that outcome. Separate
tasks that users request independently and that need different guidance.

**Before:** One skill combines form filling and text extraction.

```yaml
---
name: pdf-tools
description: Use when filling PDF forms or extracting text from PDF research
  papers. Guides form completion and text extraction.
---
```

**After:** Each task has its own skill.

```yaml
---
name: fill-pdf-forms
description: Use when a user asks to fill a PDF form. Guides completing the
  form using the information they provide.
---
```

```yaml
---
name: extract-paper-text
description: Use when extracting text from PDF research papers. Guides
  producing readable text in the paper's reading order.
---
```

Form filling uses field names and supplied values. Paper extraction needs
guidance on reading order. These tasks need different instructions. Keep
inspecting, filling, and checking a form together because they produce one
completed form.

### Be Concise

Be concise without losing clarity. Skills share context with the user's
request, conversation history, and other instructions. Assume the agent already
understands common concepts. Include information it needs to make a decision
or avoid a plausible mistake.

Remove repeated guidance, generic background, and filler. Keep the facts,
conditions, and explanations that make the instructions usable. Prefer a
concrete example when it explains a choice more clearly than additional prose.

**Before:**

```markdown
CSV files store tabular data. You should always make sure that dates in
these exports are interpreted as DD/MM/YYYY.
```

**After:**

```markdown
Interpret dates in these exports as DD/MM/YYYY.
```

### Use Concrete Examples

Base examples on real requests, inputs, and results. Generalize them so that
they stand on their own. The examples do not need to name or link to the
original inputs. Preserve the conditions that make the example valid.

Put each example near the guidance it explains. Show the relevant context,
the intended action or output, and why it fits. Choose examples that resolve
ambiguity or demonstrate a decision.

For example, a writing skill can show how to replace unclear shorthand:

```markdown
When "increment" means the commit under review:

Before: "Keep exact-increment evidence."
After: "Keep the evidence for the specific commit under review."

The rewrite explains which evidence to keep without changing its scope.
```

### Match Control to the Task

Give the agent flexibility when several approaches can achieve the required
outcome. Specify steps when their order or consistency matters. Explain what
each constraint protects.

- **Allow judgment.** State the goal and what a useful result must include.

  > Explain the API change to its callers. Include compatibility effects
  > and any migration steps they need.

  The agent can choose the structure and level of detail.

- **Use an order when it prevents failure.**

  > 1. Convert the document to a temporary file.
  > 2. Verify that it opens and contains the expected pages.
  > 3. Replace the requested output only after those checks pass.

  This preserves the existing output if conversion fails.

- **Provide a default with a condition for alternatives.**

  > Extract embedded PDF text when available. Use optical character
  > recognition (OCR) for pages that contain only images.

  The condition tells the agent when to change approaches.

## Skill Design Patterns

Start with all instructions in `SKILL.md`. Move instructions into references
only when one of the reference patterns below applies.

### All Instructions in `SKILL.md`

Use this pattern by default. It applies when most uses need most of the
instructions and `SKILL.md` stays within its size limits. The skill can
include scripts or assets and still follow this pattern.

### Shared Workflow with References

Use this pattern when requests share a workflow but some inputs need additional
instructions. Keep the shared steps in `SKILL.md`. Link supporting references
where their guidance becomes relevant.

```markdown
# Extract PDF Text

1. Inspect the document.
   - For scanned pages, read [OCR](references/ocr.md).
   - For pages with multiple columns, read
     [Reading order](references/reading-order.md).
2. Extract the text in page order. Preserve headings and paragraph breaks.
3. Compare the extracted text with the source. Check for missing sections
   and text mixed across columns.
```

### References by Domain or Variant

Use this pattern when the workflow stays the same but details depend on the
domain, framework, or provider. Keep shared instructions in `SKILL.md`. Put
specialized guidance in separate references and explain when to read each one.

```text
cloud-deploy/
├── SKILL.md
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

Example instructions in `SKILL.md`:

```markdown
Use the provider specified by the user or project configuration.
Read the corresponding deployment guide:

- AWS: [AWS deployment](references/aws.md)
- Google Cloud: [Google Cloud deployment](references/gcp.md)
- Azure: [Azure deployment](references/azure.md)
```

The same structure works for business domains. A sales question may need only
`references/sales.md`. A question comparing sales with revenue may also need
`references/finance.md`.

## Skill Creation Process

### Step 1: Understand the Skill's Goal

Identify the use cases the skill should support and what successful use should
produce. Use available requirements, examples, conversations, and execution
traces. Include corrections that improved an earlier result. For an existing
skill, read its instructions and relevant supporting files before deciding
what to change.

Ask only about gaps that affect the scope or expected result. Do not ask the
user to repeat information already available.

### Step 2: Choose the Structure

Use the principles and patterns above as a starting point. Include supporting
resources **only** when needed. Decide which instructions every use needs and
which details belong in references for particular cases.

For example, keep a report's required sections in `SKILL.md`. Include an asset
when the report must use a supplied document template.

Include a script when an operation must produce the same result on every use,
or when the agent would otherwise rewrite the same code each time. In
`SKILL.md`, state when to run the script and what it produces.

### Step 3: Write or Revise, Then Edit Heavily

For a new skill, create its directory and `SKILL.md`. Draft the instructions
and examples around the chosen workflow. For an existing skill, preserve
behavior outside the intended change.

A verbose first pass is fine. Then edit the draft heavily, sentence by
sentence. Remove repetition, unnecessary background, and details that do not
help the agent act. Preserve clarity, useful examples, and conditions that
affect the result. Reread each section to check that the ideas connect and
the steps follow a clear order.

### Step 4: Check the Result

Check the result against the skill's goal:

- Check that YAML frontmatter is valid and includes `name` and `description`.
  When the client lists installed skills, check that the list shows the
  intended description.
- Write requests that should activate the skill and similar requests that
  should not. Check that the description matches only the first group.
- Check that `SKILL.md` stays within its size limits.
- Check that linked resources exist and their loading conditions are clear.
- Check that examples demonstrate the intended behavior without requiring
  missing context.
- Run any new or changed scripts. Inspect their outputs for the cases they
  must handle.

When you report these checks, distinguish results you observed from judgments
made by reading. State what remains untested.

### Step 5: Refine Through Observed Execution

When execution traces or conversations that used the skill are available, use
them to improve it. Examine actions and tool results as well as final outputs.
A plausible answer can hide skipped steps or incorrect results.

If the agent wastes time on unproductive steps, check for common causes:

- **Vague instructions:** The agent tries several approaches before finding
  one that works.
- **Instructions that do not apply:** The agent follows steps intended for
  a different task.
- **Options without a clear default:** The agent explores alternatives
  without guidance on where to start.

Use that evidence to identify what guidance was missing, unclear, or
unnecessary. Make the smallest change that addresses the observed problem.

For example, suppose a CSV cleanup task must preserve customer IDs. The output
changes `00123` to `123`, and the execution shows that the agent converted IDs
to numbers.

**Before:**

> Preserve customer IDs when cleaning the CSV.

**After:**

> Preserve customer IDs when cleaning the CSV. Treat IDs as text so leading
> zeros remain intact.

Rerun the task with the revised skill. Check related cases to ensure the fix
generalizes without breaking working behavior.
