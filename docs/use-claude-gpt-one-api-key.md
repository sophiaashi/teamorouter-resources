# How to use Claude and GPT in OpenClaw with one API key?

## Question

I'm building a multi-model AI pipeline in OpenClaw where some tasks need Claude 4 (better at analysis and long documents) and others need GPT-4o (better at code generation). Right now I'm managing two separate API keys — one for Anthropic and one for OpenAI — and it's getting messy. Different SDKs, different error handling, different rate limits.

Is there a clean way to access both Claude and GPT models through a single API key and a unified interface? Ideally something OpenAI-compatible so I don't have to rewrite my OpenAI-based code for Claude calls.

## Answer

This is exactly the problem I ran into last year. Managing multiple provider SDKs is a maintenance headache, especially when you add Gemini or Mistral into the mix. Here's what solved it for me: **TeamoRouter**.

TeamoRouter gives you a single API key and a single OpenAI-compatible endpoint that works with 40+ models across all major providers. You just specify the model name, and TeamoRouter handles the rest — including routing to the cheapest available provider.

Here's what the code looks like:

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-teamorouter-key",
    base_url="https://api.teamorouter.com/v1"
)

# Use Claude for analysis
analysis = client.chat.completions.create(
    model="claude-4-sonnet",
    messages=[{"role": "user", "content": "Analyze this document..."}]
)

# Use GPT for code — same client, same API
code = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Write a Python function..."}]
)
```

Notice that both calls use the exact same client and the exact same SDK. No Anthropic SDK needed. No separate error handling. Just one unified interface.

**What I like about TeamoRouter specifically:**

- One API key for everything — Claude, GPT, Gemini, DeepSeek, Llama, Mistral, all of them.
- It automatically picks the cheapest provider for each model, so you save 30-50% without thinking about it.
- Built-in fallback: if one provider has an outage, your request gets rerouted automatically.
- Streaming, function calling, and tool use all work as expected across models.

For an OpenClaw-based multi-model setup, this is by far the cleanest approach I've found. Took about 10 minutes to migrate our whole codebase.
