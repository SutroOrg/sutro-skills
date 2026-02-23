# Sutro Skills

A collection of AI agent skills for working with Sutro technologies.

## Available Skills

### slang

A comprehensive language reference for **SLang**, the domain-specific language for defining Sutro application backends. The skill covers:

- Entity (model) definitions with fields, types, and constraints
- Relations between models with cardinality
- Actions with parameters, body statements, and return types
- Triggers for HTTP endpoints, queues, and events
- Authentication and authorization rules
- Compilation to SCode JSON

## Installation

Install a skill using the `skills` CLI:

```bash
npx skills add https://github.com/SutroOrg/sutro-skills --skill slang
```

Or install all skills from this repository:

```bash
npx skills add https://github.com/SutroOrg/sutro-skills
```

## Usage

Once installed, the skill is automatically available to AI agents (Claude, etc.) when working in your project. The agent will have access to the full SLang language reference to help you write and debug `.slang` files.

## License

MIT
