---
layout: post
title: "The Skill Index: Discovery at Scale with GenerateSkillIndex.ts"
date: 2026-01-05
categories: infrastructure
series: "Skill Infrastructure Breakdown"
part: 3
---

# The Skill Index: Discovery at Scale with GenerateSkillIndex.ts

**Date:** January 4, 2026
**Series:** Skill Infrastructure Breakdown
**Part:** 3 of 10

---

## The Problem

With 27+ skills, how do I keep track of them all? How does the system know which skills to load? I need a discovery mechanism.

---

## Enter GenerateSkillIndex.ts

```typescript
// Parses all SKILL.md files and builds a searchable index
// Usage: bun run $PAI_DIR/Tools/GenerateSkillIndex.ts
```

This tool automatically discovers and catalogs all my skills.

---

## What It Does

### 1. Discovers Skills
- Scans `skills/` directory recursively
- Finds all `SKILL.md` files
- Extracts metadata from frontmatter

### 2. Parses Intent Triggers
- Reads the `description` field from each skill
- Extracts keywords from `USE WHEN` clauses
- Builds trigger lists for skill activation

**Example:**
```
"USE WHEN user mentions YouTube videos, learning queue, OR wants to process transcripts"
↓
triggers = ["youtube", "learning", "queue", "transcripts", "videos", ...]
```

### 3. Classifies Skills by Tier
- `always` tier: Core capabilities loaded every session (CORE only)
- `deferred` tier: Loaded on-demand when triggers match
- Smart resource allocation: Load what's needed, not everything

### 4. Generates Index
- Output: `skill-index.json`
- Contains: metadata, paths, triggers, tier classification
- Updated automatically every 30 minutes via cron

---

## The Index Structure

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
    "browser": {
      "name": "browser",
      "path": "browser/SKILL.md",
      "fullDescription": "File-based browser automation",
      "triggers": ["browser", "playwright", "automation", "screenshot"],
      "tier": "deferred"
    },
    // 25 more skills...
  }
}
```

---

## Why This Matters

### The Problem Without Indexing

Adding a new capability requires:
- Manual registration somewhere
- Hardcoded imports
- Configuration file updates
- Risk of skills becoming invisible or unusable
- Maintenance burden that grows with each new skill

### The Solution With Indexing

- Add a skill directory with `SKILL.md` ✓
- Next indexing run picks it up ✓
- Immediately discoverable ✓
- No manual registration needed ✓
- Scales to 100+ skills effortlessly

---

## Scale Properties

**Current state:** 27 skills indexed automatically
**Can handle:** 100+ skills with no architectural changes
**Index regeneration:** Every 30 minutes (via cron)
**Startup cost:** Minimal (reads pre-generated index, doesn't regenerate on session start)

This architecture scales.

---

## Next Steps

The indexer runs on a cron schedule every 30 minutes. But what happens when that cron job breaks? That's a story about silent failures and why background job auditing matters.

*Next: The Cron Debugging Story - Why 3 days of silence almost went unnoticed.*
