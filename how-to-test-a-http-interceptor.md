---
description: How to test a http interceptor in Angular application
---

# How to test a http interceptor in Angular application

To test an interceptor we need the interceptor itself and its module.
Please consider refactoring if the interceptor is defined by `useValue` or `useFactory`, whereas `useClass` and `useExisting` are supported.
The problem of `useValue` and `useFactory` is that it is quite hard to distinguish them to avoid influence of other interceptors
in `TestBed`.

We need to keep `HTTP_INTERCEPTORS` token, because the interceptor is defined by it.
But this cause that all other interceptors will be kept too, therefore, we need to get rid of them via excluding `NG_MOCKS_INTERCEPTORS` token.
The issue here is that if there are more interceptors, then their mocked copies will fail
with "You provided 'undefined' where a stream was expected." error.
And the last important step is to replace `HttpClientModule` with `HttpClientTestingModule`,
so we can use `HttpTestingController` for faking requests.

```typescript
beforeEach(() =>
  MockBuilder(TargetInterceptor, TargetModule)
    .exclude(NG_MOCKS_INTERCEPTORS)
    .keep(HTTP_INTERCEPTORS)
    .replace(HttpClientModule, HttpClientTestingModule)
);
```

Let's pretend that `TargetInterceptor` adds `My-Custom: HttpInterceptor` header to every request.
To test it we need to send a request via `HttpClient`.

```typescript
const client = TestBed.get(HttpClient);

client.get('/target').subscribe();
```

The next step is to write an expectation of the request.

```typescript
const req = httpMock.expectOne('/target');

req.flush('');
httpMock.verify();
```

Now we can assert the headers of the request.

```typescript
expect(req.request.headers.get('My-Custom')).toEqual(
  'HttpInterceptor'
);
```

---

A source file of this test is here:
[TestHttpInterceptor](https://github.com/ike18t/ng-mocks/blob/master/examples/TestHttpInterceptor/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestHttpInterceptor/test.spec.ts)
to play with.

```typescript
import {
  HTTP_INTERCEPTORS,
  HttpClient,
  HttpClientModule,
  HttpEvent,
  HttpHandler,
  HttpInterceptor,
  HttpRequest,
} from '@angular/common/http';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { Injectable, NgModule } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { MockBuilder, NG_MOCKS_INTERCEPTORS } from 'ng-mocks';
import { Observable } from 'rxjs';

// An interceptor we want to test.
@Injectable()
class TargetInterceptor implements HttpInterceptor {
  protected value = 'HttpInterceptor';

  public intercept(request: HttpRequest<void>, next: HttpHandler): Observable<HttpEvent<void>> {
    return next.handle(
      request.clone({
        setHeaders: {
          'My-Custom': this.value,
        },
      })
    );
  }
}

// An interceptor we want to ignore.
@Injectable()
class MockInterceptor implements HttpInterceptor {
  protected value = 'Ignore';

  public intercept(request: HttpRequest<void>, next: HttpHandler): Observable<HttpEvent<void>> {
    return next.handle(
      request.clone({
        setHeaders: {
          'My-Mock': this.value,
        },
      })
    );
  }
}

// A module with its definition.
@NgModule({
  imports: [HttpClientModule],
  providers: [
    TargetInterceptor,
    MockInterceptor,
    {
      multi: true,
      provide: HTTP_INTERCEPTORS,
      useExisting: TargetInterceptor,
    },
    {
      multi: true,
      provide: HTTP_INTERCEPTORS,
      useClass: MockInterceptor,
    },
  ],
})
class TargetModule {}

describe('TestHttpInterceptor', () => {
  // Because we want to test the interceptor, we pass it as the first
  // parameter of MockBuilder. To correctly satisfy its dependencies
  // we need to pass its module as the second parameter. Also we
  // should to pass HTTP_INTERCEPTORS into `.mock` and replace
  // HttpClientModule with HttpClientTestingModule.
  beforeEach(() =>
    MockBuilder(TargetInterceptor, TargetModule)
      .exclude(NG_MOCKS_INTERCEPTORS)
      .keep(HTTP_INTERCEPTORS)
      .replace(HttpClientModule, HttpClientTestingModule)
  );

  it('triggers interceptor', () => {
    const client: HttpClient = TestBed.get(HttpClient);
    const httpMock: HttpTestingController = TestBed.get(HttpTestingController);

    // Let's do a simply request.
    client.get('/target').subscribe();

    // Now we can assert that a header has been added to the request.
    const req = httpMock.expectOne('/target');
    req.flush('');
    httpMock.verify();

    expect(req.request.headers.get('My-Custom')).toEqual('HttpInterceptor');
  });
});
```

[back to the homepage](./)
