# Modules

A **module** is a class with `@Module()` decorator.

- Used by NestJS for resolving module interdependency.

![Modules](img/modules-1.png)

Each application has a **root module**:

- Root node of dependency graph.
- Each module should encapsulate a highly coupled set of capabilities.

The `@Module()` decorator takes an object as option, with the keys:

| Key           | Purpose                                              |
| ------------- | ---------------------------------------------------- |
| `providers`   | Providers to be instantiated (shared across module). |
| `controllers` | Controllers to be instantiated.                      |
| `imports`     | Dependencies by the module.                          |
| `exports`     | Exposed module API.                                  |

The module `exports` explicitly lists providers / controllers / services / repositories, etc., which are **visible** to external modules, helping to **encapsulate**.

## Feature Modules

`CatsController` and `CatsService` are closely related and belong to the same domain, meaning they should be grouped together in the same **feature module**.

They can be grouped to create a `CatsModule`.

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

> The Nest CLI can be used to create a module: `nest g module cats`.

The `CatsModule` should then be imported by the root `ApplicationModule`, by

```typescript
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class ApplicationModule {}
```

The directory structure would look like

```
src
	|- cats
	|		|- dto
	|		|		|- create-cat.dto.ts
	|		|- interfaces
	|		|		|- cat.interface.ts
	| 	|- cats.service.ts
	| 	|- cats.controller.ts
	|		|- cats.module.ts
	|- app.module.ts
	|- main.ts
```

## Shared Modules

Modules are **singletons** by default, and provider instances are shared across modules.

- Each module is then automatically a **shared module**.

The `CatsService` can be exported which will be shared by dependent modules.

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

## Re-exporting Modules

```typescript
@Module({
  imports: [AModule],
  exports: [AModule],
})
export class SomeModule {}
```

## Dependency Injection

A module class can also **inject** providers (e.g. config):

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private readonly catsService: CatsService) {}
}
```

## Dynamic Modules

Dynamic modules can be created based on supplied parameters.

```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

The `forRoot()` method could return the module synchronously or asynchronously.

It can be imported and re-exported like

```typescript

```

