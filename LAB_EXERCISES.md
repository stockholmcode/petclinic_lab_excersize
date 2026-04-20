# AI-Assisted Development Lab

A hands-on workshop exploring AI-powered coding with Claude Code, using the PetClinic codebase as a playground.

## Before You Start

### Prerequisites

| Tool | Windows | macOS | Linux |
|------|---------|-------|-------|
| **.NET 10 SDK** | `winget install Microsoft.DotNet.SDK.Preview` | `brew install dotnet-sdk-preview` | [Install script](https://dot.net/v1/dotnet-install.sh) `--channel 10.0` |
| **Java 21+ & Maven** | `winget install Microsoft.OpenJDK.21` + `winget install Apache.Maven` | `brew install openjdk@21 maven` | `sudo apt install openjdk-21-jdk maven` |
| **Docker** | `winget install Docker.DockerDesktop` | `brew install --cask docker` | [Docker Engine docs](https://docs.docker.com/engine/install/) |
| **Git** | `winget install Git.Git` | Pre-installed | Pre-installed |
| **Claude Code** | Pre-installed for this session | Pre-installed for this session | Pre-installed for this session |
| **Editor** | VS Code + C# Dev Kit / IntelliJ | Same | Same |

You only need the SDK for the stack you choose (.NET or Java, not both).

Verify your setup:
```bash
dotnet --version    # should show 10.x
# or
java --version     # should show 21+
mvn --version

docker compose version
claude --version
```

### Pick Your Codebase

Each team picks one starting point:

- **.NET 10 Minimal API**: `git clone https://github.com/RKrogh/petclinic.git`
- **Java Spring Boot**: `git clone https://github.com/spring-projects/spring-petclinic.git`
- **SCG Classifieds API**: `git clone https://github.com/stockholmcode/classifieds-api`
- **Something else entirely**: Start a fresh project if you prefer.

---

## Exercise 0: Initialize Your AI Workspace (15 min)

**Goal:** Set up Claude Code in your project and learn how to orient. Think about it as a resource shared with the entire team for this specific project: version controlled and shared maintenance of its files.

### Tasks

1. Open your terminal in the cloned repo
2. Run `claude` to start a Claude Code session
3. Ask Claude to explore and explain the project architecture
4. Run `/init` to generate a `CLAUDE.md` file for the project
5. Review what Claude generated. Edit it if anything is wrong or missing.
6. Get the project running locally (ask Claude for help if needed)

### Success Criteria

- You have a `CLAUDE.md` that accurately describes the project
- The application is running and you can access it in a browser (if applicable)
- Everyone on the team understands the domain model (owners, pets, visits, vets for Pet Clinic projects)

### Things to Consider

- How specific should your `CLAUDE.md` be? What helps Claude help you?
- Explore the `/{command}` in Claude. 

---
## Exercise Menu (SCG Classigieds API)

There are tasks in the form of User Stories in `specs/interview-tasks.md` that you can use for this workshop.

## Exercise Menu (Pet Clinic Projects)

Pick some exercises that interest your team. No fixed order. The purpose is to better aquaint yourself with Claude.

---

## Theme A: Feature Work

Build something new. End-to-end: backend, frontend, tests.

### A1. Delete with Grace

**Goal:** Implement soft-delete or hard-delete for one or more entities.

Pick an entity (owner, pet, or visit) and add the ability to remove it. Consider:
- What happens to related data? (Deleting an owner with pets and visits)
- Should it be a soft delete (archived/flagged) or hard delete?
- Add a confirmation step in the UI
- Write tests that verify behavior

**Success:** You can delete an entity from the UI, related data is handled correctly, and tests prove it.

### A2. Visit History & Timeline

**Goal:** Build a proper visit management view.

Currently, visits are just a list on the owner detail page. Make them a first-class feature:
- A dedicated page showing visit history for a pet (filterable by date range)
- Ability to edit or cancel an upcoming visit (but not historical ones)
- A "schedule next visit" feature with date validation (no past dates, no double-booking)

**Success:** Visits have their own UI, can be managed independently, and have proper validation.

### A3. Pet Type Management

**Goal:** Make pet types configurable by clinic staff.

Pet types are currently seed data. Build a small admin feature:
- CRUD for pet types (add "rabbit", rename "lizard" to "reptile", retire unused types)
- Prevent deletion of types that are in use
- Show usage count per type
- Update the "add pet" form to reflect changes without restart

**Success:** A clinic admin can manage pet types through the UI, with proper safeguards against breaking existing data.

---

## Theme B: Quality & Hardening

Improve what exists. Make it production-worthy.

### B1. Test the Unhappy Paths

**Goal:** Find and test the edge cases the original developers skipped.

Use Claude to:
- Identify which code paths have no test coverage
- Find edge cases: what happens with empty strings? Unicode names? Concurrent edits?
- Write tests that exercise validation boundaries, error responses, and failure modes
- Aim for at least 5 meaningful new test cases that catch real issues

**Success:** Your new tests pass, and at least one of them revealed an actual bug or unhandled edge case in the existing code.

### B2. Input Validation Audit

**Goal:** Ensure all user input is properly validated and errors are clear.

Examine every endpoint that accepts data:
- Are all fields validated? (length, format, required vs optional)
- Are error messages helpful to the end user?
- Can you craft a request that causes a 500 instead of a 400?
- Fix any gaps. Standardize error response format across all endpoints.
- Use Claude and test with intentionally malicious input (XSS payloads, SQL fragments, oversized payloads)

**Success:** Every invalid input returns a clear, consistent error response. No 500s from bad user data.

### B3. Observability & Health

**Goal:** Add logging, metrics, or health checks that would matter in production.

Consider:
- Structured logging on key operations (created, updated, deleted, failed validation)
- A health endpoint that checks database connectivity and reports app version
- Request timing (how long do database-heavy endpoints take?)
- If the database, for example, goes down, does the app fail gracefully?

**Success:** You can observe what the application is doing from its logs and health endpoint without looking at the source code.

---

## Theme C: Architecture & Refactoring

Restructure for maintainability, without changing behavior.

### C1. Introduce a Service Layer

**Goal:** Separate business logic from HTTP/web concerns.

Both PetClinic versions have endpoints/controllers talking directly to the database layer. Introduce a service layer:
- Move business rules (duplicate pet name check, cascade logic, etc.) into services
- Keep endpoints/controllers thin: parse request, call service, return response
- Ensure all existing tests still pass after refactoring
- Add a new business rule via the service layer to prove the pattern works

**Success:** No endpoint/controller contains business logic. Behavior is unchanged. Tests pass.

### C2. Add Audit Trail

**Goal:** Track who changed what, and when.

Implement automatic tracking across all entities:
- `CreatedAt`, `ModifiedAt` timestamps
- Use framework features (EF Core interceptors, JPA entity listeners) rather than manual code in every endpoint
- Expose audit info in the API responses
- Write a migration strategy that backfills existing data

**Success:** Every create/update operation automatically records timestamps. Existing data has sensible defaults.

### C3. API Versioning & Documentation

**Goal:** Make the API consumable by external developers.

- Add API versioning (URL-based or header-based)
- Generate an OpenAPI specification
- Ensure the spec is accurate (test it by generating a client)
- Add meaningful descriptions to endpoints and schemas
- Investigate how the current setup handles API documentation (Swagger/Scalar). Improve or extend it.

**Success:** A developer unfamiliar with the project can read the API docs and successfully call every endpoint.

---

## Theme D: Build From Scratch

Not interested in PetClinic? Use Claude Code to prototype something entirely different.

### Ideas (or ignore these and do your own thing)

- **CLI tool**: Build a command-line app that does something useful. A git helper? A file organizer? A CSV transformer?
- **API integration**: Connect to a public API (weather, news, GitHub) and build something on top of it
- **Game**: A terminal-based game, a simple web game, or a puzzle solver
- **Developer tool**: A linter, a code generator, a documentation extractor
- **VS Code Extension**: Some extension to help you with one of your daily pains.

### The Only Rule

Use Claude Code as your primary development tool. Pay attention to:
- How you describe what you want
- When Claude gets it right vs. when it needs correction
- What's faster with AI and what's actually slower
- Where you still need to think and where Claude handles the thinking

**Success:** You built something using AI. You have learned a take-away: could be something that worked well, or what didn't work so you can avoid it in the future.

---

## Wrap-Up: Team Discussion (15 min)

When time is called, discuss these with your team:

1. **What surprised you?** Something Claude did that you didn't expect (good or bad).
2. **Where did you still need to think?** Where is the human in the loop?
3. **What failed?** A task that didn't work, a direction Claude took that was wrong, something you had to redo.
4. **Speed vs. quality**: Did AI-assisted code feel rushed? More solid? About the same?
5. **Would you use this tomorrow?** For what tasks, and for what would you still avoid it?
6. **What would you do differently** if you ran this exercise again?

---

## Tips

- Talk to Claude like a colleague, not a search engine. Context helps.
- If Claude goes in the wrong direction, tell it. "No, not that. I want X because Y."
- Use `/init` to set project conventions. Use `CLAUDE.md` to steer behavior.
- Divide and conquer, plan big and execute small. If Claude goes wrong, could it be a size issue?
- Commit often. Small commits make it easy to revert if Claude takes a wrong turn.
- If you're stuck, ask Claude to explain what it's doing and why.
