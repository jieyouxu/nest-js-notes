# Exception Filters

Nest has a **exceptions layer** which captures all unhandled exception for the application.

- If the exception is not catched by application, Nest automatically sends a response.

![Exception Filters](img/exception-filter-1.png)

- Unhandled `HttpException` exceptions (and subclasses) are caught by the **global exception filter** by default.
- Unrecognized exceptions return a automatic `500 Internal Server Error` JSON response.

```json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

## Base Exceptions

`HttpException` is the **base exception**.

The `CatsController` could throw

```typescript
@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```

The client calling this `GET /cats` endpoint will receive

```json
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

The `HttpException` takes the `message` and `statusCode`. It may be overriden by passing a object instead of string for `message`.

```typescript
@Get()
async findAll() {
  throw new HttpException({
    status: HttpStatus.FORBIDDEN,
    error: 'This is a custom message',
  }, 403);
}
```

## Exceptions Hierarchy

Custom exceptions should help to model the domain.

```typescript
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

```typescript
@Get()
async findAll() {
  throw new ForbiddenException();
}
```

## HTTP Exceptions

Nest provides some derived `HttpException`s from `@nestjs/common`.

- `BadRequestException`
- `UnauthorizedException`
- `NotFoundException`
- `ForbiddenException`
- `NotAcceptableException`
- `RequestTimeoutException`
- `ConflictException`
- `GoneException`
- `PayloadTooLargeException`
- `UnsupportedMediaTypeException`
- `UnprocessableEntityException`
- `InternalServerErrorException`
- `NotImplementedException`
- `BadGatewayException`
- `ServiceUnavailableException`
- `GatewayTimeoutException`

## Exception Filters

Custom exception filters can take precedence over the default global filter for custom behaviour.