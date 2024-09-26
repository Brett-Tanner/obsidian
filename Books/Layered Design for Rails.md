---
title: Layered Design for Rails Applications
description: A book on designing Rails apps in a layered fashion
---

Overall thesis seems to be that MVC is too limited for any decently-sized app. Additional layers of abstraction like service objects are needed to avoid models becoming god objects.

## Chapter 1: Abstraction layers

[Attractor Rails](https://github.com/julianrubisch/attractor-rails) provides a web interface for assessing the quality of Rails projects in terms of churn and complexity

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

## Chapter 5: Adding Abstractions

Incidentally, `head(status)` in a controller returns just the HTTP status code provided, remember I was looking for that at some point.

And `JSON.parse` takes a `symbolize_names` boolean.

### Services

Service Objects encapsulate a specific business operation, and lie between the controller/model layers (invoked in controller to pass data to model). A good example of when to use one would be I'm deciding what to do when an invoice is updated on the event site; the service object could contain all the code for sending emails or not depending on the role of the user and the attributes included in the request body.

The [Interactor](https://github.com/collectiveidea/interactor) gem provides an example of how service objects can be set up, and something like [Dry Monads](https://github.com/dry-rb/dry-monads?tab=readme-ov-file) could provide a way to improve exception handling for them.

It's important to not just dump everything into a service, model logic should still stay on the model to avoid anemic models (losing the benefits of OOP) and services should be abstracted into a single service if commonalities emerge.

## Chapter 6: Data Layer Abstractions

### Query Objects

Used to encapsulate complex queries used in multiple places, but not generic enough to define on the model. Suggested to create a base `ApplicationQuery` class with a `resolve` class method that raises a `NotImplemented` error. The initializer should take an active record relation and `resolve` should return one, to enable chaining.

You should probably provide a default initialize value of `Model.all`, and add a `resolve(args)` class method that calls `new(args).resolve` so you can just call it like `QueryObject.resolve(args)`.

If you namespace queries under the model they return a relation of like `Posts::CommentedQuery` you can have your `ApplicationQuery` class infer the default relation passed to `new` using a regex on the class name and `safe_constantize`. It's possible to add the query objects back on as scopes by adding some extra boilerplate to `ApplicationQuery`, but only makes sense to me if used across multiple models, which requires even more boilerplate.

Scopes are preferred when they're atomic, i.e. focused on one thing. But query objects make it easier to encapsulate complex logic relying on multiple fields/associations/ordering, as they're only used in specific circumstances rather than generally available on the model.

Since they're just a sub-layer of the domain layer, it's fine to just put query objects in subfolders named after their model, or 'models/queries' if they're used by multiple. Also end their filenames with '\_query.rb'.

### Arel

Arel is what AR uses under the hood to build queries; it's an AST manager that compiles the tree you build with SQL operation and value nodes to valid SQL.

[`arel-helpers`](https://github.com/camertron/arel-helpers) reduces boilerplate when building queries with Arel.

### Repositories

An intermediate layer between domain models and persistence, allowing upper layers to operate on plain objects without worrying about persistence. They should never return AR relations, only ever plain Ruby collections like arrays or hashes. Similarly, individual objects returned by any `find` analogue should be mapped to some kind of plain object rather than returning an instance of the model, which would still have access to methods like `save` or `update`.

Encouraged to use specific methods like `Post#publish` or `Post#update_draft` rather than `Post.update` as it better represents what's happening, and the respository doesn't have to be reusable with every model in the application.

Similarly, separate methods are added to the repository for each query (e.g. `search(tags)` and `authored_by(user)`). This extracts query logic from the model in a similar way to the query objects introduced above, and respositories becoming god objects can be avoided by creating different repositories for different contexts with only the methods required by that context.

## Chapter 7: Handling User Input

### Form Objects

Used to handle a specific process represented by a form, e.g. sending emails if requested, checking Ts & Cs are accepted, without adding code to the underlying model.

Most useful when there are multiple contexts in which you might need a form for a given model, each requiring their own logic. They're especially useful on 'multi-model' forms, which may or may not need to create other models associated with the primary model depending on selections within the form. Model-less forms are also a good application of form objects, for example a feedback form which simply sends the feedback as an email to support.

You wanna implement at least `new` and `save` methods, with the validations/email logic etc. as private methods, so you can use the form object in your controller just like you would with a model. Including `ActvieModel::API` gives validation support, while `ActiveModel::Attributes` gives you the attributes API.

These inclusions also give you the ability to use the form object with `form_with` just like you would the underlying model, though you'll need to override `#model_name` to the name of the underlying model in order for route generation to work.

If relying on validations in the underlying model, you'll also want to make sure its errors merged into the form object errors so they can be displayed. Never add form object-specific validations to a model though, as that defeats the point of form objects.

### Filter Objects

Used to encapsulate complex filtering/search logic rather than sticking it in a controller/model scopes. Used like `ModelFilter.filter(Model.all, params)` in the controller. Distinct from query objects as they accept and process user input, which query objects should never interact with.

Suggests using [rubanok](https://github.com/palkan/rubanok) for boilerplate.

Side note that came up here is that scopes are skipped if a `nil` value is returned from their lambda/block, so something like `scope :filter_by_status(status), -> { where(status:) if %w[published draft] }` will apply the filter if the passed status is valid, and simply skip the filter without disrupting any method chain if the passed status is nil/invalid. THIS IS A HUGE IMPROVEMENT ON HOW I WAS HANDLING COMPLEX SEARCH QUERIES ON THE KIDSUP SITES.

## Chapter 8: Representation layer

### Presenters

Helpers aren't ideal because they're global by default, and near impossible to limit the scope of in an intuitive way. Presenters solve this by letting you create a presenter for each different context, and being initialized with a base model they can delegate calls to.

Closed presenters expose a subset of the model's interface (through delegation) + the view-specific methods, while open presenters (or decorators) dynamically add new behviours to a class without affecting any other instances of the class. Rails has a built in method to create open presenters, `SimpleDelegator`. [keynote](https://github.com/rf-/keynote) provides an example of closed presenters.

Be sure not to instantiate presenters too early (like in the controller) as there's a risk of them leaking back to lower layers of abstraction and being used in situations that expect a plain model.

### Serializers

Typically done by overriding `#as_json` on the model, but can also be done by creating a presenter as above but calling it a serializer and defining an `#as_json` method. [alba](https://github.com/okuramasafumi/alba) provides a DSL for doing so.

## Chapter 9: Authorization
