# Symphony

Symphony turns project work into isolated, autonomous implementation runs, allowing teams to manage
work instead of supervising coding agents.

[![Symphony demo video preview](.github/media/symphony-demo-poster.jpg)](.github/media/symphony-demo.mp4)

_In this [demo video](.github/media/symphony-demo.mp4), Symphony monitors a Linear board for work and spawns agents to handle the tasks. The agents complete the tasks and provide proof of work: CI status, PR review feedback, complexity analysis, and walkthrough videos. When accepted, the agents land the PR safely. Engineers do not need to supervise Codex; they can manage the work at a higher level._

> [!WARNING]
> Symphony is a low-key engineering preview for testing in trusted environments.

## Running Symphony

### Requirements

Symphony works best in codebases that have adopted
[harness engineering](https://openai.com/index/harness-engineering/). Symphony is the next step --
moving from managing coding agents to managing work that needs to get done.

### Option 1. Make your own

Tell your favorite coding agent to build Symphony in a programming language of your choice:

> Implement Symphony according to the following spec:
> https://github.com/openai/symphony/blob/main/SPEC.md

### Option 2. Use our experimental reference implementation

Check out [elixir/README.md](elixir/README.md) for instructions on how to set up your environment
and run the Elixir-based Symphony implementation. You can also ask your favorite coding agent to
help with the setup:

> Set up Symphony for my repository based on
> https://github.com/openai/symphony/blob/main/elixir/README.md

## Agents: Codex, Cursor CLI, and Claude

The subsections below cover **Claude integration** (through the Codex process you configure) and
**Cursor Agent** (the `cursor` CLI) in relation to Symphony’s reference worker.

Symphony’s reference worker runs whatever shell command you set under `codex.command` in `WORKFLOW.md`
(see [elixir/README.md](elixir/README.md)). Today that command is expected to be **OpenAI Codex in
[App Server mode](https://developers.openai.com/codex/app-server/)**: the Elixir client speaks
JSON-RPC 2.0 over stdio to drive threads and turns (`elixir/lib/symphony_elixir/codex/app_server.ex`,
[`SPEC.md`](SPEC.md)).

### Claude and other models

Symphony does not embed Anthropic or other vendor SDKs. To use **Claude** (or another model) for
orchestrated turns, configure the **Codex** process you launch—typically via `codex.command` flags
such as `--config` and the model fields your Codex build supports—so the same App Server session
talks to the provider you intend. If you use **Cursor**, Claude and other models are available
inside Cursor’s own agent and CLI products; that is separate from Symphony’s subprocess unless you
point `codex.command` at a binary that preserves the Codex App Server wire protocol.

### Cursor Agent (`cursor` CLI)

Cursor ships a **CLI agent** for headless and scripted use ([Headless CLI](https://cursor.com/docs/cli/headless)).
That is useful for automation alongside or outside Symphony. This repository’s orchestration path
is tested with `codex app-server`; swapping in `cursor agent` (or similar) as `codex.command` is
only viable when the replacement implements the **same App Server JSON-RPC session** the worker
already expects. Treat any such swap as a custom integration and validate thread startup, tool
calls (including optional `linear_graphql`), and turn completion against your workflow before
depending on it unattended.

---

## License

This project is licensed under the [Apache License 2.0](LICENSE).
