# S2S Lab - UI

A small browser client and experiment console for my **speech-to-speech home lab**.

This is not meant to be a polished voice-assistant frontend. The idea is to have a useful window into what the S2S runtime is actually doing: audio, turns, events, latency, tools, state, backend and GPU information - a **lab instrument with a microphone**.

## Why a separate UI repo?

The backend lab lives in [`andreminin/s2s-lab-k8s`](https://github.com/andreminin/s2s-lab-k8s) and is where I experiment with:

- Kubernetes
- consumer GPU scheduling
- STT / LLM / TTS
- Qwen3-Omni / vLLM-Omni
- streaming
- MCP
- interruption and cancellation
- latency
- observability

The browser has a different job. It is the client and measurement surface for those experiments.

```text
s2s-lab-ui
    |
    | WebSocket / HTTP / future WebRTC
    v
s2s-lab-k8s
    |
    S2S Runtime
    |
    +-- STT
    +-- LLM
    +-- TTS
    +-- MCP
```

Keeping the repositories separate also means I can change the UI without turning the runtime project into a frontend project.

During an experiment I want to be able to see things like:

```text
conversation
audio
current state
runtime events
latency
model
GPU
transport
errors
```

A normal chat UI hides most of that. For this project, those details are the interesting part.

## What I want to see

### Voice interaction

- microphone input
- speaker output
- recording state
- waveform
- start / stop
- interruption indication

### Conversation

- partial transcript
- final transcript
- assistant response
- tool calls
- tool results
- errors

### Runtime state

The current state should be visible:

```text
IDLE
LISTENING
THINKING
TOOL_CALL
SPEAKING
INTERRUPTED
FILLER
ERROR
```

This is particularly useful when debugging VAD, barge-in, cancellation, and turn transitions.

### Event stream

The UI should show the runtime event stream rather than inventing its own semantics.

For example:

```text
TurnStarted
AudioFrame
TranscriptPartial
TranscriptFinal
GenerationStarted
TextDelta
ToolCallStarted
ToolCallCompleted
AudioStarted
AudioChunk
BargeIn
TurnCompleted
```

Events should be filterable by areas such as:

```text
audio
transcript
generation
tool
state
error
```

The canonical event contract belongs to the S2S runtime, not this repository.

## Latency

Latency is one of the main reasons this UI exists.

The console should make timings visible during an experiment:

```text
speech_end -> first audio
speech_end -> first audible response
STT partial
STT final
LLM TTFT
LLM final
TTS first audio
TTS complete
tool latency
total turn
```

The useful numbers are not just averages. I care about p50/p95/p99 when the experiment calls for them.

## Experiment metadata

A result is much more useful when I know what produced it.

The UI should expose things such as:

```text
runtime revision
model
model revision
GPU node
GPU model
GPU architecture
CUDA version
driver
transport
session
turn
```

That makes a screenshot or recording of the console useful as experiment evidence instead of just being a demo screenshot.

## Transport

The first implementation uses the existing WebSocket path.

Conceptually, the client has a transport boundary:

```text
S2S Client
   |
   +-- WebSocket
   |
   +-- HTTP where useful
   |
   +-- WebRTC later, if an experiment needs it
```

The important part is not to build every transport up front.

If an HTTP-vs-gRPC or WebRTC experiment gives a reason to change the transport, the UI should adapt to the same logical S2S event model.

## What the browser does not own

The browser is deliberately kept fairly dumb.

It should **not** become the source of truth for:

- conversational turn state
- MCP execution
- tool authorization
- model-specific orchestration
- inference routing
- durable session state
- production security policy

Those belong on the runtime/backend side.

The UI consumes the runtime contract and makes its behavior visible.

## Development direction

I expect this to be a TypeScript browser application, but the framework is not the interesting architectural decision here.

A reasonable structure is:

```text
src/
  audio/
  conversation/
  events/
  latency/
  session/
  transport/
  components/
  pages/
```

The implementation can change as the experiments evolve.

## UI experiment roadmap

The rough progression is:

```text
UI-E01  microphone -> WebSocket -> runtime -> speaker

UI-E02  conversation console
        transcript + state + session

UI-E03  event inspector
        live events + filtering + payloads

UI-E04  latency view
        TTFA + TTFT + component timings + timeline

UI-E05  backend metadata
        model + GPU + CUDA + driver + revisions

UI-E06  optional transport experiments
        WebSocket / WebRTC
```

The UI should grow only when an experiment needs another observation surface.

## Relationship to the Kubernetes lab

The split is intentionally simple:

```text
s2s-lab-k8s
  runtime semantics
  event schema
  Kubernetes
  inference
  GPU experiments
  deployment
  benchmark methodology

        |
        | runtime event contract
        v

s2s-lab-ui
  browser audio
  conversation
  visualization
  event inspection
  latency
  experiment presentation
```

The UI is not the canonical implementation of the runtime protocol.

## Carrying results into Synanton

This is a home-lab project, but the experiments are meant to have a longer tail.

I plan to use the results when designing parts of the [Synanton platform](https://github.com/synanton/platform), especially:

- [Synanton Content Extractor](https://github.com/synanton/content_extractor)
- [Synanton GPU Runtime](https://github.com/synanton/platform)

The S2S experiments should provide practical input around streaming/event boundaries, runtime behavior, GPU execution, model placement, latency, and what is actually worth carrying into a larger platform.

The UI itself is not intended to become the Synanton production UI. It is mainly a way to make the experiments easier to run, understand, and compare.
