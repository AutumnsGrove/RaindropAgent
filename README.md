# Raindrop Cleanup Agent

Autonomous agent to clean, enhance, and organize Raindrop bookmarks with intelligent title cleanup, thumbnail correction, description enhancement, and smart tagging using parallel Claude AI subagents.

## 🚀 Overview

This project uses a main orchestrator (Claude Sonnet 4.5) to manage batches of Raindrop bookmarks, spawning 5 parallel worker subagents (Claude Haiku) to process individual items. It includes optional Gradio UI, SQLite-based tag analytics, backup/undo capabilities, and cron job support for autonomous operation.

### Key Features

- **Parallel Processing**: 5 Haiku subagents working simultaneously
- **Smart Tag Management**: Reuses existing tags, analyzes frequency, prevents orphan tags
- **Thumbnail Correction**: Fixes YouTube thumbnails using yt-thumbs tool
- **Title Cleanup**: Removes site prefixes, makes Obsidian-friendly
- **Description Enhancement**: Rewrites generic or mismatched descriptions
- **Multiple Modes**: Demo (5 items), collection-specific, or entire library
- **Optional UI**: Gradio interface for visualization and manual approval
- **Backup & Undo**: JSON backups before modifications with full restore capability
- **Autonomous Operation**: Cron job support for scheduled processing
- **State Tracking**: Uses `#processed-by-script` tag to track progress

## 📋 Tech Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Python | 3.11+ |
| Environment/Runner | UV | Latest |
| AI Framework | Claude Agent SDK | Latest |
| AI Models | Sonnet 4.5 (orchestrator), Haiku (workers) | Latest |
| UI Framework | Gradio | 4.x+ |
| Database | SQLite | 3.x |
| Progress Display | Rich | Latest |
| Task Scheduling | APScheduler + cron | Latest |
| External Tools | yt-thumbs (UV tool) | Existing |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Main Orchestrator                       │
│                  (Claude Sonnet 4.5)                     │
│  - Fetches raindrops                                    │
│  - Manages batching                                     │
│  - Coordinates subagents                                │
│  - Handles state/resume                                 │
└──────────────────┬──────────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
┌──────────────┐      ┌──────────────┐
│ Tag Analyzer │      │   Worker     │
│   Subagent   │      │  Subagents   │
│ (Haiku x1)   │      │ (Haiku x5)   │
│              │      │              │
│ - Analyze    │      │ - Title fix  │
│   existing   │      │ - Thumbnail  │
│   tags       │      │ - Desc enh.  │
│ - Build freq │      │ - Tag opt.   │
│   map        │      │ - One per    │
│ - Save to DB │      │   raindrop   │
└──────────────┘      └──────────────┘
```

## 🚀 Quick Start

### 1. Setup Environment

```bash
# Ensure UV is installed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies (after project setup)
uv sync
```

### 2. Configure API Keys

Create `secrets.json` with your API keys:

```json
{
  "anthropic_api_key": "sk-ant-api03-...",
  "raindrop_api_token": "your-raindrop-token-here",
  "comment": "Add your API keys here. This file should be kept private."
}
```

**Note**: `secrets.json` is gitignored by default.

### 3. Run Demo Mode

```bash
# Process 5 random entries with backup
uv run raindrop-cleanup demo

# With UI and manual approval
uv run raindrop-cleanup demo --ui --manual

# Dry-run to preview changes
uv run raindrop-cleanup demo --dry-run
```

## 📖 Usage

### Command Structure

```bash
uv run raindrop-cleanup [MODE] [OPTIONS]
```

### Available Modes

| Mode | Description | Example |
|------|-------------|---------|
| `demo` | Process 5 random entries with backup | `uv run raindrop-cleanup demo` |
| `collection` | Process specific collection | `uv run raindrop-cleanup collection --id 12345` |
| `all` | Process entire library | `uv run raindrop-cleanup all` |
| `analyze-tags` | Run tag analysis only | `uv run raindrop-cleanup analyze-tags` |
| `undo` | Revert specific changes | `uv run raindrop-cleanup undo --id 12345` |
| `cleanup-tags` | Remove processing tags | `uv run raindrop-cleanup cleanup-tags` |

### Options/Flags

| Flag | Description | Default |
|------|-------------|---------|
| `--manual` | Require manual approval | `false` |
| `--ui` | Launch Gradio UI | `false` |
| `--dry-run` | Show changes without applying | `false` |
| `--config` | Path to config file | `./config.yaml` |
| `--limit` | Max items to process | `none` |
| `--backup-dir` | Custom backup directory | `./backups` |

## 📁 Project Structure

```
raindrop-cleanup-agent/
├── pyproject.toml              # UV project config
├── config.yaml                 # User configuration
├── secrets.json               # API keys (gitignored)
├── secrets_template.json      # Template for setup
│
├── src/
│   └── raindrop_cleanup/
│       ├── __main__.py         # CLI entry point
│       ├── api/                # Raindrop & yt-thumbs API
│       ├── agents/             # Orchestrator & workers
│       ├── db/                 # SQLite models & queries
│       ├── ui/                 # Gradio interface
│       ├── utils/              # Backup, config, progress
│       └── cron/               # Cron integration
│
├── tests/                     # Unit & integration tests
├── backups/                   # Backup storage (gitignored)
└── data/
    └── raindrop.db           # SQLite database
```

## 📚 Documentation

- **[PROJECT SPEC.md](PROJECT SPEC.md)** - Complete technical specification
- **[DIAGRAMS.md](DIAGRAMS.md)** - Architecture diagrams
- **[ClaudeUsage/](ClaudeUsage/)** - Claude Code workflow guides
- **[CLAUDE.md](CLAUDE.md)** - Project instructions for Claude

## 🔧 Configuration

Edit `config.yaml` to customize behavior:

```yaml
processing:
  batch_size: 5
  parallel_subagents: 5
  dry_run: false
  require_approval: false

title_cleanup:
  max_length: 80
  obsidian_friendly: true

tagging:
  min_tag_frequency: 3
  reuse_threshold: 0.7

thumbnails:
  fix_youtube: true
  use_yt_thumbs: true
```

## 🎯 Features in Detail

### Title Cleanup
- Removes site prefixes (`GitHub - `, `Amazon.com: `, etc.)
- Makes titles Obsidian-friendly (max 80 chars)
- Keeps titles concise and descriptive

### Tag Optimization
- Reuses existing tags (70% similarity threshold)
- Prevents orphan tags (used only once)
- Max 5 tags per item
- Queries SQLite for frequency distribution

### Thumbnail Correction
- Fixes YouTube thumbnails via `yt-thumbs` tool
- Maintains existing thumbnails for other domains
- Future: Support for additional domains

### Description Enhancement
- Rewrites generic descriptions
- Keeps informative content (20-500 chars)
- Matches description to actual content

## 🔐 Security

- API keys stored in `secrets.json` (gitignored)
- Template provided in `secrets_template.json`
- Environment variable fallbacks
- Never commit actual keys

## 🛠️ Development

### Prerequisites
- Python 3.11+
- UV package manager
- Anthropic API key
- Raindrop API token

### Setup for Development

```bash
# Clone repository
git clone [repository-url]
cd RaindropAgent

# Install dependencies
uv sync

# Copy secrets template
cp secrets_template.json secrets.json
# Edit secrets.json with your API keys

# Run tests
uv run pytest
```

## 📝 Implementation Phases

See [PROJECT SPEC.md](PROJECT SPEC.md) for detailed implementation phases.

## 🤝 Contributing

1. Follow git workflow in [ClaudeUsage/git_workflow.md](ClaudeUsage/git_workflow.md)
2. Update TODOS.md as you progress
3. Add tests for new features
4. Follow commit message standards

## 📄 License

This project is provided as-is for personal use.

---

**Last updated:** 2025-10-19
**Status:** Initial Setup
**Model:** Claude Sonnet 4.5
