# Table of Contents
* [Introduction](#introduction)
    * [Creating Collections](#creating-collections)
    * [Extending Collect](#extending-collect)
    * [Packages](#packages)
* [Available Methods](#available-methods)
    * [Constructors](#constructors)
    * [Filters](#filters)
    * [Mutators](#mutators)
    * [Sorters](#sorters)
    * [Getters](#getters)
    * [Update](#update)
    * [Iterators](#iterators)


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

There are various types of methods that Collect offers that interact with the collection in various different ways.
|     |     |
| :-- | :-- |
| **[Constructors](#constructors)** | Creates a new Collect object |
| **[Filters](#filters)** | Removes entries from collection. Returns a new Collect object |
| **[Mutators](#mutators)** | Mutates the structure of the collection. Returns a new Collect object |
| **[Sorters](#sorters)** | Changes how the collection is ordered. Only applicable for array collections. Changes the underlying table |
| **[Getters](#getters)** | Gets some information from the collection. Does not change collection |
| **[Update](#update)** | Update values in the collection |
| **[Iterators](#iterators)** | Iterates through the collection |


# Constructors


## `filter()`
Filters the collection using the given closure, keeping only entries that return truthy.
### Parameters
| **closure** | *function* | A callback function used to filter the collection |
| --- | --- | --- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4})

local filtered = collect:filter(function(value, key)
    return value > 2
end)
print(filtered:get()) --> {3, 4}
```


## `reject()`
Filters the collection using the given closure, keeping only entries that do not return truthy.
### Parameters
| **closure** | *function* | A callback function used to filter the collection |
| --- | --- | --- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4})

local filtered = collect:reject(function(value, key)
    return value > 2
end)
print(filtered:get()) --> {1, 2}
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
local players = Collect(game.Players:GetPlayers())

local healthyPlayers = players:where("Character.Humanoid.Health", ">=", 50)
print(healthyPlayers:get()) --> {Player1, Player2}
```
```lua
local ownerId = 38162374
local players = Collect(game.Players:GetPlayers())

local owners = players:where("UserId", ownerId)
print(owners:get()) --> {MayorGnarwhal}
```
```lua
local collect = Collect({1, 2, 3, 4, 5})

local filtered = collect:where(">=", 3)
print(filtered:get()) --> {3, 4, 5}
```
```lua
local collect = Collect({1, 2, 2, 3, 4})

local filtered = collect:where(2)
print(collect:get()) --> {2, 2}
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
local collect = Collect({1, 2, 3, 4, 5})

local filtered = :whereBetween(2, 4)
print(filtered:get()) --> {2, 3, 4}
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
local collect = Collect({1, 2, 3, 4, 5})

local filtered = collect:whereNotBetween(2, 4)
print(filtered:get()) --> {1, 5}
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
local adminUserIds = {38162374}
local players = Collect(game.Players:GetPlayers())

local admins = players:whereIn("UserId", adminUserIds)
print(admins:get()) --> {MayorGnarwhal}
```
```lua
local collect = Collect({1, 2, 3, 4, 5})

local filtered = collect:whereIn({1, 3, 5, 8})
print(filtered:get()) --> {1, 3, 5}
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
local collect = Collect({1, 2, 3, 4, 5})

local filtered = collect:whereNotIn({1, 3, 5, 8})
print(filtered:get()) --> {2, 4}
```


## `whereNil()`
Filters collection to values that are nil at the given path
### Parameters
| **path** | *string* | Relative path from each entry to the value to be compared |
| --- | --- | --- |


## `whereNotNil()`
### Aliases
* `whereHas()`

Filters collection to values that are nil at the given path
### Parameters
| **path** | *string* | Relative path from each entry to the value to be compared |
| --- | --- | --- |

### Code Samples
```lua
local loadedPlayers = Collect(game.Players:GetPlayers()):whereNotNil("Character")
```


## `whereOfType()`
Filters collection to values where `typeof(value) == type`
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **type** | *string* | The variable type in which to match |

### Code Samples
```lua
local collect = Collect({1, "Two", 3, Enum.KeyCode.Four, "Five"})

local strings = collect:whereOfType("string")
print(strings:get()) --> {"Two", "Five"}
```


## `whereNotOfType`
Filters collection to values where `typeof(value) ~= type`
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **type** | *string* | The variable type in which to match |


## `chunk()`
Breaks collection into multiple smaller tables of a given size
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **type** | *string* | The variable type in which to match |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5, 6, 7})

local chunks = collect:chunk(4)
print(chunks:get()) --> {{1, 2, 3, 4}, {5, 6, 7}}
```


## `chunkWhile()`
Breaks collection into multiple smaller tables, appending the current chunk while the closure returns truthy
### Parameters
| **closure** | *function* | Closure used in forming chunks. If returned value is truthy, a new chunk is created |
| --- | --- | --- |

### Code Samples
```lua
local collect = Collect({1, 1, 2, 2, 3, 3, 3, 4})

local chunks = collect:chunkWhile(function(value, key, chunk)
    return value == chunk[#chunk]
end)
print(chunks:get()) --> {{1, 1}, {2, 2}, {3, 3, 3}, {4}}
```


## `countBy()`
Counts the occurences of values in the collection
### Parameters
| **determineValue** | *string? \| function?* | Method for determining value to count. Can be a path, or a closure who's return value is the counted value. If `nil`, then the method counts each element |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 1, 3, 4, 4, 4})

local counted = collect:countBy()
print(counted:get()) --> {[1] = 2, [3] = 1, [4] = 3}
```
```lua
local collect = Collect({"alice@gmail.com", "bob@yahoo.com", "carlos@gmail.com"})

local counted = collect:countBy(function(email, key)
    return string.split(email, "@")[2]
end)
print(counted:get()) --> {["gmail.com"] => 2, ["yahoo.com"] => 1}
```


## `diffAssoc()`
### Parameters
Compares another table or collection's key, value pairs. Returns collection of key, value pairs not present in given table
| **compare** | *table \| Collect* | Table used to compare key, value pairs |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({
    color = BrickColor.Yellow();
    count = 3;
    foo = "bar";
})

local diff = collect:diffAssoc({
    color = BrickColor.Green();
    count = 3;
    foo = "baz";
    used = 2;
})
print(diff:get()) --> {color = BrickColor.Yellow(), foo = "bar"}
```


## `diffKeys()`
### Parameters
Compares against another table or collection based on its keys. Returns collection of key, value pairs in the original collection whose keys are not in the given collection.
| **compare** | *table \| Collect* | Table used to compare key, value pairs |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({
    color = BrickColor.Yellow();
    count = 3;
    used = 2;
    foo = "bar";
})

local diff = collect:diffKeys({
    color = BrickColor.Green();
    count = 3;
    baz = "bar";
})
print(diff:get()) --> {used = 2, foo = "bar"}
```


## `duplicates()`
### Parameters
Counts duplicate occurences of values in the collection
| **determineValue** | *string? \| function?* | Method for determining value to count. Can be a path, or a closure who's return value is the counted value. If `nil`, then the method counts each element |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 1, 2, 3, 4, 4, 4})

local duplicates = collect:duplicates()
print(duplicates:get()) --> {[1] = 2, [4] = 3}
```


## `flatten()`
Flattens a multi-dimension collection to a single dimension, or until a specified depth is reached
### Parameters
Counts duplicate occurences of values in the collection
| **depth** | *number?* | Maximum depth of flattening. <br> **Default:** `math.huge` |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({
    name = "Bob",
    numbers = {1, 2}
})

local flattened = collect:flatten()
print(flattened:get()) --> {"Bob", 1, 2}
```
```lua
local collect = Collect({
    Challenge1 = {
        Difficulty = "Easy";
        Reward = {
            Type = "Cash";
            Value = 100;
        };
    };
    Challenge2 = {
        Difficulty = "Medium";
        Reward = {
            Type = "XP";
            Value = 250;
        };
    }
})

local flattened = collect:flatten(1)
print(flattened:get()) --> 
--[[
    [1] = {
        Difficulty = "Easy";
        Reward = {Type = "Cash", Value = 100};
    };
    [2] = {
        Difficulty = "Medium";
        Reward = {Type = "XP", Value = 250};
    };
]]
```


## `groupBy()`
Groups the collection's entries by a given path or closure
### Parameters
| **determineGroup** | *string? \| function?* | Method for determining an entry's group. Can be a path, or a closure whose return value is the group. If `nil`, then the method groups by elements |
| :-- | :-- | :-- |

### Sample Code
```lua
local collect = Collect({
    Challenge1 = {Difficulty = "Easy"},
    Challenge2 = {Difficulty = "Easy"},
    Challenge3 = {Difficulty = "Medium"},
    Challenge4 = {Difficulty = "Hard"},
    Challenge5 = {Difficulty = "Hard"},
})

local grouped = collect:groupBy("Difficulty")
print(grouped:get()) --> 
--[[
    Easy = {
        Challenge1 = {Difficulty = "Easy"},
        Challenge2 = {Difficulty = "Easy"},
    };
    Medium = {
        Challenge3 = {Difficulty = "Medium"},
    };
    Hard = {
        Challenge5 = {Difficulty = "Hard"},
    };
]]
```


## `keyBy()`
Re-keys the collection by a given path or closure. If the same key is set multiple times, then only the last one will appear in the new collection
### Parameters
Counts duplicate occurences of values in the collection
| **determineKey** | *string \| function* | Method for determining new keys. Can be a path, or a closure whose return value is the new key. |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({
	{ProductId = "ABC", Name = "Object1"},
	{ProductId = "XYZ", Name = "Object2"},
})

local keyed = collect:keyBy("ProductId")
print(keyed:get()) -->
--[[
    ["ABC"] = {ProductId = "ABC", Name = "Object1"},
    ["XYZ"] = {ProductId = "XYZ", Name = "Object2"},
]]
```
```lua
local collect = Collect({
	{ProductId = "ABC", Name = "Object1"},
	{ProductId = "XYZ", Name = "Object2"},
})

local keyed = collect:keyBy(function(value, key)
    return string.lower(value.ProductId)
end)
print(keyed:get()) -->
--[[
    ["abc"] = {ProductId = "ABC", Name = "Object1"},
    ["xyz"] = {ProductId = "XYZ", Name = "Object2"},
]]
```


## `keys()`
Returns all of the collection's keys
```lua
local collect = Collect({
    ["ABC"] = {ProductId = "ABC", Name = "Object1"},
    ["XYZ"] = {ProductId = "XYZ", Name = "Object2"},
})

local keys = collect:keys()
print(keys:get()) --> {"ABC", "XYZ"}
```
