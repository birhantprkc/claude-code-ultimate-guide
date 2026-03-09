# Claude Code Ultimate Guide

<p align="center">
  <a href="https://florianbruniaux.github.io/claude-code-ultimate-guide-landing/"><img src="https://img.shields.io/badge/🌐_Interactive_Guide-Visit_Website-ff6b35?style=for-the-badge&logoColor=white" alt="Website"/></a>
</p>

<p align="center">
  <a href="https://github.com/FlorianBruniaux/claude-code-ultimate-guide/stargazers"><img src="https://img.shields.io/github/stars/FlorianBruniaux/claude-code-ultimate-guide?style=for-the-badge" alt="Stars"/></a>
  <a href="./CHANGELOG.md"><img src="https://img.shields.io/badge/Updated-Mar_9,_2026_·_v3.32.2-brightgreen?style=for-the-badge" alt="Last Update"/></a>
  <a href="./quiz/"><img src="https://img.shields.io/badge/Quiz-271_questions-orange?style=for-the-badge" alt="Quiz"/></a>
  <a href="./examples/"><img src="https://img.shields.io/badge/Templates-222-green?style=for-the-badge" alt="Templates"/></a>
  <a href="./guide/security-hardening.md"><img src="https://img.shields.io/badge/🛡️_Threat_DB-15_vulnerabilities_·_655_malicious_skills-red?style=for-the-badge" alt="Threat Database"/></a>
  <a href="./mcp-server/"><img src="https://img.shields.io/badge/MCP_Server-npx_ready-blueviolet?style=for-the-badge" alt="MCP Server"/></a>
</p>

<p align="center">
  <a href="https://github.com/hesreallyhim/awesome-claude-code"><img src="https://awesome.re/mentioned-badge-flat.svg" alt="Mentioned in Awesome Claude Code"/></a>
  <a href="https://creativecommons.org/licenses/by-sa/4.0/"><img src="https://img.shields.io/badge/License-CC%20BY--SA%204.0-blue.svg" alt="License: CC BY-SA 4.0"/></a>
  <a href="https://skills.palebluedot.live/owner/FlorianBruniaux"><img src="https://img.shields.io/badge/SkillHub-9_skills-8b5cf6.svg" alt="SkillHub Skills"/></a>
</p>

> **6 months of daily practice** distilled into a guide that teaches you the WHY, not just the what. From core concepts to production security, you learn to design your own agentic workflows instead of copy-pasting configs.

> **If this guide helps you, [give it a star ⭐](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/stargazers)** — it helps others discover it too.

---

## Choose Your Path

| Who you are | Your guide |
|---|---|
| 🏗️ **Tech Lead / Engineering Manager** | [Deploying Claude Code across your team →](docs/for-tech-leads.md) |
| 📊 **CTO / Decision Maker** | [ROI, security posture, team adoption →](docs/for-cto.md) |
| 💼 **CIO / CEO** | [Budget, risk, what to ask your tech team (3 min) →](docs/for-cio-ceo.md) |
| 🎨 **Product Manager / Designer** | [Vibe coding, working with AI-assisted dev teams →](docs/for-product-managers.md) |
| ✍️ **Writer / Ops / Manager** | [Claude Cowork Guide (separate repo) →](https://github.com/FlorianBruniaux/claude-cowork-guide) |
| 👨‍💻 **Developer (all levels)** | You're in the right place — read on ↓ |
| 🧭 **Career pivot / new AI role** | [AI Roles & Career Paths →](guide/ai-roles.md) |

---

## The Golden Rules

1. **Verify trust** — Claude Code can generate 1.75× more logic errors than human-written code ([ACM 2025](https://dl.acm.org/doi/10.1145/3716848)). Test everything. Use `/insights` and peer review for production paths.
2. **Vet your MCPs** — 15 vulnerabilities tracked, 655 malicious skills in supply chain. Run the 5-min audit checklist before trusting any MCP server.
3. **Manage context pressure** — At 70% context, precision drops. At 90%+, responses become erratic. Run `/compact` at 70%, `/clear` at 90%.
4. **Start simple, scale when proven** — Basic CLAUDE.md first. Add agents and skills only after 2 weeks of proven need in production.
5. **Methodologies matter more with AI** — TDD/SDD/BDD are not optional. AI accelerates bad code as fast as good code.

> Full reference: [Cheat Sheet](./guide/cheatsheet.md) · [Security Guide](./guide/security-hardening.md)

---

## Quick Start

```bash
npm install -g @anthropic-ai/claude-code
cd your-project
claude
```

Three commands that matter on day one: `/help` (discover features), `/init` (create CLAUDE.md), `/compact` (free context space).

**Personalized onboarding** (no setup needed):
```bash
claude "Fetch and follow the onboarding instructions from: https://raw.githubusercontent.com/FlorianBruniaux/claude-code-ultimate-guide/main/tools/onboarding-prompt.md"
```

**MCP Server** — use this guide from any Claude Code session without cloning:
```json
{
  "mcpServers": {
    "claude-code-guide": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "claude-code-ultimate-guide-mcp"]
    }
  }
}
```

→ [MCP Server README](./mcp-server/README.md) — 12 tools, 8 slash commands `/ccguide:*`, Haiku agent

---

## What's Inside

| Section | What you get |
|---------|-------------|
| [Ultimate Guide](./guide/ultimate-guide.md) | 22,000+ lines — the complete reference across 10 sections |
| [Cheat Sheet](./guide/cheatsheet.md) | 1-page printable daily reference |
| [Templates](./examples/) | 222 production-ready templates — agents, commands, hooks, skills |
| [Whitepapers](./whitepapers/) | 11 focused whitepapers (FR + EN) — architecture, security, privacy, teams |
| [Quiz](./quiz/) | 271 questions, 4 profiles, instant feedback with doc links |
| [Visual Diagrams](./guide/diagrams/) | 41 Mermaid diagrams — master loop, multi-agent topologies, security threats |
| [Security DB](./examples/commands/resources/threat-db.yaml) | 15 vulnerabilities + 655 malicious skills tracked |
| [Methodologies](./guide/methodologies.md) | TDD, SDD, BDD, GSD — full workflow guides with rationale |
| [Architecture](./guide/architecture.md) | How Claude Code works internally (context flow, tool orchestration) |
| [Claude Code Releases](./guide/claude-code-releases.md) | Condensed official changelog with highlights |
| [Resource Evaluations](./docs/resource-evaluations/) | 84 evidence-based assessments (5-point scoring) |
| [AI Roles & Career Paths](./guide/ai-roles.md) | 13 roles mapped — from Prompt Engineer to Harness Engineer, with career matrix and salary benchmarks |

---

## Whitepapers

11 whitepapers in French and English, covering: foundations, prompting, customization, security in production, architecture, team deployment, privacy & compliance, reference guide, agent teams, learning with AI, and budget optimization.

> **Coming soon** — currently in private access. Public release planned.

→ [Browse whitepapers/](./whitepapers/) · [Recap Cards (57 A4 reference cards)](./whitepapers/recap-cards/)

---

## Ecosystem

**cc-copilot-bridge** routes Claude Code through GitHub Copilot Pro+ for flat-rate access — [GitHub](https://github.com/FlorianBruniaux/cc-copilot-bridge)

**Complementary tools**: [everything-claude-code](https://github.com/affaan-m/everything-claude-code) (production configs) · [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) (curation) · [skills.sh](https://skills.sh/) (marketplace)

**Community**: 🇫🇷 [Dev With AI](https://www.devw.ai/) — 1500+ devs on Slack, meetups in Paris, Bordeaux, Lyon

---

## Contributing & License

**Contributing** — Corrections, quiz questions, methodologies, resource evaluations all welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md) · [GitHub Issues](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/issues) · [Discussions](../../discussions)

**Guide License**: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · **Templates**: [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/) (copy-paste freely, no attribution needed)

**Author**: [Florian Bruniaux](https://florian.bruniaux.com) — Founding Engineer @ [Méthode Aristote](https://methode-aristote.fr) · [GitHub](https://github.com/FlorianBruniaux) · [LinkedIn](https://www.linkedin.com/in/florian-bruniaux-43408b83/)

---

*Version 3.32.2 | Updated daily · Mar 9, 2026 | [cc.bruniaux.com](https://cc.bruniaux.com)*

---

## FAQ

**What is the Claude Code Ultimate Guide?**
The most comprehensive community guide for Claude Code (Anthropic's CLI for AI-assisted development). 22,760+ lines of documentation, 238 production-ready templates for agents, hooks, skills, and commands. Free and open-source (CC BY-SA 4.0).

**How many templates are included?**
238 production-ready templates across: agents (backend-architect, security-auditor, code-reviewer), commands (/pr, /commit, /generate-tests), hooks (bash + PowerShell: secrets-scanner, dangerous-actions-blocker), and skills (pdf-generator, tdd-workflow, landing-page-generator).

**Is this guide free?**
Yes. The guide is CC BY-SA 4.0 (share with attribution). Templates are CC0 (copy-paste freely, no attribution). No paywall, no login.

**How is this different from the official Anthropic docs?**
The official docs explain what Claude Code does. This guide teaches you how to use it effectively in production: security hardening, custom agents, event hooks, CI/CD integration, multi-agent workflows, and real-world patterns built from 6+ months of daily practice.

**What topics does it cover?**
Claude Code tutorial, anthropic CLI, AI coding assistant, claude code MCP servers, custom agents, claude code hooks, reusable skills, agentic coding workflows, AI pair programming, TDD/SDD/BDD with AI, context window management, data privacy with Anthropic, claude code architecture, team adoption, security hardening.
