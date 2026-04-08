---
description: AI skills and MCP servers to help AI assistants integrate Self Pass
---

# AI Developer Tools

Self provides AI coding skills and MCP servers that give AI assistants deep context about the Self Protocol. These tools work with AI coding assistants (Claude Code, Cursor, Windsurf, Codex) as well as autonomous agent frameworks that support the Model Context Protocol.

## Self Protocol Skill (Claude Code)

The [`self-skill`](https://github.com/selfxyz/self-skill) is a Claude Code skill that gives Claude deep integration knowledge about Self Protocol. It includes quick start templates, integration patterns, critical gotchas, and deployed contract addresses.

### Setup

Clone the skill into your Claude Code skills directory:

```bash
git clone https://github.com/selfxyz/self-skill.git ~/.claude/skills/self-skill
```

The skill activates automatically when you mention Self Protocol, `SelfAppBuilder`, `SelfBackendVerifier`, `SelfVerificationRoot`, or related concepts.

### What it provides

* Complete frontend and backend code templates for Next.js integration
* Integration pattern guidance (off-chain, on-chain, deep linking)
* Critical gotchas (config matching, lowercase addresses, mock passport rules)
* Deployed contract addresses for Celo Mainnet and Sepolia
* Reference files for frontend SDK, backend SDK, and smart contract patterns

## Self Agent ID MCP Server

For AI agent identity (registration, verification, signing), see the separate [Self Agent ID MCP server](../agent-id/guides/mcp-user.md) documentation.
