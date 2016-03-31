Scoped Constructor Arguments for ECMAScript Classes
---------------------------------------------------

The proposed [Class Instance Fields](https://github.com/jeffmo/es-class-fields-and-static-properties) leaves one question open: How do you create constant fields with access to constructor arguments.

This is an idea for a syntactical feature that can address this.

## Example Syntax

The syntax builds upon the current class instance fields proposal and existing class methods.

It adds an argument list to the class itself which provides a new scope around the class. The scope semantics of the class body itself doesn't change. I.e. methods and fields are not accessible without the `this.` prefix.

### Simple Arguments

```js
class Point2D (inX, inY) {
  x = inX;
  y = inY;
}
```

Desugars to:

```js
class Point2D {
  constructor(inX, inY) {
    this.x = inX;
    this.y = inY;
  }
}
```

### Captured Arguments

When the constructor arguments are referred to within methods or field initializers, the method itself is __not__ bound. It is still on the prototype. The method refers to a private slot on whatever `this` happens to be when the method is invoked.

```js
class Point2D (x, y) {
  measure() {
    return Math.sqrt(x * x + y * y);
  }
}
```

Pseudo-desugaring to [private fields](https://zenparsing.github.io/es-private-fields/):

```js
class Point2D {
  #x;
  #y;
  constructor(x, y) {
    this.#x = x;
    this.#y = y;
  }
  measure() {
    return Math.sqrt(this.#x * this.#x + this.#y * this.#y);
  }
}
```
### Conflicting Constructors

At least conflicting constructor arguments is a syntax error. Having both forms in the same class might also need to be a syntax error.

```js
class Point2D (x, y) {
  constructor(x, y, z) { // syntax error?

  }
}
```

### Static Methods

Static methods can be interleaved within this scope but they don't have access to the constructor argument.

```js
class Point2D (x, y) {
  static compare(a, b) {
    return x; // syntax error?
  }
}
```

## Status of this Proposal

This might be proposed to ECMAScript in the future but is not yet a concrete
proposal. It at least shows a plausible route forward for solving this problem after field initializers.
