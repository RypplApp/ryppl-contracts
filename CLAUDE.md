# CLAUDE.md — ryppl-contracts

Dev context for Claude Code working in the Ryppl contracts. **These rules override default behavior.**

## What this is
The **single source of truth** for the Ryppl API and the sanitized-AI-context schema — OpenAPI 3.1 (`openapi.yaml`) + standalone JSON Schemas (`schemas/`). `ryppl-mobile` (Dart), `ryppl-backend` (Java), and `ryppl-admin` (TS) all generate types from here.

## The load-bearing rules
1. **The sanitization boundary is enforced in schema.** `AiContext` is `additionalProperties: false` and forbids identifiers / raw PHI. Only a minimal, pseudonymous context may leave the device; the backend persists no conversation content. **Any new field that could carry PHI off-device must be justified in review — default is to keep it on-device.**
2. **Auth is not in the API.** Users sign in on-device via Google/Apple; the provider ID token (JWT) is a Bearer header, verified at the edge (stateless, pseudonymous `sub`). The backend mints no token and runs no login system.
3. **Versioned + backwards-compatible.** Breaking changes bump the major and are announced to all consumers. Generated code is **not** committed — each consumer runs codegen in its build and pins to a release tag.

## Working here
- Edit `openapi.yaml` / `schemas/`; run `npm run lint` to validate before committing.
- Keep DTO shapes aligned with `ryppl-backend`'s `com.ryppl.ai.dto.AiOutputs` (e.g. `Insight`, `Card`, `Milestone`, `FoodEstimate`, `ExtractedGoals`, `ExtractedLog`).
- All AI output text fields must satisfy the Ryppl companion voice (positive-only, warm, coach + partner + friend).
