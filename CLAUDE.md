# CLAUDE.md

Persistent memory for this project. Read before writing or editing content.

## What this is

A guide that helps **industry customers migrate their AI workloads from LUMI to a
production cloud**. LUMI is built for research and development, not for hosting a
running production service, so once a customer has trained or fine-tuned a model
on LUMI they need to move it somewhere it can serve requests. This guide walks
them through that.

It is a documentation website: Markdown in [content/](content/) is rendered by a
React + TanStack Start app. The site is based on the
[course-template](https://github.com/lumi-ai-factory/course-template). Authors
normally only touch `content/` and [site.config.ts](site.config.ts).

## Who reads it

SMEs, and their technical level varies enormously. The same page may be read by:

- a dedicated ML engineer who wants the commands and nothing else, and
- a near-non-technical founder who fine-tuned a model on LUMI by following our
  examples and prompting an LLM, and has never opened a cloud console.

Write for both. The working pattern (see the AWS pages) is a short plain-language
explanation of each cloud concept the first time it appears, phrased so a
technical reader can skim it and drop to the commands. Never assume prior cloud
knowledge; never talk down to the reader either.

## Scope

**In scope:** getting an already trained or fine-tuned model off LUMI and
**hosted as a production inference API** on a cloud provider. The journey is the
same everywhere and framed as four moves:

1. Move the weights off LUMI into the provider's storage.
2. Get a GPU machine or managed hosting service big enough for the model.
3. Load the model into a serving program that exposes it as an API.
4. Send requests, and switch it off when done.

**Out of scope (do not add unless asked):** training/retraining in the cloud,
data pipelines, MLOps/monitoring, non-LLM workloads. Keep the guide narrow.

## Structure (three levels)

1. **[content/index.md](content/index.md)** - general intro to the migration and
   how to choose a provider. Holds everything that is **true on every platform**
   (cost, preparing weights, the GPU-memory rule of thumb) so provider pages
   don't repeat it. The "choosing a provider" part may later be split into its
   own child page of the index.
2. **Provider intros** - one per provider: AWS, GCP, Azure, and a
   Finnish/European provider. Each covers setup shared across that provider's
   hosting methods.
3. **Children of the intros** - concrete step-by-step instructions, sizing
   tables, and troubleshooting for a specific service or task.

### Current file map

| File | Role | State |
|---|---|---|
| `content/index.md` | Level 1 intro + provider chooser | Fairly complete |
| `content/1_AWS.md` | AWS intro (IAM, sizing, quotas, cost) | Fairly complete |
| `content/1.1_Move_data.md` | Move data to AWS (`s3cmd`) | Complete; `rclone` section TODO |
| `content/1.2_EC2.md` | Host on EC2 | Stub ("Instructions by Lukas") |
| `content/1.3_SageMaker-AI.md` | Host on SageMaker | Complete |
| `content/2_GCP.md` | GCP intro | Stub |
| `content/2.1_Move_data.md` | Move data to GCP (`gcloud`) | Complete |
| `content/3_Azure.md` | Azure intro | Stub |
| `content/3.1_Move_data.md` | Move data to Azure | Stub |
| `content/4_Finnish_Cloud.md` | Finnish/European intro | Stub |
| `content/4.1_Move_data.md` | Move data to Finnish/EU | Stub |
| `content/glossary.md` | Term definitions (see `%` markers below) | Ongoing |

Some pages are authored by named people (e.g. EC2 "by Lukas"). Don't overwrite
another author's stub with generic filler; fill it in only when asked.

## Voice and style

- **No AI tells.** No emojis. **No em dashes (—) or en dashes (–)** anywhere.
  Rewrite with commas, colons, parentheses, or two sentences. This is strict.
- **British spelling everywhere** (e.g. "synchronise", "optimise", "licence").
- **Not dry, but tight.** Concise and genuinely interesting to read. Respect the
  reader's time: teach as much as they need and no more. Repetition is fine when
  it genuinely helps (e.g. re-stating the "switch it off" habit); avoid
  repetition that only pads.
- **Neutral, never an advert.** We do not promote any cloud. Give the real
  picture, honest pros and cons, and the traps, so the customer can make an
  informed choice of provider and service. Money and pitfalls stated plainly.
- **Show the pitfalls.** Common mistakes, "this fails with error X and here's
  why", and cost traps are core value, not asides.
- Use concrete numbers when useful (VRAM per GPU, GB per billion params, example
  hourly costs) and mark rules of thumb as approximate.

### Recurring anchors to reuse

- The **four moves** framing above.
- **Cost is billed by wall-clock hour while switched on, used or not.** The
  single most important habit is shutting the model down when finished. Repeat
  this per provider with the exact "how to switch off" steps.
- **GPU-memory rule of thumb:** ~2 GB per billion parameters for a 16-bit
  (bf16/fp16) model, plus headroom for the KV cache / long conversations.
- Models are served in **Hugging Face format**: one clean uncompressed folder
  (config + `*.safetensors` + tokenizer), nothing else.

## Related guides (link to these instead of re-explaining)

This guide picks up where two sibling guides leave off, so refer readers to them
rather than duplicating their material:

- **[LUMI AIF Onboarding](https://lumi-ai-factory.github.io/LUMI_AIF_Onboarding/)** -
  how to work on LUMI: access and security, the command line, nodes and storage,
  modules and containers, GitHub, and Slurm. Send readers here for any LUMI
  basics (CLI, storage paths, containers) instead of teaching them again.
- **[The Pragmatic Guide to LLMs](https://arbruiser.github.io/The-Pragmatic-Guide-to-LLMs/)** -
  how LLMs work: choosing a model, architecture, attention, multimodal, running
  inference on LUMI, customising/fine-tuning, and RAG. Send readers here for LLM
  concepts (what fine-tuning is, choosing a model, RAG) that are upstream of
  hosting.

## Authoring mechanics

**Editing whitespace:** never insert or remove blank lines or reflow/rewrap
paragraphs to "tidy" formatting. Leave line breaks and formatting to the author's
editor. Make minimal, surgical edits to the words themselves.

**Never commit.** Do not run `git commit`, `git push`, or stage changes on the
author's behalf. They commit everything themselves.

**Frontmatter** (YAML at top of each `.md`):

```yaml
---
title: "Unique Page Title"     # required; MUST be unique across all pages
nav_order: 2                    # ordering within its level
parent: "Exact Parent Title"    # links a child to its parent by exact title
description: "..."              # optional meta description; auto-derived if omitted
---
```

Nesting and breadcrumbs are driven by matching a child's `parent` to a parent's
`title` **exactly**. A typo silently drops the page to the top level (a console
warning is logged at build). Ordering within a level is `nav_order`.

**File naming convention:** `N_Provider.md` for a provider intro, `N.M_Topic.md`
for its children (e.g. `1.3_SageMaker-AI.md`).

**Cross-page links** use relative `.md` paths: `[SageMaker AI](1.3_SageMaker-AI.md)`.

**Callouts** use GitHub alert syntax with an optional title after the marker.
Variants: `NOTE`, `WARNING`, `INFO`, `TIP`, `COMMAND` (the command variant renders
a copy button).

```markdown
> [!WARNING] Optional custom title
> Body text of the callout.
```

Keep callouts **short** (a few lines at most): a callout is for a single sharp
warning, tip, note, or command, not for prose that belongs in the body. Never
place **two callouts back to back**; if two would end up adjacent, merge them or
move one into the surrounding prose.

**Glossary links:** put a trailing `%` on a term to give it a hover-over
definition pulled from `content/glossary.md`. The `%` is stripped from the output.

- `IAM%`, `**Hugging Face format**%`, and `` `s3cmd`% `` all work. Plurals are
  accepted (`containers%` resolves to the "Container" entry).
- The term must exist as a row in the glossary table. Multi-word terms work.
- A `%` after a digit (`40%`) is left alone. Write `\%` for a literal percent
  after a letter. Only an explicit `%` creates a link, so ordinary prose is safe.
- **Don't overuse it.** Tag a term at most once per page, and never in the very
  sentence that defines the term (that would be noise). Tagging the same term on
  a *different* page is good, especially where a reader arriving there may have
  forgotten what it means.
- When you introduce a new term worth defining, add it to `glossary.md`.

**Quizzes:** a ```` ```quiz ```` fenced block. `Q:` for the prompt, `- [ ]` /
`- [x]` for options, `>` for an explanation, `---` between questions.

**Diagrams:** ```` ```mermaid ````. **Math:** `$inline$` / `$$block$$` (KaTeX).

## My role

Default working mode: **I draft, you edit.** From your outline or brief I write
full pages or sections in the voice above; you refine. When a technical claim is
checkable (prices, quotas, VRAM, instance names, CLI flags), verify it rather
than assert from memory, and flag anything I could not confirm.

## Open items / TODOs

- **4th provider is undecided** (UpCloud vs a CSC offering vs other). Keep
  `4_*` pages generic and treat the specific provider as TBD until chosen.
- Build out the stubs: GCP/Azure/Finnish intros, EC2, Azure and Finnish data
  moves, and the `rclone` alternative in `1.1_Move_data.md`.

## Dev commands

- `bun run dev` - local dev server
- `bun run build` - production build (prerender)
- `bun run typecheck` - `tsc --noEmit`
- `bun run lint` / `bun run format` - ESLint / Prettier
