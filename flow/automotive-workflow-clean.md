# AUTOMOTIVE INDUSTRY AUTOMATION PIPELINE WITH SILENT OBSERVER

**Status:** Clean, Deduplicated, Neon Database Ready, Silent Observer Integrated

---

## STEP 1: HOURLY TRIGGER FIRES

Every hour, a scheduled trigger wakes up the system.
It searches specifically for automotive industry content published in the last 60 minutes.

**Search criteria:**
- Breaking news
- Breakthroughs
- Regulations
- Drama
- Innovation
- Controversy
- Recalls
- Market shocks
- Clickbait-worthy angles allowed

**Collects:**
- Article URLs
- Source names
- Publish timestamps
- Headlines
- Full raw page content (HTML/markdown/text - no summarization)

---

## STEP 2: RAW INGEST → NEON DATABASE (TABLE: raw_articles)

All raw articles are immediately stored into Neon Database.

**Table columns:**
- url
- source
- publish_time
- headline
- raw_content
- auto_category_guess
- status (RAW_INGEST)
- created_at
- processing_id

**Purpose:**
- Preserves ground truth
- Allows replay
- No filtering yet
- No decisions made

---

## STEP 3: BUFFER (3 MINUTES)

System pauses to ensure:
- All writes complete
- No partial records
- No race conditions
- Downstream agents see stable data

---

## STEP 4: POLICY TRIAGE → NEON DATABASE (TABLE: policy_evaluation)

Three independent policy agents evaluate each raw article.

**They assess:**
- TikTok Community Guidelines compliance
- Creator Marketplace rules
- Misinformation risk
- Violence/illegal framing
- Medical/financial claims
- Dangerous behavior cues

They label but do NOT rewrite content.

**Output columns:**
- article_id
- policy_risk (NONE / LOW / MED / HIGH)
- policy_notes
- allowed_framing
- agent_id
- decision (KILLED / RESTRICTED_FRAMING / POLICY_PASSED)

**Decision logic:**
- HIGH risk → KILLED (never moves forward)
- MED risk → RESTRICTED_FRAMING (allowed but with constraints)
- LOW/NONE → POLICY_PASSED (continues)

---

## STEP 5: BUFFER (3 MINUTES)

Stabilization pause before viral scoring.

---

## STEP 6: VIRAL POTENTIAL SCORING → NEON DATABASE (TABLE: viral_candidates)

Viral scoring agents read policy-passed articles.

**They evaluate:**
- Would this stop a scroll in <2 seconds?
- Can it generate comments/arguments?
- Is it visually explainable?
- Is it emotionally charged?

**Scoring metrics:**
- Wow factor (1-10)
- Clickbait elasticity
- Comment bait potential
- Visual potential
- Time sensitivity

**Rule:** Only articles scoring ≥7 survive

**Output columns:**
- article_id
- wow_score
- viral_angle_notes
- comment_bait_potential
- visual_potential
- status (VIRAL_CANDIDATE)

---

## STEP 7: BUFFER (3 MINUTES)

Pause before creative expansion.

---

## STEP 8: CONTENT EXPANSION → NEON DATABASE (TABLE: video_concepts)

Expansion agents take each viral article and generate multiple video ideas.

**Important:** They do NOT invent new topics.

**Per article, they generate:**
- 3-10 different video angles
- Hook concepts (first 1.5-2 seconds)
- Emotional framings
- Suggested formats (short/loop/story)

**Output columns:**
- article_id
- video_angle
- hook_concept
- emotion_lever
- suggested_format
- status (CONCEPT_READY)

**Key point:** One article → multiple rows in this table.

---

## STEP 9: BUFFER (3 MINUTES)

Pause before validation.

---

## STEP 10: VALIDATION & RANKING → NEON DATABASE (TABLE: approved_concepts)

Validation agents act as brutal editors.

**They remove:**
- Weak hooks
- Redundant angles
- Risky phrasing
- Boring concepts
- Duplicates

**They rank survivors on:**
- Retention likelihood
- Hook clarity
- Comment probability
- TikTok format compatibility
- Algorithm fit

**Output columns:**
- concept_id
- video_angle
- hook
- approved_framing
- final_rank
- status (APPROVED_FOR_SCRIPT)

Only top-tier concepts survive.

---

## STEP 11: BUFFER (3 MINUTES)

Pause before script synthesis.

---

## STEP 12: SCRIPT + METADATA → NEON DATABASE (TABLE: ready_for_video)

Script agent synthesizes each approved concept into complete video package.

**Creates for each row:**
- Final script
- On-screen text
- Hook (1.5s)
- Caption
- Hashtags (optimized)
- Description
- CTA (comment/save/follow variant)
- Video generation prompt
- Status (READY_FOR_VIDEO)

Each row is now: "Ready to generate video"

---

## STEP 13: VIDEO GENERATION (SORA / VEO / REPLICATE)

Video generator reads from ready_for_video table.

**For each row:**
- Uses script + visual instructions
- No avatars needed (use prompt descriptions)
- Humans can be described in text
- B-roll + kinetic text works fine

**Output:**
- Video file (URL/path saved back to database)
- Thumbnail frame (URL/path saved)
- Video duration
- Resolution metadata

---

## STEP 14: SCHEDULING + TIKTOK UPLOAD → NEON DATABASE (TABLE: posted_videos)

Scheduling/Algorithm agent reads from ready_for_video.

**Decides:**
- When to post (optimal time)
- Spacing between posts
- Account to post from
- Hashtag density
- Caption length adjustments
- Avoids spam behavior

**Action:**
- Uploads video via TikTok API
- Posts caption, hashtags, description
- Confirms upload success
- Records TikTok video ID

**Output columns:**
- video_id
- tiktok_id
- post_time
- account_id
- upload_status
- post_url
- posted_at

⚠️ **THIS IS WHERE VIDEO IS ACTUALLY UPLOADED TO TIKTOK** ⚠️

---

## STEP 15: PERFORMANCE COLLECTION → NEON DATABASE (TABLE: video_performance)

Over 24-48 hours, performance metrics are collected and aggregated.

**Metrics tracked:**
- views
- watch_time
- completion_rate
- comments
- likes
- saves
- share_velocity
- engagement_rate
- account_health_impact
- algorithm_favorability_proxy

Updated regularly from TikTok API.

---

## STEP 16: SILENT OBSERVER (LOCAL AGENT) - WATCHES EVERYTHING

The Silent Observer runs locally and ingests data from all tables:
- raw_articles
- policy_evaluation
- viral_candidates
- video_concepts
- approved_concepts
- ready_for_video
- posted_videos
- video_performance

**What it observes (logged in local event store):**
- Every decision made
- Why it was made
- Input data
- Output data
- Actual outcome
- Prediction vs reality comparison
- Transformation applied at each stage

**It learns offline by:**
- Comparing predicted scores vs actual performance
- Identifying over-filtering vs under-filtering
- Detecting agent bias patterns
- Finding prompt drift
- Spotting overfit to past wins

**It produces (but takes NO actions):**
- Analysis reports
- Pattern detection
- Scoring adjustment recommendations
- New heuristic suggestions
- Warnings about drift

These are provided to human reviewer, not automatically applied.

---

## STEP 17: FEEDBACK LOOP (READ-ONLY DISTRIBUTION)

Other agents read performance data to improve future judgment:

**Policy agents learn:**
- Which policy classifications were correct
- Which killed content in hindsight was safe
- Which risky content performed well
- Adjusts scoring weights for next cycle

**Viral agents learn:**
- Wow score vs actual performance correlation
- Overestimated signals
- Underestimated signals
- Emotion lever effectiveness
- Improves future viral scoring

**Validation agents learn:**
- Which approved concepts flopped
- Which hooks looked good but died
- Format patterns that win
- Framing patterns that win
- Tightens future validation

**Algorithm agent learns:**
- Post time vs velocity patterns
- Caption length impact on retention
- Hashtag density vs suppression correlation
- Spacing effects on algorithm favorability

None of these agents modify the past.
They only adjust future judgment.

---

## STEP 18: NEXT HOURLY CYCLE

System resets for next hour with:
- Slightly better scoring
- Slightly better judgment
- Slightly less waste
- Same pipeline
- Smarter behavior

---

## SILENT OBSERVER - LOCAL FINE-TUNING

The Silent Observer's accumulated logs are used to:
- Fine-tune local LLM (Ollama)
- Improve prompt efficiency
- Create specialized models for each pipeline stage
- Build better scoring functions
- Detect anomalies before they cause problems

**Storage:** Local disk/database
**Access:** Only to you
**Visibility:** Hidden from TikTok, hidden from users
**Latency:** Zero (no API calls)
**Memory:** Unlimited (accumulates forever)

---

## DATABASE SCHEMA (NEON POSTGRESQL)

### Table: raw_articles
- id (UUID)
- url (TEXT)
- source (TEXT)
- publish_time (TIMESTAMP)
- headline (TEXT)
- raw_content (TEXT)
- auto_category (TEXT)
- status (TEXT: RAW_INGEST)
- created_at (TIMESTAMP)
- processing_id (UUID)

### Table: policy_evaluation
- id (UUID)
- article_id (FOREIGN KEY)
- policy_risk (TEXT: NONE/LOW/MED/HIGH)
- policy_notes (TEXT)
- allowed_framing (TEXT)
- agent_id (TEXT)
- decision (TEXT: KILLED/RESTRICTED/PASSED)
- created_at (TIMESTAMP)

### Table: viral_candidates
- id (UUID)
- article_id (FOREIGN KEY)
- wow_score (NUMERIC 1-10)
- viral_angle_notes (TEXT)
- comment_bait_potential (NUMERIC)
- visual_potential (NUMERIC)
- status (TEXT: VIRAL_CANDIDATE)
- created_at (TIMESTAMP)

### Table: video_concepts
- id (UUID)
- article_id (FOREIGN KEY)
- video_angle (TEXT)
- hook_concept (TEXT)
- emotion_lever (TEXT)
- suggested_format (TEXT)
- status (TEXT: CONCEPT_READY)
- created_at (TIMESTAMP)

### Table: approved_concepts
- id (UUID)
- concept_id (FOREIGN KEY)
- video_angle (TEXT)
- hook (TEXT)
- approved_framing (TEXT)
- final_rank (NUMERIC)
- status (TEXT: APPROVED_FOR_SCRIPT)
- created_at (TIMESTAMP)

### Table: ready_for_video
- id (UUID)
- concept_id (FOREIGN KEY)
- script (TEXT)
- on_screen_text (TEXT)
- hook (TEXT)
- caption (TEXT)
- hashtags (TEXT)
- description (TEXT)
- cta (TEXT)
- video_prompt (TEXT)
- status (TEXT: READY_FOR_VIDEO)
- created_at (TIMESTAMP)

### Table: posted_videos
- id (UUID)
- video_id (FOREIGN KEY)
- tiktok_id (TEXT)
- post_time (TIMESTAMP)
- account_id (TEXT)
- upload_status (TEXT)
- post_url (TEXT)
- posted_at (TIMESTAMP)
- created_at (TIMESTAMP)

### Table: video_performance
- id (UUID)
- posted_video_id (FOREIGN KEY)
- views (INTEGER)
- watch_time (NUMERIC)
- completion_rate (NUMERIC)
- comments (INTEGER)
- likes (INTEGER)
- saves (INTEGER)
- share_velocity (NUMERIC)
- engagement_rate (NUMERIC)
- account_health (NUMERIC)
- algorithm_proxy (NUMERIC)
- updated_at (TIMESTAMP)

---

## END-TO-END SUMMARY

```
Hourly Trigger 
→ Raw Search (Tavily) 
→ Raw Ingest (Neon) 
→ Buffer 3min 
→ Policy Triage (3 agents) 
→ Policy Results (Neon) 
→ Buffer 3min 
→ Viral Scoring (2 agents) 
→ Viral Candidates (Neon) 
→ Buffer 3min 
→ Content Expansion (2 agents) 
→ Video Concepts (Neon) 
→ Buffer 3min 
→ Validation (2 agents) 
→ Approved Concepts (Neon) 
→ Buffer 3min 
→ Script Synthesis (1 agent) 
→ Ready for Video (Neon) 
→ Video Generation (Sora/VEO/Replicate) 
→ TikTok Upload 
→ Posted Videos (Neon) 
→ Performance Collection 
→ Video Performance (Neon) 
→ Silent Observer Analysis (Local) 
→ Feedback Distribution 
→ Next Hourly Cycle
```

**Silent Observer** runs in parallel throughout, watches all tables, learns offline, provides insights for human review.

---

## KEY METRICS AT A GLANCE

| Stage | Input | Output | Filter Rule | Storage |
|-------|-------|--------|-------------|---------|
| Raw Ingest | Tavily results | All articles | None | raw_articles |
| Policy | raw_articles | Filtered articles | Kill HIGH risk | policy_evaluation |
| Viral | Policy-passed | Candidate articles | Score ≥7 | viral_candidates |
| Expansion | Viral articles | Video ideas | None (1→many) | video_concepts |
| Validation | Concepts | Top concepts | Rank by fit | approved_concepts |
| Script | Concepts | Full scripts | None | ready_for_video |
| Video Gen | Scripts | Video files | None | ready_for_video |
| Upload | Videos | Posted vids | None | posted_videos |
| Performance | TikTok API | Metrics | Continuous | video_performance |

---

## DUPLICATE REMOVAL LOG

✅ Removed 12+ duplicate "what you described is" sections
✅ Removed duplicate "3 minutes buffer" explanations 
✅ Removed duplicate PHASE explanations
✅ Removed all non-workflow conversation 
✅ Consolidated all Silent Observer explanations into Step 16
✅ Consolidated all feedback loop explanations into Step 17
✅ Replaced ALL Google Sheets with Neon Database (9 references)
✅ Created single source of truth for database schema

---

**Total reduction:** From 119,713 characters → Clean workflow document (~8,000 characters)
**Format:** Plain English, ready for Mermaid diagram conversion
**Status:** Production-ready
