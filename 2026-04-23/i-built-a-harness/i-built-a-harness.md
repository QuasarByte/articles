# I Was Babysitting AI Agents. Then I Built a Harness.

*How I stopped babysitting AI agents and started shipping features*

I’ve been building with AI coding assistants for a while. For a long time, I was using them the wrong way.

I’d open a chat, describe what I needed, let the agent write code, review it, ask for fixes, get another version, and repeat. It worked — but it didn’t scale. Every session started from zero. Every new agent forgot the decisions made last week. Every context window came with fresh amnesia.

The shift happened when I stopped treating AI agents like better autocomplete and started treating them like a **team of contractors who need proper onboarding**.

That change led me to build something simple but transformative: a harness.

More importantly, it changed how I think about shipping software with AI.

## The Core Problem: Agents Don’t Have Memory

This sounds obvious, but the consequences compound fast.

Every time you start a new agent session, you’re effectively onboarding a new contractor. If the briefing is vague, the agent will make reasonable-but-wrong assumptions. It will choose naming conventions that don’t match the rest of the codebase. It will re-implement things that already exist. It will interpret ambiguous requirements differently from the last session.

The cost is not just a bad PR. The real cost is **drift**: a codebase that becomes less consistent with every feature until you lose momentum to cleanup and refactoring.

The fix is not better prompting.

It’s a harness.

## What Is a Harness?

A harness is the set of files, configurations, and conventions that let an AI agent — or a human — contribute to a project with minimal ramp-up and maximum consistency.

Mine has three layers:

**1. The Constitution** — the rules  
**2. The Map** — the structure  
**3. The Evidence** — the decisions and history

Here’s what each layer does.

## Layer 1: The Constitution (AGENTS.md)

This is the single highest-leverage file I’ve added to a project.

AGENTS.md is not a README. It’s not a style guide buried in a wiki nobody reads. It is the document that *every agent reads before touching code*.

It answers questions like:

- How do we name tests? (`methodUnderTest_scenario_expectedResult`)
- Which tags belong on unit tests versus integration tests?
- What has to be true before a feature can be called done?
- What rules are non-negotiable? No magic strings. Enums only. No raw file I/O outside the safe wrapper.

The key insight is simple: **agents follow explicit rules more reliably than implicit norms**.

If a rule lives only in your head, every agent will make its own plausible guess. Write it down, and you get consistent output across dozens of features and test files.

What surprised me most is that writing AGENTS.md improved *my* discipline too. A lot of what I documented were habits I’d followed informally for years. Once they were written down, they stopped being preferences and became standards.

## Layer 2: The Map (ARCHITECTURE.md)

If the constitution defines *how* to work, the map defines *where*.

ARCHITECTURE.md is a living guide to the codebase. It answers:

- What are the main packages, and what does each one own?
- What does the execution flow look like for a typical command?
- Where should a new feature be added, and where should it not?
- Which architectural constraints must not be violated?

Two things make this file especially valuable.

First, it captures **design decisions and their rationale**. For example: “Reports are persisted as JSON instead of being passed in memory because that decouples features and makes state inspectable.” One sentence like that can prevent the same architectural debate from resurfacing in three different sessions.

Second, it defines **boundaries**. Knowing what is out of scope is just as important as knowing what is in scope. Agents need those constraints or they will happily gold-plate.

## Layer 3: The Evidence (Features + Audit Trail)

This is where most people stop too early.

For every non-trivial feature, I write three documents:

- **feature-XXX.md** — requirements, acceptance criteria, and definition of done
- **technical-design-XXX.md** — the implementation plan, written before coding starts
- **audit-trail-XXX.md** — the decisions made during implementation, and why

The feature spec is the contract. The technical design is the plan. The audit trail is memory.

That last one is the compounding asset.

When an agent, six features later, needs to understand why the configuration system works a certain way, I don’t have to reconstruct the answer from commit history. The reasoning is already captured.

I also keep a `transcript-*.md` corpus with extracted Q&A from prior sessions, plus an `automatic-answer-*.md` file for each feature. Those files capture recurring questions and stable answers so future agents get consistent guidance without me repeating myself.

This is not overhead.

This is how you get through 30 features without turning the codebase into archaeology.

## The Permissions Layer (Don’t Skip This)

One practical thing I set up far too late was a Claude Code permissions allowlist.

Without it, agents ask for permission on nearly every file operation, test run, or git command. That creates decision fatigue — yours, not theirs. You either rubber-stamp everything or get interrupted at exactly the wrong moment.

The fix is to define what the agent can do autonomously:

- Run the unit test suite? Yes.
- Run integration tests? Yes.
- Stage and commit files? Yes, with documented conventions.
- Push to remote or deploy? No. Always ask.

It took about an hour to configure properly. It saved far more than that in interrupted flow.

## The Test Architecture That Holds It Together

None of this matters if you can’t trust the tests.

I separate tests into two tiers with Maven:

- **Unit tests** (`@Tag("unit")`) — fast, no filesystem, no Spring context, run in seconds
- **Integration tests** (`@Tag("integration")`) — real filesystem via temp directories, full Spring context, run in minutes

That separation is enforced at the build level. `mvn test` runs the fast loop. `mvn verify` runs everything.

Agents use the fast loop constantly and the full suite before declaring a feature complete.

The rule that made this system work is now baked into AGENTS.md: **every new feature ships with tests that prove it works in isolation and tests that prove it doesn’t break the whole**.

That rule turns “looks good” into something enforceable.

The result is that I can hand a new agent a feature spec, point it at the harness, and trust that what comes back has a real chance of integrating cleanly.

## What I’d Do Differently

If I were starting over, I’d do four things earlier.

**Write AGENTS.md before the first line of code.** Even a rough version helps. Retrofitting conventions later is much more expensive than establishing them early.

**Number features from day one.** Feature-001, Feature-002, and so on. It feels pedantic until you have 30 of them and need to reference one cleanly in a design doc.

**Capture decisions in real time.** An audit trail written during implementation takes minutes. Reconstructing it a week later takes hours and leaves gaps.

**Configure permissions deliberately.** Approving `mvn test` for the hundredth time is not safety. It’s noise.

## The Real Lesson

The harness is not really about controlling AI agents.

It is about building a system that lets you move fast without breaking the shape of the codebase — whether the next contributor is an AI, a contractor, or you six months from now.

The same things that make software maintainable for humans make it usable for AI: clear conventions, explicit decisions, visible boundaries, and traceable history.

The agents got better when I gave them better structure.

Honestly, so did I.

## Sources

- AI is forcing us to write good code — https://bits.logic.inc/p/ai-is-forcing-us-to-write-good-code
- ARCHITECTURE.md — https://matklad.github.io/2021/02/06/ARCHITECTURE.md.html
- Codex exec plans — https://developers.openai.com/cookbook/articles/codex_exec_plans
- Harness engineering — https://openai.com/index/harness-engineering/
