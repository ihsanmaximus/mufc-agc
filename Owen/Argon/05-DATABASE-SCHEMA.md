# 05 — DATABASE SPECIFICATION & PHYSICAL SCHEMA

**Document Ref:** TC-DOC-05-DB-SCHEMA  
**System:** PostgreSQL 16 Physical DDL, Indexing & Extensions  
**Architectural Authority:** Level 2 Physical Implementation Blueprint  

---

## 1. Database Configuration & Extensions

```sql
-- PostgreSQL 16 Enterprise Physical Schema Blueprint
-- The Champion ("It's All About MU")

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "vector";

SET TIME ZONE 'UTC';
```

---

## 2. Complete PostgreSQL 16 DDL

```sql
-- ============================================================================
-- 1. SOURCE REGISTRY & MONITORING
-- ============================================================================

CREATE TABLE sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    source_type VARCHAR(50) NOT NULL CHECK (source_type IN ('RSS', 'REST_API', 'YOUTUBE_PLAYLIST', 'OFFICIAL_WEB')),
    base_url TEXT NOT NULL,
    tier INT DEFAULT 2 CHECK (tier IN (1, 2, 3)),
    is_active BOOLEAN DEFAULT TRUE,
    polling_frequency_seconds INT DEFAULT 600,
    config JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE source_health (
    source_id UUID PRIMARY KEY REFERENCES sources(id) ON DELETE CASCADE,
    last_poll_at TIMESTAMPTZ,
    last_success_at TIMESTAMPTZ,
    consecutive_failures INT DEFAULT 0,
    last_http_status INT,
    last_error_message TEXT,
    is_quarantined BOOLEAN DEFAULT FALSE,
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================================================
-- 2. RAW AUDIT & INGESTION LAYER
-- ============================================================================

CREATE TABLE raw_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_id UUID NOT NULL REFERENCES sources(id) ON DELETE RESTRICT,
    external_id TEXT NOT NULL,
    content_hash CHAR(64) NOT NULL,
    raw_payload JSONB NOT NULL,
    fetched_at TIMESTAMPTZ DEFAULT NOW(),
    processing_status VARCHAR(30) DEFAULT 'PENDING' CHECK (processing_status IN ('PENDING', 'PROCESSING', 'PROCESSED', 'FAILED', 'DISCARDED')),
    error_log TEXT,
    CONSTRAINT uq_source_external_id UNIQUE (source_id, external_id)
);

CREATE INDEX idx_raw_items_hash ON raw_items(content_hash);
CREATE INDEX idx_raw_items_status ON raw_items(processing_status) WHERE processing_status = 'PENDING';

-- ============================================================================
-- 3. AUTHORITATIVE SPORTS DOMAIN
-- ============================================================================

CREATE TABLE competitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_id VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    code VARCHAR(20) NOT NULL, -- 'PL', 'FAC', 'UEL', 'UCL', 'EFL'
    season VARCHAR(20) NOT NULL,
    logo_url TEXT,
    tier INT DEFAULT 1,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE teams (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_id VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    short_name VARCHAR(50) NOT NULL,
    tla CHAR(3) NOT NULL, -- 'MUN', 'LIV', 'MCI'
    crest_url TEXT,
    is_primary_club BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE matches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    competition_id UUID NOT NULL REFERENCES competitions(id),
    external_id VARCHAR(50) UNIQUE NOT NULL,
    home_team_id UUID NOT NULL REFERENCES teams(id),
    away_team_id UUID NOT NULL REFERENCES teams(id),
    kickoff_time TIMESTAMPTZ NOT NULL,
    status VARCHAR(30) NOT NULL CHECK (status IN ('SCHEDULED', 'TIMED', 'IN_PLAY', 'PAUSED', 'FINISHED', 'POSTPONED', 'CANCELLED')),
    home_score INT DEFAULT 0,
    away_score INT DEFAULT 0,
    period VARCHAR(20) CHECK (period IN ('PRE', '1H', 'HT', '2H', 'ET', 'PK', 'FT')),
    minute INT,
    stats JSONB DEFAULT '{}'::jsonb,
    lineups JSONB DEFAULT '{}'::jsonb,
    official_broadcast_url TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_matches_kickoff ON matches(kickoff_time DESC);
CREATE INDEX idx_matches_status ON matches(status);

CREATE TABLE match_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    match_id UUID NOT NULL REFERENCES matches(id) ON DELETE CASCADE,
    minute INT NOT NULL,
    extra_minute INT,
    event_type VARCHAR(50) NOT NULL CHECK (event_type IN ('GOAL', 'ASSIST', 'YELLOW_CARD', 'RED_CARD', 'SUBSTITUTION', 'VAR_DECISION', 'PENALTY_SAVED')),
    team_id UUID NOT NULL REFERENCES teams(id),
    player_name VARCHAR(100) NOT NULL,
    detail JSONB DEFAULT '{}'::jsonb,
    external_event_id VARCHAR(100) UNIQUE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_match_events_match ON match_events(match_id, minute ASC);

CREATE TABLE standings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    competition_id UUID NOT NULL REFERENCES competitions(id),
    team_id UUID NOT NULL REFERENCES teams(id),
    position INT NOT NULL,
    played INT NOT NULL,
    won INT NOT NULL,
    drawn INT NOT NULL,
    lost INT NOT NULL,
    goals_for INT NOT NULL,
    goals_against INT NOT NULL,
    goal_difference INT NOT NULL,
    points INT NOT NULL,
    form VARCHAR(20),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    CONSTRAINT uq_comp_team UNIQUE (competition_id, team_id)
);

CREATE INDEX idx_standings_comp_pos ON standings(competition_id, position ASC);

-- ============================================================================
-- 4. CONTENT INTELLIGENCE & EDITORIAL
-- ============================================================================

CREATE TABLE story_clusters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    canonical_title TEXT NOT NULL,
    primary_category VARCHAR(50) NOT NULL,
    first_detected_at TIMESTAMPTZ DEFAULT NOW(),
    last_updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE processed_content (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    raw_item_id UUID REFERENCES raw_items(id) ON DELETE SET NULL,
    story_cluster_id UUID REFERENCES story_clusters(id) ON DELETE SET NULL,
    content_type VARCHAR(50) NOT NULL CHECK (content_type IN ('MATCH_REPORT', 'MATCH_PREVIEW', 'INJURY_UPDATE', 'TRANSFER_CONFIRMED', 'TRANSFER_RUMOUR', 'TACTICAL_ANALYSIS', 'INTERVIEW', 'GENERAL_NEWS')),
    title TEXT NOT NULL,
    slug TEXT UNIQUE NOT NULL,
    summary_id TEXT NOT NULL,
    body_markdown TEXT NOT NULL,
    transfer_status VARCHAR(30) CHECK (transfer_status IN ('CONFIRMED', 'ADVANCED', 'RUMOUR', NULL)),
    confidence_score NUMERIC(4,3) NOT NULL CHECK (confidence_score >= 0.0 AND confidence_score <= 1.0),
    fact_check_passed BOOLEAN DEFAULT TRUE,
    provenance JSONB NOT NULL,
    minhash_signature TEXT,
    embedding vector(768),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_processed_content_embedding ON processed_content USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
CREATE INDEX idx_processed_content_slug ON processed_content(slug);

-- ============================================================================
-- 5. VIDEO INTELLIGENCE
-- ============================================================================

CREATE TABLE videos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    youtube_video_id VARCHAR(50) UNIQUE NOT NULL,
    channel_id VARCHAR(100) NOT NULL,
    channel_title VARCHAR(150) NOT NULL,
    title TEXT NOT NULL,
    description TEXT,
    published_at TIMESTAMPTZ NOT NULL,
    thumbnail_url TEXT NOT NULL,
    video_category VARCHAR(50) DEFAULT 'HIGHLIGHTS' CHECK (video_category IN ('HIGHLIGHTS', 'PRESS_CONFERENCE', 'INTERVIEW', 'CLASSIC_MATCH', 'TRAINING')),
    is_embeddable BOOLEAN DEFAULT TRUE,
    related_match_id UUID REFERENCES matches(id) ON DELETE SET NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_videos_match ON videos(related_match_id);

-- ============================================================================
-- 6. PUBLISHING & PRESENTATION LAYER
-- ============================================================================

CREATE TABLE published_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content_id UUID REFERENCES processed_content(id) ON DELETE CASCADE,
    video_id UUID REFERENCES videos(id) ON DELETE CASCADE,
    section_code VARCHAR(50) NOT NULL CHECK (section_code IN ('HERO_PRIMARY', 'MATCH_CENTER', 'TRANSFER_HUB', 'WATCH_AGAIN', 'NEWS_FEED', 'STANDINGS_WIDGET')),
    display_priority INT DEFAULT 0,
    status VARCHAR(30) DEFAULT 'PUBLISHED' CHECK (status IN ('DRAFT', 'PENDING_REVIEW', 'PUBLISHED', 'ARCHIVED', 'REJECTED')),
    published_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ,
    cache_tag VARCHAR(100) NOT NULL,
    CONSTRAINT check_single_content_type CHECK (
        (content_id IS NOT NULL AND video_id IS NULL) OR 
        (content_id IS NULL AND video_id IS NOT NULL)
    )
);

CREATE INDEX idx_published_delivery ON published_items(section_code, display_priority DESC, published_at DESC) WHERE status = 'PUBLISHED';

-- ============================================================================
-- 7. AUDIT & TELEMETRY
-- ============================================================================

CREATE TABLE pipeline_job_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    queue_name VARCHAR(100) NOT NULL,
    job_id VARCHAR(100) NOT NULL,
    stage_number INT NOT NULL,
    stage_name VARCHAR(100) NOT NULL,
    duration_ms INT NOT NULL,
    status VARCHAR(30) NOT NULL CHECK (status IN ('SUCCESS', 'FAILED', 'RETRY')),
    tokens_consumed INT DEFAULT 0,
    cost_usd NUMERIC(8,6) DEFAULT 0.000000,
    error_details JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_job_logs_created ON pipeline_job_logs(created_at DESC);
```