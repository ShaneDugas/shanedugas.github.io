---
layout: post
title: "The Skill System: 27 Capabilities, Elegantly Organized"
date: 2026-01-04
categories: infrastructure
series: "Skill Infrastructure Breakdown"
part: 2
---

# The Skill System: 27 Capabilities, Elegantly Organized

**Date:** January 4, 2026
**Series:** Skill Infrastructure Breakdown
**Part:** 2 of 10

---

## Overview

My skills directory contains **27 distinct capabilities**, each in its own isolated directory with a `SKILL.md` file defining its contract. This modular approach is what makes the system scalable.

---

## Core Design Principle

Each skill:
- Lives in its own directory
- Has a `SKILL.md` interface contract
- Declares its "triggers" - keywords or intents that activate it
- Can be loaded independently without affecting others

---

## Skills Currently Installed

### Always-Loaded (Core Foundation)
- `CORE` - Identity, response format, mandatory principles

### Deferred (Loaded on Demand)
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
- **Plus 13 others...**

---

## How Trigger-Based Loading Works

Each skill declares its triggers in the `SKILL.md` file using the `USE WHEN` format:

```markdown
description: Process YouTube playlists into structured learning content.
USE WHEN user mentions YouTube videos, learning queue, OR wants to process transcripts.
```

When I ask a question, the system:
1. Extracts keywords from my question
2. Matches them against all skill triggers
3. Loads only the skills that match
4. Presents those capabilities to me

This is **low-friction capability addition** - no registration, no configuration updates, no manual discovery needed.

---

## Why This Matters

Without modular skills:
- Adding a new capability requires manual registration
- Hardcoded imports everywhere
- Configuration files sprawl
- Skills become invisible or accidentally unused

With the modular skill system:
- Add a skill directory with `SKILL.md` ✓
- System discovers it automatically ✓
- Triggers determine when it loads ✓
- No manual intervention needed ✓

---

## Next Steps

The 27 skills are managed automatically through an indexing system that scans the directory every 30 minutes and builds a searchable index. This enables true skill discovery at scale.

*Next: The Skill Index - How automatic discovery actually works.*
