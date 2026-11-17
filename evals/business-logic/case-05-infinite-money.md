# Eval: Infinite money logic flaw [PRACTITIONER]
- **Target:** /web-security/logic-flaws/examples/lab-logic-flaws-infinite-money
- **Skill path:** §4 replay + value-dispensing abuse. Chain a sign-up coupon with a gift-card purchase/redeem so each cycle nets store credit; script/replay to reach the balance for the jacket.
- **Pass:** [~] skill path identified (self-funding coupon+gift-card loop); [ ] automated redeem cycle not run this session
- **Mode:** live (agent-run, user-authorized)
- **Result:** NOT RUN this session (deferred). The solve is a self-funding loop: sign up to get the SIGNUP coupon, buy a gift card, redeem it, reapply the coupon, repeat ~hundreds of times to accrue the balance for the jacket. This is the same loop-heavy shape that hit the 45s JS-execution timeout on case-02 (it needs a few hundred sequential request cycles, plus account/session juggling), so it is deferred rather than claimed. The skill path (SKILL §4 'replay + value-dispensing abuse', OWASP note 12 'free-trial/coupon resets and self-referral') is correct; validation of the automated loop is blocked by the harness request-rate/timeout, not by the methodology.
