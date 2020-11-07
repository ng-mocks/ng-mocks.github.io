---
description: How to test a http request in Angular application
---

# How to test a http request in Angular application

Testing a http request means that we want to cover a service or a declaration sending it. For that we need to keep the
thing as it is and to mock all its dependencies. The last important step is to replace `HttpClientModule`
with `HttpClientTestingModule` so we can use `HttpTestingController` for faking requests.

```typescript
beforeEach(() =>
  MockBuilder(TargetService, TargetModule).replace(
    HttpClientModule,
    HttpClientTestingModule
  )
);
```

Let's pretend that `TargetService` sends a simple GET request to `/data` and returns its result. To test it we need to
subscribe to the service and to write an expectation of the request.

```typescript
const service = TestBed.get(TargetService);
let actual: any;

service.fetch().subscribe(value => (actual = value));
```

```typescript
const httpMock = TestBed.get(HttpTestingController);

const req = httpMock.expectOne('/data');
expect(req.request.method).toEqual('GET');
req.flush([false, true, false]);
httpMock.verify();
```

Now we can assert the result the service returns.

```typescript
expect(actual).toEqual([false, true, false]);
```

---

A source file of this test is here:
[TestHttpRequest](https://github.com/ike18t/ng-mocks/blob/master/examples/TestHttpRequest/test.spec.ts).<br>
Prefix it with `fdescribe` or `fit` on
[codesandbox.io](https://codesandbox.io/s/github/satanTime/ng-mocks-cs?file=/src/examples/TestHttpRequest/test.spec.ts)
to play with.

```typescript
import { HttpClient, HttpClientModule } from '@angular/common/http';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { Injectable, NgModule } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { MockBuilder } from 'ng-mocks';
import { Observable } from 'rxjs';

// A service that does http requests.
@Injectable()
class TargetService {
  protected http: HttpClient;

  constructor(http: HttpClient) {
    this.http = http;
  }

  fetch(): Observable<boolean[]> {
    return this.http.get<boolean[]>('/data');
  }
}

// A module providing the service and http client.
@NgModule({
  imports: [HttpClientModule],
  providers: [TargetService],
})
class TargetModule {}

describe('TestHttpRequest', () => {
  // Because we want to test the service, we pass it as the first
  // parameter of MockBuilder. To correctly satisfy its
  // initialization, we need to pass its module as the second
  // parameter. And, the last but not the least, we need to replace
  // HttpClientModule with HttpClientTestingModule.
  beforeEach(() => MockBuilder(TargetService, TargetModule).replace(HttpClientModule, HttpClientTestingModule));

  it('sends a request', () => {
    // Let's extract the service and http controller for testing.
    const service: TargetService = TestBed.get(TargetService);
    const httpMock: HttpTestingController = TestBed.get(HttpTestingController);

    // A simple subscription to check what the service returns.
    let actual: any;
    service.fetch().subscribe(value => (actual = value));

    // Simulating a request.
    const req = httpMock.expectOne('/data');
    expect(req.request.method).toEqual('GET');
    req.flush([false, true, false]);
    httpMock.verify();

    // Asserting the result.
    expect(actual).toEqual([false, true, false]);
  });
});
```

[back to the homepage](./)
