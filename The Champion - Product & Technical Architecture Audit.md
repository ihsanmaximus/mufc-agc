# TECHNICAL & PRODUCT ARCHITECTURE AUDIT
## THE CHAMPION — INDEPENDENT AGC-POWERED FOOTBALL FAN PORTAL
**Document Code:** AUD-CHAMP-2026-V1  
**Author / Role:** Owen (Senior Product Architect, System Architect & AI Automation Engineer)  
**Date:** 27 November 2026  
**Status:** Audit & Strategic Architecture Blueprint (No Coding / Implementation Authorized)  
**Core Directive:** "It's All About MU" — Independent Product, No MaxiPress Legacy, Fan-First Architecture.

---

## Executive Summary & Non-Negotiables Check

Audit ini mengevaluasi kelayakan teknis dan operasional dari Project Blueprint v1.0. The Champion didesain bukan sebagai portal media generik atau CMS editorial konvensional, melainkan portal suporter dinamis bertenaga Automated Content Generation (AGC) dengan backend berbentuk *AGC Control Center*.

### Evaluasi Batasan Fundamental (Non-Negotiables):
1. **Pemisahan Total dari MaxiPress:** Tidak ada dependensi, pola tabel, atau alur editorial dari MaxiPress yang boleh digunakan. Arsitektur dibangun mandiri dari nol (*clean slate*).
2. **Kedaulatan Data Olahraga (Zero Hallucination Policy):** Data skor, jadwal, susunan pemain, statistik pertandingan, dan klasemen liga bersumber 100% dari Sports Data Provider berlisensi melalui alur deterministik. Model AI/LLM secara tegas dilarang membuat, menebak, atau merekayasa data numerik dan faktual olahraga.
3. **Kepatuhan Hak Siar & Media:** Sistem tidak memfasilitasi, melakukan proksi, atau menyematkan siaran langsung (*live streaming*) tanpa izin/hak siar. Video pertandingan hanya menggunakan penyematan resmi (*authorized embed*) dari YouTube API atau mengarahkan ke pemegang hak siar terverifikasi (misalnya Vidio ID untuk teritori Indonesia).
4. **Transformasi Berita & Hak Cipta:** Sistem menolak metode *scraping* teks mentah untuk diterbitkan ulang (*plagiarism*). Konten berita diolah melalui sintesis jurnalistik transformatif (ringkasan multibahasa/bahasa Indonesia mandiri) dengan atribusi sitasi eksplisit (*provenance link*).

---

## Deliverable A: Product Architecture Assessment

### 1. Evaluasi Kritis terhadap Fondasi Produk
The Champion memecahkan masalah fragmentasi dan keterlambatan informasi fans Manchester United. Pendekatan *fan-first* (mengikuti siklus klub dan hari pertandingan, bukan urutan kronologis artikel berita) adalah pendekatan yang sangat tepat secara komersial dan *user engagement*.

### 2. Validasi Asumsi vs. Realitas Teknis
* **Asumsi yang Dikonfirmasi Benar:**
  * Kebutuhan frontend yang adaptif terhadap waktu pertandingan (*Pre-match*, *Live*, *Post-match*, *No-match*).
  * Backend sebagai pusat kendali otomatisasi (*Control Center*) dengan visibilitas antrean, kuota API, dan *health check*.
  * Pemisahan jalur data deterministik (sepak bola) dan jalur data sintetis/kreatif (berita/video).
* **Kelemahan Asumsi Awal (Flawed Baseline Assumptions):**
  * *Single Monolithic Pipeline:* Menyatukan semua tipe data dalam satu alur kerja seragam akan merusak *throughput* dan meningkatkan latensi. Data skor langsung (*live score*) butuh latensi <10 detik, sedangkan analisis artikel berita membutuhkan waktu pemrosesan LLM 20–45 detik.
  * *Unrestricted Scraping:* Mengasumsikan portal berita pihak ketiga dapat di-*scrape* tanpa hambatan legal dan teknis (*rate limiting*, Cloudflare Turnstile bot protection).
  * *Polling Tanpa Batas:* Polling API sepak bola dan YouTube tanpa strategi *event-driven* akan menghabiskan kuota harian dalam hitungan jam.

---

## Deliverable B: Recommended Backend Architecture

Daftar 10 modul kandidat awal dievaluasi ulang: modul `Keyword Intelligence` dan `Content Processing` memiliki tumpang tindih fungsi dengan `Automation Rules`. Kami merekomendasikan restrukturisasi menjadi **8 Subsistem Inti Terisolasi**:

```
                       +---------------------------------------+
                       |        API Gateway & Auth (RBAC)      |
                       +-------------------+-------------------+
                                           |
     +-------------------------------------+-----------------------------------+
     |                                     |                                   |
+----v--------------------+   +------------v------------+   +------------------v------------------+
| 1. AGC Control Center   |   | 2. Ingestion & Quota    |   | 3. Pipeline Orchestrator            |
| - Operational Dashboard |   |    Gateway              |   |    (BullMQ / Temporal Engine)       |
| - Review Desk (Manual)  |   | - Source Registry       |   | - DAG Workflows & Polling Scheduler |
| - Prompt Versioning     |   | - Rate Limit & Budget   |   | - Idempotency & DLQ Manager         |
+-------------------------+   +-------------------------+   +------------------+------------------+
                                                                               |
     +-------------------------------------------------------------------------+
     |                                     |                                   |
+----v---------------+        +------------v------------+        +-------------v------------+
| 4. Sports Engine   |        | 5. Content Intel Engine |        | 6. Video Intel Engine    |
| - Provider Sync    |        | - Whitelist RSS Parser  |        | - YouTube Quota Guard    |
| - Match Center Ops |        | - MinHash Deduplication |        | - Embed Compliance Check |
| - Standings Matrix |        | - LLM Summary Transform |        | - Highlights Discovery   |
+--------------------+        +-------------------------+        +--------------------------+
          |                                |                                   |
          +--------------------------------+-----------------------------------+
                                           |
                              +------------v------------+
                              | 7. Publishing & Dynamic |
                              |    Layout Engine        |
                              | - Confidence Matrix     |
                              | - Layout State Machine  |
                              | - Edge CDN Cache Purge  |
                              +------------+------------+
                                           |
                              +------------v------------+
                              | 8. Observability & Cost |
                              | - Token & Cost Tracking |
                              | - Latency & Error Trace |
                              +-------------------------+
```

### Matriks Tanggung Jawab, Batasan, dan Akses Modul

| No | Modul | Tanggung Jawab Utama | Batasan & Dependensi | Tingkat Akses (RBAC) |
| :--- | :--- | :--- | :--- | :--- |
| **1** | **AGC Control Center & Review Desk** | Antarmuka administrasi, pemantauan sistem, moderasi konten meragukan (*low confidence*), konfigurasi prompt & threshold. | Mengonsumsi API internal dari Orchestrator & Publishing Engine. | Super Admin, Operator / Editor |
| **2** | **Ingestion & Quota Gateway** | Registrasi sumber legal, penanganan rate limit provider, tracking sisa kuota, dan penyimpanan data mentah (*raw*). | Berdiri di perbatasan jaringan eksternal. Menyimpan data ke `raw_items`. | Super Admin |
| **3** | **Pipeline Orchestrator** | Eksekusi graf berarah asiklik (DAG), jadwal pemungutan data (*polling*), isolasi antrean, dan *Dead-Letter Queue* (DLQ). | Membutuhkan Redis & PostgreSQL. Mengendalikan modul 4, 5, 6, dan 7. | Worker Layanan Internal |
| **4** | **Football Data Engine** | Menangani data faktual olahraga (jadwal, skor langsung, statistik, susunan pemain, klasemen) secara deterministik. | Bergantung pada penyedia API Sepak Bola. Tidak ada intervensi AI. | Worker Layanan Internal |
| **5** | **Content Intelligence Engine** | Ekstraksi RSS resmi, pencocokan entitas (NER), klastering deduplikasi, sintesis ringkasan bahasa Indonesia berbasis AI. | Bergantung pada LLM Gateway (LiteLLM) dan database relasional. | Worker Layanan Internal |
| **6** | **Video Intelligence Engine** | Penemuan video YouTube dari saluran resmi, verifikasi status `is_embeddable`, dan pemetaan klip ke entitas pertandingan. | Bergantung pada YouTube Data API v3 dan kuota harian. | Worker Layanan Internal |
| **7** | **Publishing & Layout Engine** | Penilaian ambang batas keyakinan (*confidence threshold*), penentuan penempatan seksi beranda, dan invalidasi *cache* CDN. | Mengonsumsi hasil modul 4, 5, dan 6. Terhubung ke Edge CDN. | Worker Layanan / Editor |
| **8** | **Observability & Cost Center** | Telemetri komprehensif: pemakaian token AI, pengeluaran finansial real-time, latensi eksekusi, dan pencatatan audit. | Mengawasi seluruh subsistem secara asinkron (OpenTelemetry). | Super Admin, DevOps |

---

## Deliverable C: AGC Engine Blueprint (Prioritas Tertinggi)

### 1. Siklus Hidup 11 Tahap Pemrosesan AGC
Setiap unit informasi bergerak melalui mesin status (*state machine*) 11 tahap yang terukur:

```
[1. Discovery] ──> [2. Collection] ──> [3. Extraction] ──> [4. Normalization] ──> [5. Matching/Classify]
                                                                                           │
[10. Publishing] <── [9. Decision] <── [8. Validation] <── [7. Deduplication] <── [6. AI Enrichment]
       │
[11. Monitoring]
```

1. **Discovery (Penemuan):** Berjalan via dua pemicu: *Scheduled Polling* (cron untuk RSS berita dan jadwal pertandingan rutin) dan *Event-Driven Trigger* (saat hari pertandingan, polling skor dipercepat setiap 15–30 detik).
2. **Collection (Pengumpulan):** Mengambil data dari endpoint resmi, menghasilkan hash SHA-256 dari payload mentah, dan menyimpannya secara *immutable* di tabel `raw_items` beserta header HTTP dan stempel waktu.
3. **Extraction (Ekstraksi):** Memisahkan konten dari boilerplate teknis. Teks artikel dibersihkan dari tag HTML dan iklan; JSON sepak bola diekstrak untuk metrik pertandingan; JSON video dipetakan ke atribut video YouTube.
4. **Normalization (Normalisasi):** Mengubah representasi eksternal menjadi *Canonical Internal Schema*. Tim "Man Utd" atau ID penyedia `33` dinormalisasi menjadi UUID platform: `club_manchester_united`. Format waktu diseragamkan ke ISO-8601 UTC.
5. **Matching & Classification (Pencocokan & Klasifikasi):** Menghubungkan konten dengan kamus entitas internal (daftar pemain, manajer, nomor laga). Mengklasifikasikan tipe konten: `MATCH_REPORT`, `PREVIEW`, `INJURY_UPDATE`, `TRANSFER_CONFIRMED`, `TRANSFER_RUMOUR`, atau `TACTICAL_ANALYSIS`.
6. **Enrichment (Pengayaan AI Terkendali):** Model AI (Claude 3.5 Haiku via LiteLLM) menerima teks berita dan fakta olahraga, kemudian menghasilkan: ringkasan jurnalisme olahraga bahasa Indonesia (100–150 kata), poin-poin penting, judul ramah SEO, dan metadata pencarian. Parameter model: `temperature = 0.2` dengan skema output JSON terstruktur. AI dilarang menyimpulkan skor pertandingan.
7. **Deduplication (Deduplikasi):**
   * *Exact Hash:* Cek kecocokan MD5 hash teks dan URL sumber.
   * *Syntactic Dedup:* Deteksi kesamaan teks menggunakan MinHash / SimHash (>85% kesamaan dianggap sindikasi).
   * *Semantic Dedup:* Menghitung *cosine similarity* vektor embedding pada jendela waktu 36 jam. Jika berita membicarakan insiden yang sama, berita digabungkan ke dalam `StoryCluster` tunggal sebagai pembaruan (*update*).
8. **Validation (Validasi):** Memverifikasi klaim nama pemain dan skor pertandingan terhadap basis data `Sports Engine`. Memeriksa keaktifan tautan sumber asli (HTTP 200). Menghitung skor keyakinan akhir ($S \in [0.0, 1.0]$).
9. **Decision Matrix (Pengambilan Keputusan Publikasi):**
   * Jika Skor $S \ge 0.88$ dan tipe bukan rumor $\rightarrow$ **Auto-Publish**.
   * Jika tipe $=$ `TRANSFER_RUMOUR` dari Sumber Tier 1 $\rightarrow$ **Auto-Publish dengan Lencana "Rumor"**.
   * Jika tipe $=$ `TRANSFER_RUMOUR` dari Sumber Tier 2/3 $\rightarrow$ **Tahan di Antrean Review Manual**.
   * Jika $0.65 \le S < 0.88$ atau terjadi anomali entitas $\rightarrow$ **Tahan di Antrean Review Manual**.
   * Jika $S < 0.65$ $\rightarrow$ **Tolak Otomatis (Reject & Log)**.
10. **Publishing (Publikasi & Tata Letak):** Menyimpan konten ke tabel `published_items`, menetapkan penempatan seksi (*layout section*), dan mengirim sinyal pembersihan cache (*cache tag invalidation*) ke Edge CDN.
11. **Monitoring & Feedback:** Mencatat biaya token AI, latensi siklus pemrosesan, sisa kuota API eksternal, dan tingkat akurasi untuk evaluasi kesehatan sumber data.

---

### 2. Spesialisasi Pipeline AGC

Sistem tidak boleh menggunakan satu pipeline generik. Dibangun 5 pipeline khusus di bawah satu orkestrator terpusat:

```
                          +-------------------------------+
                          | Orchestrator (BullMQ / Redis) |
                          +---------------+---------------+
                                          |
        +------------------+--------------+---------------+------------------+
        |                  |                              |                  |
+-------v-------+  +-------v-------+              +-------v-------+  +-------v-------+
| Football Data |  | News Intel    |              | Video Intel   |  | Player Intel  |
| Pipeline      |  | Pipeline      |              | Pipeline      |  | Pipeline      |
+---------------+  +---------------+              +---------------+  +---------------+
```

#### A. Football Data Pipeline
* **Cakupan:** Jadwal, hasil, skor langsung, statistik mendalam (xG, penguasaan bola), susunan pemain, klasemen.
* **Penyedia:** Sportmonks API v3 atau API-Football.
* **Karakteristik:** 100% deterministik, latensi rendah, tanpa sentuhan LLM.
* **Strategi Refresh:** Non-matchday: setiap 12 jam; Pre-match (H-1 jam): setiap 15 menit; Live Match: polling setiap 15–30 detik (atau Webhook); Post-match: pembersihan data pada +5m, +15m, dan +60m.

#### B. News Intelligence Pipeline
* **Cakupan:** Berita resmi klub, konferensi pers manajer, laporan cedera, rumor dan konfirmasi transfer.
* **Penyedia:** Feed RSS resmi Man Utd, media kredibel (BBC Sport, Sky Sports, The Athletic), jurnalis transfer Tier-1.
* **Karakteristik:** Berfokus pada ekstraksi, deduplikasi klaster, dan sintesis ringkasan bahasa Indonesia secara netral dengan sitasi sumber yang jelas.
* **Strategi Refresh:** Setiap 15 menit pada hari biasa; setiap 5 menit pada jendela pascapertandingan dan hari penutupan bursa transfer (*transfer deadline day*).

#### C. Video Intelligence Pipeline
* **Cakupan:** Sorotan resmi pertandingan (*highlights*), tayangan ulang resmi, konferensi pers, dan arsip momen emas.
* **Penyedia:** Saluran resmi YouTube Manchester United, Premier League, dan pemegang hak siar resmi.
* **Karakteristik:** Validasi kepatuhan penyematan (`is_embeddable == true`), pengecekan batasan wilayah (*region restriction*), dan mitigasi kuota YouTube API v3.
* **Strategi Refresh:** Pengecekan rutin setiap 2 jam; pencarian agresif setiap 10 menit mulai menit ke-15 setelah peluit akhir pertandingan dibunyikan hingga 6 jam pascapertandingan.

#### D. Player Intelligence Pipeline
* **Cakupan:** Profil skuat, nomor punggung, statistik individu musim berjalan, riwayat cedera, dan akun media sosial resmi terverifikasi.
* **Penyedia:** Sports Data API digabung dengan direktori resmi pemain klub.
* **Karakteristik:** Sinkronisasi berkala harian/mingguan. Validasi akun medsos resmi (mengecualikan akun fans/palsu).

#### E. Historical / Curated Pipeline
* **Cakupan:** Arsip laga terbaik (*Best Matches*), momen legendaris (Treble 1999, Moscow 2008), profil legenda klub.
* **Penyedia:** Pustaka metadata terkurasi yang terhubung ke arsip video resmi.
* **Karakteristik:** Operasi semi-statis, diperbarui saat jeda internasional (*international break*) atau jeda pramusim.

---

### 3. Skenario Otomatisasi Pascapertandingan (Post-Match Automation Walkthrough)

Alur kerja sistematis saat pertandingan Manchester United berakhir:

```
[Peluit Akhir: Status Menjadi 'FT']
                 │
                 ├──> (1) Football Data Pipeline menangkap status 'FINISHED' / 'FT'.
                 │        - Mengunci skor akhir dan statistik pertandingan.
                 │        - Memperbarui tabel klasemen sementara liga.
                 │        - Memicu event: match.concluded { matchId: "MU_LIV_2026" }.
                 │
                 ├──> (2) Pipeline Orchestrator mendistribusikan tugas paralel:
                 │        │
                 │        ├──> Task A: Pembuatan Ringkasan Taktis.
                 │        │    - Model AI menerima data pencetak gol, kartu, statistik xG.
                 │        │    - Menghasilkan ringkasan laga akurat tanpa halusinasi skor.
                 │        │    - Auto-publish ke Match Center & Latest Results.
                 │        │
                 │        ├──> Task B: Pelacakan Wawancara & Reaksi.
                 │        │    - News Engine mencari kutipan pascapertandingan manajer & pemain.
                 │        │    - Deduplikasi dan pengaitan dengan entitas pemain.
                 │        │
                 │        └──> Task C: Penemuan Video Sorotan (Highlights).
                 │             - Video Engine memeriksa playlist sorotan pemegang hak siar.
                 │             - Validasi embed dan tautan tayang resmi (Watch Again).
                 │
                 └──> (3) Publishing Engine memperbarui Status Beranda -> 'POST_MATCH'.
                          - Hero beranda menampilkan Kartu Skor Akhir & Video Highlights.
                          - Mengirim sinyal pembersihan cache Edge CDN untuk rute terkait.
```

* **Manajemen Idempotensi:** Setiap eksekusi pascapertandingan menggunakan kunci kunci idempoten: `lock:match_concluded:{match_id}`. Polling berulang tidak akan menghasilkan duplikasi ringkasan atau publikasi ganda.
* **Rencana Kontinjensi (Fallback):** Jika video sorotan resmi belum tersedia di YouTube dalam 30 menit pertama, seksi *Watch Again* secara otomatis menampilkan cuplikan audio wawancara klub atau video pratinjau pra-laga sebagai *fallback*, menghindari elemen antarmuka kosong (*blank state*).

---

## Deliverable D: Frontend & Fan Experience Architecture

### 1. Dynamic Homepage Composition Engine
Tata letak beranda The Champion tidak diatur secara manual oleh editor setiap hari. Tata letak dipandu oleh **Club Match State Machine** yang menyesuaikan urutan seksi secara dinamis berdasarkan kebutuhan suporter saat itu:

```
                         +-----------------------------------+
                         | Dynamic Section Composition Engine|
                         +-----------------+-----------------+
                                           |
     +--------------------+----------------+--------------------+--------------------+
     |                    |                                     |                    |
+----v-----+        +-----v----+                          +-----v----+         +-----v----+
| PRE-     |        | LIVE     |                          | POST-    |         | NO-MATCH |
| MATCH    |        | MATCH    |                          | MATCH    |         | MIDWEEK  |
| (H-48 Jam|        | (90 Mnt) |                          | (FT+24 Jam|        | (Jeda)   |
+----+-----+        +-----+----+                          +-----+----+         +-----+----+
     |                    |                                     |                    |
     | Hero: Hitung Mundur| Hero: Real-time Match Center        | Hero: Skor Akhir & | Hero: Berita Utama/
     | Jadwal & Lawan     | (Skor, Menit, Kartu, Link Siaran)   | Highlights Video   | Transfer Hub
     |                    |                                     |                    |
     | Seksi 2: Prediksi  | Seksi 2: Live Timeline Komentar &   | Seksi 2: Laporan   | Seksi 2: Jadwal
     | Line-up & Form     | Statistik Pertandingan              | Taktis & Wawancara | Terdekat & Klasemen
     |                    |                                     |                    |
     | Seksi 3: Klasemen  | Seksi 3: Dampak Klasemen Langsung   | Seksi 3: Rekaman   | Seksi 3: Players Hub
     | & Berita Terkini   |                                     | & Sorotan Gol      | & Best Matches
```

### 2. Spesifikasi Status Beranda Dinamis

1. **Pre-Match State (H-48 Jam hingga Kick-off):**
   * *Hero Area:* Hitung mundur (*countdown*), nama kompetisi, stadion, waktu sepak mula dalam zona WIB, serta tautan ke saluran siaran resmi.
   * *Konten Pendukung:* Susunan pemain resmi (terbuka otomatis 60 menit sebelum sepak mula), rekor pertemuan (*head-to-head*), dan kutipan persiapan tim.
2. **Live Match State (Kick-off hingga Selesai + 15 Menit):**
   * *Hero Area:* Papan skor langsung (*real-time scoreboard*), indikator status (Babak 1, HT, Babak 2), menit berjalan, dan pencetak gol langsung.
   * *Konten Pendukung:* Statistik waktu-nyata (penguasaan bola, tembakan ke gawang, pelanggaran), log kejadian (*timeline event*), dan tautan audio/siaran resmi.
3. **Post-Match State (Peluit Akhir hingga +24 Jam):**
   * *Hero Area:* Kartu Hasil Akhir, pencetak gol, statistik ringkas, dan video sorotan resmi tersemat (*Watch Again*).
   * *Konten Pendukung:* Laporan analisis pertandingan otomatis, kutipan konferensi pers manajer, dan dampak klasemen liga.
4. **No-Match / Midweek State (Periode di Luar Hari Pertandingan):**
   * *Hero Area:* Berita eksklusif klub atau pengumuman transfer resmi.
   * *Konten Pendukung:* Radar jadwal berikutnya, bursa transfer (dengan pemisahan tegas lencana *Confirmed* vs. *Rumour*), statistik pemain, klasemen, dan koleksi laga bersejarah (*Best Matches*).

### 3. Kontrak Data Komposisi Beranda (JSON API Contract)
Frontend mengonsumsi satu kontrak data terpadu dari `/api/v1/layout/homepage`:

```json
{
  "state": "PRE_MATCH",
  "activeMatchContext": {
    "matchId": "mu-vs-mci-20261129",
    "opponent": "Manchester City",
    "competition": "Premier League",
    "kickoffUtc": "2026-11-29T16:30:00Z",
    "kickoffWib": "23:30 WIB",
    "officialBroadcast": "Vidio"
  },
  "sections": [
    {
      "sectionId": "hero-match",
      "priority": 1,
      "component": "CountdownMatchHero",
      "refreshIntervalSeconds": 60,
      "payload": { "venue": "Old Trafford" }
    },
    {
      "sectionId": "team-news",
      "priority": 2,
      "component": "LineupComparison",
      "refreshIntervalSeconds": 300,
      "payload": { "confirmed": false }
    },
    {
      "sectionId": "standings-preview",
      "priority": 3,
      "component": "MiniStandings",
      "refreshIntervalSeconds": 3600,
      "payload": { "muPosition": 3 }
    }
  ],
  "meta": {
    "generatedAt": "2026-11-27T07:30:00Z",
    "cacheTtlSeconds": 120
  }
}
```

---

## Deliverable E: Database & Data Contracts

### 1. Conceptual Entity-Relationship Model
Data diorganisasi secara relasional dengan pemisahan yang ketat antara lapisan *Raw Ingested*, *Normalized Sports Data*, *Enriched Content*, dan *Published Presentation*:

```
[sources] 1──< [raw_items] 1──< [processed_content] 1──< [published_items]
   │                                   │
[source_health]                        v
                        [duplicate_clusters]

[competitions] 1──< [matches] 1──< [match_events]
                         │
     +───────────────────┼───────────────────+
     │                   │                   │
     v                   v                   v
[teams] 1──< [players] 1─< [player_stats]  [videos] 1──< [video_match_links]
```

### 2. Rancangan Skema Fisik Utama (PostgreSQL DDL)

```sql
-- 1. Lapisan Sumber & Manajemen Kesehatan
CREATE TABLE sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    source_type VARCHAR(50) NOT NULL, -- 'RSS', 'REST_API', 'YOUTUBE_CHANNEL'
    base_url TEXT NOT NULL,
    tier INT DEFAULT 2,              -- 1: Official/Tier 1, 2: Media Kredibel, 3: Tabloid
    is_active BOOLEAN DEFAULT TRUE,
    config JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE source_health (
    source_id UUID PRIMARY KEY REFERENCES sources(id) ON DELETE CASCADE,
    last_polled_at TIMESTAMPTZ,
    last_success_at TIMESTAMPTZ,
    consecutive_failures INT DEFAULT 0,
    is_quarantined BOOLEAN DEFAULT FALSE,
    last_error_message TEXT
);

-- 2. Lapisan Raw Data (Audit & Provenance)
CREATE TABLE raw_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_id UUID NOT NULL REFERENCES sources(id),
    external_id TEXT NOT NULL,
    content_hash CHAR(64) NOT NULL, -- SHA-256
    payload JSONB NOT NULL,
    fetched_at TIMESTAMPTZ DEFAULT NOW(),
    processed_status VARCHAR(30) DEFAULT 'PENDING',
    CONSTRAINT uq_source_external UNIQUE (source_id, external_id)
);
CREATE INDEX idx_raw_items_hash ON raw_items(content_hash);

-- 3. Lapisan Data Olahraga Deterministik
CREATE TABLE matches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_id VARCHAR(50) UNIQUE NOT NULL,
    competition_name VARCHAR(100) NOT NULL,
    home_team_name VARCHAR(100) NOT NULL,
    away_team_name VARCHAR(100) NOT NULL,
    kickoff_time TIMESTAMPTZ NOT NULL,
    status VARCHAR(30) NOT NULL, -- 'SCHEDULED', 'IN_PLAY', 'FINISHED', 'POSTPONED'
    period VARCHAR(20),          -- '1H', 'HT', '2H', 'FT'
    minute INT,
    home_score INT DEFAULT 0,
    away_score INT DEFAULT 0,
    statistics JSONB DEFAULT '{}'::jsonb,
    lineups JSONB DEFAULT '{}'::jsonb,
    official_broadcast_url TEXT,
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_matches_kickoff ON matches(kickoff_time);

CREATE TABLE match_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    match_id UUID REFERENCES matches(id) ON DELETE CASCADE,
    minute INT NOT NULL,
    extra_minute INT,
    event_type VARCHAR(50) NOT NULL, -- 'GOAL', 'CARD', 'SUBSTITUTION', 'VAR'
    player_name VARCHAR(100),
    team_identifier VARCHAR(50) NOT NULL,
    detail JSONB DEFAULT '{}'::jsonb
);

-- 4. Lapisan Konten Olahan & AI
CREATE TABLE processed_content (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    raw_item_id UUID REFERENCES raw_items(id),
    content_type VARCHAR(50) NOT NULL, -- 'MATCH_REPORT', 'NEWS', 'TRANSFER', 'INTERVIEW'
    title TEXT NOT NULL,
    slug TEXT UNIQUE NOT NULL,
    summary TEXT NOT NULL,
    body_markdown TEXT NOT NULL,
    transfer_status VARCHAR(30),       -- 'CONFIRMED', 'RUMOUR', NULL
    confidence_score NUMERIC(3,2) NOT NULL,
    provenance JSONB NOT NULL,         -- Sumber URL asli, jurnalis, penerbit
    cluster_id UUID,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 5. Lapisan Publikasi & Antarmuka
CREATE TABLE published_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content_id UUID REFERENCES processed_content(id) ON DELETE CASCADE,
    section_code VARCHAR(50) NOT NULL, -- 'HERO', 'MATCH_CENTER', 'TRANSFER_HUB', 'WATCH_AGAIN'
    priority INT DEFAULT 0,
    status VARCHAR(30) DEFAULT 'PUBLISHED',
    published_at TIMESTAMPTZ DEFAULT NOW(),
    cache_tag VARCHAR(100) NOT NULL
);
CREATE INDEX idx_published_routing ON published_items(section_code, priority DESC, published_at DESC);

-- 6. Lapisan Video Intelligence
CREATE TABLE videos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    youtube_video_id VARCHAR(50) UNIQUE NOT NULL,
    channel_title VARCHAR(100) NOT NULL,
    title TEXT NOT NULL,
    thumbnail_url TEXT,
    is_embeddable BOOLEAN DEFAULT TRUE,
    related_match_id UUID REFERENCES matches(id),
    published_at TIMESTAMPTZ NOT NULL
);
```

### 3. Siklus Hidup dan Retensi Data
* `raw_items`: Dipindahkan ke penyimpanan dingin (Object Storage S3/Cloudflare R2) setelah 30 hari. Metadata hash dipertahankan selamanya untuk audit hukum.
* `system_logs` & `job_runs`: Dirotasi dan dibersihkan otomatis setiap 14 hari dengan partisi tabel bulanan.
* `matches`, `players`, `published_items`: Disimpan permanen sebagai aset arsip dan fondasi SEO suporter.

---

## Deliverable F: Technology Stack Recommendation

| Lapisan Sistem | Rekomendasi Terpilih | Opsi yang Ditolak | Justifikasi & Analisis Trade-off |
| :--- | :--- | :--- | :--- |
| **Frontend Framework** | **Next.js 15 (App Router, React Server Components)** | Astro, Remix, SPA Murni (Vite) | Memiliki perpaduan terbaik antara *Incremental Static Regeneration* (ISR) untuk konten statis dan *Server-Side Streaming* untuk hari pertandingan. Sangat optimal untuk Core Web Vitals dan SEO suporter. |
| **Backend & API Engine** | **Node.js (TypeScript) dengan Fastify / NestJS** | Python (FastAPI), Go (Golang) | Berbagi kontrak tipe (*type-sharing*) langsung antara frontend dan backend. Fastify memiliki throughput I/O sangat tinggi, jauh lebih cepat daripada Django/Express, dan waktu rilis lebih lincah dibanding Go. |
| **Database Utama** | **PostgreSQL 16** | MongoDB, MySQL | Integritas data relasional sangat vital untuk memetakan pertandingan, kartu, susunan pemain, dan log publikasi. Tipe kolom `JSONB` menyediakan fleksibilitas skema mentah tanpa mengorbankan integritas relasional. |
| **Queue & Orchestration** | **BullMQ didukung Redis 7** | RabbitMQ, Celery, AWS SQS | BullMQ mendukung alur kerja kompleks (DAGs), penundaan jadwal (*job delay*), isolasi kuota antar-antrean, serta penanganan *dead-letter* secara bawaan di lingkungan Node.js/TypeScript. |
| **AI Orchestration** | **Claude 3.5 Haiku via LiteLLM Gateway** | OpenAI GPT-4o, Custom Fine-tuned LLM | Claude 3.5 Haiku memberikan kepatuhan skema JSON tertinggi dengan latensi ekstraksi <1,5 detik dan biaya per token sangat ekonomis. LiteLLM mencegah ketergantungan pada satu vendor (*vendor lock-in*). |
| **Penyedia Data Sepak Bola** | **Sportmonks API v3 (atau API-Football)** | Football-Data.org, Opta/StatsBomb | Sportmonks menyediakan cakupan lengkap Premier League, piala domestik, kompetisi Eropa, data statistik xG, serta lisensi komersial yang terjangkau bagi produk tahap awal. Opta/StatsBomb terlalu mahal untuk fase peluncuran awal. |
| **Edge CDN & Caching** | **Cloudflare Pro/Enterprise** | Fastly, AWS CloudFront | Integrasi pembersihan cache berbasis tag (*Cache-Tag Purging*), perlindungan bot/DDoS bawaan, dan hosting aset media tanpa biaya egress melalui Cloudflare R2. |
| **Observability** | **OpenTelemetry + Grafana Cloud / Sentry** | Datadog, New Relic | Memberikan pelacakan galat aplikasi dan metrik antrean antarsistem secara menyeluruh tanpa risiko lonjakan biaya (*cost runaway*) dari vendor enterprise komersial. |

---

## Deliverable G: Development Roadmap & MVP Definition

Proses implementasi dibagi menjadi 6 fase berurutan yang saling bergantung (*dependency-aware*):

```
+-----------------------------------------------------------------------------------------+
| Phase 0: Pengesahan Arsitektur, Kontrak Data & Kontrak API (Minggu 1 - 2)               |
+-----------------------------------------------------------------------------------------+
                                             │
+--------------------------------------------v--------------------------------------------+
| Phase 1: Fondasi Platform, Database & Antrean Pekerjaan (Minggu 3 - 5)                  |
+-----------------------------------------------------------------------------------------+
                                             │
+--------------------------------------------v--------------------------------------------+
| Phase 2: Implementasi Football Data Engine & State Machine (Minggu 6 - 8)               |
+-----------------------------------------------------------------------------------------+
                                             │
+--------------------------------------------v--------------------------------------------+
| Phase 3: AGC News & Video Intelligence Pipelines (Minggu 9 - 11)                        |
+-----------------------------------------------------------------------------------------+
                                             │
+--------------------------------------------v--------------------------------------------+
| Phase 4: Dynamic Frontend Fan Portal & Match Center (Minggu 12 - 14)                    |
+-----------------------------------------------------------------------------------------+
                                             │
+--------------------------------------------v--------------------------------------------+
| Phase 5: Uji Beban, Mitigasi Risiko & Peluncuran Produksi (Minggu 15 - 16)              |
+-----------------------------------------------------------------------------------------+
```

### Definisi Minimum Viable Product (MVP) Terkecil yang Kredibel:
Bukan mencoba membangun seluruh 9 seksi sekaligus, ruang lingkup MVP yang dapat membuktikan model operasi AGC adalah:
1. **Football Data Engine Aktif:** Otomatisasi data pertandingan Manchester United (Jadwal, Skor Langsung, Hasil, Klasemen Liga).
2. **Video Intelligence Ringan:** Penemuan otomatis video sorotan resmi YouTube dari saluran resmi Manchester United dan Premier League pascapertandingan.
3. **Dynamic Homepage Engine:** Menghadirkan beranda yang dapat beralih otomatis antara status *Pre-match*, *Live*, *Post-match*, dan *No-match*.
4. **AGC Control Center dengan Review Desk:** Panel admin untuk melihat log antrean, status kesehatan sumber, dan moderasi manual berita/rumor transfer.

---

## Deliverable H: Risk Assessment & Operational Boundaries

```
[Matriks Batasan Operasional & Manajemen Risiko]
Tinggi   │ [Siaran Ilegal]                 [Downtime Provider Olahraga]
         │ (Tindakan: Larangan Keras,      (Tindakan: Fallback Cache
         │  Hanya Link Resmi)               Stale-While-Revalidate)
         │
         │ [Pelanggaran Hak Cipta Teks]    [Halusinasi Fakta oleh AI]
         │ (Tindakan: Sintesis Bahasa      (Tindakan: Isolasi AI dari Skor,
         │  Indonesia & Sitasi Sumber)      Batas Threshold Ketat)
         │
Rendah   │ [Antrean Pekerjaan Tertunda]    [Lonjakan Biaya Token AI]
         │ (Tindakan: Isolasi Worker        (Tindakan: Model Ringan Haiku,
         │  dan Auto-scaling)               Budget Capping di Gateway)
         └─────────────────────────────────────────────────────────────
           Dampak Rendah                   Dampak Tinggi
```

| Area Risiko | Ancaman Spesifik | Mitigasi Arsitektural & Batasan Operasional |
| :--- | :--- | :--- |
| **Hak Siar Pertandingan** | Tuntutan hukum dan pemblokiran domain akibat menyediakan tautan streaming ilegal. | **Larangan Mutlak:** Platform tidak pernah menyediakan stream bajakan. Seksi siaran hanya memuat tautan pengalihan (*redirection*) resmi ke mitra pemegang lisensi resmi (misal: Vidio ID). |
| **Hak Cipta Konten Berita** | Somasi DMCA dari kantor berita atas penggandaan artikel berita secara utuh (*scraping*). | **Sintesis Transformatif:** Teks mentah tidak pernah diterbitkan langsung. Sistem menyusun ringkasan editorial independen 100–150 kata dalam bahasa Indonesia dengan mencantumkan nama penerbit dan tautan sumber asli. |
| **Halusinasi AI** | AI mengarang skor, pencetak gol, atau mengonfirmasi rumor transfer palsu. | **Pemisahan Sumber Kebenaran:** Skor dan statistik hanya berasal dari tabel SQL olahraga. AI tidak memiliki izin menulis ke skema pertandingan. Rumor transfer wajib menyandang label visual "Rumour" dan dibatasi verifikasinya. |
| **Kuota YouTube API** | Kehabisan kuota 10.000 unit harian di tengah pertandingan penting. | **Arsitektur Kuota Cerdas:** Melarang kueri bebas `search.list` (100 unit). Menggunakan penelusuran playlist resmi `playlistItems.list` (1 unit per panggilan) dengan jadwal pemantauan yang diperketat hanya di jam pascapertandingan. |
| **Kegagalan Provider Data** | API Sepak Bola mengalami *down* atau gangguan saat laga berlangsung. | **Fallback Stale-While-Revalidate:** Frontend secara otomatis menyajikan data terakhir yang tersimpan di cache disertai indikator *"Memperbarui data..."*, mencegah antarmuka rusak (*crash*). |

---

## Jawaban atas Pertanyaan Kunci Audit

1. **Apa arsitektur terbaik untuk The Champion sebagai produk AGC independen?**  
   Arsitektur *Modular Monolith* atau klaster layanan berbasis TypeScript (Node.js/Fastify + Next.js 15), dengan PostgreSQL 16 sebagai basis data relasional, BullMQ/Redis sebagai orkestrator antrean, dan integrasi Cloudflare CDN untuk pengiriman aset suporter di tingkat *edge*.
2. **Apakah AGC sebaiknya menggunakan beberapa pipeline khusus di bawah satu orkestrator?**  
   **Ya, wajib.** Menyatukan alur data skor langsung yang butuh kecepatan tinggi dengan ekstraksi berita berbasis AI akan menyebabkan *bottleneck*. Dibutuhkan 5 pipeline terisolasi (Football, News, Video, Player, Historical) di bawah satu orkestrator terpusat (BullMQ).
3. **Berapa batas MVP terkecil yang membuktikan model operasi AGC?**  
   MVP yang mencakup: Sinkronisasi data pertandingan Manchester United dari Sportmonks/API-Football, penyematan otomatis video sorotan resmi pascapertandingan via YouTube API, antarmuka beranda adaptif dengan 4 status siklus laga, serta panel kendali admin (*Control Center*) untuk konfigurasi aturan dan moderasi berita.
4. **Bagaimana backend memungkinkan admin mengontrol otomatisasi tanpa mengelola setiap konten secara manual?**  
   Melalui **Declarative Policy Rules**: admin mengatur bobot otoritas sumber (*Source Tiering*), menentukan batas ambang keyakinan ($S \ge 0.88$ untuk terbit otomatis), serta memanfaatkan *Review Desk* yang hanya memunculkan konten anomali, sengketa fakta, atau rumor transfer.
5. **Bagaimana komposisi frontend merespons konteks pertandingan dan intensi fans?**  
   Melalui **Dynamic Homepage Composition Engine** yang mendeteksi mesin status laga (`PRE_MATCH`, `LIVE_MATCH`, `POST_MATCH`, `NO_MATCH`) dari backend dan mengubah urutan komponen visual agar informasi yang paling dicari fans selalu berada di posisi teratas (*hero position*).
6. **Provider dan API mana yang layak digunakan, serta berapa batas dan biayanya?**  
   **Sportmonks API v3** atau **API-Football** (~$30–$90/bulan) untuk data olahraga lengkap berlisensi; **YouTube Data API v3** (Gratis dalam batas kuota 10.000 unit/hari dengan pola pemanggilan cerdas); serta **Claude 3.5 Haiku / OpenAI GPT-4o-mini** via LiteLLM (~$0.25 per 1 juta token) untuk transformasi teks terstruktur yang sangat hemat.
7. **Apa yang wajib ditinjau manusia (*human-review*) sebelum dipublikasikan?**  
   Rumor transfer dari sumber Tier-2 atau Tier-3, konten yang terdeteksi memiliki kontradiksi entitas dengan database statistik, konten dengan skor keyakinan rendah ($S < 0.88$), dan berita krisis (isu hukum pemain atau pergantian manajer klub).
8. **Apa keputusan arsitektur berisiko tertinggi yang harus diselesaikan sebelum pengembangan dimulai?**  
   (1) Mengamankan kontrak lisensi komersial penyedia data sepak bola dengan jaminan latensi data langsung; (2) Menetapkan batasan hukum sitasi sintesis berita guna mencegah pelanggaran hak cipta; dan (3) Mengimplementasikan mekanisme penguncian idempoten (*Redis Distributed Lock*) untuk mencegah duplikasi konten saat lonjakan lalu lintas pascapertandingan.

---
**THE CHAMPION — IT'S ALL ABOUT MU.**