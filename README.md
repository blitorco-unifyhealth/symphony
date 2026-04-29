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

Symphony’s Elixir worker runs one subprocess per issue: whatever shell command you set as
`codex.command` in `WORKFLOW.md` (see [elixir/README.md](elixir/README.md)). The supported,
tested path is **OpenAI Codex in [App Server mode](https://developers.openai.com/codex/app-server/)**:
JSON-RPC 2.0 over stdio for threads and turns (`elixir/lib/symphony_elixir/codex/app_server.ex`,
[`SPEC.md`](SPEC.md)).

The sections below separate **Claude** (model choice inside the Codex process) from **Cursor Agent**
(Cursor’s own CLI), because they are configured in different places even though both can involve
Anthropic models in your organization.

### Claude integration (via Codex App Server)

Symphony does not bundle Anthropic SDKs. **Claude** (or any other model your Codex install supports)
shows up only through **how you start `app-server`**: the full `codex.command` string in `WORKFLOW.md`
is executed on the worker, so flags, config files, and environment variables are whatever your Codex
build expects.

Concrete pattern from the Elixir docs: pass model selection into Codex with `--config` on the same
line as `app-server`:

```yaml
codex:
  command: "$CODEX_BIN --config 'model=\"gpt-5.5\"' app-server"
```

Swap the `model=...` value (and any provider/auth setup on the machine) for the Claude-capable
configuration your Codex release documents. Unattended orchestration flows often also rely on
optional tools such as `linear_graphql` (see [elixir/README.md](elixir/README.md)); confirm those
still work after changing model or provider.

### Cursor Agent (`cursor` CLI)

**Cursor Agent** is Cursor’s headless/scriptable interface ([Headless CLI](https://cursor.com/docs/cli/headless),
[CLI overview](https://cursor.com/docs/cli/overview)). You typically run it inside a Git checkout,
for example:

```bash
cursor agent --print --force "your prompt here"
```

That is **not** the same protocol as Codex App Server. Cursor uses its own session, permissions,
and model picker (including Claude when enabled in Cursor). Repository automation that lives in
**Cursor rules and Agent Skills** (for example under `.cursor/` or paths described in Cursor’s docs)
applies to Cursor-driven sessions, not automatically to Symphony’s subprocess unless you redesign
the boundary.

### Using Cursor Agent with Symphony

This repo’s orchestration is validated against `codex app-server`. Pointing `codex.command` at
`cursor agent` (or any other binary) is only safe when that binary implements the **same App Server
JSON-RPC session** the Elixir worker already speaks. Without such a bridge, treat **Symphony +
Codex** and **Cursor Agent in a checkout** as two integration surfaces:

| Surface | What you configure | Typical use |
| --- | --- | --- |
| Symphony worker | `codex.command`, `WORKFLOW.md`, Codex skills under `.codex/` | Long-running issue workspaces, Linear-driven runs |
| Cursor Agent | Cursor CLI/IDE, Cursor rules and skills | Interactive or scripted edits in a repo clone |

If you build or adopt a bridge, revalidate thread startup, tool calls (including optional
`linear_graphql`), and turn completion before relying on it in unattended workflows.

---

## License

This project is licensed under the [Apache License 2.0](LICENSE).
