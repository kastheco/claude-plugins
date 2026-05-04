# kas-claude-plugins

A marketplace of Claude Code plugins for workflow automation, task tracking, and code review.

## Available Plugins

| Plugin | Description | Version |
|--------|-------------|---------|
| [kas](./plugins/kas/) | Workflow automation with session management and code review | 2.0.0 |
| [task](./plugins/task/) | ClickUp task management with kas workflow integration | 2.0.0 |
| [kascheck](./plugins/kascheck/) | Production-readiness audit (security, code quality, infrastructure, testing) | 1.0.0 |

## Installation

### Add the Marketplace

```bash
# Add marketplace
claude plugin marketplace add brkastner/kas-claude-plugins

# List available plugins
claude plugin marketplace list kas-claude-plugins
```

### Install a Plugin

```bash
# Install kas plugin
claude plugin install kas@kas-claude-plugins
```

### Local Development

```bash
# Clone the repo
git clone https://github.com/brkastner/kas-claude-plugins.git

# Use specific plugin with Claude Code
claude --plugin-dir ./plugins/kas
```

## Plugin Documentation

Each plugin has its own README with detailed usage instructions:

- [kas plugin](./plugins/kas/README.md) - Workflow automation with sessions and code review
- [task plugin](./plugins/task/README.md) - ClickUp task management
- [kascheck plugin](./plugins/kascheck/README.md) - Production-readiness audits

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on adding new plugins.

## License

MIT
