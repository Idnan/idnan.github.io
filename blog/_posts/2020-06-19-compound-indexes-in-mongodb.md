---
layout: post
title: Compound Indexes in MongoDB
comments: true
---

Indexes are very important to databases, and mongodb is no different. Having proper indexes makes your queries more efficient. In this article we will discuss how to write more efficient indexes and how to check and verify if your queries are using proper indexes or not.

An index that refers to multiple fields is called a compound index. Consider a collection **users** with fields **firstName** and **lastName**. And here's our query that we are running on the **users** collection:

```
db.getCollection("users").find({ firstName: "john", lastName: "doe" })
```

Since there are two fields in our query, we need to create a compound index. We can create a compound index using the below query:

```
db.getCollection("users").createIndex({ firstName: -1, lastName: -1 })
```

The value `-1` defines an index that orders items in the descending order. You can also assign `1` which defines the orders of items in ascending order. You can read more about it [in MongoDB docs](https://docs.mongodb.com/manual/core/index-compound/#sort-order)

Now we have the index but how can we make user that queries are using the proper index or not. We can do that by calling [explain](https://docs.mongodb.com/manual/reference/method/cursor.explain/) on our query; which instead of providing the results of query provides the execution plan:

```
db.getCollection("users").find({firstName: "john", lastName: "doe"}).explain()
```

Running the above command will result in following output. The field at `winningPlan.inputStage.indexName` would give us the index name used by the query if any.

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20200619/snap-1.png" alt=""/>
</figure>

One thing to note about the compound indexes is that MongoDB creates multiple indexes using the beginning subsets of the indexed fields. In simple words when we created the above index, it created two indexes in the background:
```javascript
{ firstName: -1 }
{ firstName: -1, lastName: -1 }

// If we had three fields in our index e.g. { firstName: -1, lastName: -1, email: -1 }
// Our indexed would have been the following
{ firstName: -1 }
{ firstName: -1, lastName: -1 }
{ firstName: -1, lastName: -1, email: -1 }
```

Which means that running the below query will also use the same index that we created above
```
db.getCollection("users").find({firstName: "john"}).explain()
```

However, if you run the following query it is not going to use any of the indexes.  
```
db.getCollection("users").find({lastName: "doe"}).explain()
```

Because our compound index did not create any index for the field `lastName`.

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20200619/snap-2.png" alt=""/>
</figure>

To fix this we will have to crate [a single field index](https://docs.mongodb.com/manual/core/index-single/)

```
db.getCollection("user").createIndex({lastName: -1})
```

And that wraps it up. Feel free to leave your feedback or questions in the comments section below.