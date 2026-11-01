# Estudo-MCP — Model Context Protocol in Practice

![Sprint](https://img.shields.io/badge/Sprints-8%2F8-success)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![MCP SDK](https://img.shields.io/badge/MCP-python--sdk-000000)
![uv](https://img.shields.io/badge/uv-package%20manager-DE5FE9?logo=uv&logoColor=white)
![Pytest](https://img.shields.io/badge/pytest-tested-0A9EDC?logo=pytest&logoColor=white)

🇧🇷 [Leia em português](README.pt-BR.md)

A guided, hands-on study of the **Model Context Protocol (MCP)**, built from scratch over 8 incremental sprints. Each sprint delivers a working, tested MCP Server — not just documentation examples — covering the entire protocol: Tools, Resources, Prompts, Sampling, Roots, remote transport and authentication.

---

## Table of Contents

- [About the Project](#about-the-project)
- [Sprint Progress](#sprint-progress)
- [What Was Built](#what-was-built)
- [Real Bugs Found Along the Way](#real-bugs-found-along-the-way)
- [Repository Structure](#repository-structure)
- [How to Run](#how-to-run)
- [Methodology](#methodology)
- [References](#references)

---

## About the Project

The [Model Context Protocol](https://github.com/modelcontextprotocol) is the open standard that lets AI applications (Hosts) connect to external tools and data sources (Servers) in a uniform way — removing the need for a proprietary integration for every application × tool combination.

This repository documents a from-scratch study of MCP that goes beyond reading the specification: each sprint produces a **real MCP server, with automated tests and end-to-end validation** (via a real `ClientSession`, MCP Inspector, or both), progressively covering:

`Tools` → `Resources & Prompts` → `Inspection/debugging` → `Custom Client` → `Sampling & Roots` → `HTTP Transport + Auth` → `Final project`

---

## Sprint Progress

| # | Sprint | Deliverable | Status |
|---|--------|------------|:---:|
| 1 | [MCP fundamentals + environment setup](docs/sprints/sprint-01.md) | `uv` environment + validated official SDK | ✅ |
| 2 | [First MCP Server — Tools](docs/sprints/sprint-02.md) | Task manager ([`servers/sprint-02-tools-server`](servers/sprint-02-tools-server)) | ✅ |
| 3 | [Resources & Prompts](docs/sprints/sprint-03.md) | Notes library ([`servers/sprint-03-resources-prompts`](servers/sprint-03-resources-prompts)) | ✅ |
| 4 | [Testing with MCP Inspector](docs/sprints/sprint-04.md) | Visual and `--cli` validation | ✅ |
| 5 | [Custom MCP Client + real Host](docs/sprints/sprint-05.md) | Generic client ([`servers/sprint-05-custom-client`](servers/sprint-05-custom-client)) | ✅ |
| 6 | [Sampling & Roots](docs/sprints/sprint-06.md) | Server implementing both primitives ([`servers/sprint-06-sampling-roots`](servers/sprint-06-sampling-roots)) | ✅ |
| 7 | [HTTP Transport + Authentication](docs/sprints/sprint-07.md) | `streamable-http` server with Bearer token ([`servers/sprint-07-http-auth`](servers/sprint-07-http-auth)) | ✅ |
| 8 | [Final project](docs/sprints/sprint-08.md) | Git repository assistant ([`servers/sprint-08-final-project`](servers/sprint-08-final-project)) | ✅ |

Each sprint in `docs/sprints/` follows a fixed format: **objective, concepts taught, architectural decisions, validation questions, code review (with a grade) and interview questions** — see [Methodology](#methodology).

---

## What Was Built

| Server | MCP primitives | Highlight |
|---|---|---|
| `sprint-02-tools-server` | Tools | 4 Tools with in-memory state, unit tests + smoke test via a real Client |
| `sprint-03-resources-prompts` | Resources (static + template) and Prompts | Two Prompt return forms (`str` and `list[Message]`) |
| `sprint-05-custom-client` | Generic Client | Works against any `stdio` server; 4 actions (list/call/read/prompt) |
| `sprint-06-sampling-roots` | Sampling and Roots | Server → Client → Server round trip validated with stub callbacks |
| `sprint-07-http-auth` | HTTP Transport + Auth | Custom `TokenVerifier`, tested with missing/invalid/valid tokens |
| `sprint-08-final-project` | Tools + Resources + Prompts | Git repository assistant, tested against this repo's real history |

All servers are built with the [official Python SDK](https://github.com/modelcontextprotocol/python-sdk) (`FastMCP`) and have their own test suite (`uv run pytest`).

---

## Real Bugs Found Along the Way

Building the exercises against real Clients (rather than only reading the documentation) surfaced concrete problems, which were fixed during development:

| Sprint | Bug | Cause | Fix |
|---|---|---|---|
| [5](docs/sprints/sprint-05.md) | Traceback when propagating a Tool error | `SystemExit` raised inside `async with` breaks the `anyio.TaskGroup` cleanup | Return the exit code and call `SystemExit` only outside the async scope |
| [7](docs/sprints/sprint-07.md) | Silent `DeprecationWarning` | HTTP client API was renamed (`streamablehttp_client` → `streamable_http_client`) | Migrated to the current API, with an explicit `httpx.AsyncClient` |
| [7](docs/sprints/sprint-07.md) | Orphan process holding the port | `uv run` on Windows spawns a child process that survives `Popen.terminate()` | `taskkill /PID <pid> /T /F` (kills the whole process tree) |
| [8](docs/sprints/sprint-08.md) | `git log`/`git show` hanging indefinitely | The `git` subprocess inherited the server's `stdin`, which under `stdio` is the MCP protocol pipe | Explicit `stdin=subprocess.DEVNULL` on every `subprocess.run` |
| [8](docs/sprints/sprint-08.md) | Resource template not matching nested paths | The SDK generates `[^/]+` for each `{param}` — it does not accept `/` | Slashes encoded as `%2F` in the URI, decoded in the Server |

---

## Repository Structure

```
MCP/
├── README.md
├── docs/
│   └── sprints/                        # 1 document per sprint
│       ├── sprint-01.md … sprint-08.md
└── servers/                             # 1 independent uv project per sprint
    ├── sprint-01-hello-mcp/
    ├── sprint-02-tools-server/
    ├── sprint-03-resources-prompts/
    ├── sprint-05-custom-client/
    ├── sprint-06-sampling-roots/
    ├── sprint-07-http-auth/
    └── sprint-08-final-project/
```

Each folder in `servers/` is a self-contained [`uv`](https://docs.astral.sh/uv/) project with its own `pyproject.toml`, `uv.lock` and virtual environment — no dependencies are shared between sprints.

---

## How to Run

Prerequisite: [`uv`](https://docs.astral.sh/uv/) installed (it resolves the Python version automatically).

```bash
# Enter a specific sprint's project
cd servers/sprint-02-tools-server

# Install dependencies
uv sync

# Run the tests
uv run pytest -v

# Start the server (stdio transport, or streamable-http in Sprint 7)
uv run server.py
```

To explore a server interactively without writing Client code, use the [MCP Inspector](https://github.com/modelcontextprotocol/inspector) (see Sprint 4):

```bash
npx @modelcontextprotocol/inspector uv run server.py
```

---

## Methodology

Each sprint in `docs/sprints/` follows the same structure:

1. **Objective** — what the sprint delivers
2. **Concepts taught** — what needs to be understood before the exercise
3. **Architectural decisions** — design choices, with discarded alternatives and the reasoning
4. **Validation questions** — check conceptual understanding, answered in my own words
5. **Code review** — grade and concrete feedback on the code produced in the sprint
6. **Interview questions** — technical-interview-style questions, with a grade and identified gaps

---

## References

- [Model Context Protocol — specification and official SDKs](https://github.com/modelcontextprotocol)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- [uv — Python project manager](https://docs.astral.sh/uv/)
