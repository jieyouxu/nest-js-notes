# Custom Providers

NestJS allows things to bind directly to its IoC container, such as:

- Constant values
- Configuration objects (environment based)
- External libraries
- Pre-calculated values depending on other defined providers

NestJS uses **tokens** to identify dependencies:

- Auto-generated tokens are equal to classes.
- A token must be chosen for custom providers.
- Tokens should be held in separated files, e.g. `constants.ts`.

## Use value

The `useValue` syntax allows:

- Defining constant value.
- Placing external libraries into Nest container.
- Replace real implementation with mock.

```typescript
import { connection } from './connection';

const connectionProvider = {
  provide: 'CONNECTION',
  useValue: connection,
};

@Module({
  providers: [connectionProvider],
})
export class ApplicationModule {}
```

The custom provider can be injected via the `@Inject()` decorator, with the **token** as the single argument.

```typescript
import { Inject, Injectable } from '@nestjs/common';

@Injectable()
export class CatsRepository {
  constructor(@Inject('CONNECTION') connection: Connection) {}
}
```

### Overriding Default Provider’s Value

The default provider’s value can be overriden  (for mocking etc.) by simplying using the existing class as token, but use the mock as `useValue`.

```typescript
import { CatsService } from './cats.service';

const mockCatsService = {};
const catsServiceProvider = {
	// Token used by NestJS to resolve dependency
  privde: CatsService,
  // Actual value
  useValue: mockCatsService,
};

@Module({
  imports: [CatsModule],
  providers: [catsServiceProvider],
})
```

## Use Class

The `useClass` syntax allows using different classes based on different factors. For example, there may be an abstract / default `ConfigService` class. Depending on current environment, NestJS can use different implementation of the `ConfigService`.

```typescript
const configServiceProvider = {
  provide: ConfigService,
  useClass: process.env.NODE_ENV === 'development'
  	? DevelopmentConfigService
  	: ProductionConfigService,
};

@Module({
  providers: [configServiceProvider],
})
export class ApplicationModule {}
```

## Use Factory

The `useFactory` syntax allows creating providers *dynamically*.

- The factory function can either depend on several different providers or be independent.
- The factory may accept arguments which NestJS will resolve.
- The factory function can return values asynchronously.

```typescript
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    retun new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
})
export class ApplicationModule {}
```

> Should the factory function depend on other providers, their tokens must be passed inside the `inject` array.

## Use Existing

The `useExisting` syntax allows creating aliases for existing providers.

```typescript
@Injectable()
class LoggerService {};

const loggerAliasProvider = {
  provide: 'AliasedLoggerService',
  useExisting: LoggerService,
};

@Module({
  providers: [LoggerService, loggerAliasProvider],
})
export class ApplicationModule {}
```

## Export Custom Provider

A custom provider can be exported via either a token or as an object.

```typescript
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: ['CONNECTION'],
})
export class ApplicationModule {}
```

Alternatively,

```typescript

```

