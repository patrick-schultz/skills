---
name: codebase-design
description: Shared vocabulary for designing deep modules. Use when the user wants to design or improve a module's interface, find deepening opportunities, decide where a seam goes, make code more testable or AI-navigable, or when another skill needs the deep-module vocabulary.
---

# Codebase Design

Design **deep modules**: a lot of behaviour behind a small interface, testable through that interface, with seams only where a dependency genuinely varies. Use this language and these principles wherever code is being designed or restructured. The aim is leverage for callers, locality for maintainers, and testability for everyone.

## Glossary

Use these terms exactly: don't substitute "component," "service," "API," or "boundary." Consistent language is the whole point.

**Module**: anything with an interface and an implementation. Deliberately scale-agnostic: a function, class, package, or tier-spanning slice. _Avoid_: unit, component, service.

**Interface**: everything a caller must know to use the module correctly: the type signature, but also invariants, ordering constraints, error modes, required configuration, and performance characteristics. It covers what the module requires as well as what it provides: the data, configuration, and behaviour a caller must supply. _Avoid_: API, signature (too narrow, they refer only to the type-level surface).

**Implementation**: what's inside a module, its body of code. Distinct from **Adapter**: a thing can be a small adapter with a large implementation (a Postgres repo) or a large adapter with a small implementation (an in-memory fake). Reach for "adapter" when the seam is the topic; "implementation" otherwise.

**Depth**: leverage at the interface. The amount of behaviour a caller (or test) can exercise per unit of interface they have to learn. A module is **deep** when a large amount of behaviour sits behind a small interface, **shallow** when the interface is nearly as complex as the implementation.

**Seam** _(Michael Feathers)_: a place where you can alter a module's behaviour without editing the module: a dependency that something outside the module chooses. In code: an injected object, a generic parameter, a callback, a registry lookup, or a link- or build-time substitution. A seam carries behaviour, not data. A config value or environment variable is a parameter, though it can be the **enabling point** (where an adapter is chosen) for a seam elsewhere. A seam belongs to a dependency, not to a module: a module has one interface but any number of seams, and the seams its callers fill are part of that interface. A dependency the module hard-wires is part of its implementation, so the module's tests exercise it too, and when a change there breaks the module's contract, those tests should fail. _Avoid_: seam for the place tests attach (that is the interface); boundary (overloaded with DDD's bounded context).

**Adapter**: a concrete thing that fills a seam, implementing the interface of the dependency the seam stands for. Describes *role* (what slot it fills), not substance (what's inside).

**Leverage**: what callers get from depth. More capability per unit of interface they learn. One implementation pays back across N call sites and M tests.

**Locality**: what maintainers get from depth. Change, bugs, knowledge, and verification concentrate in one place rather than spreading across callers. Fix once, fixed everywhere.

## Deep vs shallow

**Deep module** = small interface + lots of implementation:

```
┌─────────────────────┐
│   Small Interface   │  ← Few methods, simple params
├─────────────────────┤
│                     │
│  Deep Implementation│  ← Complex logic hidden
│                     │
└─────────────────────┘
```

**Shallow module** = large interface + little implementation (avoid):

```
┌─────────────────────────────────┐
│       Large Interface           │  ← Many methods, complex params
├─────────────────────────────────┤
│  Thin Implementation            │  ← Just passes through
└─────────────────────────────────┘
```

When designing an interface, ask:

- Can I reduce the number of methods?
- Can I simplify the parameters?
- Can I hide more complexity inside?

## Principles

- **Depth is a property of the interface, not the implementation.** A deep module can be internally composed of small, mockable, swappable parts; they just aren't part of the interface. A module can have **internal seams** (private to its implementation, used by its own tests) as well as **external seams** in its interface, which its callers fill.
- **The deletion test.** Imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.
- **The interface is the test surface.** Callers and tests cross the same interface, and tests may fill its seams with their own adapters. If you want to test *past* the interface, the module is probably the wrong shape.
- **One adapter means a hypothetical seam. Two adapters means a real one.** Don't introduce a seam unless something actually varies across it.

## Designing for testability

Good interfaces make testing natural:

1. **Accept the dependencies tests must replace, don't create them.** Slow, nondeterministic, and external ones; the rest can stay hard-wired.

   ```typescript
   // Testable
   function processOrder(order, paymentGateway) {}

   // Hard to test
   function processOrder(order) {
     const gateway = new StripeGateway();
   }
   ```

2. **Return results, don't produce side effects.**

   ```typescript
   // Testable
   function calculateDiscount(cart): Discount {}

   // Hard to test
   function applyDiscount(cart): void {
     cart.total -= discount;
   }
   ```

3. **Small surface area.** Fewer methods = fewer tests needed. Fewer params = simpler test setup.

## Relationships

- A **Module** has exactly one **Interface** (the surface it presents to callers and tests).
- **Depth** is a property of a **Module**, measured against its **Interface**.
- A **Module**'s **Interface** exposes zero or more **Seams**: dependencies its callers choose.
- An **Adapter** fills a **Seam**, satisfying the **Interface** of the dependency the seam stands for.
- **Depth** produces **Leverage** for callers and **Locality** for maintainers.

## Rejected framings

- **Depth as ratio of implementation-lines to interface-lines** (Ousterhout): rewards padding the implementation. We use depth-as-leverage instead.
- **"Interface" as the TypeScript `interface` keyword or a class's public methods**: too narrow: interface here includes every fact a caller must know.
- **"Boundary"**: overloaded with DDD's bounded context. Say **interface** (where callers and tests attach) or **seam** (where a dependency is substituted).
- **"Seam" as the place tests attach**: that is the interface. Calling it a seam blurs testing *through* an interface with substituting a dependency *behind* it.

## Going deeper

- **Deepening a cluster given its dependencies**, see [DEEPENING.md](DEEPENING.md): dependency categories, seam discipline, and replace-don't-layer testing.
- **Exploring alternative interfaces**, see [DESIGN-IT-TWICE.md](DESIGN-IT-TWICE.md): spin up parallel sub-agents to design the interface several radically different ways, then compare on depth, locality, and seam placement.
