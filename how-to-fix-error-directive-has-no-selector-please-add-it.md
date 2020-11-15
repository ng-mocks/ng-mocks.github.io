---
description: How to fix Error: Directive has no selector, please add it!
---

# How to fix Error: Directive has no selector, please add it!

This issue means that a module imports a declaration (usually a parent class) which does not have a selector.
Such directives and components are created during a [migration](https://angular.io/guide/migration-undecorated-classes)
if their parent classes haven't been decorated yet.

The right fix is to remove these declarations from modules, only final classes should be specified in there.

If you cannot remove them for a reason, for example it is a 3rd-party library,
then you need to write tests with usage of [`MockBuilder`](https://www.npmjs.com/package/ng-mocks#mockbuilder) and its [`.exclude`](https://www.npmjs.com/package/ng-mocks#mockbuilderexclude) feature.

```typescript
beforeEach(() =>
  MockBuilder(MyComponent, MyModule)
    .exclude(ParentDirective)
);
```

That fixes declarations of the module and resolves the error,
a directive without a selector has been gone from the module definition.   

[back to the homepage](./)
