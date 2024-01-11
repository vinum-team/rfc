# Stream State Object
## Summary
The introduction of a `Stream` state object that adopts Observable-like semantics.

## Motivation
To find a generalized solution for all streaming-like needs, and replace objects like `Wrap` and `wrapSource`/`sourceObject` in older version.

A Stream will be able to generically solve all the problems that these objects aim to solve, but with clearer syntax, more freedom, and generally a better experience.
## Design
This new object will accept a `scope` argument like all other state objects, and a `producer` function that sends data through the "stream".

When a dependent *depends*/*subscribes* to a Stream object, we create a `Value` that represents a channel, which we will call `Channel_1` in our case and connect our dependent to `Channel_1` and disconnect the connection between it and the Stream object.

After that, we spawn an instance of our `producer` function with a `send` closure that only writes to the channel that is connected to the dependent (which is `Channel_1` in this situation).

When a `send` fire occurs, a `::Write` call is done to `Channel_1`, which will update the channel value with the one provided with the send call, and notifiy all of the channel dependents (in this case, its only one dependent) of the change.

So for example, when our `::On` operator is invoked, the `numberStream` object will create a new Channel object and an instance of our producer function. It will map the channel object to our dependent (which in this example, is an internal one), and run the producer with a `send` closure.

This closure when called, will update the channel object that it corelates to, and the dependent of that channel (which will make sense to call the subscriber if you want).
```lua
local numberStream = Stream(scope, function(send)
    for i = 1, 10 do
        send(i)
    end
end)

Ops:On(numberStream, function()
    print("got a number!")
end)
```

However, there is a problem in this; ***you can't read the data***. While we could either mutate the `::Read` operator semnatics, or introduce a new function specifically for this, the core problem still persists, which *we don't know which channel to read from*.

The current solution is to introduce global recording of the graph tree's node that is currently being updated, and make whatever function for reading to read it using that information (`stateObject.value.value[Context.current_node]`).  

However, another issue, which relates to the issue of general updating, is that if the subscriber is an object like *Computes*, which are objects that would likely cause a stack overflow. 

Since Streams are initialized when a new dependent is added, and Computes re-add themselves to their dependencies (to ensure that dependencies are dynamic), they could re-run whatever system we have in place that detects dependent additions, which could cause many issues like stack overflow in practice, while in theory, can introduce *cyclic dependencies* easily.

Of course, the solution to this is to make the new-dependent detection system smarter, and allow it to not reinvoke the stream everytime the Compute re-adds itself, but this carries few drawbacks that will be documented in the next section

## Drawbacks
A very clear drawback to this is the huge changes that will be needed to be made to the general node system to allow a concept such this one.

For example, to fullfil the needs of the Stream object, we need:
* A new-dependent detection system that is smart enough to determine whether thew dependent is actually new, or just one that re-added itself.
* A new context system that records the node that is currently being updated, to allow Operators like Read to actually get the wanted value.
* Various minor changes to the internal node system, such as determining how to know which dependent is a new one and when it's not.

While these points don't sound big on paper, they are absolutely huge in practice and the amount of manpower and work needed to solve these. In addition, these changes **will introduce performance costs if done incorrectly** 

## Alternatives
Not implement this feature.