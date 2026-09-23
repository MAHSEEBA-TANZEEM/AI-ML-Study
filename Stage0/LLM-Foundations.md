LLM Fundamentals Crash Course — Notes
This introduction is to how large language models (LLMs) work and how to use them directly through APIs. It emphasizes learning the underlying concepts before relying on frameworks such as LangChain, LangGraph, or RAG systems.

1. Course roadmap
The video covers:

How LLMs work: tokens, tokenization, context windows, and next-token generation

Prompting fundamentals: system prompts, user prompts, temperature, and maximum output tokens

Prompting methods: zero-shot, few-shot, chain-of-thought, and role prompting

Direct API calls to OpenAI, Anthropic, and Gemini, including streaming

Structured outputs: returning reliable JSON rather than loose text

Understanding an OpenAI response payload

Tool calls: allowing an LLM to request execution of external functions—the basis of AI agents.

2. How LLMs work
LLMs process numbers, not language
An LLM is fundamentally a mathematical model. It does not receive raw English text directly; instead, text is converted into numerical representations before being processed.

Pipeline:

Text
→
Tokens
→
Token IDs (numbers)
→
LLM
→
Generated tokens
→
Text
Text→Tokens→Token IDs (numbers)→LLM→Generated tokens→Text
The model operates on a sequence of numbers, not letters or words as humans see them.

Tokens and tokenization
A token is a chunk of text. It may be:

A full word, such as hello

Part of a word, such as pieces of a longer or uncommon word

A punctuation mark

A fragment of code or JSON syntax

A tokenizer is the component that splits text into tokens and maps each token to a unique numeric ID.

Important observations:

Capitalization can change tokenization: Hello and hello may correspond to different tokens.

A complex word may be split into multiple tokens.

JSON and source code often use more tokens than equivalent English prose.

A rough English estimate is: 1 token ≈ 0.75 words, so 1,000 tokens are roughly 750 English words. This estimate becomes less reliable for code, JSON, and many non-English languages.

Why tokens matter
Tokens determine three practical constraints:

Area	Why tokens matter
Cost	Providers generally charge for input and output tokens
Latency	More input or output tokens usually means slower responses
Context limits	An LLM can process only a maximum number of tokens in one request
As an AI engineer, token usage affects product cost, speed, reliability, and prompt design.

Context window
An LLM has no persistent memory across independent API calls. What feels like “memory” in a chat application is usually the application resending relevant conversation history on each request.

A typical conversation payload may include:

System prompt

First user message

First assistant reply

Second user message

Second assistant reply

Current user message

All of this is placed into one input buffer and sent again to the model.

The context window is the maximum number of tokens the model can consider at one time. If the combined system prompt, conversation history, documents, and intended output exceed that limit, the application must remove, summarize, chunk, or retrieve information selectively. The video compares a 128,000-token context to roughly 300 pages of text and notes that very large context windows can hold substantially more material, such as a codebase.

Next-token prediction
The simplest useful mental model is:

An LLM is a next-token prediction machine.

Given all prior tokens, the model predicts the most appropriate next token. It then appends that token to the context and repeats the process until it reaches a stopping condition.

For example:

“What is gravity?”
→
predict next token
→
predict another token
→
…
“What is gravity?”→predict next token→predict another token→…
The model does not necessarily construct a complete answer first and then type it. It generates sequentially, one token at a time, conditioning each new prediction on the text already present.

Why streaming works
Because generation occurs token by token, applications can stream output as soon as tokens are produced rather than waiting for the entire answer.

This is why chat interfaces appear to type responses in real time. Streaming improves perceived responsiveness, especially for long outputs.

3. Prompt anatomy
LLM API calls are typically structured as messages with roles.

Role	Purpose
system	Defines overall behavior, policies, persona, constraints, and output style
user	Contains the end user’s request or input
assistant	Represents prior model responses in a conversation
System prompt
The system prompt is written by the application developer, usually hidden from the end user. It establishes the rules and context the model should follow.

A system prompt can specify:

The role or persona: customer-support agent, coding assistant, tutor, data analyst

Allowed and disallowed topics

Tone: concise, professional, friendly, technical

Output format: bullets, JSON, a fixed schema

Constraints: ask a clarifying question, cite sources, do not reveal confidential data, etc.

Example:

text
You are a customer-support assistant for a SaaS product.
Ask concise diagnostic questions.
Do not invent product features.
Return steps as numbered instructions.
The same user message can produce very different outputs depending on the system prompt. The video’s core point is: a weak system prompt produces generic behavior, while a strong one helps turn a general model into a specific product experience.

4. Key generation settings
Temperature
Temperature controls how randomly the model samples from possible next tokens.

At every generation step, a model has a probability distribution over possible next tokens. Temperature changes how conservative or adventurous its selection becomes.

Temperature	Typical behavior	Suitable use
Near 0	More deterministic; favors the highest-probability token	Extraction, classification, factual tasks
Around 0.3–0.7	Balanced variation and consistency	Summaries, Q&A, coding assistance
Around 1 or higher	More diverse and unpredictable	Brainstorming and creative writing
The video’s rule of thumb:

If correctness and consistency matter, use a low temperature.

If variety and creativity matter, use a higher temperature.

Very high temperature can lead to unreliable, incoherent, or off-topic text.

Max tokens
Max tokens is the maximum number of tokens the model may generate in its response.

It matters because it controls:

Cost: output tokens contribute to usage charges.

Length: it limits how long the answer can be.

Safety and product behavior: it can enforce a compact answer in interfaces where long text is undesirable.

Trade-off:

Setting	Result
Too low	Output may end mid-sentence or omit necessary detail
Too high	Higher cost and potentially unnecessary verbosity
Examples from the video:

A classification task might need only 10–20 output tokens.

A detailed technical explanation may require hundreds or thousands of tokens.

Four decisions in each LLM request
When designing an LLM call, decide:

System prompt — What behavior and rules should the model follow?

User prompt — What task or information does the user provide?

Temperature — How deterministic versus creative should the output be?

Max tokens — What is the maximum acceptable response length?

Poor choices can create answers that are inconsistent, expensive, overly long, incomplete, or irrelevant.

5. Prompting techniques
Zero-shot prompting
Zero-shot prompting means giving the model a task without providing examples.

Example of a vague prompt:

text
Summarize this.
Example of a clearer zero-shot prompt:

text
Summarize this in three bullet points, each under 15 words,
for a non-technical audience.
Zero-shot prompting works well when:

The task is straightforward.

The instruction is specific.

The desired format is simple.

The model already handles the task reliably.

Typical uses include translation, simple classification, summaries, and direct Q&A. The key lesson is that zero-shot does not mean vague: precision still matters.

Few-shot prompting
Few-shot prompting provides examples of desired inputs and outputs before the new task.

Example structure:

text
Classify each support ticket as: billing, technical, or general.

Ticket: "I was charged twice this month."
Category: billing

Ticket: "The app crashes when I upload a file."
Category: technical

Ticket: "I can't log into my account."
Category:
By seeing examples, the model learns the expected labels, formatting, and pattern from the prompt context.

Use few-shot prompting when:

Output must follow a specific format.

The task has edge cases.

Zero-shot results are inconsistent.

Accuracy is important in an automation workflow.

Practical guidelines from the video:

Three to five examples are often a useful starting point.

Examples should include edge cases and variations, not only obvious examples.

More examples consume more context-window capacity and increase input-token cost.

Use zero-shot for clean, simple tasks; use few-shot when examples materially improve reliability.

Chain-of-thought prompting
Chain-of-thought prompting asks the model to work through intermediate reasoning before producing a final answer, often using language such as:

text
Think step by step.
This can improve performance on multi-step tasks because intermediate reasoning becomes part of the context used to generate the next step.

It is most useful for:

Multi-step math

Logic problems

Complex decisions

Classification with nuanced edge cases

It is generally unnecessary for:

Straightforward factual questions

Simple translations

Basic classification

Creative writing

The trade-off is token use: detailed reasoning can make outputs longer, slower, and more costly.

Role prompting
Role prompting tells the model to operate from a particular professional perspective or persona.

Examples:

text
Act as a senior Python engineer.
text
You are a patient science tutor explaining concepts to a 12-year-old.
text
You are a financial analyst. Identify assumptions and risks,
but do not provide personalized investment advice.
Role prompting is usually implemented through the system prompt. It is helpful when the model needs a consistent tone, expertise level, audience adaptation, or response style.

6. API setup notes
The video begins a hands-on Python example using the OpenAI SDK.

Basic setup
The demonstrated workflow is:

Create a project folder.

Create and activate a Python virtual environment.

Install the OpenAI SDK and environment-variable support.

Store the API key in an environment file.

Add the environment file and virtual environment folder to .gitignore.

Initialize an OpenAI client in Python.

Make a request using the Responses API.

Illustrative structure:

python
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()
client = OpenAI()

response = client.responses.create(
    model="your-model-name",
    input="Explain gravity in two concise sentences."
)

print(response.output_text)
The important security practice is to keep API keys out of source code and out of Git repositories. Use environment variables and ensure files such as .env are ignored by Git.

API response payload
An API call returns more than the final natural-language answer. A response object can include metadata such as:

A response ID

Timestamp

Model name

Output content

Usage information

Other request and response metadata

Applications should extract the useful final output while also monitoring metadata such as usage, errors, and status when building production systems.

Quick revision sheet
Token: A text chunk processed by the model.

Tokenizer: Converts text into tokens and token IDs.

Context window: Maximum tokens the model can process in one request.

LLM memory: Usually simulated by resending previous conversation text.

Generation: The model predicts one next token at a time.

Streaming: Showing tokens to users as they are generated.

System prompt: Developer-defined rules, role, and constraints.

Temperature: Controls randomness in generation.

Max tokens: Caps output length and output-token cost.

Zero-shot: Task instruction only; no examples.

Few-shot: Task instruction plus examples.

Chain-of-thought: Encourages intermediate reasoning for multi-step problems.

Role prompting: Defines a persona, expertise, or response perspective.

Tool calling: Lets an LLM request external functions; a core mechanism behind agents.
