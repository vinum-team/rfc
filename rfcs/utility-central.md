# Central
## Summary
Introduce a new function to handle state centralization in an ergonomic way, and remove Selects.

## Motivation
State Centralization is hard to achieve in an ergonomic way with existing Vinum's already offered tools such as Selects. One main painpoint of Selects is that they don't typecheck due to lack of support for something like KeyOf in Luau's typechecking. Additionally, Selected States are entirely dynamic in shape, meaning code using Selects could be fragile and prone to errors.

Central could solve this by introducing a way to "group" fixed-shape state, while also remaining consistent with the rest of the library.

## Design
Centrals accept a `CentralInput`:
```lua
local myCharacterInfoCentral = scope:Central({
    health = 100, -- is regarded as "stableKey"
    midHealth = function(RNode, myCentral) -- "is regarded as "reactiveKey"
        -- myCentral is the same as myCharacterInfoCentral
        return Use(RNode, myCentral.health) / 2
    end
})
```

Centrals, being just a utility function, loop through `CentralInput` and perform the following:
1. Create an output table
2. Loop through the input table, and filter out values regarded as reactiveKeys
3. the remaining keys regarded as stableKeys, which we will convert them to Soucres and then put them in the output table, with the key as the orignial key (`health` in the previous example) and value as the created Source.
4. Convert the remaining reactiveKeys to Deriveds. 

The output table for the previous code snippet will have the same shape as doing something like this:
```lua
local characterInfo = {
    health = scope:Source(100)
}

characterInfo.midHealth = scope:Derived(function(RNode)
    return Use(RNode, characterInfo.health)
end)
```

While achieving what Centrals do is very possible natively, the problems arise when you need to add a derived that depends on an object that lives inside the grouping table.

## Drawbacks
This doesn't have any drawbacks since its a utility function, and the removal of Selects doesn't carry any drawbacks. 
## Alternatives
None.