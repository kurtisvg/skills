---
name: plan-work
description: Use when planning a feature, fix, milestone, or multi-step
  implementation from requirements, when dividing work into commits, or when
  writing a design document. Guides the system design, the component
  designs, the alternatives for major choices, and an execution plan of
  small commits.
---

# Plan Work

Use this skill to produce a design and an execution plan from requirements.
An execution plan is an ordered list of small commits. The design and the
execution plan together are the plan. Write the design before the execution
plan. Design the system first, and then each component.

Before you design, read the request, the repository guidance, and the code
that the work changes. Separate facts from assumptions. Ask each open
question that changes the design before you write the design.

## Design the System First

When the work needs more than one component, start with the system design.
When it needs one component, go to "Design Each Component".

A system design names the components and shows how they connect. State
these items for each component:

- **Responsibility:** what the component does, and what it does not do.
- **Connections:** which components call it, and which components it calls.
- **Ownership:** which data and which resources it owns.

Inspect the existing components and their callers first. Follow the existing
boundaries. Do not move a responsibility only to make a new component
simpler. Omit the internals of each component.

For example, a request to create a project must store metadata and prepare
a Git directory:

| Component       | Responsibility                                                      | Connections                    |
| --------------- | ------------------------------------------------------------------- | ------------------------------ |
| HTTP handler    | Parses requests and returns responses.                              | Calls the project manager.     |
| Project manager | Coordinates the metadata and the Git directory. Owns the directory. | Calls the store.               |
| Store           | Stores the metadata. Owns the database schema.                      | Called by the project manager. |

Then show the call chain. The call chain lists the calls in order, with the
operation and the data of each call. Include the calls that follow a
failure:

```text
POST /projects {name}
└─ handler: parses the name
   └─ Manager.Create(ctx, name) → Project
      ├─ 1. Store.Insert(ctx, project)    stores the metadata
      ├─ 2. creates the Git directory
      └─ on failure of 2: Store.Delete(ctx, id)
```

## Design Each Component

After the system design, write a smaller design for each new or changed
component. Start with the component that the entry point calls. The needs
of each caller then shape the contract of the component that it calls.

State the contract of the component. The contract is everything that a
caller can rely on:

- **Operations:** the signature of each operation, and one typical call.
- **Errors:** the failures that the caller can see.
- **Ownership:** the data and the resources that the component owns, and
  when it releases them.

Design only the operations that the call chain uses. Do not add an
operation, a field, or a column for later work.

For example, the project manager needs one operation:

```go
// Create stores the metadata of a project and prepares its Git directory.
// When the directory cannot be created, Create removes the metadata and
// returns the error.
func (m *Manager) Create(ctx context.Context, name string) (Project, error)
```

A typical call is `project, err := manager.Create(ctx, name)`.

When implementation shows that a contract must change, update the design,
the component, and its callers together.

## Consider Alternatives for Major Choices

A design choice is major when it is expensive to change after
implementation, or when it affects more than one component. For each major
choice, compare two or three real options before you choose one.

Consider alternatives in these cases:

- **A new dependency:** Compare it with the standard library and with code
  that the repository already has.
- **A new component:** Compare it with an extension of an existing
  component.
- **The owner of a responsibility:** Compare the components that could hold
  the data or perform the step.
- **Where data is stored:** Compare a database table, a file, and memory.
- **When work runs:** Compare work inside the request with work in a
  background task.
- **The order of steps that can fail:** Compare the orders, and the state
  that each order leaves after a failure.
- **A contract that is hard to change:** Compare the options for a database
  schema, a file format, a public API, or a command-line flag.

State these items for each major choice:

- **Options:** each option in one sentence.
- **Choice:** the option that you recommend.
- **Reason:** why it is better, and why you rejected each of the others.

For example, `Create` must store metadata and create a directory:

| Option                                                                    | Result                                                                                   |
| ------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Store the metadata first. Remove it when the directory cannot be created. | Chosen. A unique constraint on the name rejects a duplicate before any directory exists. |
| Create the directory first. Remove it when the metadata cannot be stored. | Rejected. Two requests with one name can both write to the directory before one fails.   |

Do not list alternatives for a routine choice. Do not invent an option that
nobody would choose.

## Order the Work

A behavior is one thing that a caller can ask a component to do, such as
the creation of a project. Start with the minimal form of each component.
Extend a component only after the components are connected. Order the
commits in three stages:

1. **Build each component in its minimal form.** Do not add a struct field
   or a schema column until a commit needs it. Each commit includes tests
   that call the component through its contract. A reviewer can then verify
   the component before its caller exists.
2. **Connect the components.** Connect them before any component gains a
   second behavior. The result is one working path: a request that passes
   through every component, from the entry point to the last component.
3. **Add behaviors.** Each commit adds one behavior to the working path.

A minimal component has few behaviors, and each behavior is complete.
Include the failure handling that the behavior requires. For example,
`Create` removes the metadata when the directory cannot be created. Omit
other behaviors. Do not omit failure handling.

A commit can use only what earlier commits provide. Put a tool, a fixture,
or a change to program startup in the first commit that needs it.

For the project example:

| #   | Stage             | Commit                                             |
| --- | ----------------- | -------------------------------------------------- |
| 1   | Minimal component | `feat(store): store projects with id and name`     |
| 2   | Minimal component | `feat(project): add Manager.Create`                |
| 3   | Connect           | `feat(server): add POST /projects`                 |
| 4   | Add a behavior    | `feat(project): list projects`                     |
| 5   | Add a behavior    | `feat(project): rename a project`                  |

The `projects` table starts with two columns. The first commit that reads a
`state` column adds that column.

## Size the Work

Aim for about 250 added lines in a commit. The limit is 500. Count code and
tests. Do not count deleted lines or generated files. When the repository
states its own limits, use those.

A commit can exceed 250 lines to keep a behavior with its tests. When a
commit would exceed 500 lines, divide it:

- Narrow the behavior. Deliver fewer cases now, and add the others in the
  next commit.
- Move preparation, such as a refactor, into an earlier commit.
- Do not move the tests into a separate commit.

Line counts in a plan are estimates. When a commit exceeds the limit during
implementation, and no good reason exists, stop and revise the plan. Divide
the commit, or change the order of the commits.

A milestone is a group of commits that together deliver one usable outcome.
Aim for 5 to 8 commits in a milestone. Fewer commits are fine. Divide a
milestone that needs more than 10 commits into sub-milestones.

Give each sub-milestone its own outcome, its own branch, and its own ordered
list of commits. State which sub-milestones it depends on. Make the minimal
components and their connection the first sub-milestone.

## Present the Plan

Present the design and the execution plan in one structure. Use it for a
plan in the conversation and for a saved design document. Use only the
sections that the plan needs.

```markdown
# Design: <Title>

## Summary

## Problem

## Goals

## Non-Goals

## Design

### Overview

### System Design

### Component Designs

## Alternatives Considered

## Execution Plan

## Open Questions
```

- **Summary:** State the problem and the chosen design in one paragraph.
- **Problem:** Describe the current behavior, and then what is wrong or
  missing. Start with what the reader already knows.
- **Goals:** List the outcomes that the work must produce.
- **Non-Goals:** List the related outcomes that the work does not include.
- **Overview:** Describe the approach in a few sentences, before the
  details. Name the main components and the main choice.
- **System Design:** Give the table of components and the call chain. Omit
  this section when the work has one component.
- **Component Designs:** Give the contract of each new or changed component.
- **Alternatives Considered:** Give the options, the choice, and the reason
  for each major choice.
- **Execution Plan:** List the commits in order. Group them by
  sub-milestone when the plan has sub-milestones.
- **Open Questions:** List the decisions that the user must make, and the
  assumptions that you made.

State these items for each commit in the execution plan:

- **Title:** the first line of the commit message.
- **Size estimate:** the approximate number of added lines.
- **Description:** the behavior that exists after the commit, and the check
  that proves it.

For example:

| #   | Title                                          | Size estimate | Description                                                                                                                           |
| --- | ---------------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `feat(store): store projects with id and name` | ~120          | Adds the `projects` table, `Store.Insert`, and `Store.Delete`. A test inserts a project and reads it.                                 |
| 2   | `feat(project): add Manager.Create`            | ~180          | `Create` stores the metadata and prepares the Git directory. A test makes the directory creation fail and checks that no row remains. |
| 3   | `feat(server): add POST /projects`             | ~150          | The handler calls `Manager.Create`. A test sends the request and checks the stored project.                                           |

Make each check exact. "Test the failure case" does not state the expected
result. "No row remains after a failed `Create`" does.

A design or an execution plan does not authorize implementation, a commit,
or a push. Wait for the user to approve the plan, unless the user already
authorized the work.

### Save a Design Document

Keep the plan in the conversation. Save it as a design document only when
the user asks. Use the first location that applies:

1. The location that the user names.
2. The location that the repository uses for design documents.
3. `design.md` in the directory of the feature.

When a design document for the feature exists, revise it. Do not create a
second one. Move long calculations and supporting evidence to an appendix.

Add these lines under the title:

```markdown
- **Status:** Draft
- **Created:** YYYY-MM-DD
- **Depends on:** links to related designs, or none
```

Give the document one status:

| Status     | Meaning                                                   |
| ---------- | --------------------------------------------------------- |
| Draft      | The design is open for changes.                           |
| Accepted   | The user approved the design. The work can be unfinished. |
| Rejected   | The user declined the design.                             |
| Superseded | A later design replaces it. Link to that design.          |
