# Campaign Type Identification (WAJIB SEBELUM BUILD)

## Campaign Type Identification (WAJIB SEBELUM BUILD)

Setiap project crypto punya mekanisme airdrop berbeda. SimpleChain bot 
(testnet + checkin + faucet) HANYA SATU CONTOH. Jangan copy-paste 
struktur SimpleChain untuk semua project.

Baca `references/campaign-types.md` untuk taxonomy lengkap (12 types) 
+ decision matrix + per-type bot structure variations + decision flowchart.

Ringkasan 13 campaign types:

| Type | Nama | Contoh | Bot Complexity |
|------|------|--------|----------------|
| A | Testnet Farming (API) | SimpleChain, Union | Medium |
| B | On-Chain Volume | Hyperliquid, Drift | High |
| C | Quest Platform | Galxe, Zealy, Layer3 | Medium-High |
| D | Social Tasks (direct API) | Coresky, Telegram mini-apps | Medium |
| E | Quiz/Trivia | Binance WOTD, project quizzes | Low-High |
| F | Faucet Only | Semua testnet faucets | Low |
| G | Referral Farm | Hamster Kombat, Notcoin | Medium |
| H | Telegram Mini-App | TapSwap, Notcoin, Blum | Medium |
| I | Node Running | Celestia, EigenLayer | High (DevOps) |
| J | Snapshot Claim | Uniswap, ENS, Arbitrum | Low |
| K | XP Social Task | Afyniti | Medium |
| L | Agent Marketplace | OKX.AI | Medium |
| M | DEX Aggregator Volume | FLPP.io | High |
| N | Discord Role-Gated | Ritual Foundation | Medium |
| O | Perpetual DEX (GMX-fork) | HertzFlow | High |

### Type O: Perpetual DEX (GMX-fork)
On-chain perpetual trading via smart contract multicall. REST API is read-only.
- **Auth**: None for API; wallet signing for on-chain txs
- **Bot complexity**: High (EVM tx building, multicall, ABIs)
- **SDK**: Check npm for project SDK (@hertzflow/sdk-v2)
- **Key pitfall**: sendTokens doesn't work with custom test tokens — use direct transfer()
- **Pattern**: `references/gmx-fork-perp-dex-pattern.md`
- **HertzFlow pool/vault endpoint doc**: `~/HERMES DATA/airdrop-tracker/knowledge-graph/data/hertzflow-pools-vaults-endpoints.md`
- **Pricing**: Rp 80K/$5+

- **Sub-operations (all via multicall):**
- **Trading**: approve SyntheticsRouter + multicall(ExchangeRouter, [sendWnt, sendTokens, createOrder])
- **Pool Deposit**: approve SyntheticsRouter + multicall(ExchangeRouter, [sendWnt, sendTokens, createDeposit])
- **Pool Withdraw**: transfer(LP→WithdrawalVault) + multicall(ExchangeRouter, [sendWnt, createWithdrawal])
- **Vault (HLV/GLV) Deposit**: approve SyntheticsRouter + multicall(HlvRouter, [sendWnt, sendTokens, createHlvDeposit])
- **Vault (HLV/GLV) Withdraw**: transfer(HLV→WithdrawalVault) + multicall(HlvRouter, [sendWnt, createHlvWithdrawal])

**Contract discovery (GMX-forks):**
Frontend JS bundles contain contract name→address mappings. Search pattern: `ContractName:"0x..."`. Common contracts: ExchangeRouter, DepositVault, WithdrawalVault, OrderVault, HlvRouter/GlvRouter, ReferralStorage, ShiftVault, DataStore, SyntheticsReader. Some forks rename GLV→HLV but keep function names as `createGlvDeposit`/`createGlvWithdrawal`.

### Cara Identifikasi Campaign Type

1. Baca project website + dashboard + docs
2. Cek: ada dashboard task? on-chain interaction? quiz? faucet? 
   referral? Telegram bot? node? snapshot claim?
3. Satu project = kombinasi multiple types (SimpleChain = A + F)
4. Document semua types yang relevant

### Adaptive Build Process

JANGAN langsung copy SimpleChain structure. Ikuti:

1. **Identify** campaign type(s) dari project info
2. **Select** bot archetype dari decision matrix 
   (lihat `references/campaign-types.md` section 2)
3. **Adapt** structure — add/remove modules per archetype:
   - No captcha? Delete src/captcha.js
   - No wallet signature? Replace auth.js dengan JWT/API-key auth
   - On-chain tasks? Add src/contracts.js + ABI files
   - Quiz? Add src/quiz.js + src/answers.js
   - Non-EVM chain? Swap ethers.js → @solana/web3.js / cosmjs / @mysten/sui.js
4. **Adapt** menu options — jangan pakai menu 7-option jika bot simple
5. **Adapt** config.json fields per campaign type

### Task Type Distinction (dalam bot)

Dalam satu bot, bedakan 2 jenis task:

**API-Only Tasks:** Cukup POST ke endpoint untuk klaim points. 
Contoh: daily check-in, social verify, quiz submit. Full automation.

**On-Chain Tasks:** Butuh smart contract interaction. 
Contoh: swap, LP, bridge, stake. Butuh ethers.js Contract + ABI + gas.

Strategi: Untuk API-only → automate langsung. Untuk on-chain → 
opsi A (bot klaim points setelah user manual swap) atau opsi B 
(bot automate on-chain juga, butuh router contract + ABI, harga premium).

