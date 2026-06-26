---
name: bdd-scenarios
description: Agent-only reference for defining Behavior-Driven Development acceptance scenarios when delegating work. Use before running a design phase for a behavior-shaped ship task, before filling the acceptance-scenarios block in a brief, or when judging whether a task warrants scenarios at all. Covers the discovery/formulation/automation flow, Given/When/Then semantics, scenario best practices and anti-patterns, proportionality, and the executable-vs-prose decision.
user-invocable: false
---

# bdd-scenarios

Use this reference when a behavior-shaped ship task needs its intent pinned down before a crewmate builds it: to run the design phase with the captain, to author the Given/When/Then scenarios that go into the brief's acceptance-scenarios block, and to decide whether a task warrants scenarios at all.

Keep the always-inline rules in `AGENTS.md` authoritative: behavior-shaped ship tasks get captain-approved acceptance scenarios in the brief; trivial mechanical and non-behavioral changes do not; scout tasks never do.

## 1. What BDD is — and the trap

BDD (Dan North, 2006) is a reframing of TDD around *behaviour* instead of *tests*. The load-bearing fact, which every authoritative source insists on, is:

> **BDD is a collaboration and specification practice, not a testing tool.** The tests are a byproduct; the value is the shared understanding that stops you building the wrong thing.

Cucumber's three practices, applied in order:

1. **Discovery** — a conversation about a concrete upcoming change, surfacing real examples from the user's perspective ("what it *could* do"). This is where the value is: the examples nobody thought of.
2. **Formulation** — writing the agreed examples in language both humans and machines can read ("what it *should* do").
3. **Automation** — turning each example into a test that drives the code, exactly like TDD ("what it *actually does*").

The trap is adopting only Automation. Scenarios written by one person *after* the design is settled are "just test automation" — you pay the ceremony cost without the shared-understanding benefit. Discovery and Formulation are where the payoff lives.

## 2. Firstmate's adaptation

Firstmate has no literal "Three Amigos" room, but the three perspectives map cleanly:

- **Captain = the business voice.** What is valuable, what "correct" means.
- **Firstmate = the analyst / facilitator.** Runs the design phase, asks the skeptical and edge-case questions, formulates the scenarios.
- **Crewmate = the implementer.** Turns the agreed scenarios into code and tests.

This preserves BDD's core: the scenarios are shaped in a conversation between the captain and firstmate, **not** authored by firstmate alone and dispatched. The flow:

1. **Discovery + Formulation (design phase).** For a behavior-shaped ship task, draft Given/When/Then scenarios and review them with the captain in Lavish *before any code exists* — when changing direction is free. Use the `plan` / `input` Lavish playbooks; the captain annotates, corrects, and approves. Their approval is the green light to dispatch.
2. **Into the brief.** The approved scenarios fill the `{SCENARIOS}` placeholder in the ship brief's `# Acceptance scenarios` section. They are now the crewmate's definition of "correct."
3. **Automation (crewmate).** The crewmate implements the change so every scenario holds and covers each with a passing test (see §7 for how). A scenario with no passing test is not done. This generalizes the existing rule that a reproduced bug's repro becomes its regression test.

A lightweight design phase (one or two scenarios, an obvious feature) can happen in a couple of chat exchanges rather than a full Lavish surface — match the ceremony to the stakes.

## 3. When to use scenarios — proportionality and fit

BDD scenarios are **required for behavior-shaped ship tasks** and **omitted elsewhere**. Judge by whether the change has observable behaviour a stakeholder could have an opinion about.

**Use scenarios** — features, bug fixes with observable symptoms, changed business rules, anything user-facing or with an observable contract.

**Skip scenarios** (write `N/A` in the brief block, with a one-line reason):
- Trivial mechanical changes: typo, dependency bump, rename, formatting.
- Pure refactors and internal-only changes with no behaviour change.
- Performance tuning and low-level algorithmic work where no stakeholder reads the spec — ordinary unit/benchmark tests fit better.
- All scout tasks (their deliverable is a report, not a change).

The authoritative proportionality test: scenarios pay off only when they capture behaviour someone other than the implementer cares to validate. If the captain would have no opinion about the scenario, it is ceremony — skip it and let the project's normal tests carry the change.

## 4. How to define a scenario — Given/When/Then semantics

The semantics are strict; getting them right is most of the skill.

- **Given** = context / preconditions. Puts the system into a known state. Describes *state*, not actions. **No actions, no assertions in a Given.**
- **When** = the single action or event under test. **Exactly one `When` per scenario** — this is the cardinal rule. More than one `When`/`Then` pair means more than one behaviour; split it.
- **Then** = the observable, verifiable outcome. **Assertions only** — never an action or a state change.

Mnemonic: Given = past context, When = present action, Then = future outcome.

Shape: `As a <role>, I want <capability>, so that <value>` frames the feature; each scenario is one concrete example under it.

## 5. Best practices

1. **Declarative, not imperative — the central rule.** Describe the behaviour, not the UI mechanics. The litmus test: *"Will this wording need to change if the implementation changes?"* If yes, it is too low-level.
2. **One behaviour per scenario.** One `When`/`Then` pair.
3. **Business / domain language.** No CSS selectors, endpoints, table names, or "click the button." Use the ubiquitous language the captain and the domain use.
4. **Concrete but not incidental.** Use real, specific values, but include only what matters to the behaviour. Avoid both vagueness ("some money") and noise (irrelevant fields).
5. **Independent and order-independent.** Each scenario provisions its own state and passes regardless of run order.
6. **Named actor, consistent voice.** Pick an actor and a tense and keep them; do not mix first and third person.
7. **No conjunctions inside a step.** Split "and" into separate steps.
8. **Group by rule.** When several scenarios illustrate one business rule, say so; one rule with too many examples is a sign the rule is too complex.

Example — the same behaviour, badly and well:

```
# BAD (imperative; breaks on any UI change; mixes setup mechanics into the behaviour)
Given I visit "/login"
When I enter "Bob" in the "username" field
And I enter "tester" in the "password" field
And I press the "login" button
Then I should see the "welcome" page

# GOOD (declarative; survives implementation change; states the behaviour)
Given "Bob" is a registered user
When "Bob" logs in with valid credentials
Then he sees his dashboard
```

## 6. Anti-patterns (reject these in review)

- **Imperative UI script** — step-by-step field entry and button presses. Collapse to the behaviour (`When "Bob" logs in`).
- **DOM/implementation coupling** — selectors, URLs, DB tables in steps. Couples the spec to the implementation; breaks on refactor.
- **Conjunction step** — `Given I have shades and a brand new Mustang`. Split into two steps.
- **Multiple behaviours** — more than one `When`/`Then` pair. Split into separate scenarios.
- **Wrong step type** — an assertion in a `Given`, or an action in a `Then`.
- **Vague scenario** — "When I withdraw some money / Then the balance is reduced." Use concrete values.
- **Scenario-outline bloat** — a row per near-duplicate case. Keep one row per genuine equivalence class.

## 7. Automation: executable vs prose — match the repo

**You do not need Gherkin or Cucumber to do BDD.** Given/When/Then works equally as plain-language acceptance criteria turned into ordinary tests (Fowler: GWT "is the same as Arrange-Act-Assert; you don't need Cucumber"). Decide per repo — never impose a new runner on a repo that has none:

- **Repo already has a BDD/Gherkin runner** → add the scenarios as feature files in that runner's idiom and wire step definitions. Detect by looking for `*.feature` files, a `features/` dir, or the framework in the dependency manifest.
- **Repo has no BDD runner (the common case)** → the scenarios become ordinary unit/integration tests, named to mirror each scenario's Given/When/Then. This is the default; it preserves behaviour-first specification with zero new tooling, regex glue, or step-definition maintenance.

The brief instructs the crewmate to make this choice; the skill is here so firstmate can sanity-check it during review.

Runner notes by ecosystem (current as of 2026), for when a repo does use one:
- **Ruby / JS / JVM** — Cucumber / Cucumber-JS / Cucumber-JVM. Serenity BDD adds living-documentation reporting on top.
- **Python** — `behave` (stakeholder-readable, no native parallelism) or `pytest-bdd` (rides the pytest ecosystem: xdist parallelism, reporting). Prefer `pytest-bdd` when the repo already uses pytest.
- **.NET** — **Reqnroll**, not SpecFlow. SpecFlow reached end-of-life on 2024-12-31; Reqnroll is its maintained successor.
- **Go** — Godog (official Cucumber for Go; runs alongside `go test`).
- **JS/TS, browser** — `playwright-bdd` generates native Playwright tests from feature files; `jest-cucumber` runs Gherkin as ordinary Jest tests.

Keep BDD/E2E at the thin top of the test pyramid — test at the appropriate level, as close to the code as possible. Browser-level scenarios are the icing, not the cake.

## 8. Procedure (end to end)

1. Classify the task. Behavior-shaped ship task → continue. Trivial/non-behavioral → write `N/A` with a reason in the brief block, skip the rest. Scout → no scenarios.
2. Run the design phase with the captain: draft Given/When/Then scenarios, review in Lavish (or briefly in chat for small changes), iterate, get approval. Capture rules and the examples that illustrate them; record open questions rather than guessing.
3. Apply §4–§6: declarative, one behaviour each, business language, concrete values, no anti-patterns.
4. Fill the `{SCENARIOS}` block in the ship brief with the approved scenarios; spawn as normal.
5. On `done`, sanity-check that each scenario is covered by a passing test and that the crewmate matched the repo's testing reality (§7) before relaying to the captain.

## Sources

- Dan North, "Introducing BDD" — https://dannorth.net/blog/introducing-bdd/
- Cucumber, "BDD" (three practices) — https://cucumber.io/docs/bdd/
- Cucumber, "BDD is not test automation" — https://cucumber.io/blog/bdd/bdd-is-not-test-automation/
- Cucumber, "Where should you use BDD?" — https://cucumber.io/blog/bdd/where_should_you_use_bdd/
- Cucumber, "Writing better Gherkin" — https://cucumber.io/docs/bdd/better-gherkin/
- Cucumber, "Gherkin Reference" — https://cucumber.io/docs/gherkin/reference/
- Matt Wynne, "Introducing Example Mapping" — https://cucumber.io/docs/bdd/example-mapping/
- Martin Fowler, "GivenWhenThen" — https://martinfowler.com/bliki/GivenWhenThen.html
- Gojko Adzic, "Specification by Example" — https://gojko.net/books/specification-by-example/
- Liz Keogh, "What is BDD?" — https://lizkeogh.com/2015/03/27/what-is-bdd/
- Reqnroll, "SpecFlow end-of-life" — https://reqnroll.net/news/2025/01/specflow-end-of-life-has-been-announced/
