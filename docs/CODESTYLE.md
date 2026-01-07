# Code Style

This document describes the coding standards and conventions for the `action-helm-deploy` GitHub Action.

## YAML Formatting

Code should be formatted using yamllint. The configuration file is located at `.yamllint` in the root of the repository.

### Running the Linter

```bash
make lint
```

### Key Guidelines

- Use 2 spaces for indentation
- Avoid trailing whitespace
- Use lowercase for keys
- Use descriptive names for workflow steps
- Use British English in comments and descriptions
- Add blank lines between major sections

### Example

```yaml
---
name: "My Action"
description: |
  This action does something useful.
inputs:
  my_input:
    description: "Description of the input"
    required: true
```

## Shell Scripts

The action contains embedded shell scripts in `action.yml`. Follow these guidelines:

### Variable Handling

- Use double quotes for variable expansion
- Check required variables with explicit error messages
- Exit with non-zero status on errors
- Use meaningful variable names

### Example

```bash
if [ -z "${REQUIRED_VAR}" ]; then
  echo "Error: REQUIRED_VAR is not set." >&2
  exit 1
fi
```

### Error Handling

- Always check command exit codes
- Provide descriptive error messages
- Use `>&2` to redirect error messages to stderr
- Exit with appropriate non-zero codes on failure

### Script Structure

- Set variables at the top
- Perform validation early
- Group related operations
- Add comments for complex logic

## GitHub Action Inputs

### Naming Conventions

- Use `snake_case` for input names
- Use descriptive names that indicate purpose
- Group related inputs together in the action definition

### Default Values

- Provide sensible defaults where possible
- Use empty string `""` for optional inputs without defaults
- Document when inputs are conditionally required

## Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

| Prefix | Purpose |
|--------|---------|
| `feat:` | New features |
| `fix:` | Bug fixes |
| `docs:` | Documentation changes |
| `chore:` | Maintenance tasks |
| `refactor:` | Code refactoring |
| `test:` | Test additions or changes |

### Example

```
feat: Add AWS ECR registry support

Add support for deploying Helm charts from AWS ECR using OIDC authentication.
```

## Pull Requests

- Create feature branches from `main`
- Use descriptive branch names: `feat/description`, `fix/description`
- Ensure all linting checks pass before requesting review
- Keep pull requests focused on a single change
- Write clear descriptions of what the PR changes

## Documentation

- Use British English in all documentation
- Keep the README.md concise with links to detailed docs
- Update USAGE.md for user-facing changes
- Update this CODESTYLE.md for developer-facing changes
- Use Mermaid diagrams for workflow visualisation
