# Standard Disintegrated Internal Reactive System (SDIRS)
## Summary
A RFC that defines the standard behavior of a new internal system responsible for the management of reactive nodes and the dispatch of updates.

## Motivation
It's highly important to have a strictly-defined system in place for the management of reactive updates, as that will decrease the possibility of encountering code bugs and unintended site-effects.

While the current system works, it *could* be vulernable to various edge cases due to the behavior and expections of various parts of the system remain undefinied.

To solve all of this, SDIRS is introduced.

## Design
This RFC's Standard Disintegrated Internal Reactive System *(SDIRS)* consists of various components that have a peer-to-peer relation or a dependency-to-dependent relation, each one documented separately.

### Object Types
This is the following spec for Luau types.
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
    onDependentAdded: ((self: RNode, dependent: RNode) -> ())?
}
```
* CTask: A *table* named X *(not required to meet any Specs or Paradigms such as OOP)* that must contain a `cancel` function that accepts X as its first argument, and is allowed to contain other data.
* Owner: A *table* named O that *could* have a `onUpdate` function that takes O as its first argument, and is free to contain other data. 
* RNode: A *struct* named R that must have an owner (Owner), dependents (a map of RNodes), cancelableTasks (an array of CTask), and optionally an `onDependentAdded` method. Unlike both CTask and Owner, RNode is completely Concrete and isn't allowed to house other undetermined data.

### Update Process
This is the following spec for the update process *(it *MUST* be noted that this is a spec for the goals that are needed to be met, rather than concrete steps each implementation of SDIRS need to meet)*

The following conditions must be met in *order*:
* Get a RNode named X
* Loop through the items of X's cancelableTasks and `cancel` them.
* The map `dependents` field of X holds moves to a temporary variable named `prevDependents` and the field itself holds a new empty map.
* Get X's Owner and call its onUpdate method, if one exists. *(An owner must not have an onUpdate method if it doesn't concern itself about the update process, like Values. For example, the dependents of a Value will be the ones concerning themseleves with keeping the connection with the Value, not the other way around)*
* Repeat on the `prevDependents`

Those requirements ensure that:
* The dependents of a RNode are always dynamic. 
* Previous CTasks are cleaned up before new possible ones are spawned.

It should also be important to note that the Update Process (UP) doesn't concern itself with the reconnecting between dependencies and their dependents, as this responsibility is usually hold by Owners. For example, an Owner of a RNode that is responsible for watching changes should reconnect with the watched node during the Owner's `onUpdate`. *(fun fact: the act of watching a "state" in Vinum simply translates to watchedState's RNode being the only dependency of the watcher's Rnode)*


### Dependent Injection
Additionally, SDIRS isn't concerned with exacty how you add dependents to dependencies or how many layers of complex behavior. The only spec SDIRS is concerned with is that when adding a new dependent, one should add the dependent ***and*** call the `onDependentAdded` of the dependency. 


## Drawbacks
There are no drawbacks.

## Alternatives
There are no alternatives as this RFC isn't about finding a new algorithm, it's mainly about standardizing the general infrastructure.