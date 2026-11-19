# usememos/memos: server never verifies it issued the JWT, so a forged token takes over any account

- **Vuln type / sub-variant:** improper JWT handling / broken access control (CWE-284); forged JWT accepted because issuance/signature is not verified, leading to account takeover
- **Target type:** usememos/memos self-hosted note app (Golang), access token as JWT (memos v0.12.2)
- **Recon signal:** the app authorizes actions from a JWT access token; test whether the server actually validates that it issued and signed the token versus trusting the decoded claims
- **Root cause:** the system does not verify whether the JWT was issued by the server (signature/issuer not validated), so a token edited client-side (e.g. via token.dev) is accepted as valid
- **Exploitation steps:**
  1. Log in and capture your own JWT access token.
  2. Edit the token's claims at token.dev to point at another user (change the subject/user identity).
  3. Send a request with the forged token.
  4. The server accepts it without verifying issuance, letting you modify the target user's email, password, and other personal data.
- **Bypasses used:** claim tampering on an unsigned/unverified JWT (no valid signature needed because the server does not check)
- **Impact:** full account takeover of any user (change their password/email), privacy violations, data breach; rated Critical (9.8), CVE-2023-4696
- **Bounty:** not stated (huntr; fixed upstream)
- **Lesson / pattern:** decoding a JWT is not verifying it; the server must validate the signature against its own key and reject tokens it did not issue. A token whose claims can be changed and still accepted is a direct account-takeover primitive, not an information leak.
- **Source:**
  - Title: Account TakeOver Due to Improper Handling of JWT Tokens in usememos/memos
  - Author: M Nadeem Qazi (@mnqazi)
  - URL: https://huntr.com/bounties/4747a485-77c3-4bb5-aab0-21253ef303ca
  - Date: 2023-04-20
  - Disclosed report: CVE-2023-4696
  - Raw file: `sources/access-control/24-huntr-jwt-handling-ato.md`

Tags: jwt, signature-not-verified, forged-token, account-takeover, broken-access-control, cwe-284, cve
