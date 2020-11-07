---
description: How to test an attribute directive in Angular application
---

# How to test an attribute directive in Angular application

Attribute directives usually manipulate DOM, or mark a group of similar elements.
For a test, it means we need to render a custom template, where we use the directive, then we can assert its behavior.

Therefore, we should mock everything except the directive.
Or we can simply pass it alone if the directive does not have dependencies:

```typescript
beforeEach(() => MockBuilder(TargetDirective));
```

The next step is to render a custom template. Let's assume that the selector of the directive is `[target]`.
Now let's render it in a test:

```typescript
const fixture = MockRender(`<div target></div>`);
```

Because we use [`MockRender`](https://www.npmjs.com/package/ng-mocks#mockrender) we know that the element with the directive can be accessed by
`fixture.point`.

Now we can cause events the directive listens on,
or to access its instance for further assertions:

```typescript
fixture.point.triggerEventHandler('mouseenter', null);
const instance = ngMocks.get(fixture.point, TargetDirective);
```

---

A source file of this test is here:
[TestAttributeDirective](https://github.com/ike18t/ng-mocks/blob/master/examples/TestAttributeDirective/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestAttributeDirective/test.spec.ts)
to play with.

```typescript
import { Directive, ElementRef, HostListener, Input } from '@angular/core';
import { MockBuilder, MockRender, ngMocks } from 'ng-mocks';

// The purpose of the directive is to add a background color
// on mouseenter and to remove it on mouseleave.
// By default the color is yellow.
@Directive({
  selector: '[target]',
})
class TargetDirective {
  @Input() public color = 'yellow';

  protected ref: ElementRef;

  constructor(ref: ElementRef) {
    this.ref = ref;
  }

  @HostListener('mouseenter') onMouseEnter() {
    this.ref.nativeElement.style.backgroundColor = this.color;
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.ref.nativeElement.style.backgroundColor = null;
  }
}

describe('TestAttributeDirective', () => {
  ngMocks.faster(); // the same TestBed for several its.

  // Because we want to test the directive, we pass it as the first
  // parameter of MockBuilder. We can omit the second parameter,
  // because there are no dependencies.
  beforeEach(() => MockBuilder(TargetDirective));

  it('uses default background color', () => {
    const fixture = MockRender(`<div target></div>`);

    // By default, without the mouse enter, there is no background
    // color on the div.
    expect(fixture.nativeElement.innerHTML).not.toContain('style="background-color: yellow;"');

    // Let's simulate the mouse enter event.
    // fixture.point is out root element from the rendered template,
    // therefore it points to the div we want to trigger the event
    // on.
    fixture.point.triggerEventHandler('mouseenter', null);

    // Let's assert the color.
    expect(fixture.nativeElement.innerHTML).toContain('style="background-color: yellow;"');

    // Now let's simulate the mouse mouse leave event.
    fixture.point.triggerEventHandler('mouseleave', null);

    // And assert that the background color is gone now.
    expect(fixture.nativeElement.innerHTML).not.toContain('style="background-color: yellow;"');
  });

  it('sets provided background color', () => {
    // When we want to test inputs / outputs we need to use the second
    // parameter of MockRender, simply pass there variables for the
    // template, they'll become properties of
    // fixture.componentInstance.
    const fixture = MockRender(`<div [color]="color" target></div>`, {
      color: 'red',
    });

    // Let's assert that the background color is red.
    fixture.point.triggerEventHandler('mouseenter', null);
    expect(fixture.nativeElement.innerHTML).toContain('style="background-color: red;"');

    // Let's switch the color, we don't need `.point`, because we
    // access a middle component of MockRender.
    fixture.componentInstance.color = 'blue';
    fixture.detectChanges(); // shaking the template
    fixture.point.triggerEventHandler('mouseenter', null);
    expect(fixture.nativeElement.innerHTML).toContain('style="background-color: blue;"');
  });
});
```

[back to the homepage](./)
