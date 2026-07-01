# Endpoint Hunter Pitfalls & Lessons Learned


- **Verify auth method from UI BEFORE tracing API.** "Connect Wallet" button
  bisa muncul SETELAH OAuth login (e.g. Orbition = Google OAuth first, wallet
  link second). SEBELUM mulai trace: (1) buka website, (2) klik "Sign In" /
  "Connect Wallet", (3) cek opsi auth yang tersedia (Google? Wallet? Email?),
  (4) cek apakah ada wallet-only login tanpa OAuth. Ini menentukan feasibility
  automation — project yang HANYA OAuth butuh browser automation atau user
  token injection. Lesson dari Orbition (2026-07-01): bot di-build full API
  padahal auth hanya Google OAuth → wasted effort.
- **Chain ID mismatch: project description vs API trace.** Project marketing often says "Base Mainnet" or "Ethereum L2" but the actual SIWE message reveals the real chain (e.g. Chain ID: 40 = Telos EVM). ALWAYS extract chain ID from the SIWE `Chain ID:` field in auth/login request payload, or from RPC endpoint calls (`eth_chainId`). Do NOT trust the project website description. Example: Afyniti says "Base Mainnet" on website but SIWE uses Chain ID 40 (Telos EVM). Wrong chain = bot sends txs to wrong network.
- **Quest verify body can be empty.** Some platforms (e.g. Afyniti) accept POST to `/quest-verifications/{id}/verify` with `Content-Length: 0` and empty body. Don't assume a JSON body is always needed — check the original trace first.

- **Config endpoint first**: Always check `/api/config`, `/config`, `/env`, or
  `/api/settings` endpoints — often return auth config, feature flags, referral codes.
- **Reading endpoint files with emoji**: User-provided DevTools traces often contain
  emoji (→, ✓, etc.) that cause encoding issues when read via `read_file` or `cat`.
  Use `cat -v` to safely read these files in 50-80 line chunks:
  ```bash
  sed -n '1,80p' ~/LYID-BOTS/API_Endpoint_Reference/[Project]_Endpoint.txt | cat -v
  ```
  If `read_file` returns `<<ccr:...>>` compressed content, fall back to `terminal("sed -n 'X,Yp' <file> | cat -v")`.
  `/settings` before anything else. These public endpoints reveal the full 
  architecture (API URLs, subdomains, feature flags, WebSocket URLs) without 
  any authentication.
- **Manual network trace > JS bundle mining**: When automated tracing stalls 
  (Server Actions, Cloudflare, obfuscated code), STOP and ask the user to 
  manually trace network calls in browser DevTools. 10 min manual capture 
  > hours of automated JS parsing.
- **RSC (React Server Components) ≠ REST**: Some Next.js apps use RSC. 
  Detection: Content-Type `text/plain;charset=UTF-8`, Accept 
  `text/x-component`, body is JSON array `[...]`, response is RSC wire 
  format. Parse with regex, NOT `.json()`.
- **Error message mining**: 401 error messages reveal expected auth format. 
  "Use 'Bearer ***" tells format. "Invalid session token" vs "Missing" 
  tells if header was parsed.
- **Server Action detection**: If endpoint returns 400 "Bad request" for 
  ALL REST attempts (GET, POST, with body, with headers, with cookies), 
  it's likely a Next.js Server Action requiring browser context.
- **Dual-platform auth**: Check for separate auth systems per subdomain. 
  Rewards platform (NextAuth session cookie) vs testnet app (Bearer token) 
  may need separate auth.
- **Cloudflare bypass**: API endpoints (`/api/config`, `/api/v1/*`) often 
  pass Cloudflare even when the main page is blocked. Test API paths 
  directly, don't give up after Cloudflare blocks the homepage.
- **`id` vs `job_id`**: Hermes cron jobs store ID as `id`, not `job_id`. 
  Always use `job.id || job.job_id` when reading cron data.
- **Error message mining**: When an endpoint returns 401, read the error body. 
  Different messages reveal different info:
  - "Missing authorization header" = no header sent (endpoint exists, needs auth)
  - "Invalid session token" = header parsed, token value wrong (format is correct)
  - "Invalid authorization format. Use 'Bearer ***" = tells you the exact format expected
  - 405 "Method not allowed" = endpoint exists, wrong HTTP method (try POST if GET 405)
- **Server Action detection**: If an endpoint returns 400 "Bad request" for ALL 
  REST attempts (GET, POST, with body, with headers, with cookies), it's likely 
  a Next.js Server Action. These are callable only from internal Next.js client 
  code via `Next-Action` header, not via REST. Solution: browser automation 
  required for that specific flow. See case study.
- **Dual-platform auth**: Some projects have separate platforms with different 
  auth systems. Example: Canopy Network has rewards platform (NextAuth+SIWE, 
  session cookie) AND testnet app (Bearer token API). Check EACH subdomain 
  separately for auth endpoints. One auth token does NOT work across platforms.
- **Cloudflare protection**: Main page may be Cloudflare-blocked (403 "Just a 
  moment..."), but API endpoints (`/api/config`, `/api/v1/*`) often pass through. 
  Always try API paths directly even if the homepage is blocked.
- **NextAuth + SIWE nonce**: For projects using NextAuth with SIWE, the nonce 
  is often the CSRF token from `/api/auth/csrf`. The form submission includes 
  an `accessToken` field (same value as signature). However, some servers 
  generate nonce via Server Action and the CSRF token alone is rejected. 
  If REST login fails with CredentialsSignin, fall back to browser login.
- **REST SIWE (separate from NextAuth)**: Some platforms expose SIWE as REST 
  endpoints: POST /api/v1/auth/siwe/nonce/ to get nonce, then POST 
  /api/v1/auth/siwe/verify/ with signed message to get Bearer token. 
  Each platform has a DIFFERENT statement text. Both NextAuth and REST SIWE 
  can exist on the SAME project (e.g. Canopy: rewards=NextAuth, testnet=REST).
- **RSC (React Server Components)**: Some Next.js apps use Server Actions 
  instead of REST API. Content-Type is text/plain;charset=UTF-8 (NOT 
  application/json). Accept is text/x-component. Body is 
  JSON.stringify([{...}]) sent as text/plain. Response is RSC format — 
  parse with regex: extract JSON from lines starting with digit+colon.
- **Manual network trace fallback**: When all automatic methods fail, ask 
  user to provide DevTools Network trace as text file. Parse Request URL, 
  Method, Payload, Response, Headers. This is the most reliable method — 
  real user session bypasses Cloudflare and bot detection.
- **Custom address formats**: Some chains use address formats different from 
  EVM. Canopy uses hex WITHOUT 0x prefix (`3036398b...` not `0x3036398b...`). 
  Always check what format the API expects by examining request payloads in 
  the JS bundle or network trace.
- **When to ask user for manual trace**: If REST-based auth tracing is stuck 
  after reasonable effort (tested multiple endpoint combinations, JS bundle 
  mining done, Server Action detected), ASK THE USER to manually trace 
  network calls in their browser DevTools. Provide them a clear checklist 
  of what to capture (URL, method, headers, body, response). This unblocks 
  auth flow faster than continued automated probing.

- **Proprietary wallet detection**: Some projects use custom wallets (not MetaMask/Phantom). Signs: `window.sphere`, `window.unicity`, custom `intent()` API, Nostr keys instead of EVM. These create an automation BOTTLENECK — cannot use ethers.js/web3.js. Options: (A) browser automation with wallet extension, (B) reverse-engineer wallet protocol, (C) manual auth once → save JWT → automate API calls. ALWAYS check for `window.<wallet>` objects in JS bundle before assuming standard EVM/Solana auth.
- **WebSocket wallet bridge pattern**: Some projects use a persistent WebSocket where the WALLET responds to backend queries (not just push notifications). The wallet acts as a WS client answering `wallet_query` messages with `wallet_response`. This means the wallet extension must stay connected during quest completion. Document the WS URL, message types, and close codes.
- **Custom `X-Client` header**: Some APIs require a custom client identifier header (e.g. `X-Client: quest`) alongside `Authorization`. Check for this in fetch wrapper functions in JS bundles — missing it returns 401 even with valid JWT.
- **`/api/achievements` vs `/api/quests`**: Some platforms split rewards into two systems — quests (time-limited tasks) and achievements (cumulative milestones). Both need separate endpoint coverage. Achievement claim is a separate POST from quest claim.
- **Supabase publishable key is NOT secret**: `sb_publishable_...` keys are embedded in frontend JS. Tidak perlu protect — semua orang bisa lihat. Gunakan langsung di bot headers.
- **Supabase public tables/functions**: Beberapa tables dan RPC functions bisa diakses TANPA user auth (hanya pakai publishable key). Contoh: `social_tasks` table, `generate_referral_code` RPC. Ini berguna untuk recon — enumerate tasks tanpa login.
- **Supabase auth settings endpoint**: `GET /auth/v1/settings` selalu PUBLIC dan reveals semua enabled auth providers (google, discord, email, dll). Ini endpoint pertama yang harus di-check saat detect Supabase backend.
- **PostgREST RPC parameter discovery via error messages**: When probing Supabase RPC functions, send a request with WRONG parameter names. PostgREST returns a helpful error that reveals the EXACT function signature. Example: sending `{p_answer_text: "KDX"}` to `submit_daily_quiz_answer` returns: `"Perhaps you meant to call the function public.submit_daily_quiz_answer(p_answer_option, p_answer_text, p_quiz_id)"`. This reveals all parameter names and their order WITHOUT needing documentation. Technique: try empty body `{}` first (returns "Could not find function without parameters"), then try single guessed param to trigger the "Perhaps you meant" hint. Works on ALL PostgREST-based projects.
- **Supabase quiz/trivia RPCs**: Some projects have daily quiz features via Supabase RPC functions — NOT in social_tasks table. Probe for: `get_daily_quiz_page`, `get_daily_quiz_leaderboard`, `get_user_daily_quiz_status`, `submit_quiz_answer`, `get_quiz_questions`. Some need user auth, some are public (leaderboard). Quiz types seen: `secret_code`, `trivia`, `daily_question`. These may not appear in Python bot reference — always probe independently.
- **Supabase PostgREST filter syntax**: Table queries pakai `?select=*&column=eq.value&order=col.asc`. Bukan custom query params — ini standard PostgREST syntax. Filter operators: `eq.`, `neq.`, `gt.`, `gte.`, `lt.`, `lte.`, `like.`, `in.(val1,val2)`.
- **Supabase RPC returns array**: Response dari `/rest/v1/rpc/` biasanya wrapped dalam array `[{success: true, message: "..."}]`. Jangan lupa unwrap: `result[0]?.success`.
- **Supabase RPC parameter discovery via PGRST202**: When probing Supabase RPC functions, sending wrong parameter names returns a helpful error: `"Perhaps you meant to call the function public.X(p_param1, p_param2, p_param3)"`. This reveals the EXACT function signature without needing to find it in JS bundles. Technique: send empty body `{}` first → get 404 with hint → use revealed params. KieDex example: `submit_daily_quiz_answer` was discovered via `p_answer_option, p_answer_text, p_quiz_id` in error hint.
- **Supabase daily quiz pattern**: Some Supabase-based projects have daily quiz features with RPC functions: `get_daily_quiz_page` (returns question, type, options), `submit_daily_quiz_answer` (params: p_answer_option, p_answer_text, p_quiz_id), `get_user_daily_quiz_status` (history), `get_daily_quiz_leaderboard` (top players). Quiz types: `secret_code` (cipher/puzzle, answer_text), `quiz` (multiple choice, answer_option), `puzzle` (text answer). The `daily_quiz_submissions` table exists but inserts go through RPC (RLS blocks direct insert).
- **"Staging" API may be production**: Some projects use `api-staging.*` or
  `*-staging.*` subdomains as their PRODUCTION API. Check: does `api-staging.*`
  return real data (200 with live tokens/orders)? Does `api.*` resolve at all?
  If staging has data and prod doesn't, staging IS the production API.
  Example: FLPP.io uses `api-staging.flpp.io` (prod data, 200) while
  `api.flpp.io` doesn't resolve (DNS failure). Don't skip "staging" endpoints.
- **Wallet address as Bearer token (no signature)**: Some Solana projects use
  the wallet address directly as the Bearer token for their bot/AI API.
  `Authorization: Bearer ${walletAddress}` — no JWT, no signature, no nonce.
  Detection: bot API returns 401 "Invalid token type" with random string but
  accepts valid Solana pubkey format. This is the SIMPLEST auth pattern.
- **JS bundle `getName:` extraction pattern**: Next.js apps often define API
  endpoint templates as `getName:({param:t})=>\`/api/v1/path/${t}\`` in
  minified JS chunks. This is MORE reliable than generic regex because it
  captures the exact endpoint template with parameter names. Search pattern:
  `getName:\(\{[^}]*\}\)=>\`[^`]*\`` in all JS chunks. Also look for
  `method:"POST",host:"https://..."` patterns for host + method info.
- **NestJS backend detection**: Error responses with `requestID`, `success`,
  `data.message`, `data.error`, `metadata.ts` = NestJS framework. Endpoints
  follow REST conventions. Probe `/api/v1/health` first — NestJS apps almost
  always have it.
- **On-chain-only features have no REST endpoints**: Perps, perpetual futures,
  and other on-chain features aggregated from Drift/Jupiter/FlashTrade/Adrena
  have NO REST API. Don't waste time probing `/api/v1/perps/*` — these features
  are pure on-chain Solana program calls. Document as "on-chain only" in
  endpoint docs.
- **SDK as ABI source shortcut**: Before manually tracing ABIs + contract addresses from JS bundles, check if an official SDK exists on NPM: `npm search <project>`. Packages like `@hertzflow/sdk-v2` ship all ABIs, contract addresses, and helper functions pre-configured. Install + explore `node_modules/<sdk>/build/cjs/src/` for: abis/*.js (contract ABIs), configs/contracts.js (addresses), configs/tokens.js (token list), configs/chains.js (chain IDs). This saves HOURS vs reverse-engineering minified JS bundles. Pitfall: SDK may be TypeScript-only with ESM — use the `build/cjs/` directory for CommonJS compatibility. HertzFlow example: SDK had 20+ contract addresses + complete ABIs for ExchangeRouter, ReferralStorage, etc.
- **Gitbook llms.txt fast extraction**: Many crypto projects use Gitbook for docs. Fetch `https://<project>.gitbook.io/<docs>/llms.txt` for full page index with descriptions. Append `.md` to any page URL for raw markdown. Also supports Q&A: `<page-url>?ask=<question>`. HertzFlow example: llms.txt revealed 30+ doc pages including API endpoints, smart contract architecture, SDK overview from a single curl request.
- **Custom test token faucet via mint()**: Some testnet tokens have public `mint(address, amount)` function (MethodID `0x40c10f19`). Faucet = direct contract call, not web API. Check ABI for `mint` before looking for web faucet endpoint. HertzFlow example: faucet was simple mint() on USDT contract `0x6335881872FEcab922d1d83c6Bae6E27C5a9209c`.
- **GMX-fork contract address mining from frontend JS**: GMX-fork DEXes (HertzFlow, Vertex, etc.) embed ALL contract addresses in a single frontend JS chunk as `ContractName:"0x..."` pairs. To find: (1) grep main page HTML for JS chunk paths, (2) fetch each chunk, (3) grep for `ContractName:"0x[0-9a-fA-F]{40}"` pattern. This reveals the FULL contract map (20+ contracts) in seconds — much faster than SDK or explorer lookup. Contracts typically found: ExchangeRouter, DepositVault, WithdrawalVault, OrderVault, ShiftVault, HlvRouter/GlvRouter, HlvVault/GlvVault, DataStore, SyntheticsReader, ReferralStorage, SubaccountRouter, ExternalHandler, EventEmitter, Multicall, MultichainXxxRouter, ChainlinkPriceFeedProvider.
- **GMX-fork GLV/HLV naming**: GMX forks may rename "GLV" (Gmx Liquidity Vault) to project-specific names (e.g. HertzFlow uses "HLV" = HertzFlow Liquidity Vault). BUT the Solidity function names remain `createGlvDeposit`/`createGlvWithdrawal` (GMX fork heritage). The router contract is renamed: `HlvRouter` instead of `GlvRouter`. Detection: grep frontend JS for `HlvRouter` or `GlvRouter` to find the vault router address.
- **GMX-fork pool/vault operation patterns**: Pool and vault operations in GMX forks follow a 2-step pattern: (1) transfer token to vault, (2) multicall via router. Different vaults for different operations: Pool deposit: `transfer USDT → DepositVault → multicall(ExchangeRouter, [sendWnt, createDeposit])`. Pool withdraw: `transfer marketToken → WithdrawalVault → multicall(ExchangeRouter, [sendWnt, createWithdrawal])`. Vault deposit: `transfer USDT → DepositVault → multicall(HlvRouter, [sendWnt, createGlvDeposit])`. Vault withdraw: `transfer HlvToken → WithdrawalVault → multicall(HlvRouter, [sendWnt, createGlvWithdrawal])`. CRITICAL: pool withdraw sends LP tokens (marketToken = market_address) to WithdrawalVault, NOT DepositVault.
- **GMX-fork ABI source shortcut**: Before reverse-engineering ABIs from JS bundles, check GMX V2 GitHub (gmx-io/gmx-synthetics). Contract interfaces are in `contracts/withdrawal/IWithdrawalUtils.sol`, `contracts/glv/glvDeposit/IGlvDepositUtils.sol`, `contracts/glv/glvWithdrawal/IGlvWithdrawalUtils.sol`. Struct definitions give exact ABI tuple components. Fork may rename contracts but function signatures stay identical.
- **Function selector lookup via openchain.xyz**: When a contract function reverts or selector doesn't match expected ABI, look up the actual function signature from the 4-byte selector: `curl -sL 'https://api.openchain.xyz/signature-database/v1/lookup?function=0xSELECTOR&filter=true'` → returns exact function name + param types. Also available: `https://www.4byte.directory/api/v1/signatures/?hex_signature=0xSELECTOR` (less complete). Use when: (1) viem encodes wrong selector, (2) contract reverts with "execution reverted: 0x" on function call, (3) fork renamed Solidity function names. Lesson from HertzFlow (2026-07-01): bot used `createGlvDeposit` → 0x15ce7b36, actual contract expects `createHlvDeposit` → 0xc31d4ea8. Openchain lookup revealed the exact signature in seconds.
- **Privy SIWE auth detection**: Some projects use Privy (privy.io) for wallet auth. Detection: `privy.<domain>/api/v1/siwe/authenticate` in network tab. Headers: `privy-app-id`, `privy-client`. NOT needed for bot automation — on-chain trades bypass Privy entirely. HertzFlow example: Privy auth detected but bot uses direct smart contract calls.
- **Pipeline execution MUST stop after scoring**: When triggered by "Research project ini:", the agent MUST stop after Fase 0 (research + scoring) and present results to user. JANGAN langsung lanjut ke Fase 1 (endpoint tracing) atau Fase 3 (build bot) tanpa konfirmasi user. Lesson dari HertzFlow (2026-07-01): agent langsung build bot tanpa henti → skip license-check.js, push full code ke showcase repo, lupa update master index. Pipeline yang benar: research → scoring → STOP → present → user konfirmasi → lanjut.
- **Pipeline completeness checklist**: After building any bot, verify ALL pipeline phases are done before declaring complete. Common missed phases: (1) Fase 1 Scoring — must save to ~/LYID-BOTS/projects/, (2) Fase 3 License Integration — license-check.js + KNOWN_SCRIPTS registration, (3) Fase 4b Showcase — README only, NOT source code, (4) Fase 5 Sales Listing — LISTING_TELEGRAM.txt + sales_log.md + knowledge graph, (5) Fase 5 Master Index — update rmndkyl/lyid-scripts table + bundle pricing, (6) Fase 6 Dashboard — verify bot detected. Lesson dari HertzFlow: 5 phases missed karena pipeline dijalankan ad-hoc tanpa checklist.
- **Showcase repo = README ONLY**: rmndkyl/[project]-bot PUBLIC repo HANYA berisi README.md, TIDAK ada source code. Jangan push full code ke showcase repo. Lesson dari HertzFlow: full code ter-push ke rmndkyl/hertzflow-bot → user harus manual set private. Fix: gunakan orphan branch + force push dengan README saja.
- **License bot KNOWN_SCRIPTS registration**: After adding license-check.js to bot, MUST also register in ~/LYID-BOTS/lyid-license-bot/bot.js KNOWN_SCRIPTS array. Format: { name: '[project]-bot', display: '[Project] Bot', price: [HARGA], desc: '...' }. Without this, license server won't recognize the script.
- **Sales listing + knowledge graph update**: After building bot, MUST create LISTING_TELEGRAM.txt + update sales_log.md + update knowledge graph projects.md. These are NOT optional — they're part of the pipeline.
- **Master index update**: After building bot, MUST update rmndkyl/lyid-scripts with new script row + recalculate bundle pricing. Formula: Starter = sum(3 cheapest) × 0.8, Pro = sum(all) × 0.7.
  own API does NOT have quest/rewards endpoints. All quest logic lives in
  Claimr's API (prod.claimr.io). Document Claimr endpoints separately.
- **Browserbase console expression serialization**: When using `browser_console` with Browserbase stealth mode, complex return types (objects, arrays via `JSON.stringify`, `Array.from().map()`) often return `null` even when the expression is valid. Workaround: return simple strings only. Instead of `JSON.stringify(Array.from(document.querySelectorAll('a')).map(a => ({href: a.href})))`, use `Array.from(document.querySelectorAll('a')).map(a => a.href).join('\n')`. String concatenation works; object serialization doesn't. This is a Browserbase serialization limitation, not a JS error. Always test with a simple expression first (e.g. `document.querySelectorAll('a').length`) before attempting complex extractions.
- **Agent marketplace / AI platform research**: Some crypto platforms (e.g. OKX.AI) are AI agent marketplaces, not DeFi/testnets. They have different signals: task bounty marketplaces, agent-as-service-provider models, evaluator/arbitrator staking roles, escrow smart contracts. Research focus shifts from "can we automate daily tasks?" to "is there an airdrop for early platform participants?" Key signals: on-chain escrow usage, staking requirements (OKB), role registration (User/ASP/Evaluator), early volume metrics. See `references/okx-ai-agent-marketplace.md` for case study.
- **TEE-based wallets (OKX Agentic Wallet)**: Some platforms generate wallets inside a Trusted Execution Environment — private keys are NEVER exposed to the user or agent. Login is email+OTP, wallet auto-created. Cannot export private key or seedphrase. Cannot use ethers.js/web3.js to sign — signing happens server-side in TEE. Implication: bot cannot directly sign on-chain transactions; must use platform's CLI/API for all wallet operations. OKX OnchainOS CLI (`onchainos`) is the interface. See `references/okx-ai-agent-marketplace.md` for full flow.
- **OnchainOS CLI installation on Windows/MSYS**: The `install.sh` script does NOT support MSYS/git-bash (uname returns MINGW64_NT-*, which falls through to "Unsupported OS"). Workaround: download the Windows binary directly from GitHub releases (`onchainos-x86_64-pc-windows-msvc.exe`), place in `~/.local/bin/`. Skills install via `npx skills add okx/onchainos-skills --yes -g` works fine in MSYS but PromptScript global install fails (non-blocking — skills work for other agents like Claude Code, Cursor, Hermes).
- **OwnableUnauthorizedAccount (0x118cdaa7)**: Some on-chain functions are owner-only. Custom error selector `0x118cdaa7` = `OwnableUnauthorizedAccount(address)` from OpenZeppelin Ownable. Lookups: `4byte.directory` or `api.openchain.xyz`. When bot calls createSeries/mint/admin functions and gets this error → function is restricted to contract owner, regular wallets CANNOT call it. Convallax: `createSeries()` on Core contract is owner-only — only protocol team can create new series. Bot must check `seriesExists()` on-chain before attempting commit, skip non-existent series.
- **Frontend proxy vs direct API — dual client pattern**: Some projects expose API via both a frontend proxy (`www.project.com/api/`) and a direct API (`api.project.com`). The frontend proxy may not handle all paths (nested routes return 404), while the direct API handles everything but may have intermittent DNS issues. Pattern: use frontend proxy for reads (markets, series), direct API for writes (commit, confirm), with fallback between them. Convallax: `www.convallax.com/api/markets` works, `www.convallax.com/api/quote-requests/{id}/commit` returns 404, `api.convallax.com/quote-requests/{id}/commit` returns 400 with clear error.
- **DNS intermittent failures**: Some API domains (`api.project.com`) may have intermittent DNS resolution failures (`ENOTFOUND`). Always implement fallback to frontend proxy or alternative endpoint. Convallax: `api.convallax.com` intermittently fails DNS but `www.convallax.com/api/` is always reachable.
- **seriesId mismatch from random parameters**: When a platform generates seriesId from parameters (conditionId + strikeBps + expiry + optionType), the bot MUST use EXACT parameters from an existing on-chain series. Random parameters generate NEW seriesIds that don't exist on-chain → commit fails. Correct flow: (1) get series list from API, (2) filter unsettled, (3) check `seriesExists()` on-chain, (4) use that series' exact parameters for quote request. Convallax lesson: random strike/expiry → seriesId mismatch → "Series does not exist on-chain" on commit.
- **Quote request expiry (~60s)**: RFQ quote requests expire quickly. Don't add delays (polling, series checks) between quote request and commit. If series check is needed, do it BEFORE creating the quote request. Convallax: quote requests expire in ~60s, polling + series check consumed most of the window.
- **INSUFFICIENT_FUNDS for gas**: Always check native token balance before sending transactions. ethers.js estimates `gasLimit × maxFeePerGas` which is higher than actual cost. If balance < estimate but > actual cost, tx still fails. Add balance check: `if (balance < 0.05 POL) skip`. Convallax: actual gas ~0.007 POL but estimate was 0.043 POL → failed with 0.0397 balance.
- **Market maker relay / RFQ pattern**: Some DeFi platforms use Request-for-Quote where backend coordinates price discovery between traders and market makers, then settles on-chain. Flow: POST quote-request → poll/stream quotes → POST commit → on-chain fill(signedOrder). Key pitfall: market makers may NOT respond (especially testnet). Bot must handle empty quotes with timeout + retry. Response shape varies: `{bestQuote, quoteCount, status, closed}` not `{quotes: []}`. See `references/convallax-options-case-study.md`.
- **JS bundle contract address mining (Next.js SPA)**: Frontend Next.js apps embed contract addresses in JS chunks as `ContractName:"0x..."` with `NEXT_PUBLIC_*` env var fallbacks. To find: (1) grep HTML for `/_next/static/chunks/*.js`, (2) fetch each chunk, (3) grep for `ContractName:\"0x[0-9a-fA-F]{40}\"`. Two address sets = mainnet + testnet — cross-reference known USDC addresses to distinguish. Also extract inline ABI: `name:\"functionName\",inputs:[...]`. Convallax: found 4 contracts + fill() ABI from single chunk.
- **`node -c` MSYS approval pitfall**: Running `node -c file.js` in MSYS/git-bash triggers "Command required approval (script execution via -e/-c flag)" and may return "stdin is not a tty". More reliable: `node -e "require('./file')"` — loads module, validates syntax as side effect. For batch: `node -e "require('./index'); require('./src/api'); require('./src/trader')"`.
- **CORS blocking**: Beberapa API block request dari browser lain. Pakai
  `curl` via `terminal` untuk test endpoint secara langsung.
- **Rate limiting**: Jangan hammer API saat tracing. Tunggu response sebelum 
  request berikutnya.
- **Cloudflare Turnstile**: Beberapa endpoint (check-in, faucet) memerlukan 
  Turnstile token. Sitekey bisa di-extract dari JS bundle SPA — lihat
  section "Turnstile Sitekey Extraction" dibawah. Token bisa di-auto-solve 
  via 2captcha (lihat script-architect references/2captcha-turnstile.md).
- **Dual-platform auth architecture**: Beberapa project punya DUA platform 
  terpisah dengan auth berbeda. Contoh: Canopy Network punya 
  rewards platform (NextAuth+SIWE, session cookie) dan testnet app 
  (REST API, Bearer token). Jangan asumsikan 1 platform = 1 auth. 
  Cek `/api/config` atau config endpoint publik untuk menemukan semua 
  subdomain/endpoint. Dokumentasikan setiap platform secara terpisah.
- **NextAuth nonce endpoint (400 "Bad request")**: Endpoint seperti 
  `/api/auth/nonce` pada app NextAuth+SIWE bisa saja Next.js Server Action 
  yang TIDAK bisa dipanggil via REST (GET/POST langsung). Selalu return 
  400 "Bad request" walau dengan body/headers benar. Solusi: 
  (a) cek JS bundle — nonce mungkin = `getCsrfToken()` dari `/api/auth/csrf`, 
      ATAU (b) butuh browser automation untuk trace network call yang 
      sebenarnya. Jangan habiskan waktu brute-forcing endpoint ini.
- **Cloudflare bot protection on main app**: Subdomain testnet/app sering 
  diblok Cloudflare ("Just a moment..." challenge) saat diakses via 
  `requests` atau `browser_navigate`. Tapi endpoint API publik seperti 
  `/api/config` biasanya lolos. Gunakan endpoint publik untuk enumerasi 
  subdomain, lalu trace auth flow dari JS bundle.
- **JS bundle auth pattern mining**: Saat REST API trace gagal (401 di 
  semua endpoint), fetch JS chunks dan cari: `signIn(`, `getCsrfToken`, 
  `SiweMessage`, `signMessage`, `Authorization`, `Bearer`. Context 
  sekitar pattern ini sering reveal auth flow lengkap (nonce source, 
  statement text, field names yang dikirim ke callback).
- **Variable login response structure**: Setiap project beda field name 
  untuk token (`token`, `accessToken`, `jwt`, `data.token`). Saat tracing, 
  dokumentasikan exact field path. Jika tidak bisa trace login response 
  lengkap, implementasikan fallback chain di script.
- **Signature algorithms**: Jika project pakai custom signature, perlu 
  reverse-engineer JS bundle untuk menemukan algo-nya. Cari di 
  `browser_console` dengan keyword `sign`, `hmac`, `crypto`.

## Turnstile Sitekey Extraction (SPA Apps)
