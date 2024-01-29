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

We can also conclude more points from the previous spec, which are:
* A CTask is a table that has a `cancel` method.
* X is a struct whose has an owner that has a `onUpdate` method.

Furthermore, we can setup an offical definition for X's type:
* X is a normal table, not an object, not a metatable.
* X's Owner is *something* that *can* but *not must* has a `onUpdate` method.

As such, the following Luau types can be formalized:
```lua
type CTask = {
    cancel: (self: CTask) -> ()
}

type Owner = {
    onUpdate: ((self: Owner) -> ())?
}

type RNode = {
    owner: Owner,
	dependents: {[RNode]: boolean},
	cancelableTasks: { CTask },
}
```

## Drawbacks
There are no drawbacks.

## Alternatives
There are no alternatives as this RFC isn't about finding a new algorithm, it's mainly about standardizing the general infrastructure.