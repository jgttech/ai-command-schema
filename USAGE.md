# Usage Guide

This guide explains how to use the AI Commands Schema to define custom AI commands.

## Quick Start

1. **Create a command file** (e.g., `my-command.yml`)
2. **Add schema reference** at the top:
   ```yaml
   # yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json
   ```
3. **Define your command** with at minimum:
   ```yaml
   command: :my-command
   prompt: |
     Your instructions here
   ```

## Field Reference

### Required Fields

#### `command`
- **Type:** string
- **Pattern:** Must start with `:` followed by alphanumeric, underscore, or hyphen
- **Purpose:** Unique identifier for the command
- **Examples:** `:format`, `:test`, `:deploy-prod`

#### `prompt`
- **Type:** string (multi-line supported)
- **Purpose:** The instructions the AI should follow when executing this command
- **Tips:** 
  - Be clear and specific
  - Use `%s` or similar placeholders for dynamic content
  - Reference other commands with their `:command` syntax

### Optional Fields

#### `description`
- **Type:** string
- **Purpose:** Human-readable explanation of what the command does
- **Use when:** You want to document the command's purpose

#### `examples`
- **Type:** array of strings
- **Purpose:** Show typical invocations of the command
- **Use when:** The command has arguments or multiple usage patterns
- **Example:**
  ```yaml
  examples:
    - ":format src/"
    - ":format src/main.py --verbose"
  ```

#### `pre`
- **Type:** array of strings or objects
- **Purpose:** Commands to execute BEFORE running this command's prompt
- **Execution:** Runs in array order
- **Use cases:** Validation, setup, preparation

**Simple format:**
```yaml
pre:
  - :validate
  - :backup
```

**Complex format with error handling:**
```yaml
pre:
  - command: :validate
    continue_on_failure: false       # Stop if this fails (default)
    on_failure: :log-error          # Run this command if validation fails
    on_success: :update-status      # Run this command if validation succeeds
  - :test  # Can mix simple and complex formats
```

**Fields for complex format:**
- `command` (required): The command to execute
- `continue_on_failure` (optional, default `false`): Whether to continue if this command fails
- `on_failure` (optional): Command to execute if this fails. AI validates this command exists.
- `on_success` (optional): Command to execute if this succeeds. AI validates this command exists.

**Important:** AI will validate that all referenced commands (including in `on_failure`/`on_success`) exist and inform you if they don't. If `on_failure` or `on_success` commands themselves fail, AI will report the failure clearly but won't attempt complex recovery.

#### `post`
- **Type:** array of strings or objects
- **Purpose:** Commands to execute AFTER running this command's prompt
- **Execution:** Runs in array order
- **Use cases:** Cleanup, saving, notifications

**Simple format:**
```yaml
post:
  - :save
  - :notify-team
```

**Complex format with error handling:**
```yaml
post:
  - command: :save
    continue_on_failure: true        # Continue even if save fails
    on_failure: :log-save-error
  - command: :notify
    on_success: :celebrate
```

Same field structure as `pre` - supports both simple strings and complex objects with error handling.

#### `argv`
- **Type:** object (argument definitions)
- **Purpose:** Define command-line style arguments
- **Structure:**
  ```yaml
  argv:
    argument-name:
      type: string|number|boolean|array|object  # Required
      format: positional|named|flag              # Optional, defaults to positional
      required: true|false                       # Optional, defaults to false
      default: value                             # Optional
      description: "Help text"                   # Optional
      validation:                                # Optional
        exists: true|false                       # For paths only
  ```

**Argument Types:**
- `string` - Text values
- `number` - Numeric values
- `boolean` - True/false values
- `array` - Lists of values
- `object` - Structured data

**Argument Formats:**
- `positional` (default) - Order-based: `:command value1 value2`
- `named` - Key-value: `:command --name=value`
- `flag` - Boolean: `:command --verbose`

**Validation:**
- `exists: true` - For path arguments, the file/directory must exist before execution

#### `output`
- **Type:** object
- **Purpose:** Specify expected output format and constraints
- **Structure:**
  ```yaml
  output:
    format: plain_text    # Optional, defaults to plain_text
    constraints:          # Optional, array of strings
      - "Constraint 1"
      - "Constraint 2"
  ```

**Available Formats:**
- Text: `plain_text`, `markdown`, `html`
- Data: `json`, `yaml`, `toml`, `xml`
- Code: `python`, `javascript`, `typescript`, `rust`, `go`, `zig`, `odin`, `elixir`, `bash`, `sql`, and many others

**Constraints:**
- Free-form strings describing output requirements
- Examples: "no AI signature", "under 80 characters", "use conventional commit format"

## Command Chaining

Commands can form workflows using `pre` and `post`:

**Simple workflow:**
```yaml
command: :deploy
pre:
  - :validate    # Runs first
  - :test        # Runs second
post:
  - :save        # Runs after deploy
  - :notify      # Runs last
prompt: |
  Deploy the application to production
```

**Execution order:** `:validate` → `:test` → deploy prompt → `:save` → `:notify`

**Advanced workflow with error handling:**
```yaml
command: :deploy
pre:
  - command: :validate
    continue_on_failure: false
    on_failure: :notify-failure
  - command: :backup
    continue_on_failure: true  # Continue even if backup fails
  - :test
post:
  - command: :update-docs
    on_success: :notify-success
    on_failure: :rollback
prompt: |
  Deploy the application to production
```

**Execution flow:**
1. Run `:validate` → if fails, run `:notify-failure`, then **stop**
2. Run `:backup` → if fails, **continue anyway**
3. Run `:test` → if fails, **stop**
4. Run main deploy prompt
5. Run `:update-docs` → if succeeds, run `:notify-success`; if fails, run `:rollback`

**Key principles:**
- AI validates all referenced commands exist before execution
- If `on_failure`/`on_success` commands fail, AI reports the issue but doesn't attempt recovery
- Default is `continue_on_failure: false` (fail-fast is safer)
- You control the flow logic; AI just executes and reports

## Placeholders

Use placeholders in prompts for dynamic content:

```yaml
command: :commit
prompt: |
  Generate a commit message for these changes:
  %s
```

Common placeholder styles:
- `%s` - Generic placeholder
- `{variable}` - Named placeholder
- `$ARG` - Environment-style

## Best Practices

1. **Be explicit** - Use clear, descriptive command names
2. **Document intent** - Add `description` field to explain purpose
3. **Show examples** - Include `examples` for complex commands
4. **Use constraints** - Be specific about output requirements
5. **Chain wisely** - Use pre/postcommands for clear workflows
6. **Keep prompts focused** - Each command should have a single, clear purpose

## IDE Support

### VS Code

1. Install [YAML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)
2. Add schema reference to your YAML files (see Quick Start)
3. Get autocomplete, validation, and hover documentation

### Other Editors

Any editor supporting JSON Schema with YAML can use this schema via the `yaml-language-server` comment directive.

## Validation

### Command Line

Using [ajv-cli](https://github.com/ajv-validator/ajv-cli):

```bash
npm install -g ajv-cli ajv-formats
ajv validate -s schema.json -d "*.yml"
```

### Python

```python
import yaml
import json
import jsonschema
from urllib.request import urlopen

# Load schema from GitHub
schema_url = "https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json"
with urlopen(schema_url) as response:
    schema = json.loads(response.read())

# Load and validate your command
with open('your-command.yml') as f:
    command = yaml.safe_load(f)

jsonschema.validate(command, schema)
print("✓ Valid command definition")
```

## Troubleshooting

**Schema not found:**
- Ensure you're using the correct URL in the `yaml-language-server` comment
- Check that your IDE's YAML extension is installed and enabled

**Validation errors:**
- Required fields: `command` and `prompt` must be present
- Command pattern: Must start with `:` followed by valid characters
- Referenced commands: In `precommands`/`postcommands`, all entries must follow `:command` pattern

**Command not working:**
- Verify your AI tool is configured to recognize this schema
- Check that command names are unique within your command set
- Ensure referenced commands (in pre/postcommands) exist
