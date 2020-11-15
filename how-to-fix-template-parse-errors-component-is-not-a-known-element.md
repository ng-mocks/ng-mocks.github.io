---
description: How to fix Template parse errors: \<component\> is not a known element
---

# How to fix Template parse errors: \<component\> is not a known element

This error might happen in a test when we have a mock module of the module
a testing component depends on, but its declarations have not been exported.

{% raw %}
```typescript
@NgModule({
  declarations: [DependencyComponent],
})
class MyModule {}
```
```typescript
beforeEach(() => {
  TestBed.configureTestingModule({
    declarations: [
      MyComponent, // <- the only declaration we care about.
    ],
    imports: [
      MockModule(MyModule),
    ],
  });
  return TestBed.compileComponents();
});
```
{% endraw %}

In this case, a test will throw `Template parse errors: <DependencyComponent> is not a known element`.

The problem here is that `DependencyComponent` isn't exported,
and to get access to a mock `DependencyComponent` we need either
to declare it on the same level where `MyComponent` has been declared
or to export `DependencyComponent`.

there are 3 solutions to do it:

1. to call [`MockComponent`](https://www.npmjs.com/package/ng-mocks#how-to-create-a-mock-component) on it directly in the `TestBed`

   {% raw %}
   ```typescript
   beforeEach(() => {
     TestBed.configureTestingModule({
       declarations: [
         MyComponent,
         MockComponent(DependencyComponent),
       ],
     });
     return TestBed.compileComponents();
   });
   ```
   {% endraw %}

2. to use [`ngMocks.guts`](https://www.npmjs.com/package/ng-mocks#ngmocksguts),
   it does the same things as the first solution,
   but provides mocks of all imports and declarations from `MyModule`.
   
   {% raw %}
   ```typescript
   beforeEach(() => {
     TestBed.configureTestingModule(ngMocks.guts(MyComponent, MyModule));
     return TestBed.compileComponents(); 
   });
   ```
   {% endraw %}

3. to use [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder),
   its behavior differs from the solutions above. It creates a mock `MyModule`,
   that exports all its imports and declarations including a mock `DependencyComponent`. 
   
   {% raw %}
   ```typescript
   beforeEach(() => MockBuilder(MyComponent, MyModule));
   ```
   {% endraw %}

Profit. More detailed information about pros and cons of each approach you can read in
[motivation and easy start from ng-mocks](https://www.npmjs.com/package/ng-mocks#motivation-and-easy-start).

[back to the homepage](./)
