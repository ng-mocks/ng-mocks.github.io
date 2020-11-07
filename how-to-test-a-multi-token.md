---
description: How to test a multi token in Angular application
---

# How to test a multi token in Angular application

If you did not read ["How to test a token"](./how-to-test-a-token.html), please do it first.

Testing multi tokens is quite similar with the difference that `TestBet.get` returns an array of all providers
matching the token.

```typescript
const values = TestBed.get(TOKEN_MULTI);
expect(values).toEqual(jasmine.any(Array));
expect(values.length).toEqual(4);
```

---

A source file of this test is here:
[TestMultiToken](https://github.com/ike18t/ng-mocks/blob/master/examples/TestMultiToken/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestMultiToken/test.spec.ts)
to play with.

```typescript
import { Injectable, InjectionToken, NgModule } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { MockBuilder } from 'ng-mocks';

const TOKEN_MULTI = new InjectionToken('MULTI');

class ServiceClass {
  public readonly name = 'class';
}

@Injectable()
class ServiceExisting {
  public readonly name = 'existing';
}

// A module that provides all services.
@NgModule({
  providers: [
    ServiceExisting,
    {
      multi: true,
      provide: TOKEN_MULTI,
      useClass: ServiceClass,
    },
    {
      multi: true,
      provide: TOKEN_MULTI,
      useExisting: ServiceExisting,
    },
    {
      multi: true,
      provide: TOKEN_MULTI,
      useFactory: () => 'FACTORY',
    },
    {
      multi: true,
      provide: TOKEN_MULTI,
      useValue: 'VALUE',
    },
  ],
})
class TargetModule {}

describe('TestMultiToken', () => {
  // Because we want to test the token, we pass it as the first
  // parameter of MockBuilder. To correctly satisfy its initialization
  // we need to pass its module as the second parameter.
  beforeEach(() => MockBuilder(TOKEN_MULTI, TargetModule));

  it('creates TOKEN_MULTI', () => {
    const tokens = TestBed.get(TOKEN_MULTI);

    expect(tokens).toEqual(jasmine.any(Array));
    expect(tokens.length).toEqual(4);

    // Verifying that the token is an instance of ServiceClass.
    expect(tokens[0]).toEqual(jasmine.any(ServiceClass));
    expect(tokens[0].name).toEqual('class');

    // Verifying that the token is an instance of ServiceExisting.
    // But because it has been replaced with its mock copy 
    // we should see an empty name.
    expect(tokens[1]).toEqual(jasmine.any(ServiceExisting));
    expect(tokens[1].name).toBeUndefined();

    // Checking that we have here what factory has been created.
    expect(tokens[2]).toEqual('FACTORY');

    // Checking the set value.
    expect(tokens[3]).toEqual('VALUE');
  });
});
```

[back to the homepage](./)
