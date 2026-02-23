# Sutro Skills

A collection of AI agent skills for working with [Sutro](https://withsutro.com) — the no-ops backend infrastructure for building production-ready backends.

## Available Skills

### slang

A comprehensive language reference for **SLang**, the programming language for defining Sutro backends. The skill covers:

- Entity (model) definitions with fields, types, and constraints
- Relations between models with cardinality
- Actions with parameters, body statements, and return types
- Triggers for HTTP endpoints, queues, and events
- Authentication and authorization rules

For official documentation and tutorials, visit: [docs.withsutro.com](https://docs.withsutro.com/docs/SLang/introduction)

## Installation

Install a skill using the `skills` CLI:

```bash
npx skills add https://github.com/SutroOrg/sutro-skills --skill slang
```

## Usage

Once installed, the skill is automatically available to AI agents (Claude, etc.) when working in your project. The agent will have access to the full SLang language reference to help you write and debug `.slang` files.

## Learn More

- [Sutro Website](https://withsutro.com)
- [Sutro Documentation](https://docs.withsutro.com)
- [SLang Introduction](https://docs.withsutro.com/docs/SLang/introduction)
- [Quickstart Guide](https://docs.withsutro.com/docs/getting-started/slang-quickstart)
