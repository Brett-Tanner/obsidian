---
title: Layered Design for Rails Applications
description: A book on designing Rails apps in a layered fashion
---

## Chapter 4: Antipatterns

Callbacks, concerns and global state can all lead to code that violates separation of concerns. This chapter looks at how to use them while minimising this drawback.

### Callbacks

Posits that controller callbacks are generally OK to extract logic that isn't the 'main point' of the action and it doesn't introduce a hidden dependency.

The appropriateness of AR callbacks depends on how closely coupled they are to the model.

For example logic updating/normalizing/ensuring a value for required fields is fine to put in a callback (though normalizing can be done with the [`normalizes` API](https://api.rubyonrails.org/classes/ActiveRecord/Normalization.html) as of 7.1). Counter caches are another example of acceptable callbacks as they improve performance and add minimally to the model's responsibilities

Undesirable callbacks usually have conditionals, or interact with objects in a very different domain/layer to the user like mailers or a CRM system.

If an undesirable callback needs to happen in response to an event, consider emitting that event in the callback instead, then having the relevant subscriber act on it when the event is received. That way the model doesn't need knowledge of the subscriber/s. You can do a basic version of this with `ActiveSupport::Notifications` and `ActiveSupport::Subscriber`, but gems like [downstream](https://github.com/palkan/downstream) or [active_event_store](https://github.com/palkan/active_event_store) can give you some abstractions to work with.

If there are a group of related callbacks which represent a standalone process, they should be extracted to a concern or similar.

### Concerns

Important to create concerns which encapsulate shared behaviour rather than things of the same category like associaions/validations. Should namespace and store in subfolder of models named after the model if only used by a single model.

Drawbacks of concerns are:

- Private methods are not private to other concerns on same model
- Naming collisions are difficult to foresee since in separate files
- Testing is more difficult, ideally want to test in isolation but what if model it's included in has methods etc. that interfere?
- Callbacks in concerns exacerbate the issues with callbacks, as they're now spread around the codebase.

Basically concerns should be used to encapsulate groups of code related to support functionality; if removing the concern from a model causes its tests to fail it's an extracted chunk of code, not a concern.

If there's a separate concept which needs to be shared between multiple models, it should be extracted to its own model rather than a concern.

When you have complex validations surrounding some kind of value, consider extracting it as a 'value object' using [`Data.define`](https://docs.ruby-lang.org/en/3.3/Data.html).

## Useful Stuff

[Attractor Rails](https://github.com/julianrubisch/attractor-rails) provides a web interface for assessing the quality of Rails projects in terms of churn and complexity
