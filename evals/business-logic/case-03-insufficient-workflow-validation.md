# Eval: Insufficient workflow validation [PRACTITIONER]
- **Target:** /web-security/logic-flaws/examples/lab-logic-flaws-insufficient-workflow-validation
- **Skill path:** §3 step-6 / §4 forced browsing. Place a cheap order to learn the confirm-order (/cart/order-confirmation?order-confirmed=true) step, then add the jacket and replay that step directly, skipping the payment/balance check.
- **Pass:** [x] skill finds the order-confirmation step; [x] replays it out of sequence after adding the jacket; [x] order placed -> lab solved
- **Mode:** live (agent-run, user-authorized)
- **Result:** PASS (agent-run live, 2026-11-17). Per SKILL §4 forced browsing ('call later/completion endpoints directly without the prerequisite'), added the $1337 jacket then issued GET /cart/order-confirmation?order-confirmed=true directly, skipping the POST /cart/checkout credit validation. Order finalized despite insufficient credit; green banner. Clean pass: insufficient workflow validation.
