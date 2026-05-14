# DriveJournal

> A C#/.NET system that ingests CAN bus telemetry from a Honda Civic via a Raspberry Pi, persists drive sessions, and presents them on a live dashboard and historical trip journal.

---

## 1. What this is (the elevator pitch)

> _Write 2–3 sentences a non-technical friend would understand. What problem does this solve for whoever uses it? Avoid jargon — no "telemetry pipeline" or "domain layer." Just the human story._

A user wants to understand how their car is performing during drives, real time and historically. 
By tapping into the brain of the car, we can extract data about speed, fuel efficiency and more.
Thus, we can store the data and/or present it in real time in a way to help user drive habits and understand their car better.

---

## 2. Who uses it

> _List each "persona" — even if it's just one (you, the driver). For each, write one sentence about what they care about most. If you imagine a future where a friend uses it too, add them as a second persona._

- Driver: cares about understanding car performance and driving habits.

---

## 4. Requirements — what the system should do

> _Write 5–10 user stories in the form "As a `<persona>`, I want to `<do something>` so that `<reason>`." Order them roughly by importance. Be honest: some stories will be must-haves (you'd ditch the project without them) and some will be nice-to-haves. Mark them._

**Must-have**

- I want to track my cars performance real time.
- I want to view my cars performance history.
- I want to see my history in a way that helps me understand my driving habits and car performance.
- I want to compare historical drives to see trends over time.
- I want to compare historical drives to real time to see trends over time.
- I want to set goals for my driving habits and see how I'm doing against them.
- I want to set destinations and save routes and see how I performed on those routes over time.

**Nice-to-have**

- Customize dashboard with different widgets.
- Leaderboard comparing my performance to other drivers (friends or anonymous).

---

## 5. How data flows through the system

> _Describe the journey of one CAN frame from the moment it leaves the engine until it appears on a dashboard. Use plain English, one step per line. Don't worry about project names yet — just verbs and nouns ("the Pi reads bytes from the CAN bus", "those bytes are wrapped in JSON", etc.)._

TODO. Example shape:

1. A sensor in the car emits a CAN frame on the bus.
2. The Raspberry Pi reads the frame via `python-can`.
3. The Pi wraps the raw frame in JSON and publishes to the MQTT broker.
4. The backend service subscribes to the MQTT topic and ingests the JSON frame.
5. The backend transforms the JSON into a domain event and persists it in PostgreSQL.
6. The backend also broadcasts the domain event to connected dashboard clients via SignalR.
7. The dashboard receives the event and updates the real-time performance metrics.

---

## 6. Technology choices (and why)

> _For each major piece of the stack, write one line on what it is and one line on why you picked it. This is the doc you'll thank yourself for in six months when you wonder "why on earth did I choose X?"_

- **C# / .NET 8** - Standard, full-featured language and framework with great support for all layers of the stack, from hardware interaction to web development.
- **PostgreSQL** — Easy organization of relational data, strong .NET support, and good local development experience with Docker.
- **MQTT (Mosquitto / EMQX)** — Fast and lightweight pub/sub protocol ideal for real-time telemetry, with good .NET client libraries.
- **Blazor WebAssembly** — Modern front end framework that allows us to write C# end-to-end, with good support for real-time updates via SignalR.
- **SignalR** — Standard for real-time communication in .NET, perfect for pushing live updates to the dashboard.
- **Docker** — Easy to containerize the backend and MQTT broker for consistent local development and future deployment.

---

## 7. Out of scope (for now) - Future plans

> _Explicitly list the things this project is **not** going to do. This is where senior engineers shine — knowing what to NOT build is harder than knowing what to build. Examples: "multiple cars", "public sharing", "mobile app", "secure deployment to the internet"._

- Apache Kafka or other complex event streaming platforms for ingesting CAN frames.
- Kubernetes or other complex orchestration tools for multiple containers.

---

## 8. Open questions

> _What don't you know yet? What decisions are you deferring? Listing these up front prevents them from being silent assumptions that bite you later._

- How c# development works with .net 8 and blazor web assembly.
- How to integrate backend and frontend development.

---

## 9. How to run this locally

> _(Fill in once Phase 0 step 4 is done. For now, leave a placeholder.)_

TODO

---

_This document is the product brief. The component-level architecture (project layout, dependencies) lives in [`STRUCTURE.md`](./STRUCTURE.md) — written **after** this brief is approved._
