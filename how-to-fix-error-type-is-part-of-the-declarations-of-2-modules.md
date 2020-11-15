---
description: How to fix Error: Type is part of the declarations of 2 modules
---

# How to fix Error: Type is part of the declarations of 2 modules

If you encounter the issue, highly likely it means that a mock declaration,
usually a mock module, contains something, that is declared in the `TestBed` module directly.

Let's imagine a situation that we have a module which exports declarations, for example directives, we need in our test.
In the same time, we have another module that has another declarations, our component depends on,
we would like to create a mock copy out of it, but, in the same time, it imports the same module we want to keep as it is
via an import in `TestBed`.

```typescript
TestBed.configureTestingModule({
  imports: [
    SharedModule,
    MockModule(ModuleWithServicesAndSharedModule),
  ],
  declarations: [ComponentToTest],
});
```

The problem is clear: when we create the mock module, [`MockModule`](https://www.npmjs.com/package/ng-mocks#how-to-create-a-mock-module) recursively creates its mock dependencies, and, therefore, it creates a mock copy of `SharedModule`.
Now imported and mock declarations are part of 2 modules.

To solve this, we need to let [`MockModule`](https://www.npmjs.com/package/ng-mocks#how-to-create-a-mock-module) know, that `SharedModule` should stay as it is.

There are good and bad news.
The bad news is that [`MockModule`](https://www.npmjs.com/package/ng-mocks#how-to-create-a-mock-module) does not support that,
but the good news is that `ng-mocks` has [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder) for such a complicated case.
The only problem now is to rewrite `beforeEach` to use [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder) instead of [`MockModule`](https://www.npmjs.com/package/ng-mocks#how-to-create-a-mock-module).
A possible solution might looks like:

```typescript
beforeEach(() =>
  MockBuilder(ComponentToTest)
    .keep(SharedModule)
    .mock(ModuleWithServicesAndSharedModule)
);
```

The configuration says that we want to test `ComponentToTest`, which depends on `SharedModule` and `ModuleWithServicesAndSharedModule`, but `SharedModule` should stay as it is.

Now, during the building process, [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder) will keep `SharedModule` as it is, although it is a dependency of the mock module, and that avoids declarations of the same things in 2 modules.

More detailed information how to use it you can find in the section called [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder).

[back to the homepage](./)
