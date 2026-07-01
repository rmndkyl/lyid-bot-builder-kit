---
name: endpoint-hunter
description: "Trace API endpoints for crypto project automation scripts."
version: 0.3.0-lite
author: LYID
tags: [Crypto, Airdrop, API, Automation, Reverse-Engineering]
license: commercial
---

# Endpoint Hunter (LYID Bot Builder Kit)

Reverse-engineer API/endpoint flow dari project crypto (web app, mini-app, 
testnet) untuk dokumentasi automation. Output: struktur endpoint yang siap 
dipakai oleh skill `script-architect`.

## Self-Learning Framework (SOUL + MEMORY + TASKS)

**SOUL** = this SKILL.md (who you are, how to behave).
**MEMORY** = `MEMORY.md` — learned patterns, failed approaches, cross-session knowledge.
**TASKS** = `TASKS.md` — active traces, queue, completion status.

**BEFORE every trace:**
1. Read `MEMORY.md` — check if this project type has known patterns/pitfalls
2. Read `TASKS.md` — check if project already traced or queued
3. After trace: update `MEMORY.md` with new learnings (patterns, failures, costs)
4. After trace: update `TASKS.md` status

**AFTER every trace:**
- New auth pattern discovered? → append to MEMORY.md "Learned Auth Patterns"
- New API quirk found? → append to "Learned API Patterns"
- Approach failed? → append to "Failed Approaches"
- Gas/cost data collected? → append to "Gas & Cost Learnings"

## When to Use

- User menyebut project crypto baru yang perlu di-automate
- User mau cari endpoint flow untuk daily check-in, claim, swap, dll.
- User bilang "trace API", "cari endpoint", "reverse engineer" untuk project crypto
- Sebelum membuat script automation baru

## Prerequisites

- Browser toolset aktif (browser_navigate, browser_click, browser_snapshot)
- web_extract untuk baca dokumentasi API publik
- Akses ke DevTools (user membantu inspect jika perlu)

## Procedure

### Fase 0: Project Research (dari URL)

1. Extract basic info dari URLs (name, chain, features)
2. Research funding and investors (web_search)
3. Analyze social metrics (Twitter followers, Discord members)
4. Determine auth method preview (wallet? OAuth? custom?)
5. On-chain verification (contracts, faucet activity, unique wallets)
6. Sybil resistance assessment (identity requirements, multi-wallet feasibility)
7. Historical airdrop ROI comparison with similar projects
8. Gas cost estimation per operation
9. Snapshot/TGE signal detection
10. Generate project profile

### Fase 1: Reconnaissance

1. Open target website in browser
2. Identify tech stack (React/Next.js, Vue, etc.)
3. Check Network tab for API calls
4. Note authentication method (wallet connect, OAuth, custom)
5. Map user flows (sign in -> dashboard -> actions)

### Fase 2: API Discovery

1. Intercept all API calls during user flow
2. Document endpoints (method, URL, headers, body)
3. Identify patterns (REST, GraphQL, RPC)
4. Map data flow (what goes in, what comes out)
5. Check for WebSocket connections

### Fase 3: Authentication Analysis

1. Determine auth type (see references/auth-patterns.md)
2. Extract tokens (JWT, session, API key)
3. Document token lifecycle (creation, refresh, expiry)
4. Test token reuse across sessions

### Fase 4: Task Mapping

1. List all user actions (check-in, claim, swap, etc.)
2. Map each action to API endpoints
3. Document required parameters
4. Identify dependencies (action B needs result from action A)

### Fase 5: Validation

1. Test endpoints with curl/Postman
2. Verify reproducibility (same request = same result)
3. Document edge cases (rate limits, errors, timeouts)
4. Create endpoint reference file

### Fase 6: Documentation

1. Write endpoint reference
2. Include examples (curl commands)
3. Document auth flow (step by step)
4. Note pitfalls and gotchas

## Output Format

Create file: API_Endpoint_Reference/[Project]_Endpoint.txt

## Scoring Matrix

| Factor | Weight | Criteria |
|--------|--------|----------|
| Automation Potential | 0-20 | Daily tasks? Repetitive? Multi-wallet? |
| Market Demand | 0-20 | Hype? Funding? Community size? |
| Technical Complexity | 0-20 | Auth type? Captcha? Custom protocol? |
| Revenue Potential | 0-20 | Token value? Distribution model? |
| Risk Level | 0-20 | Scam indicators? Regulatory risk? |

Total: /100 -> Tier (FREE/PREMIUM/HIGH PRIORITY)

## References (Included)

- references/auth-patterns.md — Common auth patterns
- references/pitfalls.md — Common mistakes to avoid
- references/deep-analysis-techniques.md — Advanced research methods

## References (Full Version Only)

The full version includes 30+ case studies with specific project data:
- Convallax, SimpleChain, HertzFlow, Unicity, Orbition, Kiedex, Canopy, UpOnly, Watt2Trade, FLPP, Afyniti
- SIWE patterns, RSC detection, Google OAuth PKCE, Supabase patterns
- Solana Anchor extraction, Turnstile sitekey extraction
- Pre-seeded MEMORY.md with 35+ real-world learnings

Upgrade to Full Version for complete reference library.
