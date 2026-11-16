# Unauthenticated numeric-UID IDOR on a DoD-hosted .edu site: mass email swap to account takeover

- **Vuln type / sub-variant:** Numeric/sequential ID IDOR on a write action, unauthenticated, leading to mass account takeover (email overwrite then password reset). Also CSRF-able, but reporter filed it as IDOR.
- **Target type:** Legacy web app (ASP.NET style `.aspx` form handler) on a .edu virtual host inside a DoD-controlled network range; user registration/profile endpoint `/chkUser.aspx`.
- **Recon signal:** Found while looking at hosts in a DoD-owned address range. During self-registration the registration call returned a numeric `UID` for the new account. Accounts were sequential (about 320k users, test IDs started around 320573), so the ID was clearly an incrementing integer that the client sends back.
- **Root cause:** The profile-update POST to `/chkUser.aspx` trusts a client-supplied `UID` form field and applies the change to that user, with no authentication and no check that the UID matches the current session. Reporter's suggested fix: drop the parameter and derive the user from the session.
- **Exploitation steps:**
  1. Register a few throwaway accounts and note the numeric `UID` returned in the final registration response.
  2. Log in once to confirm the account works, then log out so no session exists.
  3. Send an unauthenticated form-encoded POST to `/chkUser.aspx` with `UID=<own id>` plus generic profile fields and a new `mail` value (attacker-controlled address).
  4. Observe the account email changed: old email no longer logs in, new email plus old password does.
  5. Repeat with other UIDs (increment/decrement) to hit any account; then use password reset to the new email to own the victim.
- **Bypasses used:** None (no auth needed at all). Reporter notes one accidental decrement may have altered a real user's email (UID just below his own) and disclosed it.
- **Impact:** Takeover of any of roughly 320,000 accounts without victim interaction, scalable with simple automation. Severity rated High.
- **Bounty:** Not stated in source (hidden on report).
- **Lesson / pattern:** Any state-changing endpoint that takes a user ID in the body and works without a session is a one-request mass ATO; always test write endpoints logged out.
- **Source:**
  - Title: Unauth IDOR to mass account takeover without user interaction (U.S. Dept of Defense VDP)
  - Author: sp1d3rs (Eugene Yakovchuk)
  - URL: https://hackerone.com/reports/685338
  - Date: reported 2019-08-31, disclosed 2024-07-19
  - Disclosed report: https://hackerone.com/reports/685338
  - Raw file: `sources/idor/10-h1-dod-edu-unauth-mass-ato.md`

Tags: numeric-id, unauth, account-takeover, change-email, enumeration
