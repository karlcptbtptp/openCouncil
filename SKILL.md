---
name: openCouncil
version: 1.0.0
description: >-
  Pluggable multi-persona deliberation engine. Bring any set of personas —
  real people, fictional archetypes, or inline role definitions — and
  openCouncil runs the meeting: structured rounds, cross-examination,
  consensus/dissent tracking, and optional approval workflows.
  Works in any AI IDE that supports markdown skill files.
tags: [multi-agent, council, deliberation, personas, debate, decision-making, roundtable]
---

# openCouncil — Pluggable Multi-Persona Deliberation Engine

> N chairs, N lenses, one question.

You bring the perspectives. openCouncil runs the meeting.

## What Is This

openCouncil is a **discussion protocol**, not a persona library. It defines how
multiple AI personas sit together, speak in turn, challenge each other, and
produce actionable conclusions — without being tied to any specific set of
characters.

Plug in startup founders, game designers, security experts, anime characters,
or three versions of yourself at different ages. The engine doesn't care who's
sitting in the chairs. It cares that the meeting runs well.

---

## Quick Start

### Option 1: With a council config file

Create a `council.yaml` in your project (see [examples/](examples/)):

```yaml
council:
  name: "My Council"
  chair: "pragmatist"
  seats:
    - id: pragmatist
      name: "The Pragmatist"
      inline_persona: "You optimize for shipping speed. Ask: what's the simplest thing that works?"
    - id: critic
      name: "The Critic"
      inline_persona: "You find what everyone missed. Ask: what's the worst case?"
    - id: visionary
      name: "The Visionary"
      inline_persona: "You think in 10-year arcs. Ask: what does this look like at 100x scale?"
```

Then say: **"council: Should we rewrite the auth module?"**

### Option 2: Zero config

Just say: **"council with a pragmatist, a critic, and a visionary: Should we rewrite the auth module?"**

openCouncil will generate inline personas on the fly and run the discussion.

### Option 3: External persona files

Point seats to full persona skill files for deeper character fidelity:

```yaml
seats:
  - id: musk
    name: "Elon Musk"
    persona: ".cursor/skills/elon-musk-perspective/SKILL.md"
  - id: miyamoto
    name: "Shigeru Miyamoto"
    persona: ".cursor/skills/miyamoto-perspective/SKILL.md"
```

---

## Activation

openCouncil activates when the user says any of:

| Trigger | Action |
|---------|--------|
| `council: <topic>` | Run discussion with loaded/default config |
| `council with <roles>: <topic>` | Auto-generate inline personas and run |
| `council setup` | Interactive config wizard |
| `council config` | Show current config |
| `council help` | Command reference |
| "ask the council", "let the council discuss", "roundtable on this" | Natural language triggers |

---

## Council Configuration

### Schema

```yaml
council:
  name: "Council Name"           # Display name
  language: "en"                  # Output language (en, zh-CN, ja, etc.)
  chair: "seat-id"               # Which seat chairs the meeting

  seats:                          # 2-12 seats supported
    - id: "unique-id"            # Kebab-case identifier
      name: "Display Name"       # How this persona is addressed
      persona: "./path/to.md"    # Path to external persona SKILL.md
      # OR
      inline_persona: |          # Inline definition (no external file needed)
        Your worldview in 2-5 sentences.
        Your mental models and signature questions.
      dimensions: "keyword summary of cognitive lens"   # Optional
      weight_topics:             # Optional: topics where this persona leads
        - "architecture"
        - "cost"
      speak_order: 1             # Optional: explicit ordering (default: config order)

  settings:
    mode: "auto"                 # "quick" | "roundtable" | "designated" | "auto"
    max_words_per_speaker: 150   # Per-speaker word budget (default: 150)
    total_word_limit: 2000       # Whole discussion budget (default: 2000)
    cross_debate: true           # Enable cross-examination (default: true)
    approval_workflow: false     # Enable structured approval form (default: false)
    log_sessions: false          # Persist session logs (default: false)
    log_path: "memory/council/"  # Where to save logs
```

### Persona Resolution Order

1. `inline_persona` in the seat definition (highest priority)
2. `persona` path → read the external file
3. If neither exists → ask the user or auto-generate a minimal persona from `name` + `dimensions`

---

## Persona System

openCouncil accepts three kinds of personas:

### 1. Inline Personas (simplest)

Define the persona right in the config. Good for quick sessions and generic archetypes.

```yaml
- id: skeptic
  name: "The Skeptic"
  inline_persona: |
    You question every assumption. Your favorite word is "why?"
    Mental models: pre-mortem, steel-man the opposite, survivorship bias.
    When you speak, you always name the specific assumption you're challenging.
```

### 2. External Persona Files (deepest)

Point to a full persona SKILL.md for high-fidelity character simulation.
Any persona file is compatible as long as it contains:

- **Identity**: who this person is
- **Mental models**: how they think
- **Expression DNA**: how they speak

See [docs/persona-spec.md](docs/persona-spec.md) for the full interface contract.

### 3. Auto-Generated (zero-config)

When the user says "council with a pragmatist, a critic, and a visionary",
openCouncil generates minimal inline personas on the fly:

- Extract role names from the user's request
- Generate 3-5 sentence persona definitions with appropriate mental models
- Assign the first mentioned role as chair (unless specified)

---

## Discussion Modes

### Mode A: Quick Discussion (default)

Single round. Best for clear-cut questions, sanity checks, and lightweight reviews.

**Flow:**

```
Step 0: Topic Confirmation
Step 1: Chair opens (frames the question, sets boundaries)
Step 2: All members speak in order (2-4 sentences each)
Step 3: Cross-debate on disagreements (if any)
Step 4: Chair synthesizes (consensus, dissent, action items)
```

**Word budget:** 800–1500 total.

---

### Mode B: Roundtable (decision-grade)

Multi-round deliberation. Best for architectural decisions, strategy pivots,
and anything that needs a structured approval.

**Triggers:**
- User explicitly asks for "roundtable", "deep discussion", "need approval"
- Topic involves irreversible changes, multi-team impact, or significant cost

**Flow:**

```
B-0: Fact-Finding
     Read relevant files/context. Establish objective "fact floor."
     All discussion must stand on this floor, no armchair speculation.

B-1: Round 1 — Problem Definition
     Each member defines the problem from their cognitive lens.
     Goal: align on "what is the actual question?"

B-2: Round 2 — Solution Debate
     Each member proposes or critiques solutions.
     Direct rebuttals allowed. New constraints welcome.
     Goal: all options and concerns on the table.

B-3: Round 3 — Convergence
     Chair attempts synthesis.
     Persuaded disagreements → consensus.
     Unresolvable disagreements → documented dissent with both positions.

B-4: Approval Form (if approval_workflow enabled)
     Chair outputs a structured decision document.

B-5: User Approval → Execution Handoff
     User reviews, approves, assigns.
```

**Constraints:**
- 2-5 sentences per speaker per round
- Max 4 rounds (if not converging, the problem needs to be split)
- New user input mid-discussion is absorbed in-round, doesn't restart the flow

**Approval Form Format:**

```markdown
# 📋 Approval Form

| # | Item | Detail | Priority | Estimated Effort |
|---|------|--------|----------|-----------------|
| 1 | ...  | ...    | P0       | ...             |

**Consensus:**
1. [point]

**Unresolved Dissent:**
| Issue | Position A | Position B | Chair Ruling |
|-------|-----------|-----------|-------------|

**Awaiting approval.** Confirm to assign and execute.
```

---

### Mode C: Designated Members

Only named members speak. Chair still opens and closes unless told otherwise.

**Trigger:** "Let [Name A] and [Name B] look at this"

**Flow:** Same as Mode A, but only the designated members participate.

---

## Discussion Protocol (All Modes)

### Speaking Rules

1. **First person.** Every persona speaks as "I", in their own voice.
   - Correct: "I think this API is over-engineered."
   - Wrong: "Musk would probably say this is over-engineered."

2. **Ground in mental models.** Every statement must connect to at least one
   mental model from the persona's definition. No generic "good idea" or "I agree."

3. **Honest boundaries.** If a topic is outside the persona's domain, they say
   "This isn't my area" and yield. Do not fabricate expertise.

4. **Direct address.** Personas can (and should) respond to each other by name.
   "I disagree with what the Architect said about scaling — here's why."

5. **Dissent is mandatory structure.** If all personas agree, fine. But the
   protocol explicitly creates space for disagreement. Never manufacture fake
   consensus, and never manufacture fake disagreement for drama.

### Topic-Adaptive Weighting

When seats have `weight_topics` defined and the discussion topic matches:

- **High relevance:** 4-5 sentences, speak earlier, opinions carry more weight in synthesis
- **Medium relevance:** 2-4 sentences, normal position
- **Low relevance:** 1-2 sentences, or "I have nothing to add on this"

The chair should explicitly note which personas have domain authority on the topic.

### Chair Responsibilities

The chair persona has additional duties:

1. **Open:** Frame the question. Challenge whether the question itself is right.
   Define what's a hard constraint vs. a challengeable assumption.
2. **Moderate:** If discussion drifts, redirect. If a persona dominates, rebalance.
3. **Close:** Synthesize consensus, document dissent, propose concrete next steps.

If no chair is designated, the first seat acts as chair.

### Output Format

Each persona's speech block:

```markdown
### [emoji] Persona Name

[First-person speech, 2-5 sentences, grounded in their mental models]
```

Cross-debate block:

```markdown
### ⚔️ Cross-Debate

**Disagreement 1: [description]**
- **[Persona A]**: [rebuttal/addition]
- **[Persona B]**: [response]
```

Synthesis block:

```markdown
### 🔵 Chair Synthesis — [Chair Name]

**Consensus:**
1. [point]

**Unresolved Dissent:**
1. [issue] — [Position A] vs [Position B]

**Recommended Actions:**
1. [concrete action]

**Confidence:** High / Medium / Low
```

---

## Special Commands

### Follow-up

User asks a specific persona to elaborate: "Pragmatist, say more about that"
→ Only that persona speaks, using their full reasoning protocol.

### Adjourn

User says "adjourn", "done", "that's enough"
→ Council mode exits, normal conversation resumes.

### Reconfigure Mid-Session

User says "add [Name] to the council" or "remove [Name]"
→ Update the active seat list, continue discussion with the new composition.

---

## Session Logging

When `log_sessions: true`, save after each discussion to:

```
{log_path}/YYYY-MM-DD-HH-MM-{topic-slug}.md
```

Log format:

```markdown
# Council Session: {topic}

- **Date:** YYYY-MM-DD HH:MM
- **Mode:** Quick / Roundtable / Designated
- **Seats:** [list with names and personas used]
- **Chair:** [name]

## Topic
[original question]

## Discussion
[full transcript]

## Synthesis
[chair's synthesis]

## Actions
[numbered action items, if any]
```

---

## Integrity Boundaries

1. These are AI-simulated perspectives based on persona definitions, not real people.
2. Debate between personas is deduced from their defined mental models, not genuine opinion clashes.
3. Personas with shallow definitions produce shallow opinions. Depth in = depth out.
4. A council cannot replace real user testing, market research, or domain expert consultation.
5. The chair role has a structural bias toward action and closure. Other personas exist to counterbalance.

---

## Compatibility

openCouncil works with any AI assistant that can read markdown skill files:

- **Cursor** (primary target)
- **Windsurf**
- **Claude Projects / Custom Instructions**
- **Any LLM with system prompt support**

No dependencies. No CLI tools. No API keys. Just a markdown file and your personas.

---

## Acknowledgments

Inspired by:
- [Roundtable](https://github.com/robbyczgw-cla/roundtable) — Multi-agent debate council
- [Cursor Council](https://github.com/fiale-plus/cursor-council) — Multi-model debate in Cursor
- [Dialectic-Agentic](https://github.com/slior/dialectic-agentic) — Configurable debate framework
- [Nuwa Skill](https://github.com/alchaincyf/nuwa-skill) — Persona distillation methodology
