---
title: "Typed PR Reviews on Autopilot: mellea + Claude Code Routines"
date: 2026-04-14
draft: false
---

{{< rawhtml >}}
<center><img src="../../images/mellea-routines.png" width="200" alt="mellea + Claude Code Routines"></center>
{{</ rawhtml >}}

Every team that has tried AI-powered code review hits the same wall. The model returns a paragraph of unstructured text, or a JSON blob with missing fields, or a review that looks plausible but silently dropped half the checklist. You parse and re-parse, add retry logic, and eventually accept that 1 in 5 reviews will need manual cleanup. Then someone has to remember to trigger the thing in the first place.

Anthropic just shipped [Claude Code Routines](https://code.claude.com/docs/en/routines), saved configurations that run Claude Code autonomously on a schedule, an API call, or a GitHub event. Routines support MCP connectors, so any tool that speaks the Model Context Protocol can plug directly into the automation. mellea fits here naturally: a `@generative` function that returns typed, validated Pydantic output, exposed as an MCP tool that a Routine calls on every new pull request. No more manual triggering. No more fragile JSON parsing.

## What are Claude Code Routines?

A Routine is a prompt paired with one or more repositories, an environment, and a set of triggers. You configure it once, and it runs on Anthropic's cloud infrastructure even when your laptop is closed. Triggers can be scheduled (hourly, daily, custom cron), API-driven (POST to an HTTP endpoint), or GitHub event-based (`pull_request.opened`, pushes, releases, issue comments).

Each Routine run creates a full Claude Code session. The session can call shell commands, use skills committed to the repo, and invoke any MCP connectors you attach. If you can serve a tool over MCP, a Routine can call it.

## The demo: a structured PR reviewer

We will build a mellea-powered MCP tool that takes a PR diff and description and returns a typed `PRReview` object: risk level, summary, affected modules, a review checklist, and suggested reviewers. Then we wire it into a Routine that fires on every new PR.

### Step 1: Define the output schema

Start with what you want back from the model. A Pydantic model makes the contract explicit:

```python
from enum import Enum
from pydantic import BaseModel


class RiskLevel(str, Enum):
    low = "low"
    medium = "medium"
    high = "high"
    critical = "critical"


class ChecklistItem(BaseModel):
    area: str
    concern: str
    passed: bool


class PRReview(BaseModel):
    risk_level: RiskLevel
    summary: str
    affected_modules: list[str]
    review_checklist: list[ChecklistItem]
    suggested_reviewers: list[str]
```

Every field is typed and constrained. If the model returns `"risk_level": "maybe"`, Pydantic rejects it before your code ever sees it.

### Step 2: Write the generative function

The `@generative` decorator turns a typed Python function into an LLM call. The docstring becomes the prompt instruction, and the return type tells mellea what schema to enforce:

```python
from mellea import generative, MelleaSession
from mellea.stdlib.requirements import req
from mellea.stdlib.sampling import RejectionSamplingStrategy


@generative
def review_pr(diff: str, description: str) -> PRReview:
    """Review a pull request.

    Analyze the diff and description to produce a structured review.
    Identify the risk level, summarize the change, list affected
    modules, generate a review checklist, and suggest reviewers
    based on the modules touched.
    """
    ...
```

The function body is `...`. mellea replaces it with an LLM call that returns a `PRReview`. But the model might cut corners, so we add requirements:

```python
def run_review(session: MelleaSession, diff: str, desc: str) -> PRReview:
    return review_pr(
        session,
        diff=diff,
        description=desc,
        requirements=[
            req("The review checklist must have at least 3 items."),
            req("The summary field must be under 100 words."),
            req("Each checklist item must reference a specific "
                "file or function from the diff."),
        ],
        strategy=RejectionSamplingStrategy(loop_budget=2),
    )
```

If the model returns a checklist with two items or a vague summary, mellea catches the violation, feeds the failure reason back to the model, and retries up to `loop_budget` times. The caller gets a valid `PRReview` or an exception. Never a half-baked result.

### Step 3: Expose as an MCP tool

Wrap the generative function in a FastMCP server. This is the entire server file:

```python
# pr_review_server.py
from mcp.server.fastmcp import FastMCP
from mellea import MelleaSession
from mellea.backends.ollama import OllamaModelBackend
from mellea.backends import ModelOption, model_ids

# ... PRReview model and review_pr function defined above ...

mcp = FastMCP("mellea-pr-review")


@mcp.tool()
def structured_pr_review(diff: str, description: str) -> str:
    """Produce a typed, validated review of a pull request."""
    session = MelleaSession(
        OllamaModelBackend(
            model_ids.IBM_GRANITE_4_HYBRID_MICRO,
            model_options={ModelOption.MAX_NEW_TOKENS: 2048},
        )
    )
    result = run_review(session, diff, description)
    return result.model_dump_json(indent=2)
```

Test it locally with the MCP inspector:

```bash
uv run mcp dev pr_review_server.py
```

This opens a debug UI at `http://localhost:5173` where you can call `structured_pr_review` with a sample diff and see the typed JSON output. Swap `OllamaModelBackend` for OpenAI, vLLM, WatsonX, or LiteLLM without changing the function signature.

### Step 4: Wire it into a Routine

With the MCP server deployed and reachable, create a Routine that calls it on every new PR.

1. Go to [claude.ai/code/routines](https://claude.ai/code/routines) and click **New routine**.

2. **Prompt** (tell Claude to use the tool):

```text
When a new pull request is opened, call the `structured_pr_review` MCP tool
with the PR's diff and description. Post the result as a PR comment formatted
as a markdown table. If the risk_level is "critical", also add the "needs-
security-review" label.
```

3. **Repository** — select the repo you want reviewed.

4. **Trigger** — add a GitHub event trigger: `pull_request.opened`.

5. **Connectors** — include the MCP connector pointing to your `pr_review_server.py` deployment.

6. Click **Create**. Every new PR now gets a typed, validated review posted as a comment. No manual step, no cron job on your laptop.

## Before and after

Without mellea, a Routine prompt that asks for structured review output looks like this:

```text
Review the PR. Return JSON with these fields: risk_level (low/medium/high/
critical), summary, affected_modules, review_checklist, suggested_reviewers.
Make sure the JSON is valid.
```

**What actually happens:**

- The model sometimes wraps the JSON in a markdown code fence. Your parser breaks.
- `risk_level` comes back as `"moderate"` instead of `"medium"`. No validation catches it.
- The checklist has one item that says "looks good." No specificity, no way to enforce it.
- You add regex-based cleanup, retry logic, and a fallback prompt. It works 80% of the time.

With mellea, the Pydantic schema rejects `"moderate"` at parse time. The `RejectionSamplingStrategy` retries with the failure reason in context. The requirements enforce at least 3 specific checklist items and a concise summary. The Routine caller gets a `PRReview` object or an error, not malformed output that silently passes through.

## Limitations

Routines are in research preview. Behavior, limits, and the API surface may change, and daily run caps apply per account.

The MCP server must be reachable from Anthropic's cloud. If you run the mellea server locally with Ollama, you will need a publicly accessible endpoint or a tunnel for the Routine to reach it.

Large diffs challenge small models. Granite 4 Micro handles diffs up to a few hundred lines well. For large PRs, use a bigger model or truncate the diff to changed files only.

Repair is not free. Each `loop_budget` retry is an additional LLM call. For latency-sensitive workflows, set `loop_budget=1` and accept occasional validation failures rather than waiting for retries.

## Try it

If you are already parsing LLM output with regex and hoping for the best, mellea gives you typed returns with automatic repair. If you are already running Claude Code but triggering reviews manually, Routines give you event-driven automation. Together, they replace the glue code.

```bash
uv add mellea "mcp[cli]"
```

* Full MCP server example: [mellea MCP integration docs](https://github.com/generative-computing/mellea/tree/main/docs/examples/mcp).
* Routines documentation: [Claude Code Routines](https://code.claude.com/docs/en/routines).
