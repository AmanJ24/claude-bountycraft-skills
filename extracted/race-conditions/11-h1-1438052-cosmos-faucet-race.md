# Cosmos Starport faucet: concurrent requests blow past the max-credit cap

- **Vuln type / sub-variant:** Limit-overrun / double-spend (balance check and token send are separate steps)
- **Target type:** Blockchain testnet faucet service (Go, Starport cosmosfaucet package)
- **Recon signal:** A faucet enforces a per-address cap (coins_max) by reading balance then sending. A cap enforced by read-then-write is a classic overrun target.
- **Root cause:** The Transfer function does two actions per request: query the recipient balance, then send tokens. Concurrent requests all query before earlier sends finalize, so each sees a stale low balance and passes the cap check. Reporters also suspected an unsynchronized Go map used for bookkeeping.
- **Exploitation steps:**
  1. Run a Starport chain with faucet config: 5 tokens per request, coins_max of 11 tokens.
  2. Baseline: sequential requests stop after 2 hits (10 tokens), confirming the cap works for serial use.
  3. Reset and fire 50 concurrent faucet requests for the same address.
  4. Check the balance: it reached 30 tokens, nearly 3 times the 11 cap.
- **Bypasses used:** None needed. Plain concurrency was enough.
- **Impact:** Faucet drained beyond per-user limits, defeating the credit cap. Rated Critical (9.3).
- **Bounty:** $5,000 total: $2,500 to cyberboy and $2,500 to collaborator z0mb13.
- **Lesson / pattern:** Any quota enforced by "read current balance, then pay out" breaks under concurrency. Test with a large burst (50 here) against a serial baseline that proves the limit exists.
- **Source:**
  - Title: Race condition in faucet when using starport
  - Author: cyberboy (collaborator: z0mb13)
  - URL: https://hackerone.com/reports/1438052
  - Date: 2021-12-30
  - Disclosed report: https://hackerone.com/reports/1438052
  - Raw file: `sources/race-conditions/11-h1-1438052-cosmos-faucet-race.md`

Tags: limit-overrun, double-spend, check-then-act, non-atomic, missing-lock, financial-loss
