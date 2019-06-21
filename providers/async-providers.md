# Asynchronous Providers

If the application must only start when some **asynchronous tasks** are completed (e.g. establishing database connection), **asynchronous providers** (hereafter abbreviated as **async providers**) should be used.

An async provider can be created via `useFactory` syntax, with the factory function returning a `Promise`.

```typescript
const asyncConnectionProvider = {
  provide: `AYSYNC_CONNECTION`,
  useFactory: async () => {
    const connection = await createConnection(options);
    return connection;
  },
};
```

## Injection

The async provider can be injected to other components via its token. Dependents of the async provider will only be instantiated after the async provider is **resolved**.