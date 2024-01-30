# Standard Disintegrated Internal Reactive System (SDIRS)
## Summary
A RFC that defines the standard behavior of a new internal system responsible for the management of reactive nodes and the dispatch of updates.

## Motivation
It's highly important to have a strictly-defined system in place for the management of reactive updates, as that will decrease the possibility of encountering code bugs and unintended site-effects.

While the current system works, it *could* be vulernable to various edge cases due to the behavior and expections of various parts of the system remain undefinied.

To solve all of this, SDIRS is introduced.

## Design
The least interesting part of SDIR, is the function responsible for launching updates which should do the following in this order (the algorithm responsible for determining what dependents do update is not important to this RFC):
* Get a dependent named X
* Cancel all CTasks of X
* Call an `onUpdate` method of X's owner, if one exists *(a method must not exist if the constructing owner of X doesn't have the ability to self-calculate such as Values)*
* Repeat on the dependents of X.
* Disconnect all relations between dependents of X and X.

We can also conclude more points from the previous spec, which are:
* A CTask is a table that has a `cancel` method.
* X is a struct whose has an owner that has a `onUpdate` method.

Furthermore, we can setup an offical definition for X's type:
* X is a normal table, not an object, not a metatable.
* X's Owner is *something* that *can* but *not must* has a `onUpdate` method.

As such, the following Luau types can be formalized:
```lua
type CTask = {
    cancel: (self: CTask) -> (),
    [any]: any
}

type Owner = {
    onUpdate: ((self: Owner) -> ())?,
    [any]: any
}

type RNode = {
    owner: Owner,
    dependents: {[RNode]: boolean},
    cancelableTasks: { CTask },
}
```

Another important feature to implement is the ability to alert an RNode if it got a new dependent. To implement this, an optional `onDependentAdded` method must be added to to RNode, and an internal `add_dependent` function must be implemented.

As such, the type of RNode should be something like this:
```lua
type RNode = {
    owner: Owner,
    dependents: {[RNode]: boolean},
    cancelableTasks: { CTask },
    onDependentAdded: ((self: RNode, dependent: RNode) -> ())?
}
```

Finally, its important to note that due to dynamic dependencies behavior (the clearing of relations between dependencies and their dependents), this method could be called everytime a dependent updates, as such, the method should be optional and `add_dependent` should be able to determine if it exists or not.

## Drawbacks
There are no drawbacks.

## Alternatives
There are no alternatives as this RFC isn't about finding a new algorithm, it's mainly about standardizing the general infrastructure.