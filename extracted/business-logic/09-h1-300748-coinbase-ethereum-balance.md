# Failed contract transfer still credited as deposit (Coinbase ETH balance inflation)

- **Vuln type / sub-variant:** Business logic, crediting on an unreverted internal transaction (balance manipulation via reverted smart contract execution)
- **Target type:** Crypto exchange deposit/credit pipeline (ETH receiving code for contract-originated transfers)
- **Recon signal:** The researchers examined how the exchange handles ETH arriving from a smart contract rather than a plain wallet. Contract-originated transfers have internal transactions that can be rolled back, which a naive deposit watcher may not account for.
- **Root cause:** The exchange credited a user balance when it observed an internal transfer to the user's deposit address, but did not check that the enclosing contract execution actually succeeded. When the outer transaction reverted, the chain undid the transfers, yet the exchange ledger kept the credit. (Source is partial: only the vendor and researcher summaries are public, comment bodies are not rendered.)
- **Exploitation steps:**
  1. Deploy a contract that forwards ether to several valid exchange deposit addresses.
  2. Add a final recipient that always throws on receipt, so the whole call reverts at the last step.
  3. Fund the contract and execute it. The chain reverts everything, so no ether really leaves the contract.
  4. The exchange still shows the earlier internal transfers as credited to the account (the on-chain address looks empty while the exchange balance shows funds).
  5. Repeat to grow the balance, then withdraw to an external wallet.
- **Bypasses used:** None needed. The trick is relying on EVM revert semantics versus the exchange's off-chain bookkeeping.
- **Impact:** Arbitrary inflation of an exchange balance leading to cash-out of funds that never existed. Rated Critical. Vendor stated it found only accidental loss and no exploitation attempts, and fixed it within hours by changing contract handling logic.
- **Bounty:** Awarded, amount hidden (disclosed Mar 2018).
- **Lesson / pattern:** When ledger credit is derived from observed events rather than final state, test whether failed, reverted, or partially applied upstream operations still trigger the credit.
- **Source:**
  - Title: Ethereum account balance manipulation (Coinbase)
  - Author: vicompany
  - URL: https://hackerone.com/reports/300748
  - Date: 2017-12-27 (disclosed 2018-03-21)
  - Disclosed report: https://hackerone.com/reports/300748
  - Raw file: `sources/business-logic/09-h1-300748-coinbase-ethereum-balance.md`

Tags: balance-manipulation, financial-loss, assumption-flaw, order-of-operations, missing-server-validation
