
---
title: "GlobeNews — Top 5 Stories of the Day on an Interactive 3D Globe"
author: Krishna
stage: Prototype
market: Digital News / Data Visualization / Consumer Web
stack: React + Three.js / Globe.GL, Claude API (Anthropic), NewsAPI or GDELT, Vite, Tailwind CSS
date: 2026-03-30
---

---

## 00 · Problem Statement

### Core Insight

News consumption is broken in two opposite directions: people either doomscroll through an endless anxiety-inducing feed, or they check out entirely. There is almost no product that delivers **a bounded, beautiful, spatial understanding of what is happening in the world today** — the feeling of holding the planet in your hands and seeing where the stories are coming from.

The world is a place. News happens in places. Yet every major news product renders location as a byline at best, and ignores it completely at worst.

### Pain Clusters

**Cluster 1 — Cognitive overload from unbounded feeds**
Reuters Institute (2025) found that news avoidance is rising because readers experience anger, fear, and sadness from exposure to too much news. PressReader data shows political content consumption dropped 12% in 2025. People want signal, not noise.

**Cluster 2 — No spatial context**
When a story breaks in Taipei, Lagos, or Bogotá, readers have no visceral sense of *where* that is or why it matters relative to other events happening today. Geography is stripped out by the algorithmic feed format.

**Cluster 3 — Attention-hostile formats competing for the wrong thing**
Average social media attention per post is down to ~8 seconds (2025 data). The solution isn't to make news faster — it's to make it *experiential* enough to warrant a genuine dwell. A 3D globe that auto-spins to the day's 5 biggest stories is a format that earns attention through wonder, not through anxiety or FOMO.

**Cluster 4 — No obvious daily "done" moment**
Infinite scroll has no off-ramp. A top-5 globe has one: you've seen the five stories, you understand the shape of the day, you're done. This is a product philosophy, not a feature.

---

## 01 · Target User

### Who

**Primary:** Curious adults 25–45 who consider themselves "news-aware" but are actively reducing doomscrolling. They have a global mindset — traveled, educated, often work in knowledge industries. They open a news app once or twice a day looking for a "pulse check," not a deep read.

**Secondary:** Students and educators who want a spatial/visual way to discuss current events.

### Trigger Moments

- Morning routine (7–9am): "What happened overnight?"
- Lunch break (12–1pm): "Did anything big break this morning?"
- Pre-sleep (9–10pm): "What were the day's headlines?"

### Mental Model

The user is not looking for a newspaper. They want the feeling of a **morning briefing from a well-informed colleague** — compressed, confident, no hedging. The globe format matches this: it presents a finished, curated take on the world, not a raw stream.

### Trust Baseline

Medium-low. This user has seen AI hallucinate news. They will be skeptical of AI-generated synopses. The product must cite source outlets on every card (e.g., "per Reuters") and link to full articles. The 3D visual framing actually helps here — it signals "this is a visualization tool, not a news outlet," which lowers the burden of editorial trust.

---

## 02 · Existing Solutions & Gaps

| Product | What It Does | Why It Fails for This Use Case |
|---|---|---|
| **Globanix** | Real-time globe with 194 news sources, AI flagging | Too many markers, no curated "top 5," no 3D objects/animations — just dots. Looks like a data dashboard, not an experience. |
| **Google News / Apple News** | Algorithmic feed, personalized | Infinite scroll, no geography, no "done" moment, no spatial delight. |
| **The Daily / Up First** | Audio briefing, top stories | No visual, no spatial dimension, linear format. |
| **Flourish 3D Globe** | No-code data viz tool | Static, no real-time news, no AI pipeline, no click-to-synopsis UX. |
| **ArcGIS Earth** | Enterprise geospatial platform | Enterprise/GIS use case. Not a consumer news product. |
| **GitHub globe.gl** | Open-source WebGL globe | Library only, no product layer, no news integration. |

### The Actual Gap

No product combines: (1) daily editorial curation to exactly 5 stories, (2) a visually distinct 3D object/animation *per story type*, (3) a globe as the primary nav paradigm, (4) an AI-generated synopsis that cites its source. The closest thing — Globanix — is still a map with dots. The wedge is **spatial delight + radical constraint (top 5 only)**.

---

## 03 · MVP Scope

### In Scope

| Feature | Why |
|---|---|
| Interactive 3D globe (drag, rotate, zoom) | Core experience — non-negotiable |
| 5 animated 3D markers placed at story lat/lon | The key differentiator; each story type gets a distinct visual object |
| Click-to-open synopsis card (150 words, source link) | Core utility loop |
| AI geotagging: extract lat/lon from headline + body | Required to place stories on globe |
| AI synopsis generation with mandatory source citation | Core AI feature |
| Daily refresh (once per day, 6am UTC) | Sets the "bounded" product promise |
| Category-to-3D-object mapping (5 categories) | Drives the visual variety |
| Mobile-responsive (touch drag on globe) | Majority of access will be mobile |
| Dark mode globe aesthetic (space background) | Essential to the visual language |

### Out of Scope (v1)

| Feature | Why Not |
|---|---|
| User personalization / accounts | Adds auth complexity; contradicts "universal daily briefing" concept |
| More than 5 stories | Breaks the product's core philosophy |
| Real-time / live updates | Too complex; daily cadence is the MVP promise |
| Audio narration | Nice-to-have, Phase 2 |
| Country-level browsing | Globanix already does this; not our wedge |
| Social sharing of individual stories | Phase 2 |
| Push notifications | Phase 2 |
| Non-English languages | Phase 2 |

### The 5 Category → 3D Object Map (MVP)

| Category | 3D Object / Animation |
|---|---|
| Geopolitics / War / Diplomacy | Pulsing red shield or castle tower |
| Natural disaster / Climate | Animated tornado / wave spiral |
| Economy / Markets | Rising gold bar / coin stack |
| Science / Technology | Orbiting satellite ring |
| Human interest / Social | Glowing lantern |

---

## 04 · AI System Design

### Pipeline Overview

```
[NewsAPI/GDELT raw feed] 
    → Step 1: Story Ranking (Deterministic)
    → Step 2: Geolocation Extraction (AI)
    → Step 3: Category Classification (AI)
    → Step 4: Synopsis Generation (AI)
    → Step 5: Output Assembly (Deterministic)
    → [Stored JSON served to frontend]
```

---

### Step 1: Story Ranking — DETERMINISTIC

**Input:** Raw RSS/API feed from NewsAPI (top-headlines endpoint, English, all sources), last 24h.

**Process:** Sort by `(article_count_per_cluster × source_tier_weight)`. Source tier is a hardcoded lookup table (e.g., Reuters = 1.0, AP = 1.0, BBC = 0.95, Buzzfeed = 0.4). Cluster similar stories by keyword overlap using TF-IDF cosine similarity (threshold ≥ 0.6). Select top 5 clusters.

**Output:** `top5_clusters[]` — each cluster is a list of raw articles about the same story.

**Failure Mode:** Fewer than 5 clusters → backfill with next-ranked clusters regardless of tier weight. If total articles < 5 → surface error to admin, do not publish.

**Why deterministic:** Ranking by coverage volume and source credibility is a math problem, not a judgment problem. Never use AI for this.

---

### Step 2: Geolocation Extraction — AI

**Input per story:** `headline` (string), `body_excerpt` (first 400 chars of lead article)

**Prompt:**
```
You are a geographic parser. Given a news headline and excerpt, 
return ONLY the primary geographic location of this event as JSON:
{"lat": float, "lon": float, "location_name": string, "confidence": "high"|"medium"|"low"}
If no specific location exists (e.g., a purely financial story), 
return the HQ country of the primary subject.
Never return null lat/lon — default to country centroid if needed.
```

**Output Schema:**
```json
{
  "lat": 25.2048,
  "lon": 55.2708,
  "location_name": "Dubai, UAE",
  "confidence": "high"
}
```

**Failure Mode:** If `confidence = "low"`, fall back to keyword-matching against a country centroid lookup table (deterministic fallback). Log for human review.

---

### Step 3: Category Classification — AI

**Input:** `headline` + `body_excerpt`

**Prompt:**
```
Classify this news story into exactly ONE of these five categories:
geopolitics | disaster | economy | science | human_interest
Return only the category label. No explanation.
```

**Output Schema:**
```json
{ "category": "geopolitics" }
```

**Failure Mode:** If model returns a value outside the 5 allowed labels (rare with strict prompt), default to `"human_interest"`. Never crash the pipeline on classification failure.

---

### Step 4: Synopsis Generation — AI

**Input:** `headline`, `body_excerpt` (400 chars), `source_name`, `source_url`

**Prompt:**
```
You are a concise news briefer. Write a 2–3 sentence synopsis of 
this news story. Rules:
- Max 60 words
- Plain language, no jargon
- End with: "— Source: [source_name]"
- Do NOT editorialize or add opinions
- Do NOT invent facts not present in the headline and excerpt
```

**Output Schema:**
```json
{ "synopsis": "string (max 60 words, ends with '— Source: X')" }
```

**Failure Mode:** If synopsis exceeds 80 words or does not contain "— Source:", retry once. On second failure, use headline + "Full story at [source_name]" as the fallback synopsis. Never display a synopsis without a source attribution.

---

### Step 5: Output Assembly — DETERMINISTIC

Merge all AI outputs into the canonical `DailyGlobe` JSON object and write to CDN/static file (`/api/today.json`). Timestamp with UTC date. No AI involved.

---

### Canonical Output Schema

```json
{
  "date": "2026-03-30",
  "generated_at": "2026-03-30T06:02:14Z",
  "stories": [
    {
      "id": "story_001",
      "rank": 1,
      "headline": "string",
      "synopsis": "string (max 60 words, ends with — Source: X)",
      "source_name": "string",
      "source_url": "string (URL)",
      "category": "geopolitics | disaster | economy | science | human_interest",
      "geo": {
        "lat": 0.0,
        "lon": 0.0,
        "location_name": "string",
        "confidence": "high | medium | low"
      },
      "marker": {
        "object_type": "shield | tornado | gold_bar | satellite | lantern",
        "color_hex": "#FF4444",
        "pulse_animation": true
      }
    }
  ]
}
```

### Agent Boundaries (What AI May and May Not Do)

| Action | AI | Deterministic |
|---|---|---|
| Rank stories by coverage volume | ✗ | ✓ |
| Extract lat/lon from text | ✓ | Fallback only |
| Classify story category | ✓ | Fallback only |
| Write 60-word synopsis | ✓ | |
| Choose which source to link | ✗ | ✓ (top article in cluster) |
| Assign 3D object type | ✗ | ✓ (category → object lookup) |
| Decide to run/not run pipeline | ✗ | ✓ (cron, min cluster count check) |
| Claim a story is "breaking" or "urgent" | ✗ | ✗ (prohibited entirely in v1) |

---

## 05 · Prototype Validation Plan

### Bet 1: The Globe Format Earns Longer Dwell Than a List

**Hypothesis:** Users who see the top 5 stories presented on a 3D globe will spend more time on the page and open more story cards than users who see the same 5 stories as a plain vertical list.

**Test Design:**
- Build two versions: Globe (A) vs. Styled List with same story cards (B)
- Recruit 30 participants via UserTesting.com (target: 25–45, news-curious, US/UK)
- Measure: Time on page, number of cards opened, qualitative reaction

**Success Metric:**
- Globe version: avg dwell ≥ 90 seconds (vs. expected ~35s for a list)
- Globe version: ≥ 3 of 5 cards opened on average
- Qualitative: "I'd come back to this" from ≥ 60% of participants

**Test Script (5 key questions):**
1. "Without me telling you anything — what do you think this site does?"
2. "Walk me through what you'd do first on this page."
3. "Click on the story that interests you most. Tell me what you expected to see."
4. "If you were told this refreshes every morning with today's 5 top stories, how often might you visit?"
5. "What's missing? What would make you actually bookmark this?"

---

### Bet 2: The AI Synopsis Is Trusted Because It Cites a Source

**Hypothesis:** Users will accept and trust a 60-word AI-generated synopsis as long as it ends with a named source attribution — without needing to verify the linked article.

**Test Design:**
- Show 10 users three synopsis cards side by side: (A) AI synopsis with "— Source: Reuters", (B) AI synopsis with no attribution, (C) The actual Reuters lede paragraph
- Ask: "Which of these would you trust to give you the gist of this story?"
- Follow up: "Would you feel misled if you later found out (A) was written by AI?"

**Success Metric:**
- ≥ 70% of users rate (A) as "trustworthy enough" without clicking the source link
- ≤ 20% feel "misled" when told (A) is AI-generated with real source data
- No user prefers (B) over (A) — source attribution is the critical trust lever

---

## 06 · Open Questions

| Question | Owner | Priority |
|---|---|---|
| Which news data source? NewsAPI (paid, clean) vs GDELT (free, noisy, global) vs direct RSS parsing? | Krishna / Engineering | 🔴 Blocker — decide before pipeline build |
| What's the daily pipeline trigger? Vercel cron job, GitHub Actions, or manual admin trigger for prototype? | Engineering | 🟡 High — needed for day-1 demo |
| How do we handle stories with no clear geographic anchor (e.g., "Fed raises rates")? Do we place them at Washington DC by default, or show them as a floating orbital object above the globe? | Design / Product | 🟡 High — affects 1–2 stories per day |
| Globe performance on mid-range Android? Globe.GL has known framerate issues on lower-end devices with dense point layers. With only 5 markers, this should be fine, but needs a real-device test. | Engineering | 🟡 High |
| Do we auto-rotate the globe on load (cinematic spin to story 1) or let the user take control immediately? | Design | 🟢 Medium — UX polish decision |
| How do we handle days when 2+ top stories are in the same country/region? Do markers overlap, stack, or use a slight lat/lon offset? | Design / Engineering | 🟢 Medium |
| Is there a legal / copyright concern with displaying 60-word summaries of articles from commercial outlets like AP or NYT? | Legal / Product | 🟡 High — needs a quick legal read before public launch |
| Should we build a shareable "Today's Globe" static image for social (e.g., a screenshot card of the globe + 5 headlines)? | Product | 🟢 Low — Phase 2 candidate |
