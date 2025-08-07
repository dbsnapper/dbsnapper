# VSCode Development Tasks

This directory contains VSCode tasks and launch configurations for DBSnapper documentation development.

## Quick Start Tasks

The fastest way to get started is with the compound tasks marked with emojis:

### 🚀 Start Development (Regular)
- **Shortcut**: `Ctrl+Shift+P` → `Tasks: Run Task` → `🚀 Start Development (Regular)`
- **What it does**: Installs dependencies and starts regular MkDocs server
- **Use when**: Writing/editing documentation content
- **URL**: http://127.0.0.1:8000

### 🎯 Start Development (Versioned)  
- **Shortcut**: `Ctrl+Shift+P` → `Tasks: Run Task` → `🎯 Start Development (Versioned)`
- **What it does**: Installs dependencies and starts Mike versioned server
- **Use when**: Testing version switching and multi-version workflows
- **URL**: http://127.0.0.1:8000

### 🔄 Fresh Setup
- **Shortcut**: `Ctrl+Shift+P` → `Tasks: Run Task` → `🔄 Fresh Setup`
- **What it does**: Creates fresh virtual environment and installs all dependencies
- **Use when**: First time setup or after environment issues

### 📦 Build & Validate
- **Shortcut**: `Ctrl+Shift+P` → `Tasks: Run Task` → `📦 Build & Validate`
- **What it does**: Builds static site and lists available versions
- **Use when**: Checking build status and version availability

## Individual Tasks

### Development Servers

| Task | Description | URL |
|------|-------------|-----|
| `mkdocs-serve` | Regular MkDocs development server | http://127.0.0.1:8000 |
| `mike-serve` | Versioned docs server with version switcher | http://127.0.0.1:8000 |

### Building & Testing

| Task | Description | Output |
|------|-------------|--------|
| `mkdocs-build` | Build static documentation site | `./site/` directory |
| `mike-list-versions` | Show all available versions | Console output |

### Version Management

| Task | Description | Interactive? |
|------|-------------|--------------|
| `mike-deploy-version` | Deploy new version and update latest | ✅ (prompts for version) |
| `mike-deploy-dev` | Deploy current branch as dev version | No |

### Environment Setup

| Task | Description | When to Use |
|------|-------------|-------------|
| `setup-venv` | Create fresh virtual environment | First setup or after issues |
| `install-dependencies` | Install from requirements.txt | After pulling new changes |
| `update-pip-dependencies` | Update all packages to latest | Maintenance |

## Keyboard Shortcuts

Add these to your `keybindings.json` for faster access:

```json
[
  {
    "key": "ctrl+shift+s",
    "command": "workbench.action.tasks.runTask",
    "args": "🚀 Start Development (Regular)"
  },
  {
    "key": "ctrl+shift+v", 
    "command": "workbench.action.tasks.runTask",
    "args": "🎯 Start Development (Versioned)"
  },
  {
    "key": "ctrl+shift+b",
    "command": "workbench.action.tasks.runTask", 
    "args": "📦 Build & Validate"
  }
]
```

## Launch Configurations

Use `F5` or the Run panel to start:

- **🚀 Start MkDocs Development Server**: Regular development server
- **🎯 Start Mike Versioned Server**: Versioned server with switching

## Troubleshooting

### Virtual Environment Issues
Run `🔄 Fresh Setup` task to recreate the environment.

### Dependency Issues  
Run `install-dependencies` or `update-pip-dependencies` task.

### Port Conflicts
If port 8000 is busy, tasks will automatically find alternative ports or you can modify the `--dev-addr` parameter in tasks.json.

### Cairo Library Issues (macOS)
If you get cairo-related errors:
```bash
brew install cairo
```

## Task Configuration Details

All tasks use:
- **Shell**: zsh with login profile (`-c -l`)
- **Working Directory**: Workspace root  
- **Virtual Environment**: `./.venv/bin/activate`
- **Error Handling**: VSCode problem matchers where applicable

Tasks are grouped by:
- **build**: Development and deployment tasks
- **test**: Validation and listing tasks

## Adding New Tasks

To add custom tasks:

1. Edit `.vscode/tasks.json`
2. Follow the existing pattern with venv activation
3. Add appropriate `detail` and `group` properties
4. Consider adding to compound tasks if it's part of a workflow

## Files

- `tasks.json` - Task definitions
- `launch.json` - Debug/run configurations  
- `README.md` - This documentation