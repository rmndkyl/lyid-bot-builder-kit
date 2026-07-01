---
name: script-architect
description: "Generate crypto automation bot scripts from endpoint docs."
version: 0.3.0-lite
author: LYID
tags: [Crypto, Airdrop, Automation, JavaScript, NodeJS]
license: commercial
---

# Script Architect (LYID Bot Builder Kit)

Generate script automation (Node.js/JavaScript) dari dokumentasi endpoint 
yang dihasilkan oleh skill `endpoint-hunter`. Mengikuti pattern dan style 
kode yang sudah ada di GitHub org `layerairdrop`.

## Self-Learning Framework (SOUL + MEMORY + TASKS)

**SOUL** = this SKILL.md (who you are, how to behave).
**MEMORY** = `MEMORY.md` — build patterns, auth integrations, failed approaches, pricing.
**TASKS** = `TASKS.md` — active builds, fleet health, completion status.

**BEFORE every build:**
1. Read `MEMORY.md` — check known patterns for this campaign type + auth method
2. Read `TASKS.md` — check if project already built or queued
3. After build: update `MEMORY.md` with new learnings (bugs fixed, patterns discovered)
4. After build: update `TASKS.md` status

**AFTER every build:**
- New campaign type pattern? -> append to MEMORY.md "Learned Campaign Type Patterns"
- New auth integration? -> append to "Learned Auth Integration"
- Bug fixed? -> append to "Learned Build Patterns" or "Failed Approaches"
- Pricing changed? -> update "Learned Pricing & Licensing"

## When to Use

- User minta buat bot/script automation untuk project crypto
- Setelah skill `endpoint-hunter` selesai trace API flow
- User bilang "buat script", "generate bot", "automation untuk project X"
- Konversi endpoint documentation menjadi working code

## Prerequisites

- Dokumentasi endpoint dari `endpoint-hunter` (atau user provide manual)
- Node.js installed (node --version untuk verify)
- Pattern reference dari repo layerairdrop yang sudah ada

## WAJIB SEBELUM BUILD: Identify Campaign Type

Baca `references/campaign-types.md` (taxonomy 15 types + decision matrix).

Ringkasan:
- A=Testnet, B=On-Chain Volume, C=Quest Platform, D=Social Tasks
- E=Quiz, F=Faucet, G=Referral, H=Telegram Mini-App, I=Node
- J=Snapshot, K=XP Social, L=Agent Marketplace, M=DEX Aggregator
- N=Discord Role-Gated, O=Perpetual DEX (GMX-fork)

ALWAYS test auth flow BEFORE building full bot. Open website, click Sign In,
verify what auth methods are available. If only OAuth (Google/GitHub) -- bot
CANNOT automate login via API alone. Plan for token injection or browser automation.

## Architecture Pattern (Layer Airdrop Style)

```
project-name-bot/
  src/
    api.js          # API calls (endpoint wrappers)
    auth.js         # Authentication logic
    tasks.js        # Task execution logic
  utils/
    utils.js        # Helper functions (random, delay, readFile)
    logger.js       # Colored logger (chalk-based)
    proxy.js        # Proxy manager (HTTP/SOCKS5, Monosans, rotation)
    banner.js       # LYID watermark banner (figlet + gradient-string)
    axios-retry-setup.js  # Auto-retry 3x exponential backoff
    wallet-queue.js       # p-queue wrapper for multi-wallet
  config.json       # Configuration (delay, retries, mode, base_url)
  privateKey.txt    # Wallet private keys (user fills, .gitignored)
  proxy.txt         # Proxy list (optional, .gitignored)
  index.js          # Entry point with CLI menu + proxy selection
  package.json
  .gitignore
```

## Build Steps

### Step 1: Project Setup
1. Create project directory
2. Initialize package.json
3. Install dependencies: axios, chalk, figlet, gradient-string, dotenv, ethers
4. Copy shared utils from ~/LYID-BOTS/shared/

### Step 2: Auth Module (src/auth.js)
1. Implement auth based on endpoint-hunter findings
2. Support wallet signature, OAuth token injection, JWT
3. Handle token refresh

### Step 3: API Module (src/api.js)
1. Wrap each endpoint as a function
2. Use axios with retry setup
3. Handle rate limits (429) and errors

### Step 4: Task Module (src/tasks.js)
1. Implement each user action as a function
2. Handle dependencies between tasks
3. Add random delays between actions

### Step 5: Entry Point (index.js)
1. CLI menu (daily, claim, swap, etc.)
2. Proxy selection
3. Wallet loop with p-queue
4. Banner display

### Step 6: Testing
1. Test with single wallet first
2. Verify all tasks complete
3. Check error handling
4. Test with --daily flag

### Step 7: Git Push
1. .gitignore (node_modules/, privateKey.txt, proxy.txt)
2. git init, add, commit
3. gh repo create layerairdrop/[project]-bot --private --source=. --push

## Common Patterns

### Daily Task Pattern
```javascript
async function daily(wallet) {
  const token = await auth(wallet);
  await checkIn(token);
  await claimRewards(token);
  await performTasks(token);
}
```

### Multi-Wallet Pattern
```javascript
const PQueue = (await import('p-queue')).default;
const queue = new PQueue({ concurrency: 1 });
for (const wallet of wallets) {
  queue.add(() => daily(wallet));
}
```

## References (Included)

- references/campaign-types.md — 15 campaign type taxonomy
- references/pitfalls.md — Common mistakes to avoid
- references/campaign-type-identification.md — How to identify campaign type

## References (Full Version Only)

43 references including:
- Specific project examples (convallax, simplechain, hertzflow, unicity, orbition, etc.)
- OAuth templates (Google PKCE, GitHub)
- Proxy patterns, logger patterns, banner patterns
- GMX-fork patterns, Supabase patterns, Solana patterns
- License check integration, Telegram bot campaign
- Pre-seeded MEMORY.md with 25+ real-world learnings
- 7 code templates (oauth-auth.js, oauth-run-wallet.js, google-oauth-auth.js, etc.)

Upgrade to Full Version for complete reference library + code templates.
