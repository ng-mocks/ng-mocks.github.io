---
description: How to test a structural directive in Angular application
---

# How to test a structural directive in Angular application

Structural directives influence render of their child view, you definitely have used `*ngIf`, that's the thing.
Steps for testing are quite close to the steps for testing attribute directives: we need to render a custom template and
then to assert behavior of the directive.

Let's configure `TestBed`. The first parameter for [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder) is the directive itself,
and if it has dependencies we should pass its module as the second parameter:

```typescript
beforeEach(() => MockBuilder(TargetDirective));
```

The next step is to render a custom template. Let's assume that the selector of the directive is `[target]`.
Now let's render it in a test:

```typescript
const fixture = MockRender(
  `<div *target="value">
    content
  </div>`,
  {
    value: false,
  }
);
```

Let's assert that the content has not been rendered.

```typescript
expect(fixture.nativeElement.innerHTML).not.toContain('content');
```

With help of [`MockRender`](https://www.npmjs.com/package/ng-mocks#mockrender) we can access the element of the directive via `fixture.point` to cause events on,
and we can change the `value` via `fixture.componentInstance.value`.

```typescript
fixture.componentInstance.value = true;
```

Because `value` is `true` now, the content should be rendered.

```typescript
fixture.detectChanges();
expect(fixture.nativeElement.innerHTML).toContain('content');
```

---

A source file of this test is here:
[TestStructuralDirective](https://github.com/ike18t/ng-mocks/blob/master/examples/TestStructuralDirective/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/ng-mocks/examples?file=/src/examples/TestStructuralDirective/test.spec.ts)
to play with.

```typescript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';
import { MockBuilder, MockRender } from 'ng-mocks';

// This directive is the same as `ngIf`,
// it renders its content only when its input has truly value.
@Directive({
  selector: '[target]',
})
class TargetDirective {
  protected templateRef: TemplateRef<any>;
  protected viewContainerRef: ViewContainerRef;

  constructor(templateRef: TemplateRef<any>, viewContainerRef: ViewContainerRef) {
    this.templateRef = templateRef;
    this.viewContainerRef = viewContainerRef;
  }

  @Input() set target(value: any) {
    if (value) {
      this.viewContainerRef.createEmbeddedView(this.templateRef);
    } else {
      this.viewContainerRef.clear();
    }
  }
}

describe('TestStructuralDirective', () => {
  // Because we want to test the directive, we pass it as the first
  // parameter of MockBuilder. We can omit the second parameter,
  // because there are no dependencies.
  beforeEach(() => MockBuilder(TargetDirective));

  it('hides and renders its content', () => {
    const fixture = MockRender(
      `
        <div *target="value">
          content
        </div>
    `,
      {
        value: false,
      }
    );

    // Because the value is false the "content" should not be
    // rendered.
    expect(fixture.nativeElement.innerHTML).not.toContain('content');

    // Let's change the value to true and assert that the "content"
    // has been rendered.
    fixture.componentInstance.value = true;
    fixture.detectChanges();
    expect(fixture.nativeElement.innerHTML).toContain('content');

    // Let's change the value to false and assert that the
    // "content" has been hidden.
    fixture.componentInstance.value = false;
    fixture.detectChanges();
    expect(fixture.nativeElement.innerHTML).not.toContain('content');
  });
});
```

[back to the homepage](./)
