---
title: In-depth analysis of Java-HashMap
date: 2020-12-03 12:14:18
tags: 
	- Java
	- HashMap
	- Data structures
categories: ['Data structures']
id: Java-HashMap
---
# Introduction

---
Hash table is a very important data structure with a rich set of applications. The core of many caching techniques (such as memcached) is actually maintaining a large hash table in memory. This article will explain the principles of HashMap implementation in the java collection framework and analyse the JDK7 HashMap source code.

<!-- more -->

# 1. What is HashMap

Before discussing the hash table, we firstly have an overview of the performance of other data structures in performing basic operations such as adding, searching, etc.

**Array:** Array uses a continuous segment of memory cells to store data. For a given subscript search,  the time complex is $O(1)$; For a search by a given value, it is necessary to traverse the array, comparing the given keywords and array elements one by one, with a time complex of $O(n)$. Of course, for ordered arrays, the dichotomous search, interpolation search, Fibonacci search, etc. can be used, increasing the complexity of the search for $O(\log n)$; For the general insertion and deletion operations, involving the movement of array elements, the average complexity is also $O(n)$.

**Linear chain table:** For the addition, deletion, etc. of a link table (after the location of the specified operation has been found), only the references between the nodes need to be processed, which has a time complexity of $O(1)$, while the search operation needs to traverse the link table one by one for comparison, which has a complexity of $O(n)$.

**Binary tree:** The insertion, search and deletion operations carried out on a relatively balanced, ordered binary tree have an average complexity of $O(\log n)$.

**Hash tables:** Compared to the data structures above, adding, deleting, searching, etc. in a hash table has very high performance and can be done in just one location, regardless of hash conflicts (which can be discussed later), with a time complexity of $O(1)$. I will show how the hash table is achieved to reach the stunning constant order complexity of $O(1)$.

As we know, there are only two types of physical storage structures for data structures: sequential storage structures (like stacks, queues, trees, graphs, etc. that are abstracted from logical structures and mapped to memory, which are also two forms of physical organisation). As we mentioned above, finding an element in an array based on its subscript can be achieved in a single location. Hash tables make use of this property, the backbone of the hash table being the array.

For example, if we want to add or find an element, we can do this by mapping the current element's keyword to a location in the array via a  function that locates the array's subscript once.

This function can be described simply as: store location = $f(keywords)$, this function $f$ is generally known as the hash function, and the design of this function will have a direct impact on the performance of the hash table. As an example, if we want to perform an insert operation in a hash table, whose process is shown in the figure below:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/04/DqKdIJ.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Insertion operation</div>
</center>

The same principle applies to the search operation, where the actual storage address is calculated via the hash function and then the corresponding address is retrieved from the array.

**Hash conflict:**

However, nothing is perfect, so what if two different elements have the same actual storage address via a hash function? In other words, when we perform a hash operation on an element, get a storage address and then, when we want to insert it, we find that it is already occupied by another element. This is in fact a so-called hash conflict, also known as a hash collision. As we have mentioned before, the design of the hash function is crucial. A good hash function will ensure that the calculation is as simple as possible and that the hash address is evenly distributed. So how are hash conflicts resolved? There are various solutions to hash conflicts: the open address method (in the event of a conflict, the search continues for the next unoccupied storage address), the re-hash function method, the chain address method, and HashMap uses the chain address method, which is an array + chain table method.

# 2. Implementation principle of HashMap

The backbone of a HashMap is an array of Entries, which are the basic building blocks of a HashMap, each of those containing a key-value pair. (A Map is actually a collection that holds the mapping between two objects.)

```java
//The backbone array of a HashMap can be seen as an Entry array 
//with an initial value of an empty array {}, the length of which must be a power of 2
//The reasons why this was done are analysed in detail later.
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```

Entry is a static internal class in a HashMap. The code is as follows:

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;	//Storing references to the next Entry, single-chain table structure
    int hash;	//The hash code value of the key is hashed and stored in the Entry to avoid double counting.

    /**
    * Creates new entry.
    */
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
}
```

## overall structure

The overall structure of a HashMap is therefore as follows:

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/04/DqG000.png" width='80%'>
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Overall structure of a HashMap</div>
</center>

Put it simply, a HashMap consists of an array + a linked table. The array is the main body of the HashMap, while the linked table exists mainly to resolve hash conflicts. For the search  operation, the link table still needs to be traversed, and then the key object's equals method can be used to find the link table one by one. Therefore, in terms of performance, the less the link table appears in a HashMap, the better the performance will be.

## A few other important fields

```java
/* Number of key-value pairs actually stored */
transient int size;

/** Threshold value, which is the initial capacity when table == {} 
* (the default initial capacity is 16); 
* when table is filled, i.e. when memory space has been allocated to it.
* HashMap needs to refer to the threshold when scaling, 
* which is discussed in more detail later
*/.
int threshold;

/** load factor, which represents how much the table is filled, the default is 0.75
* The reason for the load factor is also to mitigate hash conflicts, 
* because if the initial bucket is 16 and you wait until it is full of 16 elements before expanding, 
* some buckets may have more than one element in them.
* So the default loading factor is 0.75, 
* which means that a HashMap of size 16 will expand to 32 by the 13th element.
*/
final float loadFactor;

/** The number of times the HashMap has been changed, 
* due to the non-threaded safety of the HashMap, when iterating on the HashMap
* If the structure of the HashMap has changed during the period due to the involvement of other threads (e.g. put, remove, etc.).
* A ConcurrentModificationException needs to be thrown
*/
transient int modCount;
```

##  constructors

HashMap has four constructors, the other constructors use default values if the user does not pass in the *initialCapacity* and *loadFactor* parameters.

The default *initialCapacity* is 16 and the default *loadFactory* is 0.75.

Let's take a look at one of these:

```java
public HashMap(int initialCapacity, float loadFactor) {
	//Here the incoming initial capacity is verified 
    //and cannot exceed a maximum of MAXIMUM_CAPACITY = 1<<30(230)
	if (initialCapacity < 0)
	throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
	if (initialCapacity > MAXIMUM_CAPACITY)
		initialCapacity = MAXIMUM_CAPACITY;
	if (loadFactor <= 0 || Float.isNaN(loadFactor))
		throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

	this.loadFactor = loadFactor;
	threshold = initialCapacity;
　　　　　
	init();//The init method is not actually implemented in the HashMap, 
    	   //but is implemented in its subclasses such as linkedHashMap.
}

```

## put operation

Let's now look at the implementation of the put operation:

```java
public V put(K key, V value) {
	//If the table array is empty {}, 
    //the array is filled (allocating actual memory space to the table),
    //with threshold as the input reference.
	//In this case the threshold is the initialCapacity.<default is 1<<4 (24=16)>
	if (table == EMPTY_TABLE) {
		inflateTable(threshold);
	}
   //If the key is null, the storage location is table[0] or on the conflicting chain of table[0].
	if (key == null)
		return putForNullKey(value);
	int hash = hash(key);//The hashcode of the key is further calculated to ensure an even hash.
	int i = indexFor(hash, table.length);//Get the actual position in the table
	for (Entry<K,V> e = table[i]; e != null; e = e.next) {
	//If this corresponding data already exists, 
    //an overwrite is performed. Replace the old value with the new value and return the old value
		Object k;
		if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
			V oldValue = e.value;
			e.value = value;
			e.recordAccess(this);
			return oldValue;
		}
	}
	modCount++;	//Guaranteed failure of rapid response if the internal structure of the HashMap changes during concurrent accesses
	addEntry(hash, key, value, i);//Adding an entry
	return null;
}
```

The *inflateTable* method is used to allocate storage space in memory for the master array table. By *roundUpToPowerOf2* (toSize) you can ensure that the capacity is greater than or equal to the nearest power of toSize, e.g. toSize=13, then capacity=16;to_size=16,capacity=16;to_size=17,capacity=32.

```java
private void inflateTable(int toSize) {
	int capacity = roundUpToPowerOf2(toSize);	//capacity must be a power of 2
	/**Here is the threshold assignment, taking the minimum value of capacity*loadFactor and MAXIMUM_CAPACITY+1.
	* The capaticy must not exceed MAXIMUM_CAPACITY, unless the loadFactor is greater than 1.
	*/
	threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
	table = new Entry[capacity];
	initHashSeedAsNeeded(capacity);
}
```

The processing in *roundUpToPowerOf2* is such that the length of the array must be a power of 2. *Integer.highestOneBit* is used to get the value represented by the leftmost bit (the other bits are 0).

```java
private static int roundUpToPowerOf2(int number) {
	// assert number >= 0 : "number must be non-negative";
	return number >= MAXIMUM_CAPACITY
			? MAXIMUM_CAPACITY
			: (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}
```

**Hash function**

```java
/**This is a magic function that uses a lot of isomorphisms, transpositions, etc.
* Further calculations of the key's hashcode and adjustment of the binary bits ensure that 
* the resulting storage locations are as evenly distributed as possible.
*/
final int hash(Object k) {
	int h = hashSeed;
	if (0 != h && k instanceof String) {
		return sun.misc.Hashing.stringHash32((String) k);
	}

	h ^= k.hashCode();

	h ^= (h >>> 20) ^ (h >>> 12);
	return h ^ (h >>> 7) ^ (h >>> 4);
}
```

The value calculated by the above hash function is further processed by indexFor to obtain the actual storage location.

```java
/**
 * return the index of array
 */
static int indexFor(int h, int length) {
	return h & (length-1);
}
```

h&(length-1) ensures that the index obtained is within the array, for example, the default capacity is 16, length-1=15, h=18, which is converted to binary as index=2. Bit operations are much higher performance for computers (there are a lot of bit operations in HashMap).

So the process for determining the final storage location is as follows.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/05/DOrEhd.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">process for determining</div>
</center>

A further look at the implementation of addEntry:

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
	if ((size >= threshold) && (null != table[bucketIndex])) {
		resize(2 * table.length);	//Scaling when size exceeds the critical threshold and a hash conflict is imminent
		hash = (null != key) ? hash(key) : 0;
		bucketIndex = indexFor(hash, table.length);
	}

	createEntry(hash, key, value, bucketIndex);
}
```

We can learn from the code above that when a hash conflict occurs and the size is greater than the threshold, we need to expand the array. When expanding, we need to create a new array twice the length of the previous one, and the transfer all elements of the current Entry array. The new array is twice the length of the previous after expansion, so expansion is relatively resource-consuming operation.

# 3. Why must the array length of a HashMap be the power of 2

Let's continue analysis the resize method above:

```java
void resize(int newCapacity) {
	Entry[] oldTable = table;
	int oldCapacity = oldTable.length;
	if (oldCapacity == MAXIMUM_CAPACITY) {
		threshold = Integer.MAX_VALUE;
		return;
	}

	Entry[] newTable = new Entry[newCapacity];
	transfer(newTable, initHashSeedAsNeeded(newCapacity));
	table = newTable;
	threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

If the array is expanded, the length of the array changes and the storage location index = h&(length-1),index may also change and the index needs to be recalculated, let's look at the transfer method first.

```java
void transfer(Entry[] newTable, boolean rehash) {
	int newCapacity = newTable.length;
    //The code in the for loop goes through the chain table one by one, 
    //recalculating the index positions and copying the old array data into the new array 
    //(the array does not store the actual data, so it's just a copy reference)
	for (Entry<K,V> e : table) {
		while(null != e) {
			Entry<K,V> next = e.next;
			if (rehash) {
				e.hash = null == e.key ? 0 : hash(e.key);
			}
			int i = indexFor(e.hash, newCapacity);
			//Point the next chain of the current entry to the new index position,
            //newTable[i] may be empty, or it may also be an entry chain,
            //in which case it is inserted directly at the head of the chain table.
			e.next = newTable[i];
			newTable[i] = e;
			e = next;
		}
	}
}
```

This method takes the data from the old array and traverses it one by one, chain by chain, throwing it into the new expanded array. The array index position is calculated by hash scrambling the key value hashcode and then performing a bitwise operation with length-1 to obtain the final array index position.

For example, if the binary representation of 16 is 10000, then length-1 is 15, and the binary is 01111, similarly the expanded array length is 32, binary is 100000, length-1 is 31, and the binary is 011111. It is guaranteed that the low bit is all 1, while the expansion has only one difference, i.e. an extra leftmost 1, so that when passing h&(length-1), as long as the leftmost difference bit corresponding to h is 0, the new array index is guaranteed to be the same as the old one (greatly reducing the need to re-swap the data position of the old array which has been well hashed before), with personal understanding.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/06/DXzISg.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">length of HashMap array</div>
</center>

Also, keeping the length of the array to the power of 2, with the lower bits of length-1 all being 1, results in a more uniform index of the obtained array.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/06/DjS79O.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">length of HashMap array</div>
</center>

As we can see, with the & operation above, the higher bits have no effect on the result (the hash function uses a variety of bit operations, possibly also to make the lower bits more hashed), we are only concerned with the lower bits, and if the lower bits are all 1, then for the lower part of h, any change in one bit will have an effect on the result, i.e. to get to the storage position index=21, h's This is the only combination of the lower positions. This is the reason why the array length is designed so that it must be a power of 2.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/06/Dj9JyQ.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">length of HashMap array</div>
</center>

If it is not the power of 2, i.e. the low bit is not all 1, then, to make index = 21, the low part of h is no longer unique and the chance of a hash conflict becomes even greater. At the same time, the bit corresponding to the index will not be equal to 1 in any case, and the corresponding array positions will be wasted.

## get method

```java
 public V get(Object key) {
　　//If the key is null, then just go to table[0] to retrieve it.
	if (key == null)
		return getForNullKey();
	Entry<K,V> entry = getEntry(key);
	return null == entry ? null : entry.getValue();
}
```

The get method returns the corresponding value by means of a key, and if the key is null, it goes directly to table[0]. Let's look at the getEntry method again

```java
final Entry<K,V> getEntry(Object key) {
            
	if (size == 0) {
		return null;
	}
	//Calculation of the hash value via the key's hashcode value
	int hash = (key == null) ? 0 : hash(key);
	//indexFor (hash&length-1) gets the final array index and then traverses the chain table 
    //to find the corresponding records by matching them with the equals method
	for (Entry<K,V> e = table[indexFor(hash, table.length)];
		 e != null;
		 e = e.next) {
		Object k;
		if (e.hash == hash && 
			((k = e.key) == key || (key != null && key.equals(k))))
			return e;
	}
	return null;
}    
```

As can be seen, the get method is relatively simple to implement, key(hashcode)->hash->indexFor->final index position, find the corresponding position table[i], check if there is a link table, traverse the link table, and pass the key's The equals method compares and finds the corresponding record. It is important to note that some people feel that the above judgement of e.hash == hash is not necessary when traversing the chain table after locating the array position and then using only the equals judgement. This is not the case, think about it, if the incoming key object overrides the equals method but does not override the hashCode, and it happens that this object is positioned in this array, if only using equals may be equal, but its hashCode is not consistent with the current object, in this case, according to the conventions of Object's hashCode, can not return Instead of the current object, null should be returned, as will be further explained in the examples that follow.

# 4. Overriding the equals method requires overriding the hashCode method.

Finally we come to the age-old question, which is mentioned in various sources, "overwrite the hashcode as well as the equals". Let's take a small example to see what happens if we override equals without overriding the hashcode.

```java
public class MyTest {
    private static class Person{
        int idCard;
        String name;

        public Person(int idCard, String name) {
            this.idCard = idCard;
            this.name = name;
        }
        @Override
        public boolean equals(Object o) {
            if (this == o) {
                return true;
            }
            if (o == null || getClass() != o.getClass()){
                return false;
            }
            Person person = (Person) o;
            //两个对象是否等值，通过idCard来确定
            return this.idCard == person.idCard;
        }

    }
    public static void main(String []args){
        HashMap<Person,String> map = new HashMap<Person, String>();
        Person person = new Person(1234,"乔峰");
        //put到hashmap中去
        map.put(person,"天龙八部");
        //get取出，从逻辑上讲应该能输出“天龙八部”
        System.out.println("结果:"+map.get(new Person(1234,"萧峰")));
    }
}
```

    实际输出结果：null

This result is easy to understand if we already have some understanding of the principles of HashMap. Although the keys we use for the get and put operations are logically equivalent (equal by comparison with equals), there is no overriding of the hashCode method, so that for the put operation, key(hashcode1)->hash-> indexFor->final index position. Whereas when retrieving values by key key(hashcode1)->hash->indexFor->final index position, as hashcode1 is not equal to hashcode2, an array of positions is not located and the value is not retrieved. The logically incorrect value null is returned (it is also possible to happen to locate an array of positions. But it also determines whether its entry hash value is equal, as mentioned in the get method above).

So, when rewriting the methods of equals, care must be taken to rewrite the hashCode method. It is also important to ensure that calls to the hashCode method return the same integer value for two objects that are judged to be equal by equals. If the equals judge two objects that are not equal, the hashCode can be the same (hash conflicts will occur and should be avoided).

# 5. Performance optimisation of HashMap in JDK 1.8

What if there is too much data on an array of slots on the chain (i.e. the zip is too long) causing a drop in performance?
JDK1.8 has added a red and black tree on the basis of JDK1.7 to optimise the HashMap. That is to say, when the chain table exceeds 8, the chain table will be converted into a red and black tree to improve the performance of HashMap by taking advantage of the rapid addition, deletion and rechecking of the red and black tree, in which the insertion, deletion and search algorithms of the red and black tree will be used.

# Attachment: HashMap put method logic diagram (JDK 1.8)

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/12/06/DjEVhR.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">logic diagram of put method</div>
</center>