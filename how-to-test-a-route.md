---
description: How to test a route in Angular application
---

# How to test a route in Angular application

Testing a route means that we want to assert that a specific page renders a specific component.

With `ng-mocks` you can be confident, that a route exists and all its dependencies are present in the related module,
otherwise tests will fail.

However, to test that, we need to configure `TestBed` a bit differently: it is fine to mock all components and declarations,
we should only keep the `RouterModule` as it is, and to add `RouterTestingModule` with empty routes.
This guarantees that the application routes will be used, and tests fail if a route or its dependencies have been removed.

```typescript
beforeEach(() =>
  MockBuilder(RouterModule, TargetModule).keep(
    RouterTestingModule.withRoutes([])
  )
);
```

The next and very import step is to wrap a test callback in `it` with `fakeAsync` function and to render `RouterOutlet`.
We need this, because `RouterModule` relies on async zones.

```typescript
// fakeAsync --------------------------|||||||||
it('renders /1 with Target1Component', fakeAsync(() => {
  const fixture = MockRender(RouterOutlet);
}));
```

After we have rendered `RouterOutlet` we should initialize the router, also we can set the default url here.
As mentioned above, we should use zones and `fakeAsync` for that.

```typescript
const router = TestBed.get(Router);
const location = TestBed.get(Location);

location.go('/1');
if (fixture.ngZone) {
  fixture.ngZone.run(() => router.initialNavigation());
  tick();
}
```

Now we can assert the current route and what it has rendered.

```typescript
expect(location.path()).toEqual('/1');
expect(() => ngMocks.find(fixture, Target1Component)).not.toThrow();
```

That's it.

Additionally, we might assert that a link on a page navigates to the right route.
In this case, we should pass a component of the link as the first parameter to [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder),
to `.keep` `RouterModule` and to render the component instead of `RouterOutlet`.

```typescript
beforeEach(() =>
  MockBuilder(TargetComponent, TargetModule)
    .keep(RouterModule)
    .keep(RouterTestingModule.withRoutes([]))
);
```

```typescript
it('navigates between pages', fakeAsync(() => {
  const fixture = MockRender(TargetComponent);
}));
```

The next step is to find the link we want to click. The click event should be inside of zones, because it triggers navigation.
Please notice, that `button: 0` should be sent with the event to simulate the left button click.

```typescript
const links = ngMocks.findAll(fixture, 'a');
if (fixture.ngZone) {
  fixture.ngZone.run(() => {
    links[0].triggerEventHandler('click', {
      button: 0, // <- simulating the left button click, not right one.
    });
  });
  tick();
}
```

Now we can assert the current state: the location should be changed to the expected route, and the fixture should contain
its component.

```typescript
expect(location.path()).toEqual('/1');
expect(() => ngMocks.find(fixture, Target1Component)).not.toThrow();
```

---

A source file of these tests is here:
[TestRoute](https://github.com/ike18t/ng-mocks/blob/master/examples/TestRoute/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/ng-mocks/examples?file=/src/examples/TestRoute/test.spec.ts)
to play with.

```typescript
import { Location } from '@angular/common';
import { Component, NgModule } from '@angular/core';
import { fakeAsync, TestBed, tick } from '@angular/core/testing';
import { Router, RouterModule, RouterOutlet } from '@angular/router';
import { RouterTestingModule } from '@angular/router/testing';
import { MockBuilder, MockRender, ngMocks } from 'ng-mocks';

// A layout component that renders the current route.
@Component({
  selector: 'target',
  template: `
    <a routerLink="/1">1</a>
    <a routerLink="/2">2</a>
    <router-outlet></router-outlet>
  `,
})
class TargetComponent {}

// A simple component for the first route.
@Component({
  selector: 'target1',
  template: 'target1',
})
class Target1Component {}

// A simple component for the second route.
@Component({
  selector: 'target2',
  template: 'target2',
})
class Target2Component {}

// Definition of the routing module.
@NgModule({
  declarations: [Target1Component, Target2Component],
  exports: [RouterModule],
  imports: [
    RouterModule.forRoot([
      {
        component: Target1Component,
        path: '1',
      },
      {
        component: Target2Component,
        path: '2',
      },
    ]),
  ],
})
class TargetRoutingModule {}

// Definition of the main module.
@NgModule({
  declarations: [TargetComponent],
  exports: [TargetComponent],
  imports: [TargetRoutingModule],
})
class TargetModule {}

describe('TestRoute:Route', () => {
  beforeEach(() => {
    return MockBuilder(RouterModule, TargetModule).keep(
      RouterTestingModule.withRoutes([]),
    );
  });

  it('renders /1 with Target1Component', fakeAsync(() => {
    const fixture = MockRender(RouterOutlet);
    const router: Router = TestBed.get(Router);
    const location: Location = TestBed.get(Location);

    // First we need to initialize navigation.
    location.go('/1');
    if (fixture.ngZone) {
      fixture.ngZone.run(() => router.initialNavigation());
      tick(); // is needed for rendering of the current route.
    }

    // We should see Target1Component component on /1 page.
    expect(location.path()).toEqual('/1');
    expect(() => ngMocks.find(Target1Component)).not.toThrow();
  }));

  it('renders /2 with Target2Component', fakeAsync(() => {
    const fixture = MockRender(RouterOutlet);
    const router: Router = TestBed.get(Router);
    const location: Location = TestBed.get(Location);

    // First we need to initialize navigation.
    location.go('/2');
    if (fixture.ngZone) {
      fixture.ngZone.run(() => router.initialNavigation());
      tick(); // is needed for rendering of the current route.
    }

    // We should see Target2Component component on /2 page.
    expect(location.path()).toEqual('/2');
    expect(() => ngMocks.find(Target2Component)).not.toThrow();
  }));
});

describe('TestRoute:Component', () => {
  // Because we want to test navigation, it means that we want to
  // test a component with 'router-outlet' and its integration with
  // RouterModule. Therefore, we pass the component as the first
  // parameter of MockBuilder. Then, to correctly satisfy its
  // initialization, we need to pass its module as the second
  // parameter. And, the last but not the least, we need keep
  // RouterModule to have its routes, and to add
  // RouterTestingModule.withRoutes([]), yes yes, with empty routes
  // to have tools for testing.
  beforeEach(() => {
    return MockBuilder(TargetComponent, TargetModule)
      .keep(RouterModule)
      .keep(RouterTestingModule.withRoutes([]));
  });

  it('navigates between pages', fakeAsync(() => {
    const fixture = MockRender(TargetComponent);
    const router: Router = TestBed.get(Router);
    const location: Location = TestBed.get(Location);

    // First we need to initialize navigation.
    if (fixture.ngZone) {
      fixture.ngZone.run(() => router.initialNavigation());
      tick(); // is needed for rendering of the current route.
    }

    // By default our routes do not have a component.
    // Therefore non of them should be rendered.
    expect(location.path()).toEqual('/');
    expect(() => ngMocks.find(Target1Component)).toThrow();
    expect(() => ngMocks.find(Target2Component)).toThrow();

    // Let's extract our navigation links.
    const links = ngMocks.findAll('a');

    // Checking where we land if we click the first link.
    if (fixture.ngZone) {
      fixture.ngZone.run(() => {
        // To simulate a correct click we need to let the router know
        // that it is the left mouse button via setting its parameter
        // to 0 (not undefined, null or anything else).
        links[0].triggerEventHandler('click', {
          button: 0,
        });
      });
      tick(); // is needed for rendering of the current route.
    }
    // We should see Target1Component component on /1 page.
    expect(location.path()).toEqual('/1');
    expect(() => ngMocks.find(Target1Component)).not.toThrow();

    // Checking where we land if we click the second link.
    if (fixture.ngZone) {
      fixture.ngZone.run(() => {
        // A click of the left mouse button.
        links[1].triggerEventHandler('click', {
          button: 0,
        });
      });
      tick(); // is needed for rendering of the current route.
    }
    // We should see Target2Component component on /2 page.
    expect(location.path()).toEqual('/2');
    expect(() => ngMocks.find(Target2Component)).not.toThrow();
  }));
});
```

[back to the homepage](./)
