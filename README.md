# Sutro Skills

A collection of AI agent skills for working with [Sutro](https://withsutro.com) backends.

[Sutro](https://withsutro.com) is a no-ops backend infrastructure for building production-ready backends. You define a backend—including logic and security—and Sutro runs it, scales it, and migrates it without any infrastructure work.

For official documentation, tutorials, and getting started guides, visit: **https://docs.withsutro.com**

## Available Skills

### slang

SLang is the programming language for defining Sutro backends. It provides a simple but powerful way to define entire backends—including entities, logic, and security—without boilerplate or complex syntax.

This skill provides a comprehensive syntax reference for AI agents, covering:

- Entity (model) definitions with fields, types, and constraints
- Relations between models with cardinality
- Actions with parameters, body statements, and return types
- Triggers for HTTP endpoints, queues, and events
- Authentication and authorization rules

## Installation

Install a skill using the `skills` CLI:

```bash
npx skills add https://github.com/SutroOrg/sutro-skills --skill slang
```

## Usage

Once installed, the skill is automatically available to AI agents (Claude, etc.) when working in your project. The agent will have access to the full SLang language reference to help you write and debug `.slang` files.

## Learn More

- [Sutro Website](https://withsutro.com)
- [Documentation](https://docs.withsutro.com)
- [SLang Introduction](https://docs.withsutro.com/docs/SLang/introduction)
- [Quickstart Guide](https://docs.withsutro.com/docs/getting-started/slang-quickstart)

## License

MIT
