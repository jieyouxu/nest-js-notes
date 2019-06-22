# Guards

A **guard** is an `@Injectable()` class which implements the `CanActivate` interface.

Guards have the *single responsibility* of determining whether the request should be handled by the handler (**authorization**).

- Guards have access to `ExecutionContext` and knows what is executed next.
- Guards are executed *after* middlewares, *before* interceptors or pipes.

## Authorization Guard

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

## Execution Context

```typescript
export interface ExecutionContext extends ArgumentsHost {
  getClass<T = any>(): Type<T>;
  getHandler(): Function;
}
```

## Role-based Authentication

Base:

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
```

### Binding Guards

Guards can be:

- Controller-scoped
- Method-scoped
- Global-scoped

The guard can be used via `@UseGuards()`.

```typescript
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

### Reflection

Custom metadata can be attached to routes via `@SetMetadata()` decorator, which can supply `role` data to allow the guard to make decisions.

Create custom decorators to represent this:

```typescript
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

```typescript
@Post()
@Roles('admin')
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

### Continued Role-based Authentication

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    const hasRole = () => user.roles.some((role) => roles.includes(role));
    return user && user.roles && hasRole();
  }
}
```

This assumes `request.user` contains user instance and allowed roles.

Upon insufficient privileges, Nest returns

```json
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

A guard returning `false` throws a `ForbiddenException`.

Exceptions thrown by guards will be handled in the **exception layer**.

