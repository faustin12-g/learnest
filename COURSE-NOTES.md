# NestJS Course Notes — Detailed Explanations

This document expands every point in [README.md](README.md) into a full, plain-language explanation, grounded in the real NestJS documentation (docs.nestjs.com, source: the official `nestjs/docs.nestjs.com` GitHub repo). Read this first; then we practice each piece hands-on in order.

> **Scope note:** This first pass covers **Section 0 (Before We Start)** and **Section 1 (Overview — Core Building Blocks)** in full detail, since that's what we're about to build. Once we reach Section 2 (Fundamentals) and Section 3 (Techniques) in the README, I'll pull the real docs for those and add a matching section here — that way the notes stay accurate instead of me guessing ahead about choices (like which database library) we haven't made yet.

---

## A 2-minute TypeScript primer (since you're new to it)

You already know JavaScript, so here's just the handful of TypeScript ideas that show up constantly in NestJS code. Skim this once — we'll call back to it as these appear for real.

- **Types**: In JS you write `let age = 25`. In TS you can (optionally) say `let age: number = 25`, meaning "this variable must always hold a number." If you try to assign a string later, TypeScript errors out *before your code ever runs*. This catches bugs early.
- **Classes**: Same as JS classes (`class Foo { }`) — TypeScript just lets you type-annotate their properties and methods.
- **Interfaces**: A TypeScript-only concept — a description of the *shape* of an object, with no actual implementation. Example:
  ```typescript
  interface Cat {
    name: string;
    age: number;
  }
  ```
  This says "anything called a `Cat` must have a `name` (string) and `age` (number)." It's purely for the type-checker; it produces zero JavaScript at runtime.
- **Decorators**: This is the big one for NestJS. A decorator is a special function, written as `@SomethingLikeThis()`, placed directly above a class, method, or property. It **attaches metadata** (extra descriptive information) to that thing, which NestJS reads at startup to know what to do with it. For example, `@Controller('cats')` above a class doesn't change what the class *does* internally — it tells NestJS "treat this class as something that handles HTTP requests starting with `/cats`." You'll see decorators constantly: `@Get()`, `@Injectable()`, `@Module()`, etc.
- **Generics**: Something like `Promise<Cat[]>` or `Observable<T>`. The part in `<...>` says *what kind of thing* is inside the wrapper. `Promise<Cat[]>` means "a Promise that, when it resolves, gives you an array of Cats." You don't need to master these yet — just recognize `<T>` as "the type of the thing inside."

That's genuinely most of what you need to start reading Nest code. More TS features will get their own callout box the first time we use them for real.

---

## Section 0 — Before We Start

### 0.1 What is NestJS, and why not just plain Node/Express?

NestJS is a **framework** for building the backend/server side of an application, running on **Node.js**. Straight from the official docs:

> "Nest (NestJS) is a framework for building efficient, scalable Node.js server-side applications... combines elements of OOP (Object Oriented Programming), FP (Functional Programming), and FRP (Functional Reactive Programming)."

Some important facts, all from the docs:

- **It runs on top of Express by default** (and can optionally run on a faster alternative called Fastify). This means NestJS doesn't replace your Node.js/Express knowledge — it *sits on top of it* and adds structure. You can still reach down and use raw Express request/response objects if you ever need to.
- **The core problem it solves is architecture.** Plain Node.js gives you total freedom but zero guidance — every developer/team ends up organizing a project differently, which becomes painful as the app grows. Nest gives you a standard, opinionated way to organize code so any Nest developer can open any Nest project and immediately recognize the structure.
- **Its architecture is heavily inspired by Angular** (the frontend framework). If you ever look at Angular code later, the patterns (decorators, dependency injection, modules) will look familiar.
- **Full TypeScript support**, though plain JavaScript is technically possible too. We'll use TypeScript, matching the majority of real-world Nest code and the docs' primary examples.

### 0.2 Prerequisites

Per the official docs, you need **Node.js version 20 or higher**. You already have **v22.23.1**, so you're set. You also need `npm` (comes bundled with Node — you have v10.5.1).

### 0.3 Creating the project — locally, no global install

The docs' standard instructions say to install the Nest CLI globally:

```bash
npm i -g @nestjs/cli
nest new project-name
```

Since you specifically don't want a global install, we'll use `npx` instead, which downloads the CLI temporarily just to run this one command — nothing gets permanently installed on your system outside the project itself:

```bash
npx @nestjs/cli new project-name
```

This produces the **exact same result** as the global approach. Once the project exists, its own `package.json` will list `@nestjs/cli` as a local dev dependency, and every future CLI command (like generating a new controller) will run through `npx nest ...` or the npm scripts already set up for you — still with nothing global.

> Note: the CLI will ask you to pick a package manager (npm/yarn/pnpm) interactively during creation. We'll pick **npm** since that's what's already on your machine.

### 0.4 Tour of the generated project (what each file is for)

Once scaffolding finishes, you'll have a `src/` folder with these core files (straight from the docs' table):

| File | What it's for |
|---|---|
| `app.controller.ts` | A basic controller with a single example route. |
| `app.controller.spec.ts` | Unit tests for that controller (`.spec.ts` = test file convention). |
| `app.module.ts` | The **root module** — the entry point that ties the whole app together. |
| `app.service.ts` | A basic service with a single example method — where logic lives. |
| `main.ts` | The **entry file**: the very first code that runs, which boots up the application. |

Here's what `main.ts` looks like, and what each line means:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

- `NestFactory` is the core class Nest gives you to actually construct a running application.
- `NestFactory.create(AppModule)` says "build the whole app, starting from this root module" — Nest then reads `AppModule`'s metadata and wires up everything underneath it (controllers, providers, imported modules...).
- `app.listen(3000)` starts an HTTP server listening on port 3000 (or whatever's in the `PORT` environment variable), the same way you'd do in plain Express.

Running it:

```bash
npm run start        # starts the app once
npm run start:dev    # starts it in "watch mode" — auto-restarts when you save a file (this is what we'll use while developing)
```

Once running, visiting `http://localhost:3000` in a browser shows `Hello World!` — that's the example route in `app.controller.ts` responding.

The project also comes with **ESLint** (a linter — flags problematic or inconsistent code patterns) and **Prettier** (a formatter — auto-formats code style) already configured, runnable via `npm run lint` and `npm run format`.

---

## Section 1 — Overview (Core Building Blocks)

This is the heart of "how Nest thinks." All the docs use a running example of a `Cats` API (managing cat records) — we'll follow the same example here to explain the concept, then you'll build the equivalent for **Tasks** in our own project.

### 1.1 Controllers

**What they do:** Controllers are responsible for **receiving incoming HTTP requests and sending back responses**. That's their *entire* job — they should stay thin and hand off any real logic to a service (see 1.2).

**How you define one:**

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

Breaking this down:
- `@Controller('cats')` — this decorator marks the class as a controller, and `'cats'` is a **route prefix**. Every route defined inside this class will start with `/cats`.
- `@Get()` above the `findAll()` method says "when an HTTP GET request comes in matching this route, run this method." Since the method itself adds no extra path, and the controller prefix is `cats`, the full route is `GET /cats`.
- The **method name** (`findAll`) is entirely up to you — Nest doesn't care what you call it. Only the decorators matter.
- Because the method just **returns** a value (a string here), Nest automatically sends that back as the HTTP response body, with status code `200` by default (`201` automatically for `@Post()` routes). This is called the "Standard" response approach, and it's the recommended one — you just `return` data like a normal function, no manual response-object plumbing needed (though you *can* drop down to the raw Express response object with `@Res()` if you ever need very fine control).

**All the HTTP method decorators available:** `@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()`, `@Head()`, and `@All()` (matches every method).

**Getting data out of the request** — instead of manually digging into a raw `request` object like in plain Express, Nest gives you dedicated decorators you place directly on method *parameters*:

| Decorator | Gets you |
|---|---|
| `@Param('id')` | A named URL segment, e.g. `/cats/:id` |
| `@Body()` | The parsed JSON request body |
| `@Query('page')` | A query-string value, e.g. `?page=2` |
| `@Headers('auth')` | A request header |
| `@Req()` | The raw underlying request object (escape hatch) |

Example combining a route param with a GET:

```typescript
@Get(':id')
findOne(@Param('id') id: string): string {
  return `This action returns a #${id} cat`;
}
```

Calling `GET /cats/3` runs this method with `id` equal to `"3"`.

**DTOs (Data Transfer Objects):** when accepting data in a `POST`/`PUT` body, Nest strongly recommends defining a **class** describing the expected shape:

```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

Important detail from the docs: use a **class**, not a TypeScript `interface`, for DTOs. Interfaces are erased completely when TypeScript compiles down to JavaScript, so at runtime Nest wouldn't have any information left about the shape — but classes still exist as real objects at runtime, which Nest's validation system (covered under Pipes) relies on.

**A controller must be registered** in a module's `controllers` array to actually be used (we'll see this in 1.3).

### 1.2 Providers & Dependency Injection

**What a provider is:** a plain class that can be **injected as a dependency** into other classes. Most of your actual business logic and data-handling lives in providers — most commonly ones called **services**.

**Why split controller vs. service?** The docs put it simply: "Controllers should handle HTTP requests and delegate more complex tasks to providers." Keeping data logic separate from request-handling logic keeps each class focused on one job (this is part of the SOLID principles the docs reference — a well-known set of good object-oriented design guidelines).

**A basic service:**

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

- `@Injectable()` is the decorator that tells Nest "this class can be managed by Nest's dependency injection system" (also called an **IoC container** — Inversion of Control — meaning *Nest* decides when to create and hand you instances, rather than you manually calling `new CatsService()` everywhere).

**Using the service inside a controller — this is Dependency Injection in action:**

```typescript
@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  findAll(): Cat[] {
    return this.catsService.findAll();
  }
}
```

What's happening in that constructor line — this is a very common thing that confuses TS beginners, so let's slow down:

```typescript
constructor(private catsService: CatsService) {}
```

This is TypeScript shorthand doing **two things at once**:
1. Declaring a class property called `catsService`.
2. Assigning whatever gets passed into the constructor to that property.

It's identical in effect to writing:
```typescript
private catsService: CatsService;
constructor(catsService: CatsService) {
  this.catsService = catsService;
}
```
— just shorter. The `private` keyword here isn't about dependency injection specifically; it's what triggers this shorthand and marks the property as only accessible inside this class.

Now, the actual dependency injection magic: because the constructor parameter is typed as `CatsService`, Nest looks at that type, automatically creates (or reuses) an instance of `CatsService`, and hands it to the controller — you never write `new CatsService()` yourself anywhere. **Nest wires it up for you.** This is the "relationship forming" the docs mean when they say providers "can be injected... allowing objects to form various relationships with each other."

**Provider scope/lifetime:** By default, a provider is a **singleton** — one single shared instance exists for the lifetime of the whole application, reused everywhere it's injected. (It's possible to make a provider request-scoped instead — a new instance per incoming request — but that's an advanced topic for later.)

**Registering the provider:** just like controllers, a provider must be listed in a module's `providers` array before Nest will know about it (next section).

### 1.3 Modules

**What a module is:** a class decorated with `@Module()`, used purely to **organize** your application into logical groups. Every Nest app has at least one module — the **root module**, conventionally `AppModule` — which is the starting point Nest uses to build what the docs call the "application graph" (an internal map of how everything connects).

```typescript
@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

The four properties you can configure:

| Property | Meaning |
|---|---|
| `controllers` | Controllers belonging to this module that Nest should instantiate |
| `providers` | Services/providers this module creates, usable within the module |
| `imports` | Other modules whose *exported* providers this module wants to use |
| `exports` | Which of this module's own providers should be usable by other modules that import it |

**Key concept — encapsulation:** by default, a module's providers are private to that module. If `ModuleB` wants to use a provider from `ModuleA`, `ModuleA` must explicitly `export` it, and `ModuleB` must `import` `ModuleA`. This is deliberate — it keeps a module's internals from leaking everywhere, forcing clear, intentional boundaries between features.

**Feature modules:** in a real app, you group closely-related things (e.g., everything about "cats," or in our case, "tasks") into their own module — a `CatsModule` containing `CatsController` + `CatsService`. This is then imported into the root `AppModule`:

```typescript
@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

**Shared modules / singletons:** modules themselves are singletons too. If `CatsService` is exported from `CatsModule`, then *every* other module that imports `CatsModule` shares the exact same instance of `CatsService` — not separate copies. This matters if the service holds any internal state, and it's more memory-efficient.

**Global modules (`@Global()`):** normally you must import a module everywhere its providers are needed. Decorating a module with `@Global()` makes its exported providers available app-wide without needing to import it repeatedly. The docs explicitly warn: **don't overuse this** — it's meant for truly app-wide things (e.g., a database connection helper), not a general shortcut, because it hides where dependencies actually come from.

**Dynamic modules:** an advanced pattern where a module exposes a static method (conventionally `forRoot()`) that returns a *configured* version of itself — e.g., a database module you configure with connection details when importing it. We'll only need this once we get to real database integration in Section 3, so we won't dwell on it now — just know the term exists.

### 1.4 Middleware

**What it is:** a function that runs **before** a route handler is reached. This is a direct carry-over from Express — Nest middleware behaves exactly like Express middleware. Per the docs, middleware can:

- run any code
- modify the request/response objects
- end the request-response cycle early (e.g., reject unauthorized requests)
- call `next()` to pass control forward to the next thing in the chain

**Class-based middleware:**

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

> **New TS concept — `implements`:** writing `implements NestMiddleware` is a promise to the TypeScript compiler: "this class will have all the methods/shape that the `NestMiddleware` interface requires." If you forget the `use()` method, TypeScript will error at compile time, before you ever run the app. This is how interfaces act as contracts.

Notice this is **not** applied via `@Module()`'s normal properties. Instead, the module class implements `NestModule` and defines a `configure()` method:

```typescript
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```

`forRoutes()` is flexible — you can pass a path string, a specific `{ path, method }` object, or even a controller class directly (applying it to every route in that controller). There's also `.exclude(...)` to skip specific routes.

**Functional middleware:** if your middleware needs no dependency injection (no constructor injection of a service, etc.), you can write it as a plain function instead of a class — simpler for simple cases like logging.

**Where it fits:** middleware runs first, before guards, pipes, interceptors, or the route handler — it's the earliest interception point in the request lifecycle.

### 1.5 Exception Filters

**What they solve:** when something throws an error anywhere in your app, you need consistent, predictable error responses instead of crashes or leaking internal details. Nest has a **built-in global exception filter** that automatically catches unhandled exceptions and turns them into sensible JSON error responses — you get this for free with zero setup.

**Throwing a standard exception:**

```typescript
@Get()
findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```

This produces:
```json
{ "statusCode": 403, "message": "Forbidden" }
```

`HttpException` is the base class; Nest also ships many ready-made subclasses so you rarely construct `HttpException` directly: `BadRequestException`, `NotFoundException`, `UnauthorizedException`, `ForbiddenException`, `ConflictException`, `InternalServerErrorException`, and more — each maps to the correct HTTP status code automatically.

**Custom exception filters** — when you want full control (custom response shape, logging, etc.), you write your own filter:

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
    });
  }
}
```

- `@Catch(HttpException)` tells Nest "only intercept exceptions of this type." Leaving it empty (`@Catch()`) catches literally everything.
- `ArgumentsHost` is a helper object that gives access to the underlying request/response regardless of context (HTTP, WebSockets, etc.) — you'll see it again with Guards and Interceptors, since all three sit at similar points in the request lifecycle.

**Applying a filter** works at three possible levels, from narrowest to broadest — this same "three levels" pattern repeats for Pipes, Guards, and Interceptors too, so it's worth internalizing once:
- **Method-scoped**: `@UseFilters(HttpExceptionFilter)` above one handler method
- **Controller-scoped**: same decorator above the whole controller class
- **Global-scoped**: `app.useGlobalFilters(new HttpExceptionFilter())` in `main.ts`, applying to the entire app

### 1.6 Pipes

**What they do — two jobs:**
1. **Transformation** — convert input into the desired shape (e.g., a route param arrives as the string `"7"`, a pipe converts it into the number `7`).
2. **Validation** — check incoming data is valid; if not, throw an exception (which stops the request from ever reaching your handler).

Pipes run **right before** your route handler method executes, operating on the arguments about to be passed in.

**Built-in pipes** ship with Nest: `ValidationPipe`, `ParseIntPipe`, `ParseFloatPipe`, `ParseBoolPipe`, `ParseArrayPipe`, `ParseUUIDPipe`, `ParseEnumPipe`, `DefaultValuePipe`, and a couple more.

**Example — transformation:**

```typescript
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

Here, `ParseIntPipe` is placed as a second argument to `@Param()`. If someone requests `GET /cats/abc` (not a number), Nest automatically responds with a `400 Bad Request` before your method body ever runs — you don't need to write that check yourself.

**Example — validation, the pattern we'll use most:** combine a DTO class with decorators from the `class-validator` library:

```typescript
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;

  @IsString()
  breed: string;
}
```

Then bind Nest's built-in `ValidationPipe` (globally, so it checks every incoming DTO app-wide):

```typescript
// main.ts
app.useGlobalPipes(new ValidationPipe());
```

Now, any request body that doesn't match the DTO's rules is automatically rejected with a `400` — without you writing manual `if` checks anywhere. This is the exact pattern we'll use for the Task Manager's "create task" and "update task" endpoints.

**Why not just validate manually inside the route handler?** The docs explain this breaks the **single responsibility principle** — the handler's job is to handle the request, not validate it — and a validator function you call manually is easy to forget to call. Pipes make validation *automatic and declarative*, attached directly to where the data enters the system (the docs call this "the system boundary").

### 1.7 Guards

**What they do:** decide whether a given request is **allowed to proceed** to the route handler at all — this is **authorization** (and often paired with authentication: verifying *who* someone is vs. verifying *what they're allowed to do*).

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request); // your own logic
  }
}
```

- Every guard implements one method: `canActivate()`, returning (or resolving to) `true` (allow) or `false` (deny — Nest automatically throws a `403 Forbidden`).
- `ExecutionContext` is a more powerful version of the `ArgumentsHost` we saw with exception filters — it adds extra methods useful for building generic guards that understand which specific handler/controller is about to run.

**Why guards instead of middleware for authorization?** The docs make a sharp distinction: middleware is "dumb" — it has no idea which specific handler is about to run next. Guards *do* know, via `ExecutionContext`, which lets you write much smarter, route-aware permission logic (e.g., "this specific route requires the `admin` role").

**Role-based access, using custom metadata** — this is a slightly more advanced but very common pattern: you attach custom metadata to a route (e.g., "this route requires the `admin` role") using a custom decorator, then read that metadata inside the guard using a helper called `Reflector`:

```typescript
export const Roles = Reflector.createDecorator<string[]>();

// on a route:
@Post()
@Roles(['admin'])
create(@Body() dto: CreateCatDto) { ... }

// inside the guard:
canActivate(context: ExecutionContext): boolean {
  const roles = this.reflector.get(Roles, context.getHandler());
  if (!roles) return true;
  const { user } = context.switchToHttp().getRequest();
  return matchRoles(roles, user.roles);
}
```

We won't need this level of sophistication immediately (our Task Manager starts without auth), but it's exactly what we'll reach for once we add "only see your own tasks" style rules.

**Binding guards:** same three-level pattern as filters — `@UseGuards()` at method or controller level, or `app.useGlobalGuards()` for the whole app.

### 1.8 Interceptors

**What they do:** interceptors **wrap around** a route handler's execution, letting you run logic both **before and after** it runs. The docs describe this using the term Aspect-Oriented Programming (AOP) — don't worry about the label, just the capability list:

- Run extra logic before/after a handler executes
- Transform the value a handler returns, before it's sent to the client
- Transform an exception a handler throws
- Completely skip the handler and return something else instead (e.g., a cached response)

**The shape of an interceptor:**

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');
    const now = Date.now();
    return next.handle().pipe(
      tap(() => console.log(`After... ${Date.now() - now}ms`)),
    );
  }
}
```

> **New concept — `Observable` / RxJS:** `next.handle()` returns something called an `Observable` (from a library called RxJS — Reactive Extensions for JavaScript). Think of it, for now, as *similar to a Promise* — it represents a value that will arrive later (the eventual result of your route handler) — except Observables come with a large toolbox of operators (`tap`, `map`, `catchError`, `timeout`...) for reacting to or transforming that value as it flows through. You don't need to learn RxJS deeply right now — just recognize the pattern: "call `next.handle()` to actually run the route handler, then `.pipe(...)` extra behavior onto the result."

Concretely: code **before** `next.handle()` runs *before* your controller method. Code inside `.pipe(...)` runs *after* it returns. If an interceptor never calls `next.handle()` at all, the route handler never executes — this is how a caching interceptor can short-circuit and return a cached value instead.

**A very common real use** — reshaping every response into a consistent envelope:

```typescript
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(map(data => ({ data })));
  }
}
```

This turns any handler's return value, e.g. `[]`, into `{ "data": [] }` for every single endpoint, in one place.

**Binding:** identical three-level pattern again — `@UseInterceptors()` at method/controller level, or `app.useGlobalInterceptors()` globally.

### 1.9 Custom Decorators

By now you've used many *built-in* decorators (`@Get()`, `@Body()`, `@Param()`...). Nest also lets you build your **own**, which is extremely useful for keeping code DRY (Don't Repeat Yourself).

**Motivating example:** say your authentication system attaches a `user` object onto the request (`request.user`). Without a custom decorator, every handler that needs the current user repeats:
```typescript
const user = req.user;
```
Instead, define this once:

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

And now use it anywhere, just like a built-in decorator:

```typescript
@Get()
findOne(@User() user: UserEntity) {
  console.log(user);
}
```

**Passing data to the decorator** — the first parameter (`data`) lets you parameterize it, e.g. `@User('firstName')` to pluck just one field:

```typescript
export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const user = ctx.switchToHttp().getRequest().user;
    return data ? user?.[data] : user;
  },
);
```

**Composing decorators:** if you find yourself always stacking the same group of decorators together (e.g., a guard + some metadata + Swagger docs decorators for every authenticated route), `applyDecorators()` lets you bundle them into one custom decorator, e.g. a single `@Auth('admin')` that expands into all four under the hood.

---

## How Section 1's pieces fit together (the request lifecycle, in order)

It's worth seeing the full picture once these are all introduced. For an incoming HTTP request, Nest processes things in this order:

1. **Middleware** runs first (e.g., logging, or Express-level concerns).
2. **Guards** run next — decide if the request is even allowed to proceed (`canActivate`).
3. **Interceptors** (the "before" portion, i.e. code before `next.handle()`) run.
4. **Pipes** run — validate/transform the arguments about to be passed in.
5. Your **route handler** (in the Controller) finally executes, usually delegating to a **Provider/Service**.
6. **Interceptors** get a second turn (the "after" portion, i.e. code inside `.pipe(...)`) to transform the response.
7. If anything threw an exception anywhere along this chain, an **Exception Filter** catches it and shapes the final error response instead.

All of this is orchestrated inside a **Module**, which is how Nest knows which Controllers/Providers/etc. exist and how they relate to each other.

---

## What's next

Once you've read through this and it makes sense (ask me about anything that doesn't!), we start Section 0 hands-on: running `npx @nestjs/cli new task-manager-api` and touring the real generated files together. Then we build the Task Manager's Controllers, Services, and Module for real, one at a time, following this same order.
