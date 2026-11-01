---
title: "modelkit typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["ModelAPIException", "ModelClient", "ModelProvider", "ModelReply", "ModelSSEDecoder", "ModelToolCall", "SSEEvent"]
dependencies: []
symbols: ["ModelAPIException", "ModelClient", "ModelProvider", "ModelReply", "ModelSSEDecoder", "ModelToolCall", "SSEEvent"]
related: ["api.md", "getting-started.md", "index.md"]
---

# modelkit typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## ModelProvider

`Public Enum ModelProvider As Integer`

### Values

| Name | Type | Value / Default | Description |
| --- | --- | --- | --- |
| `Grok` | `ModelProvider` | `1` | — |
| `MetaMuse` | `ModelProvider` | `2` | — |

## ModelAPIException

`Public Class ModelAPIException Inherits Exception`

The error message excludes raw response bodies; metadata is sanitized before exposure.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Status` | `Integer` | No | No | — | — |
| `RequestID` | `String` | No | No | — | — |
| `RetryAfter` | `String` | No | No | — | — |
| `RateLimitRemaining` | `String` | No | No | — | — |
| `APIMessage` | `String` | No | No | — | — |

## ModelToolCall

`Public Class ModelToolCall`

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `CallID` | `String` | Yes | No | — | — |
| `ItemID` | `String` | Yes | No | — | — |
| `Name` | `String` | Yes | No | — | — |
| `ArgumentsText` | `String` | Yes | No | — | — |
| `RawJSON` | `JSONItem` | Yes | No | — | — |

### ParseArguments

`Public Function ParseArguments() As JSONItem`

Arguments are untrusted: parse and validate against the host's own schema before execution.

No parameters.

#### Returns

| Type |
| --- |
| `JSONItem` |

### ResultItem

`Public Function ResultItem(content As String, responsesAPI As Boolean = True) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `content` | `String` | `ByVal` | Required |
| `responsesAPI` | `Boolean` | `ByVal` | `True` |

#### Returns

| Type |
| --- |
| `JSONItem` |

## ModelReply

`Public Class ModelReply`

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Text` | `String` | Yes | No | — | — |
| `ToolCalls` | `Array Of ModelToolCall` | Yes | No | — | — |
| `FinishReason` | `String` | Yes | No | — | — |
| `ResponseID` | `String` | Yes | No | — | — |
| `RawJSON` | `JSONItem` | Yes | No | — | — |

### AppendOutputTo

`Public Sub AppendOutputTo(history As JSONItem)`

Appends every raw output item, including reasoning replay, or the whole assistant chat message.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `history` | `JSONItem` | `ByVal` | Required |

Returns: no value.

## ModelKit functions

### FunctionTool

`Public Function FunctionTool(name As String, description As String, parameters As JSONItem, responsesAPI As Boolean = True) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `name` | `String` | `ByVal` | Required |
| `description` | `String` | `ByVal` | Required |
| `parameters` | `JSONItem` | `ByVal` | Required |
| `responsesAPI` | `Boolean` | `ByVal` | `True` |

#### Returns

| Type |
| --- |
| `JSONItem` |

### TextMessage

`Public Function TextMessage(role As String, content As String, responsesAPI As Boolean = True) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `role` | `String` | `ByVal` | Required |
| `content` | `String` | `ByVal` | Required |
| `responsesAPI` | `Boolean` | `ByVal` | `True` |

#### Returns

| Type |
| --- |
| `JSONItem` |

### ParseModelReply

`Public Function ParseModelReply(raw As JSONItem, responsesAPI As Boolean = True) As ModelReply`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `raw` | `JSONItem` | `ByVal` | Required |
| `responsesAPI` | `Boolean` | `ByVal` | `True` |

#### Returns

| Type |
| --- |
| `ModelReply` |

## ModelClient

`Public Class ModelClient`

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `TimeoutSeconds` | `Integer` | No | No | `60` | — |
| `MaxResponseBytes` | `Integer` | No | No | `16777216` | — |
| `Provider` | `ModelProvider` | Yes | No | — | — |
| `BaseURL` | `String` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(provider As ModelProvider, token As String, apiBase As String, allowLoopbackHTTP As Boolean = False)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `provider` | `ModelProvider` | `ByVal` | Required |
| `token` | `String` | `ByVal` | Required |
| `apiBase` | `String` | `ByVal` | Required |
| `allowLoopbackHTTP` | `Boolean` | `ByVal` | `False` |

Returns: no value.

### ForGrok

`Public Shared Function ForGrok(token As String) As ModelClient`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `token` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `ModelClient` |

### ForMetaMuse

`Public Shared Function ForMetaMuse(token As String) As ModelClient`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `token` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `ModelClient` |

### CreateResponse

`Public Function CreateResponse(model As String, input As JSONItem, tools As JSONItem) As ModelReply`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `model` | `String` | `ByVal` | Required |
| `input` | `JSONItem` | `ByVal` | Required |
| `tools` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `ModelReply` |

### ContinueResponse

`Public Function ContinueResponse(model As String, input As JSONItem, tools As JSONItem, previousResponseID As String, store As Boolean = False) As ModelReply`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `model` | `String` | `ByVal` | Required |
| `input` | `JSONItem` | `ByVal` | Required |
| `tools` | `JSONItem` | `ByVal` | Required |
| `previousResponseID` | `String` | `ByVal` | Required |
| `store` | `Boolean` | `ByVal` | `False` |

#### Returns

| Type |
| --- |
| `ModelReply` |

### Chat

`Public Function Chat(model As String, messages As JSONItem, tools As JSONItem) As ModelReply`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `model` | `String` | `ByVal` | Required |
| `messages` | `JSONItem` | `ByVal` | Required |
| `tools` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `ModelReply` |

## SSEEvent

`Public Class SSEEvent`

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `EventName` | `String` | No | No | — | — |
| `Data` | `String` | No | No | — | — |
| `ID` | `String` | No | No | — | — |
| `IsDone` | `Boolean` | No | No | — | — |

## ModelSSEDecoder

`Public Class ModelSSEDecoder`

Incremental SSE parsing for external streaming transports; HTTP clients above remain buffered.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `IsFailed` | `Boolean` | Yes | No | — | — |
| `IsDone` | `Boolean` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(maxLineBytes As Integer = 1048576, maxEventBytes As Integer = 8388608)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `maxLineBytes` | `Integer` | `ByVal` | `1048576` |
| `maxEventBytes` | `Integer` | `ByVal` | `8388608` |

Returns: no value.

### Reset

`Public Sub Reset()`

No parameters.

Returns: no value.

### Feed

`Public Function Feed(chunk As Bytes) As Array Of SSEEvent`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `chunk` | `Bytes` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Array Of SSEEvent` |

### Finish

`Public Sub Finish()`

EOF discards an incomplete event, as required by SSE. Invalid UTF8 still fails.

No parameters.

Returns: no value.
