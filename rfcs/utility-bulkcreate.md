# BulkCreate
## Summary
Introduce a new function to handle state centralization in an ergonomic way, and remove Selects.

## Motivation
State Centralization is hard to achieve in an ergonomic way with existing Vinum's already offered tools such as Selects. One main painpoint of Selects is that they don't typecheck due to lack of support for something like KeyOf in Luau's typechecking. Additionally, Selected States are entirely dynamic in shape, meaning code using Selects could be fragile and prone to errors.

BulkCreate could solve this by introducing a way to "group" fixed-shape state, while also remaining consistent with the rest of the library.

## Design
Centrals accept a `function` that takes a table to populate:
```lua
local myCharacterInfo, myScope = BulkCreate(function(data, scope)
    data.Health = scope:Source(100)
    data.midHealth = scope:Derived(function(self)
        return Use(self, data.Health)
    end)
end)

print(Read(myCharacterInfo.Health)) -- 100

Character.Dead:Connect(function()
    Kill(myScope)
end)
```

BulkCreate, being a simple utility function, will create an empty table. then fire the callback with the created table and a created scope, and then return that table plus the scope.

A design this simple is also very performant behind the hood, and the previous painpoints that Selects brought are entirely removed *(data is a table, and the type system is happy with this)*.

## Drawbacks
This doesn't have any drawbacks since its a utility function, and the removal of Selects doesn't carry any drawbacks. 
## Alternatives
None.