---
title: "AI Unit Economics"
description: "How to model the real per-task cost of agentic AI development: cost per PR, token decomposition, cost-reduction levers, and the break-even point of an autonomous agent"
tags: [cost, ops, guide]
---

# AI Unit Economics

> **Audience**: Tech leads, platform engineers, and engineering managers who need to reason about what agentic AI actually costs per unit of work, not just what shows up on the monthly invoice.
>
> **Scope**: The economic reasoning behind agentic development cost. For the technical implementation of budgets and per-team enforcement, see [api-gateway.md](./api-gateway.md). For per-session logging and cost estimation, see [observability.md](./observability.md).

---

## A note on what this page is

This page is mostly original synthesis, not a collection of field reports. Across the practitioner sources this guide draws from (conference talks, podcasts, and meetups covering agentic AI and software development), no one models the real cost of agentic AI at scale in any depth. That absence is itself a finding: teams adopt agents, then discover the cost structure after the fact.

So this page reasons from first principles about the cost model, and pulls in a handful of attributable practitioner observations only where they support a specific point. Every attributed claim names its source. Where a number is illustrative rather than measured, it says so. Nothing here is a benchmark, and no figure is presented as an established fact unless it is sourced.

---

## 1. Why agentic cost is not SaaS cost

Classic SaaS pricing is a fixed cost per seat. You pay the same whether a user runs one report or a hundred. Capacity planning is a headcount question, and the finance team can forecast next quarter by counting licenses.

Agentic AI breaks that model in three ways.

**The cost is variable per task, not fixed per seat.** A developer with a Claude Code license does not cost a flat amount. One day they close a two-line fix that costs a few cents in tokens. The next day they run a multi-agent refactor across forty files that costs several dollars. The seat is not the unit of cost. The task is.

**The cost of the same task varies run to run.** Ask an agent to fix the same failing test twice and you can get two different token totals. The model may re-read a file it already saw, take an extra reasoning step, or call one more tool. Variance is inherent, not a defect. It means you budget against a distribution, not a fixed line item.

**Failed work still costs money.** A run that loops without converging, re-reads the same large file five times, or explores a dead end burns input and output tokens the whole way. In classic SaaS a failed action is free. Here, a task that produces nothing can cost more than one that succeeds, because failure often means more iterations, not fewer. This is the part that surprises teams: the invoice grows even when the output does not.

The framing that captures the stakes came out of an enterprise AI transformation account: a token is intelligence and real value, and the work is turning tokens into industrial ROI (*The Product Crew, AI transformation field report, 2026, in the context of Ask For The Moon*). Restated for this page: every token you spend is a bet that the work it produces is worth more than the token cost. Unit economics is the discipline of checking whether that bet pays off, task by task.

---

## 2. Building a cost per task

To manage cost per task, you first have to measure it. A single agentic task decomposes into a small number of billable components. Here is the full list for a Claude Code run:

| Component | What it is | Typical driver |
|-----------|-----------|----------------|
| Input tokens | Everything sent to the model: prompt, files read, tool results, conversation history | Context size, number of file reads |
| Output tokens | Everything the model generates: reasoning, code, explanations | Task complexity, verbosity |
| Cache writes | Context written to the prompt cache on first use | First read of a large stable file |
| Cache reads | Context served from cache instead of full input price | Repeated reads of the same context |
| Tool calls | Each tool invocation adds its result to the next input | Number of Read, Bash, Grep, WebFetch calls |
| Sub-agent runs | Each sub-agent is its own input plus output cost | Number and depth of delegated tasks |

Output tokens dominate the bill on most models because output is priced several times higher than input. At the cost rates this guide uses for illustration (see [observability.md §Cost Rates](./observability.md#cost-rates)), Sonnet output is $15 per million tokens against $3 for input, and Opus output is $75 against $15. A task that generates a lot of code or long reasoning is expensive on the output side even if its input was small.

### A concrete calculation

Take a mid-sized pull request: an agent reads eight files, reasons about a change, edits four of them, and runs the test suite twice. A rough token budget:

```
Input side:
  8 file reads          ~40,000 tokens (first read, priced as input or cache write)
  2 test-run outputs      ~8,000 tokens (tool results fed back as input)
  reasoning re-reads     ~20,000 tokens (context resent on each turn)
  system + instructions   ~5,000 tokens
  -----------------------------------
  input subtotal         ~73,000 tokens

Output side:
  reasoning              ~12,000 tokens
  4 file edits            ~6,000 tokens
  explanations            ~2,000 tokens
  -----------------------------------
  output subtotal        ~20,000 tokens
```

At Sonnet illustrative rates ($3 input, $15 output per million):

```
input:  73,000 / 1,000,000 x $3   = $0.219
output: 20,000 / 1,000,000 x $15  = $0.300
                                    -------
                        task total ~ $0.52
```

The same PR run on Opus (illustrative $15 input, $75 output) lands near $2.60, roughly five times more. The numbers above are illustrative to show the method, not a measured benchmark. The point is the shape: measure the components, price them, and you have a cost per task you can compare across models and across approaches.

### A minimal cost function

Expressed as pseudo-code, the per-task cost is a sum over turns and sub-agents:

```
cost_per_task =
    sum over each model turn:
        input_tokens  * input_rate
      + output_tokens * output_rate
      + cache_write_tokens * cache_write_rate
      + cache_read_tokens  * cache_read_rate
  + sum over each sub_agent:
        cost_per_task(sub_agent)   # recursive: sub-agents have their own turns
```

Cache reads are priced far below fresh input (a fraction of the input rate), which is why a stable, reused context is cheaper than one you rebuild every turn. That single fact drives most of the levers in the next section.

The raw token counts for a real session live in the JSONL files at `~/.claude/projects/<project>/`. Tools like `ccusage` (see [observability.md §External Monitoring Tools](./observability.md#external-monitoring-tools)) read those files and produce exact per-session costs, so you do not have to estimate by hand once you want real numbers.

---

## 3. The real cost-reduction levers

Four levers move the per-task cost meaningfully. None of them require a pricing negotiation. They are engineering decisions.

### Route by complexity

Not every task needs the most capable model. A prompt that formats a file, renames a symbol, or answers a lookup does not need Opus-level reasoning. Routing simple prompts to a cheaper model and reserving the expensive one for genuinely ambiguous work cuts the bill without cutting quality where quality matters.

One production account put a number on it: routing prompts by complexity to cheaper models cut the production cost by roughly half in the observed case (*Antonio Goncalves, [IFTTD ep 357](https://www.ifttd.io/)*). Half is that team's result, not a universal constant, but the direction generalizes. The gap between Haiku, Sonnet, and Opus output rates is large enough that moving even a third of your traffic down a tier changes the monthly total.

### Isolate heavy work in sub-agents

Pushing a heavy task into a sub-agent with its own clean context, which returns only a synthesis to the supervisor, keeps the main context from filling with intermediate detail (*Dev With AI Meetup, multiple speakers, 2026*). The economic effect is direct: the supervisor's context stays small, so every subsequent turn on the main thread resends fewer tokens. Without isolation, the intermediate output of a heavy task pollutes the main context and inflates the input cost of every turn that follows, for no added value.

The trade-off is that a sub-agent has its own input and output cost. Isolation pays off when the heavy task would otherwise bloat a long main thread. It does not pay off for a task so small that spawning a sub-agent costs more than the pollution it avoids.

### Cap iterations and set explicit exit criteria

The most avoidable cost is a loop that does not converge. An agent that keeps trying, re-reading the same files, and re-reasoning over the same context can spend several dollars producing nothing. Two guards prevent it: an explicit definition of done, so the agent knows when to stop, and a hard maximum on iterations, so a stuck run fails fast instead of burning tokens indefinitely. A run that hits the iteration cap and stops has a bounded cost. A run without a cap does not.

### Reuse cached context

Because cache reads are priced far below fresh input, a stable context that the model reads repeatedly is much cheaper than one rebuilt from scratch each turn. Keeping the large, stable parts of the context (project instructions, reference files, schemas) in a form the cache can serve turns repeated reads from a full-price input into a fraction of it. This compounds over a long session.

---

## 4. The break-even point of an autonomous agent

The appealing pitch for an autonomous agent is that it runs around the clock and never rests. The economic question is whether the work it produces around the clock is worth more than what it costs to run around the clock. There is a volume and complexity threshold below which a continuously running agent costs more than it returns.

Two forces set that threshold. Above it, an agent handling a high volume of well-scoped tasks earns its keep, because the alternative is a lot of human time on repetitive work. Below it, an agent left running on ambiguous or low-volume work spends tokens on exploration and re-work that a human would have short-circuited.

A concrete failure signal comes from a team that delegated to agents continuously without supervision and lost velocity as a result (*Dev With AI Meetup, Gallet & Dahan, 2026*). The lost velocity is a hidden cost that never appears on the token invoice. Time spent reviewing, correcting, or unwinding agent output is a real cost of the agent, even though it is paid in engineer hours rather than dollars per million tokens. Read that account as a warning sign, not a rule: it shows that unsupervised continuous delegation can push a team below the break-even line, where the total cost (tokens plus human cleanup) exceeds the value produced.

The infrastructure around a high-volume automated client is its own cost line. A client generating 10,000 to 50,000 operations per day needs capacity planning and rate limiting to stay within provider limits (*ByteByteGo, 2025*). At that volume, the cost of the surrounding infrastructure (queues, retries, rate-limit handling, monitoring) is added on top of the inference cost. Break-even is not only about tokens. It is about tokens plus the systems that keep a high-throughput agent inside its limits, plus the human time spent supervising it.

A practical way to reason about the threshold: estimate the fully loaded cost of an agent on a class of tasks (tokens plus infrastructure plus expected human cleanup), and compare it against the cost of the same work done by a human or a simpler non-agentic pipeline. If the agent is cheaper only when you ignore the cleanup, you are below break-even.

---

## 5. Budget and governance per team

Once you can compute a cost per task, you can set budgets against it. The economic reasoning belongs here; the technical enforcement belongs in [api-gateway.md](./api-gateway.md).

The reasoning: a per-team budget turns an unbounded variable cost into a bounded one. If the backend team's agentic work runs around a known cost per task and a known task volume, a monthly cap sized to that product gives a hard ceiling. When the cap is hit, further calls are refused rather than silently added to the invoice. That converts a surprise at invoice time into a signal during the month.

The mechanics of setting virtual keys, per-team budgets, model allowlists, and usage dashboards are covered in detail in [api-gateway.md](./api-gateway.md): virtual keys carry team metadata, a budget cap returns a `429` when exceeded, and a model allowlist stops a cheap-task team from calling the most expensive model by accident. This page stays on the reasoning; that page is the implementation.

The one economic point worth stating here: budgets should be sized against measured cost per task, not guessed. A cap set too low blocks legitimate work and pushes people to route around it. A cap set with no basis in measured cost is theater. Measure first (§2), then set the cap.

---

## 6. What this page does not cover yet

This page reasons about the cost model. It deliberately does not include:

- **A proprietary benchmark.** There is no measured dataset of cost-per-PR across teams here. The numbers in §2 are illustrative to teach the method, not results.
- **A provider pricing comparison.** Per-token prices change often enough that a comparison table would be stale within a release or two. For current rates, check the provider's pricing page and the live figures that tools like `ccusage` read from your own sessions.
- **A cost-per-outcome model.** Tying token cost to business value (revenue per feature, defects avoided) is the harder half of ROI and is out of scope here. This page stops at cost per task; connecting cost to value is a separate exercise per team.

If you have measured cost-per-task data from your own work, it is exactly the kind of field report that would turn parts of this synthesis into evidence. Until then, treat this as a reasoning framework to apply, not a set of numbers to trust.

---

## See Also

- [api-gateway.md](./api-gateway.md) for the technical implementation of budgets, virtual keys, and per-team caps
- [observability.md](./observability.md) for per-session cost estimation and the tools that read exact token counts
- [practitioner-insights.md](../ecosystem/practitioner-insights.md) for the field reports referenced here in full context
