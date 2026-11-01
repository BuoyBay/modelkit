---
title: "ModelKit getting started"
summary: "Build an offline function-tool payload and parse a synthetic Responses reply."
category: "Guides"
order: 10
keywords: ["offline", "schema", "reply", "history", "functiontool"]
related: ["index.md", "api.md", "typed-api.md"]
---

# Getting started

Copy `modelkit-0.1.0.buipkg` into the consumer's `packages/` and `Import modelkit`. The package is source-only and uses Buoy's HTTP runtime.

## Build and parse locally

This complete example builds a function-tool definition and user message, then parses a synthetic Responses reply. It uses no credentials, account, network, or inference.

```buoy
Import modelkit

' Offline payload example; create a ModelClient and call CreateResponse on a worker to send it.
Sub Main()
    Dim schema As JSONItem = Json.Parse("{\"type\":\"object\",\"properties\":{\"itemId\":{\"type\":\"string\"}},\"required\":[\"itemId\"]}")
    Print Json.Stringify(FunctionTool("project_read_item", "Read a project item's unsaved buffer", schema))
    Print Json.Stringify(TextMessage("user", "Explain the active document"))
    Dim fixture As JSONItem = Json.Parse("{\"id\":\"offline-example\",\"status\":\"completed\",\"output\":[{\"type\":\"message\",\"role\":\"assistant\",\"content\":[{\"type\":\"output_text\",\"text\":\"Agent replies and tools use the same project model.\"}]}]}")
    Print ParseModelReply(fixture).Text
End Sub
```

## Send an explicit request

Create `ModelClient.ForGrok(token)` or `ModelClient.ForMetaMuse(token)` with a token acquired through your host's credential flow. Discover/select a model available to that account. Call `CreateResponse(model, input, tools)` on a worker with JSON arrays for input and tools; pass an empty array when no tools are needed. Do not log tokens or raw provider responses.

The configured constructor supports proxies and loopback fixtures but validates origins: userinfo, query/fragment parts, and traversal reject. `TimeoutSeconds` defaults to 60. `MaxResponseBytes` defaults to 16 MiB and is enforced after transport buffering, not during download.

## Continue a tool conversation

After a reply, call `reply.AppendOutputTo(history)` to preserve every original assistant/output item. Validate each `ModelToolCall.ParseArguments()` result against your schema and policy. Execute only approved operations. Append one `call.ResultItem(resultText)` per tool call; correlation uses `CallID`, which differs from a Responses output item's `ItemID`.

Send the full history with another stateless Responses request, or deliberately opt into `ContinueResponse` server-managed continuation with `store=True`. Chat Completions must replay the original assistant message and corresponding tool results; it validates their correlation before sending.

The separate `ModelSSEDecoder` parses events from a transport you supply. `ModelClient` itself requests nonstreaming HTTP and does not combine streamed function arguments.

Continue with the [API reference](api.md), or return to [ModelKit](index.md).
