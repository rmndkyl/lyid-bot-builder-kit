---
name: project-lifecycle-tracker
description: "Track crypto project lifecycle for automation profitability."
version: 0.2.0-lite
author: LYID
tags: [Crypto, Airdrop, Lifecycle, Automation, Pipeline]
license: commercial
---

# Project Lifecycle Tracker (LYID Bot Builder Kit)

Evaluasi project crypto untuk menentukan apakah worth dijadikan script 
automation premium. Menggunakan data dari knowledge graph dan airdrop 
tracker untuk scoring profitability.

## Self-Learning Framework (SOUL + MEMORY + TASKS)

**SOUL** = this SKILL.md (who you are, how to behave).
**MEMORY** = `MEMORY.md` — scoring calibration, market intel, ROI benchmarks.
**TASKS** = `TASKS.md` — evaluation queue, weekly review schedule.

**BEFORE every evaluation:**
1. Read `MEMORY.md` — check scoring calibration for similar projects
2. Read `TASKS.md` — check if project already evaluated or queued
3. After evaluation: update `MEMORY.md` with scoring rationale
4. After evaluation: update `TASKS.md` with score + decision

**AFTER every evaluation:**
- New investor signal pattern? -> append to MEMORY.md
- Scoring factor adjusted? -> append to "Scoring Calibration Data"
- TGE signal detected? -> append to "TGE Timeline Patterns"
- ROI data point added? -> append to "Historical ROI Benchmarks"

## When to Use

- User menyebut project crypto baru dan mau tahu apakah worth di-automate
- User minta evaluasi "project ini layak dibuat script?"
- Sebelum menjalankan `endpoint-hunter` untuk project baru
- Saat menentukan pricing strategy untuk script premium
- Weekly review project mana yang masih aktif vs sudah expired

## Prerequisites

- Knowledge graph data
- Airdrop tracker data
- web_search untuk cek status project terkini

## Scoring Matrix

Setiap project dinilai dari 5 faktor (0-20 points each, max 100):

### 1. Automation Potential (0-20)

| Score | Kriteria |
|-------|----------|
| 20 | Daily task yang repetitive (check-in, claim, swap) |
| 15 | Weekly task dengan multiple steps |
| 10 | One-time task dengan multiple wallets |
| 5 | Task sederhana, jarang diulang |
| 0 | Tidak bisa di-automate (perlu CAPTCHA manual, KYC) |

### 2. Market Demand (0-20)

| Score | Kriteria |
|-------|----------|
| 20 | Project hyped, 10K+ participants di Discord/Telegram |
| 15 | Funding > $10M, backing dari top VCs (a16z, Paradigm) |
| 10 | Funding $1-10M, backing menengah |
| 5 | Project kecil, komunitas < 1K |
| 0 | Project mati/scam indicator |

### 3. Technical Complexity (0-20)

| Score | Kriteria |
|-------|----------|
| 20 | Standard REST API, wallet auth, no captcha |
| 15 | Custom auth, minor quirks, no captcha |
| 10 | OAuth required, some manual steps |
| 5 | Complex auth, captcha, custom protocol |
| 0 | Full browser automation required, heavy captcha |

### 4. Revenue Potential (0-20)

| Score | Kriteria |
|-------|----------|
| 20 | Token likely > $1000/wallet, high distribution |
| 15 | Token likely $500-1000/wallet |
| 10 | Token likely $100-500/wallet |
| 5 | Token likely < $100/wallet |
| 0 | No token planned, or very low value |

### 5. Risk Level (0-20)

| Score | Kriteria |
|-------|----------|
| 20 | No red flags, established team, audited contracts |
| 15 | Minor concerns, but overall positive |
| 10 | Some red flags, proceed with caution |
| 5 | Significant concerns, high risk |
| 0 | Scam indicators, avoid |

## Tier Classification

| Total Score | Tier | Action |
|-------------|------|--------|
| 81-100 | HIGH PRIORITY | Build immediately, premium price ($7+) |
| 61-80 | PREMIUM | Build when available, standard price ($4-7) |
| 41-60 | FREE | Build as free script (basic features) |
| 0-40 | SKIP | Not worth building |

## Enhanced Factors

| Factor | Impact |
|--------|--------|
| Sybil resistance NONE/LOW | +3 (multi-wallet mudah) |
| Sybil resistance MEDIUM | +0 (standard) |
| Sybil resistance HIGH/KYC | -5 (multi-wallet sulit) |
| Historical ROI > $500/wallet | +2 |
| Historical ROI < $100/wallet | -2 |
| Gas cost < $1/day/wallet | +1 |
| Gas cost > $5/day/wallet | -3 |

## Output Format

```
PROJECT: [name]
SCORE: [total]/100
TIER: [HIGH PRIORITY/PREMIUM/FREE/SKIP]

BREAKDOWN:
- Automation Potential: [score]/20
- Market Demand: [score]/20
- Technical Complexity: [score]/20
- Revenue Potential: [score]/20
- Risk Level: [score]/20

ENHANCED FACTORS:
- Sybil resistance: [level] [modifier]
- Historical ROI: [estimate] [modifier]
- Gas cost: [estimate] [modifier]

RECOMMENDATION: [build/skip/wait]
PRICING: [free/$X]
NOTES: [any important observations]
```

## References (Included)

- None (self-contained methodology)

## References (Full Version Only)

- Pre-seeded MEMORY.md with 30+ real-world scoring data points
- Market intelligence from DeFiLlama, CoinGecko, airdrops.io
- ROI benchmarks from 11 live-tested projects
- TGE timeline patterns from real airdrops

Upgrade to Full Version for pre-seeded knowledge base.
