[TOC]

### Built-in Data Types

#### Tuples

Tuples are used to store multiple items in a single variable.

A tuple is a collection which is **ordered** and **unchangeable**.

Tuples are written with round brackets.

#### Lists

#### Sets

Sets are used to store multiple items in a single variable.

A set is a collection which is *unordered*, *unchangeable*\*, and *unindexed*.

*** Note:** Set *items* are unchangeable, but you can remove items and add new items.

#### Dictionaries

##### Dictionary Methods

- `setdefault()`: Returns the value of the specified key. If the key does not exist: insert the key, with the specified value.

### Build-in Function

#### map

The `map()` function executes a specified function for each item in an iterable. The item is sent to the function as a parameter.

e.g.: use function to each elements in the list

```python
list(map(lambda x: len(x), ['apple', 'banana', 'cherry']))
# [5, 6, 6]
```

