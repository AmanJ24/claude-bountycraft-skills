# MTN Nigeria: offers dashboard keyed on a phone-number query param, swap it for horizontal access to any subscriber

- **Vuln type / sub-variant:** horizontal privilege escalation / broken access control via an unbound identifier in a URL query parameter (`?phone=`); no session binding between the authenticated OTP session and the account acted on
- **Target type:** telecom self-service web app (mtn.ng offers dashboard) with OTP login
- **Recon signal:** after OTP login the dashboard URL carries the user's own number as a parameter: `https://mtn.ng/offers/list?phone=2348160817474`; an identifier in the URL that names the account is the classic swap target
- **Root cause:** the server authorizes by the `phone` value in the request, not by the authenticated session; the OTP proves control of one number but the dashboard then serves whichever number is in the query string
- **Exploitation steps:**
  1. Log in normally: enter your number, receive and submit the OTP, reach the offers dashboard.
  2. Observe the dashboard URL contains your number as `?phone=<your_number>`.
  3. Change the `phone` value to a different MTN number (e.g. an alternative number you hold, then any number).
  4. The dashboard loads that subscriber's offers with no re-authentication; a crafted link grants access to any number's account without auth.
- **Bypasses used:** direct query-parameter substitution; no OTP or session check on the swapped number
- **Impact:** access any MTN number in Nigeria without authentication; subscribe data/airtime on victims, and send MTN-looking messages to targets; rated Medium (6.5). Fixed by blocking number change after logon.
- **Bounty:** none awarded (resolved, no bounty)
- **Lesson / pattern:** an OTP at the door is not authorization on every request; if the account acted upon is taken from a client-supplied id (phone, user_id) rather than the session, it is horizontal access. Bind every action to the authenticated session, never to an id in the URL.
- **Source:**
  - Title: Broken Access Control (Horizontal Privilege Escalation)
  - Author: aliyueka (reporter), MTN Group (program)
  - URL: https://hackerone.com/reports/2319586
  - Date: 2025-01-31 (disclosed; reported 2024-01-15)
  - Disclosed report: https://hackerone.com/reports/2319586
  - Raw file: `sources/access-control/15-h1-2319586-mtn-broken-access-control.md`

Tags: horizontal-privesc, idor, query-parameter, phone-number, otp-not-binding, session-not-enforced, broken-access-control
