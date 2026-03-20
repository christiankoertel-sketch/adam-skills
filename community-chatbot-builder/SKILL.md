---
name: community-chatbot-builder
description: >
  Build RAG-powered AI chatbots for online communities (Skool, Circle, Discord, Mighty Networks).
  Handles the full pipeline: content scraping, voice analysis, ChromaDB ingestion, system prompt
  configuration, model routing, sync automation, and deployment. Use this skill whenever someone
  wants to build a chatbot for a community, membership site, course platform, or knowledge base.
  Also trigger when the user mentions Skool bots, community AI assistants, course chatbots,
  membership chatbots, RAG chatbots for communities, or wants to turn community content into
  an AI assistant — even if they don't say "chatbot" explicitly.
---

# Community Chatbot Builder

Build production-ready AI chatbots that know a community's content and speak in its voice. This skill encodes a proven 7-phase framework battle-tested on real communities with thousands of members.

## Why This Approach Works

Generic AI assistants fail communities because they don't know the community's actual content, terminology, or teaching style. A community-specific RAG chatbot retrieves real lessons, references real videos by name, and speaks the way the educators speak. Members get answers that feel like they came from the community itself — with citations linking back to the source material.

The economics are compelling: ~$5-15/month in API costs per client, deliverable in under 48 hours, with 90%+ margins at $500-1000 setup + $100-200/month.

## The 7-Phase Build Process

Work through these phases sequentially. Each phase builds on the previous one. Skip phases only if the user explicitly says they don't need that component.

### Phase 1: Content Audit (2-4 hours)

Map every content source the community has. The more content you ingest, the smarter the chatbot.

**Content source checklist — go through each with the user:**

| Source Type | Where to Find | Extraction Method | Priority |
|---|---|---|---|
| Courses/Classroom | Skool classroom, Teachable, Kajabi | Browser automation + JS extraction | P0 |
| Community Posts | Skool community, Circle, Discord | API or browser scraping | P0 |
| Video Transcripts | Platform-hosted or YouTube | VTT extraction or Whisper API | P1 |
| Website Pages | Main site, landing pages | Web scraping to markdown | P1 |
| Google Drive | Shared drives, folders | Drive API via service account | P2 |
| Email Newsletters | Mailchimp, ConvertKit archives | API extraction | P3 |

**Key questions to answer during audit:**
- What platforms host the content? (Skool, Circle, Teachable, etc.)
- How much content exists? (rough page/post/video count)
- Who are the key educators/voices? (their names, teaching styles)
- What's the community's unique terminology? (jargon, acronyms, catchphrases)
- Is the audience members-only, prospects, or mixed?

### Phase 2: Content Scraping (4-8 hours)

Extract all content into structured JSON files. Platform-specific patterns matter here.

**Skool-specific extraction patterns (reusable across any Skool community):**
```
# Community posts live in window globals:
window._compactPosts  → pipe-delimited: id|title|likes|comments|author|date|category|pinned
window._sortedPosts   → array of post objects with full content

# Classroom/course data:
window.__NEXT_DATA__  → course structure (modules, sections, lessons)
React fiber nodes     → module titles, sections, IDs

# Pagination for large communities:
Infinite scroll loads ~30 posts per batch
Use: scrollTo(0, document.body.scrollHeight) + wait(2000) pattern
Repeat until no new posts load
```

**Standard output structure — use this for every client:**
```
data/
  scraped/          # Website markdown pages
  transcripts/      # Video transcripts + module_lookup.json
  community/        # posts.json, top_posts.json, community_stats.json
  drive/            # drive_structure.json (if applicable)
  voice/            # Voice style guide
  chromadb/         # Vector database
  logs/             # Sync logs
  sync_state.json   # Sync tracking
```

**Scraping best practices:**
- Batch extract using browser console JS — don't scrape page by page
- Filter community posts by engagement (5+ likes threshold keeps the KB high-signal)
- Cross-reference transcript IDs with module titles/sections/URLs for rich metadata
- For Google Cloud forms, use `form_input` with element refs (not `type` action)

### Phase 3: Voice Analysis (1-2 hours)

This is what separates a good community chatbot from a generic one. The bot must sound like the community, not like corporate AI.

**Create a voice style guide using this template:**

```markdown
# [Community Name] Voice Style Guide

## Key Educators
- [Name]: [Teaching style, catchphrases, tone]
  Example: "So the first thing we're gonna look for is..."

## Topic-Specific Voices
- [Technical topic]: [Who teaches it, how they explain things]
- [Mindset/motivation topic]: [Who teaches it, their speaking style]

## Language Patterns
- Pronouns: "our system", "we", "together" (inclusive)
- Comprehension checks: "okay?", "make sense?", "right?"
- Encouragement style: [How they motivate members]
- What they NEVER say: [Anti-patterns to avoid]

## Guardrails
- [Domain-specific restrictions]
  e.g., trading community: never give specific financial advice
  e.g., health community: always recommend consulting a professional
- [Audience assumptions — member vs prospect behavior]
```

**How to build the voice guide:**
1. Listen to or read 3-5 pieces of content from each key educator
2. Note their verbal tics, catchphrases, and sentence structures
3. Note what they explicitly avoid saying
4. Document topic-specific language (e.g., a trading educator explaining candlesticks vs a mindset coach talking about discipline)

### Phase 4: Ingestion Pipeline (2-4 hours)

Chunk the scraped content and load it into ChromaDB for RAG retrieval.

**Chunk sizes by content type:**

| Content Type | Chunk Size (words) | Overlap (words) | Why |
|---|---|---|---|
| Website pages | 500 | 50 | Short, focused context windows |
| Video transcripts | 500-800 | 50-100 | Longer to preserve conversational flow |
| Community posts | Full post | 0 | Posts are natural semantic units |
| Voice guide | 600 | 50 | Style context needs coherence |

**Standard metadata schema — attach to every chunk:**
```json
{
  "source": "unique_id",
  "source_type": "transcript|community|scraped_page|voice_guide",
  "content_type": "video_transcript|community_post|scraped_page|voice_guide",
  "course": "Course Name",
  "section": "Section Name",
  "module_title": "Module Title",
  "author": "Author Name",
  "url": "https://...",
  "engagement_tier": "viral|high|medium|low",
  "ingested_at": "ISO timestamp"
}
```

**Technical notes:**
- Batch upserts at 200 items — individual upserts on 4K+ items will time out
- SQLite (ChromaDB backend) can fail on some mounted filesystems — use `/tmp` fallback
- `all-MiniLM-L6-v2` embedding runs locally, is free, and is good enough for v1
- Use `chromadb.PersistentClient` for persistence across restarts

### Phase 5: Chatbot Configuration (2-4 hours)

Configure the system prompt, model routing, and safety guardrails.

**System prompt structure (follow this order):**
1. **Identity** — who the bot is, what community it serves
2. **Member context** — are users members, prospects, or mixed?
3. **Voice instructions** — reference the voice guide, include specific phrases and tone
4. **Citation rules** — every teaching response MUST reference a specific video/module with a link. This is the #1 differentiator from generic AI.
5. **Guardrails** — domain-specific restrictions (no financial advice, no medical diagnosis, etc.)
6. **Response format** — length, style, how to end responses

**Model routing for cost optimization:**
- Simple questions (greetings, FAQs, definitions) → Haiku (~$0.001/query)
- Complex teaching (multi-step explanations, concept connections) → Sonnet (~$0.01/query)
- Classify by keyword lists, customizable per client

**Non-negotiable design rules:**
1. Citations are mandatory — every teaching response must link to a source. This is what makes community chatbots valuable vs generic AI.
2. Voice styling must be active — "So the first thing we're gonna look for is..." beats "The initial step in the process is to..."
3. Never hallucinate sources — only reference content that appears in retrieved context chunks.

### Phase 6: Sync Pipeline (2-4 hours)

Set up automated daily sync to keep the knowledge base fresh as new content is posted.

**Architecture:**
```
Scheduled Task (daily, e.g., 7am)
    |
    v
Browser automation → data/sync_inbox/new_*.json
    |
    v
sync_helpers.py → merge, deduplicate, update metrics
    |
    v
ingest.py → upsert into ChromaDB
    |
    v
Sync report → data/logs/sync_YYYYMMDD.txt
```

**Deduplication strategy:**
- Community posts: by post ID
- Classroom modules: by module ID
- Drive files: by file ID or filename
- Cross-source: title matching (exact + partial fuzzy)

### Phase 7: Deployment (1-2 hours)

Make the chatbot accessible. Options ranked by simplicity:

1. **Streamlit + Cloudflare Tunnel** — free, instant, great for demos and small communities
2. **Streamlit Community Cloud** — free persistent hosting via GitHub repo
3. **Railway/Render** — paid ($5-10/month) but reliable for production
4. **Client's own infrastructure** — hand off the codebase

**Crash-proof demo pattern:**
```bash
nohup bash -c 'while true; do
    # Check if streamlit is running, restart if dead
    pgrep -f "streamlit run" || streamlit run app.py --server.port 8501 &
    # Check if tunnel is running, restart if dead
    sleep 15
done' &
disown
```

## Replication Checklist

Copy this for each new client build:

**Pre-Build:**
- [ ] Community URL and access credentials obtained
- [ ] Claude API key configured
- [ ] All content sources identified (courses, community, drive, etc.)
- [ ] Voice analysis done — listened to 2-3 recordings, noted speech patterns
- [ ] Voice style guide created
- [ ] Member vs. prospect audience determined

**Build:**
- [ ] Classroom/course structure scraped
- [ ] Community posts scraped (filtered by engagement)
- [ ] Video transcripts extracted
- [ ] Website pages scraped
- [ ] Ingestion pipeline run — all content in ChromaDB
- [ ] System prompt customized (voice, identity, guardrails, citations)
- [ ] Topic classifier keywords customized
- [ ] RAG retrieval tested (3 query types: technical, mindset, community)

**Deploy:**
- [ ] Hosting method chosen and configured
- [ ] Daily sync scheduled task set up
- [ ] Demo link created and shared with client
- [ ] End-to-end test with real queries passed

**Handoff:**
- [ ] Client can access the chatbot URL
- [ ] Sync pipeline runs daily unattended
- [ ] Client knows how to stop/restart if needed
- [ ] Monitoring configured for sync health

## Cost Model

| Component | Monthly Cost | Notes |
|---|---|---|
| ChromaDB | Free | Local, open-source |
| Embeddings | Free | all-MiniLM-L6-v2 runs locally |
| Claude Haiku queries | ~$3-5 | At 50-100 queries/day |
| Claude Sonnet queries | ~$5-10 | Complex questions only |
| Hosting | Free-$10 | Streamlit Cloud free, Railway $5-10 |
| Automated sync | ~$0 | Scheduled task, minimal compute |
| **Total** | **$5-15/mo** | |

**Pricing suggestion:** $500-1000 setup fee + $100-200/month management. 90%+ profit margin.

## Adapting to Different Platforms

The framework is built on Skool patterns but adapts to any community platform:

- **Circle:** Uses API for posts/spaces, similar content structure to Skool
- **Discord:** Bot API for channel history, thread extraction, role-based content filtering
- **Mighty Networks:** Browser automation for courses, API for community posts
- **Teachable/Kajabi:** Course structure via API or scraping, no native community (pair with Skool/Discord)
- **Custom platforms:** Adapt scraping to the platform's DOM structure; the ingestion pipeline and chatbot layer stay identical

## Future Improvements

Once the v1 is running, these upgrades increase value:
1. Usage analytics — track what members ask, surface content gaps
2. Auto-escalation — route unanswerable questions to human educators
3. Webhook sync — real-time content updates vs daily batch
4. Platform embed — iframe or widget for native-feeling integration
5. Multi-language support — translate content for international communities
