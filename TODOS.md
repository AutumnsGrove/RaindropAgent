# Project TODOs - Raindrop Cleanup Agent

**Status**: Initial Setup Complete
**Next Phase**: Core Infrastructure
**Last Updated**: 2025-10-19

---

## 🚀 Immediate Next Steps

### 1. Setup & Configuration
- [ ] Create `secrets.json` from `secrets_template.json` with actual API keys
- [ ] Create `config.yaml` with initial configuration
- [ ] Test Raindrop API connection
- [ ] Test Anthropic API connection

### 2. Initial Development
- [ ] Implement basic Raindrop API wrapper (`src/raindrop_cleanup/api/raindrop.py`)
- [ ] Implement secrets loading utility (`src/raindrop_cleanup/utils/config.py`)
- [ ] Create sample unit tests

---

## Phase 1: Core Infrastructure

### Database & Persistence
- [ ] Create SQLite schema (`src/raindrop_cleanup/db/models.py`)
  - Tags table with frequency tracking
  - Processing history table
  - Config/state table
- [ ] Implement database operations (`src/raindrop_cleanup/db/queries.py`)
- [ ] Test database creation and queries

### API Wrappers
- [ ] Implement Raindrop API wrapper (`src/raindrop_cleanup/api/raindrop.py`)
  - Authentication
  - Fetch raindrops (all, by collection)
  - Update raindrop (title, description, tags, cover)
  - Fetch all tags
  - Rate limiting
- [ ] Implement yt-thumbs integration (`src/raindrop_cleanup/api/yt_thumbs.py`)
- [ ] Test API wrapper with real Raindrop account

### Backup System
- [ ] Implement backup functionality (`src/raindrop_cleanup/utils/backup.py`)
  - Create JSON backups before modifications
  - Backup directory management
  - Backup retention (30 days)
- [ ] Implement restore/undo functionality
- [ ] Test backup and restore

### Configuration
- [ ] Implement config loader (`src/raindrop_cleanup/utils/config.py`)
  - Load YAML config
  - Merge with defaults
  - Environment variable support
- [ ] Create default `config.yaml`

### CLI Framework
- [ ] Enhance CLI with all modes (`src/raindrop_cleanup/__main__.py`)
  - demo mode
  - collection mode
  - all mode
  - analyze-tags mode
  - undo mode
  - cleanup-tags mode
- [ ] Implement progress bars with Rich (`src/raindrop_cleanup/utils/progress.py`)

---

## Phase 2: Agent Development

### Orchestrator Agent (Sonnet 4.5)
- [ ] Implement main orchestrator (`src/raindrop_cleanup/agents/orchestrator.py`)
  - Fetch raindrops from Raindrop API
  - Filter out processed items (#processed-by-script tag)
  - Create batches of 5 items
  - Spawn 5 Haiku subagents per batch
  - Aggregate results
  - Apply updates to Raindrop API

### Tag Analyzer Subagent (Haiku)
- [ ] Implement tag analyzer (`src/raindrop_cleanup/agents/analyzer.py`)
  - Fetch all existing tags
  - Calculate frequency distribution
  - Store in SQLite
  - Identify orphan tags
  - Suggest consolidations

### Worker Subagents (5x Haiku)
- [ ] Implement worker subagent (`src/raindrop_cleanup/agents/worker.py`)
  - Title cleanup logic
  - Thumbnail correction (YouTube detection)
  - Description enhancement
  - Tag optimization (query SQLite)
  - Return structured JSON response

### Agent Communication
- [ ] Test agent spawning and communication
- [ ] Implement error handling for subagent failures
- [ ] Test parallel processing with 5 workers

---

## Phase 3: Processing Logic

### Title Cleanup
- [ ] Implement title cleanup patterns
  - Remove common site prefixes (GitHub, Amazon, Reddit, YouTube, etc.)
  - Enforce max length (80 chars)
  - Make Obsidian-friendly
  - Handle edge cases

### Thumbnail Correction
- [ ] Implement YouTube URL detection
- [ ] Integrate with yt-thumbs tool
- [ ] Handle thumbnail update via Raindrop API
- [ ] Error handling for failed thumbnail fetches

### Description Enhancement
- [ ] Implement description analysis
  - Detect generic descriptions
  - Rewrite if needed (20-500 chars)
  - Maintain informative content
  - Match content to URL

### Tag Optimization
- [ ] Implement tag reuse logic
  - Query SQLite for existing tags
  - Calculate similarity (threshold: 0.7)
  - Limit to 5 tags per item
  - Avoid orphan tags
  - Prefer frequent tags

### Batch Processing
- [ ] Implement main processing loop
  - Batch creation (5 items)
  - Parallel subagent spawning
  - Result aggregation
  - API updates
  - Progress tracking

---

## Phase 4: UI & UX

### Gradio Interface
- [ ] Implement Gradio app (`src/raindrop_cleanup/ui/gradio_app.py`)

### Processing Tab
- [ ] Side-by-side comparison view (current vs. proposed)
- [ ] Image previews for thumbnails
- [ ] Approve/Reject/Approve All buttons
- [ ] Real-time progress updates

### Tag Analytics Tab
- [ ] Bar chart of tag frequency
- [ ] Tag cloud visualization
- [ ] Orphan tags list
- [ ] Duplicate detection

### History Tab
- [ ] List of processed items
- [ ] Undo button per item
- [ ] Bulk undo
- [ ] Export changelog

### Settings Tab
- [ ] Live config editing
- [ ] Save config
- [ ] Test connection buttons
- [ ] View logs

---

## Phase 5: Advanced Features

### State Management
- [ ] Implement #processed-by-script tagging
- [ ] Resume capability (skip already processed)
- [ ] Cleanup tags command

### Manual Approval Workflow
- [ ] Implement approval queue
- [ ] UI integration for approval
- [ ] Batch approval
- [ ] Skip/Reject handling

### Dry-Run Mode
- [ ] Implement dry-run flag
- [ ] Show proposed changes without applying
- [ ] Log all operations
- [ ] Summary report

### Undo Mechanism
- [ ] Implement undo by raindrop ID
- [ ] Load from backup JSON
- [ ] Restore original values via API
- [ ] Remove #processed-by-script tag
- [ ] Update history status to 'reverted'

### Cron Job Integration
- [ ] Implement macOS launchd plist generator
- [ ] Install-cron command
- [ ] Uninstall-cron command
- [ ] Test scheduled execution
- [ ] Logging for autonomous runs

---

## Phase 6: Testing & Polish

### Unit Tests
- [ ] Test API wrapper functions
- [ ] Test database operations
- [ ] Test config loading
- [ ] Test backup/restore
- [ ] Test title cleanup patterns
- [ ] Test tag optimization logic

### Integration Tests
- [ ] Test end-to-end processing (demo mode)
- [ ] Test with real Raindrop account (sandbox)
- [ ] Test Gradio UI functionality
- [ ] Test cron job execution

### Documentation
- [ ] Update README with usage examples
- [ ] Document config.yaml options
- [ ] Create troubleshooting guide
- [ ] Add API documentation
- [ ] Create video demo

### Performance Optimization
- [ ] Profile processing speed
- [ ] Optimize SQLite queries
- [ ] Reduce API calls where possible
- [ ] Test with large collections (1000+ items)

---

## 🎯 Success Metrics

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

## 🚧 Future Enhancements (Post-MVP)

### Reach Goals
- [ ] Additional thumbnail sources (tools for other domains)
- [ ] Machine learning for user approval patterns
- [ ] Duplicate detection and merging
- [ ] Smart collection organization
- [ ] Export/import processing rules
- [ ] Full web interface (beyond Gradio)
- [ ] Multi-user support
- [ ] Analytics dashboard

### Linux Support
- [ ] Test on Linux
- [ ] Create standard cron job setup
- [ ] Handle platform-specific paths

---

## 📝 Notes

- Focus on demo mode first to validate the entire pipeline
- Use dry-run extensively during development
- Test with a backup Raindrop collection before production
- Monitor API rate limits closely
- Keep backups for at least 30 days during development

---

*Generated: 2025-10-19*
*Based on: PROJECT SPEC.md*
