# ai-context

A collection of Claude Code plugins to enhance your AI development workflow.

## What is this?

This repository serves as a **Claude Code plugin marketplace** - a curated collection of plugins that extend Claude Code's capabilities. Each plugin is designed to integrate seamlessly with Claude Code's hooks, skills, and MCP (Model Context Protocol) systems.

## Plugins

### 📊 [context-telemetry](./context-telemetry)

Auto-track your Claude Code usage patterns with skills, agents, and MCP tools.

**Features:**
- Automatic tracking of all skill invocations, agent spawns, and MCP tool calls
- Web dashboard with usage analytics and trends
- Zero overhead, fire-and-forget design
- Local-first data storage in SQLite
- Timeline views and usage insights

[View Documentation →](./context-telemetry/README.md)

## Installation

### Recommended: Install via Claude Code

The easiest way to install this plugin marketplace:

```
/plugin add marketplace https://github.com/umairkhancis/ai-context.git
```

This will:
- Clone the repository to your Claude Code plugins directory
- Register all plugins from the marketplace
- Make them available immediately in your session

## Project Structure

```
ai-context/
├── .claude-plugin/
│   └── marketplace.json     # Plugin registry for Claude Code
├── context-telemetry/       # Usage tracking & analytics plugin
│   ├── src/
│   ├── hooks/
│   └── README.md
└── README.md                # This file
```

## How It Works

This repository uses Claude Code's plugin marketplace format:

1. **Plugin Registry**: `.claude-plugin/marketplace.json` defines available plugins
2. **Individual Plugins**: Each subdirectory contains a complete plugin with:
   - MCP server implementation (optional)
   - Hook configurations (optional)
   - Skills and custom tools (optional)
   - Independent README and documentation

3. **Installation Flow**: 
   - Claude Code reads `marketplace.json`
   - Each plugin's `source` directory contains initialization scripts
   - Plugins auto-install dependencies on first use

## Creating Your Own Plugin

Want to add a plugin to this collection?

1. Create a new directory for your plugin
2. Add plugin metadata to `.claude-plugin/marketplace.json`:

```json
{
  "plugins": [
    {
      "name": "your-plugin-name",
      "description": "What your plugin does",
      "source": "./your-plugin-name"
    }
  ]
}
```

3. Create your plugin structure:
   - `src/` - Server code, hooks, tools
   - `README.md` - Documentation
   - `CLAUDE.md` - Guidance for Claude Code when working with your plugin
   - `.claude-plugin/plugin.json` - Plugin configuration

See [context-telemetry](./context-telemetry) as a reference implementation.

## Requirements

- **Claude Code**: Latest version (cli, desktop, or web)
- **Node.js**: >= 22.5 (for JavaScript-based plugins)

## Plugin Development Guidelines

When building plugins for this collection:

1. **Self-contained**: Each plugin should work independently
2. **Auto-install**: Use init scripts to handle dependencies
3. **Local-first**: Keep data on user's machine by default
4. **Zero-config**: Work out of the box with sensible defaults
5. **Documented**: Provide clear README and CLAUDE.md files
6. **Non-blocking**: Never block Claude Code's main workflow

## Contributing

Contributions welcome! To add a new plugin:

1. Fork this repository
2. Create a new plugin directory
3. Update `.claude-plugin/marketplace.json`
4. Submit a pull request

Please ensure your plugin:
- Has comprehensive documentation
- Follows the project structure conventions
- Includes tests (where applicable)
- Works with the latest Claude Code version

## License

MIT

## Author

**Umair Ahmed Khan**
- GitHub: [@umairkhancis](https://github.com/umairkhancis)
- Email: umairkhan.cis@gmail.com

## Support

- **Issues**: [GitHub Issues](https://github.com/umairkhancis/ai-context/issues)
- **Discussions**: [GitHub Discussions](https://github.com/umairkhancis/ai-context/discussions)
- **Claude Code Help**: Run `/help` in Claude Code or visit [claude.ai/code](https://claude.ai/code)

## Roadmap

Future plugins planned:
- 🔍 Code review automation
- 📝 Documentation generator
- 🧪 Test coverage tracker
- 🚀 Deployment workflow manager
- 📊 Performance monitoring

Have an idea for a plugin? [Open an issue](https://github.com/umairkhancis/ai-context/issues/new)!

---

Built with ❤️ for the Claude Code community
