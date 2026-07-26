# Learning NestJS — Course Notes & Progress Tracker

This is our master syllabus for learning [NestJS](https://docs.nestjs.com) from scratch, built while creating one real project together: a **Task Manager API**.

We work **one topic at a time**. Each topic gets checked off here once we've covered the theory *and* you've practiced it in the project. As we go, we may add short notes under each item to remind future-us what we learned or decided.

## How we work

- I explain each concept in plain language, grounded in the official docs.
- I show a small example.
- You practice by writing the code yourself in the project — I don't write it for you.
- We recap, then move to the next topic.
- Nothing is installed globally — we use `npx @nestjs/cli` so tools stay local to this project.

## The Project

A **Task Manager API**: a backend service to create, list, update, delete, and organize tasks. Simple enough to stay clear, rich enough to justify every concept (auth guards for "only see your own tasks", validation pipes for task input, etc.).

---

## Curriculum

### 0. Before We Start
- [ ] What is NestJS, and why use it over plain Express/Node
- [ ] Prerequisites check (Node.js, npm, editor)
- [ ] Creating the project with the Nest CLI (`npx @nestjs/cli new`)
- [ ] Tour of the generated project structure

### 1. Overview (Core Building Blocks)
- [ ] **Controllers** — receiving and routing incoming HTTP requests
- [ ] **Providers & Services** — where business logic and data handling live
- [ ] **Modules** — how features are grouped and wired together
- [ ] **Middleware** — code that runs before a request reaches your route handler
- [ ] **Exception Filters** — catching errors and shaping error responses
- [ ] **Pipes** — validating and transforming incoming data (e.g., request bodies)
- [ ] **Guards** — controlling access (authentication/authorization)
- [ ] **Interceptors** — wrapping extra behavior around a request/response (logging, transforming responses, caching)
- [ ] **Custom Decorators** — building your own reusable `@SomeDecorator()` annotations

### 2. Fundamentals (Deeper Mechanics)
- [ ] Custom Providers — more control over how dependencies are created
- [ ] Asynchronous Providers — providers that need async setup (e.g., a DB connection)
- [ ] Dynamic Modules — modules that can be configured when imported
- [ ] Injection Scopes — controlling the lifetime of a provider instance
- [ ] Circular Dependencies — what they are and how to resolve them
- [ ] Module Reference / Lazy-loading Modules
- [ ] Execution Context — inspecting the current request across guards/interceptors/filters
- [ ] Lifecycle Events — hooking into app startup/shutdown
- [ ] Testing — unit and end-to-end (e2e) testing a NestJS app

### 3. Techniques (Practical, Everyday Tools)
- [ ] Configuration — managing environment variables safely (`@nestjs/config`)
- [ ] Database Integration — connecting to a real database (TypeORM or Prisma)
- [ ] Validation — using DTOs (Data Transfer Objects) + `class-validator`
- [ ] Serialization — controlling exactly what data gets sent back to the client
- [ ] Versioning — versioning your API endpoints
- [ ] Task Scheduling — running code on a schedule (cron-like jobs)
- [ ] Logging — built-in and custom loggers
- [ ] File Upload — handling uploaded files
- [ ] HTTP Module — making outgoing HTTP requests from your app

### 4. Security
- [ ] Authentication — verifying who a user is (e.g., login, JWT tokens)
- [ ] Authorization — verifying what a user is allowed to do
- [ ] Encryption & Hashing — protecting sensitive data like passwords
- [ ] Helmet, CORS, Rate Limiting — common production security hardening

### 5. Wrap-up / Where To Go Next
- [ ] OpenAPI (Swagger) — auto-generating API documentation
- [ ] Deployment basics
- [ ] Optional deep dives: GraphQL, WebSockets, Microservices (only if you want to continue past REST)

---

## Progress Log

*(We'll jot dated notes here as we complete sections, e.g. quirks we hit, decisions we made.)*

- 2026-07-26 — Course plan agreed. Starting from Section 0.
