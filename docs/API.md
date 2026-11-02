# ModelKit

Source-only Buoy clients for **Grok/xAI** and **Meta Muse** with explicit model
selection, function-tool calls and conversation replay. Both providers document
Responses and OpenAI-compatible Chat Completions endpoints. This package uses
their published wire formats; it does not install a coding-agent harness.

## Use

Copy `packages/modelkit-0.1.0.buipkg` into a project's `packages/` and use
`Import modelkit`. [The example](../examples/modelkit_demo.bui) builds payloads
offline without credentials or inference calls.

- `ModelClient.ForGrok(token)` uses `https://api.x.ai/v1`.
- `ModelClient.ForMetaMuse(token)` uses `https://api.meta.ai/v1`.
- `New ModelClient(provider, token, apiBase, allowLoopbackHTTP)` supports configured
  proxies and local fixtures. HTTPS is required unless explicit loopback HTTP is
  enabled; userinfo, query/fragment components and path traversal are rejected.
- `CreateResponse(model, input, tools)` sends `/responses` with `store=False`,
  `stream=False`, and encrypted reasoning included for explicit replay.
- `ContinueResponse(model, input, tools, previousResponseID, store)` allows explicit
  server-managed continuation; a previous response ID requires `store=True`.
- `Chat(model, messages, tools)` sends `/chat/completions`. This compatibility
  endpoint supports text and local function tools; it does not preserve Meta's
  reasoning across turns. Prefer Responses for agent workflows.

Models are always supplied by the caller. Discover and choose a model available
to the user's account; the library does not substitute one or claim entitlement.
Tokens remain in the client instance and are never persisted or exposed through
public fields. Requests use finite timeouts, TLS verification and disabled
redirects. Run calls on a worker thread: `URLConnection` blocks and buffers the
whole response. `TimeoutSeconds` defaults to 60; `MaxResponseBytes` defaults to
16 MiB and checks the response **after transport buffering**, not while receiving.

## Tools and replies

`FunctionTool(name, description, parameters, responsesAPI=True)` builds the
correct tool definition for the chosen dialect. Parameters must be an object
JSON schema. `TextMessage(role, text, responsesAPI=True)` builds text messages.
Pass an empty JSON array when no tools are needed.

`ModelReply` exposes `Text`, `ToolCalls`, `FinishReason`, `ResponseID`, `RawJSON`
and `AppendOutputTo(history)`. Responses replies retain every raw output item,
including encrypted reasoning, for stateless multi-turn replay. Chat replies
retain the entire original assistant message, including provider extensions.
Properties returning JSON are detached copies. For Responses `FinishReason` is
the response's `status`; for Chat it is the first choice's `finish_reason`.
Incomplete/failed replies are not reclassified as completed.

Each immutable `ModelToolCall` retains `CallID`, `ItemID`, `Name`,
`ArgumentsText`, and `RawJSON`. `ParseArguments()` requires a valid JSON object;
it does not validate the object against your tool's schema. `ResultItem(text,
responsesAPI=True)` builds a result correlated to **CallID**, not the distinct
Responses output-item ID. Append the original assistant/output items and then
one result per call before requesting the next turn. Chat validates this
correlation before sending. The host must validate schemas, apply its own tool
policy and execute tools; the package never runs a model-specified command.

`ModelAPIException` includes HTTP `Status`, sanitized `APIMessage`, `RequestID`,
`RetryAfter`, and `RateLimitRemaining`. Transport errors omit raw messages to
avoid leaking configuration or credentials. Response text is not logged. Before any external JSON reaches Buoy's parser,
a private lexical guard caps nesting at 128, rejects raw/escaped NUL and
unpaired surrogate escapes, validates strict JSON numeric spelling, and rejects
nonfinite numbers. This also guards copied raw replies and parsed tool arguments.
A literal escaped backslash followed by `u0000` remains valid text. These checks
turn known runtime-parser crash/identity-loss cases into catchable exceptions.

## SSE utility

`ModelSSEDecoder.Feed(Bytes)` accepts fragmented UTF-8 and returns `SSEEvent`
objects with `EventName`, `Data`, `ID`, and `IsDone` (`[DONE]`). CRLF, LF, CR,
comments, repeated data lines and event IDs are supported. Each completed data
payload must be valid guarded JSON or exactly `[DONE]`; malformed JSON poisons
the decoder until reset. Data remains available as its original text. `Finish()` discards
an incomplete final event. `Reset()` clears parser state after a failure.
Defaults cap lines at 1 MiB, events at 8 MiB, feeds at 32 MiB and events per feed
at 4096. Invalid UTF-8 or exceeded limits poison the decoder until reset.
The utility parses events for an external streaming transport; ModelClient
itself deliberately requests nonstreaming HTTP. It does not accumulate streamed
function arguments or infer completion from arbitrary SSE events.

## Verification and scope

Only text and local function tools are exposed. The package does not implement
provider-hosted tools, files/images/audio, background responses, cancellation,
usage aggregation or a complete autonomous agent loop. Raw response JSON remains
available for provider-specific extensions.

Primary references:
[Meta API formats](https://dev.meta.ai/docs/protocols),
[Meta Responses](https://dev.meta.ai/docs/protocols/responses),
[Meta Chat Completions](https://dev.meta.ai/docs/protocols/chat-completions),
[xAI Responses](https://docs.x.ai/developers/rest-api-reference/inference/responses),
[xAI Chat Completions](https://docs.x.ai/developers/rest-api-reference/inference/chat-completions).
