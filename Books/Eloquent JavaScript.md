---
title: Eloquent JavaScript (4th Edition)
description: An introduction to JS, and a refresher for me before I start at Moneytree
---

# Chapter 1 - Values, Types & Operators

`null` & `undefined` only equal each other, so if you check for equality with `null` you know it's also not undefined.

`??` is like `||` but only returns the right hand value if the left hand value is exactly `null` or `undefined`. Can be preferable to `||` because, for example, `0 || 100` returns 100 because 0 is coerced to `false`, but `0 ?? 100` returns 0.

## Strings

If you want a literal newline or to interpolate you need to use backticks (``) but the escaped version (\n) works in anything. There are no differences between '' and "" other than needing to escape the same quotation marks.

When ordering strings, uppercase are always 'less' than lowercase because they're sorted by unicode char codes.

For strings, `.indexOf()` can take a multi-character string.

## Numbers

`NaN` is the only value in JS not equal to itself.

# Chapter 2 - Program Structure

Exponentiation is `**`.

`do { body } while (condition)` will always execute at least once.

# Chapter 3 - Functions

Function definitions can be nested within function definitions.

You can pass extra args to a function and it'll just ignore them without throwing or anything lol.

Apparently a loop is generally more performant than recursion.

# Chapter 4 - Data Structures

If you want to access an Object property in a similar way to `send` in Ruby, use `[]`. For example if you wanna dynamically access the property stored in some variable (`Object[var]`) or access a property which wouldn't work as a dot property (`Object["Wouldn't work"]`). Incidentally, this is how array index accesses work.

You can add Object properties with spaces etc. by quoting them like `let obj = { "foo bar": 1 }`.

`"property"" in object` will return true if the property exists on the object (even if undefined), false otherwise.

`Object.assign(target, source)` copies all properties from `source` into `target`.

Syntax for implied values in an object is `{ var1, var2 }`, as opposed to `{ var1:, var2: }` in Ruby.

`for (let x of array/string)` works like a for loop on arrays, you can use it on Objects with `Object.keys(obj)` as the arg following `of`.

`(...args) => {}` is the syntax for rest params. You can also use it to spread an array or object into another one, in the case of an object properties defined twice will keep only the last one.

You can use destructuring in the args to a function, so if you're passing a 3 element array `([a, b, c]) => {}` will bund local variables `a`, `b` and `c`.

Optional property access (`?.`) can be used with functions (`func?.()`) and square bracket property access (`obj?.[key]`)

# Chapter 5 - Higher Order Functions

Mentions that a loop can be faster than chaining higher order stuff like `map`, `filter`, etc. as you're not instantiating a new array each time, but usually readability is more important.

# Chapter 6 - OOP

When defining functions with `function`, `this` in a method refers to the object it's called on, even if the method was defined outside the object and later added to it. On the other hand, arrow functions have access to the `this` value from where they're defined.

Alternately you can call the function like `func.call(this, ...args)` to provide an explicit `this`.

Shorthand for defining a function on an object is `let obj = { funcName(args) { body } }`.

Getters and setters have 'get/set' before their names like `const obj = { get prop() { return this.prop.toString() } }`. Especially useful if you want 'virtual' properies derived from an actual property, like a `Temperature` object which internally stores only celsius but has a getter/setter for fahrenheit, which is derived from/adjusts the celsius value appropriately.

## Classes

Adding a `#` at the start of a method name makes it private. You can also add private properties in the same way, however they must be created in the constructor, it's not possible to assign them after creation.

Methods or properties with `static` before their name are stored on the constructor, allowing them to be used to create new instances of the class. For example, in the `Temperature` example mentioned above, the regular constructor likely takes a temperature in celsius. However you could define `static fromFahrenheit(fTemp) { return new Temperature(f to c conversion) }` on the class to enable instances to be initialized from a fahrenheit temperature.

The inheritance syntax is `class SubClass extends SuperClass { ... }`. When `super` is used in the constructor (must be before any `this` calls), it calls the `SuperClass` constructor. When used outside the constructor it enables accessing properties on the parent class, like `super.parentClassMethod()` or `super.parentProperty`.

## Prototypes

You can use `Object.create(prototype)` to create an object which 'inherits' the properties from `prototype`.

Using a plain object as a map/hash has issues, for example it inherits from `Object.prototype` so has `.toString()` as a 'key', and keys must be strings. You can get around the inherited methods using `const map = Object.create(null)`, and for the second issue there's a `Map` class which allows any type of keys.

Also useful to note `Object.keys()` only returns the objects own keys, not those inherited from the prototype, and you can use `Object.hasOwn(obj, property)` as an alternative to `in` if you want to exclude inheriteed properties.

Remember `instanceof` looks up the prototype chain, not just at the object itself.

## Symbols

Unique values defined like `const foo = Symbol('foo')`.

The string you pass is just what you get when converting the symbol to a string, used for identification. Two symbols created by passing the same string are not equal, and the symbol is not the string. `Symbol('a') != Symbol('a')`, even though `Symbol('a').toString() === Symbol('a').toString()`.

They allow you to define property names which might otherwise conflict with another property, for example `length` on Array.

```js
const length = Symbol("length");
Array.prototype[length] = 0;

console.log([1, 2].length);
// 2
console.log([1, 2][length]);
// 0
```

You can use a symbol as a property name in an object declaration by wrapping it in `[]` like `const obj = { [length]: 0 }`.

## Iterators

To make an object iterable you need to define a method named with `Symbol.iterator`, which is a symbol value defined by the language for this purpose. In addition to letting you use `for ... of` with the object, it also enables the spread operator (`...`) to spread the object's values into an array.

When called, that method should return an object which implements the iterator interface (`next()`; which returns the next result, `value`; the next value if any and `done`; which should be true if there are no more results).
