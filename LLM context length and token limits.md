The context length of ChatGPT varies depending on the version you're using:

1. **ChatGPT (GPT-3.5)**:
   - The context length is **4,096 tokens**.
   - Tokens include input text and the model's responses. (Roughly 1 token ≈ 4 characters of English text.)

2. **ChatGPT (GPT-4)**:
   - Available in two context-length variants:
     - **8,192 tokens** (default for most applications).
     - **32,768 tokens** (available in GPT-4-turbo for extended use cases).

### Notes on Context Length:
- **Definition of Tokens**: Tokens are pieces of text; for example, "ChatGPT" is 2 tokens ("Chat" and "GPT").
- **Practical Implications**: The total number of tokens in a conversation (including system instructions, user input, and model output) cannot exceed the model's limit.
- **Use Cases for Longer Contexts**: The extended context in GPT-4 makes it ideal for analyzing long documents, maintaining continuity in extended conversations, or processing complex tasks.

ChatGPT-4 (also referred to as GPT-4) has the following context length options:

1. **8,192 Tokens** (Default):
   - This is the standard context length available for most GPT-4 instances.
   - Suitable for medium-length conversations or documents.

2. **32,768 Tokens** (Extended Context):
   - Available in **GPT-4-turbo**.
   - Ideal for handling long documents, in-depth analysis, or extensive interactions.

### Key Points About Context Length
- **Token Breakdown**: Both input (your prompt) and output (model's response) count toward the token limit.
- **Token Estimation**: 1 token ≈ 4 characters of English text, so 32,768 tokens translate to roughly 24,000–25,000 words.
- **Usage Scenarios**:
  - The 8,192-token limit is sufficient for most general-purpose applications.
  - The 32,768-token limit is optimal for large-scale document processing, like summarization, legal text review, or research analysis.

-------------


Here’s a breakdown of the token / context-length limits (i.e. how many tokens a model can “see” at once, combining system + history + user input + model output) for some of the prominent LLMs (ChatGPT, OpenAI’s models, Claude, etc.), along with caveats. Note that these are approximate, subject to variant / deployment constraints, and evolving.

---

## Key definitions / conventions

* *Context window* (or *context length*): the total number of tokens (or maximum allowable tokens) that the model can consider in one forward pass (i.e. all of the conversation + prompts + system + output).
* *Input tokens* + *output tokens* both count toward that window.
* In a chat setup, that includes: system prompt(s), all previous user and assistant messages (history), and the new user message + the model’s response.
* Sometimes there are additional constraints (e.g. “max output tokens” or “max input tokens”) enforced internally.
* What is usable in a particular deployment (ChatGPT UI, API, region) may differ from the theoretical model limit.

---

## OpenAI / GPT family

| Model                                        | Approx Context Window / Token Limit                   | Notes / caveats                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GPT-3 (base)                                 | ~ 2,048 tokens                                        | Classic GPT-3 (text models) used ~2,048 token windows. ([Wikipedia][1])                                                                                                                                                                                                                                                                                                                                                                             |
| GPT-3.5 / earlier ChatGPT / “Turbo” variants | ~ 16,000 tokens (in some versions)                    | Some “Turbo” / ChatGPT-3.5 variants had extended windows (documentation indicates “largest publicly available context window is 128k for GPT-4 Turbo, earlier models like 3.5 Turbo support up to 16k”) ([Milvus][2])                                                                                                                                                                                                                               |
| GPT-4 (standard)                             | 8,192 tokens and 32,768 tokens variants               | GPT-4 was released in versions with 8,192 and 32,768 token windows. ([Wikipedia][3])                                                                                                                                                                                                                                                                                                                                                                |
| GPT-4 Turbo                                  | ~ 128,000 tokens                                      | The “Turbo” version is often cited as having a 128k token context window. ([Milvus][2])                                                                                                                                                                                                                                                                                                                                                             |
| GPT-4o                                       | 128,000 tokens                                        | GPT-4o is often described as supporting a 128k token context window (which includes both input + output) ([TechTarget][4])                                                                                                                                                                                                                                                                                                                          |
| GPT-4.1                                      | up to 1,000,000 tokens (in some deployments)          | OpenAI claims GPT-4.1 supports a 1 million token context window in certain configurations. ([The Verge][5]) <br>But note: in many deployments (ChatGPT UI, Azure OpenAI in certain regions) it is still capped to lower values (e.g. 128k). ([Microsoft Learn][6])                                                                                                                                                                                  |
| GPT-5                                        | 400,000 tokens total (272,000 input + 128,000 output) | The OpenAI “model card / commentary” and analysis sites present GPT-5 as having ~ 272,000 tokens for input (i.e. prompt + history) and 128,000 tokens for the output (i.e. response) in total. ([Simon Willison’s Weblog][7]) <br>Thus the *total* is ~ 400k tokens. However, in ChatGPT UI or some variants, the full window may not be exposed. ([All Things How][8]) <br>One variant says 272K input + 128K output = 400K. ([All Things How][8]) |

**Implications for “last user message” size:**
If a model’s context window is, say, 128,000 tokens total, then the maximum size you could send in a single “last user message” (ignoring history) would be that 128,000 minus what’s occupied by system prompt + any preceding history + allowance for output. In practice, that means the last user message must be smaller — often a few tens of thousands of tokens at most, depending on how much history the model must retain.

For GPT-5’s case: with 400,000 total and 128,000 allowed for output, roughly 272,000 tokens remain for history + current input. So the last user message + prior chat could occupy up to ~ 272,000 (minus the system tokens) in the most generous scenario. But you wouldn’t typically send the *entire* context in one message — the system and history are pre-accumulated in memory.

---

## Claude (Anthropic)

* Claude’s API / Pro versions currently have a **200,000 token** context window (for input + output) in many offerings. ([support.anthropic.com][9])
* Recently, Claude Sonnet 4 has been upgraded to support **1,000,000 token** context in the Anthropic API (i.e. extended mode) in addition to the base 200k window. ([Anthropic][10])
* As of standard usage, the 200k token window is treated as the maximum total of message history + new input + output. ([Claude Docs][11])
* In chat usage, the system prompt + history + current user + output must all fit within that 200k window. ([Claude Docs][11])

Thus, in Claude’s default setting: if output is, say, capped to 4,096 tokens or some limit, the remaining ~195,000+ tokens could be used for system + history + current user input.

---

## Some other models / open-source variants

* Many smaller open models (e.g. variants of LLaMA, LLaMA 2) have “native” context windows of 2,048, 4,096, 8,192 etc. (depending on the version). ([AGI Sphere][12])
* Research works look at pushing beyond 128k (e.g. LongRoPE, other extended context methods) to millions of tokens. ([arXiv][13])

---

## Summary & caveats

* The *context window* is the upper bound on how much total token content (history, system, input, output) can be handled in a single pass.
* The “last user message” is only one part of that — it shares space with history and required output.
* In practice, many deployments enforce stricter limits than the theoretical model ceilings (region, UI, API tier).
* Also, not all parts of “history” are equally preserved: older messages may be truncated or summarized by the system to stay within window limits.
* These numbers are rapidly evolving: new models or updates may further extend limits (e.g. GPT-4.1, Claude Sonnet 4’s 1M window, research methods pushing to millions).

[1]: https://en.wikipedia.org/wiki/GPT-3?utm_source=chatgpt.com "GPT-3"
[2]: https://milvus.io/ai-quick-reference/what-is-the-maximum-context-window-for-openais-models?utm_source=chatgpt.com "What is the maximum context window for OpenAI's models? - Milvus"
[3]: https://en.wikipedia.org/wiki/GPT-4?utm_source=chatgpt.com "GPT-4"
[4]: https://www.techtarget.com/whatis/feature/GPT-4o-explained-Everything-you-need-to-know?utm_source=chatgpt.com "GPT-4o explained: Everything you need to know - TechTarget"
[5]: https://www.theverge.com/news/647896/openai-chatgpt-gpt-4-1-mini-nano-launch-availability?utm_source=chatgpt.com "OpenAI debuts its GPT-4.1 flagship AI model"
[6]: https://learn.microsoft.com/en-us/answers/questions/2280883/azure-openai-model-gpt-4-1-context-window-exceeded?utm_source=chatgpt.com "gpt-4.1 context window exceeded with way less than 1M tokens"
[7]: https://simonwillison.net/2025/Aug/7/gpt-5/?utm_source=chatgpt.com "GPT-5: Key characteristics, pricing and model card"
[8]: https://allthings.how/gpt-5-context-window-limits-and-usage-in-chatgpt-and-api/?utm_source=chatgpt.com "GPT-5 Context Window Limits and Usage in ChatGPT and ..."
[9]: https://support.anthropic.com/en/articles/7996856-what-is-the-maximum-prompt-length?utm_source=chatgpt.com "What is the maximum prompt length? | Anthropic Help Center"
[10]: https://www.anthropic.com/news/1m-context?utm_source=chatgpt.com "Claude Sonnet 4 now supports 1M tokens of context - Anthropic"
[11]: https://docs.anthropic.com/en/docs/build-with-claude/context-windows?utm_source=chatgpt.com "Context windows - Anthropic API"
[12]: https://agi-sphere.com/context-length/?utm_source=chatgpt.com "Context length in LLMs: All you need to know"
[13]: https://arxiv.org/abs/2402.13753?utm_source=chatgpt.com "LongRoPE: Extending LLM Context Window Beyond 2 Million Tokens"


--------------

Here’s a “living table” (as of mid-2025) of context / token limits or claimed context windows for several major LLMs (OpenAI, Anthropic, Google, etc.), along with caveats and notes about how much of that is usable for the last user message + history + system prompt.

| Model / System                                             | Claimed Context Window (total tokens)                                        | Breakdown / Notes                                                                                                                                            | Practical constraints / caveats                                                                                                                                                                                                                       |
| ---------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **OpenAI GPT-5 (Chat / API)**                              | ~ 128,000 tokens (in Chat mode) ([OpenAI Platform][1])                       | The “Model – OpenAI API” docs state the GPT-5 Chat model supports 128,000 tokens of context. ([OpenAI Platform][1])                                          | That is the *total* window (system prompt + history + user input + output). So the last user message + prior history + system must fit below 128K. Also, in some tiers, there are smaller UI limits (e.g. 32 K in Pro UI) ([X (formerly Twitter)][2]) |
| **OpenAI GPT-5 Pro**                                       | 400,000 tokens total (272,000 input + 128,000 output) ([OpenAI Platform][3]) | In the model card / docs, GPT-5 Pro is said to support a large “input + output” split (272K input, 128K output) for a total of ~400K. ([OpenAI Platform][3]) | But in many user-facing deployments or UI tiers, the effective limit may be far lower. (E.g. UI might only allow 128K or less.) Also, very large prompts >200K may incur higher cost or be rate-limited. ([Anthropic][4])                             |
| **Claude Sonnet 4**                                        | Up to **1,000,000 tokens** (in API / extended mode) ([Anthropic][4])         | Anthropic announced that Claude Sonnet 4 now supports a 1 million token context window on their API. ([Anthropic][4])                                        | The extended 1M window appears to be currently available via API / enterprise usage. For smaller prompts or base usage, they still have legacy 200K windows. ([Anthropic][5])                                                                         |
| **OpenAI GPT-4.1**                                         | 1,000,000 tokens                                                             | Various commentary / blogs claim GPT-4.1 provides a 1 million token context window. ([Medium][6])                                                            | These are mostly from secondary sources / analysis rather than official OpenAI docs. The actual UI or API deployment may enforce stricter limits.                                                                                                     |
| **Google Gemini (2.5, Flash / Pro, etc.)**                 | 1,000,000 tokens                                                             | Some sources report that Gemini models already offered 1M token context windows before many competitors. ([research.trychroma.com][7])                       | As with others, the claimed theoretical context may be more than what's exposed in UI or API usage. Also, quality / coherence can degrade at extreme lengths.                                                                                         |
| **Qwen / Alibaba models (Qwen3, etc.)**                    | 128,000 tokens (for many Qwen3 variants)                                     | According to Alibaba / Qwen documentation, many of the Qwen3 models support a 128K token context window. ([Wikipedia][8])                                    | That’s still high compared to older models, but far from the million-token frontier.                                                                                                                                                                  |
| **Legacy / smaller models (GPT-3, earlier GPT-3.5, etc.)** | 2,048 tokens (older GPT-3), 8,192 or 16,384 (some GPT-3.5 / Turbo variants)  | Historically, older models had small windows; some newer “Turbo” variants extend that.                                                                       | Many older models are now supplanted; their small windows are insufficient for long conversations or large documents.                                                                                                                                 |

---

### Interpretation: Last user message + history + system prompt

Given a model’s *total context window*, here's how to think about how much of that is available for the *last user message*:

* The system prompt (e.g. “You are ChatGPT …”) consumes a certain number of tokens.
* All prior user + assistant messages (the *history*) consume tokens.
* The new user message adds more tokens.
* The model’s *output* (its response) also must be counted (or at least reserved) in many systems, so you can’t push the entire context window into input — you need headroom for the reply.

As an example:

* Suppose a model supports 128K tokens total.
* You might allocate, say, 10K tokens (or more) for output.
* So the sum of system prompt + history + new user message must be ≤ (128K − output budget).
* If system + history already takes 40K, then your last user message + any appended context must fit in ~ (128K − 40K − 10K) = 78K tokens.

With larger models (e.g. 400K windows, 1M windows), you get much more flexibility in how large that last user message can be, but you're always gated by what history has already consumed.


[1]: https://platform.openai.com/docs/models/gpt-5-chat-latest?utm_source=chatgpt.com "Model - OpenAI API"
[2]: https://x.com/rohanpaul_ai/status/1953549303638557183?utm_source=chatgpt.com "Rohan Paul on X: \"GPT-5 context window limits in ChatGPT"
[3]: https://platform.openai.com/docs/models/gpt-5-pro?utm_source=chatgpt.com "Model - OpenAI API"
[4]: https://www.anthropic.com/news/1m-context?utm_source=chatgpt.com "Claude Sonnet 4 now supports 1M tokens of context - Anthropic"
[5]: https://www.anthropic.com/claude/sonnet?utm_source=chatgpt.com "Claude Sonnet 4 - Anthropic"
[6]: https://medium.com/%40divyanshbhatiajm19/the-ai-model-race-claude-4-vs-gpt-4-1-vs-gemini-2-5-pro-dab5db064f3e?utm_source=chatgpt.com "The AI Model Race: Claude 4 vs GPT-4.1 vs Gemini 2.5 Pro - Medium"
[7]: https://research.trychroma.com/context-rot?utm_source=chatgpt.com "Context Rot: How Increasing Input Tokens Impacts LLM Performance"
[8]: https://en.wikipedia.org/wiki/Qwen?utm_source=chatgpt.com "Qwen"
