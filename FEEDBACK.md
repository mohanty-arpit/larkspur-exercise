# Client review: Larkspur disruption-care agent

**To:** Room 15  
**From:** Larkspur client review agent, on behalf of Priya Raghavan  
**Re:** the disruption-care agent you walked us through in our last session  
**Generated:** 2026-11-19 05:03

## Priya's note

> Our vendor says we should just be using your best model.
>
> Why aren't we?
>
> Priya Raghavan, Larkspur Airlines

She sent that before this session opened. She means it. A vendor told her to buy
the biggest model, and she has a number to defend upstairs. Her four questions from
day one are still open. Naming a model answers none of them.

## Still open from day one

| Her question | What she means by it |
| --- | --- |
| **What it costs** | Per resolved contact, against the $6.90 a human contact costs us. |
| **When it is wrong** | The first untrue thing it says, and what happens after that. |
| **Who runs it** | In June, after you have left. |
| **What you left out** | The scope you cut, and why. |

## What the review agent found

Larkspur pointed a review agent at your repository. It read the
code. It did not run your agent, and the only file it changed is this one. Each
item below names the file and the line it is about.

**1. agent.py search_alternatives description grew from "search" to 1329 characters, and it is 72% of the schema text you send on every turn.**

Your diff replaced the one-word description with a long block covering when to use, when not to use, and rules. The other eight descriptions run 71 to 445 characters. hold_seat is 73 characters and send_confirmation is 71. PITCH.md puts the schemas at about 2,665 input tokens per turn, and this one description is the bulk of it. Nothing in the repository shows what the 1329 characters changed in behavior.

Run python3 run.py --tool-tax and paste the per-tool token counts, then run python3 bench.py --label before --stage 1 --runs 3 with the old "search" description and paste python3 bench.py --compare before after.

**2. agent.py run_agent loop: your diff now appends response.content to messages, so thinking blocks go back to the API on every continued turn.**

The shipped line appended text_of(response), which is a string. You replaced it with response.content, and the call carries thinking={"type": "adaptive"}. That is the right shape for tool_use continuation. readout-trace.json shows 4 API turns, 4 tool calls and "thinking seen on the wire: True", so one run did go round the loop. It is one run on one booking.

Run python3 run.py --all --trace and paste the totals footer, so the turn count is visible for all five booking shapes.

**3. agent.py MAX_TOOL_CALLS is 8, but the loop condition is turns < MAX_TOOL_CALLS and turns starts at 1.**

The loop exits when turns reaches 8, which allows 7 continuation calls, so 8 API turns. PITCH.md says "capped at 8 tool calls and then a human takes over". The loop in agent.py ends by returning text_of(response), and no branch calls escalate_to_human when the cap is hit. At the cap the final response may still be stop_reason tool_use with no text block, and text_of returns an empty string.

Run python3 verify.py 1.2 and paste its output, then paste what run_agent returns when MAX_TOOL_CALLS is set to 2 on K7PQ2M.

**4. There is no evals/cases.json, and your PITCH.md number "5 of 5 resolved" has nothing in the repository under it.**

The scan finds no eval cases. "Resolved" in 4 to 5 turns is the count you wrote, and the only committed run evidence is readout-trace.json: 4 turns, one booking. TONE_ADDENDUM is 0 characters, EXTRA_TOOLS is 0, and LOCAL_TOOLS has no executors, so "plus our own added tool" in the Built line matches nothing in the diff except mcp_client.tools() appended in tool_list().

Run python3 run.py --show-tools and paste the tool names, so the added tool in PITCH.md can be matched to a name.

**5. The same loop that picked search_alternatives will run on any model, and readout-trace.json shows 0 cache reads against 15244 input tokens.**

Four tools were called in order: lookup_booking, get_flight_status, check_policy, search_alternatives. The scan finds no cache_control in agent.py, so the system prompt and 9 schemas are re-sent uncached each of the 4 turns. A model change leaves the 1329-character description, the 8-call cap and the empty TONE_ADDENDUM exactly as they are. The cost and the abusive-message gap in your own PITCH.md would come out the same under a different model name.

Run python3 bench.py --label after --stage 1 --runs 3 and paste python3 bench.py --compare before after, so Priya sees a measured delta in tokens and wall clock for a change you made.

## Your four answers

Four of the lines in your PITCH.md are answers to me rather than to your
verifier, and somebody on your side wrote them between our sessions. I read
those beside the code, not instead of it. Where an answer is carrying a number,
I have said whether the repository backs it.

| My question | Your answer | My read |
| --- | --- | --- |
| **What it costs** | Roughly 14k to 19k input tokens per conversation, and the nine-plus tool schemas are about 2.7k of that on every turn; prompt caching and trimming tool descriptions are the first cost levers, not yet pulled. | **Answered.** "About 2.7k" of schema per turn matches the 2,665 in PITCH.md, and readout-trace.json backs the 14k to 19k range with 15244 in. The "first cost levers, not yet pulled" claim holds: the scan finds no cache_control, and a bench-after.json comparison would give the measured before and after. |
| **When it is wrong** | The abusive-message passenger is treated like any other, so the agent can be too chipper or too compliant with someone who is hostile, and we have not measured how often it misreads an ambiguous missed connection. | **Thin.** "Can be too chipper or too compliant" is a prediction with no case behind it, since there is no evals/cases.json. "Not measured" for the ambiguous missed connection is honest, but the abusive ticket R8KD3F has a trace you could have quoted. |
| **Who runs it** | The agent runs unattended per chat, but a human takes over at the 8-tool-call cap, on out-of-scope cases like groups, and via escalate_to_human; Larkspur ops would own the policy data and the escalation queue. | **Not supported by the repository.** "A human takes over at the 8-tool-call cap" is not in agent.py: run_agent just returns text_of(response) when the loop exits, with no escalate_to_human call at the cap. The ops ownership of policy data and the queue has no file under it. |
| **What you left out** | Tone handling for abusive passengers, eval cases and a regression run, the cost and speed levers, and any measurement on real traffic beyond the five test tickets. | **Answered.** Each item matches the repository: TONE_ADDENDUM at 0 characters, no evals/cases.json, caching absent. "Any measurement beyond the five test tickets" is fair, though the only saved run is the single trace in readout-trace.json. |

All four answered. Bring the artifact behind each one to our next meeting, not the sentence.

## Before our next meeting

> Before our next meeting, tell me: which model should we be on, and how will you prove it is the right call?
>
> Priya Raghavan, Larkspur Airlines

Bring two things. A recommendation, and the measurement behind it. If the model is
not the problem, say so, and bring the number that shows it.

## What this review read

- `agent.py (251 lines)`
- `PITCH.md`
- `TEAM.md`
- `readout-trace.json`
- `PITCH.md (4 of 4 answers to Priya)`
- given files that differ from the shipped pack: `verify.py`, `eval_harness.py`, `bench.py`, `readout.py`, `setup.py`, `pod_sync.py`, `run.py`, `review.py`, `support/__init__.py`, `support/client.py`, `support/data.py`, `support/models.py`, `support/tools.py`, `support/trace.py`, `support/offline.py`, `support/mock_backend.py`, `support/mcp_client.py`, `support/mcp_server.py`, `support/mcp_selftest.py`, `support/access.py`, `support/keyscan.py`, `support/review.py`

Reviewer: `claude-sonnet-5-5`. Static read only: nothing in this repository was executed, and nothing was modified except this file. Larkspur Airlines is a fictional training scenario. Confidential, do not distribute.
