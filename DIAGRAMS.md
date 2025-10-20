## 1. Complete System Architecture

```mermaid
graph TB
    subgraph "User Interface Layer"
        CLI[CLI Commands]
        UI[Gradio UI - Optional]
    end

    subgraph "Orchestration Layer"
        ORCH[Main Orchestrator<br/>Claude Sonnet 4.5]
        CONFIG[Config Manager<br/>YAML Loader]
        STATE[State Manager<br/>Tag Tracking]
    end

    subgraph "Agent Layer"
        TAG_AGENT[Tag Analyzer<br/>Haiku Subagent]
        WORKER1[Worker 1<br/>Haiku]
        WORKER2[Worker 2<br/>Haiku]
        WORKER3[Worker 3<br/>Haiku]
        WORKER4[Worker 4<br/>Haiku]
        WORKER5[Worker 5<br/>Haiku]
    end

    subgraph "Data Layer"
        API[Raindrop API<br/>Wrapper]
        DB[(SQLite DB<br/>Tag Analytics)]
        BACKUP[Backup Storage<br/>JSON Files]
    end

    subgraph "External Services"
        RAINDROP[Raindrop.io<br/>API]
        YT_THUMBS[yt-thumbs<br/>UV Tool]
    end

    CLI --> ORCH
    UI --> ORCH
    ORCH --> CONFIG
    ORCH --> STATE
    ORCH --> TAG_AGENT
    ORCH --> WORKER1
    ORCH --> WORKER2
    ORCH --> WORKER3
    ORCH --> WORKER4
    ORCH --> WORKER5
    
    TAG_AGENT --> DB
    WORKER1 --> DB
    WORKER2 --> DB
    WORKER3 --> DB
    WORKER4 --> DB
    WORKER5 --> DB
    
    ORCH --> API
    ORCH --> BACKUP
    API --> RAINDROP
    
    WORKER1 --> YT_THUMBS
    WORKER2 --> YT_THUMBS
    WORKER3 --> YT_THUMBS
    WORKER4 --> YT_THUMBS
    WORKER5 --> YT_THUMBS

    style ORCH fill:#e1f5ff
    style TAG_AGENT fill:#fff3cd
    style WORKER1 fill:#d4edda
    style WORKER2 fill:#d4edda
    style WORKER3 fill:#d4edda
    style WORKER4 fill:#d4edda
    style WORKER5 fill:#d4edda
    style DB fill:#f8d7da
    style RAINDROP fill:#cfe2ff
```

## 2. Processing Pipeline - Detailed Flow

```mermaid
flowchart TD
    START([Start Processing]) --> INIT[Initialize System]
    INIT --> LOAD_CONFIG[Load config.yaml]
    LOAD_CONFIG --> INIT_API[Initialize Raindrop API]
    INIT_API --> INIT_DB[Initialize SQLite DB]
    INIT_DB --> TAG_ANALYSIS{Tag Analysis<br/>Enabled?}
    
    TAG_ANALYSIS -->|Yes| RUN_ANALYZER[Spawn Tag Analyzer<br/>Subagent]
    TAG_ANALYSIS -->|No| FETCH
    RUN_ANALYZER --> ANALYZE[Analyze All Tags<br/>Calculate Frequency]
    ANALYZE --> STORE_TAGS[Store in SQLite]
    STORE_TAGS --> FETCH
    
    FETCH[Fetch Raindrops from API] --> FILTER[Filter Out<br/>#processed-by-script]
    FILTER --> CHECK_BATCH{More Items<br/>to Process?}
    
    CHECK_BATCH -->|No| POST_PROCESS
    CHECK_BATCH -->|Yes| CREATE_BATCH[Create Batch of 5]
    CREATE_BATCH --> BACKUP_BATCH[Create JSON Backups]
    BACKUP_BATCH --> SPAWN[Spawn 5 Haiku Workers]
    
    SPAWN --> W1[Worker 1:<br/>Process Item 1]
    SPAWN --> W2[Worker 2:<br/>Process Item 2]
    SPAWN --> W3[Worker 3:<br/>Process Item 3]
    SPAWN --> W4[Worker 4:<br/>Process Item 4]
    SPAWN --> W5[Worker 5:<br/>Process Item 5]
    
    W1 --> AGG[Aggregate Results]
    W2 --> AGG
    W3 --> AGG
    W4 --> AGG
    W5 --> AGG
    
    AGG --> APPROVAL{Manual Approval<br/>Required?}
    
    APPROVAL -->|Yes| SHOW_UI[Display in UI]
    SHOW_UI --> WAIT[Wait for User Decision]
    WAIT --> USER_DECISION{Approved?}
    USER_DECISION -->|No| SKIP[Skip Changes]
    USER_DECISION -->|Yes| APPLY
    
    APPROVAL -->|No Auto| APPLY[Apply Changes via API]
    
    APPLY --> ADD_TAG[Add #processed-by-script]
    ADD_TAG --> SAVE_HISTORY[Save to processing_history]
    SAVE_HISTORY --> UPDATE_PROGRESS[Update Progress Bar]
    UPDATE_PROGRESS --> RATE_LIMIT[Rate Limit Check]
    RATE_LIMIT --> CHECK_BATCH
    
    SKIP --> UPDATE_PROGRESS
    
    POST_PROCESS[Post-Processing] --> GEN_REPORT[Generate Summary Report]
    GEN_REPORT --> UPDATE_TAG_FREQ[Update Tag Frequencies]
    UPDATE_TAG_FREQ --> CLEANUP_DECISION{Cleanup Tags<br/>Requested?}
    CLEANUP_DECISION -->|Yes| REMOVE_TAGS[Remove All<br/>#processed-by-script]
    CLEANUP_DECISION -->|No| EXPORT
    REMOVE_TAGS --> EXPORT[Export Changelog]
    EXPORT --> END([End])

    style START fill:#e1f5ff
    style END fill:#d4edda
    style W1 fill:#fff3cd
    style W2 fill:#fff3cd
    style W3 fill:#fff3cd
    style W4 fill:#fff3cd
    style W5 fill:#fff3cd
    style APPLY fill:#d4edda
    style SKIP fill:#f8d7da
```

## 3. Worker Subagent Processing Logic

```mermaid
flowchart TD
    START([Receive Raindrop]) --> ANALYZE[Analyze Current State]
    ANALYZE --> QUERY_DB[Query SQLite for<br/>Existing Tags]
    QUERY_DB --> TITLE[Title Cleanup]
    
    TITLE --> REMOVE_PREFIX[Remove Site Prefixes]
    REMOVE_PREFIX --> SHORTEN[Shorten to 80 chars]
    SHORTEN --> OBSIDIAN[Make Obsidian-Friendly]
    
    OBSIDIAN --> THUMBNAIL{Is YouTube<br/>URL?}
    THUMBNAIL -->|Yes| YT_THUMBS[Call yt-thumbs Tool]
    THUMBNAIL -->|No| KEEP_THUMB[Keep Existing Thumbnail]
    YT_THUMBS --> DESC
    KEEP_THUMB --> DESC
    
    DESC[Description Enhancement] --> CHECK_DESC{Description<br/>Generic?}
    CHECK_DESC -->|Yes| REWRITE[Rewrite Based on Content]
    CHECK_DESC -->|No| KEEP_DESC[Keep Description]
    REWRITE --> TAG_OPT
    KEEP_DESC --> TAG_OPT
    
    TAG_OPT[Tag Optimization] --> ANALYZE_TAGS[Analyze Content for Tags]
    ANALYZE_TAGS --> MATCH_EXISTING[Match Against<br/>Existing Tags]
    MATCH_EXISTING --> SIMILARITY{Similarity ≥ 0.7?}
    
    SIMILARITY -->|Yes| REUSE_TAG[Reuse Existing Tag]
    SIMILARITY -->|No| NEW_TAG_CHECK{Will Tag Be<br/>Reused?}
    
    NEW_TAG_CHECK -->|Likely| CREATE_NEW[Create New Tag]
    NEW_TAG_CHECK -->|Unlikely| SKIP_TAG[Skip Tag Creation]
    
    REUSE_TAG --> TAG_COUNT{Tag Count<br/> ≤ 5?}
    CREATE_NEW --> TAG_COUNT
    SKIP_TAG --> TAG_COUNT
    
    TAG_COUNT -->|Yes| MORE_TAGS{More Tags<br/>to Process?}
    TAG_COUNT -->|No| BUILD_RESULT
    MORE_TAGS -->|Yes| ANALYZE_TAGS
    MORE_TAGS -->|No| BUILD_RESULT
    
    BUILD_RESULT[Build Result JSON] --> RETURN([Return to Orchestrator])

    style START fill:#e1f5ff
    style RETURN fill:#d4edda
    style YT_THUMBS fill:#cfe2ff
    style REUSE_TAG fill:#d4edda
    style CREATE_NEW fill:#fff3cd
    style SKIP_TAG fill:#f8d7da
```

## 4. State Management & Resume Flow

```mermaid
stateDiagram-v2
    [*] --> Unprocessed: New Raindrop
    
    Unprocessed --> Processing: Selected for Batch
    
    Processing --> Analyzed: Worker Subagent Completes
    
    Analyzed --> PendingApproval: Manual Mode
    Analyzed --> ReadyToApply: Auto Mode
    
    PendingApproval --> Approved: User Approves
    PendingApproval --> Rejected: User Rejects
    
    Rejected --> Unprocessed: Skip Changes
    
    Approved --> ReadyToApply
    
    ReadyToApply --> Applied: API Update Success
    ReadyToApply --> Failed: API Error
    
    Applied --> Tagged: Add #processed-by-script
    
    Tagged --> [*]: Complete
    
    Failed --> Retry: Retry Attempt
    Failed --> ManualReview: Max Retries Exceeded
    
    Retry --> ReadyToApply
    
    ManualReview --> [*]
    
    Tagged --> Reverted: Undo Command
    Reverted --> Unprocessed: Restore from Backup
    
    note right of Unprocessed
        No #processed-by-script tag
        Will be picked up in next run
    end note
    
    note right of Tagged
        Has #processed-by-script tag
        Filtered out in future runs
    end note
```

## 5. Data Flow Diagram

```mermaid
flowchart LR
    subgraph "Input Sources"
        RAINDROP_API[Raindrop API<br/>GET /raindrops]
        CONFIG_FILE[config.yaml]
        ENV[Environment Vars]
    end
    
    subgraph "Processing"
        ORCHESTRATOR[Main Orchestrator]
        WORKERS[Worker Pool<br/>5x Haiku]
        TAG_ANALYZER[Tag Analyzer]
    end
    
    subgraph "Storage"
        SQLITE[(SQLite DB)]
        BACKUP_DIR[Backup JSON Files]
        PROCESSING_LOG[Processing History]
    end
    
    subgraph "Output"
        RAINDROP_UPDATE[Raindrop API<br/>PATCH /raindrop/:id]
        UI_DISPLAY[Gradio UI Display]
        REPORT[Summary Report]
    end
    
    RAINDROP_API -->|Raindrop Data| ORCHESTRATOR
    CONFIG_FILE -->|Settings| ORCHESTRATOR
    ENV -->|API Token| ORCHESTRATOR
    
    ORCHESTRATOR -->|Tag Stats Request| TAG_ANALYZER
    TAG_ANALYZER -->|Tag Frequency| SQLITE
    
    ORCHESTRATOR -->|Batch of 5| WORKERS
    WORKERS -->|Query Tags| SQLITE
    SQLITE -->|Tag Recommendations| WORKERS
    
    WORKERS -->|Proposed Changes| ORCHESTRATOR
    ORCHESTRATOR -->|Backup Original| BACKUP_DIR
    ORCHESTRATOR -->|Log Changes| PROCESSING_LOG
    
    ORCHESTRATOR -->|Apply Updates| RAINDROP_UPDATE
    ORCHESTRATOR -->|Display Changes| UI_DISPLAY
    ORCHESTRATOR -->|Final Report| REPORT
    
    PROCESSING_LOG -->|Undo Request| BACKUP_DIR
    BACKUP_DIR -->|Original Data| RAINDROP_UPDATE

    style ORCHESTRATOR fill:#e1f5ff
    style WORKERS fill:#d4edda
    style TAG_ANALYZER fill:#fff3cd
    style SQLITE fill:#f8d7da
```

## 6. Gradio UI Navigation Flow

```mermaid
graph TD
    LAUNCH[Launch UI<br/>--ui flag] --> MAIN_APP[Gradio App]
    
    MAIN_APP --> TAB1[Tab 1: Processing]
    MAIN_APP --> TAB2[Tab 2: Tag Analytics]
    MAIN_APP --> TAB3[Tab 3: History]
    MAIN_APP --> TAB4[Tab 4: Settings]
    
    subgraph "Tab 1: Processing View"
        TAB1 --> LEFT[Left Panel:<br/>Current State]
        TAB1 --> RIGHT[Right Panel:<br/>Proposed Changes]
        LEFT --> TITLE_CUR[Title]
        LEFT --> DESC_CUR[Description]
        LEFT --> TAGS_CUR[Tags]
        LEFT --> THUMB_CUR[Thumbnail]
        RIGHT --> TITLE_NEW[Title ✨]
        RIGHT --> DESC_NEW[Description ✨]
        RIGHT --> TAGS_NEW[Tags ✨]
        RIGHT --> THUMB_NEW[Thumbnail ✨]
        TAB1 --> BUTTONS[Action Buttons]
        BUTTONS --> APPROVE[✓ Approve]
        BUTTONS --> REJECT[✗ Reject]
        BUTTONS --> APPROVE_ALL[✓✓ Approve All]
        TAB1 --> PROGRESS[Progress Bar<br/>15/50 - 30%]
    end
    
    subgraph "Tab 2: Tag Analytics"
        TAB2 --> FREQ_CHART[Frequency Bar Chart]
        TAB2 --> TAG_CLOUD[Tag Cloud]
        TAB2 --> ORPHANS[Orphan Tags List]
        TAB2 --> DUPLICATES[Potential Duplicates]
        TAB2 --> QUERY_INT[SQL Query Interface]
    end
    
    subgraph "Tab 3: History"
        TAB3 --> HISTORY_LIST[Processed Items List]
        HISTORY_LIST --> ITEM_DETAIL[Item Details]
        ITEM_DETAIL --> UNDO_BTN[Undo Button]
        TAB3 --> BULK_UNDO[Bulk Undo]
        TAB3 --> EXPORT_LOG[Export Changelog]
    end
    
    subgraph "Tab 4: Settings"
        TAB4 --> LIVE_CONFIG[Config Editor<br/>YAML]
        TAB4 --> SAVE_CONFIG[Save Changes]
        TAB4 --> TEST_CONN[Test API Connection]
        TAB4 --> VIEW_LOGS[View Logs]
    end

    style MAIN_APP fill:#e1f5ff
    style TAB1 fill:#d4edda
    style TAB2 fill:#fff3cd
    style TAB3 fill:#cfe2ff
    style TAB4 fill:#f8d7da
    style APPROVE fill:#d4edda
    style REJECT fill:#f8d7da
```

## 7. Cron Job Architecture (macOS)

```mermaid
sequenceDiagram
    participant User
    participant CLI
    participant Launchd as macOS launchd
    participant Script as UV Runner
    participant Agent as Orchestrator
    participant Raindrop as Raindrop API
    participant Log as Log Files
    
    User->>CLI: uv run raindrop-cleanup install-cron
    CLI->>CLI: Generate plist file
    CLI->>Launchd: Install ~/Library/LaunchAgents/<br/>com.raindrop.cleanup.plist
    Launchd-->>CLI: ✓ Installed
    CLI-->>User: Cron job scheduled for 3:00 AM
    
    Note over Launchd: Wait until 3:00 AM
    
    Launchd->>Script: Execute at scheduled time
    Script->>Script: Set up environment
    Script->>Agent: uv run raindrop-cleanup all
    
    Agent->>Agent: Initialize system
    Agent->>Raindrop: Fetch raindrops
    Raindrop-->>Agent: Return data
    Agent->>Agent: Process in batches
    Agent->>Raindrop: Apply updates
    Agent->>Log: Write results
    
    Agent-->>Script: Complete with status
    Script-->>Launchd: Exit code 0
    Launchd->>Log: Write stdout/stderr
    
    Note over Launchd: Schedule next run<br/>(tomorrow 3:00 AM)
    
    User->>CLI: uv run raindrop-cleanup uninstall-cron
    CLI->>Launchd: Remove plist
    Launchd-->>CLI: ✓ Removed
    CLI-->>User: Cron job uninstalled
```

## 8. Error Handling & Recovery Flow

```mermaid
graph TD
    START([Operation Begins]) --> TRY[Try Execute]
    
    TRY --> SUCCESS{Success?}
    
    SUCCESS -->|Yes| LOG_SUCCESS[Log Success]
    LOG_SUCCESS --> NEXT[Continue Processing]
    
    SUCCESS -->|No| ERROR_TYPE{Error Type?}
    
    ERROR_TYPE -->|Rate Limit<br/>429| BACKOFF[Exponential Backoff]
    BACKOFF --> WAIT[Wait retry_after seconds]
    WAIT --> RETRY_CHECK{Retry<br/>Attempts < 3?}
    
    ERROR_TYPE -->|Network Error<br/>Timeout| RETRY_CHECK
    
    ERROR_TYPE -->|Invalid Data<br/>400| LOG_ERROR[Log Error]
    LOG_ERROR --> SKIP[Skip Item]
    SKIP --> NEXT
    
    ERROR_TYPE -->|Auth Error<br/>401| CRITICAL[Critical Error]
    CRITICAL --> HALT[Halt Processing]
    HALT --> NOTIFY[Notify User]
    NOTIFY --> END
    
    ERROR_TYPE -->|Subagent<br/>Failure| RESTART_AGENT[Restart Subagent]
    RESTART_AGENT --> RETRY_CHECK
    
    ERROR_TYPE -->|Database<br/>Error| CRITICAL
    
    RETRY_CHECK -->|Yes| INCREMENT[Increment Counter]
    INCREMENT --> TRY
    
    RETRY_CHECK -->|No| MAX_RETRY[Max Retries Reached]
    MAX_RETRY --> LOG_FAIL[Log Failure]
    LOG_FAIL --> SKIP
    
    NEXT --> MORE{More Items?}
    MORE -->|Yes| START
    MORE -->|No| COMPLETE([Complete])
    
    END([Exit with Error])

    style START fill:#e1f5ff
    style COMPLETE fill:#d4edda
    style END fill:#f8d7da
    style CRITICAL fill:#f8d7da
    style LOG_SUCCESS fill:#d4edda
    style BACKOFF fill:#fff3cd
```

## 9. Undo/Rollback Mechanism

```mermaid
sequenceDiagram
    participant User
    participant CLI
    participant Undo Handler
    participant DB as SQLite DB
    participant Backup as Backup Storage
    participant API as Raindrop API
    
    User->>CLI: uv run raindrop-cleanup undo --id 12345
    CLI->>Undo Handler: Execute undo for ID 12345
    
    Undo Handler->>DB: Query processing_history<br/>WHERE raindrop_id = 12345
    DB-->>Undo Handler: Return processing record
    
    Undo Handler->>Undo Handler: Check status = 'applied'
    
    alt Already Reverted
        Undo Handler-->>CLI: Error: Already reverted
        CLI-->>User: ❌ This change was already undone
    else Can Revert
        Undo Handler->>Backup: Load backup JSON<br/>from timestamp
        Backup-->>Undo Handler: Original raindrop data
        
        Undo Handler->>API: PATCH /raindrop/12345<br/>with original data
        API-->>Undo Handler: ✓ Update successful
        
        Undo Handler->>API: Remove #processed-by-script tag
        API-->>Undo Handler: ✓ Tag removed
        
        Undo Handler->>DB: UPDATE processing_history<br/>SET status = 'reverted'
        DB-->>Undo Handler: ✓ Updated
        
        Undo Handler->>Undo Handler: Log undo action
        Undo Handler-->>CLI: Success with change summary
        CLI-->>User: ✓ Successfully reverted changes<br/>Title: "New" → "Old"<br/>Tags: [...] → [...]
    end
```

## 10. Tag Optimization Decision Tree

```mermaid
graph TD
    START([Content Analysis Complete]) --> EXTRACT[Extract Potential Tags<br/>from Content]
    EXTRACT --> HAVE_TAGS{Extracted<br/>Tags?}
    
    HAVE_TAGS -->|No| USE_EXISTING[Use Only Existing Tags<br/>Based on Context]
    HAVE_TAGS -->|Yes| ITERATE[For Each Potential Tag]
    
    ITERATE --> QUERY[Query SQLite for<br/>Similar Existing Tags]
    QUERY --> FOUND{Found<br/>Similar?}
    
    FOUND -->|Yes| CALC_SIM[Calculate Similarity Score]
    CALC_SIM --> SIM_CHECK{Score ≥ 0.7?}
    
    SIM_CHECK -->|Yes| REUSE[✓ Reuse Existing Tag]
    SIM_CHECK -->|No| NEW_TAG_LOGIC
    
    FOUND -->|No| NEW_TAG_LOGIC[Evaluate New Tag Creation]
    
    NEW_TAG_LOGIC --> FREQ_PREDICT{Likely to Be<br/>Reused?}
    FREQ_PREDICT -->|Yes| CHECK_COUNT{Current Tags<br/> < 5?}
    FREQ_PREDICT -->|No| SKIP_TAG[✗ Skip Tag]
    
    CHECK_COUNT -->|Yes| CREATE[✓ Create New Tag]
    CHECK_COUNT -->|No| STOP[Stop Adding Tags]
    
    REUSE --> ADD_TO_LIST[Add to Tag List]
    CREATE --> ADD_TO_LIST
    SKIP_TAG --> MORE
    USE_EXISTING --> ADD_TO_LIST
    
    ADD_TO_LIST --> MORE{More Potential<br/>Tags?}
    MORE -->|Yes| ITERATE
    MORE -->|No| FINAL[Return Final Tag List]
    
    STOP --> FINAL
    FINAL --> END([Complete])

    style START fill:#e1f5ff
    style END fill:#d4edda
    style REUSE fill:#d4edda
    style CREATE fill:#fff3cd
    style SKIP_TAG fill:#f8d7da
    style STOP fill:#f8d7da
```

## 11. Demo Mode Flow

```mermaid
flowchart TD
    START([User Runs Demo Mode]) --> CMD[uv run raindrop-cleanup demo]
    CMD --> UI_FLAG{--ui Flag?}
    
    UI_FLAG -->|Yes| LAUNCH_UI[Launch Gradio Interface]
    UI_FLAG -->|No| CLI_MODE[CLI Mode]
    
    LAUNCH_UI --> UI_READY
    CLI_MODE --> UI_READY
    
    UI_READY[System Ready] --> FETCH_ALL[Fetch All Raindrops]
    FETCH_ALL --> RANDOM[Select 5 Random Items]
    RANDOM --> FILTER_PROC[Filter Out Already Processed]
    FILTER_PROC --> CHECK_COUNT{Got 5<br/>Items?}
    
    CHECK_COUNT -->|No| NEED_MORE[Need More Items]
    NEED_MORE --> RANDOM
    CHECK_COUNT -->|Yes| CREATE_BACKUP
    
    CREATE_BACKUP[Create Backup Directory<br/>./backups/demo_{timestamp}/] --> BACKUP_5[Backup All 5 Items to JSON]
    BACKUP_5 --> SHOW_BACKUP[Display Backup Location]
    SHOW_BACKUP --> SPAWN[Spawn 5 Haiku Workers]
    
    SPAWN --> P1[Process Item 1]
    SPAWN --> P2[Process Item 2]
    SPAWN --> P3[Process Item 3]
    SPAWN --> P4[Process Item 4]
    SPAWN --> P5[Process Item 5]
    
    P1 --> AGG[Aggregate All Results]
    P2 --> AGG
    P3 --> AGG
    P4 --> AGG
    P5 --> AGG
    
    AGG --> MANUAL{Manual<br/>Approval?}
    
    MANUAL -->|Yes UI| SHOW_UI[Display Side-by-Side<br/>in Gradio]
    SHOW_UI --> WAIT[Wait for User Review]
    WAIT --> APPROVE{User<br/>Approves?}
    
    MANUAL -->|Yes CLI| SHOW_CLI[Display in Terminal]
    SHOW_CLI --> PROMPT[Prompt for Approval]
    PROMPT --> APPROVE
    
    MANUAL -->|No Auto| APPLY_ALL[Apply All Changes]
    
    APPROVE -->|Yes| APPLY_ALL
    APPROVE -->|No| SKIP_ITEM[Skip This Item]
    SKIP_ITEM --> MORE_ITEMS{More Items<br/>in Demo?}
    MORE_ITEMS -->|Yes| SHOW_UI
    
    APPLY_ALL --> UPDATE_API[Update via Raindrop API]
    UPDATE_API --> ADD_TAG[Add #processed-by-script]
    ADD_TAG --> MORE_ITEMS
    MORE_ITEMS -->|No| SUMMARY
    
    SUMMARY[Generate Demo Summary] --> SHOW_SUMMARY[Show Results:<br/>- Applied: X/5<br/>- Skipped: Y/5<br/>- Backup Location<br/>- Undo Available]
    SHOW_SUMMARY --> TEST_UNDO{Want to Test<br/>Undo?}
    
    TEST_UNDO -->|Yes| UNDO_PROMPT[Prompt for Item ID]
    UNDO_PROMPT --> RUN_UNDO[Run Undo Command]
    RUN_UNDO --> VERIFY[Verify Restoration]
    VERIFY --> END
    
    TEST_UNDO -->|No| END([Demo Complete])

    style START fill:#e1f5ff
    style END fill:#d4edda
    style CREATE_BACKUP fill:#fff3cd
    style APPLY_ALL fill:#d4edda
    style SKIP_ITEM fill:#f8d7da
```

## 12. Rate Limiting Strategy

```mermaid
graph TD
    REQUEST[API Request Needed] --> CHECK_BUCKET{Token Bucket<br/>Has Capacity?}
    
    CHECK_BUCKET -->|Yes| CONSUME[Consume Tokens]
    CHECK_BUCKET -->|No| CALC_WAIT[Calculate Wait Time]
    
    CONSUME --> MAKE_REQUEST[Make API Call]
    MAKE_REQUEST --> SUCCESS{Success<br/>Status?}
    
    SUCCESS -->|200 OK| UPDATE_STATS[Update Success Stats]
    UPDATE_STATS --> REFILL[Refill Bucket<br/>Background Process]
    REFILL --> COMPLETE
    
    SUCCESS -->|429 Rate Limit| EXTRACT_HEADER[Extract retry-after Header]
    EXTRACT_HEADER --> CALC_WAIT
    
    SUCCESS -->|Other Error| ERROR_HANDLER[Error Handler]
    
    CALC_WAIT --> EXPONENTIAL{Multiple<br/>Retries?}
    EXPONENTIAL -->|Yes| BACKOFF[Exponential Backoff<br/>delay = base × 2^attempts]
    EXPONENTIAL -->|No| USE_HEADER[Use retry-after Value]
    
    BACKOFF --> WAIT_TIMER
    USE_HEADER --> WAIT_TIMER[Sleep for Calculated Time]
    
    WAIT_TIMER --> LOG_WAIT[Log Wait Event]
    LOG_WAIT --> UPDATE_BUCKET[Bucket Refilled During Wait]
    UPDATE_BUCKET --> CHECK_BUCKET
    
    ERROR_HANDLER --> RETRY_LOGIC{Retry<br/>Count < 3?}
    RETRY_LOGIC -->|Yes| CALC_WAIT
    RETRY_LOGIC -->|No| FAIL[Mark as Failed]
    FAIL --> COMPLETE
    
    COMPLETE([Continue Processing])

    style REQUEST fill:#e1f5ff
    style COMPLETE fill:#d4edda
    style WAIT_TIMER fill:#fff3cd
    style FAIL fill:#f8d7da
    style REFILL fill:#cfe2ff
```