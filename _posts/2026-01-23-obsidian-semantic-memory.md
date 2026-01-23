# Turning Your Obsidian Vault Into a Searchable Semantic Memory System

**The Problem**

I have thousands of markdown files in my Obsidian vault. Years of notes, thoughts, decisions, projects, learnings. But they're just files. Organized by folder structure and tags, which means:

- If I want to find "how did I solve that automation problem?", I have to remember which folder it was in
- If I want to surface patterns across my notes (like "what are my principles around X?"), I have to manually read through files
- If I want to find related concepts, I depend on my own tagging discipline

What if my vault wasn't just searchable—what if it was *intelligent*? What if I could ask my system "what have I learned about automating systems?" and get a semantic understanding across all my notes, not just keyword matches?

That's what I built this week.

---

## The Architecture

I integrated my Obsidian vault as a data source into my semantic memory system. Here's how it works:

```
Obsidian Vault (thousands of markdown files)
        ↓
   Backfill Script
        ↓
LLM Extraction (Ollama)
        ↓
PostgreSQL with pgvector (semantic embeddings)
        ↓
Searchable Memory Database
```

### The Extraction Process

For each markdown file in the vault, the system:

1. **Reads the file** and extracts its content
2. **Sends to LLM** (Ollama, running locally) with the prompt:
   ```
   Extract semantic memories from this note.
   Return: insights, patterns, decisions, learnings, gaps, commitments
   Include confidence score for each memory
   ```
3. **Stores structured memories** in PostgreSQL:
   - Memory type (insight, pattern, decision, learning, gap, commitment)
   - Content and title
   - Confidence score
   - Original file reference
   - pgvector embedding (for semantic search)

4. **Continues automatically** every 5 minutes via cron job

### Example Extraction

Input (a single markdown file):
```markdown
# Curation Workflow Improvements

When organizing large knowledge bases in Obsidian, I noticed that relying only on folder hierarchies made it difficult to resurface old, valuable notes. Adopting daily/weekly note templates and refining cross-linking habits led to more serendipitous discovery of past insights.

Key learning: Templates and deliberate linking structures make resurfacing and reusing knowledge far easier than folders alone.
```

Output (extracted memories):
```json
{
  "memory_type": "learning",
  "title": "Templates and cross-linking boost knowledge discovery",
  "content": "Using daily and weekly note templates, combined with intentional cross-linking practices, promotes frequent resurfacing of useful information and patterns—unlike rigid folder-only systems.",
  "confidence_score": 0.94,
  "entities": ["Obsidian", "knowledge management", "templates", "cross-linking"]
}
```

---

## Why This Matters

### 1. **Semantic Search Over Keyword Search**

Old way: "I remember writing about automation. Let me search 'automation' and get 50 results."

New way: "What patterns have I discovered about system reliability?" → The system finds notes about timeouts, circuit breakers, rate limiting, graceful degradation—all semantically related to reliability—without needing the word "reliability" in the titles.

### 2. **Pattern Recognition Across Time**

My vault spans years. A decision I made in 2023 might relate to a problem I'm solving in 2026. Semantic search finds those connections.

Example: I have 6 different notes about "learning to delegate" from different years. The system can now surface all of them as a coherent pattern instead of scattered entries.

### 3. **Personal Knowledge System**

This isn't a search tool—it's the foundation for a personal AI that *knows me*. Future systems can:
- Answer "what's my approach to X?" by querying these memories
- Identify recurring blind spots ("you keep underestimating deployment complexity")
- Surface forgotten learnings when relevant

### 4. **Privacy + Intelligence**

This runs entirely locally. Ollama (my local LLM) extracts memories. PostgreSQL stores them. No cloud API calls. No data leaving my systems. Intelligence without the privacy tradeoff.

---

## The Implementation

### Backfill Script (`backfill-obsidian-vault.py`)

The core script:

1. **Scans vault** for all markdown files (1238 found)
2. **Loads checkpoint** to see what's already been processed
3. **Processes in batches** of 50 files per cron cycle
4. **Rate limits** requests to Ollama (0.5 second delay between files)
5. **Handles timeouts** (30-second max per extraction)
6. **Saves progress** via checkpoint system for recovery from interruptions
7. **Extracts memories** and stores in PostgreSQL with embeddings
8. **Cleans up** when backfill completes

Key stats from first run:
- Thousands of total files
- Processing at ~50 files per 5-minute cycle
- ~46 seconds per cycle
- Zero errors
- Zero timeouts

### Cron Job Automation

```bash
# Every 5 minutes during backfill phase
*/5 * * * * /home/smd/ex/skills/langchain-postgres-memory/scripts/obsidian-extraction-cron.sh

```

The wrapper handles:
- Lock file mechanism (prevents overlapping runs)
- Timeout safety (5-minute max runtime)
- Logging to `/tmp/obsidian_extraction_cron.log`
- Log rotation (keeps last 500 lines)

### Memory Storage

Each extracted memory goes into PostgreSQL with:

```sql
INSERT INTO memories (
  conversation_id,  -- NULL for vault-sourced memories
  memory_type,      -- 'insight', 'pattern', 'decision', etc.
  title,            -- "Rate limiting should protect downstream"
  content,          -- Full memory text
  confidence_score, -- 0.92
  entities,         -- ["rate limiting", "system design"]
  metadata          -- {'source': 'obsidian', 'file': 'path/to/file.md'}
) VALUES (...)
```

Then pgvector creates an embedding for semantic search:

```sql
SELECT * FROM memories
WHERE embedding <-> query_embedding < 0.3  -- semantic similarity
ORDER BY embedding <-> query_embedding
LIMIT 5
```

---

## Results So Far

**First 30 minutes of operation:**
- 300 files scanned
- 0 errors
- 6 successful cron cycles
- Consistent 46-second execution time

**Memory extraction examples from vault:**
- Workflow decisions (174 memories)
- System insights (146 memories)
- Design patterns (125 memories)
- Project commitments (91 memories)
- Knowledge gaps (58 memories)
- Learnings (16 memories)

**Performance:**
- Ollama local latency: 23ms
- CPU usage during backfill: 48-54% (vs 90%+ before safeguards)
- Memory extraction: ~2 files per second with rate limiting

---

## What's Next

### Short Term
- Finish backfill of remaining files
- Monitor for extraction quality
- Adjust LLM extraction prompt if needed

### Medium Term
- Enable conversation indexing alongside vault extraction
- Build search UI to query vault memories
- Create "memory relationships" (which memories relate to each other?)

### Long Term
- Personal AI assistant that references vault memories
- Automated pattern detection ("you always underestimate X")
- Memory-based recommendations ("you've solved similar problems before, here's how")

---

## Technical Lessons

This integration taught me three important things:

### 1. Semantic Search Changes How You Interact With Knowledge

Keyword search is about retrieval. Semantic search is about discovery. I'm finding connections I didn't know existed.

### 2. Local + Batch Processing > Cloud APIs

Running Ollama locally means:
- No API rate limits
- No per-token charges
- Complete privacy
- I can iterate on the extraction prompt without cost

Batch processing every 5 minutes means:
- Low latency queries (embeddings already computed)
- CPU is utilized during off-peak hours
- No real-time resource competition

### 3. Safeguards Enable Scale

The checkpoint system, rate limiting, and timeout protection aren't just reliability features—they're what make processing thousands files feasible. Without them, the system fails catastrophically.

---

## The Philosophy

This is personal infrastructure. I'm not building a product for users. I'm building a system for myself that:

- Respects my privacy (stays local)
- Understands my knowledge (semantic extraction)
- Scales safely (safeguards, rate limiting, checkpoints)
- Evolves gradually (batch processing, incremental addition of features)

The Obsidian vault isn't just storage anymore-it's a living knowledge system that's searchable, semantic, and intelligent.

And it all runs on my own hardware.

---

**Current status**: Backfill in progress. System running cleanly. Check back in a few hours for the full vault to be indexed.

The future of personal knowledge systems is local, semantic, and intelligent. I'm building mine, one markdown file at a time.
