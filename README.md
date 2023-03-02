# Collect

Collect is a module designed to make working with tables in Roblox Luau significantly easier with much cleaner code. 
Collect supports simple arrays and complex nested dictionaries. 

This project was heavily inspired by Laravel [Collections](https://laravel.com/docs/10.x/collections) and the Laravel [Query Builder](https://laravel.com/docs/10.x/queries), and much of the functionality shares the same names as their Laravel counterparts.

<hr>


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
