---
layout: post
title: "What Needs Attention: Gaps, Risks, and Maintenance Issues"
date: 2026-01-10
categories: infrastructure
series: "Skill Infrastructure Breakdown"
part: 10
---

# What Needs Attention: Gaps, Risks, and Maintenance Issues

**Date:** January 4, 2026
**Series:** Skill Infrastructure Breakdown
**Part:** 10 of 10

---

## Five Areas Requiring Attention

### 1. Backup Path Standards Violation

**Issue:** Browser skill (and possibly others) violates PAI CORE standards for backup location.

**Current:** Skills create backup directories inside themselves:
```
skill-dir/Backups/browser-skill-backup-2026-01-04.tar.gz
```

**Required:** Use centralized backup location:
```
$PAI_DIR/history/backups/browser-skill-2026-01-04.tar.gz
```

**Action:** Quick audit of all skills for backup path violations. Update installations.

**Severity:** Medium (governance issue, not functional)

---

### 2. Environment Variable Validation

**Issue:** Skill installation assumes `$PAI_DIR` is set but doesn't verify.

**Current behavior:**
```bash
if $PAI_DIR is unset:
  # Script tries to create /skills/my-skill/
  # Creates in wrong location, confusing error
```

**Required:**
```bash
if [ -z "$PAI_DIR" ]; then
    echo "Error: \$PAI_DIR not set. Add it to the shell profile."
    exit 1
fi
```

**Action:** Add validation to all installation scripts.

**Severity:** Low (mostly affects initial setup)

---

### 3. Cron Job Auditing

**Issue:** 3-day indexing outage wasn't noticed.

**Current state:** Indexer runs every 30 minutes. If it fails, nobody knows.

**Recommendations:**

```bash
# 1. Check index staleness
if [ $(find . -name skill-index.json -mmin +60) ]; then
    alert "Skill index is stale"
fi

# 2. Add explicit logging
*/30 * * * * /path/to/indexer >> /var/log/skill-indexer.log 2>&1

# 3. Monthly audit checklist
# - Does indexer run consistently?
# - Is index timestamp recent?
# - Did new skills get picked up?
```

**Action:** Implement monthly cron job audits. Add staleness detection.

**Severity:** High (silent failures compound)

---

### 4. Memory System Database Warnings

**Issue:** `PGVector` implementation shows deprecation warnings:

```
LangChainDeprecationWarning: The class `OllamaEmbeddings` was
deprecated in LangChain 0.3.1 and will be removed in 1.0.0
```

```
LangChainPendingDeprecationWarning: This class is pending deprecation
```

**Current:** System works. Warnings are non-critical.

**Long-term:** Vector storage architecture needs migration planning.

**Action:**
- Document deprecation warnings
- Plan migration to `langchain-ollama` package
- Test compatibility with new implementations
- Schedule migration (non-urgent, but queue it)

**Severity:** Low (works now, maintenance for future)

---

### 5. Documentation Completeness

**Issue:** Browser skill installation documentation assumes agent-based prompting but doesn't provide tooling.

**Current:**
```markdown
"Before proceeding, confirm with the user:
1. Backup location acceptable?
2. Conflict resolution approach?
```

**Problem:** There's no `AskUserQuestion` integration. The prompts aren't actually there.

**Action:** Either:
- Add `AskUserQuestion` integration for agent-based prompting, OR
- Remove the assumptions and provide default behavior

Pick one approach and document it clearly.

**Severity:** Medium (UX inconsistency)

---

## Priority Matrix

| Issue | Severity | Urgency | Effort |
|-------|----------|---------|--------|
| Backup path standards | Medium | Medium | Low |
| Environment validation | Low | Low | Low |
| Cron job auditing | High | High | Medium |
| Database deprecations | Low | Low | High |
| Documentation completeness | Medium | Low | Medium |

---

## Recommended Action Timeline

### This Week
- [ ] Audit all skills for backup path violations
- [ ] Add `$PAI_DIR` validation to installation scripts
- [ ] Set up cron job staleness detection

### This Month
- [ ] Implement monthly cron audit checklist
- [ ] Document current migration path for PGVector
- [ ] Decide on browser skill prompting approach

### This Quarter
- [ ] Execute PGVector migration (plan and test)
- [ ] Review all background jobs for silent failure risks

---

## The Bigger Picture

These aren't architectural problems. They're maintenance and governance issues.

**The good news:** My infrastructure is fundamentally sound. These gaps don't break anything.

**The bad news:** They'll compound over time if not addressed. Background jobs fail silently. Standards drift. Warnings accumulate.

The difference between infrastructure that works for 1 year and infrastructure that works for 5 years is attention to these "minor" maintenance items.

---

## Conclusion

My skill infrastructure is mature. It discovers capabilities automatically. It extracts and stores learned insights. It organizes metadata pragmatically. It indexes skills at scale.

The foundation is solid.

Now it's about maintenance, governance, and staying ahead of technical debt.

The gaps identified here are actionable. None of them are blocking issues. But addressing them systematically will keep my system healthy and maintainable as it scales.

---

**Series Complete.** This infrastructure breakdown covered:
1. Three-layer architecture
2. Skill system design
3. Automatic skill discovery
4. Cron job auditing story
5. Browser skill architecture
6. Semantic memory system
7. Timestamp-based file organization
8. Admin infrastructure
9. What works well
10. What needs attention

*Next steps: Pick the highest-priority maintenance items and integrate them into my workflow.*
