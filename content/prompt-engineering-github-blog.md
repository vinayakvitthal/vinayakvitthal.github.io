# Prompt Engineering: Techniques That Actually Matter

> *A practical guide to getting reliable, high-quality outputs from LLMs — with real examples and patterns you can use today*

---

## Table of Contents

- [Why Prompt Engineering Still Matters](#why-prompt-engineering-still-matters)
- [Core Techniques](#core-techniques)
  - [1. Be Explicit About Format & Length](#1-be-explicit-about-format--length)
  - [2. Role + Context Framing](#2-role--context-framing)
  - [3. Chain-of-Thought (CoT) Prompting](#3-chain-of-thought-cot-prompting)
  - [4. Few-Shot Examples](#4-few-shot-examples)
  - [5. Output Constraints & Schema Forcing](#5-output-constraints--schema-forcing)
  - [6. Negative Prompting — Tell It What NOT to Do](#6-negative-prompting--tell-it-what-not-to-do)
  - [7. ReAct — Reasoning + Acting](#7-react--reasoning--acting)
  - [8. Self-Consistency & Sampling](#8-self-consistency--sampling)
- [System Prompt Architecture](#system-prompt-architecture)
- [Prompt Patterns for Common Tasks](#prompt-patterns-for-common-tasks)
- [What Doesn't Work (And Why)](#what-doesnt-work-and-why)
- [Evaluation: How to Know If Your Prompt Is Good](#evaluation-how-to-know-if-your-prompt-is-good)
- [Resources](#resources)

---

## Why Prompt Engineering Still Matters

With every new model release, someone declares "prompt engineering is dead." It isn't.

Models are getting better at understanding intent — but the gap between a mediocre prompt and a great one still produces dramatically different results. In production systems, that gap means the difference between a feature that works reliably and one that fails 20% of the time.

Prompt engineering is less about magic words and more about **clear communication and constraint**. Think of it as writing a precise spec for a very capable but very literal contractor.

This guide focuses on techniques that hold up across models and tasks — not tricks that worked once on GPT-3.

---

## Core Techniques

### 1. Be Explicit About Format & Length

The single highest-ROI change in most prompts.

**❌ Vague:**
```
Summarize this article.
```

**✅ Explicit:**
```
Summarize this article in 3 bullet points, each under 20 words.
Focus only on the business implications. Skip technical details.
```

**Why it works:** Models default to verbose, general outputs. Constraints force compression and prioritization — which is usually what you actually want.

**Format options to specify:**
- Output length (`under 100 words`, `exactly 3 items`, `one paragraph`)
- Structure (`bullet points`, `numbered list`, `JSON`, `markdown table`)
- Tone (`professional`, `casual`, `like you're explaining to a 10-year-old`)
- What to omit (`no caveats`, `no preamble`, `don't repeat the question`)

---

### 2. Role + Context Framing

Give the model an identity and a situation. This activates relevant "knowledge modes."

```
You are a senior backend engineer reviewing a pull request.
The codebase uses Python 3.11, FastAPI, and Postgres.
The team prioritizes readability over cleverness.

Review this function and suggest improvements:
[code here]
```

**The key components:**
- **Who** the model is (`senior backend engineer`)
- **What context** it's operating in (`FastAPI, Postgres`)
- **What it values** (`readability over cleverness`)

Role framing is especially powerful for:
- Code review (activates senior-engineer judgment)
- Writing (activates editorial voice)
- Analysis (activates consultant framing)
- Customer support drafts (activates empathetic tone)

```python
# In a system prompt for a production app
SYSTEM_PROMPT = """
You are a customer support specialist for Acme SaaS.
Your tone is warm, concise, and solution-focused.
You only answer questions about our product.
If you don't know something, say so and offer to escalate.
Never make up features or pricing.
"""
```

---

### 3. Chain-of-Thought (CoT) Prompting

For reasoning tasks, asking the model to "think step by step" dramatically improves accuracy.

**❌ Direct answer prompt:**
```
A store sells apples for $0.50 each and oranges for $0.75 each.
If you buy 4 apples and 3 oranges, what's the total?
```
*(Model often rushes to an answer and miscalculates)*

**✅ Chain-of-Thought:**
```
A store sells apples for $0.50 each and oranges for $0.75 each.
If you buy 4 apples and 3 oranges, what's the total?

Think through this step by step before giving your final answer.
```

**Zero-shot CoT trigger phrases:**
- `Think step by step.`
- `Let's work through this carefully.`
- `Break this down before answering.`
- `Reason through the problem first.`

**Few-shot CoT** (even more powerful for complex tasks):

```
Q: If I have 3 boxes with 8 items each, and I remove 5 items total, how many remain?
A: Let me work through this step by step.
   - 3 boxes × 8 items = 24 items total
   - 24 - 5 = 19 items remain
   Answer: 19

Q: [Your actual question here]
A: Let me work through this step by step.
```

---

### 4. Few-Shot Examples

Show, don't just tell. Examples are often more reliable than instructions.

**The pattern:**
```
Classify customer feedback as: POSITIVE, NEGATIVE, or NEUTRAL.

Examples:
Input: "This product changed my life, absolutely love it!"
Output: POSITIVE

Input: "Arrived broken, terrible packaging."
Output: NEGATIVE

Input: "It's fine, does what it says."
Output: NEUTRAL

Now classify:
Input: "Took forever to arrive but the quality is great."
Output:
```

**Rules for good few-shot examples:**
- Use 2–5 examples (more isn't always better — it inflates context)
- Cover edge cases you care about
- Keep examples diverse, not repetitive
- Match the format you want in the output exactly
- Put your actual input last

---

### 5. Output Constraints & Schema Forcing

For applications that consume LLM output programmatically, JSON schema forcing is essential.

**Without constraints:**
```
Extract the name, email, and company from this text:
"Hi, I'm Sarah Chen from Acme Corp. You can reach me at sarah@acme.com"
```
*Output might be a paragraph, a list, a table — inconsistent and hard to parse.*

**With schema forcing:**
```
Extract information from the text below.
Respond ONLY with valid JSON matching this exact schema — no other text:

{
  "name": "string",
  "email": "string or null",
  "company": "string or null"
}

Text: "Hi, I'm Sarah Chen from Acme Corp. You can reach me at sarah@acme.com"
```

**In code — using structured outputs (OpenAI):**
```python
from openai import OpenAI
from pydantic import BaseModel

class ContactInfo(BaseModel):
    name: str
    email: str | None
    company: str | None

client = OpenAI()
response = client.beta.chat.completions.parse(
    model="gpt-4o-2024-08-06",
    messages=[{"role": "user", "content": "Extract: Hi, I'm Sarah Chen from Acme Corp..."}],
    response_format=ContactInfo,
)
print(response.choices[0].message.parsed)
# ContactInfo(name='Sarah Chen', email='sarah@acme.com', company='Acme Corp')
```

---

### 6. Negative Prompting — Tell It What NOT to Do

Models respond well to explicit exclusions. Don't just describe what you want — describe what you don't want.

```
Write a short product description for this ergonomic chair.

Do NOT:
- Use the word "revolutionary" or "game-changing"
- Make unverifiable health claims
- Exceed 80 words
- Use a bulleted list

DO:
- Focus on comfort and design
- Use a conversational but professional tone
```

**Common things worth negating:**
- `Do not add caveats or disclaimers unless explicitly asked`
- `Do not repeat back the question or instructions`
- `Do not say "As an AI language model..."`
- `Do not use filler phrases like "Certainly!" or "Great question!"`
- `Do not invent information you're not certain about — say "I don't know" instead`

---

### 7. ReAct — Reasoning + Acting

For **agentic tasks** — where a model needs to take multiple steps, use tools, and observe results — the ReAct pattern structures the loop.

```
You have access to these tools:
- search(query) → returns web search results
- calculator(expression) → evaluates math expressions
- get_weather(city) → returns current weather

To complete a task:
1. THINK: reason about what you need to do next
2. ACT: call a tool with the required input
3. OBSERVE: note the tool's output
4. Repeat until you can give a final answer

Task: What's the temperature difference between the current temperature
in Mumbai and the boiling point of water?
```

**Example model output with ReAct:**
```
THINK: I need the current temperature in Mumbai, then compare to water's boiling point (100°C).
ACT: get_weather("Mumbai")
OBSERVE: Current temp in Mumbai: 34°C

THINK: Boiling point of water is 100°C. Difference = 100 - 34 = 66°C.
ANSWER: The boiling point of water is 66°C higher than the current temperature in Mumbai.
```

This is the backbone of most AI agent frameworks (LangChain, LlamaIndex, AutoGPT).

---

### 8. Self-Consistency & Sampling

For high-stakes reasoning tasks, generate multiple responses and take a majority vote or the best-reasoned answer.

```python
import openai
from collections import Counter

def self_consistent_answer(question: str, n: int = 5) -> str:
    responses = []
    for _ in range(n):
        response = openai.chat.completions.create(
            model="gpt-4o",
            messages=[
                {"role": "user", "content": f"{question}\nThink step by step."}
            ],
            temperature=0.7  # Higher temp = more diverse responses
        )
        responses.append(response.choices[0].message.content)

    # Extract final answers and take majority vote
    # (In practice, parse the final answer from each response)
    return responses  # Then analyze for consensus

# Useful for: classification, fact extraction, code generation
```

**When to use:** Legal/compliance classification, medical triage, financial decisions, any task where consistency matters more than speed.

---

## System Prompt Architecture

For production applications, think of your system prompt as having distinct sections:

```
[IDENTITY]
You are [name/role], [brief description].

[CONTEXT]
You are operating as [product context].
Users are [who they are].

[CAPABILITIES]
You can help with: [explicit list]
You have access to: [tools/data]

[CONSTRAINTS]
Never: [hard limits]
Always: [non-negotiables]
If unsure: [fallback behavior]

[FORMAT]
Respond in [format].
Keep responses [length guideline].
Tone: [tone description].
```

**Real example:**
```
You are Aria, a support assistant for CloudBase, a developer infrastructure platform.

You are talking to software engineers and DevOps professionals who are customers of CloudBase.
They expect direct, technical answers without hand-holding.

You can help with: billing questions, technical troubleshooting, account settings, and feature explanations.
You have access to our knowledge base and the user's account information.

Never: make up pricing, promise features that don't exist, or share other customers' data.
Always: provide documentation links when explaining technical concepts.
If you can't resolve an issue: offer to create a support ticket with a 4-hour SLA.

Keep responses under 150 words unless a technical explanation requires more.
Use markdown formatting for code. Tone: direct, friendly, technical.
```

---

## Prompt Patterns for Common Tasks

### Summarization
```
Summarize the following [document type] for a [target audience].
Focus on: [key themes].
Ignore: [what to skip].
Format: [3 bullet points / 1 paragraph / executive summary].
Length: under [N] words.
```

### Classification
```
Classify the input into exactly one of these categories: [A, B, C, D].
If ambiguous, choose the closest match.
Respond with only the category name — no explanation.

Examples:
[2-3 labeled examples]

Input: [text to classify]
```

### Code Generation
```
Write a [language] function that [specific behavior].
Requirements:
- [requirement 1]
- [requirement 2]
Include:
- Type hints
- A docstring
- 2-3 unit tests
Do not include: package imports I didn't ask for, main() boilerplate.
```

### Data Extraction
```
Extract the following fields from the text below.
If a field is missing, use null.
Return only valid JSON, no other text.

Fields: {field1: type, field2: type, field3: type}

Text: [input text]
```

---

## What Doesn't Work (And Why)

| Technique | Why It Fails |
|-----------|-------------|
| `"Answer as best you can"` | Too vague — the model already tries to do this |
| `"Be creative but accurate"` | Contradictory constraints confuse the model |
| Extremely long system prompts | Critical instructions at the end get lost (recency/primacy bias) |
| `"Never make mistakes"` | Models can't guarantee correctness — adds false confidence |
| Repeating the same instruction 5x | Repetition ≠ emphasis; use structure instead |
| `"Think outside the box"` | Generic phrase with no actionable meaning |
| Over-constraining | Too many "don't do X" rules creates failure modes |

**The #1 failure mode:** Prompts that describe *what you want the output to look like* but not *what the model should actually reason about*. Describe the reasoning process, not just the output.

---

## Evaluation: How to Know If Your Prompt Is Good

Gut feeling isn't good enough for production. Use these methods:

**1. Regression testing**
```python
TEST_CASES = [
    {"input": "...", "expected": "POSITIVE"},
    {"input": "...", "expected": "NEGATIVE"},
    # 20-50 cases covering edge cases
]

def eval_prompt(prompt, test_cases):
    correct = 0
    for case in test_cases:
        output = call_llm(prompt, case["input"])
        if output.strip() == case["expected"]:
            correct += 1
    return correct / len(test_cases)
```

**2. LLM-as-judge**
```
You are evaluating an AI response for quality.

Criteria:
- Accuracy (1-5): Is the information correct?
- Relevance (1-5): Does it answer the question asked?
- Conciseness (1-5): Is it appropriately brief?

Question: [original question]
Response: [model output]

Score each criterion and provide a one-sentence justification.
```

**3. A/B testing in production** — Route 5-10% of traffic to a new prompt variant. Measure task completion, user corrections, escalation rate.

---

## Resources

- [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
- [Anthropic's Prompt Engineering Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Chain-of-Thought Paper (Wei et al., 2022)](https://arxiv.org/abs/2201.11903)
- [ReAct Paper (Yao et al., 2022)](https://arxiv.org/abs/2210.03629)
- [Self-Consistency Paper (Wang et al., 2022)](https://arxiv.org/abs/2203.11171)
- [DSPY — Programmatic Prompt Optimization](https://github.com/stanfordnlp/dspy)
- [PromptFoo — Prompt Testing Framework](https://github.com/promptfoo/promptfoo)

---

*Found this useful? ⭐ Star the repo and share it with your team.*  
*Have a technique I missed? Open an issue or submit a PR.*