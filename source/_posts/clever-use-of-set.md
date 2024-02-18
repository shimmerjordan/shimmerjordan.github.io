---
title: Clever use of set
date: 2021-11-01 14:30:12
tags: 
	- Data structures
	- Python
	- Algorithm
categories: Data structures
id: clever-use-of-set
---

I noticed that we can use set to simplify algorithm in many problems, including [distribute candies](https://leetcode-cn.com/problems/distribute-candies) and [keyboard row](https://leetcode-cn.com/problems/keyboard-row/). Thus, I conclude some clever use of `set` in python and do some comparison between `list`.

<!--more-->

# 1. Introduction to `set`

We know that the core of data structure `dict`  in Python is to map relationships between a set of **key** and **value**, while **key** can not be repeated. `Set` is similar to `dict` to some degree that elements in `set` is duplicate. Additionally, these elements are disorder, which means each print of `set` might be of different order inside.  Same as the **key** in `dict`, elements in `set` must be **constant** instead of any variable that can be changed.

To create a `set`, we can call `set()` and pass elements in a list into it: `s = set(['A', 'B', 'C'])`

# 2. Methods & Properties

## Properties

- The set representation set is very similar to a list with differences:

  - Sets can only store immutable objects
  - The objects stored in a `set` are unordered
  - Sets cannot have duplicate elements

- We can use `{}` to create sets

- `Set()` can be used to convert `sequences `and `dictionaries` into `sets`.

- We can use `len()` to get the number of elements in a `set`

- We can use `add()` to add elements to a `set`

- `update()` adds elements from one `set` to another

- `pop()` removes an element from a set **at random**, usually the last element.

- `remove()` removes the specified element from the set

- `clear()` clears the set

## Examples:

```python
set_a={8,7,'pthon',True,8.8,7,7,7}  # elements in set cannot be changed
print('1.',set_a,type(set_a))    	# elements in set in disordered and duplicate
dict_a={'a1':'python','b2':'java','c':8}
set_b =set(dict_a)    				# we can use set() to transform sequence to set (transform key in dict)
print('2.',set_b,type(set_b))
```

## Operations on sets

```python
& 	# Intersection
| 	# Concurrency
- 	# Difference set operations
^ 	# Also or set

<= 		# Check if a set is a subset of another set
< 		# Check if a set is a true subset of another set
>= 		# Check if a set is a superset of another set
> 		# Check if a set is a true superset of another set
```

See  [keyboard row](https://leetcode-cn.com/problems/keyboard-row/) for example.