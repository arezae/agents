# Agent Playbook: token-efficient Claude agent systems

Distilled rules for building agent systems that spend fewer tokens without losing quality.
One rule per line, each with a source and date. Maintained by the weekly update task
(see `skills/agent-playbook/skill.md`). Max 150 lines. Delete outdated rules; do not append history.
Tags: [multi] applies mainly to multi-agent designs. [secondary] source is a summary, verify before relying on it.

## Core principles

- Treat context as a finite attention budget; aim for the smallest set of high-signal tokens that gets the result. [Anthropic, 2025-09-29](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- Prefer just-in-time retrieval (the agent fetches with search, glob, grep) over front-loading everything. [Anthropic, 2025-09-29](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- For long tasks, keep notes in files outside the window and let sub-agents return short summaries. [Anthropic, 2025-09-29](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- Do the simplest thing that works; smarter models need less prescriptive scaffolding. [Anthropic, 2025-09-29](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

## Single agent or many

- Start with one well-designed agent; add agents only for context pollution, real parallelism, or specialization. [Anthropic, 2026-01-23](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them)
- [multi] Budget 3-10x the tokens of a single agent for the same task; overhead comes from duplicated context, coordination, and handoff summaries. [Anthropic, 2026-01-23](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them)
- [multi] Anthropic's research system used about 15x the tokens of a chat (single agents about 4x); justify it only for high-value, breadth-first work. [Anthropic, 2025](https://www.anthropic.com/engineering/multi-agent-research-system)
- [multi] Token usage explained about 80% of quality variance in their browsing eval, so track quality per token spent rather than adding tokens blindly. [Anthropic, 2025](https://www.anthropic.com/engineering/multi-agent-research-system)
- [multi] Tasks where all agents must share the same context or have many dependencies fit multi-agent designs poorly. [Anthropic, 2025](https://www.anthropic.com/engineering/multi-agent-research-system)
- [multi] Keep writing single-threaded; parallel writers make conflicting implicit choices. [Cognition via Latent Space, 2026-04-22, secondary](https://latent.space/wiki/walden-yan-7c572c23/)
- [multi] A reviewer that starts with clean context (not the coder's reasoning) worked best as a second pair of eyes. [Cognition via Latent Space, 2026-04-22, secondary](https://latent.space/wiki/walden-yan-7c572c23/)
- With 15-20+ tools, try tool search before splitting into more agents. [Anthropic, 2026-01-23](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them)
- [multi] Claude Code agent teams use roughly 7x the tokens when teammates run in plan mode; keep teams small, use Sonnet teammates, keep spawn prompts short, shut teammates down when done. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)

## Long-running work

- Run a one-time initializer: feature checklist, progress log, startup script, first commit; later sessions start from these. [Anthropic, 2025-11-26](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- Track features in JSON where agents may only flip status; models corrupt JSON less than Markdown. [Anthropic, 2025-11-26](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- One feature per session; end each session in a committed, documented, working state. [Anthropic, 2025-11-26](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- Test end to end like a user; unit tests alone lead agents to declare features done too early. [Anthropic, 2025-11-26](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- Separate the generator from a skeptical evaluator; agents rate their own output too generously. [Anthropic, 2026-03-24](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- Use an evaluator only for tasks beyond the model's solo ability; inside it, it is pure overhead. [Anthropic, 2026-03-24](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- Re-test every harness component when the model changes and remove what is no longer needed. [Anthropic, 2026-03-24](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- Know the price of a full harness: in their example about $200 versus $9 for a solo run, with much better output; spend only when the task needs it. [Anthropic, 2026-03-24](https://www.anthropic.com/engineering/harness-design-long-running-apps)

## Claude Code token levers

- Run /clear between unrelated tasks (/rename first if you want to resume). [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Give compaction instructions (/compact with a focus, or a "Compact instructions" block in CLAUDE.md). [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Default to Sonnet; keep Opus for hard architecture or multi-step reasoning; use Haiku for simple sub-agents. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Lower /effort for simple tasks; thinking cannot be switched off on the current models and adaptive models ignore fixed budgets. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Keep CLAUDE.md under about 200 lines; move workflow-specific instructions into skills, which load on demand. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Prefer CLI tools (gh, aws, gcloud) to MCP servers; disable MCP servers you are not using (tool definitions are deferred by default now). [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Preprocess verbose output with hooks (for example show only failing test lines) before Claude reads it. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Send noisy work (tests, doc fetches, logs) to sub-agents; their requests still count against usage, so pick a smaller model for them. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Write specific prompts (file, function, outcome) instead of "improve this codebase". [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Use plan mode for complex work, stop early when it drifts (Esc, /rewind), give verification targets, test incrementally. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Measure before tuning: /usage shows usage attributed to skills, sub-agents, plugins and MCP servers with recommendations; /context shows what fills the window. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Protect the prompt cache: a message after a break longer than the cache lifetime reprocesses the full context. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Idle loops, scheduled tasks, goal check-ins and cross-session messages each resend the full context; stop or hold the ones you do not need. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Compacting a large context is itself a large request; when you do not need continuity, /clear is cheaper. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)

## Context hygiene

- Quality degrades as input length grows across major models; curate, compress and isolate context instead of filling the window. [secondary] [Digital Applied summary of Chroma research, 2026-05-26](https://www.digitalapplied.com/blog/context-engineering-agent-reliability-playbook-2026)

## Knowledge and memory

- For personal-scale research, keep an LLM-maintained, interlinked Markdown wiki (raw sources in, wiki out, periodic lint) instead of rebuilding retrieval each time. [secondary] [Karpathy LLM Wiki gist, 2026-04](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

## Skills

- Skills are folders with a SKILL.md; only the description loads at startup and the body loads when invoked, so put rarely used knowledge there. [Claude Code docs, checked 2026-11-06](https://code.claude.com/docs/en/costs)
- Review community skills before installing them, as you would any open-source code. [secondary] [Verdent guide to skills repositories](https://www.verdent.ai/ja/guides/claude-skills-github-repository)

## Open questions

- Which of the sub-agent and effort tips change with Sonnet 5.5 and Opus 5.5? Re-test and update.
- Which rules have measured token savings in a real multi-agent pipeline? Add numbers when available.
