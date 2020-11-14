---
description: How to test a provider of a component in Angular application
---

# How to test a provider of a component in Angular application

If we have a component with providers for testing, we need to mock everything
except the provider:

```typescript
beforeEach(() => MockBuilder(TargetService, TargetComponent));
```

This code will setup `TestBed` with a mocked copy of `TargetComponent`, but leave `TargetService` as it is,
so we would be able to assert it.

In the test we need to render the mocked component, find its element in the fixture and extract the service from the element.
If we use [`MockRender`](https://www.npmjs.com/package/ng-mocks#mockrender) we can access the element of the component via `fixture.point`.

```typescript
const fixture = MockRender(TargetComponent);
const service = fixture.point.injector.get(TargetService);
```

Profit. Now we can assert behavior of the service.

---

A source file of this test is here:
[TestProviderInComponent](https://github.com/ike18t/ng-mocks/blob/master/examples/TestProviderInComponent/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/ng-mocks/examples?file=/src/examples/TestProviderInComponent/test.spec.ts)
to play with.

{% raw %}
```typescript
import { Component, Injectable } from '@angular/core';
import { MockBuilder, MockRender } from 'ng-mocks';

// A simple service, might have contained more logic,
// but it is redundant for the test demonstration.
@Injectable()
class TargetService {
  public readonly value = 'target';
}

@Component({
  providers: [TargetService],
  selector: 'target',
  template: `{{ service.value }}`,
})
class TargetComponent {
  public readonly service: TargetService;

  constructor(service: TargetService) {
    this.service = service;
  }
}

describe('TestProviderInComponent', () => {
  // Because we want to test the service, we pass it as the first
  // parameter of MockBuilder.
  // Because we do not care about TargetComponent, we pass it as
  // the second parameter for being replaced with a mock copy.
  beforeEach(() => MockBuilder(TargetService, TargetComponent));

  it('has access to the service via a component', () => {
    // Let's render the mock component. It provides a point
    // to access the service.
    const fixture = MockRender(TargetComponent);

    // The root element is fixture.point and it is the TargetComponent
    // with its injector for extracting internal services.
    const service = fixture.point.injector.get(TargetService);

    // Here we go, now we can assert everything about the service.
    expect(service.value).toEqual('target');
  });
});
```
{% endraw %}

[back to the homepage](./)
