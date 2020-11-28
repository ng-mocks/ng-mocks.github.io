---
description: How to test a routing resolver in Angular application
---

# How to test a routing resolver in Angular application

If you did not read ["How to test a route"](./how-to-test-a-route.html), please do it first.

When we want to test a resolver it means we need to mock everything except the resolver and `RouterModule`.
Optionally, we can disable guards to avoid influence of their mocked methods returning falsy values and blocking routes.

```typescript
beforeEach(() =>
  MockBuilder(DataResolver, TargetModule)
    .exclude(NG_MOCKS_GUARDS)
    .keep(RouterModule)
    .keep(RouterTestingModule.withRoutes([]))
);
```

To test the resolver we need to render `RouterOutlet`.

```typescript
const fixture = MockRender(RouterOutlet);
```

Additionally, we also need to properly customize mocked services if the resolver is using them to fetch data.

```typescript
const dataService = TestBed.get(DataService);

dataService.data = () => from([false]);
```

The next step is to go to the route where the resolver is, and to trigger initialization of the router.

```typescript
const location = TestBed.get(Location);
const router = TestBed.get(Router);

location.go('/target');
if (fixture.ngZone) {
  fixture.ngZone.run(() => router.initialNavigation());
  tick();
}
```

Because data is provided to a particular route, we need to find its component in the `fixture` and
to extract `ActivatedRoute` from its injector.
Let's pretend that `/target` renders `TargetComponent`.

```typescript
const el = ngMocks.find(fixture, TargetComponent);
const route: ActivatedRoute = el.injector.get(ActivatedRoute);
```

Profit, now we can assert the data the resolver has provided.

```typescript
expect(route.snapshot.data).toEqual({
  data: {
    flag: false,
  },
});
```

---

A source file of this test is here:
[TestRoutingResolver](https://github.com/ike18t/ng-mocks/blob/master/examples/TestRoutingResolver/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/ng-mocks/examples?file=/src/examples/TestRoutingResolver/test.spec.ts)
to play with.

```typescript
import { Location } from '@angular/common';
import { Component, Injectable, NgModule } from '@angular/core';
import { fakeAsync, TestBed, tick } from '@angular/core/testing';
import {
  ActivatedRoute,
  Resolve,
  Router,
  RouterModule,
  RouterOutlet,
} from '@angular/router';
import { RouterTestingModule } from '@angular/router/testing';
import { MockBuilder, MockRender, ngMocks } from 'ng-mocks';
import { combineLatest, from, Observable, of } from 'rxjs';
import { map } from 'rxjs/operators';

// A simple service simulating a data request.
@Injectable()
class DataService {
  protected flag = true;

  public data(): Observable<boolean> {
    return from([this.flag]);
  }
}

// A resolver we want to test.
@Injectable()
class DataResolver implements Resolve<{ flag: boolean }> {
  public constructor(protected service: DataService) {}

  public resolve() {
    return combineLatest([this.service.data()]).pipe(
      map(([flag]) => ({ flag })),
    );
  }
}

// A resolver we want to ignore.
@Injectable()
class MockResolver implements Resolve<{ mock: boolean }> {
  protected mock = true;

  public resolve() {
    return of({ mock: this.mock });
  }
}

// A dummy component.
// It will be replaced with a mock copy.
@Component({
  selector: 'target',
  template: 'target',
})
class TargetComponent {}

// Definition of the routing module.
@NgModule({
  declarations: [TargetComponent],
  exports: [RouterModule],
  imports: [
    RouterModule.forRoot([
      {
        component: TargetComponent,
        path: 'target',
        resolve: {
          data: DataResolver,
          mock: MockResolver,
        },
      },
    ]),
  ],
  providers: [DataService, DataResolver, MockResolver],
})
class TargetModule {}

describe('TestRoutingResolver', () => {
  // Because we want to test the resolver, it means that we want to
  // test its integration with RouterModule. Therefore, we pass
  // the resolver as the first parameter of MockBuilder. Then, to
  // correctly satisfy its initialization, we need to pass its module
  // as the second parameter. And, the last but not the least, we
  // need to keep RouterModule to have its routes, and to
  // add RouterTestingModule.withRoutes([]), yes yes, with empty
  // routes to have tools for testing.
  beforeEach(() => {
    return MockBuilder(DataResolver, TargetModule)
      .keep(RouterModule)
      .keep(RouterTestingModule.withRoutes([]));
  });

  // It is important to run routing tests in fakeAsync.
  it('provides data to on the route', fakeAsync(() => {
    const fixture = MockRender(RouterOutlet);
    const router: Router = TestBed.get(Router);
    const location: Location = TestBed.get(Location);
    const dataService: DataService = TestBed.get(DataService);

    // DataService has been replaced with a mock copy,
    // let's set a custom value we will assert later on.
    dataService.data = () => from([false]);

    // Let's switch to the route with the resolver.
    location.go('/target');

    // Now we can initialize navigation.
    if (fixture.ngZone) {
      fixture.ngZone.run(() => router.initialNavigation());
      tick(); // is needed for rendering of the current route.
    }

    // Checking that we are on the right page.
    expect(location.path()).toEqual('/target');

    // Let's extract ActivatedRoute of the current component.
    const el = ngMocks.find(TargetComponent);
    const route: ActivatedRoute = el.injector.get(ActivatedRoute);

    // Now we can assert that it has expected data.
    expect(route.snapshot.data).toEqual(
      jasmine.objectContaining({
        data: {
          flag: false,
        },
      }),
    );
  }));
});
```

[back to the homepage](./)
