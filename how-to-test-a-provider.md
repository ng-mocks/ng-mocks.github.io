---
description: How to test a provider in Angular application
---

# How to test a provider in Angular application

Usually, you don't need `TestBed` if you want to test a simple
provider, the best way would be to write isolated pure unit tests.

Nevertheless, [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder) might help here too. If a provider has complex dependencies, or you want to verify
that its module creates the provider in a particular way, then simply pass the provider as the first parameter and its module as the second one.

```typescript
beforeEach(() => MockBuilder(TargetService, TargetModule));
```

In a test you need to use the global injector for getting an instance of the service to assert its behavior:

```typescript
const service = TestBed.get(TargetService);
expect(service.echo()).toEqual(service.value);
```

What might be useful here is knowledge of how to customize the dependencies.
There are 3 options: `.mock`, `.provide` and `MockInstance`. All of them are a good choice:

```typescript
beforeEach(() =>
  MockBuilder(TargetService, TargetModule)
    .mock(Service2, {
      trigger: () => 'mock2',
    })
    .provide({
      provide: Service3,
      useValue: {
        trigger: () => 'mock3',
      },
    })
);
```

```typescript
beforeAll(() => {
  MockInstance(Service1, {
    init: instance => {
      instance.trigger = () => 'mock1';
    },
  });
});
```

Despite the way providers are created: `useClass`, `useValue` etc.
Their tests are quite similar.

---

A source file of a test without dependencies is here:
[TestProvider](https://github.com/ike18t/ng-mocks/blob/master/examples/TestProvider/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestProvider/test.spec.ts)
to play with.

A source file of a test with dependencies is here:
[TestProviderWithDependencies](https://github.com/ike18t/ng-mocks/blob/master/examples/TestProviderWithDependencies/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestProviderWithDependencies/test.spec.ts)
to play with.

A source file of a test with `useClass` is here:
[TestProviderWithUseClass](https://github.com/ike18t/ng-mocks/blob/master/examples/TestProviderWithUseClass/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestProviderWithUseClass/test.spec.ts)
to play with.

A source file of a test with `useValue` is here:
[TestProviderWithUseValue](https://github.com/ike18t/ng-mocks/blob/master/examples/TestProviderWithUseValue/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/examples/TestProviderWithUseValue/test.spec.ts)
to play with.

A source file of a test with `useExisting` is here:
[TestProviderWithUseExisting](https://github.com/ike18t/ng-mocks/blob/master/examples/TestProviderWithUseExisting/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestProviderWithUseExisting/test.spec.ts)
to play with.

A source file of a test with `useFactory` is here:
[TestProviderWithUseFactory](https://github.com/ike18t/ng-mocks/blob/master/examples/TestProviderWithUseFactory/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/examples/TestProviderWithUseFactory/test.spec.ts)
to play with.

```typescript
import { Injectable } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { MockBuilder } from 'ng-mocks';

// A simple service, might have contained more logic,
// but it is redundant for the test demonstration.
@Injectable()
class TargetService {
  public readonly value = true;

  public echo(): boolean {
    return this.value;
  }
}

describe('TestProvider', () => {
  beforeEach(() => MockBuilder(TargetService));

  it('returns value on echo', () => {
    const service = TestBed.get(TargetService);

    expect(service.echo()).toEqual(service.value);
  });
});
```

[back to the homepage](./)
