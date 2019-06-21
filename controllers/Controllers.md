# Controllers

**Controllers** handle incoming *requests* and return *responses* to the client.

![Controllers](img/controllers-1.png)

- **Routing** controls which request(s) each controller receives.
  - Each controller may have $> 1$ route, with each route performing different actions.
- A basic controller is created with:
  - Classes, and
  - **Decorators** (which associates classes with required *metadata*, including routing information).

## Routing

The `@Controller` decorator (required) can be used to define a basic controller.

- A route path prefix can be specified.

For example,

```typescript
import { Controller, Get } from '@nestjs/common';

// Path prefix: `/cats`
@Controller('cats')
export class CatsController {
  // `GET /cats`
  @Get()
  findAll(): string {
    return 'cats[]';
  }
}
```

> The controller can be created with Nest CLI via `nest g controller cats`.

The `@Get()` *HTTP request method decorator* tells Nest.js to register the `findAll()` method as the handler for `GET /cats`.

- If the `@Get` decorator is passed a path string, e.g. `@Get('summary’)`, then it maps to `GET /cats/summary` with `summary` concatenated after the base controller path.

With Nest.js, if using the standard option for creating responses, then:

- A JavaScript `object` or `array` is automatically serialized to `JSON`.
- A JavaScript `string` is simply returned.

The response **status code** is `200` by default for HTTP verbs other than `POST`, which returns `201` by default.

- This can be changed by adding the `@HttpCode()` decorator for the handler.

### Request Object

Handlers can obtain *request* details by obtaining access to the *request object* from underlying platform, such as from `Express`, by:

- Adding `@Req()` decorator to the handler’s signature.

For example,

```typescript
//...
import { Request } from 'express';

//...
  findAll(@Req() request: Request): string {
    // ...
  }
//...
```

> `Express` typings can be obtained via `@types/express`.

The *request object* represents the HTTP request. It has properties for:

- Request query string
- HTTP headers
- HTTP body

Some decorators are provided to simplify extracting the desired properties (Express):

| NestJS Decorators         | Express Request Object               |
| ------------------------- | ------------------------------------ |
| `@Request()`              | `req`                                |
| `@Response`               | `res`                                |
| `@Next()`                 | `next`                               |
| `@Session()`              | `req.session`                        |
| `@Param(key?: string)`    | `req.params` or `req.params[key]`    |
| `@Body(key?: string)`     | `req.body` or `req.body[key]`        |
| `@Query(key?: string)`    | `req.query` or `req.query[key]`      |
| `@Headers(name?: string)` | `req.headers` or `req.headers[name]` |

### Resources

The `GET` route handler is for fetching resources. `POST` route handlers can be used for creating resources.

The `@Post` decorator notifies NestJS to register the handler `create()` for handling `POST /cats`

```typescript
import { Controller, ..., Post } from '@nestjs/common';

@Controller('cats')
// ...
	@Post
	create(): string {
    // ...
  }
// ...
```

Each of the HTTP request methods have their corresponding decorators:

- `@Put()`
- `@Delete()`
- `@Patch()`
- `@Options()`
- `@Head()`
- `@All()`

### Route Wildcards

Routes can also be specified via regular expressions.

### Status Code

Default response codes can be overriden by `@HttpCode(newCode: number)` decorator for each handler.

```typescript
@Post()
@HttpCode(204)
create(): string { 
  // ... 
}
```

Dynamic status codes can handled via library-specific response object (via `@Res()` injection), or by throwing an *exception*.

### Headers

A custom *response header* can be specified via `@Header()` decorator or library-specific response object `res.header()`.

### Route Paramters

Route parameter **tokens** can be added to the path of the route for obtaining dynamic value, e.g. `cats/:id` allows capturing of the `:id` value.

- The route parameters can then be accessed via the `@Param()` decorator.

```typescript
@Get(':id')
findById(@Param() params): string {
  console.log(params.id);
	return `Hello ${params.id}`;
}
```

The specific `params.id` can also be accessed directly via

```typescript
@Get(':id')
findById(@Param('id') id): string {
  return `Hello ${id}`;
}
```

### Order of Routes

Routes are handled by registration order. If more generic route handlers are registered first, they may shadow and override more specific route handlers.

- If a `@All()` handler is registered first (declared first), it will override a `@Get()` handler which is registered later (declared later). Here, declare the more specific `@Get()` handler *first*.

### Scopes

Almost everything is shared across incoming requests - singletons.

### Asynchronicity

NestJS supports `async` functions. Each `async` function must return a `Promise` or RxJS’s `Observable` (observable streams).

```typescript
// Promise
@Get()
async findAll(): Promise<any[]> {
  return [];
};

// Observable
@Get()
findAll(): Observable<any[]> {
  return of([]);
}
```

### Request Payloads

HTTP request bodies can be obtained via `@Body()` decorator.

The payload body *must* have a **Data Transfer Object** (**DTO**) **schema**, using typescript *classes*, in order to support **Pipes** and other features which utilize run-time metatype.

```typescript
export class CreateCatDTO {
  readonly name: string;
  readonly age: number;
  readonly breed: string;
}
```

Then this DTO can be used in a controller

```typescript
@Post()
async create(@Body() createCatDTO: CreateCatDTO) {
  return 'new cat';
}
```

### Error Handling

Errors are handled in the Exception Layer.

### Registering Controllers

NestJS will need to know about the `CatsController` so it can instantiate it.

- **Controllers** always belong to a **module**, which can be defined via the `@Module()` decorator.

The `CatsController` can be registered to belong to a `ApplicationModule` by

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class ApplicationModule {}
```

Upon registering, NestJS can perform reflection to mount the suitable controllers.