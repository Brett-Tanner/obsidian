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
