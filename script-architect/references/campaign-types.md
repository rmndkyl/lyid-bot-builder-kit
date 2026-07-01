# Airdrop Campaign Types & Bot Archetypes

> Setiap project crypto punya mekanisme airdrop berbeda. SimpleChain bot 
> hanyalah SATU contoh (testnet + daily checkin + faucet). Bot untuk project 
> lain bisa sangat berbeda tergantung campaign type-nya.
>
> File ini adalah taxonomy campaign types + bot archetypes yang harus 
> diadaptasi per project. Gunakan ini sebagai MENU PILIHAN — bukan template 
> tunggal. Setiap project bisa kombinasi multiple types.

---

## 1. Campaign Type Taxonomy

### Type A: Testnet Farming (API-based)

Project punya task platform sendiri (dashboard web) dengan API endpoints. 
User connect wallet, complete daily tasks via API calls.

**Markers:**
- Ada dashboard/task page (task.project.com, app.project.com)
- Login via wallet signature (ethers.js signMessage)
- Daily check-in, task list, points system
- Kadang ada faucet claim

**Examples:** SimpleChain, Union, Plume, Movement, MegaETH testnets

**Bot scope:** Full automation possible (API-only)
**Difficulty:** Medium (perlu trace API + handle auth)
**Pricing:** Rp 50-100K

### Type B: On-Chain Interaction Farming

Project butuh user melakukan on-chain transactions: swap, LP, bridge, 
stake, mint. Points dihitung dari on-chain activity volume.

**Markers:**
- Tidak ada dashboard task terpisah
- Points = on-chain tx volume / frequency
- Butuh gas token native chain
- Ada DEX/bridge/staking contract

**Examples:** Hyperliquid (trading volume), Drift (perp trading), 
Jupiter (swap volume), Blur (bid volume), UpOnly (buy/sell volume)

**Bot scope — THREE variants:**

*Variant B1: EVM Contract Bot* — Hard. Butuh ethers.js Contract + ABI + 
gas estimation + MEV consideration. Seringkali butuh strategi trading.
Difficulty: High. Pricing: Rp 75-150K.

*Variant B2: Solana Anchor Volume Farm* — Medium-High. Butuh @solana/web3.js 
+ raw Anchor instruction building. Pattern: buy → submit tx signature to 
backend API → unlock/sell → submit signature. Volume = round-trip count × 
trade amount. Backend tracks volume from on-chain signatures.
Difficulty: Medium-High (perlu trace IDL + PDA seeds dari JS bundle).
Pricing: Rp 75-100K.

*Variant B3: DEX Aggregator Volume Farm* — Medium. API returns serialized 
Solana instructions (computeBudget + setup + swap + cleanup). Bot parses 
+ builds VersionedTransaction with Address Lookup Tables + signs + sends. 
No raw Anchor/IDL needed. No auth — wallet address = identifier. 
Pattern: getSwapQuote → getSwapInstructions → build VersionedTransaction 
→ sendAndConfirm. Volume = swap count × amount.
Difficulty: Medium (API handles route finding + instruction building).
Pricing: Rp 80K.
Bot structure:
```
src/
├── api.js      # REST API (swap quote, swap instructions, wallet info, tokens)
└── trader.js   # VersionedTransaction builder + sender (no raw Anchor)
```
Key difference from B1/B2: DEX aggregator APIs handle all complex parts 
(route finding, instruction serialization, ALT management). Bot is thin 
wrapper: quote → tx → sign → send. Much simpler than raw Anchor.
Reference: `~/LYID-BOTS/flpp-bot/` (FLPP.io Flipper DEX)

*Variant B3: DEX Aggregator Volume Farm* — Medium. API returns serialized 
Solana instructions. Bot parses + builds VersionedTransaction with Address 
Lookup Tables + signs + sends. No raw Anchor/IDL needed. No auth — wallet 
address = identifier. Pattern: getSwapQuote → getSwapInstructions → build 
VersionedTransaction → sendAndConfirm. Volume = swap count × amount.
Difficulty: Medium (API handles route finding + instruction building).
Pricing: Rp 80K.
Reference: `references/flpp-dex-aggregator-example.md`

**Key difference from B1/B2:** DEX aggregator APIs handle all the complex 
parts (route finding, instruction serialization, ALT management). Bot is 
just a thin wrapper: quote → tx → sign → send. Much simpler than raw 
Anchor instruction building.

*Variant B4: EVM Perp DEX SDK Bot* — Medium. Uses pre-built SDK (e.g.
@hertzflow/sdk-v2) with viem for contract interactions. SDK provides all
ABIs, contract addresses, and helper functions. Bot is thin wrapper:
init SDK → approve USDT → createOrder. No raw ABI encoding needed.
**See Type O for full details** including pool/vault operations.
Key difference from B1: SDK handles all contract complexity.
Dependencies: @hertzflow/sdk-v2, viem.
Difficulty: Medium (SDK does heavy lifting).
Pricing: Rp 80K.
Reference: `~/LYID-BOTS/hertzflow-bot/`

**Bot scope:** Hard — butuh smart contract interaction, gas management,
MEV consideration. Seringkali tidak bisa full automate (perlu strategi
trading, bukan blind repeat).
**Difficulty:** High (butuh ethers.js Contract + ABI + gas estimation)
**Pricing:** Rp 75-150K (atau tidak layak build — terlalu complex)

### Type C: Quest Platform Campaigns

Project host campaign di platform quest pihak ketiga: Galxe, Zealy, 
Layer3, QuestN, Intract. User complete tasks di platform tersebut.

**Markers:**
- Campaign URL: galxe.com/... atau zealy.io/... atau layer3.xyz/...
- Tasks: follow Twitter, join Discord, retweet, visit website, 
  quiz answers, connect wallet, verify social
- Verification: API callback dari platform quest

**Examples:** Banyak project baru launch Galxe campaign sebelum testnet

**Bot scope:** Terbatas — platform quest punya anti-bot + verification 
system. Beberapa task bisa di-automate (quiz dengan jawaban fix, 
social verify dengan API), tapi banyak yang perlu manual.
**Difficulty:** Medium-High (perlu intercept platform quest API)
**Pricing:** Rp 25-50K (karena terbatas)

### Type D: Social Task Campaigns (Direct API)

Project punya dashboard sendiri untuk social tasks: follow Twitter, 
join Telegram, retweet, YouTube subscribe. Verification via API.

**Markers:**
- Dashboard: app.project.com/tasks
- Task type: "Follow @project_twitter", "Join t.me/project"
- Verification: POST /task/verify dengan taskId
- Kadang perlu screenshot upload

**Examples:** Banyak Telegram mini-apps (Notcoin-style), 
web3 social platforms

**Bot scope:** Partial — bisa automate task claim (POST verify), 
tapi user harus benar-benar follow/join dulu (atau pakai API 
social platform untuk fake-verify, tapi berisiko ban).
**Difficulty:** Medium
**Pricing:** Rp 25-50K

**Variant: Multi-Ecosystem (Type D+)**
Beberapa platform (e.g. Afyniti) punya multiple ecosystems/sub-platforms
dengan routing via `ecosystemId` parameter. Bot harus iterate semua
ecosystems dan process quests per-ecosystem. Quest completion endpoint
bisa jadi sangat simple (POST empty body ke `/quest-verifications/{id}/verify`).
Daily check-in di-flag via `isDailyCheckin: true` di quest object, bukan
hanya dari actionType.

**Examples:** Afyniti (5 ecosystems: Afyniti/Amua/Telos/Efdot/Futurist)

### Type E: Quiz/Trivia Campaigns

User jawab quiz untuk dapat points/reward. Quiz bisa:
- Fixed questions (cari jawaban di docs)
- Dynamic questions (generate dari content)
- Daily quiz (1 per hari, different question)

**Markers:**
- Dashboard: "Daily Quiz", "Trivia", "Answer to earn"
- Form input atau multiple choice
- Points = correct answers × streak

**Examples:** Binance Word of the Day, Coinbase learn, 
project-specific quizzes

**Bot scope:** High automation if questions predictable. 
Untuk dynamic questions, butuh LLM to answer.
**Difficulty:** Low-Medium (fixed) / High (dynamic)
**Pricing:** Rp 25-50K (fixed) / Rp 75-100K (dynamic + LLM)

### Type F: Faucet/Drip Campaigns

Pure faucet — claim free tokens setiap X hours. Minimal complexity.

**Markers:**
- Page: /faucet, /claim, /drip
- Captcha (hampir selalu Turnstile/reCAPTCHA)
- Cooldown timer (1h, 6h, 24h)
- Wallet address input

**Examples:** Semua testnet faucet (SimpleChain, MegaETH, 
Monad, etc.)

**Bot scope:** Full automation dengan captcha solver
**Difficulty:** Low
**Pricing:** Rp 25-35K (standalone faucet bot)

### Type G: Referral/Affiliate Farming

Points dari invite/referral. Bot butuh generate referral links, 
manage multiple accounts, simulate referrals.

**Markers:**
- Dashboard: "Invite friends", referral code/link
- Points = referrals × bonus per referral
- Kadang multi-tier (referral of referral)

**Examples:** Hamster Kombat, Notcoin, Telegram mini-apps

**Bot scope:** Full automation dengan multi-wallet + proxy rotation
**Difficulty:** Medium (sybil detection avoidance)
**Pricing:** Rp 50-75K

### Type H: Telegram Bot Campaigns (2 Sub-tipe)

**WAJIB bedakan sub-tipe saat identifikasi campaign:**

#### Type H1: Telegram Mini-App (WebApp)
Bot buka web page di dalam Telegram (in-app browser). Full SPA embedded.

**KEY INSIGHT: Automation = API calls, bukan Telegram interaction.**
Bot Telegram hanyalah frontend. Backend-nya REST API biasa.

**Auth flow (QUERY_ID pattern):**
```
1. User buka bot di Telegram WebApp (web.telegram.org)
2. Telegram inject QUERY_ID ke Local Storage
3. Extract QUERY_ID dari DevTools (Application → Local Storage)
4. POST QUERY_ID ke auth endpoint:
   POST /api/v1/auth/provider/PROVIDER_TELEGRAM_MINI_APP
   Body: { "query": "<QUERY_ID>", "referralToken": "xxx" }
5. Response: { "token": { "access": "jwt_token" } }
6. Semua API calls pakai Authorization: Bearer ***
```

**Multi-subdomain pattern:** Banyak Telegram bots pakai subdomain
berbeda per concern:
- `user-domain.*` → auth (login, user info)
- `game-domain.*` → core gameplay (farming, game, balance)
- `earn-domain.*` → task/quest system

**Markers:**
- Bot: @project_bot
- Button "Open App" → web page terbuka di Telegram
- Interaksi tap/swipe/complete tasks di web UI
- Points = taps × upgrades × referrals

**Examples:** Notcoin, Hamster Kombat, TapSwap, Blum,
banyak "tap-to-earn" 2024-2026, Spekter Agency (a16z backed, rogue-lite game, earn Sparks)
**Common API endpoints:**
```
POST /auth/provider/PROVIDER_TELEGRAM_MINI_APP  → Bearer token
GET  /user/balance                               → balance + farming status
POST /farming/claim                              → claim farm reward
POST /farming/start                              → start farming session
POST /daily-reward?offset=-420                   → daily claim
GET  /tasks                                      → task list (grouped by category)
POST /tasks/{taskId}/start                       → start task
POST /tasks/{taskId}/claim                       → claim task reward
POST /game/play                                  → get game ID
POST /game/claim                                 → claim game points {gameId, points}
```

**Automation approaches (prioritas):**
1. **Reverse-engineer API** (terbaik) — trace WebApp network calls, panggil langsung
2. **QUERY_ID extraction** — user extract dari DevTools, bot pakai untuk auth
3. **Browser automation** (fallback) — Playwright + inject Telegram.WebApp mock

**Bot scope:** Full automation via API — extract QUERY_ID once,
call API endpoints. Mirip Type A tapi auth pakai QUERY_ID
(bukan wallet signature). Jarang ada captcha.
**Difficulty:** Medium
**Pricing:** Rp 50-75K

**Reference:** `references/telegram-miniapp-blum-example.md`

#### Type H2: Inline Button Bot (Pure Telegram)
Seluruh interaksi via inline keyboard buttons di dalam chat. Tidak ada WebApp.
User klik button → bot proses → update response.

**Markers:**
- Bot: @project_bot dengan /start
- Inline keyboard buttons (bukan web page)
- Navigasi: Menu → Submenu → Task → Claim
- Kadang ada quiz/trivia inline

**Examples:** DORA Round 2 bot, banyak airdrop bot sederhana

**Automation approaches (prioritas):**
1. **Reverse-engineer API** (terbaik) — trace backend calls dari bot
2. **MTProto userbot** (fallback) — gramjs (JS) atau Telethon (Python), login sebagai USER account, bisa klik button bot lain
3. **Telegram Bot API** (TIDAK BISA) — hanya bisa kirim, tidak bisa handle callback_query dari interaksi sendiri

**Key limitation:** Telegram Bot API TIDAK BISA automate interaksi user dengan bot lain. Hanya MTProto userbot (login sebagai akun user) yang bisa klik inline buttons.

**Bot scope:** Medium — MTProto userbot works, tapi butuh nomor HP per akun.
**Difficulty:** Medium-High
**Pricing:** Rp 50-75K

**Baca `references/telegram-bot-campaign.md` untuk detail lengkap.**

### Type I: Node Running / Infrastructure

Project butuh user run node (light node, validator, RPC node). 
Points = uptime + bandwidth.

**Markers:**
- "Run a node", "Be a node operator"
- Download software (CLI, Docker)
- Uptime tracking
- Stake required (kadang)

**Examples:** Celestia light node, EigenLayer operator, 
Solana RPC

**Bot scope:** Infrastructure setup, bukan automation script. 
Bot bisa handle: auto-restart, health check, multi-node deploy.
**Difficulty:** High (DevOps, bukan web automation)
**Pricing:** Rp 100-200K (atau tidak layak — terlalu infra)

### Type J: Snapshot/Retroactive Airdrop

Tidak ada active farming — airdrop berdasarkan snapshot on-chain 
history. User yang sudah interact sebelum date X dapat tokens.

**Markers:**
- "Snapshot date: [date]"
- "Eligibility: interacted with [protocol] before [date]"
- Claim page: claim.project.com
- Tidak ada task untuk "farming" aktif

**Examples:** Uniswap UNI, ENS, Arbitrum ARB, 
dYdX DYDX

**Bot scope:** Tidak ada farming bot — user tinggal claim. 
Bot bisa: check eligibility, multi-wallet claim.
**Difficulty:** Low (claim only)
**Pricing:** Rp 25-35K (claim bot multi-wallet)

---

### Type K: XP Social Task Campaigns

Projects that reward users with XP points for completing social tasks
(follow, retweet, join Discord, etc.) via their own dashboard. Different
from Type C (Galxe/Zealy) — tasks are on the project's own platform.

**Examples:** Afyniti
**Auth:** Usually wallet signature (SIWE)
**Bot scope:** Auto-verify social tasks, claim XP
**Difficulty:** Medium
**Pricing:** Rp 40-60K

---

### Type L: Agent Marketplace

AI agent service platforms where agents earn by completing tasks for users.
NOT traditional airdrop farming — earn via task bounties + on-chain reputation.

**Examples:** OKX.AI
**Auth:** Email + OTP → TEE-based wallet (Agentic Wallet)
**Bot scope:** Register as ASP, browse tasks, apply, deliver research/analysis
**Key difference:** No daily tasks or quests — task marketplace with escrow payments
**Chain:** X Layer (gas-free)
**CLI:** OnchainOS (`onchainos` binary, not npm package)
**Difficulty:** Medium
**Pricing:** Rp 50-80K (ASP registration + task automation)
**Airdrop signal:** Early platform participation → potential token reward

---

### Type M: DEX Aggregator Volume Farm

Solana DEX aggregators that return serialized transaction instructions.
Bot builds VersionedTransaction + Address Lookup Tables, signs, sends.

**Examples:** FLPP.io (Flipper DEX)
**Auth:** None (wallet address = identifier)
**Bot scope:** getSwapQuote → getSwapInstructions → buildTx → sendAndConfirm
**Key pattern:** API returns base64 instructions, bot parses + builds Solana tx
**Dependencies:** @solana/web3.js, bs58 (v5, NOT v6 ESM-only)
**Difficulty:** High
**Pricing:** Rp 80-100K

---

### Type N: Discord Role-Gated Airdrops

Project requires Discord community roles before granting access to
testnet tokens, faucet, or airdrop eligibility. The Discord engagement
itself is the gating mechanism.

**Markers:**
- Faucet requires "access code" generated per Discord role holder
- Testnet access limited to specific role holders
- Bot/deployer tasks only available after role verification
- Discord server has role-based channel access

**Examples:** Ritual Foundation (Discord roles gate faucet + Genesis 1000)

**Bot scope:** Two-phase: (1) Discord engagement to earn roles
(use Discord Chat BOT), then (2) testnet interaction bot (Type A).
Phase 1 cannot be fully automated — requires sustained chat activity.

**Tool:** `layerairdrop/Discord-Chat-BOT` (Python selfbot)
**Reference:** `references/discord-chat-bot-engagement.md`
**Difficulty:** Medium
**Pricing:** Rp 50-80K

---

### Type Q: Options on Prediction Markets (REST Quote + On-Chain Settle)

Prediction market options platforms with hybrid off-chain/on-chain flow.
Off-chain REST API for price discovery (quote-request → market maker competition → commit),
on-chain for settlement (USDC approve → fill() → ERC-1155 mint).

**Markers:**
- Prediction market options (calls, puts on event probabilities)
- REST API for quote request + commit (off-chain, no gas)
- On-chain fill() for settlement (EVM, gas required)
- Market maker relay (SSE/WebSocket for quote streaming)
- USDC collateral, ERC-1155 option tokens
- Auth: NONE for traders (on-chain fill = auth), API key for market makers

**Examples:** Convallax (Polymarket options, Polygon Amoy testnet, Alliance DAO backed)

**Bot scope:** Full automation — faucet + quote request + commit + fill() loop
**Key pattern:** 2-phase trade:
1. Off-chain: POST /quote-requests → commit
2. On-chain: approve USDC → fill(signedOrder)
**Multi-wallet:** Very feasible (Privy email/Google per wallet, no KYC)
**Pitfalls:**
- Market maker must respond to quote request (latency-dependent, need retry)
- Quote requests expire (typically 60s) — commit quickly, no delays
- API prefix inconsistency: discovery endpoints use /v1/, trading uses bare paths
- On-chain fill() needs exact spender/amount from commit response (takerApproval object)
- `createSeries()` may be owner-only (OwnableUnauthorizedAccount) — check seriesExists() on-chain first
- seriesId = hash(conditionId, strikeBps, expiry, optionType) — must use EXACT params from existing series
- Frontend proxy may not forward nested POST paths — use direct API for commit with fallback
- DNS intermittent on direct API — implement fallback to frontend proxy
- Check POL/native balance before trading (ethers estimate > actual cost)

**Difficulty:** Medium-High
**Pricing:** Rp 65-80K
**Reference:** `endpoint-hunter/references/convallax-options-on-prediction-markets.md`

### Type Q (Options on Prediction Markets)
```
src/
├── api.js          # REST API: markets, series, quote-requests, commit, faucet
├── trader.js       # On-chain: approve USDC → fill(signedOrder) via ethers.js
└── (no auth.js)    # No auth needed — on-chain fill = authorization
```
config.json needs: base_url (api.convallax.com), rpc_url (Polygon Amoy), wallet, markets refresh interval.
Menu: Faucet (all wallets), Trade (random market), Farm Volume, Run All, Exit
Key: 2-phase flow — off-chain quote discovery + on-chain settlement.
Pitfall: Market maker response latency → need retry/timeout on quote polling.
Reference: `endpoint-hunter/references/convallax-options-on-prediction-markets.md`

---

## 2. Bot Archetype Decision Matrix

Saat dapat project baru, identifikasi campaign type(s) dulu. 
Satu project bisa kombinasi (SimpleChain = Type A + Type F).

| Campaign Type | Bot Archetype | Captcha? | Multi-wallet? | Est. Difficulty |
|---------------|---------------|----------|---------------|-----------------|
| A: Testnet Farming | API Bot + Menu | Maybe | Yes | Medium |
| B: On-Chain Volume | Contract Bot | Rare | Yes | High |
| B3: DEX Aggregator | Swap Instruction Bot | No | Yes | Medium |
| B3: DEX Aggregator Volume | API + VersionedTx | No | Yes | Medium |
| B4: EVM Perp DEX SDK | SDK + viem | No | Yes | Medium |
| C: Quest Platform | Limited Auto | Maybe | Yes | Medium-High |
| D: Social Tasks | API Verify Bot | Rare | Yes | Medium |
| E: Quiz (fixed) | Quiz Bot | Maybe | Yes | Low-Medium |
| E: Quiz (dynamic) | LLM Quiz Bot | Maybe | Yes | High |
| F: Faucet Only | Faucet Bot | Always | Yes | Low |
| G: Referral Farm | Sybil Bot | Rare | Yes (critical) | Medium |
|| H | Telegram Mini-App | QUERY_ID Bot | Rare | Yes | Medium |
| H2: Telegram Inline Bot | MTProto Userbot | Rare | Yes (phone) | Medium-High |
| I: Node Running | Infra Script | No | Optional | High (DevOps) |
| J: Snapshot Claim | Claim Bot | Maybe | Yes | Low |
| K: XP Social Task | API Verify Bot | Rare | Yes | Medium |
| L: Agent Marketplace | ASP Task Bot | No | No | Medium |
| M: DEX Aggregator Volume | API + VersionedTx | No | Yes | High |
| O: Perpetual DEX (GMX) | Multicall Bot (viem) | No | Yes | High |
| P: Uniswap V3 DEX Swap/LP | ethers Contract Bot | No | Yes | Medium |
| N: Discord Role-Gated | Engagement + Testnet Bot | No | Yes | Medium |
| Q: Options on Prediction Markets | REST Quote + On-Chain fill() | No | Yes | Medium-High |

---

## 3. Per-Type Bot Structure Variations

Base structure (all bots):
```
project-bot/
├── index.js
├── config.json
├── package.json
├── privateKey.txt (gitignored)
├── proxy.txt (gitignored)
├── .gitignore
├── README.md
├── src/
├── utils/
└── LISTING_TELEGRAM.txt
```

### Type A (Testnet Farming) — like SimpleChain
```
src/
├── api.js          # TaskAPI: login, checkin, tasks, faucet
├── auth.js         # Wallet signature → Bearer token
└── captcha.js      # If Turnstile present
```
Menu: Check-in, Faucet, Tasks, Info, Run All, Full Daily, Exit

### Type B (On-Chain Volume) — EVM
```
src/
├── contracts.js    # Contract instances + ABIs
├── trader.js       # Swap/bridge/stake logic
├── gas.js          # Gas estimation + optimization
└── wallet.js       # Wallet management, nonce tracking
```
Menu: Swap, Bridge, Stake, Auto-Trade (strategy), Exit
No captcha usually. Complex — needs strategy module.

### Type B (On-Chain Volume) — Solana Anchor
```
src/
├── api.js          # REST API (faucet, user info, process-manual signature)
├── trader.js       # Anchor instruction building, PDA derivation, ATA management
└── (no auth.js)    # No auth needed — wallet address = identifier
```
config.json needs: program_id, devnet constants (mints, pool addresses), 
PDA seed strings, referral_address, rpc_url.
Menu: Faucet, Set Referral, Buy, Unlock, Farm Volume, Full Daily, Exit
Key: must submit on-chain tx signature to backend after each trade.

### Type B3 (DEX Aggregator Volume Farm) — Solana
```
src/
├── api.js          # REST API (swap quote, swap instructions, wallet info, tokens)
└── trader.js       # VersionedTransaction builder + ALT manager (no raw Anchor)
```
config.json needs: base_url, rpc_url, slippage_bps, farm_rounds, 
farm_amount_lamports, input_mint, output_mint.
Menu: Wallet Info, Top Tokens, Single Swap, Farm Volume, Run All, Exit
Key: API returns serialized instructions — bot just signs + sends.
No auth needed (wallet address = identifier). No captcha.
Reference: `~/LYID-BOTS/flpp-bot/`

### Type B3 (DEX Aggregator Volume) — Solana
```
src/
├── api.js          # REST API (swap quote, swap instructions, wallet info — ALL public, no auth)
└── trader.js       # VersionedTransaction builder, Address Lookup Table fetcher, tx sender
```
config.json needs: base_url, rpc_url, slippage_bps, farm_rounds, farm_amount_lamports, input_mint, output_mint.
Menu: Wallet Info, Top Tokens, Single Swap, Farm Volume, Run All Wallets, Exit
Key: API returns serialized instructions (base64). Bot parses into TransactionInstruction objects,
builds VersionedTransaction with ALTs, signs, sends. No raw Anchor/IDL needed.
Reference: `references/flpp-dex-aggregator-example.md`

### Type D (Social Tasks — Direct API)
```
src/
├── api.js          # getTasks, verifyTask, getUserInfo
└── auth.js         # Wallet sig (Pattern A/B/C depending on project)
```
Menu: Verify All Tasks, Get User Info, Run Daily (all wallets), Exit
No captcha usually. Bot POSTs verify per task — user must actually do the social action first.

## Type D+G: Social Tasks + Referral

Combination of daily social task verification + referral tracking. Bot completes social tasks (follow Twitter, join Discord, etc.) via verify endpoint, then tracks referral progress.

**Pattern:**
- GET /tasks → array in `res.data.list` (Watt2Trade) or `res.data.data`
- POST /verify-task with `{taskId, action}` (NOT `taskAction`)
- Each task has status: pending/completed
- Referral count tracked separately via user info endpoint

**Auth:** REST wallet signature Pattern C → JWT cookies (access_token + refresh_token)

**Examples:** Watt2Trade (27 tasks, 6 wallets, no captcha, score 82)

**Pitfalls:**
- Task array field name varies: `res.data.list` vs `res.data.data` — always `Object.keys(res.data)` first
- Verify payload field names vary: `{taskId, action}` vs `{taskAction}` — check endpoint docs
- JWT cookies may expire mid-run — implement cookie refresh on 401
```
src/
├── api.js          # getTasks (res.data.list!), verifyTask ({taskId, action}), getUserInfo (referral_code)
└── auth.js         # Cookie-based JWT (Pattern C) or Bearer (Pattern B)
```
Menu: Verify All Tasks, Get User Info + Referral, Run Daily (all wallets), Exit
Pattern: 20-30 social tasks per wallet, each verified via POST. Referral code from getUserInfo.
Example: Watt2Trade (27 tasks, Pattern C cookie auth, no captcha). See `references/watt2trade-example.md`.
Key pitfalls: task array may be `res.data.list` not `res.data.data`; verify payload may be `{taskId, action}` not `{taskId, taskAction}`.

### Type E (Quiz)
```
src/
├── quiz.js         # Question fetcher + answerer
├── answers.js      # Fixed answer database (if predictable)
└── api.js          # Submit answer endpoint
```
Menu: Answer Daily Quiz, Bulk Answer (multi-wallet), Exit
For dynamic quiz: integrate LLM (OpenAI/local) to generate answers.

### Type F (Faucet Only)
```
src/
├── api.js          # Faucet claim endpoint
└── captcha.js      # Turnstile solver (required)
```
Menu: Claim Faucet (all wallets), Exit
Simplest bot — just captcha + claim loop.

### Type H2 (Telegram Inline Button Bot)
```
src/
├── client.js       # MTProto client (gramjs/Telethon)
├── handler.js      # Inline button click handler + response parser
├── tasks.js        # Task completion logic (check-in, quiz, social)
└── auth.js         # Session management + phone verification
```
Menu: Start Bot, Complete Tasks, Daily Check-in, Run All (multi-account), Exit
Key: BUTUH MTProto userbot (gramjs/Telethon), BUKAN Telegram Bot API.
Sessions disimpan per nomor HP. 1 proxy per akun.
See `references/telegram-bot-campaign.md`.

### Type G (Referral Farm)
```
src/
├── api.js          # Register, get referral code, verify referral
├── wallet.js       # Generate wallets, fund from main
└── proxy.js        # Critical — 1 proxy per referral account
```
Menu: Generate Accounts, Fund Wallets, Complete Referrals, Exit
High sybil risk — needs careful proxy + fingerprint rotation.

### Type J (Snapshot Claim)
```
src/
├── api.js          # Check eligibility, claim endpoint
└── checker.js      # Multi-wallet eligibility scanner
```
Menu: Check Eligibility, Claim All (eligible wallets), Exit
Simplest — just check + claim. No farming.

---

## 4. Adaptive Build Process

Saat build bot untuk project baru, JANGAN langsung copy SimpleChain 
structure. Ikuti process ini:

### Step 1: Identify Campaign Type(s)

Baca project info + dashboard + docs. Identifikasi:
- Ada dashboard task? → Type A or D
- Ada on-chain interaction requirement? → Type B
- Hosted di Galxe/Zealy/Layer3? → Type C
- Ada quiz/trivia? → Type E
- Faucet only? → Type F
- Referral system? → Type G
- Telegram mini-app (WebApp)? → Type H1
- Telegram inline buttons only? → Type H2
- Node running? → Type I
- Just snapshot claim? → Type J

Satu project = multiple types. Document semua.

### Step 2: Select Bot Archetype

Based on campaign types identified, pick bot archetype:
- Pure Type A → SimpleChain-like (api.js + auth.js + menu)
- Type A + F → SimpleChain-like + faucet module
- Type B → Contract interaction bot (complex)
- Type E fixed → Quiz database bot
- Type E dynamic → LLM-powered quiz bot
- Type F only → Minimal faucet bot
- Type J → Claim checker + multi-wallet claim

### Step 3: Adapt Structure

Start from base structure, add/remove modules per archetype:
- No captcha? Delete src/captcha.js + turnstile config
- No wallet signature? Replace auth.js with API-key/JWT auth
- On-chain tasks? Add src/contracts.js + ABI files
- Quiz? Add src/quiz.js + src/answers.js
- Referral? Add src/wallet.js (generator) + enhance proxy.js

### Step 4: Adapt Menu

Menu options harus reflect campaign types:
- Type A: Check-in, Tasks, Info, Run All
- Type F: Claim Faucet
- Type E: Answer Quiz
- Type J: Check Eligibility, Claim All
- Type G: Generate Accounts, Complete Referrals

Jangan pakai menu SimpleChain jika project hanya faucet — 
sesuaikan.

### Step 5: Adapt Config

config.json fields depend on campaign type:
```json
{
    // Common
    "delay_min": 2000,
    "delay_max": 6000,
    "retry_count": 3,
    "proxy_mode": "direct",
    
    // Type A/B/D (API-based)
    "inviteCode": "...",
    "baseUrl": "https://api.project.com",
    
    // Type A/F (captcha)
    "captcha_provider": "2captcha",
    "captcha_api_key": "...",
    "turnstile": { "task_sitekey": "...", "faucet_sitekey": "..." },
    
    // Type B (on-chain)
    "chain": { "chainId": 1, "rpc": "...", "tokenSymbol": "..." },
    "contracts": {
        "router": "0x...",
        "staking": "0x..."
    },
    
    // Type E (quiz)
    "quiz": { "answerSource": "database" | "llm", "llm_api_key": "..." },
    
    // Type G (referral)
    "referral": { "mainWallet": "0x...", "fundingAmount": "0.001" }
}
```

---

## 5. Anti-Patterns (JANGAN LAKUKAN)

1. **Copy SimpleChain structure untuk project non-testnet** — 
   tidak semua project punya check-in/faucet. Sesuaikan.

2. **Hardcode "daily check-in" sebagai fitur utama** — 
   banyak project tidak punya check-in. Mereka punya quiz, swap, 
   atau referral sebagai core mechanic.

3. **Asumsikan wallet signature auth** — 
   beberapa project pakai API key, JWT, atau session cookie. 
   Trace dulu sebelum pilih auth method.

4. **Pakai menu 7-option untuk bot simple** — 
   faucet-only bot tidak perlu menu 7 option. Cukup "Claim All, Exit".

5. **Tambah captcha.js padahal project tidak punya captcha** — 
   jangan over-engineer. Hapus modul yang tidak perlu.

6. **Pakai ethers.js padahal project non-EVM** — 
   Solana pakai @solana/web3.js, Cosmos pakai cosmjs, 
   Sui pakai @mysten/sui.js. Sesuaikan chain.

7. **Pakai Telegram Bot API untuk automate bot lain** — 
   Telegram Bot API TIDAK BISA klik button dari bot orang lain. 
   Hanya bisa kirim pesan sebagai bot. Untuk automate interaksi 
   user dengan bot lain, WAJIB pakai MTProto userbot (gramjs/Telethon). 
   Lihat `references/telegram-bot-campaign.md`.

---

## 6. Reference: Existing Bot Examples

Di org `layerairdrop` dan `rmndkyl`, bot yang sudah ada sebagai 
pattern reference per campaign type:

| Repo | Campaign Type | Pattern |
|------|---------------|---------|
| simplechain-bot | A + F | Testnet API + faucet, wallet sig auth, Turnstile |
| canopy-bot | A + F (dual platform) | NextAuth+SIWE + Bearer token API, rewards + testnet app |
| uponly-bot | B + F + G (Solana) | Anchor raw instruction building, volume farming, no-auth API |
| Empeiria-Testnet-BOT | A | Testnet API, no captcha |
| Bubuverse-BOT | H | Telegram mini-app (WebApp) |
| initia-daily-bot | A | Testnet daily, on-chain + API |
| Lava-Commit | A | Testnet, RPC commit |
| Coresky-BOT | D | Social task verification |
| watt2trade-bot | D + G | REST wallet sig (Pattern C), JWT cookies, no captcha, 27 social tasks/wallet, referral |
| orbition-bot | D (Google OAuth) | Google OAuth PKCE auth, 9 social quests, mining start, no captcha, server-side blind verify, free script |
| unicity-bot | C + D (Proprietary Wallet) | BIP32 + custom ECDSA signing (NOT SIWE), CryptoJS wallet decryption, two-API arch, social quest verify (start→check→poll), GitHub/Twitter API integration |
| blum-airdrop-bot (reference) | H (Telegram Mini-App) | QUERY_ID auth, multi-subdomain API (user/game/earn-domain), farming + daily + tasks + game, cron jobs |
| kiedex-bot | D + A + F (Supabase) | Supabase PostgREST + Edge Functions, Google OAuth standard (NOT PKCE), array response handling, server-side trade execution, no captcha, $5 |
| flpp-bot | B3 (DEX Aggregator) | Solana swap instruction API (no raw Anchor), VersionedTransaction + ALT, no auth (wallet=identifier), volume farming, Claimr rewards widget |
| flpp-bot | B3 (DEX Aggregator) | DEX aggregator API returns serialized instructions, VersionedTransaction + Address Lookup Tables, no auth, volume farming buy/sell round-trip, no captcha, $5 |
| hertzflow-bot | B4 (EVM Perp DEX SDK) | @hertzflow/sdk-v2 + viem, on-chain createOrder (two-step keeper execution), USDT collateral, referral binding on-chain, read-only REST API for market data, no captcha, $5 |

Saat build bot baru, cari reference dengan campaign type similar 
dulu, adapt pattern-nya. Jangan mulai dari scratch jika pattern 
sudah ada.

---

### Type O: Perpetual DEX (GMX-fork)

**Trigger**: On-chain perpetual trading, GMX-style ExchangeRouter, multicall pattern.

**Examples**: HertzFlow, GMX, Vertex, Kwenta

**Automation**:
- Faucet: mint() on test token contract (if available)
- Trade: transfer(USDT→OrderVault) + multicall(ExchangeRouter, [sendWnt, createOrder])
- **Vault (HLV/GLV) Deposit**: transfer(USDT→**HlvDepositVault**) + multicall(HlvRouter/GlvRouter, [sendWnt, createGlvDeposit]) — CRITICAL: HlvDepositVault ≠ DepositVault!
- Pool Withdraw: transfer(LP→WithdrawalVault) + multicall(ExchangeRouter, [sendWnt, createWithdrawal])
- **Vault (HLV/GLV) Deposit**: transfer(USDT→**HlvDepositVault**) + multicall(HlvRouter/GlvRouter, [sendWnt, createGlvDeposit]) — CRITICAL: HlvDepositVault ≠ DepositVault!
- **Vault (HLV/GLV) Deposit**: transfer(USDT→**HlvDepositVault**) + multicall(HlvRouter/GlvRouter, [sendWnt, createGlvDeposit]) — CRITICAL: HlvDepositVault ≠ DepositVault!
- Referral: bindReferrer() on ReferralStorage
- Read-only: REST API for markets, prices, trade history, pools, vaults

**Contract discovery**: Frontend JS bundles contain `ContractName:"0x..."` pattern. Common: ExchangeRouter, DepositVault, WithdrawalVault, OrderVault, HlvRouter/GlvRouter, ReferralStorage. Some forks rename GLV→HLV but keep ABI function names as `createGlvDeposit`/`createGlvWithdrawal`.

**Key pitfalls**:
- sendTokens() doesn't work with custom test tokens — use direct transfer()
- orderType MUST be 2 (MarketIncrease), NOT 0 (MarketSwap)
- acceptablePrice: Long = max uint256, Short = 0
- USD values in API = 36 decimals (1e36), not 18
- WithdrawalVault is SEPARATE from DepositVault
- sizeDeltaUsd scaled by 1e30

**Key difference from Type B (On-Chain Volume)**: Type B uses direct DEX swap/trade. Type O uses GMX-fork ExchangeRouter with multicall pattern (sendWnt + transfer/createOrder). Requires understanding of orderType enum, acceptablePrice, sizeDeltaUsd scaling (1e30). Type O also has pool LP and vault (HLV/GLV) operations.

**Pricing**: $5-6 (medium-high complexity)

**Bot structure**:
```
├── index.js          # Main: faucet + trade + pool + referral
├── accounts.json     # Private keys per wallet
├── utils/banner.js
├── utils/logger.js
└── package.json      # @hertzflow/sdk-v2, viem, axios
```

### Type P: Uniswap V3 DEX Swap/LP (On-Chain Task Automation)

Project has on-chain tasks requiring DEX interaction (swap, LP) alongside API tasks.
DEX is Uniswap V3 fork — may have ABI modifications (e.g. ExactInputSingleParams
without `deadline` field → 7 fields instead of 8).

**Examples:** SimpleChain (swap + LP + faucet + daily tasks)
**Auth:** None for DEX (wallet = identifier), wallet signature for task API
**Key pattern:** ethers.js Contract → SwapRouter02 + NonfungiblePositionManager
**Pricing:** Rp 65-80K

**Critical pitfalls:**
1. Fork struct field count ≠ standard — compare ethers selector vs explorer TX calldata
2. `receipt.status` ethers v6: use `status && status != 0` NOT `=== 1n`
3. WETH approval needed even when sending native token as msg.value
4. Pool existence + liquidity must be verified before swap
5. Discovery: explorer search "SwapRouter" → eth_getCode → check source for struct mods

**Bot structure:**
```
src/
├── api.js      # Task API (auth, checkin, tasks, faucet)
├── auth.js     # Wallet signature auth
├── dex.js      # DEXClient: swap, LP, wrap, approve
└── captcha.js  # If Turnstile present
```

**Reference:** `~/LYID-BOTS/simplechain-bot/` (src/dex.js)
**Pitfall ref:** bot-operational-audit `references/uniswap-v3-fork-patterns.md`

---

## Decision Flowchart

```
Project baru masuk
    │
    ▼
Ada dashboard web?
├── YA → Cari API endpoints (endpoint-hunter skill)
│   │
│   ├── Auth method?
│   │   ├── Wallet signature → Type A/B/D (ethers.js)
│   │   ├── Google OAuth PKCE → Type D (manual login flow, tokens.json persistence)
│   │   ├── JWT/API key → Type A/D (axios + header)
│   │   └── Session cookie → Type A/D (cookie jar)
│   │
│   ├── Task type?
│   │   ├── Daily check-in → Type A
│   │   ├── On-chain swap/LP → Type B (needs contracts.js)
│   │   ├── Social verify → Type D
│   │   ├── Quiz → Type E
│   │   └── Faucet → Type F
│   │
│   └── Captcha?
│       ├── Turnstile → src/captcha.js + 2captcha
│       ├── reCAPTCHA → src/captcha.js + 2captcha (v2/v3)
│       └── None → skip captcha module
│
└── TIDAK → Cari platform lain
    ├── Galxe/Zealy/Layer3? → Type C (limited automation)
    ├── Telegram bot (WebApp)? → Type H1 (QUERY_ID auth → API calls)
    ├── Telegram bot (inline buttons)? → Type H2 (MTProto userbot)
    ├── Snapshot claim? → Type J (claim only)
    ├── Run node? → Type I (infra)
    ├── Agent marketplace? → Type L (ASP task bot)
    ├── DEX aggregator volume? → Type M (Solana VersionedTx)
    ├── EVM Perp DEX with SDK? → Type B4 (viem + SDK, on-chain orders)
    ├── Discord role-gated? → Type N (engagement bot + testnet bot)
    ├── Options on prediction markets? → Type Q (REST quote + on-chain fill)
    └── Just need on-chain volume? → Type B (contract bot)
```
