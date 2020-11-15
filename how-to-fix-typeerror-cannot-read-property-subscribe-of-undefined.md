---
description: How to fix TypeError: Cannot read property 'subscribe' of undefined
---

# How to fix TypeError: Cannot read property 'subscribe' of undefined

This issue means that something has been replaced with a mock copy and returns a dummy result (`undefined`) instead of observable streams.

There is an answer for this error in the section called [How to create a mock observable](https://www.npmjs.com/package/ng-mocks#how-to-create-a-mock-observable),
if the error has been triggered by a mock service, and its property is of type of `undefined`.

Or you might check [`MockInstance`](https://www.npmjs.com/package/ng-mocks#mockinstance) in case if the error has been caused by a mock component or a mock directive.

[back to the homepage](./)
