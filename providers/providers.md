# Providers

Many classes are *providers*:

- Services
- Repositories
- Factories
- Helpers

Each *provider* represents a **injectable dependency** - this faciliates **dependency injection** (**DI**).

- Each provider is a class annotated with the `@Injectable()` decorator.

**Controllers** should have the single responsibility of handling HTTP requests. They should *delegate* complex tasks to **providers**.

- Helps to adhere to SOLID principles.

### Services

A `CatsService` may be responsible for persistence, for use by the `CatsController`, which makes it ideal as a provider.

```typescript
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];
  
  create(cat: Cat) { this.cats.push(cats); }
  
  findAll(): Cat[] { return [...this.cats]; }
}
```

> A service can be created via the CLI using `nest g service cats`.

The `Cat` interface helps with type-checking,

```typescript
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

The `CatsController` can then use `CatsService` to handle complex tasks

```typescript
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDTO } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}
  
  @Post()
  async create(@Body() createCatDTO: CreateCatDTO) {
    this.catsService.create(createCatDTO);
  }
  
  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

The `CatsService` dependency is **injected** via `CatsController`’s constructor.

### Dependency Injection

The previous contructor is defined as

```typescript
constructor(private readonly catsService: CatsService) {}
```

Since the type is available, NestJS can resolve the correct dependency and either inject the existing instance or create a new instance.

### Scopes

Providers by default have lifetime scope.

### Custom Providers

See [Custom Providers](custom-modules.md).

### Optional Providers

Some dependencies may be *optional* and need not be resolved (e.g. configuration object, where if none is specified, default values are used).

The `@Optional()` decorator can be used in the `constructor` signature to signify this.

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(
  	@Optional() @Inject('HTTP_OPTIONS') private readonly httpClient: T
  ) {}
}
```

### Property-based Injection

Previous examples used *constructor-based injection*. Sometimes *property-based injection* may be useful.

The `@Inject()` decorator could be used at the property level to avoid injection from the constructor, but constructor-based injection should be preferred.

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

### Provider Registration

The provider `CatsService` is defined, and it has a consumer `CatsController`, but it needs to be registered with NestJS.

This can be done by amending the module definition to add the service to the `providers` array of the `@Module()` decorator.

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class ApplicationModule {}
```

