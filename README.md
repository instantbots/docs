---
description: A brief introduction to Instant.bot and what we're all about
---

# Introduction

## What is Instant.bot?

[Instant.bot](http://instant.bot) is a customizable AI chatbot with a focus on function calling.

**You can extend it with hosted tools.** We host the functions for you on auto-scaling architecture. These tools are available as open source packages like [Stripe (retrieves customers)](https://instant.bot/packages/@keith/stripe) and [GPT Image Generator (creates images)](https://instant.bot/packages/@keith/openai-gpt-image). You can fork, modify or build your own versions of these packages, or build your own private ones. Sort of like GitHub for LLM tools.

**Once you’ve built your chatbot you can deploy where it’s most convenient.** Currently you can chat on the web via our website or in Discord, but we are working on Slack, a standalone API and more.

It’s still early: we’re in beta and have a lot to build, so if you encounter any rough edges please let us know, [feedback@instant.bot](mailto:feedback@instant.bot)!

## Who is this for?

Our ideal target customer is **developer-curious** and has a desire to build internal- or customer- facing chatbots to automate mundane daily tasks; like providing customer support in a Discord server, generating internal analytics charts, or providing refunds to customers. Our goal is to allow people to go from zero → working, function-integrated chatbot ASAP without having to be a professional software engineer.

However — **professional software engineers** should enjoy the platform, too. We are an **open core** company. Every part of the [Instant.bot](http://instant.bot) product is powered by open source software. Packages that run on our registry and gateway can be run locally or stood up on third-party providers like AWS and Vercel.

## Why are we building this?

Everybody is excited about the promise of agents that can execute arbitrary functions on your behalf. Most recently, [Model Context Protocol](https://modelcontextprotocol.io/introduction) by Anthropic has created a new standard for integrating hosted functions with language models.

However, the ecosystem is still very young. Building robust, secure agents that you can extend with (1) your own code and (2) trusted third-party code is non-trivial. Here are some questions we’ve asked ourselves;

1. **How can I verify the authenticity of hosted tools?** In a world of many integration providers (e.g. MCP servers), do I only trust major businesses, or is there a way to trust individual developers?
2. **How do I securely share third-party secrets?** I do not want hosted tools I do not control to have unlimited access to my data.
3. **How do I handle access control?** Can I allow or disallow certain tools based on the user who is requesting them?
4. **How do I reuse and modify the work of others before me?** For example, if Square’s MCP server doesn’t do exactly what I want, can I extend it?
5. **What protocols and frameworks do I have to learn?** There’s MCP, A2A, LangChain, OpenAI Agents SDK, Cloudflare Agents SDK, what do I choose and why?
6. **How do I rapidly prototype function integration with language models?**
7. **How do I ensure&#x20;**_**the model actually does what I want it to do**_**?** Simple demos work fine, but function-calling for complex tasks still seems hit-or-miss!

**At Instant.bot we’ve imagined a developer product directly integrated with a chatbot** — where writing code, testing functions, integrating function calls are an _integrated part_ of the chatbot experience. Our job is to answer each of these questions so you don’t have to and can simply focus on delivering an tool-integrated experience that matches your expectations.

## Product features

### 1. Verify tool authenticity

All hosted tools are open source API servers, with each API endpoint available as a tool. You can independently verify the code that’s running, and we also verify code authenticity via package checksums: if a package you don’t control gets overwritten with a different SHA256 value, we automatically alert you.

### 2. Securely share secrets

Third-party secrets are shared with hosted tools via **API Keychains**. Your API Keychain contains a list of secret keys you set. You approve access to a subset of these keys for each package during a configuration step. Access is revoked if a published package SHA256 changes.

### 3. Access control

Your API Keychain can optionally specific package- or endpoint-specific user controls for each package it is configured to access. In this way you can prevent usage of some tools entirely, or restrict tool access to a subset of users.

### 4. Tool reuse and modification

Since all packages are open source, they can all be trivially inspected, forked, modified, and republished by any user. You can copy public package contents into your own private forks, or build better, more robust forks to share with the community.

### 5. Protocols and frameworks

Our goal is to be protocol and framework agnostic on the package consumption side. By the time we leave beta you should be able to use Instant.bot packages with every major LLM provider (via MCP, A2A bindings) and agentic framework (LangChain, OpenAI Agents SDK, Cloudflare Agents SDK).

**Instant.bot packages are simply RESTful API servers.** They expose a traditional OpenAPI specification at `/.well-known/openapi.json`. We host them on top of auto-scaling architecture so you never have to worry about downtime.

**There is only one framework to learn:** [**Instant API**](https://github.com/instant-dev/api) is our battle-tested framework that turns simple JavaScript functions into fully-documented, type-validated API endpoints. It has scaled to billions of requests per week in production and currently powers the entire Instant.bot experience. Instant API will automatically creating standards-compliant endpoint definitions (OpenAPI, JSON Schema) from function definitions that we then use to populate your package page and tool definitions.

**Our framework can be run anywhere.** You aren’t locked in to our registry or hosting platform. You can run Instant API projects on AWS, Vercel or any major cloud provider.

Here's an example of a tool built with Instant API, a simple `hello world` endpoint that might exist at `functions/hello.js`:

```javascript
/**
 * A tool that says hello to the requester!
 * @param {string} name Your name
 * @param {integer{0,150}} age Your age between 0 and 150 (inclusive)
 * @returns {string}
 */
export async function GET (name = 'world', age = 30) {
  return `Hello ${name}, you are ${age} years old!`;
}
```

Automatically gets exported as the endpoint `{package}.instant.host/hello`. Attempting to pass in `?age=-5` via query parameters will throw a `BadRequest` error and our gateway will reject it.

### 6. Rapid prototyping

Instant.bot provides both an online IDE and [command line utility](https://github.com/instantbots/ibot) that allow you to rapidly iterate on your hosted tool packages. You can use either to test your tools with specific parameters — our online IDE has a **\[Run]** button with configurable payloads, and our CLI provides `ibot run /tool-name --param1=value`.

Deploying to our hosted tool platform is nearly instantaneous and averages \~3s, meaning testing via our online IDE is nearly as fast as building locally!

You can test how your functions integrate with your agent via our web chat interface at [https://instant.bot/chat](https://instant.bot/chat). Here can chat with your agents and test your how your function integrations behave in real-time chat without any additional setup.

### 7. Model function-calling accuracy

We’re building our own planning engine as a distributed mixture of models that combines the world’s best function-calling models (GPT-4.1, Gemini Flash) to execute on tool strategies.

This is the place where we’re currently experimenting the most. Right now our default chatbot can perform parallel tool calls with high accuracy, but we’re still building multi-step planning and user-interaction modes. Please share feedback at [feedback@instant.bot](mailto:feedback@instant.bot)!

## How does pricing work?

You can view up-to-date pricing at [Instant.bot / Pricing](https://instant.bot/pricing). Quick summary;

* We charge a monthly subscription fee for access to our chatbot
  * Starting at $20 / mo for 5,000 messages / month
* We charge per-use for hosted tools at a rate of $0.50 per 1,000 GB-s of usage
  * 200ms execution for a hosted tool with 512MB of RAM is $0.00005
  * Meaning 1,000 function calls at this rate would be $0.05

You can pay for the chatbot absent of using tools, pay for the tools absent of using our chatbot, or pay for both together! Subscription refills will automatically add credits to your function credit balance.
