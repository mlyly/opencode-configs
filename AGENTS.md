## Execution, Escalation & Dual-GPU Parallelism Rules

### 1. Plan First with `@architect-deep`, Then Implement Fast

- **Before writing code on any multi-file feature, architectural change, or complex bug**, immediately invoke the
  `@architect-deep` subagent with the relevant file paths and requirements.
- Have `@architect-deep` produce a concise, file-by-file implementation blueprint (exact interfaces, method signatures,
  edge cases, and order of changes).
- Once `@architect-deep` returns the plan, **execute all file edits yourself using the fast non-thinking `build` model
  ** (and `@fast-edit` for unit tests/boilerplate). Do not re-plan from scratch between edits.

### 2. Automatic Escalation on Stuck Problems (2-Strike Rule)

- If a compiler error (`mvn`/`gradle`/`tsc`), test failure, or runtime bug **is not resolved after 2 consecutive edit
  attempts**, **STOP guessing immediately**.
- Invoke `@architect-deep` with:
    1. The exact compiler/test error output.
    2. The code snippets involved.
    3. What was already tried and why it failed.
- Apply the root-cause fix returned by `@architect-deep`.

### 3. Parallel Architect Execution (Dual V100 Load Balancing)

- The backend gateway load-balances `qwen-architect` across **two independent V100 GPUs (`least-busy`)**.
- Whenever a task splits cleanly into **two independent modules** (for example: Backend Java REST/Service layer +
  Frontend Angular/Next.js components, or two independent microservices):
    1. Implement one module directly in the main `build` thread (runs on GPU 0).
    2. Simultaneously spawn **`@architect-parallel`** in parallel to implement the second module (routes automatically
       to GPU 1).
- Never spawn more than **2 concurrent architect tasks** at once so both primary GPUs stay at 100% single-stream speed
  without queueing.

# Workspace Agent Guidelines & Skill Router

## 1. Skill Routing Rules

Before writing, refactoring, or testing code, inspect the target directory and project files (`pom.xml`,
`build.gradle*`, `package.json`, `angular.json`, `components.json`) and load the matching skill(s) from
`.opencode/skills/`:

- **Spring Boot (`spring-boot`)**: Load when working in directories containing `pom.xml` or `build.gradle*` with
  `spring-boot-starter` dependencies.
- **Quarkus (`quarkus`)**: Load when working in directories containing `pom.xml` or `build.gradle*` with `io.quarkus` /
  `quarkus-bom` dependencies.
- **Next.js + shadcn/ui (`nextjs-shadcn`)**: Load when working in directories containing `next.config.*` or
  `package.json` with `next` (and `components.json` or Tailwind CSS).
- **Angular (`angular`)**: Load when working in directories containing `angular.json` or `package.json` with
  `@angular/core`.

*Note: In full-stack monorepos (e.g., `backend/` + `frontend/`), load the specific skill for the sub-project you are
currently modifying. If a task spans an API contract change across both, load both relevant skills.*

## 2. Universal Workspace Rules

- **Inspect Before Acting**: Read existing neighboring files to match local naming, folder conventions, and library
  versions before generating new files.
- **Minimal, Focused Diffs**: Only modify code required to fulfill the request. Do not reformat unrelated files or
  introduce unrequested dependencies.
- **Secrets & Environment**: Never hardcode credentials, tokens, or connection strings. Use environment variables or
  `.env.example` templates.
- **Mandatory Verification**: Never mark a task complete until the target sub-project compiles cleanly and its relevant
  test suite passes.
