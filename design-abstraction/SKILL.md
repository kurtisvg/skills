---
name: design-abstraction
description: Use when designing or reviewing APIs, interfaces, types, or
  component boundaries. Guides sketching the contract before implementation,
  preferring deep abstractions, and shaping inputs, results, and names.
---

# Abstraction Design

Design from what the code must do and from the code that calls it. Before
you propose a type or an interface, list the behavior that the task
requires. Then read the existing call sites, or write the calls that new
callers will make. Do not start from a hierarchy of types and then fit the
callers to it.

Keep the abstraction slim. Include what the current requirements and the
requested extension points need. Leave out operations and options for
features that may come later. For example, a requirement for a replaceable
backend can justify an interface before a second implementation exists. A
possible future feature alone does not justify one.

This skill uses two terms:

- **Abstraction:** A part of the code that callers use without knowing how
  it works, such as an interface, a type, or a module.
- **Contract:** Everything a caller must know to use the abstraction
  correctly. This includes the operations and their types. It also includes
  invariants, required call order, errors, and cleanup duties. Some guides
  call this the interface. This skill uses "interface" only for a language
  construct, such as an interface type, a protocol, or a trait.

## Design the Contract First

Before you implement a new abstraction or substantially change an existing
one, sketch its contract. List the public operations with their inputs and
results. Show how callers use it. The sketch does not require an interface
type, an extra module, or a formal design document.

Present the sketch and explain these points:

- **Callers:** Which existing parts of the code call it.
- **Responsibility:** The job that only this abstraction does.
- **Invariants:** Any invariant that it enforces. An invariant is a condition
  that must always hold, such as "every ready project has its files".
- **Lifecycle:** Which code creates it, and which code closes it. How its
  lifetime differs from the objects around it.
- **Dependencies:** Which parts of the code it depends on. Which of them it
  owns and which it borrows. It cleans up a dependency that it owns. It does
  not clean up a dependency that it borrows.
- **Errors:** Which failures callers must handle.
- **Replaced code:** Which existing types or operations it replaces,
  simplifies, or leaves internal.

Use a short call chain, an ownership diagram, or a table when the
relationships are not obvious. Keep three relationships separate: which code
calls an object, which code owns its data, and which code owns its
resources. A reference is not ownership. For example, a conversation record
stores the ID of its project, but the conversation does not start, stop, or
delete the project.

Suppose project management must support replaceable backends. A proposal
could show this contract in the `project` package:

```go
type Manager interface {
	Create(ctx context.Context, id, name string) (Project, error)
	Get(ctx context.Context, id string) (Project, error)
	Close() error
}
```

Typical use:

```go
// On server startup. The server lends its store to the manager.
projects, err := local.New(ctx, path, storage)
if err != nil {
	return err
}

// ...
// In a request handler.
p, err := projects.Create(ctx, input.ID, input.Name)
// ...
p, err = projects.Get(ctx, id)

// ...
// On server shutdown.
projects.Close()
```

The local implementation has these dependencies:

```text
Server → local.Manager (implements project.Manager)
           ├─ store.Store (borrowed)
           └─ filesystem handle (owned)
```

The server creates and closes the manager and the shared store. The local
manager borrows the store and owns its filesystem handle. This example shows
one design. Other designs do not need a manager or an interface.

## Prefer Deep Abstractions

An abstraction is **deep** when it has a small contract and a lot of
implementation behind it. An abstraction is **shallow** when it has a large
contract and little implementation behind it, so it mostly passes calls
through. Prefer deep abstractions and avoid shallow ones.

Callers of a deep abstraction learn little and get a lot. Maintainers make
changes and find faults in one place.

Depth is a property of the contract. The implementation can consist of small
parts that have their own tests and are replaced separately. Keep those
parts out of the contract.

**Before:**

```go
type Manager interface {
	InsertMetadata(ctx context.Context, id, name string) error
	CreateDirectory(ctx context.Context, id string) error
	InitRepository(ctx context.Context, id string) error
	MarkReady(ctx context.Context, id string) error
	RemoveDirectory(ctx context.Context, id string) error
	DeleteMetadata(ctx context.Context, id string) error
}
```

**After:**

```go
type Manager interface {
	Create(ctx context.Context, id, name string) (Project, error)
}
```

The first manager is shallow. Each operation passes one call through to
the store or the filesystem. Every caller must know the order of the steps
and must undo them after a failure. The second manager is deep. The steps,
their order, and the cleanup are in the implementation, behind one
operation.

Ask these questions about a proposed contract:

- Can it have fewer operations?
- Can the parameters be simpler?
- Can the abstraction hide more? Look for a sequence that callers must run
  or a callback that they must supply. Often one operation can replace it,
  or code that already owns those resources can run the sequence.

Preserve the necessary safety guarantees when you simplify a contract.

Add an abstraction only when it provides a useful contract, has a distinct
lifetime, or isolates required variation. Check each new abstraction in
these ways:

- **Compare it with direct composition.** Consider the design without the
  abstraction, where callers use the existing components directly. Explain
  the complexity that the abstraction removes and the obligations that it
  adds.
- **Apply the deletion test.** Imagine that the abstraction is deleted. If
  no caller becomes more complex, the abstraction only passed calls through.
  If every caller must now repeat its work, the abstraction is useful.
- **Test through the contract.** Tests use the abstraction the way callers
  do. If a test must reach past the contract, the abstraction probably has
  the wrong form.

In the project example, required variation justifies the interface, because
the backend must be replaceable. The filesystem helpers stay private. A
separate adapter for directories is not justified yet. It becomes justified
when the directory code must be replaceable separately from the manager, or
when other code must share its lifecycle logic.

## Shape Inputs, Results, and Names

- **Inputs:** Accept the fields that callers control. Build generated
  metadata inside the abstraction. Do not accept a field that the
  implementation ignores, or configuration that repeats another value.
- **Results:** Return what callers use. Do not add a result type that only
  carries values the caller can derive where it needs them.
- **Data and handles:** Separate persistent data from runtime handles when
  their lifetimes or guarantees differ. For example, a project record
  remains after a restart, and an open file handle does not.
- **Names:** Name types for their responsibilities and lifetimes. Name
  operations for their effects.

**Before:**

```go
// Callers set ID and Name. The manager ignores Version and State.
Create(ctx context.Context, p Project) (CreateResult, error)
```

**After:**

```go
Create(ctx context.Context, id, name string) (Project, error)
```

The first operation accepts a whole `Project`, although callers control only
two of its fields. Its result type only wraps the project. The second
operation accepts those two fields and returns the project. The manager
constructs versions and other internal metadata.

Declare an interface in the module that owns its concept, where the
dependency graph allows it. Declaring the interface beside the code that
consumes it is a useful default, not an absolute rule.

## After Implementation

- Before handoff, compare the implemented contract and its callers with the
  sketch. Remove operations that no caller uses. Describe where the
  implementation differs from the sketch in ways that matter to callers.
- Tell the user which behavior you deferred, and name the concrete
  requirement that would justify a new abstraction.
- Passing tests support correctness. They do not show that an abstraction
  is necessary.
