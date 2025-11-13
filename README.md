# AI Command Schema

[![GitHub](https://img.shields.io/badge/github-jgttech%2Fai--command--schema-blue)](https://github.com/jgttech/ai-command-schema)
[![Schema](https://img.shields.io/badge/schema-JSON%20Schema%20Draft%207-green)](https://raw.githubusercontent.com/jgttech/ai-command-schema/main/ai-command-schema.json)

A JSON Schema for defining structured AI commands using YAML. This schema helps you create clear, validated command definitions that AI assistants can easily understand and execute.

## Documentation

- **[USAGE.md](USAGE.md)** - Complete usage guide and field reference
- **[EXAMPLES.md](EXAMPLES.md)** - Real-world command examples
- **[ai-command-schema.json](ai-command-schema.json)** - The schema itself

## Quick Example

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/ai-command-schema.json

command: :format
description: Format code and commit changes
postcommands:
  - :save
prompt: |
  Review all code in the repository and apply formatting rules.
  Ensure consistent style and proper structure.
```

## Why Use This Schema?

**AI-First Design:**
This schema is designed with AI-first principles to maximize clarity and minimize ambiguity. Commands are structured to be immediately understandable by AI assistants.

**Key Benefits:**
- ✅ **Validation** - Catch errors before execution with JSON Schema validation
- ✅ **IDE Support** - Get autocomplete and inline documentation in VS Code and other editors
- ✅ **Clear Structure** - Explicit fields for arguments, output format, and command chaining
- ✅ **Self-Documenting** - Commands include descriptions, examples, and constraints
- ✅ **Reusable** - Build libraries of commands for common tasks

## Design Principles

1. **Clarity over completeness** - Simple, clear field names that AI can easily understand
2. **Minimal required fields** - Only `command` and `prompt` are required
3. **Simple validation** - Focus on what AI needs to know, not exhaustive validation rules
4. **Natural language constraints** - Output constraints are free-form strings, not rigid rules
5. **Explicit over implicit** - Defaults are clearly documented
6. **Human-readable** - Field names and values are self-explanatory

## Installation

### Using the Schema

Add this line to the top of your YAML command files:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/ai-command-schema.json
```

### IDE Setup

**VS Code:**
1. Install the [YAML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)
2. Add the schema reference to your YAML files
3. Get automatic validation and autocomplete

## Command Structure

```yaml
command: :command-name      # Required: Command identifier
description: What this does # Optional: Human-readable description
examples:                   # Optional: Usage examples
  - ":command-name arg"
precommands:                # Optional: Run before main command
  - :validate
argv:                       # Optional: Argument definitions
  name:
    type: string
    format: positional
    required: false
output:                     # Optional: Output specifications  
  format: plain_text        # Defaults to plain_text
  constraints:
    - "Specific requirements"
postcommands:               # Optional: Run after main command
  - :save
prompt: |                   # Required: Instructions for AI
  Your command instructions here.
```

**For complete field documentation, see [USAGE.md](USAGE.md).**

## About the Colon Prefix

Commands use the `:command` pattern (colon followed by the command name). This convention:

- **Visually distinct** - Immediately signals "this is a command"
- **Familiar precedent** - Used in vim (`:w`, `:q`), emacs, and IRC
- **Conflict-free** - Doesn't interfere with file paths, variables, or natural language
- **Easy to parse** - Simple pattern for both humans and AI

Alternative prefixes (`@`, `#`, `/`, `!`, `$`) either conflict with existing conventions or are less clear.

## Features

- **Command Chaining** - Use `precommands` and `postcommands` for clear workflows
- **Flexible Arguments** - Support for positional, named, and flag-style arguments
- **Output Control** - Specify format (plain_text, markdown, json, python, etc.) and constraints
- **Validation Hints** - Tell AI when files must exist before execution
- **Usage Examples** - Show typical invocations inline
- **Strict Validation** - No additional properties allowed; schema catches typos

## Real-World Examples

See [EXAMPLES.md](EXAMPLES.md) for complete examples including:
- Code review and formatting
- Test generation
- Commit message creation  
- API documentation
- Deployment workflows
- And many more!

## Contributing

Contributions are welcome! Please feel free to:
- Submit issues for bugs or feature requests
- Propose new output formats or validation options
- Share your command definitions as examples
- Improve documentation

## License

This schema is released under the MIT License. See the repository for details.

## Links

- **Repository:** https://github.com/jgttech/ai-command-schema
- **Schema URL:** https://raw.githubusercontent.com/jgttech/ai-command-schema/main/ai-command-schema.json
- **Usage Guide:** [USAGE.md](USAGE.md)
- **Examples:** [EXAMPLES.md](EXAMPLES.md)

