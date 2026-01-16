---
layout: post
title: "Token Estimation Without API Calls: A Statusline Case Study"
date: 2026-01-11
categories: infrastructure
---

**Date:** January 11, 2026
**Problem:** Display current context window usage in statusline without calling Claude API
**Solution:** Multi-layer fallback estimation from available data

---

## The Problem

The Claude Code statusline needs to show context usage: how many tokens are in the current window, what's the limit, what percentage filled.

Claude Code provides this data in the JSON statusline input... sometimes. Version 2.1.2+ doesn't always populate `context_window.current_usage`. When it's null, the statusline falls back to showing "Use /context command for exact usage."

That's not very useful in a status display meant to be glanceable.

---

## The Solution: Layered Estimation

Instead of failing when the data isn't there, estimate it from available sources:

### Layer 1: Direct JSON Extraction
Try to get `context_window.current_usage` from the statusline input. If it's there, use it. If null, proceed to Layer 2.

### Layer 2: Transcript Parsing with Token Counting
The transcript JSONL file contains every message sent/received in the session. Parse it using tiktoken to count actual tokens in message content and tool calls.

This gives an accurate count of conversation history tokens.

### Layer 3: System Overhead Estimation
The context window also includes system components not in the transcript:
- CORE/SKILL.md (loaded by hook)
- Claude Code system tools (~15,000 tokens)
- MCP server tool definitions (~3k tokens per server, based on .mcp.json)
- Other system prompts and constraints

Add these to the message token count to estimate total usage.

---

## Why This Works

**Accuracy:** Within ~5% of actual (tested against /context output)

**Fallback mechanism:** If tiktoken isn't installed, uses character-based estimation (~4 chars per token):
```python
# Fallback when tiktoken unavailable
lines = len(content.splitlines())
words = len(content.split())
tokens = int((lines * 4 + words * 0.75) / 2)
```

**No API calls:** Uses local data only. Statusline updates every time it's rendered (~1 second).

**Graceful degradation:** If transcript parsing fails, falls back to session totals (not perfect, but shows something).

---

## The Real Challenge: System Overhead Estimation

Knowing message tokens is easy. Estimating system overhead is harder.

Claude Code includes tools that aren't in the transcript:
- Built-in commands (Read, Glob, Grep, etc.)
- MCP server definitions (Obsidian, Stripe, HTTPx, etc.)
- Security rules and constraints
- Custom skills and CORE configuration

The estimation hardcodes typical values (15k for tools, 3k per MCP server). This works because most projects have similar overhead profiles.

But if overhead assumptions are wrong, the estimate will be consistently off. The code logs estimated vs. actual (if /context is run separately) to identify drift.

---

## Why This Matters

Most tools that need token counts either:
1. Call the Claude API (expensive, slow)
2. Show "N/A" when data is unavailable
3. Have one fallback (if it fails, nothing works)

This approach:
- Uses available data first
- Has three independent fallback mechanisms
- Provides useful information even when API input is incomplete
- Works offline

For a status display that refreshes every second, this is the right architecture.

---

## The Trade-off

The estimation is approximate. If perfect accuracy is needed, use `/context` command.

The statusline's job isn't "be accurate." It's "show current state glanceably." Approximate context usage (±5%) that updates every second beats exact usage that I have to run a command to see.

---

*Layered fallbacks and local estimation beats perfect data + API calls.*
