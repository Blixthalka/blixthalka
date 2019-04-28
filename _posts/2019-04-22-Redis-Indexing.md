---
layout: post
title: Creating An Index In Redis
---
Redis is an in-memory key/value database, it does not have any natural indexing capabilities.
Without an index a search on anything different than the key will be an expensive operation,
since all elements needs to be fetched and then filtered in the application code. However it is possible to create an index with some tricks.

Let's get started! First of we'll need some ordinary key/value pairs in Redis.

```
> SET person/8755BFAB '{"name": "John", "age":33}'
> SET person/696F4E5F '{"name": "Jane", "age":42}'
> SET person/6866740D '{"name": "Lily", "age":23}'
> SET person/1F6E199B '{"name": "Jack", "age":38}'
```

### Numeric Index

To be able to filter on the persons age, we will create a sorted set (or `ZSET` if we talk Redis lingo).
This set will contain the persons age as well as key.

```
> ZADD person.age 33 'person/8755BFAB'
> ZADD person.age 42 'person/696F4E5F'
> ZADD person.age 23 'person/6866740D'
> ZADD person.age 38 'person/1F6E199B'
```

Then we can find all keys of persons between age 32 and 35. 

```
> ZRANGEBYSCORE person.age 32 35
1) "person/8755BFAB"
```

And we'll just look up the key to find the actual person.

```
> get person/8755BFAB
"{\"name\": \"John\", \"age\":33}"
```

### Text Index
Now we want to find persons by their name. 
Thus we need to add an index for the name also. 
This index is also implemented by using the sorted set,
 however it gets a little bit trickier than the numeric index. 

To start we create a sorted set named `person.name` which we insert the name of the 
person and their id with a chosen delimiter (I chose `@`).

```
> ZADD person.name 0 'John@person/8755BFAB'
> ZADD person.name 0 'Jane@person/696F4E5F'
> ZADD person.name 0 'Lily@person/6866740D'
> ZADD person.name 0 'Jack@person/1F6E199B'
```

Now we can search for whole names with `ZRANGEBYLEX`. 
 
```
> ZRANGEBYLEX person.name [Jack [Jack~
1) "Jack@person/1F6E199B"
```

We enter a minimum string and a maximum string as search values.
 Note that the maximum string ends with `~` it does this because `~` has the highest value in the ascii table.
Since it has the highest value all other values are less and the string that we are comparing to will
 become within the search interval and thus returned.

When we get the return value we have to parse the string on the delimiter, extract the id and make a separate query for that id. 

```
> GET person/1F6E199B
"{\"name\": \"Jack\", \"age\":38}"
```

Also all the persons that start their name with J can be found.

```
> ZRANGEBYLEX person.name [J [J~
1) "Jack@person/1F6E199B"
2) "Jane@person/696F4E5F"
3) "John@person/8755BFAB"
```


## Transactions
You need to manually keep track of the index you are creating, 
it is a good practise to insert the writes in a transaction to make sure that both have been made or none have been made.
We don't to get in a place were only one of the writes went through, since the data will become inconsistent. 

To start a transaction use `MULTI` and then insert your writes and then execute the transaction with `EXEC`.
Like this.

```
> MULTI
OK
> SET person/696F4E5F '{"name": "Jane", "age":42}'
QUEUED
> ZADD person.age 42 'person/696F4E5F'
QUEUED
> EXEC
OK
```

## Last words
I have shown you that it is possible to create indexes with Redis.
However I would not recommend doing it.
Instead I would reevaluate the choice of Redis as a databse and maybe choose another one with these features built in. 

Over and out.
/Emil