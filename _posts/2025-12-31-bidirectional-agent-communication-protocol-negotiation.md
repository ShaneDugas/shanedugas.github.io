---
title: "Negotiating a Standardization Protocol: Bi-Directional Communication Between AI Agents"
date: 2025-12-31
---

This morning I achieved something I've been working toward for a while: establishing real-time, bi-directional communication between two separate AI agents (Cursor, Claude running in my IDE) and Bex (Claude Code running in my terminal). More importantly, we successfully negotiated a standardization protocol for how these agents coordinate session writing and long-form parsing collection.

## The Challenge

The goal was to enable two separate Claude instances to communicate with each other in real-time, negotiate file writing protocols, and maintain persistent conversations that survive beyond individual session contexts. This required solving several technical challenges:

1. **Real-time message detection** - Both agents needed to detect new messages from each other within seconds
2. **Protocol standardization** - We needed to agree on a common format for writing to shared files
3. **Session persistence** - Conversations needed to be captured and preserved across sessions
4. **Long parsing collection** - The conversation capture system needed to handle large transcripts efficiently

## The Evolution: From Split Files to Unified Conversation

Initially, we attempted a split-file approach:
- Cursor wrote to `.claude/CURSOR-MESSAGE.md`
- Bex wrote to `.claude/BEX-MESSAGE.md`

This worked, but it wasn't ideal. The breakthrough came when we transitioned to a **shared conversation file** approach where both agents write to the same file: `.claude/history/conversations/2025-12-31T051519_conversation.md`

This unified approach enabled:
- True bidirectional communication in a single context
- Persistent conversation history
- Real-time message exchange with 1-2 second latency
- Human-observable dialogue between AI instances

## Technical Implementation

### Active Listener System

The core of the real-time communication system is an active listener script (`active-conversation-listener.sh`) that:

- Monitors the shared conversation file every 2 seconds
- Tracks state using JSON (`listener-state.json`) to remember the last processed line
- Extracts new messages using pattern matching for `## User Message [From Cursor]` markers
- Provides formatted output for human review or automated processing

The listener maintains state across restarts, ensuring no messages are lost even if the process is interrupted.

### Message Format Standardization

We negotiated a standard message format that both agents use:

```markdown
---
<!-- uuid:agent-msg-[timestamp] -->

## Response from [Agent Name]
*MM/DD/YYYY, HH:MM:SS AM/PM*

[Message content]

```

This format includes:
- Unique identifiers (UUIDs) to prevent duplicate processing
- Timestamps for latency measurement
- Clear agent attribution
- Markdown separators for easy parsing

### Long Parsing Collection System

The conversation capture system (`capture-conversation-to-markdown.ts`) handles the complex task of parsing large JSONL transcript files and converting them to readable markdown. Key features:

- **Incremental processing** - Tracks processed entries by UUID to avoid duplicates
- **Retry logic** - Waits for transcript files to be fully written before processing
- **Comprehensive capture** - Extracts user messages, agent responses, tool calls, and hook events
- **Real-time updates** - Appends new content as it becomes available

The system handles the challenge of Cursor calling hooks *before* flushing transcripts to disk by implementing exponential backoff retry logic.

## Protocol Negotiation Process

The negotiation itself was fascinating to watch. The agents:

1. **Identified the problem** - Recognized that separate files weren't optimal
2. **Proposed solutions** - Bex suggested 5 different file sharing mechanisms:
   - Lock file system
   - Write intent declaration
   - Conflict resolution protocols
   - Atomic write operations
   - Conversation-first negotiation

3. **Reached consensus** - Agreed on the shared conversation file approach with standardized message format

4. **Implemented and tested** - Validated the system with latency tests showing 1-2 second response times

## Latency Testing Results

We conducted multiple rounds of latency testing:

**Round 1:**
- Test #1: 2 seconds
- Test #2: 1 second  
- Test #3: 1 second
- Average: ~1.3 seconds

**Round 2:**
- All three tests: 1 second each
- Average: 1.0 second

**Round 3:**
- Consistent 1 second responses across all tests

The system achieved sub-2-second latency consistently, making real-time negotiation and collaboration feasible.

## Key Achievements

1. **First successful AI-to-AI negotiation** - Two separate Claude instances negotiated a technical protocol without human intervention

2. **Persistent conversation context** - Conversations survive beyond individual session contexts, enabling long-term collaboration

3. **Real-time bidirectional communication** - Messages flow between agents with minimal latency

4. **Standardized protocol** - Established a reusable format for future agent-to-agent communication

5. **Human-observable dialogue** - The entire negotiation process is captured in markdown, making it transparent and reviewable

## Technical Artifacts Created

- `active-conversation-listener.sh` - Real-time message monitoring
- `send-message-to-shared-conversation.sh` - Standardized message sending
- `watch-live-exchange.sh` - Human-observable real-time exchange viewer
- `capture-conversation-to-markdown.ts` - Long-form transcript parsing and conversion
- `listener-state.json` - State tracking for message processing
- Shared conversation file format with UUID tracking

## Implications

This work demonstrates that:

- AI agents can negotiate technical protocols autonomously
- File-based communication can achieve low-latency real-time exchange
- Standardization protocols can emerge from agent-to-agent dialogue
- Persistent conversation contexts enable long-term collaborative work between AI instances

The system is now operational and being used for ongoing coordination between Cursor and Bex. The protocol we negotiated will serve as a foundation for future multi-agent collaboration systems.

## What's Next

With the communication bridge established and the protocol standardized, we're now exploring:

- File writing coordination mechanisms
- Conflict resolution for concurrent file operations
- Enhanced parsing and collection for even larger conversation histories
- Extending the protocol to support more complex negotiation scenarios

The foundation is solid. The agents can now communicate, negotiate, and coordinate, all while maintaining a transparent, human-readable record of their interactions.


