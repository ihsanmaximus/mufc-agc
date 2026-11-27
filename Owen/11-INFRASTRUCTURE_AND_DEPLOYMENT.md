# 11 — INFRASTRUCTURE AND DEPLOYMENT

**Project:** The Champion — It's All About MU  
**Subsystem:** Production Infrastructure Topology, Containerization & CI/CD  
**Document Ref:** TC-DOC-11-INFRA-DEPLOY  
**Authoritative Hierarchy:** Level 2 Specification  

---

## 1. Concrete Production Infrastructure Topology

The production stack is architected for high reliability, minimal maintenance, and sub-100ms response times:

```
[Internet Traffic]
       │
       ▼
[Cloudflare Edge Network] (DNS, DDoS, SSL, WAF, Edge Caching)
       │
       ├──> Frontend Requests ──> [Vercel / Cloudflare Pages] (Next.js 15 SSR & ISR)
       │
       └──> API Requests      ──> [DigitalOcean / AWS Application Load Balancer]
                                        │
             ┌──────────────────────────┴──────────────────────────┐
             ▼                                                     ▼
     [Backend API Nodes]                                   [BullMQ Worker Nodes]
  (Node.js Fastify - Docker)                            (Node.js Pipeline - Docker)
             │                                                     │
             └──────────────────────────┬──────────────────────────┘
                                        │
                       ┌────────────────┴────────────────┐
                       ▼                                 ▼
             [Managed PostgreSQL 16]             [Managed Redis 7]
             (Primary + Standby Replica)         (BullMQ & Memory Cache)
```

---

## 2. Containerized Service Definitions

### Service A: API Gateway & Dynamic Composer
- **Runtime:** Node.js 22 LTS Alpine Container.
- **Port:** 4000.
- **Resources:** 1 vCPU, 1 GB RAM (Autoscales to 3 instances on CPU > 70%).

### Service B: BullMQ Pipeline Worker Harness
- **Runtime:** Node.js 22 LTS Alpine Container.
- **Concurrency:** 10 parallel jobs.
- **Resources:** 2 vCPU, 2 GB RAM.

---

## 3. Master Environment Variable Contract

```env
# Node Environment
NODE_ENV=production
PORT=4000
API_BASE_URL=https://api.thechampion.club

# Database Connectivity
DATABASE_URL=postgresql://thechampion_user:${DB_PASSWORD}@postgres-cluster.internal:5432/thechampion_db?sslmode=require&pool_timeout=10&max_connections=25
DATABASE_DIRECT_URL=postgresql://thechampion_user:${DB_PASSWORD}@postgres-cluster.internal:5432/thechampion_db?sslmode=require

# Redis Queue & Caching
REDIS_URL=rediss://default:${REDIS_PASSWORD}@redis-cluster.internal:6379

# AI & LLM Gateways
GEMINI_API_KEY=AIzaSyD-mocked-key-for-production-gemini
AI_MODEL_NAME=gemini-1.5-flash
AI_TEMPERATURE=0.2

# Sports Data Provider
FOOTBALL_DATA_PROVIDER=SPORTMONKS
SPORTMONKS_API_KEY=mocked_sportmonks_token_v3
SPORTMONKS_MU_TEAM_ID=14

# YouTube Data API
YOUTUBE_API_KEY=mocked_youtube_api_v3_key

# Security & Tokens
JWT_SECRET=super_secure_random_hex_string_min_64_chars
INTERNAL_OPS_API_KEY=mocked_internal_ops_token
```

---

## 4. Development vs. Production Demarcation

| Capability / Resource | Gemini Argon Dev Environment | Production Infrastructure |
| :--- | :--- | :--- |
| **Execution Environment** | Stateless Agentic Sandbox | Containerized Always-On Microservices (Docker) |
| **Relational Database** | In-Memory / Ephemeral DDL Simulation | AWS RDS / DigitalOcean Managed PostgreSQL 16 |
| **Background Scheduler** | Turn-based execution | Persistent BullMQ Queue Workers on Redis 7 |
| **Network Egress** | Controlled Tool Gateway | Direct HTTPS with Connection Pooling & DNS Caching |