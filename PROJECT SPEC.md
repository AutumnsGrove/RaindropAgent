# Raindrop Cleanup Agent - Technical Specification

## 1. Project Overview

**Name:** `raindrop-cleanup-agent`  
**Date:** `Sunday, October 19th, 2025`
**Purpose:** Autonomous agent to clean, enhance, and organize Raindrop bookmarks with intelligent title cleanup, thumbnail correction, description enhancement, and smart tagging.

**Key Features:**
- Parallel subagent processing (5 Haiku agents)
- Multiple operation modes (demo, full, collection-specific)
- Optional Gradio UI for visualization and manual approval
- SQLite-based tag analytics
- Backup and undo capabilities
- Cron job support for autonomous operation
- State management via tagging

---

## 2. Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Language | Python | 3.11+ |
| Environment/Runner | UV | Latest |
| AI Framework | Claude Agent SDK | Latest |
| AI Models | Sonnet 4 (orchestrator), Haiku (workers) | Latest |
| UI Framework | Gradio | 4.x+ |
| Database | SQLite | 3.x |
| Progress Display | Rich | Latest |
| Task Scheduling | APScheduler + cron | Latest |
| External Tools | yt-thumbs (UV tool) | Existing |

---

## 3. Architecture

### 3.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Main Orchestrator                       │
│                  (Claude Sonnet 4.5)                       │
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
        │                     │
        │                     │
        ▼                     ▼
┌─────────────────────────────────────┐
│         Shared Resources             │
│  - SQLite DB (tag analytics)        │
│  - Backup storage (JSON)            │
│  - Config (YAML)                    │
│  - Raindrop API Wrapper             │
└─────────────────────────────────────┘
```

### 3.2 Component Breakdown

#### 3.2.1 Main Orchestrator (Sonnet 4.5)
- Entry point for all operations
- Fetches raindrops from Raindrop API
- Filters out already-processed items (`#processed-by-script` tag)
- Creates batches of 5 items
- Spawns 5 Haiku subagents per batch
- Aggregates results
- Applies updates to Raindrop API
- Manages backups and undo operations

#### 3.2.2 Tag Analyzer Subagent (Haiku)
- Runs once at startup (or on-demand)
- Fetches all existing tags from Raindrop
- Calculates tag frequency distribution
- Stores results in SQLite
- Provides recommendations for tag reuse

#### 3.2.3 Worker Subagents (5x Haiku)
- Each processes exactly one raindrop
- Performs all cleanup operations:
  - Title cleanup
  - Thumbnail correction (YouTube via `yt-thumbs`)
  - Description enhancement
  - Tag optimization (queries SQLite for existing tags)
- Returns proposed changes as structured JSON
- Autonomous execution with optional manual approval

#### 3.2.4 Raindrop API Wrapper
- Abstracts all Raindrop API calls
- Handles authentication
- Implements rate limiting
- Provides clean Python interface

#### 3.2.5 SQLite Database
- Stores tag analytics
- Tracks processing history
- Enables tag visualization in UI

#### 3.2.6 Optional Gradio UI
- Tab 1: Processing view (side-by-side comparison)
- Tab 2: Tag analytics (frequency charts, distribution)
- Tab 3: History/undo
- Real-time progress updates
- Manual approval workflow

---

## 4. Data Models

### 4.1 SQLite Schema

```sql
-- Tag analytics
CREATE TABLE tags (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL,
    frequency INTEGER DEFAULT 0,
    last_used TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Processing history
CREATE TABLE processing_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    raindrop_id INTEGER NOT NULL,
    processed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    changes JSON,
    backup JSON,
    status TEXT -- 'pending', 'approved', 'applied', 'reverted'
);

-- Config/state
CREATE TABLE config (
    key TEXT PRIMARY KEY,
    value TEXT
);
```

### 4.2 Backup Format (JSON)

```json
{
  "raindrop_id": 12345,
  "timestamp": "2025-10-19T15:30:00Z",
  "original": {
    "title": "Original Title",
    "excerpt": "Original description",
    "tags": ["tag1", "tag2"],
    "cover": "https://..."
  },
  "changes": {
    "title": "New Title",
    "excerpt": "New description",
    "tags": ["tag1", "tag3", "tag4"],
    "cover": "https://youtube.com/..."
  }
}
```

### 4.3 Config File (YAML)

```yaml
# config.yaml
raindrop:
  api_token: "${RAINDROP_API_TOKEN}"  # From env var
  
processing:
  batch_size: 5
  parallel_subagents: 5
  dry_run: false
  require_approval: false
  
demo:
  sample_size: 5
  auto_backup: true
  
title_cleanup:
  max_length: 80
  remove_patterns:
    - "GitHub - "
    - "Amazon.com: "
    - "reddit - "
    - " - YouTube"
  obsidian_friendly: true
  
description:
  min_length: 20
  max_length: 500
  rewrite_generic: true
  
tagging:
  min_tag_frequency: 3
  max_new_tags_per_item: 5
  reuse_threshold: 0.7
  analyze_on_startup: true
  
thumbnails:
  fix_youtube: true
  use_yt_thumbs: true
  fix_other_domains: false  # Reach goal
  
state:
  processing_tag: "processed-by-script"
  resume_enabled: true
  
cron:
  enabled: false
  schedule: "0 3 * * *"  # 3 AM daily
  timezone: "America/Los_Angeles"
  
api:
  rate_limit: 100  # requests per minute
  retry_attempts: 3
  retry_delay: 2  # seconds
  
ui:
  enabled: false
  port: 7860
  share: false
```

---

## 5. CLI Interface

### 5.1 Command Structure

```bash
# Using UV
uv run raindrop-cleanup [MODE] [OPTIONS]
```

### 5.2 Modes

| Mode | Description | Example |
|------|-------------|---------|
| `demo` | Process 5 random entries with backup | `uv run raindrop-cleanup demo` |
| `collection` | Process specific collection | `uv run raindrop-cleanup collection --id 12345` |
| `all` | Process entire library | `uv run raindrop-cleanup all` |
| `analyze-tags` | Run tag analysis only | `uv run raindrop-cleanup analyze-tags` |
| `undo` | Revert specific changes | `uv run raindrop-cleanup undo --id 12345` |
| `cleanup-tags` | Remove processing tags | `uv run raindrop-cleanup cleanup-tags` |

### 5.3 Options/Flags

| Flag | Description | Default |
|------|-------------|---------|
| `--manual` | Require manual approval for each change | `false` |
| `--ui` | Launch Gradio UI | `false` |
| `--dry-run` | Show proposed changes without applying | `false` |
| `--config` | Path to config file | `./config.yaml` |
| `--limit` | Max items to process | `none` |
| `--backup-dir` | Custom backup directory | `./backups` |

### 5.4 Examples

```bash
# Demo mode with UI and manual approval
uv run raindrop-cleanup demo --ui --manual

# Process entire library autonomously
uv run raindrop-cleanup all

# Process specific collection with dry-run
uv run raindrop-cleanup collection --id 12345 --dry-run

# Analyze tags and show UI
uv run raindrop-cleanup analyze-tags --ui

# Undo changes for specific raindrop
uv run raindrop-cleanup undo --id 67890

# Remove all processing tags after completion
uv run raindrop-cleanup cleanup-tags
```

---

## 6. Processing Pipeline

### 6.1 Initialization Phase

```
1. Load config.yaml
2. Initialize Raindrop API client
3. Initialize SQLite database
4. Run tag analyzer subagent (if analyze_on_startup: true)
5. Load or create backup directory
```

### 6.2 Main Processing Loop

```
For each batch of 5 raindrops:
  1. Filter out items with #processed-by-script tag
  2. Create backup JSON files
  3. Spawn 5 Haiku subagents (one per raindrop)
  4. Each subagent:
     a. Analyze current state
     b. Query SQLite for existing tags
     c. Clean title
     d. Fix thumbnail (if YouTube, call yt-thumbs)
     e. Enhance description
     f. Optimize tags
     g. Return proposed changes as JSON
  5. Aggregate results from subagents
  6. If --manual or --ui:
     - Display changes in UI
     - Wait for approval
  7. If approved (or auto mode):
     - Apply changes via Raindrop API
     - Add #processed-by-script tag
     - Save to processing_history
  8. Update progress bar
  9. Rate limit pause if needed
```

### 6.3 Post-Processing

```
1. Generate summary report
2. Update tag frequency in SQLite
3. Optional: Remove #processed-by-script tags (if requested)
4. Export changelog
```

---

## 7. Subagent Implementation

### 7.1 Worker Subagent Prompt Template

```python
WORKER_PROMPT = """
You are a Raindrop bookmark cleanup specialist. Analyze the following bookmark and propose improvements.

# Current Bookmark
Title: {title}
Description: {description}
Tags: {tags}
URL: {url}
Cover: {cover}

# Existing Tag Distribution (Top 50)
{tag_frequency_json}

# Instructions
1. **Title Cleanup**: Remove site prefixes, excessive descriptions. Make it concise and Obsidian-friendly (max 80 chars).
2. **Thumbnail**: If YouTube URL, indicate need for yt-thumbs. Otherwise, keep existing.
3. **Description**: Rewrite if generic or mismatched. Keep it informative (20-500 chars).
4. **Tags**: 
   - REUSE existing tags when semantically similar (threshold: 0.7)
   - Only create new tags if truly distinct and likely to be reused
   - Max 5 tags per item
   - Avoid orphan tags (used only once)

# Output Format (JSON only)
{{
  "title": "Improved title",
  "description": "Enhanced description",
  "tags": ["existing-tag1", "existing-tag2", "new-tag"],
  "thumbnail_action": "use_yt_thumbs" | "keep" | "none",
  "reasoning": "Brief explanation of changes"
}}
"""
```

### 7.2 Tag Analyzer Subagent Prompt

```python
TAG_ANALYZER_PROMPT = """
You are a tag analytics specialist. Analyze the following list of tags and their frequencies.

# All Tags
{all_tags_json}

# Instructions
1. Calculate frequency distribution
2. Identify commonly used tags (frequency >= 3)
3. Identify potential synonyms/duplicates
4. Suggest consolidation opportunities
5. Store results in SQLite

# Output Format (JSON)
{{
  "total_tags": 150,
  "frequently_used": ["ai", "coding", "python", ...],
  "orphan_tags": ["one-time-tag1", ...],
  "duplicates": [["ai", "artificial-intelligence"], ...],
  "recommendations": ["Consolidate X and Y", ...]
}}
"""
```

---

## 8. Gradio UI Design

### 8.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│  Raindrop Cleanup Agent                   [Start] [Stop]│
├──────────┬──────────────────────────────────────────────┤
│          │                                              │
│ Processing│  ┌─────────────────┬─────────────────┐      │
│ Tag Analytics│     Current     │    Proposed     │      │
│ History  │   │                 │                 │      │
│ Settings │   │  Title: Old     │  Title: New     │      │
│          │   │  Desc: ...      │  Desc: ...      │      │
│          │   │  Tags: ...      │  Tags: ...      │      │
│          │   │  Cover: [img]   │  Cover: [img]   │      │
│          │   │                 │                 │      │
│          │   └─────────────────┴─────────────────┘      │
│          │                                              │
│          │  [Approve] [Reject] [Approve All Remaining]  │
│          │                                              │
│          │  Progress: ████████░░░░ 15/50 (30%)          │
└──────────┴──────────────────────────────────────────────┘
```

### 8.2 Tab Descriptions

**Tab 1: Processing**
- Side-by-side comparison of current vs. proposed
- Image previews for thumbnails
- Approve/reject buttons
- Progress bar
- Real-time updates

**Tab 2: Tag Analytics**
- Bar chart of tag frequency
- Tag cloud visualization
- Orphan tags list
- Duplicate detection
- SQLite query interface

**Tab 3: History**
- List of all processed items
- Undo button per item
- Bulk undo
- Export changelog

**Tab 4: Settings**
- Live config editing
- Save config
- Test connection
- View logs

---

## 9. Cron Job Integration

### 9.1 macOS (Primary)

Use `launchd` for scheduling:

```xml
<!-- ~/Library/LaunchAgents/com.raindrop.cleanup.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.raindrop.cleanup</string>
    <key>ProgramArguments</key>
    <array>
        <string>/path/to/uv</string>
        <string>run</string>
        <string>raindrop-cleanup</string>
        <string>all</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>3</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/raindrop-cleanup.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/raindrop-cleanup.err</string>
</dict>
</plist>
```

### 9.2 Linux (Reach Goal)

Use `cron`:

```bash
# crontab -e
0 3 * * * cd /path/to/raindrop-cleanup && /path/to/uv run raindrop-cleanup all
```

### 9.3 Setup Command

```bash
# Install cron job
uv run raindrop-cleanup install-cron --time "03:00"

# Uninstall
uv run raindrop-cleanup uninstall-cron
```

---

## 10. Error Handling & Safety

### 10.1 Error Categories

| Error Type | Handler | Recovery |
|------------|---------|----------|
| API Rate Limit | Exponential backoff | Wait and retry |
| Network Error | Retry 3x | Log and skip |
| Invalid Data | Log and skip | Continue processing |
| Subagent Failure | Restart subagent | Retry once |
| Database Error | Log critical | Halt processing |

### 10.2 Backup Strategy

- Create JSON backup before ANY modification
- Store in `./backups/{timestamp}/raindrop_{id}.json`
- Retain backups for 30 days
- Undo reads from backup

### 10.3 Undo Mechanism

```python
def undo_changes(raindrop_id: int):
    1. Query processing_history for raindrop_id
    2. Load backup JSON
    3. Apply original values via Raindrop API
    4. Remove #processed-by-script tag
    5. Update status to 'reverted' in DB
```

---

## 11. Testing Strategy

### 11.1 Demo Mode Testing

```bash
# Run demo mode with 5 random entries
uv run raindrop-cleanup demo --ui --manual

# Verify:
# - Backups created
# - Changes look reasonable
# - UI displays correctly
# - Manual approval works
# - Undo works
```

### 11.2 Dry-Run Testing

```bash
# Test full collection without applying changes
uv run raindrop-cleanup collection --id 12345 --dry-run

# Verify:
# - No API writes
# - Changes logged correctly
# - Progress bar works
```

### 11.3 Tag Analyzer Testing

```bash
# Run tag analysis independently
uv run raindrop-cleanup analyze-tags --ui

# Verify:
# - SQLite populated
# - Frequency calculations correct
# - UI displays charts
```

---

## 12. Project Structure

```
raindrop-cleanup-agent/
├── pyproject.toml              # UV project config
├── uv.lock                     # Lockfile
├── config.yaml                 # User configuration
├── README.md                   # Documentation
│
├── src/
│   └── raindrop_cleanup/
│       ├── __init__.py
│       ├── __main__.py         # CLI entry point
│       │
│       ├── api/
│       │   ├── __init__.py
│       │   ├── raindrop.py     # Raindrop API wrapper
│       │   └── yt_thumbs.py    # yt-thumbs integration
│       │
│       ├── agents/
│       │   ├── __init__.py
│       │   ├── orchestrator.py # Main Sonnet agent
│       │   ├── worker.py       # Haiku worker subagent
│       │   └── analyzer.py     # Tag analyzer subagent
│       │
│       ├── db/
│       │   ├── __init__.py
│       │   ├── models.py       # SQLite schema
│       │   └── queries.py      # DB operations
│       │
│       ├── ui/
│       │   ├── __init__.py
│       │   └── gradio_app.py   # Gradio interface
│       │
│       ├── utils/
│       │   ├── __init__.py
│       │   ├── backup.py       # Backup/restore logic
│       │   ├── config.py       # Config loader
│       │   └── progress.py     # Progress bars (Rich)
│       │
│       └── cron/
│           ├── __init__.py
│           └── scheduler.py    # Cron integration
│
├── backups/                    # Backup storage (gitignored)
├── data/
│   └── raindrop.db            # SQLite database
│
└── tests/
    ├── __init__.py
    ├── test_api.py
    ├── test_agents.py
    └── test_ui.py
```

---

## 13. Dependencies (pyproject.toml)

```toml
[project]
name = "raindrop-cleanup-agent"
version = "0.1.0"
description = "Autonomous agent to clean and organize Raindrop bookmarks"
requires-python = ">=3.11"

dependencies = [
    "claude-agent-sdk>=0.1.0",
    "gradio>=4.0.0",
    "rich>=13.0.0",
    "pyyaml>=6.0",
    "requests>=2.31.0",
    "python-dotenv>=1.0.0",
    "apscheduler>=3.10.0",
]

[project.scripts]
raindrop-cleanup = "raindrop_cleanup.__main__:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

---

## 14. Implementation Phases

### Phase 1: Core Infrastructure (Week 1)
- [ ] Set up UV project structure
- [ ] Implement Raindrop API wrapper
- [ ] Create SQLite schema and basic queries
- [ ] Implement backup system
- [ ] Build CLI framework

### Phase 2: Agent Development (Week 1-2)
- [ ] Implement orchestrator agent (Sonnet)
- [ ] Implement worker subagents (Haiku)
- [ ] Implement tag analyzer subagent
- [ ] Test agent communication
- [ ] Implement state management (#processed-by-script)

### Phase 3: Processing Logic (Week 2)
- [ ] Title cleanup logic
- [ ] Thumbnail correction (yt-thumbs integration)
- [ ] Description enhancement
- [ ] Tag optimization with SQLite queries
- [ ] Batch processing pipeline

### Phase 4: UI & UX (Week 2-3)
- [ ] Build Gradio interface
- [ ] Implement side-by-side comparison view
- [ ] Add tag analytics visualizations
- [ ] Implement approval workflow
- [ ] Add progress bars and real-time updates

### Phase 5: Advanced Features (Week 3)
- [ ] Undo mechanism
- [ ] Cron job integration (macOS)
- [ ] Demo mode
- [ ] Dry-run mode
- [ ] Cleanup-tags command

### Phase 6: Testing & Polish (Week 3-4)
- [ ] Unit tests
- [ ] Integration tests
- [ ] Demo mode testing with real data
- [ ] Documentation
- [ ] Performance optimization

---

## 15. Success Metrics

- [ ] Successfully processes 5 random entries in demo mode
- [ ] Backup and undo work correctly
- [ ] Title cleanup improves readability
- [ ] YouTube thumbnails fixed via yt-thumbs
- [ ] Tag reuse >= 70% (avoiding orphan tags)
- [ ] Processing speed: ~10-15 items/minute
- [ ] UI responsive and intuitive
- [ ] Cron job runs reliably
- [ ] Zero data loss (backups prevent)

---

## 16. Future Enhancements (Post-MVP)

1. **Additional Thumbnail Sources**: Tools for other domains (like yt-thumbs)
2. **Machine Learning**: Train model on user's approval patterns
3. **Duplicate Detection**: Find and merge duplicate bookmarks
4. **Smart Collections**: Auto-organize into collections based on content
5. **Export/Import**: Share processing rules with others
6. **Web Interface**: Full web app instead of just Gradio
7. **Multi-user**: Support team collections
8. **Analytics Dashboard**: Deep insights into collection health
