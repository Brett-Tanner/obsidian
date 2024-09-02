Martin suggests to ignore performance when refactoring; if splitting an iteration into two separate iterations or querying the and data multiple times rather than assigning it to a variable makes the code more readable then go ahead and do that. 

Once you're done refactoring you'll find it easier to fix any performance regressions since the code is more modular and readable, if there even are any noticeable performance issues. 

*Probably a good thing to keep in mind when I do the invoice refactoring, especially since I know Rails caches queries*

## Refactoring Techniques

**Split Phase**

When you have combined logic for two separate things, like calculation of an invoice then generation of its text, split them by using the calculation to generate an intermediate data structure which is then passed to the next step. 

*For the invoice refactor, this would mean calculating the various costs/counts as a hash, then using passing that hash to a method which generates the HTML summary and a background job to generate the pdf and email it.* 