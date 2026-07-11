# Ryppl — Contracts (API source of truth)

> **Ryppl** (pronounced *"ripple"*) is an AI wellness companion — *not a tracker and not a chatbot* — that learns each person's journey and shows up with warmth, encouragement, and hope. *Small ripples. Whole life.* The product spans four repos: [`ryppl-mobile`](https://github.com/ryppl-app/ryppl-mobile) (the app), [`ryppl-backend`](https://github.com/ryppl-app/ryppl-backend) (AI gateway), [`ryppl-admin`](https://github.com/ryppl-app/ryppl-admin) (internal console), **`ryppl-contracts`** (this — the API/schema source of truth).

**Single source of truth** for the Ryppl API and the **sanitized-AI-context** schema. Every other repo generates its types from here, so the on-device → backend → AI boundary stays honest and in sync — this is where Ryppl's privacy posture (PHI stays on the device) is enforced in schema.

> Consumers: `ryppl-mobile` (Dart), `ryppl-backend` (Java/Spring), `ryppl-admin` (TypeScript).

## What's here
- `openapi.yaml` — OpenAPI 3.1: endpoints + schemas (`AiContext`, `Insight`, `Card`, `Milestone`, `FoodEstimate`, minimal account types).
- `schemas/` — standalone JSON Schemas where a non-HTTP artifact needs one (e.g. the on-device card-answer payloads).

## The two load-bearing ideas
1. **The sanitization boundary is enforced in schema.** `AiContext` is `additionalProperties: false` and explicitly forbids identifiers/raw PHI. The device assembles + validates against it; the backend persists no conversation content.
2. **Auth is not in the API.** Users sign in on-device via Google/Apple; the provider ID token (JWT) is sent as a Bearer header and **verified at the edge** (stateless, pseudonymous `sub`). The backend mints no token and runs no login system. See the `security` block in `openapi.yaml`.

## Codegen
```bash
npm install
npm run lint            # validate the spec
npm run gen:dart        # → gen/dart   (ryppl-mobile)
npm run gen:java        # → gen/java   (ryppl-backend, interfaces only)
npm run gen:ts          # → gen/ts     (ryppl-admin)
```
Generated code is not committed; each consumer runs codegen in its own build. Tag releases (`v0.1.0`, …) and pin consumers to a tag.

## Conventions
- **Versioned + backwards-compatible.** Breaking changes bump the major and are announced to all three consumers.
- Any field that would carry PHI off-device must be justified in review; default is to keep it on-device.
- All AI output text must satisfy `ryppl-companion-voice`.
