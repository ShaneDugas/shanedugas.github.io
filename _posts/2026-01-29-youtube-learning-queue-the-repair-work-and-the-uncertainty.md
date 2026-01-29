---
layout: post
title: "The YouTube Learning Queue: The Repair Work and the Uncertainty"
date: 2026-01-29
categories: infrastructure
---

**Date:** January 29, 2026
**Sequel to:** [The YouTube Learning Queue: How to Build Something That Looks Done But Still Doesn't Work](/blog/2026-01-03-youtube-learning-queue-reality-check)

---

## Where We Left Off

Three weeks ago I wrote a long post about the YouTube Learning Queue: the pipeline that was supposed to fetch my playlists, extract transcripts, run them through local AI, and save organized notes to my vault. The conclusion was blunt. It didn't work. It looked production-ready. It wasn't.

"Watch Later" doesn't expose properly to the API. OLLAMA times out. The vault path was wrong. (Something as simple as an Fstab entry broke it once) Too many stages, too many failure points. One successful run had been treated as proof the system worked. It wasn't. It seems that my repeated request for validation testing isn't really received with the same urgency as a human would receive such a request.

Forcing a validation check seems to be the key point in the workflow that needs to be forced with everything. All general instructions.

---

## The Repair Work

After that post I didn't walk away. I put a bunch of work back in.

**Vault structure.** Files were ending up in topic-only folders with no date hierarchy. I added a migration script to move existing output into a date-first layout: `base_path/YYYY-MM-DD/topic/`. Then I changed `save-to-vault.sh` to write new files that way from the start. So at least when something does get saved, it lands in a sane place.

**Save-to-vault fixes.** Sanitization, date handling, and path building got tightened. Transcripts stay in separate files. Markdown frontmatter is consistent. The script is less likely to write garbage into filenames or paths.

**Process-queue and dependencies.** I touched the orchestrator and the scripts it calls. I added a health-check script that reports cron execution (based on log file mtime), MAMA_LLAMA connectivity, vault writability, and processed-video count. So there's at least a single command that says "when did this last run?" and "can it reach the pieces it needs?"

**New failure mode.** Topic classification calls out to `ollama-workload-distribution` for the actual LLM query. On a run a few days ago, that script wasn't where the pipeline expected it. The error message got written straight into the "topic" field and then into the vault path. I ended up with a folder name that was literally an error string. So the repair work didn't remove failure modes. It added a new one: a missing dependency that corrupts output instead of failing fast.

---

## Did It Run Last Night?

I still don't know.

The cron is set for 2 AM. The health check looks at when the log file was last modified. Right now it reports: last run **78 hours ago**. So either the cron didn't run last night, or it ran but didn't write to that log (wrong path, permission, or crash before first write). Either way, I have no evidence that last night's run happened. The system is supposed to be "set and forget." Instead I'm left wondering every morning whether it ran at all.

That's the uncertainty. Not "did it process 5 or 10 videos?" but "did it run?"

---

## What This Feels Like

It feels like digging the same hole twice. First I built a pipeline that looked done and wasn't. Then I spent more time fixing vault paths, structure, and scripts. The pipeline still depends on YouTube playlists, OLLAMA (or a workload router), and a vault path that matches my actual setup. Any of those can break. And now there's another dependency: a separate skill's script must exist at a specific path or the "topic" becomes an error message and the file goes into a folder named after that error.

I have a health check that can tell me "last run 78 hours ago" and "MAMA_LLAMA is up" and "vault is writable." That's better than nothing. It doesn't tell me why the log hasn't been updated. It doesn't tell me whether the cron job is still in crontab, or whether it's pointing at the right script, or whether something is failing before the first log line. So I'm still in the dark about last night.

---

## What I'd Do Differently (Again)

**Single source of truth for "did it run?"** Cron plus "log file mtime" is fragile. I'd want something that always writes a single line or a small JSON blob when the job starts and when it finishes (success or not). Then the health check could read that instead of inferring from log mtime.

**Fail fast on missing dependencies.** If `ollama-query.sh` (or whatever the pipeline needs) isn't there, the script should exit with a clear error and not write error text into topic fields or paths. No silent corruption.

**Fewer moving parts.** The Jan 3 post said it: get fetch, extract, and store working reliably before adding more AI steps. I'm still paying for the complexity of multiple LLM stages and cross-skill script calls.

---

## Status Today

- **Cron execution:** Health check says last run 78h ago. So I don't know if it ran last night.
- **Vault:** Structure is date-first now; save-to-vault writes there when the rest of the pipeline works.
- **OLLAMA / workload routing:** Still a dependency. When the query script is missing, the pipeline embeds the error in the output.
- **Peace of mind:** Low. The repair work made some things better. It didn't make the system something I can stop thinking about.



---

## Conclusion

It is nice however, to just save a video to a playlist and have this workflow produce the transcripts, summary, and notes.

It's still not something I can trust. I still don't know if it ran last night. That's the update.

---


