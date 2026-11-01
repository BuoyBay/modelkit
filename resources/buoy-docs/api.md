---
title: "ModelKit API"
summary: "Explicit provider clients, immutable replies and tool calls, guarded JSON, and SSE decoding."
category: "Reference"
order: 20
keywords: ["response", "replay", "tool", "sse", "error", "http"]
symbols: ["ModelProvider", "ModelClient", "ModelClient.ForGrok", "ModelClient.ForMetaMuse", "ModelClient.CreateResponse", "ModelClient.ContinueResponse", "ModelClient.Chat", "FunctionTool", "TextMessage", "ParseModelReply", "ModelReply", "ModelReply.AppendOutputTo", "ModelToolCall", "ModelToolCall.ParseArguments", "ModelToolCall.ResultItem", "ModelAPIException", "SSEEvent", "ModelSSEDecoder", "ModelSSEDecoder.Feed", "ModelSSEDecoder.Finish", "ModelSSEDecoder.Reset"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# API reference

## Request construction and clients

| Symbol | Contract |
| --- | --- |
| `ModelProvider` | Explicit Grok and MetaMuse enum values |
| `ForGrok(token)` / `ForMetaMuse(token)` | Shared `ModelClient` factories for the selected provider |
| `ModelClient(provider, token, apiBase, allowLoopbackHTTP = False)` | Explicit validated provider/base/token configuration |
| `Provider` / `BaseURL` | Read-only client identity/origin |
| `TimeoutSeconds = 60` / `MaxResponseBytes = 16777216` | Mutable transport timeout and post-buffer size limit |
| `CreateResponse(model, input, tools)` | Responses request with `store=False`, `stream=False`, encrypted reasoning included for replay |
| `ContinueResponse(model, input, tools, previousResponseID, store = False)` | Explicit continuation; a nonempty previous ID requires `store=True` |
| `Chat(model, messages, tools)` | Chat Completions text and local function-tool compatibility |
| `FunctionTool(name, description, parameters, responsesAPI = True)` | Tool definition in the selected dialect; parameters must be an object schema |
| `TextMessage(role, content, responsesAPI = True)` | Build a text message in the selected dialect |
| `ParseModelReply(raw, responsesAPI = True)` | Parse provider data without a transport call |

Models are always caller-supplied. The package does not establish account entitlement, silently substitute models, or persist tokens. Requests verify TLS and refuse redirects.

## Replies and tool calls

`ModelReply` exposes `Text`, `ToolCalls`, `FinishReason`, `ResponseID`, and detached `RawJSON`. `AppendOutputTo(history)` appends every original Responses output item or the original Chat assistant message, including provider extensions. Responses finish reason is its status; Chat uses the first choice's finish reason. Failed/incomplete replies are not relabelled as completed.

`ModelToolCall` exposes immutable `CallID`, `ItemID`, `Name`, `ArgumentsText`, and detached `RawJSON`. `ParseArguments()` requires a JSON object but does not validate it against your schema. `ResultItem(content, responsesAPI = True)` correlates output to `CallID`, not `ItemID`. The host owns tool policy/execution and must preserve the original assistant/output objects before returning results.

## HTTP failures and JSON validation

`ModelAPIException` exposes HTTP `Status`, sanitized `APIMessage`, `RequestID`, `RetryAfter`, and `RateLimitRemaining`. Transport errors omit raw messages to reduce credential/configuration disclosure. Response content is never logged by the package.

External JSON is guarded before native parsing: nesting is capped at 128; raw/escaped NUL, unpaired Unicode surrogate escapes, malformed numeric spelling, and nonfinite numbers reject. Literal escaped backslash-plus-`u0000` text remains valid. The same guard applies to copied raw replies and tool arguments.

## SSE utility

`ModelSSEDecoder(maxLineBytes = 1048576, maxEventBytes = 8388608)` accepts fragmented UTF-8 through `Feed(chunk)` and returns `Array Of SSEEvent`. Events expose `EventName`, `Data`, `ID`, and `IsDone` for the `[DONE]` sentinel. CRLF, LF, CR, comments, repeated data lines, and IDs are supported.

Completed data must be guarded JSON or exactly `[DONE]`. Invalid input/limits poison the decoder until `Reset()`. `IsFailed` and `IsDone` report state. `Finish()` discards an incomplete final event. Other default bounds are 32 MiB per feed and 4,096 events per feed. This utility does not provide a streaming HTTP transport or accumulate streamed tool arguments.

See [Getting started](getting-started.md) for a compiled offline example or the [overview](index.md) for scope.


[Full typed API: properties, parameters, defaults, and returns](typed-api.md)
