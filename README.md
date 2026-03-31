**[English](README.md)** | **[中文](README.zh-CN.md)**

# Claude Code Source Analysis

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/docs-DOCUMENTATION.md-green.svg)](DOCUMENTATION.md)
[![Language](https://img.shields.io/badge/language-English-brightgreen.svg)](README.md)
[![中文](https://img.shields.io/badge/语言-中文-orange.svg)](README.zh-CN.md)

> A comprehensive reverse-engineering analysis of Claude Code's internal architecture, modules, and design patterns.

---

## What Is This?

This project is a deep-dive technical analysis of [Claude Code](https://docs.anthropic.com/en/docs/claude-code), Anthropic's official AI-powered CLI for software engineering. It documents the internal source tree structure, module boundaries, tool/command inventories, state management, and architectural patterns — all reverse-engineered from the source code.

Whether you're building tools on top of Claude Code, contributing to similar projects, or simply curious about how a production-grade AI CLI is architected, this analysis gives you a detailed map of the internals.

## Why Does This Exist?

Claude Code is a sophisticated, real-world application that combines LLM orchestration, terminal UI rendering, plugin systems, multi-agent coordination, and more — all in a single TypeScript codebase. Understanding its architecture provides valuable insight into:

- How to design tool-based LLM agent systems
- Real-world patterns for React-in-terminal UIs (Ink)
- Permission and security models for AI code assistants
- Context window management and compaction strategies
- Multi-agent task orchestration

## What's Covered

The [full analysis](DOCUMENTATION.md) covers:

| Section | Description |
|---------|-------------|
| [Project Overview](DOCUMENTATION.md#1-project-overview) | High-level capabilities and feature summary |
| [Technology Stack](DOCUMENTATION.md#2-technology-stack) | Runtime, frameworks, and dependencies |
| [Directory Structure](DOCUMENTATION.md#3-directory-structure) | Complete source tree layout (37 subdirectories) |
| [Entry Points](DOCUMENTATION.md#4-entry-points) | Bootstrap, CLI, SDK, and MCP entry points |
| [Core Architecture](DOCUMENTATION.md#5-core-architecture) | Query engine, context builder, cost tracking |
| [Tool System](DOCUMENTATION.md#6-tool-system) | All 41 tools with categories and descriptions |
| [Command System](DOCUMENTATION.md#7-command-system) | All 101 slash commands organized by function |
| [State Management](DOCUMENTATION.md#8-state-management) | Zustand-style store and key state fields |
| [Task System](DOCUMENTATION.md#9-task-system) | Task types, lifecycle, and state management |
| [Services & Integrations](DOCUMENTATION.md#10-services--integrations) | API client, MCP, analytics, compaction, and more |
| [UI Layer](DOCUMENTATION.md#11-ui-layer) | React/Ink component architecture (130+ components) |
| [Utilities](DOCUMENTATION.md#12-utilities) | 300+ utility modules across all categories |
| [Special Modes](DOCUMENTATION.md#13-special-modes) | Bridge, Kairos, Coordinator, Voice, Plan, Vim modes |
| [Plugins & Skills](DOCUMENTATION.md#14-plugins--skills) | Plugin and skill extension systems |
| [Hooks & Extensibility](DOCUMENTATION.md#15-hooks--extensibility) | Hook schemas and React hooks |
| [File Statistics](DOCUMENTATION.md#16-file-statistics) | Codebase size and module counts |
| [Architectural Patterns](DOCUMENTATION.md#17-architectural-patterns) | Key design patterns and principles |

## Quick Start

Just read the documentation:

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/claude-code-analysis.git

# Open the analysis
open DOCUMENTATION.md
```

## Disclaimer

> **This is an unofficial, independent analysis.** Claude Code is a product of [Anthropic](https://www.anthropic.com/). This project is not affiliated with, endorsed by, or sponsored by Anthropic. All trademarks belong to their respective owners.

## License

This project is licensed under the [MIT License](LICENSE).

---

📖 **[Read the Full Analysis →](DOCUMENTATION.md)**

🇨🇳 **[中文版 →](README.zh-CN.md)**
