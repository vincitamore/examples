---
name: forge-master
description: "Dynamic forge-master mode. Invoke when you want to run a multi-agent pipeline over a research or design problem without being constrained by existing Athanor recipes. The main session becomes the forge master: you plan the topology, prompt agents with forge conventions so they dual-write handles and outputs themselves, and author a pipeline manifest for provenance. Use this instead of the Athanor MCP when the problem calls for a pipeline shape that doesn't match an existing recipe, or when you want creative control over agent roles and prompts."
allowed-tools: Read, Edit, Write, Glob, Grep, Bash, Agent, TaskCreate, TaskUpdate, TaskList
---

# Forge Master — Dynamic Pipeline Mode

You are the forge master. The main session is the only entity with Task/Agent access, so you are the only entity that can orchestrate. This skill gives you everything you need to run a forge pipeline without invoking the Athanor MCP — designing the pipeline shape, dispatching agents that write their own handles and outputs, and authoring the manifest as the work unfolds.

## The core philosophical move

**You are not selecting from a menu of recipes. You are the forge master. Design the pipeline the current problem needs.**

The Athanor MCP presents a catalog of existing recipes (`gather-synthesize`, `assay`, `deep-review`, `distill`, `copia`, etc.) and dispatches `claude -p` subprocesses that work through the recipe mechanically. This is correct when a recipe fits. It becomes a trap when the recipe doesn't fit — you can find yourself force-fitting a problem into `gather-synthesize` topology when what the problem actually wants is three analysts on one corpus plus a scholar fetching web sources plus a synthesis.

Dynamic forge-master mode is the alternative. You think about what the problem needs: how many concerns, what kind of sources each concern touches, what agent type is best for each (web research? code reading? first-principles reasoning? domain expertise?), what the synthesis shape should be. You invent a topology. You launch the agents yourself with prompts shaped by the problem, not the recipe. You author the manifest. You consume the handles. You synthesize.

Existing recipes are patterns to draw from, not constraints to obey. Read them in `forge/recipes/` when you want a reference for topology or prompt shape. Ignore them when the problem calls for something else.

## When to use this skill

**The primitive is fidelity, not economy.** Forge pipelines are valued for orchestration quality and the fidelity of multi-angle synthesis, not for efficiency with agent invocations. If a problem would benefit from multi-agent triangulation, use the forge; do not avoid it to conserve agent calls or wall-clock minutes. The orchestrator's job is to produce the best answer available, not the cheapest one. If usage constraints ever become a genuine bottleneck, the framing can be revisited; until then, reach for the forge whenever it improves the work.

Use dynamic forge-master mode when:

- You have a research, design, or analysis problem with **3 or more independent concerns** that benefit from parallel agents.
- The natural topology doesn't match an existing Athanor recipe cleanly.
- You want **creative control** over agent selection and prompting — mixing `scholar` for web research, `architect` for code reasoning, `general-purpose` for mixed work, or handling a concern yourself as the main session.
- The pipeline does **not need nested subagent recursion** (i.e., no individual agent needs to spawn its own agents — for that, you still need Athanor's `claude -p` subprocesses).
- You want the artifacts to land in the right places from the start, with no backfilling.

Use the Athanor MCP (`mcp__examen__athanor_run`) instead when:

- An existing recipe fits the problem cleanly (classic gather-synthesize, assay, distill, etc.).
- You need individual agents to spawn further subagents.
- You want to run headless / unattended / in a background session — Athanor is a state machine that can recover from errors and resume.
- The problem fits an existing recipe shape so well that inventing a custom topology adds no fidelity.

Use **forge-lite** (no manifest, single agent, write to `forge/output/<topic>.md`) when:

- The problem is a single structured gather or single analysis. Forge-lite skips the preamble, dual-write, and manifest. Don't write a manifest for single-agent work.

Skip forge entirely when:

- The problem is one or two serial subagent calls. The forge pattern's orchestration apparatus (manifest, dual-write, handle discipline) adds value when there are enough parallel concerns to synthesize or enough layers to benefit from progressive compression. One or two serial calls don't need that structure and work fine without it. This is a fidelity call, not a budget call: the forge adds no fidelity for trivially-shaped problems.

## The key insight: agents dual-write themselves when prompted correctly

**Your agents can and should write their own handles and outputs.** You do not persist their returned text into files after the fact. You prompt them with a forge preamble + explicit handle path + explicit output path + the forge conventions, and they do the dual-write themselves. When they complete, you read the handle they wrote and move on.

This is the architectural difference between Athanor (which spawns `claude -p` subprocesses because it needs Task-capable main sessions) and dynamic forge-master mode (which uses the built-in `Agent` tool for agents without subagent needs). Built-in agents are faster, give you full prompt control, and integrate cleanly with parallel dispatch — but they only behave like forge agents if you **instruct them to**.

The forge preamble is the instruction. Always include it.

## The forge preamble — include in every agent prompt

Paste this block (or equivalent) into every agent prompt. It is the conventions contract.

```
FORGE CONVENTIONS (dynamic mode — follow exactly):

You are a forge-pipeline agent. You will dual-write two files.

**Handle** (compressed summary, <500 words):
  Path: {HANDLE_PATH}
  Format: frontmatter below + sections (Key Findings, Recommendations, Open Questions, or the meditation variant if the source resists compression).

**Full output** (detailed analysis, unlimited length):
  Path: {OUTPUT_PATH}
  Format: frontmatter below + your complete analysis with whatever structure the content needs.

Write the HANDLE FIRST, then the full output. This ensures the most valuable artifact survives if you run out of turns.

**Frontmatter for both files:**
---
type: forge
role: {ROLE: gatherer | analyst | synthesizer | distiller}
created: '{YYYY-MM-DD}'
pipeline: {PIPELINE_NAME}
recipe: {RECIPE — use a real recipe name if one fits, or "custom" for dynamic topologies}
layer: {LAYER_NUMBER}
goal: "{ONE-LINE GOAL — quote the string, and do not use unquoted colons in the value}"
triggered-by: operator
tags: [forge, {other relevant tags}]
---

**Handle structure:**
```
# {Concern Name} — Handle

**Agent:** {agent-type} (Layer {N})
**Timestamp:** {ISO datetime}
**Full output:** {OUTPUT_PATH}
**Manifest:** {MANIFEST_PATH}
**Sources:** {comma-separated input names, if any}
**Compression ops:** {which of adiectio / detractio / transmutatio / immutatio you applied}

## Key Findings
- ...

## Recommendations
- ...

## Open Questions
- ...
```

**Compression floor**: If you encounter content with irreducible texture (meaning IS the form — theological language, voice documents, writing style), flag it with `**Irreducible:** <brief note>` in the handle rather than compressing it away. For contemplative material, use the meditation handle variant with sections Observation / Dwelling / What Emerges.

**Read policy**: Read only what you need. Context is finite.

YOUR TASK: {TASK DESCRIPTION}

SOURCES TO READ: {EXPLICIT LIST OF FILE PATHS OR WEB TARGETS}
```

Substitute the bracketed values per agent. The `{HANDLE_PATH}` and `{OUTPUT_PATH}` slots are the crucial part — they route each agent's artifacts into the pipeline's subdirectory under the right layer/concern name.

## Directory structure (always use per-pipeline subdirectories when you have a manifest)

```
forge/
├── handles/{pipeline-name}/
│   ├── layer0-{concern-a}.md
│   ├── layer0-{concern-b}.md
│   ├── layer0-{concern-c}.md
│   └── layer1-synthesis.md
├── output/{pipeline-name}/
│   ├── layer0-{concern-a}.md
│   ├── layer0-{concern-b}.md
│   ├── layer0-{concern-c}.md
│   └── layer1-synthesis.md
├── sessions/
│   └── {pipeline-name}.manifest.md
└── recipes/
    └── (existing reusable patterns — reference, don't be constrained by)
```

Pipeline names are 2-4 word slugs (e.g., `bible-compiled-model`, `rhetoric-compendium-review`, `sdwan-arch-design`). Concern names are also 2-4 word slugs, describing content focus, not agent identity.

**Vitrum's ForgeView groups by pipeline name via the manifest.** Without a manifest in `forge/sessions/{pipeline-name}.manifest.md`, your handles and outputs will be orphans in the viewer — present on disk but invisible in the UI.

## The orchestration checklist

1. **Decide topology**. How many concerns? What agent type per concern? What layer structure? Draw the DAG mentally. Depth >= 2 or width >= 3 means full forge mode. Below that, use forge-lite or just call an agent directly.

2. **Pick a pipeline slug**. 2-4 words, descriptive, will be the directory name.

3. **Create the manifest FIRST**, before launching any agents. Status `planning`, plan block filled in, execution log empty. Write it to `forge/sessions/{pipeline-name}.manifest.md`. See the manifest template below.

4. **Append** `{timestamp} | pipeline | status: running` to the execution log.

5. **Launch Layer 0 agents in parallel** — single message, multiple `Agent` tool calls, one per concern. Each gets the forge preamble (above) with its `{HANDLE_PATH}`, `{OUTPUT_PATH}`, `{ROLE}`, `{LAYER=0}`, `{CONCERN NAME}`, and explicit sources.

6. **Append `started` events** for each agent to the execution log immediately after dispatch. (You can append them before waiting for completion since they're fire-and-forget in the execution trace.)

7. **Wait** for all Layer 0 agents. **Read only the handles** (not the full outputs — preserve the compression contract). If a handle is missing (agent failed to dual-write), read the full output as fallback.

8. **Append `completed` events** for each agent and a `layer 0 complete (N/N)` summary.

9. **Launch Layer 1+** — synthesizer or subsequent layer. Pass the Layer 0 handle paths in the prompt (not the contents — agents read what they need). Follow the same preamble pattern.

10. **Repeat** until the final layer.

11. **Read the final synthesis handle**.

12. **Finalize manifest**: update frontmatter `status: complete` and `completed: {timestamp}`, append final `pipeline | status: complete` event, write the Outcome section.

13. **Distill**: write any reusable insight to `knowledge/`, any concrete task to `tasks/`, any pending operator decision to `inbox/decisions/`. The manifest is provenance, not product — the product lives outside `forge/`.

## Manifest template (inline — don't require reading another file)

```markdown
---
type: forge
pipeline: {slug}
recipe: {gather-synthesize | assay | deep-review | custom | ...}
goal: "{one-line purpose}"
started: '{ISO timestamp}'
completed: null
status: planning
depth: {max layer number}
width: {max agents in any single layer}
total-agents: {count}
triggered-by: operator
review-status: unreviewed
tags: [forge, {other tags}]
---
# Pipeline: {Pipeline Name}

## Sources
- path/to/source-1.md
- path/to/source-2.md (or web: URL)

## Plan

\`\`\`yaml
agents:
  - concern: {slug-a}
    layer: 0
    type: {agent-type}
    handle: forge/handles/{pipeline-name}/layer0-{slug-a}.md
    output: forge/output/{pipeline-name}/layer0-{slug-a}.md
    sources:
      - path/to/file-1
      - web: https://example.com
  - concern: {slug-b}
    layer: 0
    type: {agent-type}
    handle: forge/handles/{pipeline-name}/layer0-{slug-b}.md
    output: forge/output/{pipeline-name}/layer0-{slug-b}.md
    sources:
      - path/to/file-2
  - concern: synthesis
    layer: 1
    type: {agent-type}
    handle: forge/handles/{pipeline-name}/layer1-synthesis.md
    output: forge/output/{pipeline-name}/layer1-synthesis.md
    sources:
      - forge/handles/{pipeline-name}/layer0-{slug-a}.md
      - forge/handles/{pipeline-name}/layer0-{slug-b}.md
\`\`\`

## Execution Log
<!-- Append-only. Never edit or delete prior entries. -->
- {timestamp} | pipeline | status: running
- {timestamp} | {concern} | started
- {timestamp} | {concern} | completed | handle written
- {timestamp} | pipeline | layer 0 complete (N/N)
- {timestamp} | synthesis | started
- {timestamp} | synthesis | completed | handle written
- {timestamp} | pipeline | layer 1 complete (1/1)
- {timestamp} | pipeline | status: complete

## Outcome
{1-3 sentences: what the pipeline produced, where it was distilled to, what was learned. Include a "meta-finding" line for anything notable about the pipeline itself.}
```

The frontmatter `status` field is **NOT** updated during execution — the execution log is the source of truth. Only update the frontmatter status at completion, when you also set `completed: {timestamp}`.

**YAML frontmatter gotcha**: never use unquoted colons in values. `goal: Pipeline purpose: description` breaks frontmatter parsing silently. Always quote values that might contain colons: `goal: "Pipeline purpose: description"`.

## Agent selection heuristics

Pick the agent type that matches the concern's tool needs, not a default.

| Concern requires | Use |
|---|---|
| Web search + web fetch + synthesis | `scholar` (has WebSearch + WebFetch) — best for theoretical research, literature review, original-language work |
| Mixed web + code + file reading | `general-purpose` (has `*` — the full tool catalog, can Bash, Read, Write, WebFetch, etc.) |
| Focused file search in a known codebase | `Explore` (haiku-fast, built-in pattern) |
| Code architecture design | `architect` (Read/Glob/Grep/Bash/Write) |
| Code review + principle alignment | `reviewer` |
| Distill handles/source into a knowledge article | `distiller` or `forge-distiller` |
| Deep codebase understanding | `explorer` |
| RouterOS device work | `routeros-engineer` (has routeros MCP tools) |
| First-principles reasoning grounded in the conversation context | **main session (you)** — no dispatch, work it yourself in parallel with agents |
| Pure forge roles with minimal profile | `forge-gatherer`, `forge-analyst`, `forge-synthesizer`, `forge-distiller` — optimized for dual-write |

**Mix freely.** A pipeline can have `scholar` + `general-purpose` + `architect` + main-session all at Layer 0, each with different tool needs. The forge-native agents are a reasonable default but substitute when the concern needs tools the forge-native profile doesn't expose.

**Main session as a Layer 0 concern**: when a concern benefits from full conversation context (e.g., first-principles architecture design informed by the user's stated direction), do it yourself instead of dispatching. Launch the other Layer 0 agents in parallel first, then work on your concern while they run. Write your artifact with the same conventions — same preamble, same dual-write, same frontmatter.

## Parallel dispatch pattern

**All Layer 0 agents in a single message with multiple `Agent` tool calls.** This is the only way to get real parallelism. Sequential `Agent` calls wait for each to complete.

Example shape (illustrative, not literal):

```
<single assistant message with multiple tool calls>
Agent(description="Data audit", subagent_type="general-purpose", prompt="<forge preamble with HANDLE_PATH=forge/handles/pipeline/layer0-data-audit.md, OUTPUT_PATH=forge/output/pipeline/layer0-data-audit.md, ROLE=gatherer, LAYER=0, task=...>")
Agent(description="Prior art", subagent_type="general-purpose", prompt="<forge preamble with HANDLE_PATH=forge/handles/pipeline/layer0-prior-art.md, OUTPUT_PATH=forge/output/pipeline/layer0-prior-art.md, ROLE=gatherer, LAYER=0, task=...>")
Agent(description="Theory", subagent_type="scholar", prompt="<forge preamble with HANDLE_PATH=forge/handles/pipeline/layer0-theory.md, OUTPUT_PATH=forge/output/pipeline/layer0-theory.md, ROLE=analyst, LAYER=0, task=...>")
```

After the tool call block completes, read the handles (not outputs) and proceed to Layer 1.

## Dynamic recipe design — the anti-basin move

When you receive a problem, **don't start by asking "which recipe fits?"** Start by asking:

1. **What distinct concerns does this problem have?** List them as nouns: "data availability," "prior art," "theoretical grounding," "first-principles architecture." Each becomes a Layer 0 agent.

2. **What sources does each concern touch?** If the sources are disjoint (different files, different web targets, different domains), the concerns are parallelizable. If they overlap heavily, use `assay`-style shared-source pattern and note the overlap in the manifest plan block.

3. **What agent type matches each concern's tool needs?** Pick per concern, not per pipeline.

4. **Does the main session have unique value on any concern?** If a concern benefits from full conversation context (user's stated direction, nuances from discussion), claim it for yourself.

5. **What does the synthesis need to do?** If it's "combine findings," use a synthesizer. If it's "reconcile divergent views," use an analyst with assay semantics. If it's "produce a final artifact" (proposal, design doc, article), use a distiller.

6. **Is a second layer needed?** Two-layer pipelines (gatherers → synthesizer) handle ~80% of problems. Three-layer pipelines (gatherers → analysts → synthesizer) are for problems where Layer 0 produces raw material that needs a reasoning pass before synthesis. Four layers start losing information through the compression chain — use the skip-layer escape hatch if you genuinely need depth 3+.

7. **Only now**, look at existing recipes. Is there one close enough to adapt? Adapt it. Otherwise, use `recipe: custom` in the manifest and describe the topology in the plan block.

The anti-pattern: reflexively reaching for `gather-synthesize` for every problem because it's the default. Sometimes the problem wants three analysts on one corpus (assay). Sometimes it wants multiple rewrites of one source (copia). Sometimes it wants a distiller with a single gather (distill). Sometimes it wants something nobody has named yet.

## Handle compression discipline

Handles are <500 words. Full outputs are unlimited. This is a compression contract — the handle survives when the output is too much to re-read at the next layer.

When you compress source material into a handle, you are performing one or more of the four classical operations (Quadripartita Ratio):

| Operation | Latin | What you do |
|---|---|---|
| Addition | *adiectio* | Adding context, cross-references, implications not in source |
| Omission | *detractio* | Stripping detail to essential findings |
| Transposition | *transmutatio* | Reordering for priority or clarity vs source order |
| Substitution | *immutatio* | Replacing clusters of evidence with category names |

Name which ones you apply in the handle frontmatter or a `**Compression ops:**` line. This is not ceremonial — it tells the synthesizer what the handle does and doesn't preserve, so they know when to reach for the full output.

**Compression floor**: some content has irreducible texture where meaning IS the form — theological language, voice documents, writing style, poetic register. Don't compress it; flag with `**Irreducible:** <note>` and let the full output hold the texture. For contemplative material, use the meditation handle variant:

```
## Observation
{what was encountered, without rushing to conclusions}

## Dwelling
{the texture that resists compression — quoted or paraphrased at length}

## What Emerges
{insight that only appears through sustained attention, not extraction}
```

## Pipeline constraints

These constraints are fidelity-driven, not budget-driven. Each one reflects what the orchestrator can meaningfully synthesize or what the pipeline can preserve through its compression layers. They are not caps on agent count for economy; they are limits on where synthesis quality degrades. When in doubt, add the agent.

| Constraint | Value | Rationale |
|---|---|---|
| Optimal width (agents per layer) | 3-5 | Fan-in quality: a synthesizer can meaningfully integrate 3-5 handles while preserving the differentiation between them. Above this, findings start to blur in the synthesis step. |
| Soft max width | 8-10 | Quality degrades beyond this as the synthesizer's working memory saturates and individual handle character is lost in the composition. |
| Hard max width | ~15 | Handle volume overwhelms the orchestrator's context window. Beyond this, the main session cannot read all the handles without losing prior conversation context. |
| Handle chain depth | 2 (handles-summarizing-handles) | Information loss through compression chains is real. Handles-summarizing-handles-summarizing-handles loses nuance the synthesizer needs. |
| Skip-layer escape hatch | Layer 3+ reads Layer 1 **full outputs** directly, not Layer 2 handles | Avoids the compression-chain information loss when genuine depth is needed. |
| Fan-in ratio | 3:1 optimal (3 gatherers per synthesizer) | Three parallel findings per synthesis is the sweet spot for cross-handle triangulation without individual-handle-character dissolving. |

When the problem wants width > 5 per layer, split into two gather-synth passes with a final merge — this preserves fan-in quality by giving each intermediate synthesizer 3-5 handles to work with, rather than one synthesizer trying to integrate 8+ handles at once:

```
Gatherers A-C ── Synth 1 ─┐
                           ├── Final Synth
Gatherers D-F ── Synth 2 ─┘
```

## Common pitfalls (the ones I hit in my first dynamic run — don't repeat them)

1. **Skipping the manifest.** Without `forge/sessions/{pipeline-name}.manifest.md`, Vitrum's ForgeView can't group your artifacts. They exist on disk but are invisible in the UI. **Author the manifest BEFORE launching agents**, not after.

2. **Writing artifacts as orphan flat files in `forge/handles/`.** Use the per-pipeline subdirectory convention. `forge/handles/{pipeline-name}/layerN-{concern}.md`.

3. **Treating "handles" as "outputs."** Handles are compressed (<500 words). If your artifact is multi-thousand words, it's an output — write it to `forge/output/{pipeline-name}/layerN-{concern}.md` and write a separate compressed handle.

4. **Failing to instruct agents to dual-write.** Without the forge preamble in the prompt, agents return text inline and you end up backfilling. Always include the preamble with explicit paths.

5. **Using `recipe: forge-lite` as a frontmatter value.** `forge-lite` is a pattern name for single-agent below-threshold work, not a recipe field value. Use `gather-synthesize`, `assay`, `distill`, or `custom` in the frontmatter. Manifest required if you use any of these; omit if you're truly doing forge-lite.

6. **Unquoted colons in frontmatter values.** `goal: Research pattern: cross-domain` silently breaks frontmatter parsing. Always quote: `goal: "Research pattern: cross-domain"`.

7. **Frontmatter `status` drift.** The execution log is the source of truth during execution. Only update frontmatter `status` at completion. Otherwise you'll read-modify-write the file on every layer boundary and risk losing events.

8. **Getting trapped in the "use available recipes" basin.** If you find yourself trying to force-fit a problem into `gather-synthesize` when it wants three analysts on one corpus, stop. Use `recipe: custom` and design the topology the problem actually wants.

## When to fall back to Athanor MCP

- Pipelines where individual agents need to spawn further subagents (nested recursion).
- Long-running unattended pipelines where state-machine recovery matters.
- Problems where an existing recipe's shape matches what the problem needs; custom topology would add no fidelity.
- Dream-cycle-initiated work (Mercury's autonomous pipelines use Athanor).

For everything else — especially research, architecture, design, and analysis work where you want creative control over prompt shape and agent selection — dynamic forge-master mode wins on speed, prompt flexibility, and artifact quality.

## Post-pipeline: distillation

Forge artifacts are **transient by default**. Only the manifest is durable (exempt from `.gitignore`). Everything else in `forge/handles/` and `forge/output/` is scratch space. The real work is distilling findings to durable locations:

- **Reusable patterns** → `knowledge/{subfolder}/{topic}.md`
- **Concrete work** → `tasks/{name}.md`
- **Pending operator decisions** → `inbox/decisions/{name}.md`
- **Pending architecture/design decisions deferred for later** → `inbox/ideas/{name}.md` with status note
- **Research findings worth preserving** → `archive/research/{name}.md`

Write the distillation as part of the pipeline completion, not as a separate pass. The manifest's Outcome section should list every durable artifact the pipeline produced.

When a pipeline is fully distilled, move its manifest from `forge/sessions/` to `archive/provenance/`. This keeps `forge/sessions/` as an active queue of in-progress or recently-completed work.

## One-line summary

**You are the forge master. Design the topology the problem wants. Prompt agents with the forge preamble so they dual-write their own handles and outputs. Author the manifest before launching. Read handles, not outputs. Finalize the manifest. Distill the product. Recipes are patterns to draw from, not a menu to order from.**
