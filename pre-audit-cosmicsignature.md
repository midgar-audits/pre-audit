# Preliminary Review — CosmicSignature Protocol

**Project:** CosmicSignature  
**Date:** February 23, 2026  
**Status:** Preliminary Review (Pre-Audit)
**Repo:** https://github.com/PredictionExplorer/Cosmic-Signature/tree/main
---

## EXECUTIVE SUMMARY

A preliminary review was conducted over the CosmicSignature protocol scope: `CosmicSignatureGame`, `Bidding`, `MainPrize`, `PrizesWallet`, `StakingWalletCosmicSignatureNft`, `StakingWalletRandomWalkNft`, `RandomWalkNFT`, `CosmicSignatureToken`, `CosmicSignatureNft`, `MarketingWallet`, `CharityWallet`, `CosmicSignatureDao`, `DonatedTokenHolder`, and supporting base contracts. The codebase is modular with a deep inheritance chain and follows upgradeable patterns (UUPS, storage gaps). Our review identified several potential issues across Medium, Low, and Info severities—including **3 preliminary findings at Medium severity**, the details of which are disclosed in the full report or upon engagement. A full report with detailed findings and recommendations is available under separate cover.

---

## CORE TECHNICAL QUESTIONS

**Prize distribution and balance snapshot:** All main, secondary, and charity ETH amounts in `_distributePrizes()` are derived from `address(this).balance` via percentage getters. How does the protocol ensure that this balance reflects only the intended sources (e.g. bids) at claim time, and what happens if the balance is altered—for example by donations or front-running—immediately before or during `claimMainPrize()`? Is there an explicit invariant that prize percentages apply to a single, fixed snapshot?

**Charity wallet trigger and governance:** `CharityWallet` sends ETH to a configurable `charityAddress`. The interface documents that the send entrypoint is intentionally callable by anyone. How does the protocol coordinate timing and batching of charity payouts with the owner’s intent, and what operational or governance safeguards exist if any actor can trigger a full sweep at any time?

**Legacy NFT and external calls:** `RandomWalkNFT` is noted in comments as already deployed and immutable. It returns excess ETH to the minter via a low-level call during `mint()`. How do integrators (including `CosmicSignatureGame`) account for reentrancy or state assumptions when calling or depending on this contract, and where is this documented for downstream consumers?

---

## PRELIMINARY FINDINGS

**3 Medium-severity findings** were identified in this preliminary review. One is summarized below as an example; the remaining two (with full impact and remediation) are disclosed in the full report or upon engagement.

### Example: [M-01] ETH prize distribution uses live balance

All main, secondary, and charity prize amounts are computed as percentages of `address(this).balance` at claim time (`MainPrize.getMainEthPrizeAmount()`, `getCharityEthDonationAmount()`, `SecondaryPrizes.getChronoWarriorEthPrizeAmount()`, etc.). The same contract holds ETH from both bids and from `donateEth()` / `donateEthWithInfo()` (EthDonations). There is no snapshot: the balance used is whatever is in the contract when `claimMainPrize()` runs.

**Impact:** Donations (or a front-run donation before claim) inflate the balance and thus all prize payouts beyond what bidders contributed. If percentages sum close to 100%, the contract may attempt to send more ETH than available and revert, locking the round.

**Recommendation:** Snapshot `address(this).balance` once at the start of `_distributePrizes()` and use that value for all prize calculations instead of calling the getters (which read the live balance).

---

A subset of lower-severity items is summarized below:

| ID   | Title                                                                 | Severity | Impact                                                                 |
|------|-----------------------------------------------------------------------|----------|------------------------------------------------------------------------|
| L-01 | `halveEthDutchAuctionEndingBidPrice` — divisor doubling overflow      | Low      | Repeated calls without reset can overflow; auction floor inverts.      |
| L-02 | `StakingWalletCosmicSignatureNft.deposit()` when `numStakedNfts == 0` | Low      | Division-by-zero caught in `_distributePrizes`; staking ETH retained in game. |
| I-01 | `blockhash(block.number)` in entropy                                 | Info     | Always zero; NFT seed entropy weaker than intended.                    |
| I-04 | `EthDonations.donateEth()` zero-value                                | Info     | No minimum; zero-value donations accepted and logged.                  |

Additional Medium, Low, and Info findings are included in the full report.

---

## THE VALUE PROPOSITION

Our preliminary review uncovered **17+ potential edge cases** related to prize math (balance snapshot, percentage ordering), access control and governance (charity trigger, CST/prize flows), and integration with legacy or external contracts (RandomWalkNFT, DonatedTokenHolder). A full engagement would include:

- **Prize and balance invariants:** Explicit verification that all prize and charity amounts are computed from a consistent snapshot and that donations or front-running cannot distort payouts or lock the round.
- **Charity and wallet design:** Clear documentation or code changes so that permissionless vs owner-only behavior is intentional and residual governance risk is documented.
- **Legacy and external integration:** Documented assumptions and mitigations for RandomWalkNFT and other non-upgradeable components used by the game.
- **Fix verification:** Direct collaboration with your team to validate remediations and document residual risks.

**Next steps:** Let's schedule a 15-minute technical walkthrough of these findings.

[[X](https://x.com/midgarxyz) / [Website](https://www.midgaraudits.xyz/)]

midgaraudits@gmail.com

