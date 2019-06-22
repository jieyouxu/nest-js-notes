# Pipes

A **pipe** is an `@Injectable()` class which implements the `PipeTransform` interface.

![Pipes](img/pipe-1.png)

Pipes are usually used for:

- **Transformation**: input data -> desired output.
- **Validation**: evaluate validity of input data (then pass it), or throw exception.

Pipes operate on `arguments` processed by controller route handler.

## Built-in Pipes

`ValidationPipe` and `ParseIntPipe` is built-in, available from `@nestjs/common`.

```typescript
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value;
  }
}
```

Each pipe must provide `transform(value, metadata)`:

- `value` is the argument before received by the route handler
- `metadata` has properties of `ArgumentMetadata`

```typescript
export interface ArgumentMetadata {
  readonly type: 'body' | 'query' | 'param' | 'custom';
  readonly metatype?: Type<any>;
  readonly data?: string;
}
```

| Property   | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `type`     | Type of argument, either `req.body`, `req.query`, `req.param` or a custom parameter. |
| `metatype` | Metatype of argument, e.g. `string`.                         |
| `data`     | String passed to decorator, e.g. `@Body('string')`.          |

## Validation

For the `POST /cats` `create()` handler from `CatsController` where

```typescript
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

The handler requires a `CreateCatDto` for request body, specified as

```typescript
export class CreateCatDto {
  readonly name: string;
  readonly age: number;
  readonly breed: string;
}
```

We need to check if the request body satisfies the `CreateCatDto` shape, but if the route handler validates this, it has multiple responsibilites which violdates SRP.

- A validation class which is delegated to would need to be called at start of each method, which is repetitive.
- A generic middleware cannot be created, since middlewares do not have access to execution context (i.e. the handler being called).
- Ideal use case for a **pipe**.

## Object Schema Validation

One way of object validation is **schema-based validation**.

- `Joi` library.

A **validation pipe** yields a `forall a. Either Error a` kind of return value:

- Either the value is returned unchanged, or
- An exception is thrown.

```typescript
import * as Joi from 'joi';
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class JoiValidationPipe implements PipeTransform {
  constructor(private readonly schema: Object) {}

  transform(value: any, metadata: ArgumentMetadata) {
    const { error } = Joi.validate(value, this.schema);
    if (error) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }
}
```

## Binding Pipes

Pipes can be bound to controllers or handlers by `@UsePipes()` decorator and creating a pipe instance.

```typescript
@Post()
@UsePipes(new JoiValidationPipe(createCatSchema))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

## Class Validator

The `class-validator` library can be used to perform decorator-based validation.

> `npm i --save class-validator class-transformer`

```typescript
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  readonly name: string;

  @IsInt()
  readonly age: number;

  @IsString()
  readonly breed: string;
}
```

```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToClass } from 'class-transformer';

@Injectable()
export class ValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }
    const object = plainToClass(metatype, value);
    const errors = await validate(object);
    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

- `transform()` is `async` since asynchronous pipes are supported as well.

A pipe can have the scopes of:

- Method-scoped
- Controller-scoped
- Global-scoped
- Parameter-scoped

```typescript
@Post()
async create(
  @Body(new ValidationPipe()) createCatDto: CreateCatDto,
) {
  this.catsService.create(createCatDto);
}
```

The validation pipe is bound to the `@Body()` decorator.

Or, at method scope,

```typescript
@Post()
@UsePipes(new ValidationPipe())
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

The class can also be passed in for DI

```typescript
@Post()
@UsePipes(ValidationPipe)
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

The validation pipe is rather general, so it can be globally-scoped

```typescript
async function bootstrap() {
  const app = await NestFactory.create(ApplicationModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

## Transformation

`ParseIntPipe`:

```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}
```

This pipe can be applied to parameters:

```typescript
@Get(':id')
async findOne(@Param('id', new ParseIntPipe()) id) {
  return await this.catsService.findOne(id);
}
```

Selecting existing user entity from database by id:

```typescript
@Get(':id')
findOne(@Param('id', UserByIdPipe) userEntity: UserEntity) {
  return userEntity;
}
```

## Built-in Validation Pipe

Pass `{ transform: true }` to `ValidationPipe`.

```typescript
@Post()
@UsePipes(new ValidationPipe({ transform: true }))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

