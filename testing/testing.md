# Testing

NestJS is intergrated with Jest.

## Dependencies

```bash
yarn add --dev @nestjs/testing
```

## Unit Testing

```typescript
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController: CatsController;
  let catsService: CatsService;
  
  beforeEach(() => {
    catsService = new CatsService();
    catsController = new CatsController(catsService);
  });
  
  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findALl')
          .mockImplementation(() => result);
      expect(await catsControoler.findAll()).toBe(result);
    });
  });
});
```

> Test files should be located near tested classes, with `.spec` or `.test` suffix.

The previous test is **isolated** as it does not use NestJS utilities. It also demonstrates why DI is useful since the dependency `CatsService <- CatsController` is explicit and can be mocked easily.

## Testing Utilities

NestJS provides `@nestjs/testing` utilities, with `Test` class.

```typescript
import { test } from '@nestjs/testing';
// ...

// ...
	beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [CatsController],
      providers: [CatsService],
    }).compile();
    
    catsService = module.get<CatsService>(CatsService);
    catsController = module.get<CatsController>(CatsController);
  });

	// ...
		const result = ['test'];
		jest.spyOn(catsService, 'findAll');
			  .mockImplementation(() => result);
		expect(await catsController.findAll()).toBe(result);
```

`Test` has `createTestingModule()` generates a `TestingModule` instance which has an async `compile()` method.

A real instance can be mocked by overriding existing provider with a custom provider.

## End-to-end Testing

The `supertest` library allows simulating HTTP requests.

```typescript
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { CatsModule } from '../../src/cats/cats.module';
import { CatsService } from '../../src/cats/cats.service';
import { INestApplication } from '@nestjs/common';

describe('cats', () => {
  let app: INestApplication;
  let catsService = { findAll: () => ['test'] };
  
  beforeAll(async () => {
  	const module = await Test.createTestingModule({
      imports: [CatsModule]
    })
    	.overrideProvider(CatsService)
    	.useValue(catsService)
    	.compile();
    
    app = module.createNestApplication();
    await app.init();
  });
  
  it('GET /cats', () => {
    return request(app.getHttpServer())
    	.get('/cats')
    	.expect(200)
    	.expect({
      	data: catsService.findAll(),
	    });
  });
  
  afterAll(async () => {
    await app.close();
  });
});
```

> E2E tests (integration tests) should be kept under the `e2e` directory, with `.e2e-spec` or `.e2e-test` suffixes.

