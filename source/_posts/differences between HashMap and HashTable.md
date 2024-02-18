---
title: Differences between HashMap and HashTable
date: 2020-12-06 19:31:25
tags: 
	- HashMap
	- HashTable
	- Data structures
	- Java
categories: ['Data structures']
id: HashMap-HashTable
---
The difference between a HashMap and a HashTable is a common question asked by interviewers in interviews. You can choose to focus your answer on the following main differences:

- Different value ranges for key and value
- Threading safety
- Efficiency and synchronisation
- Selection and use

The details will be discussed in the following part of this article.

<!-- more -->

# Different value ranges for key and value

Both HashMap and HashTable are tool classes that implement key-value mapping based on hash tables, with an underlying hash table structure. 

HashMap allows a key to be null and a value to be null. For HashMap, if you use the get method to return null, it does not indicate that the key does not exist in the HashMap, it is possible that the value corresponding to the key is null.

The HashTable does not allow null keys and null values.

# Threading safety

HashMap is non-synchronized, whereas HashTable is a synchronized.

> synchronized is a keyword in Java language that can be used to lock objects and methods or blocks of code. When it locks a method or block of code, at most one thread executes the code at any one time. 

This means that a HashTable is thread-safe, whereas a HashMap is not.

# Efficiency and synchronisation

HashMap is asynchronous and efficient, HashTable is synchronous and inefficient.
Although HashMap is not thread-safe, it will be much more efficient than HashTable. It makes sense to design it this way. Most of our daily use is single-threaded, and HashMap frees up this part of the operation.
The ConcurrentHashMap is also thread-safe, but it is much more efficient than the HashTable, which can be used when multi-threaded operations are required.

# Selection and use

If you don't need thread-safe then use HashMap, if you need thread-safe then use ConcurrentHashMap, which is not only thread-safe, but also more efficient than HashTable.

**HashTable is almost obsolete.**