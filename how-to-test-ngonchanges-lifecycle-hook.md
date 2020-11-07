---
description: How to test ngOnChanges lifecycle hook in Angular application
---

# How to test ngOnChanges lifecycle hook in Angular application

`TestBed.createComponent` does not support `ngOnChanges` out of the box.
That is where [`MockRender`](https://www.npmjs.com/package/ng-mocks#mockrender) might be helpful.

Simply use it instead of `TestBed.createComponent`.

```typescript
const fixture = MockRender(TargetComponent, {
  input: '',
});
// The hook has been already called here.
```

Changes of parameters will trigger the hook.

```typescript
fixture.componentInstance.input = 'change';
fixture.detectChanges(); // <- triggers the hook again.
// Here we can do desired assertions.
```

---

A source file of this test is here:
[TestLifecycleHooks](https://github.com/ike18t/ng-mocks/blob/master/examples/TestLifecycleHooks/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestLifecycleHooks/test.spec.ts)
to play with.

```typescript
import {
  AfterContentChecked,
  AfterContentInit,
  AfterViewChecked,
  AfterViewInit,
  ChangeDetectionStrategy,
  Component,
  Injectable,
  Input,
  NgModule,
  OnChanges,
  OnDestroy,
  OnInit,
} from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { MockBuilder, MockRender, ngMocks } from 'ng-mocks';

// A dummy service we are going to replace with its mock copy and to use for assertions.
@Injectable()
class TargetService {
  protected called = false;

  public ctor() {
    this.called = true;
  }

  public onInit() {
    this.called = true;
  }

  public onDestroy() {
    this.called = true;
  }

  public onChanges() {
    this.called = true;
  }

  public afterViewInit() {
    this.called = true;
  }

  public afterViewChecked() {
    this.called = true;
  }

  public afterContentInit() {
    this.called = true;
  }

  public afterContentChecked() {
    this.called = true;
  }
}

@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  selector: 'target',
  template: ``,
})
class TargetComponent
  implements OnInit, OnDestroy, OnChanges, AfterViewInit, AfterViewChecked, AfterContentInit, AfterContentChecked {
  @Input() public input: string | null = null;

  protected readonly service: TargetService;

  constructor(service: TargetService) {
    this.service = service;
    this.service.ctor();
  }

  ngOnInit(): void {
    this.service.onInit();
  }

  ngOnDestroy(): void {
    this.service.onDestroy();
  }

  ngOnChanges(): void {
    this.service.onChanges();
  }

  ngAfterViewInit(): void {
    this.service.afterViewInit();
  }

  ngAfterViewChecked(): void {
    this.service.afterViewChecked();
  }

  ngAfterContentInit(): void {
    this.service.afterContentInit();
  }

  ngAfterContentChecked(): void {
    this.service.afterContentChecked();
  }
}

@NgModule({
  declarations: [TargetComponent],
  exports: [TargetComponent],
  providers: [TargetService],
})
class TargetModule {}

describe('TestLifecycleHooks', () => {
  ngMocks.faster();

  beforeEach(() => MockBuilder(TargetComponent, TargetModule));

  it('triggers lifecycle hooks correctly via MockRender', () => {
    // First let's suppress detectChanges.
    const fixture = MockRender(
      TargetComponent,
      {
        input: '',
      },
      { detectChanges: false }
    );

    const service: TargetService = TestBed.get(TargetService);

    // By default nothing should be initialized, but ctor.
    expect(service.ctor).toHaveBeenCalledTimes(1); // changed
    expect(service.onInit).toHaveBeenCalledTimes(0);
    expect(service.onDestroy).toHaveBeenCalledTimes(0);
    expect(service.onChanges).toHaveBeenCalledTimes(0);
    expect(service.afterViewInit).toHaveBeenCalledTimes(0);
    expect(service.afterViewChecked).toHaveBeenCalledTimes(0);
    expect(service.afterContentInit).toHaveBeenCalledTimes(0);
    expect(service.afterContentChecked).toHaveBeenCalledTimes(0);

    // Now let's render the component.
    fixture.detectChanges();

    // This calls everything except onDestroy and onChanges.
    expect(service.ctor).toHaveBeenCalledTimes(1);
    expect(service.onInit).toHaveBeenCalledTimes(1); // changed
    expect(service.onDestroy).toHaveBeenCalledTimes(0);
    expect(service.onChanges).toHaveBeenCalledTimes(1); // changed
    expect(service.afterViewInit).toHaveBeenCalledTimes(1); // changed
    expect(service.afterViewChecked).toHaveBeenCalledTimes(1); // changed
    expect(service.afterContentInit).toHaveBeenCalledTimes(1); // changed
    expect(service.afterContentChecked).toHaveBeenCalledTimes(1); // changed

    // Let's change it.
    fixture.componentInstance.input = 'change';
    fixture.detectChanges();

    // Only OnChange, AfterViewChecked, AfterContentChecked should be triggered.
    expect(service.ctor).toHaveBeenCalledTimes(1);
    expect(service.onInit).toHaveBeenCalledTimes(1);
    expect(service.onDestroy).toHaveBeenCalledTimes(0);
    expect(service.onChanges).toHaveBeenCalledTimes(2); // changed
    expect(service.afterViewInit).toHaveBeenCalledTimes(1);
    expect(service.afterViewChecked).toHaveBeenCalledTimes(2); // changed
    expect(service.afterContentInit).toHaveBeenCalledTimes(1);
    expect(service.afterContentChecked).toHaveBeenCalledTimes(2); // changed

    // Let's cause more changes.
    fixture.detectChanges();
    fixture.detectChanges();

    // Only AfterViewChecked, AfterContentChecked should be triggered.
    expect(service.ctor).toHaveBeenCalledTimes(1);
    expect(service.onInit).toHaveBeenCalledTimes(1);
    expect(service.onDestroy).toHaveBeenCalledTimes(0);
    expect(service.onChanges).toHaveBeenCalledTimes(2);
    expect(service.afterViewInit).toHaveBeenCalledTimes(1);
    expect(service.afterViewChecked).toHaveBeenCalledTimes(4); // changed
    expect(service.afterContentInit).toHaveBeenCalledTimes(1);
    expect(service.afterContentChecked).toHaveBeenCalledTimes(4); // changed

    // Let's destroy it.
    fixture.destroy();

    // This all calls except onDestroy and onChanges.
    expect(service.ctor).toHaveBeenCalledTimes(1);
    expect(service.onInit).toHaveBeenCalledTimes(1);
    expect(service.onDestroy).toHaveBeenCalledTimes(1); // changed
    expect(service.onChanges).toHaveBeenCalledTimes(2);
    expect(service.afterViewInit).toHaveBeenCalledTimes(1);
    expect(service.afterViewChecked).toHaveBeenCalledTimes(4);
    expect(service.afterContentInit).toHaveBeenCalledTimes(1);
    expect(service.afterContentChecked).toHaveBeenCalledTimes(4);
  });
});
```

[back to the homepage](./)
