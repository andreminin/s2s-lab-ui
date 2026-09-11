# Speech-to-Speech Laboratory UI — Project Proposal

**Proposed repository:** `andreminin/speech-to-speech-laboratory-ui`  
**Alternative short name:** `speech-to-speech-ui`

**Purpose:** A browser client and experiment console for the `speech-to-speech-k8s` technology laboratory.

---

# 1. Why a Separate Repository

The S2S Kubernetes repository studies:

- Kubernetes;
- GPU scheduling;
- STT/LLM/TTS;
- Qwen3-Omni/vLLM-Omni;
- streaming;
- MCP;
- interruption;
- latency;
- observability.

The browser application is primarily a **client and measurement instrument** for those experiments.

Keeping it separate creates a clean boundary:

```text
speech-to-speech-laboratory-ui
              |
              | WebSocket / HTTP / future WebRTC
              v
speech-to-speech-k8s
              |
       S2S Runtime
              |
     STT / LLM / TTS / MCP
```

The UI can therefore evolve independently while remaining a consumer of the S2S runtime contract.

---

# 2. Proposed Repository Name

## Recommended

`andreminin/speech-to-speech-laboratory-ui`

Why:

- explicitly communicates that this is a lab;
- does not lock the project to React/Next/Vite;
- distinguishes it from a future production client;
- makes the purpose obvious when viewed without repository context.

## Short alternative

`andreminin/speech-to-speech-ui`

This is cleaner if the laboratory scope is already obvious from the organization and documentation.

## Names I would avoid

```text
voice-chat
voice-assistant
s2s-frontend
react-s2s
nextjs-s2s
speech-platform-ui
```

These either imply a production product or unnecessarily constrain implementation technology.

---

# 3. UI Is an Experiment Instrument

The first UI should not be a polished consumer voice assistant.

It should be a **S2S Laboratory Console**.

Its purpose is to make runtime behavior visible.

The UI should expose:

```text
conversation
audio
state
events
latency
backend
GPU
errors
```

---

# 4. Initial Layout

Conceptually:

```text
+-------------------------------------------------------------+
| S2S Laboratory                         Connected             |
+-----------------------------+-------------------------------+
|                             | Session                       |
|          waveform           | session_id                   |
|                             | conversation_id              |
|         [ microphone ]      | turn_id                      |
|                             | state                        |
|                             |                               |
| User transcript             | Pipeline                      |
| Assistant transcript        | VAD / STT / LLM / TTS        |
|                             | MCP                           |
|                             |                               |
+-----------------------------+-------------------------------+
| Event stream                                                |
| 08:41:02 TurnStarted                                       |
| 08:41:02 TranscriptFinal                                   |
| 08:41:03 GenerationStarted                                |
| 08:41:03 AudioStarted                                     |
+-------------------------------------------------------------+
| Latency: TTFA 536 ms | TTFT 241 ms | TTS first 113 ms      |
+-------------------------------------------------------------+
```

---

# 5. Core Features

## 5.1 Voice interaction

- microphone input;
- speaker output;
- recording indicator;
- audio waveform;
- start/stop interaction;
- interruption indication.

## 5.2 Conversation

Show:

- user speech/transcript;
- partial transcript;
- final transcript;
- assistant response;
- tool calls;
- tool results;
- errors.

## 5.3 State-machine view

Display:

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

Highlight the current state.

This is useful for debugging the runtime state machine rather than merely showing a chatbot conversation.

## 5.4 Event inspector

Display the runtime event stream:

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

Allow filtering by:

```text
audio
transcript
generation
tool
state
error
```

The UI should never become the canonical event schema. It consumes the runtime schema.

---

# 6. Latency Instrumentation

Show at least:

```text
speech_end → first audio
speech_end → first audible response
STT partial
STT final
LLM TTFT
LLM final
TTS first audio
TTS complete
tool latency
total turn
```

The UI should make latency visible during experiments.

---

# 7. Experiment Metadata

Expose useful runtime metadata:

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

This makes screenshots and recordings of the UI useful as experiment evidence.

---

# 8. Transport Abstraction

The UI should not encode assumptions about the backend transport.

Conceptually:

```text
S2S Client
   |
   +-- WebSocket adapter
   |
   +-- HTTP adapter where useful
   |
   +-- WebRTC adapter (future)
```

The UI should initially use the existing WebSocket path.

Do not implement WebRTC merely because the abstraction allows it.

---

# 9. Event Contract

The canonical event contract belongs in the S2S runtime project.

The UI should consume generated or validated client types.

Initial implementation can use:

```text
versioned JSON events
```

If the transport experiment later justifies protobuf/gRPC, the UI should adapt to the same logical event model.

---

# 10. Suggested Technology Direction

The repository should remain technology-neutral initially.

A modern TypeScript browser application is a reasonable candidate.

Possible structure:

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

The framework itself should be chosen for the experiment, not treated as an architectural commitment.

---

# 11. What the UI Should Not Own

The browser should not own:

- conversational MCP execution;
- tool authorization;
- model-specific orchestration;
- turn state as the source of truth;
- durable session state;
- production security policy;
- inference routing.

The browser is a client and observation surface.

The S2S Runtime owns conversational orchestration.

---

# 12. Relationship to the S2S Laboratory

```text
+-----------------------------+
| S2S Laboratory UI           |
|                             |
| Voice interaction           |
| Conversation                |
| Event inspector             |
| Latency                     |
| Experiment metadata         |
+--------------+--------------+
               |
               | runtime event contract
               |
+--------------v--------------+
| S2S Runtime                |
|                             |
| session / turn / VAD        |
| barge-in / MCP / telemetry  |
+--------------+--------------+
               |
       +-------+-------+
       |               |
   inference          MCP
       |               |
 STT/LLM/TTS       tools
```

---

# 13. Repository Boundary

Recommended split:

```text
andreminin/
├── speech-to-speech-k8s/
│   ├── runtime/
│   ├── deployments/
│   ├── experiments/
│   ├── docs/
│   └── benchmark/
│
└── speech-to-speech-laboratory-ui/
    ├── src/
    ├── public/
    ├── docs/
    └── experiments/
```

The backend repository owns:

- runtime semantics;
- event schema;
- deployment;
- inference experiments;
- benchmark methodology.

The UI repository owns:

- browser client;
- visualization;
- interaction;
- client-side audio;
- event inspection;
- experiment presentation.

---

# 14. UI Experiment Roadmap

## UI-E01 — Minimal client

```text
microphone
   ↓
WebSocket
   ↓
S2S Runtime
   ↓
speaker
```

Deliverable:

- working voice interaction.

## UI-E02 — Conversation console

Add:

- transcript;
- state;
- connection/session information.

Deliverable:

- usable laboratory console.

## UI-E03 — Event inspector

Add:

- live event stream;
- event filtering;
- event payload inspection.

Deliverable:

- runtime debugging instrument.

## UI-E04 — Latency view

Add:

- TTFA;
- TTFT;
- STT/TTS/tool timings;
- turn timeline.

Deliverable:

- experiment observation surface.

## UI-E05 — Backend metadata

Add:

- model;
- GPU;
- CUDA;
- driver;
- runtime revision;
- transport.

Deliverable:

- reproducible experiment screenshots/records.

## UI-E06 — Optional transport experiments

Only if required:

```text
WebSocket
WebRTC
```

---

# 15. Laboratory Design Principle

The UI should answer:

> "What is the S2S system doing right now, and what did this experiment actually measure?"

It should not try to answer:

> "How do we make this look like a production voice assistant?"

The first objective directly supports the laboratory.

The second would pull the project toward a different scope.

---

# 16. Recommendation

Create:

```text
andreminin/speech-to-speech-laboratory-ui
```

and keep:

```text
andreminin/speech-to-speech-k8s
```

as the backend/runtime/Kubernetes laboratory.

The UI should start small and become progressively more useful as an **experimental instrument**, especially around event inspection, latency, state transitions, and hardware/runtime metadata.
