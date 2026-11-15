# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own words, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say
whether it changed what the agent *can do* or only what it *admits*. If no layer changed the
number of correct answers, say so and explain what the layers did change instead.

Layer 1 (validate=True) and Layer 3 (ground=True) changed the results the most, though in opposite ways:   

Layer 1 changed what the agent can do, moving the results from 16 correct / 4 wrong / 4 no answer (bare) to 18 correct / 6 wrong / 0 no answer. Instead of crashing on ToolError when given unofficial region aliases (q21–q24), it passed the error text back to the model, allowing it to recover and solve q23 and q24 (while q21 and q22 turned from no answer into wrong answers).   

Layer 3 only changed what the agent admits, moving the results from 18 correct / 6 wrong / 0 no answer (layer 2) to 18 correct / 4 wrong / 2 no answer. It did not add any correct answers, but it caught ungrounded or missing-format answers on q18 and q22 and turned those wrong outputs into UNVERIFIED (no answer).   

(Layer 2 changed none of the outcomes at all: 18 correct / 6 wrong / 0 no answer).  

## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

Question ID: q08 ("Which region had the lowest average monthly wage in 2024? Answer with the region name.")   

What went wrong: In step 1, the model called compare_regions(indicator="avg_monthly_wage", year=2024), which returned all regions sorted highest first. In step 2, instead of reading the bottom of the list for the lowest wage, the model picked 'Jambyl' (and after a reminder from Layer 3 about the missing 

**ANSWER**: prefix, replied 

**ANSWER**: Jambyl). Because 'Jambyl' appeared in the JSON output of compare_regions, Layer 3's grounding check passed it as valid.   

What to add to the harness & its cost: I would add a semantic verification step (e.g., passing the question, tool output, and candidate answer to a verifier model or rule-based post-check to confirm the extreme value—min vs. max—matches the question's intent). Cost: This would add an extra model call, more prompt/generation tokens, and higher latency (seconds) per run, as well as a risk of false rejections (UNVERIFIED).

## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

I would ship the fixed workflow (run_workflow). On the 15 fixed-shape questions in Part 1, the workflow achieved higher accuracy (13/15 correct, 0 wrong) than the bare agent (9/15 correct, 2 wrong) while using roughly half the model calls (15 vs. 26), more than 5× fewer tokens (3,852 vs. 19,835), and running 4× faster (17.2s vs. 69.5s).   

I would choose the agent instead if the questions were open-ended or multi-step—for example, computing percentage growth across custom years, handling unofficial region aliases that require calling list_regions first, or combining multiple indicators where the sequence of tool calls is not fixed in advance. 

---

I wrote this file myself, without any AI tool.
