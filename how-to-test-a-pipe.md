---
description: How to test a pipe in Angular application
---

# How to test a pipe in Angular application

An approach with testing pipes is similar to directives. We pass the pipe as the first parameter of [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder),
and its module with dependencies as the second one if they exist:

```typescript
beforeEach(() => MockBuilder(TargetPipe));
```

To verify how the pipe behaives we need to render a custom template:

{% raw %}
```typescript
const fixture = MockRender(`{{ values | target}}`, {
  values: ['1', '3', '2'],
});
```
{% endraw %}

Now we can assert what has been rendered:

```typescript
expect(fixture.nativeElement.innerHTML).toEqual('1, 2, 3');
```

---

A source file of this test is here:
[TestPipe](https://github.com/ike18t/ng-mocks/blob/master/examples/TestPipe/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/ng-mocks/examples?file=/src/examples/TestPipe/test.spec.ts)
to play with.

{% raw %}
```typescript
import { Pipe, PipeTransform } from '@angular/core';
import { MockBuilder, MockRender, ngMocks } from 'ng-mocks';

// A simple pipe that accepts an array of strings, sorts them,
// and returns a joined string of the values.
@Pipe({
  name: 'target',
})
class TargetPipe implements PipeTransform {
  transform(value: string[], asc = true): string {
    let result = [...(value || [])].sort();
    if (!asc) {
      result = result.reverse();
    }

    return result.join(', ');
  }
}

describe('TestPipe', () => {
  ngMocks.faster(); // the same TestBed for several its.

  // Because we want to test the pipe, we pass it as the first
  // parameter of MockBuilder. We can omit the second parameter,
  // because there are no dependencies.
  beforeEach(() => MockBuilder(TargetPipe));

  it('sorts strings', () => {
    const fixture = MockRender(`{{ values | target}}`, {
      values: ['1', '3', '2'],
    });

    expect(fixture.nativeElement.innerHTML).toEqual('1, 2, 3');
  });

  it('reverses strings on param', () => {
    const fixture = MockRender(`{{ values | target:flag}}`, {
      flag: false,
      values: ['1', '3', '2'],
    });

    expect(fixture.nativeElement.innerHTML).toEqual('3, 2, 1');
  });
});
```
{% endraw %}

[back to the homepage](./)
