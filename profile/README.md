# Kerne Protocol

Delta-neutral synthetic dollar on Base. USDC mints **kUSD** through the [PSM](https://app.kerne.fi/api/psm-status), and every kUSD is backed 1:1 by USDC reserves anyone can recompute from chain; backing is [live and verifiable](https://kerne.fi/api/por) and sits near 100% today. The design pairs the collateral with a short perpetual hedge on Hyperliquid; that hedge engine is built and wired, but no hedge position is open right now, so the protocol currently runs as a fully USDC-reserved dollar while the delta-neutral book is seeded through Genesis. Stake **kUSD** into **skUSD** to receive the protocol's realized yield as the book distributes it. The published APY is a live model at target leverage, computed from the trailing Hyperliquid funding mean plus the Lido staking SMA with explicit deductions for strategy cost, dynamic insurance allocation, and protocol phase fee; it is not a record of realized distributions, and realized yield to date is small. Genesis phase: 0% performance fee while TVL is below $100k.

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
| OP Atlas project registry | https://atlas.optimism.io/project/0x8f7746724eb8314f2fcbeb9367dabc25dd159cb1dddecac5f7d61bc67a708113 |

## Contracts on Base (chain 8453)

| Contract | Address |
|---|---|
| kUSD (synthetic dollar) | [`0x5C2EfdF0D8D286959b42308966bc2B97f5680AA3`](https://basescan.org/address/0x5C2EfdF0D8D286959b42308966bc2B97f5680AA3) |
| skUSD (staked kUSD, ERC-4626) | [`0x96F5102C15b839757f811A98CEc3725Ac21DfA14`](https://basescan.org/address/0x96F5102C15b839757f811A98CEc3725Ac21DfA14) |
| KUSDPSM (USDC to kUSD mint; holds MINTER_ROLE) | [`0xaBDE1138aa1Ce88d1dF06422C0c3b05D70569803`](https://basescan.org/address/0xaBDE1138aa1Ce88d1dF06422C0c3b05D70569803) |
| KerneVault v2 (ERC-4626; holds MINTER_ROLE) | [`0x8ccc56B5624e2FDB592F6609d81F4c3798e3292B`](https://basescan.org/address/0x8ccc56B5624e2FDB592F6609d81F4c3798e3292B) |
| KERNE (governance, canonical v2) | [`0x230f3a63E8413D42bEe9103b98a204030206186c`](https://basescan.org/address/0x230f3a63E8413D42bEe9103b98a204030206186c) |
| 2-of-3 Safe (protocol admin) | [`0x52d3E450bA6c299B1B07298F1E87DD74732D4877`](https://basescan.org/address/0x52d3E450bA6c299B1B07298F1E87DD74732D4877) |

The full address registry is in [`contracts-public/deployments/8453.json`](https://github.com/kerne-protocol/contracts-public/blob/main/deployments/8453.json). The live KUSDPSM above is the 2026-07-10 redeploy, built from the frozen external-audit commit so the deployed bytecode matches the source under audit; the prior mint PSM (`0x07eBb486...`, 2026-06-16 ceremony) had MINTER_ROLE revoked and is temporarily retained as a kUSD-to-USDC redeem reserve while its USDC reserve migrates, with the Proof of Reserves summing all three PSM reserves through the window. KerneVault v2 was deployed in the 2026-06-16 ceremony; the earlier KUSDPSM (`0xFf3025ec...`) remains a redeem reserve, and the prior KerneVault (`0x8005bc7A...`) is retired. The original KERNE deployment (`0xfEA3D217...`) was retired and superseded by the canonical v2 above; see [kerne.fi/security](https://kerne.fi/security) for the disclosure. The skUSD above (`0x96F5102C...`) is the 2026-07-03 redeploy that reset a distorted share price back to par; the prior skUSD (`0xdEd74F7E...`) is retired and holds only residual dust.

## Verify Kerne yourself

A single command checks every public claim Kerne makes about its own state, against live RPCs and live HTTPS endpoints, with no authentication:

```bash
curl -sL https://raw.githubusercontent.com/kerne-protocol/contracts-public/main/scripts/verify_public_endpoints.sh | bash
```

Prefer not to pipe the internet to bash? Read the script first, or follow the full hostile-reader walkthrough (every claim cross-checked with `cast call`) in [`contracts-public/HOW_TO_VERIFY_KERNE.md`](https://github.com/kerne-protocol/contracts-public/blob/main/HOW_TO_VERIFY_KERNE.md).

## Audit posture

- **Tests you can run:** 69 tests across 14 suites in [`contracts-public`](https://github.com/kerne-protocol/contracts-public), green from a clean clone in two commands, with no RPC endpoint, API key or environment file, and [run in CI on every push](https://github.com/kerne-protocol/contracts-public/actions). Eight of them name the outside researcher who reported the finding they lock down.

  ```bash
  git clone --recurse-submodules https://github.com/kerne-protocol/contracts-public
  cd contracts-public && forge test
  ```

- **Tests you cannot run:** the private monorepo holds 2,516 Solidity test functions across 137 test files (measured 2026-07-31) plus Python (bot) and TypeScript (SDK) suites, and a drift-guard CI job that asserts every numeric threshold cited in the docs matches the live constant in code. We list that number for completeness, not as evidence: you cannot execute it, so weigh the 69 above instead. Earlier revisions of this page said "900+ Solidity tests" and left it at that, which was true and understated but gave a reader nothing to check.
- **External:** Kerne has engaged Hexens for its first external smart-contract audit (scope: kUSD, skUSD, KUSDPSM, KerneVault, esKERNE, the five contracts named in the report's own scope section). Hexens fieldwork ran from 2026-07-13 and Hexens delivered its initial report on 2026-07-20; remediation is underway and the final report is still pending. Initial-report contents are confidential and no result is claimed here, so the code is not yet through a completed external audit and should be treated as unaudited until the final report is public. The public bug bounty is live at [kerne.fi/security](https://kerne.fi/security), and internal adversarial audit reports are published at [kerne.fi/security/audits](https://kerne.fi/security/audits). External reports land in [`contracts-public/audits/`](https://github.com/kerne-protocol/contracts-public/tree/main/audits) as they arrive.
- **Verification:** every registry contract is source-verified on BaseScan and/or Sourcify except KerneStaking and KerneFlashArbBot, which are disclosed per-contract in the [contracts-public status table](https://github.com/kerne-protocol/contracts-public#where-the-contract-source-is). The live skUSD (`0x96F5102C`, 2026-07-03 redeploy) is source-verified on BaseScan (2026-07-10) and Sourcify. The live KUSDPSM (`0xaBDE1138`, 2026-07-10 redeploy) is Sourcify-verified with an exact creation and runtime match; BaseScan native verification is pending. KerneVault v2 (2026-06-16 ceremony) was source-verified on BaseScan and Sourcify 2026-06-17. The full forge-testable source mirror has landed: `contracts-public` builds and runs its suite from a clean clone as of 2026-07-31. It mirrors the explorer-verified sources byte for byte, so it reproduces behaviour rather than every deployed bytecode byte; per-address bytecode equality is what BaseScan and Sourcify already attest, and each bundle records the settings it was verified under.

## How Kerne is built

Kerne is built by three co-founders, and most of the code is written with AI under our direction. The commit trailers record it. We are stating it here because a fact like this should come from us rather than turn up later as something a reader discovered on their own.

Code written this way earns more outside scrutiny, not less, and that is the reason the rest of this page looks the way it does. The contracts went to Hexens for a paid external audit on the terms and with the limits set out above. A public bug bounty runs at [kerne.fi/security](https://kerne.fi/security), hosted by us rather than on a third-party platform. Reserves are signed hourly, risk triggers are published as a live feed, every contract has a verification status you can check against BaseScan and Sourcify yourself, and every place the deployed bytecode differs from its source is written down. All of that reads off public RPCs and public endpoints, so none of it asks you to take a view on who or what wrote the code.

## Community

- X / Twitter: [@KerneProtocol](https://x.com/KerneProtocol)
- Discord: [discord.gg/Xx8TSuWrCA](https://discord.gg/Xx8TSuWrCA)
- Security disclosure: kerne.systems@protonmail.com (PGP key in security.txt)
