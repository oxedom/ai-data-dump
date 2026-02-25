# ai-data-dump

> "LLMs are calculator's for words" — Simon Willison

A Slidev presentation exploring practical workflows for working with AI coding agents, covering context engineering, security, tooling, and the ERPI methodology.

## Overview

This presentation distills practical lessons learned from building software with AI coding agents. It focuses on treating LLMs as stateless calculators for code generation while maintaining human oversight and architectural control.

## Topics Covered

- **ERPI Workflow** — Epic, Research, Plan, Implement methodology
- **Context Engineering** — Managing token budgets and layered CLAUDE.md files
- **Security** — Sandboxing, prompt injection, and network access considerations
- **Git Worktrees** — Parallel development with multiple coding agents
- **Tooling** — MCP integrations (Context7, WebMCP, Playwright, Figma, GitHub CLI)
- **Agent Control Flow** — The "Ralph Wiggum Loop" for autonomous task execution

## Live Presentation

The presentation is automatically deployed to GitHub Pages on every push to main:

**[View Live Presentation](https://YOUR-USERNAME.github.io/ai-data-dump/)**

_(Update the URL above with your GitHub username)_

## Local Development

### Prerequisites

- Node.js 20+
- pnpm 9+

### Setup

```bash
# Install dependencies
cd slidev-presentation
pnpm install

# Start development server
pnpm dev
```

The presentation will open automatically at `http://localhost:3030`

### Build

```bash
# Build for production
pnpm build

# Export to PDF (requires Playwright browsers)
pnpm export
```

## Project Structure

```
ai-data-dump/
├── .claude/              # Claude Code configuration
├── .github/
│   └── workflows/
│       └── deploy.yml    # GitHub Pages deployment
└── slidev-presentation/
    ├── slides.md         # Main presentation content
    ├── package.json      # Dependencies
    └── README.md         # Slidev-specific docs
```

## Technology

- [Slidev](https://sli.dev/) - Presentation slides for developers
- [Vue 3](https://vuejs.org/) - Frontend framework
- [Vite](https://vitejs.dev/) - Build tool
- GitHub Pages - Hosting

## Key Concepts

### Context Engineering

Managing your AI's context window like RAM:
- Aim for 40-50% max usage
- Layer CLAUDE.md files by directory
- Use `/context` to visualize, `/clear` to reset
- Protect against stale memory files

### ERPI Workflow

1. **Epic** — Define the problem clearly, avoid ambiguity
2. **Research** — Explore codebase, understand patterns
3. **Plan** — Design before implementation (specs are contracts!)
4. **Implement** — Execute with agent assistance

### Security First

- Run coding agents in isolated environments
- Be aware of prompt injection attacks
- Monitor network access in generated code
- Remember: dev machines are attack surfaces

## Contributing

Feel free to open issues or submit PRs for improvements to the presentation content.

## License

MIT

## Author

Sam Brink
- Web: [www.sam-brink.com](https://www.sam-brink.com)
- Title: Builder, Photographer
