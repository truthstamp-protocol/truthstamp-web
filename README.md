# truthstamp-web
Web pages of truthstamp
# TruthStamp Web

The complete web application for TruthStamp — frontend pages, 
JavaScript client, and Supabase Edge Functions.

🌐 **Live at:** https://truthstamp.xyz

---

## What's in This Repo

| Layer              | Technology                        |
|--------------------|-----------------------------------|
| Frontend           | Vanilla HTML / CSS / JS           |
| Auth               | Supabase Auth (email + wallet)    |
| Database           | Supabase PostgreSQL               |
| Backend functions  | Supabase Edge Functions (Deno)    |
| Blockchain         | MegaETH via ethers.js             |
| Permanent storage  | Arweave via Turbo                 |
| Hosting            | GitHub Pages                      |

No build step. No framework. Plain HTML files served directly 
from GitHub Pages — fast, simple, and easy to fork.

---

## Pages

| File                  | Description                                        |
|-----------------------|----------------------------------------------------|
| `index.html`          | Homepage — hero, featured stamps, live stats       |
| `stamp.html`          | Create a new stamp                                 |
| `dashboard.html`      | User dashboard — manage stamps, credits, handle    |
| `auth.html`           | Sign in / sign up (email + wallet)                 |
| `explorer.html`       | Public stamp explorer — browse, filter, search     |
| `proof.html`          | Individual stamp proof page                        |
| `verify.html`         | Verify any content by hash                         |
| `pricing.html`        | Credits and pricing                                |
| `buy-credits.html`    | Purchase credits (crypto + Razorpay)               |
| `trust.html`          | Permanence and trust model explained               |
| `how-it-works.html`   | How TruthStamp works                               |
| `usecases.html`       | Use cases                                          |
| `arweave-index.html`  | Arweave-native stamp index (works without DB)      |
| `decrypt.html`        | Decrypt hash-only stamp content                    |
| `faq.html`            | Frequently asked questions                         |
| `terms.html`          | Terms of Service                                   |
| `privacy.html`        | Privacy Policy                                     |

---

## Project Structure

```
truthstamp-web/
├── index.html
├── stamp.html
├── dashboard.html
├── auth.html
├── explorer.html
├── proof.html
├── verify.html
├── ... (other pages)
│
├── css/
│   ├── theme.css          ← design tokens, colors, typography
│   └── components.css     ← reusable UI components
│
├── js/
│   ├── config.js          ← contract address, RPC, Supabase config
│   ├── supabase-client.js ← Supabase client initialisation
│   ├── common.js          ← shared utilities (auth guards, helpers)
│   ├── nav.js             ← navigation, mobile drawer
│   └── analytics.js       ← analytics (privacy-respecting)
│
└── supabase/
    └── functions/
        ├── _shared/
        │   ├── config.ts          ← shared config and secrets
        │   ├── blockchain.ts      ← MegaETH contract calls
        │   ├── arweave.ts         ← Arweave upload via Turbo
        │   └── sealed-crypto.ts   ← AES-256-GCM encryption for sealed files
        ├── stamp-create/
        │   └── index.ts           ← create stamp on-chain + Arweave
        ├── seal-reveal/
        │   └── index.ts           ← decrypt + upload to Arweave + reveal on-chain
        ├── wallet-auth/
        │   └── index.ts           ← SIWE wallet authentication
        └── indexer/
            └── index.ts           ← sync on-chain events to DB
```

---

## Configuration

All environment-specific values live in `js/config.js`:

```js
const TRUTHSTAMP_CONFIG = {
  SUPABASE_URL:         'https://yourproject.supabase.co',
  SUPABASE_ANON_KEY:    'your-anon-key',
  CONTRACT_ADDRESS:     '0xD8beDEa4DdaCBF681CCca5DBFa90b04bB654d0B7',
  CONTRACT_NETWORK:     'megaeth',
  CONTRACT_CHAIN_ID:    4326,
  CONTRACT_RPC_URL:     'https://mainnet.megaeth.com/rpc',
  CONTRACT_EXPLORER:    'https://mega.etherscan.io/',
  POST_AUTH_REDIRECT:   'dashboard.html',
  WALLETCONNECT_PROJECT_ID: 'your-wc-project-id',
};
```

---

## Edge Functions

Backend logic runs as Supabase Edge Functions (Deno runtime).

### `stamp-create`
Called when a user creates a stamp. Handles:
- Content hashing (SHA-256)
- Arweave upload for public stamps
- File encryption for sealed file stamps
- On-chain `createStamp()` call via platform wallet
- DB record creation

### `seal-reveal`
Called to reveal a sealed stamp after its reveal date. Handles:
- Downloading and decrypting sealed file from Supabase Storage
- Uploading decrypted content to Arweave with correct MIME type
- On-chain `revealSealed()` call
- DB update with Arweave URI
- Storage cleanup after successful reveal

### `wallet-auth`
SIWE (Sign-In With Ethereum) authentication flow:
- Issues a nonce for the user to sign
- Verifies the signature
- Creates or retrieves Supabase user session

### `indexer`
Syncs on-chain events to the database. See 
[truthstamp-indexer](https://github.com/truthstamp-protocol/truthstamp-indexer)
for full documentation.

---

## Supabase Secrets

Set these in your Supabase project before deploying Edge Functions:

```bash
supabase secrets set PLATFORM_WALLET_KEY=0xYourPrivateKey
supabase secrets set CONTRACT_ADDRESS=0xYourContractAddress
supabase secrets set CONTRACT_RPC_URL=https://mainnet.megaeth.com/rpc
supabase secrets set CONTRACT_CHAIN_ID=4326
supabase secrets set TURBO_API_KEY=YourArweaveTurboKey
supabase secrets set SUPABASE_SERVICE_ROLE_KEY=YourServiceRoleKey
```

---

## Deploy Edge Functions

```bash
supabase link --project-ref YOUR_PROJECT_REF

supabase functions deploy stamp-create
supabase functions deploy seal-reveal
supabase functions deploy wallet-auth
supabase functions deploy indexer
```

---

## Hosting

The frontend is a static site — no server required.

**GitHub Pages (default):**  
Push to `main` and enable Pages in repo Settings → Pages → Source: main branch.

**Any static host:**  
Upload all HTML, CSS, and JS files. Vercel, Netlify, and Cloudflare Pages 
all work with zero configuration.

---

## Self-Hosting

To run your own instance:

1. **Deploy the contract** — [truthstamp-contracts](https://github.com/truthstamp-protocol/truthstamp-contracts)
2. **Create a Supabase project** and apply the DB schema
3. **Set secrets** and deploy Edge Functions (see above)
4. **Update `js/config.js`** with your contract address and Supabase keys
5. **Deploy** to GitHub Pages or any static host
6. **Run the indexer** — [truthstamp-indexer](https://github.com/truthstamp-protocol/truthstamp-indexer)

Full self-hosting guide: [tsp-spec — self-hosting.md](https://github.com/truthstamp-protocol/tsp-spec)

---

## Authentication

TruthStamp supports two auth methods:

**Email** — standard Supabase email/password auth. Users get a UUID 
as their `creatorIdentity` on-chain.

**Wallet** — MetaMask or WalletConnect via SIWE. The user's Ethereum 
address becomes their `creatorIdentity` on-chain, making their stamps 
directly attributable to their wallet.

---

## Stamp Visibility Types

| Type       | Content stored | Who can see          | On Arweave |
|------------|---------------|----------------------|------------|
| Public     | Yes           | Everyone             | Yes        |
| Sealed     | Encrypted     | Owner only until reveal | After reveal |
| Hash-only  | No            | Nobody               | No         |

---

## Related Repos

- [truthstamp-contracts](https://github.com/truthstamp-protocol/truthstamp-contracts) — Smart contract
- [truthstamp-indexer](https://github.com/truthstamp-protocol/truthstamp-indexer) — Chain indexer
- [truthstamp-explorer](https://github.com/truthstamp-protocol/truthstamp-explorer) — Explorer UI
- [truthstamp-bitcoin](https://github.com/truthstamp-protocol/truthstamp-bitcoin) — Bitcoin archival layer
- [tsp-spec](https://github.com/truthstamp-protocol/tsp-spec) — Protocol specification

---

## License

MIT
