# Preliminary Review — tokenize.it

**Project:** tokenize.it  
**Date:** March 4th, 2026  
**Status:** Preliminary Review (Pre-Audit)

---

## EXECUTIVE SUMMARY

A preliminary review was conducted over the tokenize.it protocol scope: `Token`, `AllowList`, `FeeSettings`, `Crowdinvesting`, `TokenSwap`, `PrivateOffer`, `Vesting`, `PriceLinear`, and the associated clone factories. The codebase is modular and follows established patterns (ERC2771, clone factories, CREATE2 for PrivateOffer). Our review identified several potential issues across **Medium**, **Low**, and **Info** severities—including **3 preliminary findings at Medium severity**, the details of which are disclosed in the full report or upon engagement. A full report with detailed findings and recommendations is available under separate cover.

---

## CORE TECHNICAL QUESTIONS

**Oracle revert propagation:** When `Crowdinvesting` uses a dynamic price oracle (e.g. `PriceLinear`), how does the protocol behave if the oracle reverts—for example during a mandatory cool-down after parameter updates? Is the resulting purchase blackout intentional, and how do integrators surface this to users?

**Fee-on-transfer and trusted currency:** The AllowList marks currencies as `TRUSTED_CURRENCY` for use in `Crowdinvesting` and `TokenSwap`. Does the protocol assume that trusted currencies are never fee-on-transfer (or rebasing), and where is this assumption documented or enforced? What happens to `currencyReceiver` payouts if such a token is allowlisted by mistake?

**Vesting and minting allowance:** For mintable vestings, `release()` depends on the token's `mintingAllowance` for the vesting contract. How does the protocol ensure that this allowance is never reduced below the vesting's outstanding allocation, and what guarantees exist for beneficiaries if a token admin changes allowances after vesting creation?

---

## PRELIMINARY FINDINGS

**1 High-severity and 2 Medium-severity findings** were identified in this preliminary review. Their titles, impact, and remediation guidance are disclosed in the full report or upon engagement.

A subset of lower-severity items is summarized below:

| ID   | Title                                                                 | Severity | Impact                                                                 |
|------|-----------------------------------------------------------------------|----------|------------------------------------------------------------------------|
| M-01 | `PriceLinear.getPrice()` reverts during cool-down period — purchases blocked | Medium   | The oracle owner can repeatedly call `updateParameters` to censor purchases. |
| L-01 | Unpause delay constant vs error message                               | Low      | Documentation says "one day", code enforces one hour.                 |
| L-02 | `TokenSwap.sell()` rounding for small amounts                         | Low      | Sellers can receive zero currency while transferring tokens.           |

*Additional High, Medium, Low and Info findings are included in the full report.*

---

## THE VALUE PROPOSITION

Our preliminary review uncovered **12+ potential edge cases** related to fee accounting, timing (cool-down, expiration inclusivity), and integration assumptions (AllowList, factory trust windows). A full engagement would include:

- **Traceability of trust assumptions:** Explicit mapping of AllowList and fee-on-transfer assumptions across Crowdinvesting, TokenSwap, and PrivateOffer.
- **Oracle and pricing robustness:** Verification of PriceLinear cool-down semantics and of fallback or documentation for purchase blackout windows.
- **Vesting and Token alignment:** Proof that minting-allowance lifecycle cannot break mintable vesting guarantees, or clear documentation and operational procedures.
- **Fix verification:** Direct collaboration with your team to validate remediations and document residual risks.

**Next steps:** Let's schedule a 15-minute technical walkthrough of these findings.

[[X](https://x.com/midgarxyz) / [Website](https://www.midgaraudits.xyz/)]

midgaraudits@gmail.com

