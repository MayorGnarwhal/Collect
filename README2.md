# Table of Contents
* [Introduction](#introduction)
    * [Creating Collections](#creating-collections)
    * [Extending Collect](#extending-collect)
    * [Packages](#packages)
* [Available Methods](#available-methods)


# Introduction
Collect is a fluent and convenient wrapper for tables in Roblox Lua, designed for both simple and complex table manipulation.
Collect supports simple arrays to complex nested dictonaries

This project was heavily inspired by Laravel [Collections](https://laravel.com/docs/10.x/collections) and the Laravel [Query Builder](https://laravel.com/docs/10.x/queries).
Much of Collect's methods share the same names as their Laravel counterparts.


## Creating Collections
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

### Other Constructors
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


## Extending Collect
The Collect module is "macroable", allowing you to add additional methods to the module at run time. The `macro` method accepts a closure that is called when your macro is called. This closure is passed the current Collect object, providing access to the Collection's methods. The closure must return the existing, or a new, Collect object.
```lua
Collect:macro("toUpper", function(collect)
    return collect:map(function(value, key)
        return string.upper(value)
    end)
end)

local collect = Collect({"hello", "world"}):toUpper()
print(collect:get()) --> {"HELLO", "WORLD"}
```

### Macro Arguments
If necessary, you may define macros that accept additional arguments.
```lua
Collect:macro("multiplyBy", function(collect, mult)
    return collect:map(function(value, key)
        return value * mult
    end)
end)

local collect = Collect({1, 2, 3}):multiplyBy(10)
print(collect:get()) --> {10, 20, 30}
```

## Packages
For easily packagable Collect macros, Collect comes with an optional `macros` module (case-sensitive) that will automatically create the macros on run-time. This will allow you to create your own Collect packages that provide more specialized functionality. This Collect module comes pre-equiped with the `instances` package, that includes various additional methods for handling Instance objects.



# Available Methods
For the majority of the remaining Collect documentation, we will look at each method available on the `Collect` module. All of these methods can be chained to fluentry manipulate the underlying table. Most methods will return a new Collect object, allowing preservation of the original collection with necessary.


## `filter()`
Filters the collection using the given closure, keeping only entries that return truthy.
### Parameters
| **closure** | *function* | A callback function used to filter the collection |
| --- | --- | --- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4}):filter(function(value, key)
    return value > 2
end)

print(collect:get()) --> {3, 4}
```


## `reject()`
Filters the collection using the given closure, keeping only entries that do not return truthy.
### Parameters
| **closure** | *function* | A callback function used to filter the collection |
| --- | --- | --- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4}):reject(function(value, key)
    return value > 2
end)

print(collect:get()) --> {1, 2}
```


## `where()`
Filters collection using a given equation, keeping only entries that evalaute truthy
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **operator** | *string?* | An operator used for the evaluation. Valid operators are: `=`, `==`, `~=`, `!=`, `<>`, `>=`, `<=`, `<`, `>` <br> **Default Value:** `==` |
| **compare** | *any* | The fixed value compared against each entry |

### Code Samples
```lua
local collect = Collect(game.Players:GetPlayers()):where("Character.Humanoid.Health", ">=", 50)
print(collect:get()) --> {Player1, Player2}
```
```lua
local collect = Collect(game.Players:GetPlayers()):where("UserId", 38162374)
print(collect:get()) --> {MayorGnarwhal}
```
```lua
local collect = Collect({1, 2, 3, 4, 5}):where(">=", 3)
print(collect:get()) --> {3, 4, 5}
```
```lua
local collect = Collect(game.Players:GetPlayers()):where(game.Players.MayorGnarwhal)
print(collect:get()) --> {MayorGnarwhal}
```


## `whereBetween()`
Filters collections that entries whose numerical values are within [min, max]
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **min** | *number* | Minimum value in range |
| **max** | *number* | Maximum value in range |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5}):whereBetween(2, 4)
print(collect:get()) --> {2, 3, 4}
```


## `whereNotBetween()`
Filters collections that entries whose numerical values are not within [min, max]
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **min** | *number* | Minimum value in range |
| **max** | *number* | Maximum value in range |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5}):whereNotBetween(2, 4)
print(collect:get()) --> {1, 5}
```


## `whereIn()`
Filters collection to values found in a given array
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **haystack** | *table \| Collect* | Array in which to search |

### Code Samples
```lua
local whitelistUserIds = {38162374}
local collect = Collect(game.Players:GetPlayers()):whereIn("UserId", whitelistUserIds)
print(collect:get()) --> {MayorGnarwhal}
```
```lua
local collect = Collect({1, 2, 3, 4, 5}):whereIn({1, 3, 5})
print(collect:get()) --> {1, 3, 5}
```


## `whereNotIn()`
Filters collection to values that are not found in a given array
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **haystack** | *table \| Collect* | Array in which to search |

### Code Samples
```lua
local blacklistUserIds = {38162374}
local collect = Collect(game.Players:GetPlayers()):whereNotIn("UserId", blacklistUserIds)
print(collect:get()) --> {MayorGnarwhal}
```
```lua
local collect = Collect({1, 2, 3, 4, 5}):whereNotIn({1, 3, 5})
print(collect:get()) --> {2, 4}
```
