# Call model APIs with typed tools and streaming replies

Source-only Buoy clients for **Grok/xAI** and **Meta Muse** with explicit model
selection, function-tool calls and conversation replay. Both providers document
Responses and OpenAI-compatible Chat Completions endpoints. This package uses
their published wire formats; it does not install a coding-agent harness.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add modelkit --git https://github.com/BuoyBay/modelkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## Example

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

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer modelkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```

The generated agent files and editor settings stay local and are ignored by Git.

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
