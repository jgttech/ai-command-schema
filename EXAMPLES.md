# Examples

Real-world examples of AI command definitions using this schema.

## Table of Contents

- [Basic Commands](#basic-commands)
- [Commands with Arguments](#commands-with-arguments)
- [Commands with Output Specifications](#commands-with-output-specifications)
- [Command Chaining](#command-chaining)
- [Complex Workflows](#complex-workflows)

## Basic Commands

### Simple Text Transformation

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :summarize
description: Summarize the provided text
prompt: |
  Provide a concise summary of the following text in 2-3 sentences:
  
  %s
```

### Code Review

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :review
description: Review code for quality and best practices
output:
  format: markdown
  constraints:
    - "Use bullet points for findings"
    - "Categorize as: Critical, Warning, Suggestion"
prompt: |
  Review the following code for:
  - Security vulnerabilities
  - Performance issues
  - Code quality and best practices
  - Potential bugs
  
  Provide constructive feedback with specific examples.
```

## Commands with Arguments

### File Formatter

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :align
description: Align code with project standards
examples:
  - ":align src/main.py"
  - ":align lib/"
argv:
  path:
    type: string
    format: positional
    required: true
    description: Path to file or directory to align
    validation:
      exists: true
prompt: |
  Review and align the code at the following path with established project rules:
  
  Path: %s
  
  Ensure all code follows expected guidance, structure, and format.
  If something is out of place, refactor to ensure alignment.
```

### Test Generator

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :test
description: Generate unit tests for code
examples:
  - ":test src/utils.py"
  - ":test src/api.ts --coverage"
argv:
  path:
    type: string
    format: positional
    required: true
    description: Path to source file
    validation:
      exists: true
  coverage:
    type: boolean
    format: flag
    default: false
    description: Include coverage annotations
output:
  format: python  # or javascript, depending on input
  constraints:
    - "Use pytest framework for Python"
    - "Use Jest framework for JavaScript"
    - "Aim for 80%+ code coverage"
    - "Include edge cases and error scenarios"
prompt: |
  Generate comprehensive unit tests for the code at: %s
  
  Requirements:
  - Test all public functions/methods
  - Include happy path and edge cases
  - Test error handling
  - Use appropriate mocking where needed
  %s
```

## Commands with Output Specifications

### Commit Message Generator

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :save
description: Generate conventional commit message
output:
  format: plain_text
  constraints:
    - "Do NOT include an AI signature or mark of any kind"
    - "Use conventional commit format (feat:, fix:, docs:, refactor:, etc.)"
    - "First line should be under 80 characters"
    - "Be specific and descriptive"
    - "Focus on the what and why, not the how"
    - "Output ONLY the commit message, no explanations"
prompt: |
  Generate a concise, conventional commit message for these changes.
  
  Guidelines:
  - Use conventional commit format (feat:, fix:, docs:, refactor:, etc.)
  - First line under 80 characters
  - Be specific and descriptive
  - Focus on "what" and "why", not "how"
  - Output ONLY the commit message
  
  Changes:
  %s
```

### API Documentation

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :document-api
description: Generate API documentation
output:
  format: markdown
  constraints:
    - "Use OpenAPI-style structure"
    - "Include request/response examples"
    - "Document all status codes"
    - "Include authentication details"
prompt: |
  Generate comprehensive API documentation for the following endpoint:
  
  %s
  
  Include:
  - Endpoint description and purpose
  - HTTP method and path
  - Request parameters (path, query, body)
  - Response format and status codes
  - Example requests and responses
  - Authentication requirements
  - Rate limiting information
```

## Command Chaining

### Simple Command Chaining

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :format
description: Format repository code and commit changes
post:
  - :save
prompt: |
  Review all code in the repository and apply formatting rules:
  
  - Ensure consistent indentation
  - Fix trailing whitespace
  - Organize imports
  - Apply project style guidelines
  - Ensure proper code structure
  
  After formatting, generate a commit message.
```

### Advanced Error Handling

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :deploy-safe
description: Deploy with comprehensive error handling
examples:
  - ":deploy-safe staging"
  - ":deploy-safe production"
pre:
  - command: :validate
    continue_on_failure: false
    on_failure: :notify-failure
    on_success: :log-validation-success
  - command: :backup
    continue_on_failure: true  # Continue even if backup fails
    on_failure: :log-backup-warning
  - :test  # Simple format - fails fast
argv:
  environment:
    type: string
    format: positional
    required: true
    description: Target environment
post:
  - command: :update-docs
    on_success: :notify-success
    on_failure: :rollback
  - :cleanup
prompt: |
  Deploy the application to %s environment.
  
  Steps:
  1. Build the application
  2. Run database migrations if needed
  3. Deploy to target environment
  4. Verify deployment health
```

### Validate, Deploy, Notify

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :deploy
description: Deploy application with validation and notification
examples:
  - ":deploy staging"
  - ":deploy production"
pre:
  - :validate
  - :test
argv:
  environment:
    type: string
    format: positional
    required: true
    description: Target environment (staging, production)
post:
  - :save
  - :notify
prompt: |
  Deploy the application to %s environment.
  
  Steps:
  1. Build the application
  2. Run database migrations if needed
  3. Deploy to target environment
  4. Verify deployment health
  5. Update deployment documentation
```

## Complex Workflows

### Full Code Review Pipeline

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :review-pr
description: Complete pull request review workflow
examples:
  - ":review-pr feature/user-auth"
pre:
  - :lint
  - :test
  - :security-scan
argv:
  branch:
    type: string
    format: positional
    required: true
    description: Branch name to review
output:
  format: markdown
  constraints:
    - "Use PR review template format"
    - "Include approval/changes requested recommendation"
    - "Categorize feedback by severity"
post:
  - :update-pr-status
prompt: |
  Perform a comprehensive code review for branch: %s
  
  Review areas:
  1. Code quality and style
  2. Test coverage
  3. Documentation
  4. Performance implications
  5. Security considerations
  6. Breaking changes
  
  Provide:
  - Summary of changes
  - Detailed feedback by file
  - Recommendation (Approve/Request Changes/Comment)
  - Checklist of items to address
```

### Database Migration Generator

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :migrate
description: Generate database migration from schema changes
examples:
  - ":migrate add-user-roles"
  - ":migrate update-indexes --rollback"
argv:
  description:
    type: string
    format: positional
    required: true
    description: Migration description
  rollback:
    type: boolean
    format: flag
    default: false
    description: Include rollback script
pre:
  - :validate-schema
output:
  format: sql
  constraints:
    - "Include both up and down migrations"
    - "Add comments explaining complex operations"
    - "Use transactions where appropriate"
    - "Include safety checks"
post:
  - :save
prompt: |
  Generate a database migration for: %s
  
  Requirements:
  - Analyze current schema
  - Generate migration SQL
  - Include data transformations if needed
  - Add appropriate indexes
  %s
  
  Ensure:
  - Backward compatibility where possible
  - Safe rollback procedure
  - Performance optimization
  - Data integrity constraints
```

### Documentation Generator

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :docs
description: Generate comprehensive project documentation
pre:
  - :analyze-codebase
output:
  format: markdown
  constraints:
    - "Follow project documentation structure"
    - "Include code examples"
    - "Add table of contents"
    - "Use clear headings and sections"
post:
  - :save
prompt: |
  Generate comprehensive documentation for the project:
  
  Include:
  1. Project Overview
     - Purpose and goals
     - Key features
     - Architecture overview
  
  2. Getting Started
     - Prerequisites
     - Installation steps
     - Quick start guide
  
  3. Usage Guide
     - Core concepts
     - Common workflows
     - Code examples
  
  4. API Reference
     - Public APIs
     - Parameters and returns
     - Usage examples
  
  5. Development
     - Setup development environment
     - Running tests
     - Contributing guidelines
  
  6. Troubleshooting
     - Common issues and solutions
     - FAQ
```

## Language-Specific Examples

### Python Linter

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :lint-python
description: Lint Python code using project standards
argv:
  path:
    type: string
    format: positional
    required: true
    validation:
      exists: true
output:
  format: python
  constraints:
    - "Follow PEP 8 style guide"
    - "Use type hints"
    - "Maximum line length 88 characters"
    - "Use docstrings for all public functions"
prompt: |
  Lint and fix Python code at: %s
  
  Apply:
  - PEP 8 style guidelines
  - Type hints where missing
  - Proper docstrings
  - Import organization
  - Remove unused imports/variables
```

### Rust Error Handler

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/jgttech/ai-command-schema/main/schema.json

command: :rust-errors
description: Improve Rust error handling
argv:
  path:
    type: string
    format: positional
    required: true
    validation:
      exists: true
output:
  format: rust
  constraints:
    - "Use Result<T, E> for fallible operations"
    - "Use thiserror or anyhow for error types"
    - "Add context to errors"
    - "Avoid unwrap() in library code"
prompt: |
  Improve error handling in Rust code at: %s
  
  Changes:
  - Replace panics with proper error handling
  - Add error context
  - Use appropriate error types
  - Implement proper error propagation
  - Add helpful error messages
```

## Tips for Creating Effective Commands

1. **Start simple** - Begin with basic command + prompt, add complexity as needed
2. **Use examples** - Show typical invocations to clarify usage
3. **Be specific in constraints** - Clear output requirements help AI deliver what you want
4. **Chain thoughtfully** - Use pre/post to create clear workflows
5. **Validate inputs** - Use the `validation.exists` for file paths
6. **Document thoroughly** - Add descriptions to help others (and future you) understand intent
7. **Test iteratively** - Start with a simple version and refine based on results

## Error Handling Best Practices

1. **Keep it simple** - Use simple command arrays when you don't need error handling
2. **Fail-fast by default** - Don't use `continue_on_failure: true` unless you have a good reason
3. **Validate commands exist** - AI will check that referenced commands exist and warn you
4. **Handle failures explicitly** - Use `on_failure` to define what happens, don't rely on AI improvisation
5. **Report, don't recover** - If `on_failure`/`on_success` commands fail, AI reports it clearly but doesn't try to fix it
6. **You own the flow** - The schema provides structure, but you define the logic. AI just executes and reports.

**Example of good error handling:**
```yaml
pre:
  - command: :validate
    on_failure: :log-validation-error  # Clear, explicit failure handling
  - command: :backup
    continue_on_failure: true          # Backup is nice-to-have
    on_failure: :log-backup-warning
```

**Example of over-complication (avoid this):**
```yaml
pre:
  - command: :step1
    on_success: :step2
    on_failure: :step3
  # This creates a confusing execution tree - use sequential pre array instead
```
