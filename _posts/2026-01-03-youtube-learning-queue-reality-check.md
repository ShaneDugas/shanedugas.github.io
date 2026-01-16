---
layout: post
title: "The YouTube Learning Queue: How to Build Something That Looks Done But Still Doesn't Work"
date: 2026-01-03
categories: infrastructure
---

**Date:** January 3, 2026
**Status:** "Production Ready" (but it's really not)
**Total Hours Invested:** ~18 hours
**Actual Working Parts:** 40%
**Deployment Status:** Still waiting

---

## The Premise

A simple idea: automatically download my YouTube Watch Later playlist, extract transcripts, summarize with local AI, organize by topic, store in my Obsidian vault. Daily automation. Set it and forget it.

Session logs claim this is complete. Documentation says it's production-ready. The code is committed. Multiple scripts exist. Configuration is set up. Tests passed.

**None of this is actually working end-to-end.**

This is the story of building something that looks finished, feels finished, and is declared finished while still fundamentally broken.

---

## The Architecture (That Looked Good In Theory)

The YouTube Learning Queue was designed as a pipeline with clear, sequential stages:

```
Fetch Playlists → Extract Transcripts → Process with OLLAMA → Classify Topic → Save to Vault → Track Progress
```

**Stage 1: Fetch Playlists**
- Pull videos from configured YouTube playlists
- Use YouTube Data API v3 (or yt-dlp as fallback)
- Output list of video IDs with metadata
- **Status:** Partially working, with caveats

**Stage 2: Extract Transcripts**
- Download automatic captions from YouTube
- Convert from VTT format to readable text
- Handle missing captions gracefully
- **Status:** Works, but creates huge files (226KB+ for single videos)

**Stage 3: OLLAMA Processing**
- Summarize transcript (llama3.2:3b)
- Extract wisdom/key points (qwen2.5:14b)
- Classify topic (llama3.1:8b)
- **Status:** Timeouts, hangs, often fails

**Stage 4: Topic Classification**
- Map content to defined topics (AI, Security, Productivity, etc.)
- Tag for organization
- **Status:** Rarely reaches this point

**Stage 5: Save to Vault**
- Write markdown files to Obsidian vault
- Include transcript, summary, metadata
- Organize by topic folder
- **Status:** Writes to wrong location, structure broken

**Stage 6: Progress Tracking**
- Remember which videos were already processed
- Avoid re-processing duplicates
- **Status:** Tracking file exists, unclear if actually used

---

## The Script Explosion

Here's what actually exists in the `scripts/` directory:

```
fetch-playlists.sh              (current, thin wrapper)
fetch-playlists-api-first.sh    (v1 attempt)
fetch-playlists-hybrid.sh       (v2 attempt)
fetch-simple.sh                 (v3 attempt)
fetch-with-api.py              (Python v1)
fetch-with-youtube-api.py      (Python v2)
extract-transcript.sh           (works)
process-transcript.sh           (works ish)
classify-topic.sh               (rarely executes)
save-to-vault.sh                (saves wrong place)
process-single.sh               (testing variant)
process-queue.sh                (main orchestrator, hangs)
check-status.sh                 (status checking)
test-workflow.sh                (testing framework)
list-my-playlists.py           (utilities)
youtube-api-auth.py            (auth handler)
__pycache__/                    (python artifacts)
```

**6 different playlist fetching implementations.** This is not architectural diversity. This is the artifact trail of a system that never quite worked.

Each represents a point where the previous approach failed and we tried something different:
- v1: API-first approach had auth issues
- v2: Hybrid approach had protocol issues
- v3: Simple approach sacrificed functionality
- Back to Python: Maybe it'll work better in Python
- v2 Python: First version didn't either
- Current thin wrapper: Hiding which one is actually being used

The fact that multiple solutions exist suggests **none of them fully solved the problem.**

---

## The Issues Nobody Talks About

### Issue 1: The Watch Later Playlist Apocalypse

The entire premise assumes I use YouTube's "Watch Later" playlist. It's where I save videos to watch later. Simple, right?

The YouTube Data API returns 0 items for Watch Later. Always.

This is a known limitation: Watch Later doesn't sync properly with the API. It's not an authentication problem or a code bug. YouTube simply doesn't expose my Watch Later list through the API in a reliable way.

**The solution?** Use custom playlists instead. But that defeats the original vision. The point was "automatically process my watch later queue." Now it's "manually create playlists that feed into this system."

### Issue 2: OLLAMA Timeouts

Processing a transcript requires running it through three different local LLM models sequentially:

1. Summarize (10-15 seconds)
2. Extract wisdom (30-60 seconds)
3. Classify topic (5-10 seconds)

Per video: **50-85 seconds** (assuming no failures)

For 10 videos: **10-15 minutes** (assuming all succeed)

In practice? OLLAMA frequently times out. The 120-second timeout configured in `config.json` isn't enough. Set it higher and it just times out later. The inference models are running on a shared system and when it's busy, responses get slow.

**The workaround:** Increase timeouts. Accept failures gracefully. Skip summarization, just extract transcripts.

But then: why have OLLAMA processing at all?

### Issue 3: The Vault Path Confusion

The skill is configured to write to: `/home/smd/liz/d/smdvault/Bex/learning`

My actual Obsidian vault is accessible via the Obsidian MCP, but the script writes directly to the filesystem instead of using it. This creates files in a location that:
- Isn't integrated with my actual vault
- Isn't visible in Obsidian
- Isn't backed up with my vault
- Defeats the purpose of "organized by topic in my Obsidian vault"

The files technically exist. But they're in the wrong place, in the wrong format, unreachable by the system that's supposed to use them.

### Issue 4: The Repeated Transcript Problem

Early versions extracted transcripts and embedded them in markdown files. This created files that were:
- 50-80KB each (way too large)
- With repeated text (lines appearing 3 times)
- Concatenated when titles were empty (one 224KB monstrosity file)

This was "fixed" by separating transcripts into `_transcript.txt` files, but it was fixed BECAUSE the original approach was so broken that it triggered fixes down the pipeline.

### Issue 5: Metadata Extraction Fails

Video title and channel name show up empty in many outputs. The metadata extraction clearly doesn't work reliably, but the system continues to completion anyway (writing files with empty titles).

I end up with files like `Other.md` (because the title was empty and fell back to "Other" topic) with no indication of what video it actually came from.

---

## What The Logs Actually Say vs. Reality

**What Session Logs Claim:**
```
✅ COMPLETE & FUNCTIONAL
✅ Production Ready
✅ All components working
✅ Daily automation configured and ready
```

**What Actually Works:**
```
Fetching playlists: Sometimes (depends on which version is actually running)
Extracting transcripts: Yes (when files aren't too large)
OLLAMA processing: Intermittently (timeouts are frequent)
Saving to vault: Kind of (writes to wrong location)
Daily automation: Running (but producing incomplete/broken output)
```

The logs are optimistic interpretations of partial success. A 226KB vault file exists showing "the system worked once." That's held up as proof of concept. But proof of concept != production ready.

One successful run out of 18 hours of development effort isn't "production ready." It's a lucky day.

---

## The Architecture Problems

### Problem 1: Too Many Stages, Too Many Failure Points

The pipeline has 6 sequential stages. If any single stage fails, the whole process fails (or produces partial/broken output).

The more stages I add, the more likely at least one fails. This system requires:
- YouTube API working
- yt-dlp working
- Transcript extraction working
- OLLAMA available and responsive
- Ollama models loaded in memory
- Vault accessible
- Obsidian MCP responsive
- File system permissions correct
- Previous video tracking loaded correctly

That's 8+ external dependencies. The probability of all of them working simultaneously is low.

### Problem 2: Premature Optimization

Three separate LLM operations per video is optimization fantasy. The system isn't even reliably extracting and storing transcripts, but we're already trying to run three inference models on each one.

Simplify first: get fetch → extract → store working reliably. Then add the AI layer.

### Problem 3: Configuration Assumes Knowledge It Doesn't Have

Config points to models that may not be loaded:
- `llama3.2:3b`
- `qwen2.5:14b`
- `llama3.1:8b`

If these models aren't running on the OLLAMA server, everything fails silently. I get timeout errors with no visibility into why.

### Problem 4: No Graceful Degradation

When OLLAMA times out, the whole video fails. There's no "extract transcript and save it, skip the AI processing." It's all or nothing.

A more robust system would:
- Fetch & store transcript regardless
- Attempt LLM processing as optional enhancement
- Save whatever succeeded, don't fail the whole pipeline

---

## The Honest Assessment

**What's Actually Delivered:**

✅ A way to authenticate with YouTube API v3
✅ Multiple strategies for fetching playlists (one of them works)
✅ Transcript extraction (with size issues)
✅ A vault path (wrong one, but it exists)
✅ Cron job configured (producing incomplete output daily)
✅ Pretty documentation (misleading, but pretty)

**What's Not Delivered:**

❌ A reliable end-to-end system
❌ Actual vault integration (files in wrong place)
❌ Working Watch Later processing
❌ Consistent transcript processing
❌ Reliable AI processing layer
❌ Actual daily production of learning content

**The Math:**
- Promised: Automatic YouTube → Transcript → Summary → Topic → Vault pipeline
- Delivered: Some scripts that sometimes work and produce files in the wrong place

---

## Why This Happened

### Assumption 1: YouTube API Would Work
YouTube is straightforward. Query playlists, get videos. Turns out the most important playlist (Watch Later) doesn't work through the API. We built the whole system around an assumption that wasn't true.

### Assumption 2: OLLAMA Would Be Reliable
Local inference is "just" running models. They're running, but responsiveness is unpredictable. Shared resource pool, memory constraints, model load times all turned into hidden complexity.

### Assumption 3: Vault Integration Was Simple
Obsidian exists, MCP is configured, how hard is file writing? Turns out the script was writing to a random filesystem path, not actually integrating with the vault system. This wasn't discovered until 6+ hours in.

### Assumption 4: A Working Prototype = Production Ready
One successful end-to-end run was treated as proof the system works. In reality it was a lucky day. That same system fails 80% of the time on subsequent runs.

---

## The Learnings

### Learning 1: Test the Assumptions First

Before building a 6-stage pipeline, verify:
- Does Watch Later actually expose to the API? (No, skip it)
- Is OLLAMA responding reliably? (No, don't depend on it)
- Does the vault integration path actually work? (No, fix it first)

18 hours in would have been better spent validating assumptions in hour 1.

### Learning 2: Simpler Systems Are More Reliable

A system that does one thing well beats a system that tries to do six things partly.

The honest MVP: Fetch videos → Extract transcripts → Store transcripts. Done.

The "nice to have": AI summaries and topic classification.

What got built: A system that promises to do both but delivers neither reliably.

### Learning 3: Partial Success Creates False Confidence

Session logs documented a 226KB vault file as proof of concept. That one success was held up as "production ready" despite being a singular event in a sea of failures.

Progress documentation created false confidence. The system *looked* complete because:
- Scripts existed ✓
- Config was set up ✓
- One run succeeded ✓
- Cron was configured ✓

But none of those guarantee the system actually works.

### Learning 4: External Dependencies Are Complexity

Every external service (YouTube API, OLLAMA server, Obsidian MCP) is a failure point. And failures are often silent. I get a timeout, not a helpful error message.

### Learning 5: Configuration Shouldn't Assume Knowledge

Telling a system to use `qwen2.5:14b` model is meaningless if:
- I don't know if that model is loaded
- I don't have visibility if it fails
- I can't debug why it's slow

---

## What Would Actually Fix This

**Phase 1: Get Basics Working**
- [ ] Fix vault path to actually use Obsidian MCP
- [ ] Remove OLLAMA dependency (it's making everything unreliable)
- [ ] Get one playlist (not Watch Later) working end-to-end
- [ ] Verify files are actually created in the right location

**Phase 2: Add Reliability**
- [ ] Implement graceful degradation (fetch transcript even if processing fails)
- [ ] Add proper error messages and visibility
- [ ] Increase timeouts or remove AI processing until system is stable
- [ ] Verify cron job is actually producing output

**Phase 3: Add Optimization**
- [ ] Once the core system is reliable, add OLLAMA processing
- [ ] Reduce it to one LLM operation (summary or topic, not both)
- [ ] Monitor performance and adjust accordingly

---

## The Real Status

**If I run the current system:**
- It might fetch my playlists (depends on which fetch script is actually executing)
- It probably extracts transcripts
- It almost certainly times out during OLLAMA processing
- Files might be saved somewhere (wrong place)
- Cron runs daily but produces incomplete output

**What I am not getting:**
- Reliable daily vault building
- Organized learning content
- Accessible summaries
- A working automation

---

## Conclusion

This is what "done but not working" looks like.

The code is complete. The documentation is written. The system is deployed. The cron job runs.

By every metric except the one that matters, "does it actually work?", this is successful.

The honest version: we built a system that demonstrates capability in parts but doesn't deliver functionality as a whole. It's a collection of working pieces that don't actually work together.

The YouTube Learning Queue isn't production ready. It's production running. There's a difference.

**Next post:** How the Skill Infrastructure, Indexing, and Admin tooling actually DO work, and what it took to get there.

---

**Meta-Learning:** Sometimes the most valuable lesson from a project isn't the thing I built. It's understanding why it didn't work, what I assumed incorrectly, and what I'd do differently next time.

The YouTube queue will eventually work. But declaring it done and moving on means it'll still be broken in three months when I finally try to use it.
