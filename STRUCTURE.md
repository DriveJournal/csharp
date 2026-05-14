# Solution Structure

> The component-level architecture for DriveJournal. Locked in before any `dotnet new` commands are run.

---

## Folder layout

```
csharp/
├─ src/        # production projects
├─ tests/      # test projects
└─ CarTelemetry.sln
```

---

## Project list

| Project name | src or tests? | Type | Depends on | Phase introduced |
|---|---|---|---|---|
| `CarTelemetry.Domain` | src | class library | (nothing) | 1 |
| `CarTelemetry.Infrastructure` | src | class library | Domain | 2 |
| `CarTelemetry.Worker` | src | worker service | Domain + Infrastructure (DI-only at startup) | 3 |
| `CarTelemetry.Api` | src | ASP.NET Core web | Domain + Infrastructure (DI-only at startup) | 4 |
| `CarTelemetry.Web` | src | Blazor WebAssembly | Domain | 6 |
| `CarTelemetry.Domain.Tests` | tests | xUnit | Domain | 1 |
| `CarTelemetry.Infrastructure.Tests` | tests | xUnit | Domain + Infrastructure | 2 |
| `CarTelemetry.Api.Tests` | tests | xUnit | Domain + Api | 4 |
| `CarTelemetry.Web.Tests` | tests | xUnit | Domain + Web | 6 |

**Total: 9 projects** (5 production, 4 test).

---

## Dependency graph

```
                    ┌──────────────────────────────────┐
                    │     CarTelemetry.Domain          │
                    │     (entities, interfaces,       │
                    │      pure business logic)        │
                    │                                  │
                    │     DEPENDS ON NOTHING           │
                    └──────────────────────────────────┘
                          ▲       ▲       ▲       ▲
                          │       │       │       │
                          │       │       │       │
              ┌───────────┘       │       │       └────────────┐
              │                   │       │                    │
   ┌──────────┴──────────┐  ┌─────┴────┐ ┌┴────┐  ┌────────────┴───────┐
   │  Infrastructure     │  │  Worker  │ │ Api │  │  Web (Blazor WASM) │
   │  (EF Core, DBC,     │  │  (MQTT   │ │     │  │                    │
   │   external systems) │  │  ingest) │ │     │  │                    │
   └─────────────────────┘  └──────────┘ └─────┘  └────────────────────┘
              ▲                   │        │
              │  (DI-only at      │        │
              │   startup, not    └────┬───┘
              │   in business code)    │
              └────────────────────────┘
```

**Key rule:** Domain is referenced by every other project. Domain references nothing.

Worker and Api reference Infrastructure **only in `Program.cs`** to wire up dependency injection. Their business code talks only to interfaces in Domain.

---

## Self-check answers

1. Does `CarTelemetry.Domain` depend on anything outside the .NET base class library? → **No.**
2. Does any production project depend on a test project? → **No.**
3. Does `CarTelemetry.Infrastructure` ever appear in another production project's dependency list? → **Only at the DI wiring layer in Worker and Api's `Program.cs`. Business logic in any project only knows about interfaces in Domain.**
4. Does the Web (Blazor) project depend on anything other than Domain? → **No — it gets data over HTTP from Api at runtime; no compile-time dependency on the backend.**
5. How many test projects, and does each test project depend on exactly the one production project it tests? → **Four tests. Each depends on Domain (always) plus the one production project it tests.**

---

## A note on shared DTOs

When we get to Phase 4 (the Api), we'll likely want DTOs — data transfer objects — that both Api and Web need to agree on. There are two options:

1. **Put them in Domain.** Simpler. Means Web depends only on Domain.
2. **Add a `CarTelemetry.Shared` project.** Keeps Domain pure of "wire format" concerns.

We'll defer this decision until Phase 4 and revisit. For now, Domain is the placeholder.


## General Notes (for my reference)

2. What a "project" is
In .NET, a project is a folder containing related classes plus a single .csproj file that describes how they compile together. When you run dotnet build, the project compiles into one .dll file (a "library") or .exe file (a runnable program).
A solution (.sln file) is just a list of projects that belong together. Big systems are made of many small projects, each focused on one job, that reference each other to share code. When project A references project B, project A can use the classes inside B.
Think of it like this:

A class is a file
A project is a folder of related files that compile to one DLL
A solution is a list of projects that ship together

3. Why we split code into layers
Here's the big idea behind clean architecture, in one paragraph:
A piece of software has two kinds of concerns: what it is about, and how it talks to the outside world. What your app is about is permanent — a "drive session" is a drive session whether you save it to PostgreSQL or to a text file or to a NoSQL database in the cloud. How it talks to the outside world is changeable — you might swap databases, swap message brokers, swap UI frameworks. You want the permanent stuff to be cleanly separable from the changeable stuff, so that changing the changeable stuff doesn't force you to rewrite the permanent stuff.
This is why we put different concerns into different projects. The naming convention used by most .NET teams (and the project plan you have) is:

Domain — the permanent stuff: what your app is about
Infrastructure — the changeable stuff: how data gets in and out (database, MQTT, file decoding)
Worker — background processes that run continuously (your MQTT subscriber)
Api — the HTTP endpoints other systems (or your own frontend) call
Web — the user interface (your Blazor frontend)