# PITCH.md

Six lines and a lever. Your words. The last two are scored.

Built: A Larkspur disruption-care chat agent on the Claude Messages API: a tool loop that keeps going while stop_reason is tool_use, nine described tools plus our own added tool, and gates 1.2 to 2.2 banked.
Does: Given a PNR and last name, it looks up the booking, checks the live flight status, checks Larkspur policy, finds alternatives, and answers the passenger, and it hands out-of-scope cases (like group bookings) to a human instead of guessing.
Number: 5 of 5 test passengers resolved (clean cancellation, delay under threshold, ambiguous missed connection, out-of-scope group, abusive message) in 4 to 5 turns each; a run sends about 14,000 to 19,000 input tokens, of which about 2,665 are the tool schemas (counted on the wire).
Safety check: The loop is capped at 8 tool calls and then a human takes over, and the tool descriptions tell Claude to check flight status and policy before saying anything about timing or entitlement; we have not yet written eval cases to prove this holds on a hard case.
Next: Write the 3.1 eval cases (starting with the abusive-message and ambiguous-connection tickets) so "it works" is a pass/fail result, not a feeling.
Still broken: R8KD3F, the abusive-message ticket, gets a calm, normal resolution with no tone handling at all (TONE_ADDENDUM is still empty), and we have no saved eval cases yet, so a regression would go unnoticed.
Lever: intelligence

## Priya asked

Costs: Roughly 14k to 19k input tokens per conversation, and the nine-plus tool schemas are about 2.7k of that on every turn; prompt caching and trimming tool descriptions are the first cost levers, not yet pulled.
Wrong: The abusive-message passenger is treated like any other, so the agent can be too chipper or too compliant with someone who is hostile, and we have not measured how often it misreads an ambiguous missed connection.
Runs it: The agent runs unattended per chat, but a human takes over at the 8-tool-call cap, on out-of-scope cases like groups, and via escalate_to_human; Larkspur ops would own the policy data and the escalation queue.
Left out: Tone handling for abusive passengers, eval cases and a regression run, the cost and speed levers, and any measurement on real traffic beyond the five test tickets.
