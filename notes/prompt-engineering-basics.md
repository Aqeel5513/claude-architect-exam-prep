# Prompt Engineering Basics

> **In one line:** Prompt engineering means writing instructions so Claude reliably gives you the output you want, then testing and improving those instructions.

---

## 1. Start With Success Criteria (Before Writing Any Prompt)

Before you write a prompt, decide:

1. **What does a good answer look like?** (accuracy, tone, format, length)
2. **How will you test it?** Collect a set of example inputs (test cases), including tricky edge cases.
3. **Is prompting the right fix?** Some problems are better solved by choosing a different model, adding tools or retrieval, or changing the workflow.

> 💡 Exam tip: The recommended order is **define success → build test cases → write prompt → evaluate → iterate**. Don't start by "tweaking words".

---

## 2. Anatomy of a Prompt

| Part | Purpose | Example |
|---|---|---|
| **System prompt** | Sets the role, rules, and context for the whole conversation | "You are a support agent for an online bookstore." |
| **User message** | The actual task or question | "Summarize this customer complaint." |
| **Context / data** | Documents, records, or inputs Claude should use | The complaint text |
| **Examples** | Show what good output looks like | 2–3 sample complaint → summary pairs |
| **Output format** | Tells Claude exactly how to respond | "Reply in JSON with keys `issue` and `urgency`." |

---

## 3. Core Techniques

### 3.1 Be Clear and Direct
Treat Claude like a smart new colleague who has **no background** on your task.

- State the task, the audience, and the goal.
- Use numbered steps for multi-step tasks.
- Say what you **want**, not only what to avoid.

| ❌ Vague | ✅ Clear |
|---|---|
| "Write about our product." | "Write a 100-word product description of our noise-cancelling headphones for busy commuters. Friendly tone. End with one call to action." |

**Golden rule:** If a colleague would be confused by your prompt, Claude will be too.

### 3.2 Explain *Why* (Give Context)
Explaining the reason behind an instruction helps Claude make better choices in situations you didn't anticipate.

- ❌ "Never use ellipses."
- ✅ "Your response will be read aloud by a text-to-speech engine, so never use ellipses, because it can't pronounce them."

### 3.3 Use Examples (Few-Shot / Multishot Prompting)
Showing is often more effective than telling.

- Give **3–5 examples** that are relevant, varied, and cover edge cases.
- Wrap them in tags such as `<example>` so Claude can tell them apart from instructions.
- Claude copies the examples closely, so make sure they show exactly what you want.

### 3.4 Structure With XML Tags
Tags separate the different parts of a prompt so nothing gets mixed up.

```xml
<instructions>
Summarize the contract below for a non-lawyer.
</instructions>

<contract>
{{CONTRACT_TEXT}}
</contract>

<format>
Three bullet points, then a one-line risk rating.
</format>
```

- No tag names are "special". Choose clear names and use them consistently.
- You can refer to a tag in your instructions: "Using the contract in `<contract>` tags…"

### 3.5 Give Claude a Role (Role Prompting)
Setting a role in the **system prompt** focuses tone and expertise.

> "You are a senior security engineer reviewing code for vulnerabilities."

Use this for domain-specific tasks such as legal, finance, medical, or technical review.

### 3.6 Let Claude Think (Chain of Thought)
For complex reasoning (math, analysis, multi-step decisions), let Claude reason before answering.

| Level | How |
|---|---|
| Basic | "Think step by step." |
| Guided | List the specific steps Claude should reason through. |
| Structured | Ask for reasoning in `<thinking>` tags and the final answer in `<answer>` tags. |
| Built-in | Turn on **extended thinking** in the API to give Claude a reasoning budget. |

> ⚠️ Thinking improves quality but adds **latency and tokens**. Use it when the task needs it.

### 3.7 Control the Output Format
- Describe the format explicitly (e.g. JSON, a table, or a fixed number of bullets).
- Ask for output inside tags (e.g. `<summary>`) so your code can extract it easily.
- For guaranteed machine-readable output, use **structured outputs / tool use with a schema** instead of relying on prompt wording alone.
- The style of your prompt influences the style of the response: a prompt with no markdown tends to get a response with less markdown.

### 3.8 Long Documents
- Put long documents **at the top** of the prompt and your question **at the end**.
- Wrap each document in tags with metadata (`<document><source>…</source><content>…</content></document>`).
- Ask Claude to first **pull out the relevant quotes**, then answer based on them. This reduces made-up answers.

### 3.9 Prompt Chaining
Split a big task into smaller prompts, where each step's output feeds the next.

```
Research → Outline → Draft → Review/Edit
```

Benefits: each step is simpler, easier to debug, and easier to improve on its own.

---

## 4. Reducing Hallucinations

- Allow Claude to say **"I don't know"**.
- Ask it to **ground answers in quotes** from the provided documents.
- Ask it to **cite sources** for each claim.
- Check important outputs, either with a second "verify this" prompt or a human review.

---

## 5. Common Mistakes

| Mistake | Fix |
|---|---|
| Vague task | State the goal, audience, format, and length |
| Instructions mixed with data | Separate them with XML tags |
| Only saying what *not* to do | Say what you *want* instead |
| One giant prompt for a complex job | Use prompt chaining |
| No testing | Build test cases and evaluate every change |
| Examples that are all the same | Use varied examples that include edge cases |

---

## 6. Quick Review Checklist

- [ ] Success criteria and test cases defined
- [ ] Clear task, audience, and goal
- [ ] Context explains *why*
- [ ] Data separated with XML tags
- [ ] 3–5 varied examples (if format matters)
- [ ] Role set in system prompt (if domain-specific)
- [ ] Room to think (if reasoning is complex)
- [ ] Output format specified
- [ ] Tested and improved over several rounds

---

## 7. Practice Questions

<details>
<summary><b>Q1.</b> A team's prompt gives inconsistent output formats. What's the most effective first fix?</summary>

Add **2–3 examples** of the exact desired format (few-shot), and/or specify the format explicitly. For strict machine-readable output, use structured outputs or a tool schema.
</details>

<details>
<summary><b>Q2.</b> Where should a 50-page document go in a prompt relative to the question?</summary>

**Document first, question last.** Putting the query at the end tends to improve response quality on long inputs.
</details>

<details>
<summary><b>Q3.</b> Why wrap user-supplied data in XML tags?</summary>

It **separates data from instructions**, so Claude doesn't confuse the two. It also makes the prompt easier to read and to refer to ("the text in `<email>` tags").
</details>

<details>
<summary><b>Q4.</b> A task needs research, then writing, then fact-checking. One prompt or several?</summary>

**Several (prompt chaining).** Each step is focused, easier to debug, and can be improved on its own.
</details>

<details>
<summary><b>Q5.</b> What should you do before writing a prompt at all?</summary>

**Define success criteria and build test cases**, so you can measure whether a prompt change actually helps.
</details>

---

## Further Reading
- Anthropic docs: *Prompt engineering overview*: https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
