# Claude Code Instructions

This document provides instructions for Claude Code when working with this repository.

## Project Overview

This is a GitHub Action (`action-helm-deploy`) that deploys Helm charts from OCI registries (GCP Artifact Registry or AWS ECR) to Kubernetes clusters.

## Repository Structure

```
action-helm-deploy/
├── action.yml           # Main action definition (composite action)
├── README.md            # Project overview and quick start
├── CLAUDE.md            # This file - Claude Code instructions
├── docs/
│   ├── USAGE.md         # Detailed usage documentation
│   ├── CODESTYLE.md     # Coding standards and conventions
│   └── TROUBLESHOOTING.md # Common issues and solutions
└── .github/
    ├── workflows/
    │   └── main.yml     # CI/CD workflow
    ├── CODE_OF_CONDUCT.md
    ├── CONTRIBUTING.md
    ├── GOVERNANCE.md
    └── SECURITY.md
```

## Key Files

- **action.yml**: The main action definition. This is a composite action that contains all the deployment logic including authentication, Helm templating, and kubectl apply.

## Development Commands

```bash
make lint    # Run yamllint on YAML files
make build   # Validate the action configuration
```

## Making Changes

### Modifying the Action

When modifying `action.yml`:

1. Ensure all inputs have descriptions
2. Use conditional execution with `if: ${{ inputs.registry == 'gcp' }}` or `if: ${{ inputs.registry == 'aws' }}`
3. Use `shell: sh` for shell script steps
4. Handle errors explicitly with exit codes
5. Store generated manifests as artifacts for debugging

### Adding New Registry Support

To add support for a new registry:

1. Add new inputs in the `inputs` section
2. Add authentication steps with appropriate conditionals
3. Add login step to the registry
4. Add deployment step using `helm template` and `kubectl apply`
5. Update documentation in README.md and docs/USAGE.md

## Code Style

- Use British English in comments and documentation
- Follow [Conventional Commits](https://www.conventionalcommits.org/) for commit messages
- Use 2-space indentation for YAML
- Use `snake_case` for input names
- See [CODESTYLE.md](docs/CODESTYLE.md) for full guidelines

## Testing Changes

This action requires integration testing with actual cloud resources. To test:

1. Fork the repository
2. Set up test secrets (GCP or AWS credentials)
3. Create a test workflow that uses the action
4. Push changes and verify the workflow succeeds

## Documentation Updates

When making changes:

- Update README.md for significant feature changes
- Update docs/USAGE.md for user-facing changes
- Update docs/CODESTYLE.md for developer guidelines
- Update docs/TROUBLESHOOTING.md for new error scenarios
