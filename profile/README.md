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
| KerneVault v2 (ERC-4626, WETH; deposits shut on chain) | [`0x8ccc56B5624e2FDB592F6609d81F4c3798e3292B`](https://basescan.org/address/0x8ccc56B5624e2FDB592F6609d81F4c3798e3292B) |
| KERNE (governance, canonical v2) | [`0x230f3a63E8413D42bEe9103b98a204030206186c`](https://basescan.org/address/0x230f3a63E8413D42bEe9103b98a204030206186c) |
| 2-of-3 Safe (admin of skUSD and KerneVault; proposer/executor/canceller on the timelock) | [`0x52d3E450bA6c299B1B07298F1E87DD74732D4877`](https://basescan.org/address/0x52d3E450bA6c299B1B07298F1E87DD74732D4877) |
| TimelockController, 48h (admin of kUSD and all three PSM modules) | [`0x36A14976980B7Dd33136f6613545EB0A2C0a0D72`](https://basescan.org/address/0x36A14976980B7Dd33136f6613545EB0A2C0a0D72) |
| PauseGuardian (emergency stop, outside the delay) | [`0xC47adaf51907bB1871D07E18eA21dc75Ae93Cc8E`](https://basescan.org/address/0xC47adaf51907bB1871D07E18eA21dc75Ae93Cc8E) |

**Custody, scoped rather than summarised.** On 2026-08-06 the Safe handed `DEFAULT_ADMIN_ROLE` over kUSD and all three PSM modules to the TimelockController above, in one atomic batch ([`0xed24d495...d9b8`](https://basescan.org/tx/0xed24d495789540dae543793a5d8c9d884d8abfc386c0e726084da9183933d9b8), block 49623130), so changes to those four contracts now wait 48 hours in public. **That is not the same as "custody is timelocked", and this page will not say so:** the Safe still holds instant `DEFAULT_ADMIN_ROLE` on skUSD and on KerneVault v2, and skUSD alone holds roughly 88% of kUSD supply. Two earlier revisions of this table described KerneVault v2 as holding `MINTER_ROLE` and the Safe as "protocol admin"; the first stopped being true on 2026-08-03 when that grant was revoked (the live PSM is now the only minter) and the second on 2026-08-06. Check any cell with `cast call <contract> "hasRole(bytes32,address)(bool)" <role> <holder> --rpc-url https://mainnet.base.org`.

The full address registry is in [`contracts-public/deployments/8453.json`](https://github.com/kerne-protocol/contracts-public/blob/main/deployments/8453.json). The live KUSDPSM above is the 2026-07-10 redeploy, built from the frozen external-audit commit so the deployed bytecode matches the source under audit; the prior mint PSM (`0x07eBb486...`, 2026-06-16 ceremony) had MINTER_ROLE revoked and is temporarily retained as a kUSD-to-USDC redeem reserve while its USDC reserve migrates, with the Proof of Reserves summing all three PSM reserves through the window. KerneVault v2 was deployed in the 2026-06-16 ceremony; the earlier KUSDPSM (`0xFf3025ec...`) remains a redeem reserve, and the prior KerneVault (`0x8005bc7A...`) is retired. The original KERNE deployment (`0xfEA3D217...`) was retired and superseded by the canonical v2 above; see [kerne.fi/security](https://kerne.fi/security) for the disclosure. The skUSD above (`0x96F5102C...`) is the 2026-07-03 redeploy that reset a distorted share price back to par; the prior skUSD (`0xdEd74F7E...`) is retired and holds only residual dust.

## Verify Kerne yourself

A single command checks every public claim Kerne makes about its own state, against live RPCs and live HTTPS endpoints, with no authentication:

```bash
curl -sL https://raw.githubusercontent.com/kerne-protocol/contracts-public/main/scripts/verify_public_endpoints.sh | bash
```

Prefer not to pipe the internet to bash? Read the script first, or follow the full hostile-reader walkthrough (every claim cross-checked with `cast call`) in [`contracts-public/HOW_TO_VERIFY_KERNE.md`](https://github.com/kerne-protocol/contracts-public/blob/main/HOW_TO_VERIFY_KERNE.md).

## Measurement tools anyone can run

Three standalone repositories. No dependencies, no API key, no account anywhere. They measure the rest of the category rather than only Kerne, and the board the first one checks lists Kerne's own row first by rule and states its own gap in full, whatever that gap is on the day. That is the reason to hand you the tool rather than the summary.

| Repository | What it does |
|---|---|
| [**realized-apy**](https://github.com/kerne-protocol/realized-apy) | Re-derives every row of the [Kerne Honesty Index](https://kerne.fi/honesty-index) from public archive RPC, at the block heights the signed snapshot names, and tells you row by row whether the published number holds. It is a **second implementation**: it shares no code with the engine that writes the board, and it verifies the snapshot signature with keccak256 and secp256k1 written from scratch rather than with a library. It also measures any ERC-4626 dollar vault Kerne has never heard of. Method, tolerances and the standing commitment if you get a different answer: [kerne.fi/honesty-index/reproduce](https://kerne.fi/honesty-index/reproduce). |
| [**solhonesty**](https://github.com/kerne-protocol/solhonesty) | The same question asked on Solana, across Kamino, Jupiter Lend and Save, published as a [CC BY 4.0 dataset](https://huggingface.co/datasets/kerne-protocol/honesty-index) and rendered at [kerne.fi/solana-honesty](https://kerne.fi/solana-honesty). Kerne is deliberately **absent** from that board, because the board is Solana and Kerne is on Base, so any Kerne row there would be invented. |
| [**signed-por**](https://github.com/kerne-protocol/signed-por) | A vendor-neutral verifier for Signed Proof-of-Reserves attestations: EIP-191 recovery, canonical number binding, freshness with a future-skew bound. Kerne is reference deployment number one rather than the subject, and the [spec](https://github.com/kerne-protocol/signed-por/blob/main/SPEC.md) is written so anyone else can be number two. |

```bash
npx -y github:kerne-protocol/realized-apy check
```

Every protocol on the Honesty Index, including the ones the board is least flattering about, has a free, permanent and unedited [right of reply](https://kerne.fi/honesty-index/reproduce#right-of-reply) on its own row. Nothing is charged for it and no commercial conversation is a condition of it.

## Audit posture

- **Tests you can run:** 78 tests across 15 suites in [`contracts-public`](https://github.com/kerne-protocol/contracts-public) (measured 2026-08-14), green from a clean clone in two commands, with no RPC endpoint, API key or environment file, and [run in CI on every push](https://github.com/kerne-protocol/contracts-public/actions). Eight of those suites name the outside researcher who reported the finding they lock down.

  ```bash
  git clone --recurse-submodules https://github.com/kerne-protocol/contracts-public
  cd contracts-public && forge test
  ```

- **Tests you cannot run:** the private monorepo holds 2,516 Solidity test functions across 137 test files (measured 2026-07-31) plus Python (bot) and TypeScript (SDK) suites, and a drift-guard CI job that asserts every numeric threshold cited in the docs matches the live constant in code. We list that number for completeness, not as evidence: you cannot execute it, so weigh the 78 above instead. Earlier revisions of this page said "900+ Solidity tests" and left it at that, which was true and understated but gave a reader nothing to check.
- **External:** Kerne's first external smart-contract audit is complete, and the report is public. Hexens reviewed five contracts (kUSD, skUSD, KUSDPSM, KerneVault, esKERNE) at commit `0912c870`; fieldwork ran from 2026-07-13, and the final report was published in full on 2026-07-31. Result: **0 critical, 2 high, 2 medium, 4 low, 2 informational**, ten findings in total, eight fixed and two acknowledged by design. All ten sit in `KerneVault.sol`. Lead researcher: Trung Dinh. Read the report unedited at [`audits/hexens-kerne-protocol-final-2026-07-31.pdf`](https://github.com/kerne-protocol/contracts-public/blob/main/audits/hexens-kerne-protocol-final-2026-07-31.pdf) or on Hexens' own site at [hexens.io/audit-reports/kerne-protocol-july-2026](https://hexens.io/audit-reports/kerne-protocol-july-2026), and our written response to every finding, including both acknowledged ones, at [kerne.fi/insights/hexens-audit-every-finding-and-our-response](https://kerne.fi/insights/hexens-audit-every-finding-and-our-response).
- **What that audit does not cover, stated here rather than in a footnote:** it reviewed *source at one commit*, not the deployed bytecode. The live KerneVault runs bytecode built from commit `ecc95cf7` (2026-06-15), roughly three weeks earlier than the reviewed commit, so **all ten findings are live on the deployed vault**. That is why vault deposits were closed on chain on 2026-07-30 by a 2-of-3 Safe call and stay closed: the standing rule is remediated bytecode before capital, not before announcement. The vault holds no user funds, `totalSupply()` is 0, and no third party has ever held a share; withdrawals were deliberately left open. Every place deployed bytecode differs from current source is written down per contract, with the operating rule that bounds it, in [`audits/DEPLOYED_VS_SOURCE.md`](https://github.com/kerne-protocol/contracts-public/blob/main/audits/DEPLOYED_VS_SOURCE.md) and at [kerne.fi/security/deployed-vs-source](https://kerne.fi/security/deployed-vs-source). One audit of five contracts at a single commit is not a track record.
- **Ongoing:** the public bug bounty is live at [kerne.fi/security](https://kerne.fi/security), and internal adversarial audit reports are published at [kerne.fi/security/audits](https://kerne.fi/security/audits). Further external reports land in [`contracts-public/audits/`](https://github.com/kerne-protocol/contracts-public/tree/main/audits) as they arrive.
- **Verification:** every registry contract is source-verified on BaseScan and/or Sourcify except KerneStaking and KerneFlashArbBot, which are disclosed per-contract in the [contracts-public status table](https://github.com/kerne-protocol/contracts-public#where-the-contract-source-is). The live skUSD (`0x96F5102C`, 2026-07-03 redeploy) is source-verified on BaseScan (2026-07-10) and Sourcify. The live KUSDPSM (`0xaBDE1138`, 2026-07-10 redeploy) is Sourcify-verified with an exact creation and runtime match; BaseScan native verification is pending. KerneVault v2 (2026-06-16 ceremony) was source-verified on BaseScan and Sourcify 2026-06-17. The full forge-testable source mirror has landed: `contracts-public` builds and runs its suite from a clean clone as of 2026-07-31. It mirrors the explorer-verified sources byte for byte, so it reproduces behaviour rather than every deployed bytecode byte; per-address bytecode equality is what BaseScan and Sourcify already attest, and each bundle records the settings it was verified under.

## How Kerne is built

Kerne is built by three co-founders, and most of the code is written with AI under our direction. Commit trailers record part of that history rather than all of it: 14 of the 82 commits across Kerne's public repositories carry a co-authorship trailer (measured 2026-08-13). The convention started partway through and nothing enforces it, so read the trailers as a sample rather than a census. We are stating it here because a fact like this should come from us rather than turn up later as something a reader discovered on their own.

Code written this way earns more outside scrutiny, not less, and that is the reason the rest of this page looks the way it does. The contracts went to Hexens for a paid external audit on the terms and with the limits set out above. A public bug bounty runs at [kerne.fi/security](https://kerne.fi/security), hosted by us rather than on a third-party platform. Reserves are signed hourly, risk triggers are published as a live feed, every contract has a verification status you can check against BaseScan and Sourcify yourself, and every place the deployed bytecode differs from its source is written down. All of that reads off public RPCs and public endpoints, so none of it asks you to take a view on who or what wrote the code.

## Community

- X / Twitter: [@KerneProtocol](https://x.com/KerneProtocol)
- Discord: [discord.gg/Xx8TSuWrCA](https://discord.gg/Xx8TSuWrCA)
- Security disclosure: kerne.systems@protonmail.com (PGP key in security.txt)
