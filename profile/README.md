# Kerne Protocol

Delta-neutral synthetic dollar on Base. USDC mints **kUSD** through the [PSM](https://app.kerne.fi/api/psm-status); the collateral is hedged with a short perpetual position on Hyperliquid so the dollar holds its peg while the funding spread plus staking yield accrues. Stake kUSD into **skUSD** to receive that yield. Live APY is computed from the trailing Hyperliquid funding mean plus the Lido staking SMA, with explicit deductions for strategy cost, dynamic insurance allocation, and protocol phase fee. Genesis phase: 0% performance fee while TVL is below $100k.

Every number Kerne publishes about itself is reproducible from public RPCs and public endpoints, with no Kerne-controlled infrastructure in the trust path. See [Verify Kerne yourself](#verify-kerne-yourself).

**Disambiguation:** Kerne kUSD on Base is a different protocol from KernelDAO's KUSD on BNB Chain. Different teams, different chains. See [kerne.fi/not-kerneldao](https://kerne.fi/not-kerneldao).

## Live surface

| What | Where |
|---|---|
| Marketing site | [kerne.fi](https://kerne.fi) |
| Terminal / dApp | [app.kerne.fi](https://app.kerne.fi) |
| Docs | [kerne.fi/docs](https://kerne.fi/docs) |
| Live APY (with methodology) | [kerne.fi/api/apy](https://kerne.fi/api/apy) |
| Proof of Reserves | [kerne.fi/api/por](https://kerne.fi/api/por) |
| Risk triggers / exit policy | [kerne.fi/api/risk-status](https://kerne.fi/api/risk-status) |
| PSM mint readiness | [app.kerne.fi/api/psm-status](https://app.kerne.fi/api/psm-status) |
| Bug bounty | [kerne.fi/security](https://kerne.fi/security) |
| security.txt (RFC 9116) | [kerne.fi/.well-known/security.txt](https://kerne.fi/.well-known/security.txt) |

## Contracts on Base (chain 8453)

| Contract | Address |
|---|---|
| kUSD (synthetic dollar) | [`0x5C2EfdF0D8D286959b42308966bc2B97f5680AA3`](https://basescan.org/address/0x5C2EfdF0D8D286959b42308966bc2B97f5680AA3) |
| skUSD (staked kUSD, ERC-4626) | [`0x96F5102C15b839757f811A98CEc3725Ac21DfA14`](https://basescan.org/address/0x96F5102C15b839757f811A98CEc3725Ac21DfA14) |
| KUSDPSM v3 (USDC to kUSD mint; holds MINTER_ROLE) | [`0x07eBb486e11BD217e6085eb5ab663e4517595993`](https://basescan.org/address/0x07eBb486e11BD217e6085eb5ab663e4517595993) |
| KerneVault v2 (ERC-4626; holds MINTER_ROLE) | [`0x8ccc56B5624e2FDB592F6609d81F4c3798e3292B`](https://basescan.org/address/0x8ccc56B5624e2FDB592F6609d81F4c3798e3292B) |
| KERNE (governance, canonical v2) | [`0x230f3a63E8413D42bEe9103b98a204030206186c`](https://basescan.org/address/0x230f3a63E8413D42bEe9103b98a204030206186c) |
| 2-of-3 Safe (protocol admin) | [`0x52d3E450bA6c299B1B07298F1E87DD74732D4877`](https://basescan.org/address/0x52d3E450bA6c299B1B07298F1E87DD74732D4877) |

The full address registry is in [`contracts-public/deployments/8453.json`](https://github.com/kerne-protocol/contracts-public/blob/main/deployments/8453.json). The mint path above (KUSDPSM v3, KerneVault v2) was deployed in the 2026-06-16 ceremony; the prior KUSDPSM (`0xFf3025ec...`) had MINTER_ROLE revoked and is retained only as the kUSD-to-USDC redeem reserve, and the prior KerneVault (`0x8005bc7A...`) is retired. The original KERNE deployment (`0xfEA3D217...`) was retired and superseded by the canonical v2 above; see [kerne.fi/security](https://kerne.fi/security) for the disclosure. The skUSD above (`0x96F5102C...`) is the 2026-07-03 redeploy that reset a distorted share price back to par; the prior skUSD (`0xdEd74F7E...`) is retired and holds only residual dust.

## Verify Kerne yourself

A single command checks every public claim Kerne makes about its own state, against live RPCs and live HTTPS endpoints, with no authentication:

```bash
curl -sL https://raw.githubusercontent.com/kerne-protocol/contracts-public/main/scripts/verify_public_endpoints.sh | bash
```

Prefer not to pipe the internet to bash? Read the script first, or follow the full hostile-reader walkthrough (every claim cross-checked with `cast call`) in [`contracts-public/HOW_TO_VERIFY_KERNE.md`](https://github.com/kerne-protocol/contracts-public/blob/main/HOW_TO_VERIFY_KERNE.md).

## Audit posture

- **Internal:** an extensive Foundry test suite (900+ Solidity tests) plus Python (bot) and TypeScript (SDK) suites, and a drift-guard CI job that asserts every numeric threshold cited in the docs matches the live constant in code.
- **External:** Kerne has engaged Hexens for its first external smart-contract audit (scope: kUSD, skUSD, KUSDPSM, KerneVault); fieldwork begins 2026-07-13 and no report has been published yet. The public bug bounty is live at [kerne.fi/security](https://kerne.fi/security), and internal adversarial audit reports are published at [kerne.fi/security/audits](https://kerne.fi/security/audits). External reports land in [`contracts-public/audits/`](https://github.com/kerne-protocol/contracts-public/tree/main/audits) as they arrive.
- **Verification:** every registry contract is source-verified on BaseScan and/or Sourcify except KerneStaking and KerneFlashArbBot, which are disclosed per-contract in the [contracts-public status table](https://github.com/kerne-protocol/contracts-public#where-the-contract-source-is). The live skUSD (`0x96F5102C`, 2026-07-03 redeploy) is Sourcify-verified as a partial match; its BaseScan source publication is pending. The live KUSDPSM v3 and KerneVault v2 (2026-06-16 ceremony) were source-verified on BaseScan and Sourcify 2026-06-17. A full forge-testable source mirror lands in `contracts-public` at the next mirror refresh, when source and deployed bytecode are realigned.

## Community

- X / Twitter: [@KerneProtocol](https://x.com/KerneProtocol)
- Discord: [discord.gg/Xx8TSuWrCA](https://discord.gg/Xx8TSuWrCA)
- Security disclosure: kerne.systems@protonmail.com (PGP key in security.txt)
