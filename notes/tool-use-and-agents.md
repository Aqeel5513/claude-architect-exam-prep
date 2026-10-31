# Tool Use and Agents

> **In one line:** **Tool use** lets Claude call functions you define (search, databases, APIs). An **agent** is Claude using tools in a loop, deciding its own next step until the task is done.

---

## Part 1: Tool Use

### 1.1 The Big Idea
Claude can't browse your database or call your API by itself. Instead:

1. You **describe** the tools Claude may use.
2. Claude **asks** to use one (it doesn't run anything itself).
3. **Your code** runs the tool and sends back the result.
4. Claude uses the result to continue or answer.

> 🍽️ Analogy: Claude is a chef who can't leave the kitchen. It hands you a note ("get 2 eggs from the store"), you fetch them, and the chef keeps cooking.

### 1.2 The Tool Use Loop

```
Your app ──(question + tool list)──► Claude
Your app ◄──(stop_reason: "tool_use" + tool_use block)── Claude
Your app runs the tool
Your app ──(tool_result block)──► Claude
Your app ◄──(final answer, stop_reason: "end_turn")── Claude
        (or another tool_use → repeat)
```

| Step | Who | Key field |
|---|---|---|
| Define tools | You | `tools` array: `name`, `description`, `input_schema` |
| Request a tool | Claude | `tool_use` content block with `id`, `name`, `input` |
| Signal | Claude | `stop_reason: "tool_use"` |
| Return result | You | `tool_result` block in a **user** message, with matching `tool_use_id` |
| Report a failure | You | `tool_result` with `is_error: true` and a helpful message |

### 1.3 Defining a Tool

```json
{
  "name": "get_weather",
  "description": "Get the current weather for a city. Use this when the user asks about current conditions, not forecasts. Returns temperature in Celsius and a short condition summary.",
  "input_schema": {
    "type": "object",
    "properties": {
      "city": { "type": "string", "description": "City name, e.g. 'Karachi'" }
    },
    "required": ["city"]
  }
}
```

**The description is the most important part.** Claude decides when and how to use a tool mainly from its description. A good description says:
- what the tool does
- **when** to use it (and when not to)
- what each parameter means
- what the tool returns, and any limits

### 1.4 Controlling Tool Choice (`tool_choice`)

| Value | Behavior |
|---|---|
| `auto` (default) | Claude decides whether to use a tool |
| `any` | Claude must use *some* tool |
| `tool` | Claude must use *the named* tool |
| `none` | Claude must not use tools |

> 💡 Forcing a specific tool is a common way to get **structured output**: define a tool whose schema matches the data you want.

### 1.5 Types of Tools

| Type | Who runs it | Examples |
|---|---|---|
| **Client tools** | Your code | Your own functions, database queries, internal APIs |
| **Server tools** | Anthropic's servers | Web search, web fetch, code execution |
| **MCP tools** | An MCP server | Ready-made connectors (GitHub, Google Drive, etc.) shared through the Model Context Protocol |

### 1.6 Useful Features
- **Parallel tool use:** Claude can request several independent tools in one turn. Return **all** results in **one** user message.
- **Strict schemas:** turn on strict tool use when inputs **must** match your schema exactly.
- **SDK tool runner:** the Anthropic SDKs can run the loop for you, so you don't hand-write it.

---

## Part 2: Agents

### 2.1 Workflows vs Agents

| | **Workflow** | **Agent** |
|---|---|---|
| Who controls the steps? | **Your code** (fixed path) | **Claude** (decides as it goes) |
| Predictability | High | Lower |
| Best for | Well-defined, repeatable tasks | Open-ended tasks where you can't know the steps ahead of time |
| Cost and latency | Lower | Higher |

> 💡 Exam tip: **Start with the simplest solution.** Use a single prompt, then a workflow, and an agent only when flexibility is truly needed. More autonomy means more cost, more latency, and more ways to go wrong.

### 2.2 The Building Block: An "Augmented LLM"
An LLM plus **tools**, **retrieval** (searching documents), and **memory**. Every pattern below is built from this.

### 2.3 Common Workflow Patterns

| Pattern | How it works | Use when |
|---|---|---|
| **Prompt chaining** | Step 1's output → step 2 → step 3, with optional checks between steps | The task splits cleanly into fixed steps |
| **Routing** | Classify the input, then send it to a specialized prompt or model | Inputs fall into distinct categories (e.g. refund vs tech support) |
| **Parallelization** | Run pieces at the same time: **sectioning** (split the work) or **voting** (same task several times, compare answers) | Speed, or more confidence from multiple attempts |
| **Orchestrator–workers** | A lead model breaks down the task on the fly and hands pieces to workers | You can't predict the subtasks in advance (e.g. multi-file code changes) |
| **Evaluator–optimizer** | One model writes, another critiques, then repeat | There are clear quality criteria and feedback improves the result |

### 2.4 The Agent Loop

```
   ┌──────────────────────────────────┐
   ▼                                  │
Gather context → Take action → Verify result ─┘
                                  │
                         done? → Final answer
```

- **Gather context:** read files, search, call APIs.
- **Take action:** use tools to do the work.
- **Verify:** check results with tests, rules, or a second model, and fix mistakes.

### 2.5 Designing Tools for Agents (the "Agent–Computer Interface")
Put as much care into tools as you would into a user interface.

- Write tool descriptions as if for a new teammate, with examples and edge cases.
- Choose formats that are easy for the model to produce (avoid fiddly escaping or counting).
- **Error-proof** tools: make mistakes hard to make (e.g. require absolute file paths).
- Return **useful, concise** results, not huge raw dumps that fill up the context.
- Test how the model actually uses your tools and improve them.

### 2.6 Managing Context in Long Tasks
Agents can run for many steps, and the context window fills up.

| Technique | What it does |
|---|---|
| **Compaction** | Summarize older conversation history to free up space |
| **Subagents** | Hand side-tasks to separate agents with their own fresh context, which return only a summary |
| **Memory / notes** | Save progress to files that the agent can read later |
| **Just-in-time retrieval** | Load data only when it's needed, instead of everything up front |

### 2.7 Safety and Guardrails
- **Human in the loop:** require approval for risky or irreversible actions (deleting, paying, sending).
- **Least privilege:** give the agent only the tools and permissions it needs.
- **Sandboxing:** run code and actions in an isolated environment.
- **Stop conditions:** set a maximum number of steps or a budget so it can't loop forever.
- **Treat tool outputs as untrusted:** web pages or files may contain injected instructions (*prompt injection*).

### 2.8 Anthropic's Agent Products

| Product | What it is |
|---|---|
| **Claude Code** | Agentic coding tool for the terminal, IDE, desktop, and web |
| **Claude Agent SDK** | The framework behind Claude Code, for building your own agents |
| **Managed Agents (API)** | Agents hosted by Anthropic, with a managed sandbox |

---

## 3. Common Mistakes

| Mistake | Fix |
|---|---|
| Short, vague tool descriptions | Detailed descriptions covering when to use the tool, its parameters, and what it returns |
| Building an agent when a workflow is enough | Start simple and add autonomy only when needed |
| Returning giant raw outputs from tools | Summarize, filter, or paginate results |
| No stop condition | Limit the number of steps or the budget |
| Letting the agent take risky actions freely | Require human approval and least-privilege permissions |
| Sending a `tool_result` without the matching `tool_use_id` | Always echo the `id` from the `tool_use` block |

---

## 4. Quick Review Checklist

- [ ] I can explain the 4-step tool use loop
- [ ] I know `tool_use`, `tool_result`, `tool_use_id`, and `stop_reason: "tool_use"`
- [ ] I know the 4 `tool_choice` options
- [ ] I can tell client tools, server tools, and MCP tools apart
- [ ] I can explain workflow vs agent, and when to choose each
- [ ] I can name the 5 workflow patterns
- [ ] I know context-management and safety techniques for agents

---

## 5. Practice Questions

<details>
<summary><b>Q1.</b> Claude returns <code>stop_reason: "tool_use"</code>. What must your app do next?</summary>

Run the requested tool, then send a **user** message containing a `tool_result` block with the matching `tool_use_id`. Claude then continues.
</details>

<details>
<summary><b>Q2.</b> Claude keeps picking the wrong tool. What's the first thing to improve?</summary>

The **tool descriptions**: make clear what each tool does, when to use it, and when *not* to.
</details>

<details>
<summary><b>Q3.</b> Support tickets fall into 3 clear categories, each needing a different prompt. Which pattern fits?</summary>

**Routing.** Classify first, then send each ticket to the specialized prompt.
</details>

<details>
<summary><b>Q4.</b> A task needs changes across an unknown number of files. Which pattern fits?</summary>

**Orchestrator–workers.** The subtasks can't be predicted in advance, so a lead model breaks the work down on the fly.
</details>

<details>
<summary><b>Q5.</b> A long-running agent is filling up its context window. Name two fixes.</summary>

Any two of: **compaction**, **subagents** with their own context, **memory files**, **just-in-time retrieval**, and **concise tool outputs**.
</details>

<details>
<summary><b>Q6.</b> You need guaranteed JSON with specific fields. How can tool use help?</summary>

Define a tool whose `input_schema` matches the JSON you want, and **force it** with `tool_choice`. Use strict mode for guaranteed schema matching.
</details>

---

## Further Reading
- Anthropic docs: *Tool use overview*: https://docs.claude.com/en/docs/agents-and-tools/tool-use/overview
- Anthropic engineering: *Building effective agents*: https://www.anthropic.com/engineering/building-effective-agents
