# Zerou

Zerou is a game backlog tracker where each person keeps a personal list of games they want to play, are playing, or have finished.

The app is a learning project. Its real purpose is to practice a production-style AWS architecture in a small and safe environment, covering authentication, authorization, serverless APIs, file storage and asynchronous jobs.

## Architecture

- Next.js app acting as a BFF, so the browser never sees a JWT and only holds an HttpOnly session cookie
- Amazon Cognito user pool for sign up and sign in, using the authorization code flow with PKCE
- API Gateway HTTP API with a JWT authorizer in front of the backend
- AWS Lambda running the API handlers
- PostgreSQL for per-user data, with pagination
- S3 and CloudFront for game cover images, uploaded through presigned URLs
- SQS and Lambda for background jobs that enrich games with data from a public API

## Features

- Sign up, sign in and per-user game lists
- Create, edit and list games with status and rating
- Admin group with a restricted route
- Cover image upload
- Background enrichment of game data
- User migration through a Cognito trigger

## Roadmap

1. Cognito user pool and a protected GET /games route
2. Next.js BFF with PKCE login and cookie session
3. Group based authorization
4. Postgres with pagination
5. Cover images with S3 and CloudFront
6. Async jobs with SQS and Lambda
7. User migration trigger
8. Infrastructure as code

## Tech stack

TypeScript, Node.js, Next.js, AWS Lambda, API Gateway, Cognito, S3, CloudFront, SQS, PostgreSQL

## Project structure

```
zerou/
├── apps/
│   └── web/                  # Next.js BFF and UI
│       └── src/
│           ├── app/
│           │   ├── (auth)/   # login, callback and logout pages
│           │   ├── (app)/    # signed-in pages
│           │   ├── admin/    # admin-only pages
│           │   └── api/      # BFF route handlers (auth, games)
│           ├── components/
│           └── lib/          # auth, session and API client
├── services/
│   └── api/                  # Lambda code
│       ├── src/
│       │   ├── handlers/     # one entry point per Lambda (games, covers, admin, jobs, cognito)
│       │   ├── domain/       # business logic, no AWS imports
│       │   ├── db/           # schema, client and migrations
│       │   ├── integrations/ # external services (S3, SQS, game API)
│       │   └── lib/          # shared helpers (http, auth, logging)
│       └── test/
├── packages/
│   ├── shared/               # schemas and types shared by web and api
│   └── tsconfig/             # base TypeScript config
├── infra/                    # AWS CDK app, one stack per area
├── docs/
│   ├── adr/                  # architecture decision records
│   └── runbooks/             # notes for each roadmap step built in the console
└── .github/workflows/        # CI
```

Handlers stay thin and call into `domain`, which depends on `db` and `integrations`. The web app and the API share code only through `packages/shared`.

## Status

Work in progress. Each roadmap step is built by hand in the AWS console first, then repeated with infrastructure as code.