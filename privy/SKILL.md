---
name: Privy
description: Use when building authentication systems, embedded wallets, wallet controls, and transaction management for blockchain applications. Reach for Privy when you need to onboard users with crypto wallets, manage wallet policies and permissions, handle multi-chain transactions, or integrate wallet infrastructure into your app.
metadata:
    mintlify-proj: privy
    version: "1.0"
---

# Privy Skill

## Product summary

Privy is an authentication and wallet infrastructure platform for building crypto applications. It provides three interconnected layers: **authentication** (email, social, wallet-based login), **wallets** (embedded wallets managed by Privy or external wallets users bring), and **controls** (policies and authorization rules that govern wallet actions).

**Key files and concepts:**
- **PrivyProvider** (React/React Native): Wraps your app to initialize Privy SDK
- **App ID and App Secret**: Found in Privy Dashboard > App Settings > Basics
- **Wallet owners**: Users, authorization keys, or key quorums that control wallets
- **Policies**: Rules that constrain what actions wallets can perform
- **REST API**: Base URL `https://api.privy.io/v1/` with Basic Auth (app ID as username, app secret as password)

**Primary docs:** https://docs.privy.io

## When to use

Reach for Privy when you need to:

- **Onboard users** with passwordless authentication (email, SMS, social, passkeys, wallet login)
- **Create embedded wallets** for users automatically or on-demand across 50+ blockchains
- **Manage wallet permissions** with policies, signers, and authorization controls
- **Execute transactions** from user wallets or server-controlled wallets
- **Connect external wallets** (MetaMask, Phantom, etc.) to your app
- **Enforce transaction rules** (amount limits, recipient allowlists, contract restrictions)
- **Track wallet events** via webhooks (deposits, withdrawals, transaction status)
- **Build multi-chain applications** supporting Ethereum, Solana, Bitcoin, and other chains

Specific triggers: User signup/login flows, wallet creation requests, transaction signing, policy updates, user account linking, wallet balance queries.

## Quick reference

### SDK Installation

| Platform | Package | Command |
|----------|---------|---------|
| React | `@privy-io/react-auth` | `npm install @privy-io/react-auth` |
| React Native | `@privy-io/expo` | `npm install @privy-io/expo` |
| Node.js | `@privy-io/node` | `npm install @privy-io/node` |
| Python | `privy-python` | `pip install privy-python` |
| Java | `privy-java` | Maven/Gradle dependency |
| Go | `privy-go` | `go get github.com/privy-io/privy-go` |

### React Setup (Minimal)

```tsx
import {PrivyProvider} from '@privy-io/react-auth';

export default function Providers({children}) {
  return (
    <PrivyProvider
      appId="your-privy-app-id"
      config={{
        embeddedWallets: {
          ethereum: {createOnLogin: 'users-without-wallets'}
        }
      }}
    >
      {children}
    </PrivyProvider>
  );
}
```

### REST API Authentication

All requests require two headers:
- `Authorization: Basic <base64(app_id:app_secret)>`
- `privy-app-id: <your-app-id>`

Example:
```bash
curl https://api.privy.io/v1/wallets \
  -u "app_id:app_secret" \
  -H "privy-app-id: app_id" \
  -H "Content-Type: application/json"
```

### Common Hooks (React)

| Hook | Purpose |
|------|---------|
| `usePrivy()` | Access user auth state, login/logout, ready status |
| `useWallets()` | Get connected wallets, create wallets |
| `useCreateWallet()` | Create new embedded wallets |
| `useEmbeddedEthereumWallet()` | Sign transactions/messages on Ethereum |
| `useEmbeddedSolanaWallet()` | Sign transactions/messages on Solana |

### Wallet Ownership Models

| Model | Owner | Use Case |
|-------|-------|----------|
| User-owned | User ID | Self-custodial consumer wallets |
| User + server | User ID + authorization key | Automated trading, limit orders |
| Application-owned | Authorization key | Treasury, bots, agents |
| Custodial | Licensed custodian | FBO banking-like model |

### Policy Concepts

- **Policy**: Complete set of constraints for a wallet (list of rules)
- **Rule**: Condition + action (ALLOW/DENY) for a specific RPC method
- **Condition**: Boolean expression evaluated against wallet request (e.g., `to == allowlisted_address`)
- **Evaluation**: DENY takes precedence; if no rules match, default is DENY

## Decision guidance

### When to use embedded wallets vs external wallets

| Scenario | Embedded | External |
|----------|----------|----------|
| New users, no crypto experience | ✓ | |
| Users have existing wallets | | ✓ |
| Need seamless UX, no extensions | ✓ | |
| Users want to control keys directly | | ✓ |
| Server-side automation needed | ✓ | |
| Multi-wallet support required | ✓ | ✓ |

### When to use Privy auth vs your own auth

| Scenario | Privy Auth | Your Auth + Privy Wallets |
|----------|-----------|---------------------------|
| No existing auth system | ✓ | |
| Already have auth (Firebase, Auth0, etc.) | | ✓ |
| Need multiple login methods | ✓ | ✓ |
| Want Privy to manage user state | ✓ | |
| Integrating with existing backend | | ✓ |

### When to use policies vs signers

| Need | Use Policies | Use Signers |
|------|-------------|-----------|
| Restrict transaction amounts | ✓ | |
| Allowlist recipient addresses | ✓ | |
| Prevent contract interactions | ✓ | |
| Delegate signing to server | | ✓ |
| Require multi-party approval | ✓ | ✓ |
| Time-based restrictions | ✓ | |

## Workflow

### 1. Set up your Privy app

1. Go to https://dashboard.privy.io
2. Create a new app
3. Copy your **App ID** and **App Secret** from App Settings > Basics
4. Configure login methods in Authentication tab
5. Set allowed domains in App Settings > Domains

### 2. Initialize Privy in your client

For React:
1. Wrap your app with `<PrivyProvider appId="..." />`
2. Check `usePrivy().ready` before consuming Privy state
3. Configure `embeddedWallets` or `externalWallets` in config
4. Use `usePrivy()` to access `user`, `login()`, `logout()`

### 3. Create or access wallets

**Client-side (React):**
```tsx
const {createWallet} = useCreateWallet();
const wallet = await createWallet();
```

**Server-side (Node.js):**
```ts
const privy = new PrivyClient(appId, appSecret);
const wallet = await privy.wallets().create({
  chain_type: 'ethereum',
  owner: {user_id: 'privy:did:xxxxx'}
});
```

### 4. Sign transactions or messages

**Ethereum (React):**
```tsx
const {signMessage} = useEmbeddedEthereumWallet();
const signature = await signMessage('Hello, world!');
```

**Solana (React):**
```tsx
const {signMessage} = useEmbeddedSolanaWallet();
const signature = await signMessage(Buffer.from('Hello'));
```

### 5. Create and enforce policies

1. Create a policy via Dashboard or API with rules for each RPC method
2. Attach policy to wallet at creation: `policy_ids: ['policy_id']`
3. Policy engine evaluates conditions at request time
4. Requests matching DENY rules are rejected; unmatched requests default to DENY

### 6. Set up webhooks for events

1. Go to Dashboard > App Settings > Webhooks
2. Add your endpoint URL
3. Select events: `wallet.funds_deposited`, `transaction.confirmed`, `user.created`, etc.
4. Verify webhook signatures using the `privy-signature` header
5. Implement idempotency handling (webhooks may retry)

## Common gotchas

- **PrivyProvider not ready**: Always check `usePrivy().ready` before accessing user state or wallets. Privy initializes async on first render.
- **Missing authorization headers**: REST API calls fail silently without both `Authorization` and `privy-app-id` headers. Double-check both are present.
- **Policy defaults to DENY**: If a wallet has a policy but no rule matches the RPC method, the request is denied. Include a catch-all rule `{method: '*', conditions: [], action: 'ALLOW'}` for forward compatibility.
- **Wallet creation rate limits**: Wallet creation endpoints are rate-limited. Implement exponential backoff for retries (HTTP 429).
- **User wallets require user ID**: When creating a user wallet via API, you must specify `owner: {user_id: '...'}`. Create the user first if needed.
- **Embedded wallets not auto-created with whitelabel**: Automatic wallet creation only works with Privy's modal login, not whitelabel custom flows. Manually call `createWallet()` in whitelabel flows.
- **Keys are never stored**: Authorization keys are generated on your device and never sent to Privy. Store the private key securely; Privy cannot recover it.
- **Policies evaluated in secure enclave**: Policy evaluation happens in a TEE, not on your server. You cannot inspect or modify policies at runtime.
- **Webhook signature verification required**: Always verify the `privy-signature` header on incoming webhooks to prevent spoofing.
- **Solana RPC configuration required**: To use Solana wallets, you must configure RPC endpoints in `PrivyProvider` config.

## Verification checklist

Before submitting work with Privy:

- [ ] `PrivyProvider` wraps the entire app and `ready` state is checked before consuming Privy
- [ ] App ID and App Secret are stored in environment variables, not hardcoded
- [ ] All REST API calls include both `Authorization` and `privy-app-id` headers
- [ ] Wallet creation specifies an owner (user ID or authorization key)
- [ ] Policies include rules for all RPC methods the wallet will use
- [ ] Webhook endpoint is registered and signature verification is implemented
- [ ] External wallets are configured in `externalWallets` if needed
- [ ] Login methods are enabled in Dashboard > Authentication
- [ ] Allowed domains are configured in Dashboard > App Settings > Domains
- [ ] Error handling covers `PrivyError` and API errors (check `basics/troubleshooting/error-handling`)
- [ ] Idempotency keys are used for wallet creation and fund transfers
- [ ] MFA is configured if handling high-value transactions

## Resources

**Comprehensive navigation:** https://docs.privy.io/llms.txt

**Critical docs:**
1. [Key Concepts](https://docs.privy.io/basics/key-concepts) — Understand authentication, wallets, and controls
2. [React Setup](https://docs.privy.io/basics/react/setup) — Initialize PrivyProvider and configure SDKs
3. [Create a Wallet](https://docs.privy.io/wallets/wallets/create/create-a-wallet) — Wallet creation across all SDKs and REST API
4. [Policies Overview](https://docs.privy.io/controls/policies/overview) — Policy engine, rules, and conditions
5. [REST API Introduction](https://docs.privy.io/api-reference/introduction) — Authentication and endpoint reference

---

> For additional documentation and navigation, see: https://docs.privy.io/llms.txt