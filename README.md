# Table of Contents
* [Introduction](#introduction)
* [Creating Collections](#creating-collections)
* [Available Methods](#available-methods)
* [Technical Details](#technical-details)
* [Performance](#performance)

# Introduction

Collect is a module designed to make working with tables in Roblox Luau significantly easier with much cleaner code. 
Collect supports simple arrays and complex nested dictionaries. 

This project was heavily inspired by Laravel [Collections](https://laravel.com/docs/10.x/collections) and the Laravel [Query Builder](https://laravel.com/docs/10.x/queries), and much of the functionality shares the same names as their Laravel counterparts.

# Creating Collections
A Collect object is, in essense, a wrapper for table manipulation. You can create a Collect object with the `new` function, accepting any table.
```lua
local tbl = {1, 2, 3, 4}
local collect = Collect.new(tbl)
```

If `new` receives a `tbl` that is a Collect object, it will return a deep copy of the object. This can be useful for running similar queries on the same original `tbl`.
```lua
local collect = Collect.new({1, 2, 3, 4, 5}):where(">=", 2)
local collect2 = Collect.new(collect):shuffle()

print(collect:get())
-- {2, 3, 4, 5}
print(collect2:get())
-- {5, 2, 4, 3}
```

You can also construct a new table of a given length with the `rep` function. The `rep` method takes two arguments; the `value` of each entry, and the `size` of the table. `value` can be anything, including a closure function.
```lua
local collect = Collect.rep(0, 5)

print(collect:get())
-- {0, 0, 0, 0, 0}
```
```lua
local collect = Collect.rep(function(index)
	return index
end, 5)

print(collect:get())
-- {1, 2, 3, 4, 5}
```

# Extending Collections
You can add your own methods to Collect on runtime. The `macro` method accepts a closure that will be executed when your method is called. An empty Collect object will be passed with the provided closure that can be used to access the Collect methods. For example, you can implement a `toUpper` method on top of the Collect module with the following code:
```lua
Collect:macro("toUpper", function(collect)
	return collect:map(function(value)
		return string.upper(value)
	end)
end)

local collect = Collect.new({"hello", "world"}):toUpper()

print(collect:get())
-- {"HELLO", "WORLD"}
```

If necessary, your macro can take additional arguments.
```lua
Collect:macro("punctuate", function(collect, puncuation)
	return collect:map(function(value)
		return value .. puncuation
	end)
end)

local collect = Collect.new({"hello", "world"}):punctuate("!")

print(collect:get())
-- {"hello!", "world!"}
```


# Available Methods
## `filter`
- `filter(path: string, closure: function)`
- `filter(closure: function)`
- `filter()`

The `filter` method filters the collection using the given closure, keeping only those items that pass a given truth test.
```lua
local collect = Collect.new(game.Players:GetPlayers()):filter("Character.Humanoid.Health", function(health, value)
	return health > 0
end)

print(collect:get())
-- {Player1, Player2}
```

```lua
local collect = Collect.new({1, 2, 3, 4}):filter(function(value, key)
	return value > 2
end)

print(collect:get())
-- {3, 4}
```lua
If no callback is given, all non-truthy items will be removed
```lua
local collect = Collect.new({1, 2, 3, false, nil}):filter()

print(collect:get())
-- {1, 2, 3}
```
For the inverse of the `filter` method, see the [`reject`](#reject) method

<hr>

## `reject`
- `reject(path: string, closure: function)`
- `reject(closure: function)`
- `reject()`


The `reject` method filters the collection using the given closure, keeping only those items that **do not** pass a given truth test.

```lua
local collect = Collect.new({1, 2, 3, 4}):reject(function(value, key)
	return value > 2
end)

print(collect:get())
-- {1, 2}
```

For the inverse of the `reject` method, see the [`filter`](#filter) method

<hr>

## `where`
- `where(path: string, operator: string, compare: any)`
- `where(path: string, compare: any)`
- `where(operator: string, compare: any)`
- `where(compare: any)`

The `where` method filters the collection by a given key value pair. The `operator` argument is a string of any valid operator (including some non-Luau operator), and defaults to `==` if omitted. 

The valid operators are: `=`, `==`, `~=`, `!=`, `<>`, `>=`, `<=`, `<`, `>`.

```lua
local collect = Collect.new(game.Players:GetPlayers()):where("Character.Humanoid.Health", ">", 0)

print(collect:get())
-- {Player1, Player2}
```
```lua
local collect = Collect.new(game.Players:GetPlayers()):where("DisplayName", "Gnarwhal")

print(collect:get())
-- {Gnarwhal}
```
The path argument can also be omitted for more traditional arrays:
```lua
local collect = Collect.new({1, 2, 3, 4}):where(">=", 3)

print(collect:get())
-- {3, 4}
```
A single `compare` operator can be passed for equality (equivalent to `:where(".", "==", 2`)
```lua
local collect = Collect.new({1, 2, 2, 3}):where(2)

print(collect:get())
-- {2, 2}
```

<hr>

## `whereBetween`
- `whereBetween(path: string, min: number, max: number)`
- `whereBetween(min: number, max: number)`

The `whereBetween` method filters the collection by determining if a numeric value is within a given range (inclusive).
```lua
local collect = Collect.new({1, 2, 3, 4, 5, 6}):whereBetween(2, 5)

print(collect:get())
-- {2, 3, 4, 5}
```
For the inverse of the `whereBetween` method, see the [`whereNotBetween`](#whereNotBetween) method

# Technical Details
## Collect Object
A Collect object, before a getter such as `get` is run is a table with the specifications of how the query is built. An example Collect object may look like:
```lua
local collect = Collect.new({1, 2, 3, 4}):where(">=", 2):shuffle()

print(collect)
--[[
	{
		Builder = {
			Actions = {
				{
					args = {},
					func = **function**,
					name = "shuffle",
				},
			},
			Filters = {
				{
					args = {nil, ">=", 2},
					func = **function**,
					name = "where",
				},
			},
			Updates = {},
		},
		Config = {
			DebugMode = false,
			DefaultPath = ".",
		},
		Table = {1, 2, 3, 4}
	}
--]]
```

# Performance
TODO
