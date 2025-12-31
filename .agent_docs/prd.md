# High Signal VC CRM - Product Requirements Document

Version 4.0 | December 2024

## Problem Statement

Managing 100+ VC interactions across multiple fundraising rounds is painful:

- **Status resets between rounds**: No way to track the same firm across Pre-seed and Seed with independent statuses
- **Interaction history is scattered**: Emails, LinkedIn, WhatsApp messages across tools with no unified view
- **Pitch insights are lost**: No way to analyze what questions VCs ask and how responses can improve
- **Rejection context disappears**: Can't remember why a firm passed last time when re-approaching

## Core Use Case

A founder raising capital across multiple rounds (Pre-seed, Seed, Series A) needs to:

1. Track the same VC firms across different fundraises with independent status per round
2. See which firms passed on Pre-seed but might reconsider for Seed
3. Maintain relationship history with individual partners even as they move between firms
4. Analyze pitch transcripts to identify common questions and improve responses over time

## User Stories

### MVP User Stories

1. **As a founder**, I want to create a new fundraise round and see all firms default to "Prospecting" status
2. **As a founder**, I want to track each firm's status independently per round so I can re-engage firms that passed before
3. **As a founder**, I want to log interactions (emails, calls, messages) with individual partners
4. **As a founder**, I want to store pitch transcripts and have AI extract questions and answers
5. **As a founder**, I want to see rejection reasons from previous rounds when re-engaging a firm
6. **As a founder**, I want email interactions auto-synced from Gmail to relevant firm engagements

### Future User Stories

7. **As a founder**, I want to search across all transcripts for how I've answered specific questions
8. **As a founder**, I want to see which questions VCs ask most frequently
9. **As a founder**, I want alerts when engagements go stale (no interaction in 14+ days)

## Scope Definition

### In Scope (MVP)

- Fork Twenty CRM and deploy locally via Docker
- Custom entities: Firm, Person (extended), FundraiseRound, FirmEngagement, Interaction, PitchMeeting
- 21-status pipeline for firm engagement tracking
- Gmail/IMAP sync and manual interaction logging
- Transcript storage with segment parsing and Q&A extraction
- Built-in workflow automations for status updates and alerts
- Basic text search across transcript segments

### Out of Scope (MVP)

- Semantic/vector search (future: Qdrant migration)
- Automated LinkedIn/Telegram/WhatsApp sync
- Outbound email composition from CRM
- Sentiment analysis
- Multi-user/team features

## Success Metrics

1. ✅ Can create new fundraise round and track firms with independent statuses
2. ✅ Can see previous round rejection reasons when re-engaging
3. ✅ Transcripts parsed into searchable Q&A segments
4. ✅ Pipeline view shows firm status across active round
5. ✅ Workflow automations keep engagement records fresh via email sync
6. ✅ Replaces spreadsheet-based tracking

## Non-Functional Requirements

- **Performance**: Pipeline view loads in <2s with 200+ firms
- **Data**: All data stored locally (Docker), no cloud dependency
- **Privacy**: Transcripts processed locally or via user's own API keys
- **Reliability**: PostgreSQL durability, daily backup capability