---
title: "ModelKit"
summary: "Build explicit text and function-tool workflows for Grok and Meta Muse wire formats."
category: "Overview"
order: 0
keywords: ["model", "grok", "muse", "responses", "chat", "tools"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# ModelKit

ModelKit provides source-only Buoy clients for Grok/xAI and Meta Muse using their Responses and compatible Chat Completions wire formats. It exposes explicit model selection, function-tool definitions, reply parsing, and conversation replay. It does not install a coding-agent harness or choose a substitute provider/model.

## Choose a request style

Responses supports replay of all raw output items, including encrypted reasoning. Prefer that route for an agent workflow that needs stateless history. Chat Completions provides text and local function-tool compatibility but does not preserve Meta reasoning across turns.

The package exposes text and local function tools. It does not implement provider-hosted tools, file/image/audio inputs, background responses, cancellation, usage aggregation, or a complete autonomous loop. Provider-specific extensions remain accessible through raw reply JSON.

## Transport and policy

The host supplies a token and actual account-available model name. HTTPS is required except for explicitly enabled local loopback fixtures. Requests use TLS verification, disabled redirects, finite timeouts, and synchronous buffered transport. Put network methods on workers and own the cancellation/timeout UI.

Tool calls are data. The host validates argument schemas, applies its own permission policy, executes approved tools, and returns correlated results. ModelKit never executes model-provided commands. Tokens stay in the client instance and are not automatically persisted.

Read [Getting started](getting-started.md) for a credential-free payload example or the [API reference](api.md) for request/replay and SSE contracts.
