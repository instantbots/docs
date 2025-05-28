---
description: A brief introduction to Instant.bot and what we're all about
---

# Introduction

## What is Instant.bot?

[Instant.bot](http://instant.bot) is a customizable AI agent with a focus on function calling.

**You can extend your agent with hosted tools.** We host the functions for you as both REST API and MCP servers on auto-scaling architecture. These tools are available as open source packages;

* [Stripe (retrieves customers)](https://instant.bot/packages/@keith/stripe)
* [GPT Image Generator (creates images)](https://instant.bot/packages/@keith/openai-gpt-image)
* [Current weather](https://instant.bot/packages/@keith/weather)

**You can fork, modify or build your own versions of packages.** You can also build your own private packages just for you or your team. Sort of like GitHub for LLM tools.

**You can deploy your agent where it’s most convenient.** Currently you can chat on the web via our website or in Discord, but we are working on Slack, a standalone API and more.

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

**Instant.bot packages are just a collection of JavaScript functions.** [**Instant API**](https://github.com/instant-dev/api) is our battle-tested framework that turns simple JavaScript functions into fully-documented, type-validated API endpoints. It has scaled to billions of requests per week in production and currently powers the entire Instant.bot experience. Instant API will automatically create standards-compliant endpoint definitions (OpenAPI, JSON Schema) from function definitions that we then use to populate your package page and tool definitions.

**Packages are available as both REST API servers and MCP servers.** Packages expose a traditional OpenAPI specification at `/.well-known/openapi.json` and an MCP server at `/server.mcp`. We host them on top of auto-scaling architecture so you never have to worry about downtime.

**Our framework can be run anywhere.** You are not locked in to our registry or hosting platform. You can run Instant API projects on AWS, Vercel or any major cloud provider.

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

This function automatically gets exported as the endpoint `{package}.instant.host/hello`. Attempting to pass in `?age=-5` via query parameters will throw a `BadRequest` error and our gateway will reject it.

### 6. Rapid prototyping

Instant.bot provides both an online IDE and [command line utility](https://github.com/instantbots/ibot) that allow you to rapidly iterate on your hosted tool packages. You can use either to test your tools with specific parameters — our online IDE has a **\[Run]** button with configurable payloads, and our CLI provides `ibot run /tool-name --param1=value`.

Deploying to our hosted tool platform is instant, averaging \~3s per deploy, meaning testing via our online IDE is nearly as fast as building locally!

You can test how your functions integrate with your agent via our web chat interface at [https://instant.bot/chat](https://instant.bot/chat). Here you can chat with your agents and test your how your function integrations behave in real-time chat without any additional setup.

### 7. Model function-calling accuracy

We’re building our own planning engine as a distributed mixture of models that combines the world’s best function-calling models (GPT-4.1, Gemini Flash) to execute on tool strategies.

This is the place where we’re currently experimenting the most. Right now our default chatbot can perform parallel tool calls with high accuracy, but we’re still building multi-step planning and user-interaction modes. Please share feedback at [feedback@instant.bot](mailto:feedback@instant.bot)!

## How does pricing work?

You can view up-to-date pricing at [Instant.bot / Pricing](https://instant.bot/pricing). Quick summary;

* We charge a monthly subscription fee for access to our chatbot
  * Starting at $19 / mo for 5,000 messages / month
* We charge per-use for hosted tools at a rate of $0.50 per 1,000 GB-s of usage
  * 200ms execution for a hosted tool with 512MB of RAM is $0.00005
  * Meaning 1,000 function calls at this rate would be $0.05

You can pay for your agent absent of using tools, pay for the tools absent of using our agent, or pay for both together! Subscription refills will automatically add credits to your function credit balance.
