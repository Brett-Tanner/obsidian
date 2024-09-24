---
title: Structure and interpretation of computer programs
description: Heavy CS degree text using Lisp, also has a JS version
---

# Lisp

This book covers the Scheme dialect of Lisp, so notes may be specific to that dialect.

Expressions are the building blocks of the language, kind of like a function in most other languages. They're wrapped by brackets.

Prefix notation is used to construct combinations, with the operator at the start followed by a space separated list of operands of any length which the operator will be applied to. Thus 5 \* 6 \* 7 in most other languages is (\* 5 6 7) in Lisp. Combinations can be nested to create more complex combinations, like (+(/ 4 2)(\* 4 9 3)).

In general when evaluating an expression, primitive expressions are evaluated as:

- For numerals, the numbers they name
- For built in operators, the machine code that carries out the corresponding operation
- For any other name, the object associated with that name within the environment

## Special forms

### Definition

One exception, or _special form_, is variable definition. (define x 200) does not apply `define` to the value of the symbol x and the number 200, but sets the value of the symbol x to 200. There are other such special forms in the language.

Procedure definitions are handled in much the same way, like (define (_name, parameters_) _body_). For example to find the square of a parameter: (define (square x) (\* x x)).

### Conditionals

Use the form

```lisp
(cond (<p1> <e1>)
      (<p2> <e2>)
      (<pn> <en>))
```

where p is the predicate which evaluates to true or false, and e is the _consequent expression_ which is evaluated if the predicate is satisfied. If no conditions are satisfied the return value of `cond` is undefined. `else` can also used in place of the final predicate in `cond`.

`if` can be used where you would use a ternary operator in other languages, when there is only a single predicate and two expressions to be evaluated. However the expressions provided to `if` may only be single expressions, in contrast to `cond` which can take sequences of expressions.

`&&`, `||` and `!` are `and`, `or` and `not` respectively. They're applied in the usual Lisp way, as the operator at the start the expression. The first two are special forms as not every expression is necessarily evaluated, while `not` is an ordinary procedure.

`#t` and `#f` are the constants associated with true and false in Scheme.

# Evaluation models

## Substitution

### Applicative order

Is used by Lisp, avoids repeated evaluations of the same expression. In this model the expression is evaluated by recursively evaluating the operator to get the procedure to be applied, and the operands to get its arguments. e.g. (+ (square 6) (square 10)) substitutes to (+ (_ 6 6) (_ 10 10)), which can be evaluated together as it contains only primitive expressions.

### Normal order

The expression is fully expanded until only primitive operators remain, then evacuated. e.g (square(+ 5 1)) would expand to (\* (+ 5 1) (+ 5 1)) and then be evaluated.
