---
layout: post
title: "The Indexer Cron Job: Silent Failures and Background Job Auditing"
date: 2026-01-06
categories: infrastructure
series: "Skill Infrastructure Breakdown"
part: 4
---

# The Indexer Cron Job: Silent Failures and Background Job Auditing

**Date:** January 4, 2026
**Series:** Skill Infrastructure Breakdown
**Part:** 4 of 10

---

## The Problem

The skill indexer wasn't running for 3 days. Nobody noticed.

---

## Timeline

- **Jan 1, 9:30 AM:** Cron job failed silently
- **Jan 1 → Jan 4:** No indexing for 72 hours
- **New skills added:** Invisible to the system
- **System state:** Acting on a 3-day-old index

3 days. No alerts. No user-facing errors. Just silent failure.

---

## Root Cause Analysis

Wrong path in the cron configuration:

```bash
# BROKEN (path didn't exist):
*/30 * * * * /home/smd/bex/.claude/skills/langchain-postgres-memory/scripts/...

# CORRECT:
*/30 * * * * /home/smd/bex/skills/langchain-postgres-memory/scripts/...
```

**What happened:** Someone had moved the skills directory from `.claude/skills/` to `skills/` but forgot to update the cron job.

**Result:** Every 30 minutes for 72 hours, the system tried to run a command at a path that didn't exist. It failed silently. No logs that anyone looked at. No visibility.

---

## The Fix

```bash
✅ Updated cron path to correct location
✅ Indexer now runs: */30 * * * *
✅ Scheduled: :00 and :30 of every hour
✅ Index timestamp verified: 2026-01-04T14:55:43.103Z
```

---

## The Lesson

**Systems fail silently when they're automated.**

The indexer has no user-facing output. No notifications. No error messages unless I dig into logs. It just runs at :00 and :30 of every hour, succeeds or fails, and either updates the index or doesn't.

Three days passed before anyone realized it wasn't working.

---

## Why This Matters

When systems run in the background:
- They don't raise alerts on failure
- They don't crash dramatically
- They just... don't run
- Meanwhile, system state drifts

For critical background jobs:
- **Regular auditing is non-negotiable**
- Check timestamps on generated files
- Verify the job actually ran
- Have alerts for stale data

---

## Recommendations

For any background job (cron, scheduled tasks, etc.):

1. **Check timestamp of output**
   ```bash
   stat -c %y skill-index.json
   ```
   If this is older than the cron frequency, something's wrong.

2. **Set up an alert**
   ```bash
   # If index is older than 1 hour, something failed
   if [ $(find . -name skill-index.json -mmin +60) ]; then
     alert "Skill index is stale"
   fi
   ```

3. **Audit after major changes**
   After moving directories, updating paths, or changing schedules: verify the job actually runs.

4. **Log explicitly**
   Add output to the cron job itself (not just system logs):
   ```bash
   */30 * * * * /path/to/indexer.ts >> /tmp/skill-indexer.log 2>&1
   ```

---

## Next Steps

Now that skills are being indexed correctly, how do we actually use them? The next skill we evaluated for installation (the browser skill) revealed interesting architectural decisions.

*Next: The Browser Skill - File-based automation that breaks conventional patterns.*
