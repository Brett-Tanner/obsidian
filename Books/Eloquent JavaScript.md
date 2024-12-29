---
title: Eloquent JavaScript (4th Edition)
description: An introduction to JS, and a refresher for me before I start at Moneytree
---

## Chapter 1 - Values, Types & Operators

`null` & `undefined` only equal each other, so if you check for equality with `null` you know it's also not undefined.

`??` is like `||` but only returns the right hand value if the left hand value is exactly `null` or `undefined`. Can be preferable to `||` because, for example, `0 || 100` returns 100 because 0 is coerced to `false`, but `0 ?? 100` returns 0.

### Strings

If you want a literal newline or to interpolate you need to use backticks (``) but the escaped version (\n) works in anything. There are no differences between '' and "" other than needing to escape the same quotation marks.

When ordering strings, uppercase are always 'less' than lowercase because they're sorted by unicode char codes.

For strings, `.indexOf()` can take a multi-character string.

You can pass a function as the second arg to `.replace()`

### Numbers

`NaN` is the only value in JS not equal to itself.

## Chapter 2 - Program Structure

Exponentiation is `**`.

`do { body } while (condition)` will always execute at least once.

You can assign a variable inside an `if` statement, and the variable will be available in that statement.

## Chapter 3 - Functions

Function definitions can be nested within function definitions.

You can pass extra args to a function and it'll just ignore them without throwing or anything lol.

Apparently a loop is generally more performant than recursion.

## Chapter 4 - Data Structures

If you want to access an Object property in a similar way to `send` in Ruby, use `[]`. For example if you wanna dynamically access the property stored in some variable (`Object[var]`) or access a property which wouldn't work as a dot property (`Object["Wouldn't work"]`). Incidentally, this is how array index accesses work.

You can add Object properties with spaces etc. by quoting them like `let obj = { "foo bar": 1 }`.

`"property"" in object` will return true if the property exists on the object (even if undefined), false otherwise.

`Object.assign(target, source)` copies all properties from `source` into `target`.

Syntax for implied values in an object is `{ var1, var2 }`, as opposed to `{ var1:, var2: }` in Ruby.

`for (let x of array/string)` works like a for loop on arrays, you can use it on Objects with `Object.keys(obj)` as the arg following `of`.

`(...args) => {}` is the syntax for rest params. You can also use it to spread an array or object into another one, in the case of an object properties defined twice will keep only the last one.

You can use destructuring in the args to a function, so if you're passing a 3 element array `([a, b, c]) => {}` will bund local variables `a`, `b` and `c`.

Optional property access (`?.`) can be used with functions (`func?.()`) and square bracket property access (`obj?.[key]`)

## Chapter 5 - Higher Order Functions

Mentions that a loop can be faster than chaining higher order stuff like `map`, `filter`, etc. as you're not instantiating a new array each time, but usually readability is more important.

## Chapter 6 - OOP

When defining functions with `function`, `this` in a method refers to the object it's called on, even if the method was defined outside the object and later added to it. On the other hand, arrow functions have access to the `this` value from where they're defined.

Alternately you can call the function like `func.call(this, ...args)` to provide an explicit `this`.

Shorthand for defining a function on an object is `let obj = { funcName(args) { body } }`.

Getters and setters have 'get/set' before their names like `const obj = { get prop() { return this.prop.toString() } }`. Especially useful if you want 'virtual' properies derived from an actual property, like a `Temperature` object which internally stores only celsius but has a getter/setter for fahrenheit, which is derived from/adjusts the celsius value appropriately.

### Classes

Adding a `#` at the start of a method name makes it private. You can also add private properties in the same way, however they must be created in the constructor, it's not possible to assign them after creation.

Methods or properties with `static` before their name are stored on the constructor, allowing them to be used to create new instances of the class. For example, in the `Temperature` example mentioned above, the regular constructor likely takes a temperature in celsius. However you could define `static fromFahrenheit(fTemp) { return new Temperature(f to c conversion) }` on the class to enable instances to be initialized from a fahrenheit temperature.

The inheritance syntax is `class SubClass extends SuperClass { ... }`. When `super` is used in the constructor (must be before any `this` calls), it calls the `SuperClass` constructor. When used outside the constructor it enables accessing properties on the parent class, like `super.parentClassMethod()` or `super.parentProperty`.

### Prototypes

You can use `Object.create(prototype)` to create an object which 'inherits' the properties from `prototype`.

Using a plain object as a map/hash has issues, for example it inherits from `Object.prototype` so has `.toString()` as a 'key', and keys must be strings. You can get around the inherited methods using `const map = Object.create(null)`, and for the second issue there's a `Map` class which allows any type of keys.

Also useful to note `Object.keys()` only returns the objects own keys, not those inherited from the prototype, and you can use `Object.hasOwn(obj, property)` as an alternative to `in` if you want to exclude inheriteed properties.

Remember `instanceof` looks up the prototype chain, not just at the object itself.

### Symbols

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

### Iterators

To make an object iterable you need to define a method named with `Symbol.iterator`, which is a symbol value defined by the language for this purpose. In addition to letting you use `for ... of` with the object, it also enables the spread operator (`...`) to spread the object's values into an array.

When called, that method should return an object which implements the iterator interface (`next()`; which returns the next result, `value`; the next value if any and `done`; which should be true if there are no more results).

Another way to write iterators is like `function* generatorName() { iteration/loop logic }`, and calling `yield` to pass the loop value out. The benefit of this is not needing to manually create an object to save the local state (value/done); generators do it automatically.

## Chapter 8 - Bugs & Errors

Exceptions unwind the stack until they hit a `try` block which can catch them. They're created like `throw new Error()`. You can also include code you want to ensure is run regardless of whether or not an exception is thrown in a `finally` block, after `try` and in addition to/instead of `catch`.

You can't selectively catch certain types of exception in JS, an interesting design choice. So you need to manually check for it in the `catch` using the `e` param, book suggests creating an `Error` subclass if one doesn't exist for your situation like `class TypoError extends Error` and checking if `e` is an instance of that subclass.

The issue with exceptions is they can cause the program to completely abandon its work, leaving any side effects in place. Since exceptions can be thrown anywhere and handled somewhere completely different, it can be difficult/complex to correctly clean up these side effects.

### Strict Mode

Without strict mode, `for (i = 0; ...)` won't complain about the missing `let`, and will instead assign `i` as a global variable.

In strict mode, `this` is undefined for functions not called as methods. Without strict, it refers to the global scope.

## Chapter 9 - Regex

Can be created with `/regex/` or `new RegExp("regex")`.

`\w` won't match international word characters like kanji, as the initial simplistic implementation didn't consider them. However `\s` does match international whitespace characters for some reason.

Important to remember it will seek forward until it doesn't match, then backtrack one by one trying for a match. This can have serious performance implications, so I guess try not to start the regex with anything overly broad that might cause a lot of backtracking.

'Greedy' operators like `*` and `+` match as much as possible, they can be made non-greedy (only matching more when the remaining pattern does not fit the smaller match) by appending `?`.

### Methods

- `.exec(string)` returns null if no match is found and an object containing the match (or an array of matched groups) otherwise
  - The returned object has an `index` property which is the index of the match in the string
  - And `source` which is the original string
  - If subexpressions (`()`) are used, the elements of the object after the first will be all matched groups
    - Unmatched subexpressions will be represented by an `undefined` in the resulting object
    - If there are multiple matches, only the last ends up in the object
  - strings have a similar method called `.match(regex)`
- `.test(string)` returns true if the string contains a match
- `"string".search(regex)` returns the index of the first match, or -1 if no match (like `indexOf`)

### Syntax

- `-` between two characters creates a range between them (using unicode char codes)
- `\d` matches any digit, `\D` matches any non-digit
- `\s` matches any whitespace (tab, newline, space etc.), `\S` matches any non-whitespace
- `\w` matches any alphanumeric character, `\W` matches any non-alphanumeric character
- `\p{property}` matches any character with the specified property (though you need to put `u` at the end of the regex), `\P` matches any character without the specified property
- `.` matches any character except newline
- `+` matches one or more of the preceding character, `*` matches zero or more
- `?` matches zero or one, making that part of the pattern optional
- `^` matches the beginning of the string, `$` matches the end
- `|` matches either of the two patterns
- `(?=char)` is a lookahead, which requires `char` for a match but does not include it in that match
- `{num}` requires the char to appear exactly `num` times, and can also be given a range like `{start, end}`. It's possible to specify an endless range.
- Enclosing a section of the regex in `()` makes it count as a single element to the operators following it.
  - It also creates a subexpression, if you don't want that add a `?` after the opening bracket
  - Subexpressions can be referenced when calling `.replace()` on a string, like `"string".replace(/(group1) (group2)/, "$2, $1")`
  - `$&` refers to the whole match
- `[]` matches on the presence of any single character in the brackets (so `/[\d.]/` matches any digit or period), and removes the special meaning of symbols like `.` or `+`

### Options

- `//i` makes the regex case insensitive
- `//u` makes the regex unicode aware
  - You'll need this if matching emoji or anything else made up of multiple code units like Han script
- `//g` makes the regex global (find all matches)
  - Unless you're trying to replace items in a string, probably prefer using the `.matchAll()` method
  - It returns an array of match objects, whereas `.match()` returns an array of strings when used with a global regex

### Dates

Called Date but actually represents a point in time if instantiated with `new Date()`. Month numbers start at 0, but days at 1. Because Javascript.

`.getTime()` on a Date object gives you the Unix timestamp.

## Chapter 10 - Modules

Remember you can change the name of an imported binding like `import { foo as bar } from 'module'`.

If you're importing a default export, you can drop the `{}`. `*` can be used to import everything from a module (you must provide a name to access them on).

## Chapter 11 - Async

Create a promise with `cont p = Promise.resolve(value)`, and use `.then((promiseValue) => {})` to act on it once resolved. `.then()` returns a promise, so you can chain calls.

If any promise in a chain is rejected, the whole chain is marked as rejected and no further success callbacks are made.

`.catch(handlerFunc(err))` resolves to the normal value if no errors occur, or to the result of the handler function if a promise in the chain is rejected. As an alternative you can pass a rejection handler as the 2nd value to `.then()`.

Methods can also use the `async` keyword.

Without promises, if you use something like `.setTimeout()` in a `try` block, the catch won't do anything in the event of failure as it won't be on the stack by the time `.setTimeout()` throws.

Async functions/`.setTimeout()` can be delayed by a long running process, even if they've already been resolved.

When running multiple async functions at once, better to return a value from each then act on them together rather than trying to have them all mutate shared state.

`Promise.all()` takes an array of promises and returns a promise that resolves to an array of the results of the promises in the order they were resolved.

## Chapter 12 - BYO Language

A parser reads some text and outputs a data structure (AST) reflecting the structure of the program expressed by that text. Nodes in the AST have a set of properties based on their type, for example the node for a function might have its args and the body of the function as properties.

The AST is the run by an evaluator, which you pass the syntax tree and a scope object associating names with values. For each node the evaluator returns a value, e.g. a 'value' node would simply return itself, a 'variable' node would return its binding (if in scope) and a function would be evaluated, with its return value being returned.

Language constructs like `if` or `&&` (at least for this demo language) are just functions stored in a 'Special Forms' object associating the name used to call them with the function body. They may be represented like this because they have special requirements, for example `&&` should only evaluate the right hand side if the left hand side is `true`, but a function passing the two side as args would evaluate both of them when called.

The 'scope' passed to the evaluator maps binding names to their values (so a hash).

Compilers add another step between parsing and evaluation, with the aim of doing as much work as possible beforehand and optimising the code for faster/more efficient execution. Often compiled to machine code, but in theory the compilation target can be anything you want.

## Chapter 13-15

Basic browser stuff I knew already

## Chapter 16-17

Animations, canvas stuff in the context of a simple browser game

## Chapter 18 - HTTP & Forms
