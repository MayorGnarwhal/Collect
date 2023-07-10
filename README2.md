# Table of Contents
* [Introduction](#introduction)
* [Creating Collections](#creating-collections)
* [Extending Collect](#extending-collect)


# Introduction
Collect is a fluent and convenient wrapper for tables in Roblox Lua, designed for both simple and complex table manipulation.
Collect supports simple arrays to complex nested dictonaries

This project was heavily inspired by Laravel [Collections](https://laravel.com/docs/10.x/collections) and the Laravel [Query Builder](https://laravel.com/docs/10.x/queries).
Much of Collect's methods share the same names as their Laravel counterparts.


# Creating Collections
A standard Collect object can be created with the `new` constructor.
```lua
local collect = Collect.new({1, 2, 3})
```

Following Laravel standards, the Collect module is also callable as a function, using the `new` constructor under the hood. For the purposes of the documentation, this will be the standard that is used.
```lua
local collect = Collect({1, 2, 3}
```

When creating a Collect object with the `new` constructor, a reference to the provided table is used. To create a Collect object with a clone of a table, use the `clone` constructor, which will deep copy the provided table before creating an object.
```lua
local ValueModule = require(game.ReplicatedStorage.GameValues)

local collect = Collect.clone(ValueModule)
```

Collect also implements the `create` constructor from Roblox's [table library](https://create.roblox.com/docs/reference/engine/libraries/table#create), accepting a number indicating the table size, and a value that will be populated in each index.
```lua
local collect = Collect.create(3, "Roblox")
print(collect:get()) --> {"Roblox", "Roblox", "Roblox"}
```

The `create` constructor also accepts functions as a valid value argument, acting as a closure that is called to determine the value at each index.
```lua
local collect = Collect.create(5, function(index)
    return index * 2
end)

print(collect:get()) --> {2, 4, 6, 8, 10}
```


# Extending Collect
The Collect module is "macroable", allowing you to add additional methods to the module at run time.
