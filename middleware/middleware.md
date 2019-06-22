# Middleware

A **middleware** is a function called **before** the route handler:

- Have access to *request* and *response* objects.
- Has `next()` middleware function.

![Middleware](img/middleware-1.png)

Middleware functions may:

- Execute any code.
- Modify request and response objects.
- Terminate request-response lifecycle.
- Call next middleware.
  - It must call `next()` middleware to pass control.

A middleware can either be implemented as a function or an `@Injectable()` class, with the class implementing `NestMiddleware` interface.

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response } from 'express';

@Injectable()
export class LoggingMiddleware implements NestMiddleware {
  use (req: Request, res: Response, next: Function) {
    console.log('Request');
    next();
  }
}
```

## Dependency Injection

Available just like providers and decorators through `constructor`.

## Applying Middleware

Setup using `configure()` method of the module class (which must implement `NestModule` interface).

```typescript
import { Module, NestModule, RequestMethod MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class ApplicationModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({
      	path: 'cats',
      	method: RequestMethod.GET
    	});
  }
```

The middleware is then active for `GET /cats` route handler(s).

## Route Wildcards

The `path` and `method` supplied to `forRoutes` can be a regex and `RequestMethod.ALL` respectively.

## Middleware Consumer

A helper class.

- `forRoutes()` can take either:
  - Single `string`
  - Multiple `string`s
  - `RouteInfo` object
  - A `Controller` class
  - Multiple `Controller` classes
- `apply()` can take:
  - Single middleware
  - Multiple middlewares

## Excluding Routes

Routes can be excluded via `exclude()`

```typescript
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST }
  )
  .forRoutes(CatsController);
```

## Functional Middleware

A simple middleware can be a function if it does not have any dependencies.

```typescript
export function logger(req, res, next) {
  console.log(`Request...`);
  next();
};
```

## Global Middleware

A global middleware can be registered by `use()` on `INestApplication`.

