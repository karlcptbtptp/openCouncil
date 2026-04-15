# openCouncil

**Pluggable multi-persona deliberation engine for AI assistants.**

Bring any set of personas — historical figures, fictional archetypes, domain experts, or quick inline roles — and openCouncil runs a structured meeting: ordered speaking turns, cross-examination, consensus/dissent tracking, and optional approval workflows.

No dependencies. No API keys. Just a markdown skill file and your personas.

## Why

Single-perspective AI answers reinforce your own biases. Multi-persona deliberation creates structured disagreement, producing more robust decisions.

But existing solutions either hardcode their personas (locked to 3 fixed roles) or require heavy infrastructure (tmux, multiple CLI sessions, specific model APIs).

openCouncil separates the **discussion engine** from the **persona library**. You define who sits at the table. The engine runs the meeting.

## Quick Start

### 1. Install

Copy the `SKILL.md` file into your project:

```
.cursor/skills/openCouncil/SKILL.md
```

### 2. Create a Council (optional)

Create `council.yaml` in your project root:

```yaml
council:
  name: "Architecture Review"
  chair: "pragmatist"
  seats:
    - id: pragmatist
      name: "The Pragmatist"
      inline_persona: "You optimize for shipping. Mental model: YAGNI. Ask: what's the simplest thing that works?"
    - id: architect
      name: "The Architect"
      inline_persona: "You think in systems. Mental model: blast radius. Ask: what breaks at 10x scale?"
    - id: critic
      name: "The Critic"
      inline_persona: "You find blind spots. Mental model: pre-mortem. Ask: what are we not seeing?"
```

### 3. Use

In Cursor (or any compatible AI IDE):

```
council: Should we migrate from REST to GraphQL?
```

Or skip the config entirely:

```
council with a pragmatist, an architect, and a devil's advocate: Should we migrate from REST to GraphQL?
```

## Features

| Feature | Description |
|---------|-------------|
| **Pluggable Personas** | Use inline definitions, external persona files, or auto-generated roles |
| **Three Discussion Modes** | Quick (single round), Roundtable (multi-round + approval), Designated (specific members only) |
| **Auto Mode Selection** | Decision matrix (reversible/irreversible × impact scope) picks the right mode |
| **Anti-False-Consensus** | Two-level check: soft probe ("what worries you most?") → Devil's Advocate if needed |
| **Cross-Debate Triggers** | Three explicit conditions: contradiction, anti-consensus escalation, chair judgment |
| **Per-Person Confidence** | Each persona rates their confidence (High/Medium/Low) with a one-sentence rationale |
| **Emotional Arc** | Chair sets the discussion's emotional intention at Step 0 |
| **Topic-Adaptive Weighting** | Personas speak more when the topic matches their expertise |
| **Honest Boundaries** | Personas admit when something is outside their domain |
| **Approval Workflow** | Structured decision forms for high-stakes choices |
| **Session Logging** | Persist discussions for institutional memory |
| **Zero Dependencies** | Pure markdown. Works anywhere. |

## Discussion Modes

### Mode A: Quick Discussion

Single round. Chair sets emotional arc → opens → members speak → anti-consensus check → cross-debate → chair synthesizes with per-person confidence.

Best for: sanity checks, lightweight reviews, quick opinions.

### Mode B: Roundtable

Multi-round deliberation with fact-finding, problem definition, solution debate, anti-consensus check, convergence with confidence scores, and optional approval forms.

Best for: architecture decisions, strategy pivots, irreversible changes.

### Mode C: Designated Members

Only named members participate. Same flow as Quick, fewer voices.

Best for: "Let the Architect and Critic look at this."

## Persona Types

### Inline (simplest, zero files)

```yaml
inline_persona: "You are a cost optimizer. Ask: what's the unit economics?"
```

### External File (deepest)

```yaml
persona: ".cursor/skills/elon-musk-perspective/SKILL.md"
```

### Auto-Generated (zero config)

Just name the roles in your message:

```
council with an optimist, a pessimist, and a realist: Is this timeline achievable?
```

## Examples

See [`examples/`](examples/) for ready-to-use council configurations:

- **[game-design.council.yaml](examples/game-design.council.yaml)** — Game design review with UX, monetization, and operations perspectives
- **[tech-architecture.council.yaml](examples/tech-architecture.council.yaml)** — Technical architecture review
- **[data-analysis.council.yaml](examples/data-analysis.council.yaml)** — Data analysis and dashboard review

## Writing Custom Personas

See [docs/persona-spec.md](docs/persona-spec.md) for the full interface contract.

TL;DR — A compatible persona needs three things:

1. **Identity**: who they are
2. **Mental models**: how they think (2-5 named frameworks)
3. **Expression DNA**: how they speak (signature phrases, tone)

## Compatibility

Works with any AI assistant that reads markdown skill files:

- **Cursor** (primary target)
- **Windsurf**
- **Claude Projects / Custom Instructions**
- **Any LLM with system prompt support**

## Inspired By

| Project | What We Took | What We Changed |
|---------|-------------|-----------------|
| [Roundtable](https://github.com/robbyczgw-cla/roundtable) | Round 1/Round 2 cross-examination | Removed fixed Scholar/Engineer/Muse roles → any personas |
| [Cursor Council](https://github.com/fiale-plus/cursor-council) | Pluggable persona library concept | No tmux dependency → single-session protocol |
| [Dialectic-Agentic](https://github.com/slior/dialectic-agentic) | JSON-configurable debate agents | Simplified config, added approval workflow |
| [Nuwa Skill](https://github.com/alchaincyf/nuwa-skill) | Deep persona distillation methodology | Separated engine from personas |

## License

[MIT](LICENSE)

---

<p align="center">
  <i>N chairs, N lenses, one question.</i>
</p>
