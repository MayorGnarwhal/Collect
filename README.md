# Table of Contents
* [Introduction](#introduction)
* [Available Methods](#available-methods)
* [Metamethods](#metamethods)
* [Extending Collect](#extending-collect)
    * [Packages](#packages)
* [Installation](#installation)


# Introduction
This project was heavily inspired by Laravel [Collections](https://laravel.com/docs/10.x/collections) and the Laravel [Query Builder](https://laravel.com/docs/10.x/queries).
Much of Collect's methods share the same names as their Laravel counterparts.

Collect is a fluent and convenient wrapper for tables in Roblox Luau, designed for both simple and complex table manipulation. Collect supports simple arrays to complex nested dictonaries, being able to differentiate between the two, and automatically treat arrays and dictonaries differently. For example, if a Collect object is created with an array, then it will maintain that structure as values are added and removed from the collection.

Along with providing various methods for table manipulation, Collect objects can also be handled very similarily to standard Luau tables. Making use of [metamethods](#metamethods), Collect implements features such as [negative indexing](#__index) and [dictonary length](#__len).



# Available Methods
For the majority of the remaining Collect documentation, we will look at each method available on the `Collect` module. All of these methods can be chained to fluentry manipulate the underlying table. Most methods will return a new Collect object, allowing preservation of the original collection with necessary.

There are various types of methods that Collect offers that interact with the collection in various different ways.
|     |     |
| :-- | :-- |
| **[Constructors](#constructors)** | Creates a new Collect object using a standard Luau table |
| **[Filters](#filters)** | Removes entries from collection. Returns a new Collect object |
| **[Mutators](#mutators)** | Mutates the structure of the collection. Returns a new Collect object |
| **[Sorters](#sorters)** | Changes how the collection is ordered. Only applicable for array collections. Changes the underlying table |
| **[Getters](#getters)** | Returns information about the collection. Does not change collection |
| **[Update](#update)** | Changes the underlying table |
| **[Iterators](#iterators)** | Iterates through the collection |


# Constructors
## `new()`
### Parameters
Create a new Collect object with a table.
| **tbl** | *table \| Collect?* | Table used in Collect object. If tbl is another Collect object, then its underlying table is used. <br> **Default Value:** `{}` |
| --- | --- | :-- |

### Code Samples
```lua
local tbl = {1, 2, 3, 4}
local collect = Collect.new(tbl)

print(collect:get()) --> {1, 2, 3, 4}
print(collect:get() == tbl) --> true
```

`new()` is called internally with the `__call` metamethod, allowing you to call the Collect module as a function.
```lua
local collect = Collect({1, 2, 3, 4})
print(collect:get()) --> {1, 2, 3, 4}
```

## `clone()`
### Parameters
Creates a new Collect object with a deep copy of the given table
| **tbl** | *table \| Collect?* | Table used in Collect object. If tbl is another Collect object, then its underlying table is used. <br> **Default Value:** `{}` |
| --- | --- | :-- |
```lua
local tbl = {1, 2, 3, 4}
local collect = Collect.new(tbl)

print(collect:get()) --> {1, 2, 3, 4}
print(collect:get() == tbl) --> false
```


## `create()`
### Parameters
Creates a new Collect object of a specified length populated by a given value or closure
|     |     |     |
| --- | --- | --- |
| **count** | *number* | Length of the new collection |
| **value** | *any \| function* | The value put in each entry. If value is a function, then the return value of the closure is used |

### Code Samples
```lua
local collect = Collect.create(5, 1)
print(collect:get()) --> {1, 1, 1, 1, 1}
```
```
local collect = Collect.create(5, function(i)
    return i
end)

print(collect:get()) --> {1, 2, 3, 4, 5}
```



# Filters
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



# Mutators
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

### Code Samples
```lua
local collect = Collect({
    ["ABC"] = {ProductId = "ABC", Name = "Object1"},
    ["XYZ"] = {ProductId = "XYZ", Name = "Object2"},
})

local keys = collect:keys()
print(keys:get()) --> {"ABC", "XYZ"}
```


## `map()`
Re-maps the values of the collection by a given closure.
### Parameters
| **closure** | *function* | Closure used for re-mapping. Returned value overwrites current value |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:map(function(value, key)
    return value * 2
end)
print(collect:get()) --> {2, 4, 6, 8, 10}
```


## `mapWithKeys()`
Re-maps entire collection by a given closure. The closure should return a key, pair tuple that will become the new entry.
### Parameters
| **closure** | *function* | Closure used for re-mapping. Returned key, value pair overwrites current entry |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({
    {ProductId = "ABC", Name = "Object1"},
    {ProductId = "XYZ", Name = "Object2"},
})

collect = collect:mapWithKeys(function(value, key)
    return value.ProductId, value.Name
end)
print(collect:get()) -->
--[[
    ["ABC"] = "Object1",
    ["XYZ"] = "Object2",
]]
```


## `nth()`
Creates a collection consisting of every n-th element, starting at an optional offset
### Parameters
|     |     |     |
| :-- | :-- | :-- |
| **frequency** | *number* |  |
| **offset** | *number?* | Offset index to start pulling from |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5, 6, 7, 8})

collect = collect:nth(3)
print(collect:get()) --> {1, 4, 7}
```
```lua
local collect = Collect({1, 2, 3, 4, 5, 6, 7, 8})

collect = collect:nth(3, 2)
print(collect:get()) --> {2, 5, 8}
```


## `pluck()`
Re-maps collection to values at a given path. Optionally can specify how remapped collection should be keyed.
### Parameters
|     |     |     |
| :-- | :-- | :-- |
| **path** | *string* | Path to value |
| **keyPath** | *string?* | Path to value used to key |

### Code Samples
```lua
local collect = Collect({
    {ProductId = "ABC", Name = "Object1"},
    {ProductId = "XYZ", Name = "Object2"},
})

collect = collect:pluck("ProductId")
print(collect:get()) --> {"ABC", "XYZ"}
```


## `skip()`
Removes the given number of elements from the beginning of the collection. If the number number is less than 1, then a percentage of the collection's length is skipped.
### Parameters
| **count** | *number* | Number or proportion of elements to skip |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5, 6, 7, 8})

collect = collect:skip(4)
print(collect:get()) --> {5, 6, 7, 8}
```
```lua
local collect = Collect({1, 2, 3, 4, 5, 6, 7, 8})

collect = collect:skip(0.5)
print(collect:get()) --> {5, 6, 7, 8}
```


## `skipUntil()`
Removes elements from the beginning of the collection until a given closure returns truthy. Optionally, a simple value can be passed to skip until that value is found.
### Parameters
| **closure** | *function \| any* | Used to determine when skipping ends |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:skipUntil(function(value, key)
    return value >= 3
end)
print(collect:get()) --> {3, 4, 5}
```
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:skipUntil(3)
print(collect:get()) --> {3, 4, 5}
```


## `skipWhile()`
Removes elements from the beginning of the collection while a given closure returns truthy.
### Parameters
| **closure** | *function \| any* | Used to determine when skipping ends |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:skipWhile(function(value, key)
    return value <= 3
end)
print(collect:get()) --> {4, 5}
```


## `slice()`
Returns the slice of the collection from [`startIndex`, `endIndex`]. The collection's underlying table must be an array.
### Parameters
|     |     |     |
| :-- | :-- | :-- |
| **startIndex** | *number* | Inclusive starting index of the slice |
| **endIndex** | *number?* | Inclusive end index of the slice. <br> **Default Value:** Length of collection |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:slice(2, 4)
print(collect:get()) --> {2, 3, 4}
```
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:slice(2)
print(collect:get()) --> {2, 3, 4, 5}
```


## `split()`
Splits the collection into a given number of groups.
### Parameters
| **numGroups** | *number* | Number of groups to create |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:split(3)
print(collect:get()) --> {{1, 2}, {3, 4}, {5}}
```


## `take()`
Takes the given number of elements from the beginning of the collection. If the number number is less than 1, then a percentage of the collection's length is taken.
### Parameters
| **count** | *number* | Number or proportion of elements to take |
| :-- | :-- | :-- |

### Code Samples
### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5, 6, 7, 8})

collect = collect:take(4)
print(collect:get()) --> {1, 2, 3, 4}
```
```lua
local collect = Collect({1, 2, 3, 4, 5, 6, 7, 8})

collect = collect:take(0.5)
print(collect:get()) --> {1, 2, 3, 4}
```


## `takeUntil()`
Takes elements from the beginning of the collection until a given closure returns truthy. Optionally, a simple value can be passed to skip until that value is found.
### Parameters
| **closure** | *function \| any* | Used to determine when taking ends |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:takeUntil(function(value, key)
    return value >= 3
end)
print(collect:get()) --> {1, 2}
```
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:takeUntil(3)
print(collect:get()) --> {1, 2, 3}
```


## `takeWhile()`
Removes elements from the beginning of the collection while a given closure returns truthy.
### Parameters
| **closure** | *function \| any* | Used to determine when skipping ends |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:takeWhile(function(value, key)
    return value <= 3
end)
print(collect:get()) --> {1, 2, 3}
```


## `toArray()`
Turns a dictonary collection into an array of key, value pairs.
### Code Samples
```lua
local collect = Collect({
    a = 1,
    b = 2,
    c = 3,
})

collect = collect:toArray()
print(collect:get()) -->
--[[
    {key = "a", value = 1},
    {key = "b", value = 2},
    {key = "c", value = 3},
]]
```


## `unique()`
Removes all duplicate values from collection.
### Code Samples
```lua
local collect = Collect({1, 1, 2, 3, 3, 3, 4, 5})

local unique = collect:unique()
print(unique:get()) --> {1, 2, 3, 4, 5}
```


## `values()`
Returns an array of the collection's values
### Code Samples
```lua
local collect = Collect({
    ["ABC"] = {ProductId = "ABC", Name = "Object1"},
    ["XYZ"] = {ProductId = "XYZ", Name = "Object2"},
})

local values = collect:values()
print(values:get()) -->
--[[
    {ProductId = "ABC", Name = "Object1"},
    {ProductId = "XYZ", Name = "Object2"},
]]
```



# Sorts
Sort methods change the order of the collection's underlying table. If you do not wish to affect the underlying table, use the [.clone()](#clone) constructor.

**Note:** All sort methods are only available for arrays. If a sort method is called on a dictonary collection, an error will be thrown.

## `flip()`
Reverses the order of the collection.
### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect = collect:flip()
print(collect:get()) --> {5, 4, 3, 2, 1}
```


## `sort()`
Sorts the collection according to a given sort function. If no sort function is given, then collection is sorted in ascending order.
### Parameters
| **sortFunction** | *function?* | An optional comparison function used to sort. Recieves two elements, `a`, and `b`. If return value is truthy, then `a` will be ordered before `b` |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 3, 4, 5, 2})

collect:sort()
print(collect:get()) --> {1, 2, 3, 4, 5}
```
```lua
local collect = Collect({1, 3, 4, 5, 2})

collect:sort(function(a, b)
    return a > b
end)
print(collect:get()) --> {5, 4, 3, 2, 1}
```


## `sortDesc()`
Sorts the collection in descending order.
### Code Samples
```lua
local collect = Collect({1, 3, 4, 5, 2})

collect:sortDesc()
print(collect:get()) --> {5, 4, 3, 2, 1}
```


## `sortBy()`
Sorts the collection in ascending order based on the values at a given path.
### Parameters
| **path** | *string* | Relative path from each entry to the value to be compared in sort |
| :-- | :-- | :-- |

### Code Samples
```lua

```


## `sortByDesc()`
Sorts the collection in descending order based on the values at a given path.
### Parameters
| **path** | *string* | Relative path from each entry to the value to be compared in sort |
| :-- | :-- | :-- |


## `shuffle()`
Rearanges collection into a random order.
### Parameters
| **random** | *Random \| number* | [Random](https://create.roblox.com/docs/reference/engine/datatypes/Random) generator object or a random seed used for shuffle |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

local randomSeed = 1337
collect:shuffle(randomSeed)
print(collect:get()) --> {2, 4, 3, 5, 1}
```


## `weightedShuffle()`
Rearanges collection into a random order based on weighted odds, with higher weights more likely to be at the beginning of the collection.
### Parameters
|     |     |     |
| :-- | :-- | :-- |
| **path** | *string?* | Relative path from each entry to the entry's weight |
| **random** | *Random \| number* | [Random](https://create.roblox.com/docs/reference/engine/datatypes/Random) generator object or a random seed used for shuffle |

### Code Samples
```lua
local collect = Collect({
    {Value = "A", Weight = 20},
    {Value = "B", Weight = 99999},
    {Value = "C", Weight = 2},
    {Value = "D", Weight = 50},
    {Value = "E", Weight = 0.0001},
})

local randomSeed = 1337
collect:weightedShuffle("Weight", randomSeed)
print(collect:get()) -->
--[[
    {Value = "B", Weight = 99999},
    {Value = "A", Weight = 20},
    {Value = "D", Weight = 50},
    {Value = "C", Weight = 2},
    {Value = "E", Weight = 0.0001},
]]
```


# Getters
## `get()`
Returns the collection's underlying table.


## `average()`
Returns the average value of the collection.
### Parameters
| **path?** | *string* | Relative path from each entry to the value to be averaged |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 1, 2, 4})

print(collect:average()) --> 3
```


## `collect()`
Creates and returns a new Collect object that is a deep copy of the current collection.


## `concat()`
### Parameters
Joins a collection into a string, combined by the glue.
|     |     |     |
| :-- | :-- | :-- |
| **glue** | *string?* | Glue used to combine each entry <br> **Default Value:** `""` |
| **startIndex** | *number?* |  |
| **endIndex** | *number?* |  |

### Code Samples
```lua
local collect = Collect({"Foo", "Bar", "Baz"})

print(collect:concat()) --> "FooBarBaz"
```
```lua
local collect = Collect({"Foo", "Bar", "Baz"})

print(collect:concat(", ")) --> "Foo, Bar, Baz"
```


## `count()`
### Aliases
* `length()`
  
Returns the number of key, value pairs in the collection. This method also works for the length of dictonaries. Called internally by the [`__len`](#__len) metamethod, allowing the use of the `#` operator.
### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

print(collect:count()) --> 5
```
```lua
local collect = Collect({a = 1, b = 2, c = 3})

print(collect:count()) --> 3
```


## `equals()`
Returns a boolean indicating if the given table has all of the same key, value pairs. Called internally by the [`__eq'](#__eq) metamethod, allowing the use of the `#` operator.
### Parameters
| **tbl** | *table \| Collect* | Table or collection to be compared with |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect1 = Collect({
    a = 1,
    b = 2,
})
local collect2 = Collect({
    a = 1,
    b = 2,
})

print(collect1:equals(collect2)) --> true
```
```lua
local collect = Collect({1, 2, 3})

print(collect:equals({1, 2, 3})) --> true
```


## `every()`
Returns a boolean indicating if every key, value pair in the collection satisfies a given closure.
### Parameters
| **closure** | *function* | Takes two arguments, the `value` and `key` of an entry |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3, 4, 5})

print(collect:every(function(value, key)
    return value < 10
end)) --> true
```


## `first()`
Returns the first element of the collection if it is an array. If the collection is a dictonary, then a random element is returned.
### Code Samples
```lua
local collect = Collect({1, 2, 3})

print(collect:first()) --> 1
```

## `firstWhere()`
Returns the first value, key pair that satisfies the given equation. Optionally, can instead pass a single closure that returns the first truthy entry. Returns `nil` if no entry satisfies the condition.
### Parameters
|     |     |     |
| --- | --- | --- |
| **path** | *string?* | Relative path from each entry to the value to be compared <br> **Default Value:** `"."` |
| **operator** | *string?* | An operator used for the evaluation. Valid operators are: `=`, `==`, `~=`, `!=`, `<>`, `>=`, `<=`, `<`, `>` <br> **Default Value:** `==` |
| **compare** | *any* | The fixed value compared against each entry |

Or by a given closure:
| **closure** | *function* | Takes two arguments, the `value` and `key` of an entry |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({4, 8, 15, 16, 23, 42})

local value, key = collect:firstWhere(">", 10)
print(value) --> 15
print(key)   --> 3
```
```lua
local collect = Collect({
    a = 1,
    b = "two",
    c = "three",
    d = 5,
})

local value, key = collect:firstWhere(function(value, key)
    return typeof(value) == "string"
end)
print(value) --> "three"
print(key)   --> "c"
```


## `find()`
Returns the key of the given value in the collection.
### Parameters
|     |     |     |
| :-- | :-- | :-- |
| **needle** | *any* | The value to search for in the collection |
| **init** | *number?* | Optional starting index for search. Only applicable for arrays |

### Code Samples
```lua
local collect = Collect({4, 8, 15, 16, 23, 42})

print(collect:find(15)) --> 3
```
```lua
local collect = Collect({2, 8, 3, 2, 5})

print(collect:find(2, 3)) --> 4
```


## `isEmpty()`
Returns boolean indicating if collection is empty.
```lua
local collect = Collect()
print(collect:isEmpty()) --> true
```


## `isNotEmpty()`
Returns boolean indicating if collection is not empty.
```lua
local collect = Collect()
print(collect:isNotEmpty()) --> false
```


## `last()`
Returns the last element of the collection if it is an array. If the collection is a dictonary, then a random element is returned.
```lua
local collect = Collect({1, 2, 3})

print(collect:last()) --> 3
```


## `max()`
Returns a (value, max, key) tuple indicating the largest element in the collection.
### Parameters
| **path?** | *string* | Relative path from each entry to the value to be compared |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({
    {Value = "A", Weight = 20},
    {Value = "B", Weight = 99999},
    {Value = "C", Weight = 2},
    {Value = "D", Weight = 50},
    {Value = "E", Weight = 0.0001},
})

local value, max, key = collect:max("Weight")
print(value) --> {Value = "B", Weight = 99999}
print(max)   --> 99999
print(key)   --> 2
```


## `median()`
Returns the median of the collection. If the length of the collection is odd, then it returns the average of the two middle values. This method only works for arrays.
### Parameters
| **path?** | *string* | Relative path from each entry to the value to be evaluated |
| :-- | :-- | :-- |

### Code Samples


# Updates
## `insert()`
Inserts a value into the collection. The collection's underlying table must be an array.
### Parameters
|     |     |     |
| :-- | :-- | :-- |
| **position** | *number?* | Position to insert value at. If not given, then value is appended to the end of the collection |
| **value** | *any* | Value to insert into collection |

### Code Samples
```lua
local collect = Collect({"a", "b", "c"})

collect:insert(1, "d")
print(collect:get()) -> {"d", "a", "b", "c"}
```
```lua
local collect = Collect({"a", "b", "c"})

collect:insert("d")
print(collect:get()) -> {"a", "b", "c", "d"}
```


## `merge()`
Merges a given table into the collection. Changes the underlying table.
### Parameters
| **tbl** | *table \| Collect* | Table to be merged |
| :-- | :-- | :-- |

### Code Samples
```lua
local collect = Collect({1, 2, 3})

collect:merge({"a", "b", "c"})
print(collect:get()) -> {1, 2, 3, "a", "b", "c"}
```



# Metamethods
Collect implements various Lua metamethods to make working with Collect objects as similar to standard tables as possible, with the added bonus of increased functionality. 

## `__index`
Fires when `collect[key]` is indexed. Implements negative array indexing, allowing you to index from the back of an array with negative numbers.
```lua
local collect = Collect({1, 2, 3, 4, 5})
print(collect[-1]) --> 5
```


## `__newindex`
Fires when `collect[key]` is attempted to be set (ie. `collect[key] = value`). Implements negative array indexing, allowing you to index from the back of an array with negative numbers.
```lua
local collect = Collect({1, 2, 3, 4, 5})

collect[1] = 10
collect[-1] *= 10
print(collect:get()) --> {10, 2, 3, 4, 50}
```


## `__len`
Fires when the `#` operator is used (ie `#collect`). Implements the ability to count the length of a dictonary. This metamethod is simply a call to the [:count()](#count) method.
```lua
local collect = Collect({
    a = 1,
    b = 2,
})

print(#collect) --> 2
```


## `__eq`
Fires when the `==` is used to compare the equality of two Collect objects. Unlike the `==` operator with standard tables, this metamethod **does not compare the table's pointer**, but rather that each key, value pair is shared between the two tables. This metamethod is simply a call to the [:equals()](#equals) method.
```lua
local collect = Collect({
    a = 1,
    b = 2,
})
local collect2 = Collect({
    b = 2,
    a = 1,
})

print(collect1 == collect2) --> true
```
To instead compare the table pointers of two Collect objects, use the [:get()](#get) method.
```lua
local tablePointer = {1, 2, 3}
local collect1 = Collect(tablePointer)
local collect2 = Collect(tablePointer)

print(collect1:get() == collect2:get()) --> true
```


## `__pairs` and `__ipairs`
Unfortunately, Roblox Luau does not support the use of the `__pairs` and `__ipairs` metamethods. Instead, you can make use of the [:forEach()](#forEach) method for iteration, or simply iterate the underlying table with the [:get()](#get) method.



# Extending Collect
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


# Packages
For easily packagable Collect macros, Collect comes with an optional `macros` module (case-sensitive) that will automatically create the macros on run-time. This will allow you to create your own Collect packages that provide more specialized functionality. This Collect module comes pre-equiped with the `instances` package, that includes various additional methods for handling Instance objects.


# Installation
Collect can be downloaded on the Roblox store at the following link:

https://www.roblox.com/library/14343691152/Collect
