# Tiny Tamagotchi

A submission for the [**DeepLearning.AI 7-Day Learner Challenge:
Tiny Tamagotchi MVP with Spec-Driven Development**](https://community.deeplearning.ai/t/7-day-learner-challenge-tiny-tamagotchi-mvp-with-spec-driven-development/891489)
(April 15 – April 22, 2026).

The challenge, from the
[*Spec-Driven Development with Coding Agents*](https://www.deeplearning.ai/short-courses/)
short course, asks learners to build a tiny virtual pet — but the
*primary artifact for evaluation is the spec*, not the UI. This
organization is my attempt at that, taken seriously: one
engineering-owned platform, one consumer frontend, and a set of specs
that a coding agent could follow without reading any source.

## The challenge, in one paragraph

Build a living virtual pet with three meters (**Hunger**,
**Happiness**, **Energy**), three actions (**Feed**, **Play**,
**Rest**), and three states (**Normal**, **Sick**, **Evolved**). One
user, one pet, one evolution path, one recovery path, passive decay
over time — and no auth, no currencies, no permanent death. The spec
must be clear, scoped, internally consistent, and testable.

## How this organization is laid out

Rather than ship a single app, I split the challenge along a real
product seam: an engineering-owned backend platform that owns the
*contract*, and a consumer frontend that owns the *experience*. The
two meet at a versioned, typed SDK.

### [`pet-platform`](https://github.com/tiny-tamagotchi/pet-platform) — the spec-driven backend

The submission's spec lives here. It is the source of truth for the
pet's state machine, the APIs that expose it, the events it emits,
and the typed SDKs consumers depend on.

- `specs/mission.md`, `specs/roadmap.md`, `specs/tech-stack.md`
- `specs/domain/` — the pet state machine and numeric parameters
- `specs/contracts/` — Zod-first DTOs, commands, events, errors
- `features/pet-core/` — per-feature `feature-plan.md`,
  `requirements.md`, `validation.md` (the artifacts the rubric asks
  for)
- `docs/constitution.md` — the enforceable rules that prevent drift
- `docs/decisions/` — ADRs for every non-trivial tech decision
- `packages/` — `contracts`, `openapi`, `sdk`, `database`, `testing`
  (all under the `@tiny-tamagotchi-mg/*` scope)
- `services/pet-api` — NestJS service, thin orchestrator over the pure
  domain reducers

### [`tiny-tamagotchi-sanctuary`](https://github.com/tiny-tamagotchi/tiny-tamagotchi-sanctuary) — the consumer frontend

The UI half of the submission. A Lovable-managed React app that
depends *only* on the typed SDK published from `pet-platform` — it
never reaches into raw service code and never redefines contract
types.

- React 19 + TanStack Start / Router, Vite, Tailwind v4, shadcn/ui
- Deployed on Cloudflare Workers
- Consumes `@tiny-tamagotchi-mg/sdk` + `@tiny-tamagotchi-mg/contracts`
- Exactly one seam between SDK types and UI types
  (`src/lib/pet-sdk.ts`), enforced by review

![Organization architecture — pet-platform owns the contract, tiny-tamagotchi-sanctuary consumes it via the typed SDK](./assets/architecture.svg)

<sub>Source: [`assets/architecture.drawio`](./assets/architecture.drawio) (editable in
[app.diagrams.net](https://app.diagrams.net) / draw.io desktop).
A 2× PNG export is also committed at [`assets/architecture.png`](./assets/architecture.png).</sub>

## How the spec drives the code

The challenge rubric scores *completeness*, *clarity and specificity*,
*internal consistency* (within features and against the constitution),
and *testability* (validation strategies that actually exercise the
plan). Concretely, in this org that looks like:

- **Spec-first.** Every feature starts in `features/<name>/` with
  `plan.md`, `requirements.md`, and `validation.md` before any code
  lands. Code that doesn't match a spec is either a bug or requires a
  spec update first.
- **Contract-first.** Zod schemas in `@tiny-tamagotchi-mg/contracts`
  are the single source of truth. OpenAPI, the SDK, the NestJS edge,
  and persistence mappers all derive from them; CI rejects drift.
- **Constitution-enforced.** `docs/constitution.md` encodes the rules
  the rubric's *internal consistency* criterion is really asking
  about — feature specs must not contradict it.
- **Testability at two levels.** Vitest unit + property tests for the
  pure domain reducers, Supertest integration tests for the HTTP
  edge, Testcontainers-backed conformance tests running the same
  suite against both in-memory and Postgres adapters.

## MVP scope (locked to the challenge)

Matches the challenge's "Required / Not Allowed" table exactly:

- One pet per user, with a required user-chosen name.
- Three meters (`hunger`, `happiness`, `energy`), each `0..100`.
- Three actions (`feed`, `play`, `rest`).
- Three macro-states (`Normal`, `Sick`, `Evolved`) — no permanent
  death.
- One evolution path (`Normal → Evolved`, sustained-care trigger;
  `Evolved` is terminal).
- One recovery path (`Sick → Normal`, passive when meters return to
  a healthy range).
- Passive time decay is required — the pet is a living system, not a
  static record.

No auth, no multi-user, no multi-pet, no inventory, no currencies,
no mini-games, no notifications. Everything outside that list is
deferred.

## Tech stack at a glance

| Layer          | Choice                                                  |
| -------------- | ------------------------------------------------------- |
| Language       | TypeScript (>=5.6) on Node.js (>=20.11)                 |
| Backend        | NestJS, Zod, `@asteasolutions/zod-to-openapi`           |
| Persistence    | TypeORM (Data Mapper) + Postgres                        |
| Testing        | Vitest, Supertest, Testcontainers                       |
| Monorepo       | pnpm workspaces, Biome                                  |
| Frontend       | React 19, TanStack Start/Router, Vite, Tailwind v4      |
| UI kit         | shadcn/ui on Radix primitives                           |
| Deploy (web)   | Cloudflare Workers                                      |
| SDK scope      | `@tiny-tamagotchi-mg/*` (CodeArtifact-hosted)           |

## Where to start reading

If you're a reviewer for the challenge, the short path is:

1. [`pet-platform/specs/mission.md`](https://github.com/tiny-tamagotchi/pet-platform/blob/main/specs/mission.md)
2. [`pet-platform/specs/roadmap.md`](https://github.com/tiny-tamagotchi/pet-platform/blob/main/specs/roadmap.md)
3. [`pet-platform/specs/tech-stack.md`](https://github.com/tiny-tamagotchi/pet-platform/blob/main/specs/tech-stack.md)
4. [`pet-platform/docs/constitution.md`](https://github.com/tiny-tamagotchi/pet-platform/blob/main/docs/constitution.md)
5. [`pet-platform/features/pet-core/`](https://github.com/tiny-tamagotchi/pet-platform/tree/main/features/pet-core) —
   `feature-plan.md`, `requirements.md`, `validation.md`
6. [`pet-platform/specs/domain/virtual-pet-state-machine.md`](https://github.com/tiny-tamagotchi/pet-platform/blob/main/specs/domain/virtual-pet-state-machine.md)

For the frontend seam and how it consumes the contract, see
[`tiny-tamagotchi-sanctuary/CONTRIBUTING.md`](https://github.com/tiny-tamagotchi/tiny-tamagotchi-sanctuary/blob/main/CONTRIBUTING.md).

---

*Built for the DeepLearning.AI 7-Day Learner Challenge, April 2026.
Spec quality over UI polish — by design.*
