# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

Zerou is a game backlog tracker built as a learning project for a production-style AWS architecture. The repo is currently only a folder skeleton (`.gitkeep` placeholders) and `README.md`; there is no `package.json`, build, lint or test tooling yet. Do not assume commands exist — check before running anything, and update this file once tooling is added.

Each roadmap step (see README) is built by hand in the AWS console first, then repeated as infrastructure as code. Console work is documented in `docs/runbooks/`; non-obvious decisions go in `docs/adr/`.

## Architecture

Request path: browser → Next.js BFF (`apps/web`) → API Gateway HTTP API with a Cognito JWT authorizer → Lambda handlers (`services/api`) → PostgreSQL.

- **BFF owns auth.** The Next.js server runs the Cognito authorization code flow with PKCE and keeps tokens server-side. The browser only holds an HttpOnly session cookie and must never receive a JWT. Browser calls go to Next.js route handlers in `apps/web/src/app/api/`, which forward to API Gateway with the token.
- **Authorization** is per-user (every query is scoped to the caller's Cognito `sub`) plus a Cognito admin group for restricted routes. Lambdas read identity from the authorizer context, not from the request body.
- **Cover images** are uploaded directly from the browser to S3 using presigned URLs issued by a Lambda, and served through CloudFront.
- **Enrichment** runs asynchronously: an SQS queue feeds a worker Lambda (`handlers/jobs/`) that fetches data from a public game API.
- **User migration** is a Cognito trigger Lambda (`handlers/cognito/`).

## Code layout rules

Monorepo with pnpm workspaces (planned): `apps/web`, `services/api`, `packages/shared`, `packages/tsconfig`, `infra`.

- `services/api/src/handlers/` — one file per Lambda entry point, kept thin: parse event, validate, call `domain/`, map to an HTTP response.
- `services/api/src/domain/` — business logic with no AWS imports. Dependencies flow `handlers → domain → db / integrations`.
- `apps/web` never imports from `services/api`; shared schemas and types live only in `packages/shared`.
- `infra/` is an AWS CDK app (TypeScript) with one stack per area (auth, network, data, storage, api, jobs), mirroring the roadmap steps.

Planned libraries: Zod (shared validation), Drizzle ORM with drizzle-kit migrations, openid-client and iron-session in the BFF, Powertools for AWS Lambda, Vitest.

## Commits

Use short one-line Conventional Commits (`chore: ...`, `docs: ...`, `feat: ...`). Do not add a Co-Authored-By line.
