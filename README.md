# Continuous Claude Code Plugin Marketplace

Shared Claude Code plugin marketplace for Continuous engineering. Provides standardized AI-assisted development tools across all repositories.

## Quick Start

### Automatic (Recommended)
Add to your repository's `.claude/settings.json`:
```json
{
  "extraKnownMarketplaces": {
    "continuous-tools": {
      "source": {
        "source": "github",
        "repo": "smatechnologies/continuous-tools"
      }
    }
  },
  "enabledPlugins": {
    "ct@continuous-tools": true
  }
}
```

### Manual
```
/plugin marketplace add smatechnologies/continuous-tools
/plugin install ct@continuous-tools
```
