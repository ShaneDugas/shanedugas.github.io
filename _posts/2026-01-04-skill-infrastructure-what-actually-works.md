---
layout: post
title: "The Skill Infrastructure That Actually Works: Architecture, Indexing, and What We Got Right"
date: 2026-01-04
categories: infrastructure
---

**Date:** January 4, 2026
**Total Skills Indexed:** 27
**System Status:** Mostly working, with critical path fixed
**Last Indexing Run:** Every 30 minutes (now that cron is fixed)

---

## Introduction: The Infrastructure That Shipped

While the YouTube Learning Queue became a case study in what doesn't work, the underlying skill infrastructure and admin tooling have quietly been delivering value.

This is the story of what *did* work.

---

## The Architecture: Three-Layer Knowledge Management

My personal AI infrastructure evolved into a sophisticated three-layer system for managing knowledge and capability:

### Layer 1: Session Layer (Real-Time)
- Captures current work as it happens
- Tracks decisions, tool usage, context
- Stored in memory during active session
- Feeds into downstream systems

### Layer 2: History Layer (File-Based Metadata)
- Session summaries with ISO 8601 timestamps
- Learning documents (insights extracted from conversations)
- Structured file organization by date
- Searchable via filesystem and grep
- Purpose: "What did I do and when?"

### Layer 3: Memory Layer (Semantic Extraction)
- Postgres + LangChain vector database
- Semantic embeddings for similarity search
- Extracted insights, patterns, decisions, gaps
- Queryable via `search-memories.py`
- Purpose: "What did I learn?"

**Why three layers?** Each serves different query patterns:
- Session layer answers: "What am I doing right now?"
- History layer answers: "What happened on 2026-01-03?"
- Memory layer answers: "What did I learn about YouTube queue architecture?"

This redundancy is intentional. It's not a design flaw; it's a feature.

---

## The Skill System: 27 Capabilities, Elegantly Organized

My skills directory contains **27 distinct capabilities**, each in its own isolated directory with a `SKILL.md` file defining its contract.

### Skills Currently Installed

**Always-Loaded (Core Foundation):**
- `CORE` - Identity, response format, mandatory principles

**Deferred (Loaded on Demand):**
- `Agents` - Dynamic agent composition and management
- `Art` - Creative/artistic capabilities
- `Prompting` - Meta-prompting and template generation
- `SocraticTutor` - Learning verification through questioning
- `Browser` - File-based Playwright automation (NEW)
- `CreateSkill` - Skill generation framework
- `CreateCLI` - Command-line tool scaffolding
- `Fabric` - Fabric integration patterns
- `Email` - Email handling
- `Observability` - System observability and monitoring
- `langchain-postgres-memory` - Semantic memory system
- `youtube-learning-queue` - YouTube playlist processing
- Plus 13 others...

Each skill declares its "triggers" - the keywords or intents that activate it. The skill system uses these triggers to determine which capabilities to load.

---

## The Skill Index: Discovery at Scale

With 27+ skills, I needed a discovery mechanism. Enter `GenerateSkillIndex.ts`:

```typescript
// Parses all SKILL.md files and builds a searchable index
// Usage: bun run $PAI_DIR/Tools/GenerateSkillIndex.ts
```

### What It Does

1. **Discovers Skills**
   - Scans `skills/` directory recursively
   - Finds all `SKILL.md` files
   - Extracts metadata from frontmatter

2. **Parses Intent Triggers**
   - Reads `description` field
   - Extracts keywords from `USE WHEN` clauses
   - Builds trigger list for skill activation
   - Example: "USE WHEN user mentions YouTube videos, learning queue, OR wants to process transcripts" → triggers = ["youtube", "learning", "queue", "transcripts", ...]

3. **Classifies Skills**
   - `always` tier: Core capabilities loaded every session (CORE only)
   - `deferred` tier: Loaded on-demand when triggers match
   - Smart resource allocation: Don't load everything, load what's needed

4. **Generates Index**
   - Output: `skill-index.json`
   - Contains: metadata, paths, triggers, tier classification
   - Updated automatically every 30 minutes via cron

### The Index Structure

```json
{
  "generated": "2026-01-04T14:55:43.103Z",
  "totalSkills": 27,
  "alwaysLoadedCount": 1,
  "deferredCount": 26,
  "skills": {
    "youtube-learning-queue": {
      "name": "youtube-learning-queue",
      "path": "youtube-learning-queue/SKILL.md",
      "fullDescription": "Process YouTube playlists into structured learning content...",
      "triggers": ["youtube", "learning", "queue", "videos", "transcripts"],
      "workflows": [],
      "tier": "deferred"
    },
    // 26 more skills...
  }
}
```

### Why This Matters

Without skill indexing, adding a new capability requires:
- Manual registration somewhere
- Hardcoded imports
- Configuration file updates
- Risk of skills becoming invisible/unusable

With automatic indexing:
- Add a skill directory with `SKILL.md` ✓
- Next indexing run picks it up ✓
- Immediately discoverable ✓
- No manual registration needed ✓

This is low-friction capability addition.

---

## The Indexer Cron Job: The Fix That Wasn't Obvious

**Problem Found:** The skill indexer wasn't running for 3 days.

**Timeline:**
- Jan 1, 9:30 AM: Cron job failed silently
- Jan 1 → Jan 4: No indexing for 72 hours
- New skills added during this period: invisible
- System acting on stale index

**Root Cause:** Wrong path in cron configuration

```bash
# BROKEN (path didn't exist):
*/30 * * * * /home/smd/bex/.claude/skills/langchain-postgres-memory/scripts/...

# CORRECT:
*/30 * * * * /home/smd/bex/skills/langchain-postgres-memory/scripts/...
```

Someone had moved the skills directory from `.claude/skills/` to `skills/` but didn't update the cron job. The system was looking in the wrong place every 30 minutes, failing silently, and nobody noticed.

**The Fix:**
```bash
✅ Updated cron path to correct location
✅ Indexer now runs every 30 minutes: */30 * * * *
✅ Scheduled: :00 and :30 of every hour
```

**Lesson:** Systems fail silently when they're automated. The indexer has no user-facing output. It just runs, succeeds or fails in logs nobody reads, and either updates or doesn't.

Regular auditing of background jobs is non-negotiable.

---

## The Browser Skill: File-Based Architecture That Actually Works

Recent addition: `kai-browser-skill` - a file-based Playwright automation system.

### Why It's Different

Most browser automation in AI systems uses MCPs (Model Context Protocol) that:
- Load massive capability sets (~13,700 tokens startup overhead)
- Generate code on each operation
- Create boilerplate-heavy interaction models

This skill uses a different approach:
- **File-based CLI**: Pre-written commands, not generated code
- **Three simple operations**: screenshot, verify, open
- **99%+ token savings** vs. traditional browser MCPs
- **Zero code generation** required

### Architecture

```
kai-browser-skill/
├── bin/
│   └── browser-cli.ts          (Main command handler)
├── src/
│   ├── screenshot.ts           (Page capture)
│   ├── verify.ts               (Page verification)
│   └── open.ts                 (URL navigation)
├── SKILL.md                    (Interface contract)
├── INSTALL.md                  (Setup procedure)
└── examples/
    └── verify-page.ts          (Usage example)
```

### Commands

```bash
# Take screenshot
bun browser-cli.ts screenshot https://example.com

# Verify page contains content
bun browser-cli.ts verify https://example.com --text "hello"

# Open URL
bun browser-cli.ts open https://example.com
```

No complex UI interaction framework. No DOM traversal APIs. Just: screenshot, verify content exists, open URL. Covers 80% of actual use cases.

### Issues Found During Installation

When I evaluated this for installation, I found:

**1. Backup Path Conflict** (Critical)
```bash
# WRONG (violates PAI standards):
BACKUP_DIR="$PAI_DIR/Backups/browser-skill-..."

# RIGHT (follows PAI CORE standards):
BACKUP_DIR="$PAI_DIR/history/backups/browser-skill-..."
```

My CORE skill clearly states: "Never create `backups/` directories inside skills." The Browser skill installation creates exactly that. This is a standards violation that needs fixing before deployment.

**2. Interactive Prompts Assumption** (UX Issue)
The installation documentation says:
```
"Before proceeding, confirm with the user:
1. Backup location acceptable?
2. Conflict resolution approach?
3. Browser preference?"
```

But there's no actual interactive prompt mechanism. Users following the documentation manually get silent defaults instead of being asked questions. The instructions are agent-centric but don't provide agent-facing tooling.

**3. Environment Variable Validation** (Minor)
The entire pack assumes `$PAI_DIR` is set. If it's not, I get cryptic path errors. This should validate upfront:

```bash
if [ -z "$PAI_DIR" ]; then
    echo "Error: \$PAI_DIR not set"
    exit 1
fi
```

**Overall Assessment:** Excellent architecture and implementation. The issues are configuration/documentation problems, not architectural ones.

---

## The Semantic Memory System: Actually Useful

My semantic memory system (Postgres + LangChain) provides a critical capability: extracting and searching *insights* from conversations, not just metadata.

### How It Works

1. **Extraction:** After each session, identified insights are extracted:
   - `type`: learning, insight, gap, decision, pattern, commitment, workflow, correction
   - `content`: The actual insight text
   - `confidence`: How sure we are (0-1)
   - `entities`: What topics this relates to

2. **Storage:** Stored in Postgres with vector embeddings
   - Semantic similarity search (not keyword matching)
   - Enables pattern recognition across conversations
   - Queryable by topic, type, confidence

3. **Retrieval:** Search across all historical learnings
```bash
# Find insights about YouTube queue
PGPASSWORD=... python3 search-memories.py "YouTube learning queue"

# Filter by type
PGPASSWORD=... python3 search-memories.py "YouTube" --type gap --limit 10

# Filter by confidence
PGPASSWORD=... python3 search-memories.py "infrastructure" --min-confidence 0.7
```

### Why This Beats Session Logs

Session logs are metadata:
- "What happened when"
- "Which files changed"
- "What decisions were made"

Memory insights are intelligence:
- "What pattern did we discover?"
- "Why did that approach fail?"
- "What assumption was wrong?"

When I ask "what did we learn about YouTube queue authentication?" the memory system returns extracted insights from across multiple sessions. Session logs would return filenames and timestamps.

This system actually works and delivers value.

---

## The History System: Timestamp-Based Organization

File organization evolved to ISO 8601 timestamps:

```
history/
├── sessions/2026-01/
│   ├── 20260104T134953_SESSION_description-slug.md
│   └── 20260104T131657_SESSION_different-task.md
├── learnings/2026-01/
│   ├── 20260104T192518_LEARNING_insight-from-session.md
│   └── 20260104T142235_LEARNING_different-insight.md
└── research/2026-01/
    └── 20260104T153853_RESEARCH_investigation-results.md
```

### Benefits of Timestamp Organization

1. **Chronological Queries:** `ls -lt` automatically sorts by recency
2. **Date-Range Searches:** Files naturally group by YYYYMM
3. **No Fragmentation:** Don't need to decide "Is this AI? Security? YouTube?"
4. **Searchable:** `grep -r "topic" history/2026-01/` works naturally
5. **Deduplication:** Timestamps ensure unique names

### Trade-offs Accepted

I *could* organize by topic (Learning/, Security/, AI/). But that requires:
- Consensus on categorization schemes
- Regular refactoring as categorization improves
- Someone deciding where things go
- Eventual fragmentation (things fit multiple categories)

Timestamp organization says: "Let's just record what happened, when. Users can find it via search."

This is pragmatic. It works.

---

## The Admin Infrastructure That Actually Runs

### Observability System

My observability server provides real-time visibility into system state:
- WebSocket streaming of events
- Vue 3 dashboard
- File-based event tracking

One limitation: hardcoded localhost. This prevents remote access. The pragmatic solution implemented: SSH tunneling.

Rather than modifying the code (risky, unmaintainable), forward the port:
```bash
ssh -L 27123:localhost:27123 remote-host
# Then access locally: http://localhost:27123
```

This preserves external package integrity while enabling access. Excellent pragmatism.

### Documentation System

Session and learning captures happen automatically via stop-hooks. Every execution produces timestamped documentation.

The consolidation work (Jan 3) moved loose files to organized structure:
- 24 export files consolidated with ISO timestamps
- Root directory cleaned up
- Better findability

Simple. Effective. Ongoing.

---

## What We Got Right

### 1. Skill Modularity
Each skill is isolated, self-contained, and discoverable. Adding a new skill requires no registration, no configuration changes, no hardcoding. Just add a directory with `SKILL.md`.

### 2. Automatic Indexing
The skill index regenerates every 30 minutes. New skills become discoverable without manual intervention. This scales.

### 3. Semantic Intelligence
The memory system extracts actual insights, not just metadata. I can search for "what did I learn?" not just "what files were created?"

### 4. Pragmatic Organization
Timestamp-based file organization avoids categorization friction while remaining searchable and sortable.

### 5. Silent Operation
The indexer runs every 30 minutes producing no user-facing noise. It just works until it doesn't. (And then we discovered the cron issue.)

### 6. Minimal Dependencies
Skills use simple `SKILL.md` markdown contracts. The indexer is ~100 lines of TypeScript. No heavyweight frameworks.

---

## What Needs Attention

### 1. Backup Path Standards
Browser skill (and possibly others) violates PAI CORE standards for backup location. Quick audit needed.

### 2. Environment Variable Validation
Skill installation should verify `$PAI_DIR` is set before proceeding. Currently assumes it and fails cryptically.

### 3. Cron Job Auditing
3-day indexing outage wasn't noticed. Recommend:
- Monthly audit of all cron jobs
- Add logging verification (not just "does it run?")
- Alert on stale index (timestamp > 1 hour old)

### 4. Memory System Database Warnings
The `PGVector` implementation shows deprecation warnings. The underlying vector storage architecture needs migration planning.

### 5. Documentation Completeness
Browser skill installation assumes agent-based prompting but doesn't provide AskUserQuestion integration. Either add it or remove the assumptions.

---

## The System as a Whole

This infrastructure is mature. It:
- ✅ Discovers capabilities automatically
- ✅ Extracts and stores learned insights
- ✅ Organizes metadata pragmatically
- ✅ Indexes skills at scale
- ✅ Provides semantic search over learnings
- ✅ Runs background jobs (mostly) silently

It has issues:
- ⚠️ Background job auditing needed
- ⚠️ Standards violations in recent skills
- ⚠️ Database deprecation warnings

But these are maintenance issues, not architectural problems.

The skill infrastructure actually shipped. It works. It scales. And unlike the YouTube queue, it delivers on its promises.

---

## Conclusion: What Separates Working from Non-Working

**YouTube Learning Queue:** Promised end-to-end automation, delivers 40% with multiple failure points

**Skill Infrastructure:** Promises skill discovery and automated indexing, delivers exactly that

The difference:
1. **Scope clarity** - Skills indexing knows what it's trying to do (find SKILL.md files, extract metadata, build index). YouTube queue tried to do 6 complex things simultaneously.

2. **Separation of concerns** - Indexing doesn't try to download, process, or store content. It indexes. Separately, other systems handle other tasks.

3. **Graceful degradation** - If the indexer fails for a day, the worst-case is manual skill activation. If YouTube processing fails, I get incomplete files in the wrong location.

4. **Testing** - The indexer is testable: give it a directory of skills, verify it produces correct JSON. YouTube queue required YouTube's cooperation, MAMA_LLAMA responsiveness, vault permissions, etc.

5. **Visibility** - Indexer runs predictably. Cron log shows when it ran. Output is a JSON file I can inspect. YouTube queue has silent hangs and timeout errors.

The admin infrastructure that works is the infrastructure that knows its scope, stays within it, and provides visibility into what it's doing.

---

**Next in the Series:** Production lessons from PAI and what "actually working" looks like at scale.

*This infrastructure will handle 100 skills as easily as 27. The YouTube queue can't handle two playlists reliably. There's a lesson there.*
