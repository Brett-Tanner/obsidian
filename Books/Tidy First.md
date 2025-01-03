# Introduction 

Offers an answer to the 'when' of software design; when adding features to a component becomes difficult, redesign it to relieve that pressure.
# Tidyings

*Always separate tidying/refactoring commits from those which change behaviour*

## Dead Code

Delete it. Maybe put some logs in there first and monitor to see if it's actually dead, but keep in mind it's always in version control if you **really** need to bring it back. 

## Normalise Symmetries

If the same thing is done multiple ways in different places, pick one way and convert them all to that to reduce cognitive load. 

## New Interface, Old Implementation

If doing something is difficult, create the interface you wish you had to do it and have it abstract away calling the old implementation. 

## Explanatory Variables

If you have long expressions with unclear purposes, assign them to variables which clarify them. 

## Explicit Parameters 

Pass parameters explicitly rather than getting them from global state.

## One Pile

Though the book has a bias towards small, decoupled chunks of code, it recognises this can sometimes go wrong. Indicators for this are:
- Long, repeated argument lists
- repeated code, especially conditionals
- poorly named helpers
- shared mutable data structures

In the presence of these it's reasonable to throw them all together in a pile to get an overview, then tidy them out again from there in a more logical way. 

## Explanatory Comments

If you read code and have an 'aha!' moment, record it. Also anything which is obvious to you but might not be to others with different backgrounds. 


# Managing

## Separate Tidying

When you switch between behaviour and tidying changes, open a new PR. 

## Batch Sizes

Labs towards lots of small tidying PRs, less so if you aren't an ideal team that can skip reviews for tidying PRs but still leaning that way. 

## Getting Untangled

Suggests starting over and tidying first when you have mixed behavioural and tidying changes, might see something you missed and makes it easier for reviewers.

Also try to cultivate a sense of when you're creating a tangle, much easier to untangle the earlier you notice. 

## Tidy Timing

Valid to have a list you work through in slack time, prioritising tidyings that make changes to often changed code easier. 

Tidy after adding behaviour if likely to change again soonish, you have ideas on how to tidy now and the time you'll spend tidying is roughly proportionate to the time you spent changing behaviour. 

Tidy first if it helps with what you're doing now/understanding

# Theory

## Coupling 

Important to distinguish what things are coupled with respect to. For example a function and call suite are coupled with respect to name, but not with respect to the function's internal implementation. 

Want to avoid 1-N & cascading couplings, 1-1 is fine. 