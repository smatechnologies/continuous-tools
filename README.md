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
        "source": "git",
        "url": "https://dev.azure.com/SMATechnologies/Internal/_git/ClaudeCode"
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
/plugin marketplace add https://dev.azure.com/SMATechnologies/Internal/_git/ClaudeCode
/plugin install ct@continuous-tools
```

## Plugin: ct


## Updating

```
/plugin marketplace update continuous-tools
```
