---
layout: post
title: "Three-Layer Knowledge Management Architecture"
date: 2026-01-04
categories: infrastructure
series: "Skill Infrastructure Breakdown"
part: 1
---

# Three-Layer Knowledge Management Architecture

**Date:** January 4, 2026
**Series:** Skill Infrastructure Breakdown
**Part:** 1 of 10

---

## Introduction

My personal AI infrastructure evolved into a sophisticated three-layer system for managing knowledge and capability. This is the foundation that everything else builds on.

---

## The Three Layers

### Layer 1: Session Layer (Real-Time)

Captures current work as it happens:
- Tracks decisions, tool usage, and context
- Stored in memory during the active session
- Feeds into downstream systems
- Purpose: Immediate access to what I am doing right now

### Layer 2: History Layer (File-Based Metadata)

Organized historical record:
- Session summaries with ISO 8601 timestamps
- Learning documents (insights extracted from conversations)
- Structured file organization by date
- Searchable via filesystem and grep
- **Answers:** "What did I do and when?"

### Layer 3: Memory Layer (Semantic Extraction)

Intelligent knowledge store:
- Postgres + LangChain vector database
- Semantic embeddings for similarity search
- Extracted insights, patterns, decisions, gaps
- Queryable via `search-memories.py`
- **Answers:** "What did I learn?"

---

## Why Three Layers?

Each layer serves different query patterns:

- **Session layer:** "What am I doing right now?"
- **History layer:** "What happened on 2026-01-03?"
- **Memory layer:** "What did I learn about YouTube queue architecture?"

**This redundancy is intentional.** It's not a design flaw; it's a feature. Different questions require different storage and indexing strategies.

---

## Next Steps

Understanding this three-layer foundation is critical because everything else in my infrastructure builds on top of it. The skill system, the indexing mechanism, and the semantic search all work within this architecture.

*Next: The Skill System - How 27 capabilities are organized and discovered.*
