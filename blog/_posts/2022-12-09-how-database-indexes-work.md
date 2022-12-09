---
layout: post
title: How database indexes work
comments: true
---
A database index is a data structure that improves the speed of data retrieval operations on a database table. It does this by allowing the database software to quickly find and retrieve the required data from the table based on the values stored in the index.

One way to visualize how a database index works is to imagine a book with an index at the back. The index lists all of the important topics that are discussed in the book, along with the page numbers where each topic can be found. If you want to find information on a particular topic, you can simply look it up in the index, which will tell you the page number where that information is located. This allows you to quickly find the information you're looking for without having to search through the entire book.

In a similar way, a database index works by creating a separate data structure that contains a copy of the values from one or more columns in a database table. These values are then organized in a way that allows the database software to quickly search for and retrieve the required data.

For example, let's say we have a database table that contains a list of employees and their salary information, as shown below:

```
Employee ID  | Name   | Salary
-------------|--------|--------
1            | John   | $50,000
2            | Mary   | $60,000
3            | Jane   | $70,000
4            | Robert | $80,000
5            | Susan  | $90,000
```
To improve the performance of searches on this table, we could create an index on the `Salary` column. The index would contain a copy of the Salary values from each row in the table, organized in a specific way to allow for fast searching.

One way to organize the index would be to use a B-tree structure, as shown below:

```
          [$80,000]
          /        \
 [$50,000]           [$90,000]
         \           /
         [$60,000] [$70,000]
```

When searching for a particular value in a B-tree index, we start at the root node and compare the value being searched for to the value in the root node. If the value being searched for is less than the value in the root node, we continue the search in the left subtree; if it is greater, we continue the search in the right subtree. This process continues until the value being searched for is found, or until there are no more nodes to search, in which case the value is not in the table.

In above example, the index has been organized into a hierarchical tree structure, with each node in the tree representing a group of values. The top node in the tree, known as the root node, contains the value `$80,000`. The root node has two children: a left subtree containing the value `$50,000`, and a right subtree containing the value $90,000.

The left subtree, in turn, has two children: a leaf node containing the value `$60,000`, and an empty right subtree. The right subtree, on the other hand, has a single leaf node containing the value `$70,000`.

To search for a particular salary value in the index, we would start at the root node and compare the value being searched for to the value in the root node. If the value being searched for is less than the value in the root node, we would continue the search in the left subtree; if it is greater, we would continue the search in the right subtree. This process would continue until the value being searched for is found, or until there are no more nodes to search, in which case the value is not in the table.

For example, if we are searching for the salary of `$60,000`. We start at the root node, which contains the value `$80,000`. Since `$60,000` is less than `$80,000`, we continue the search in the left subtree.

In the left subtree, we find the node containing the value `$50,000`. Since `$60,000` is greater than `$50,000`, we continue the search in the right subtree. In the right subtree, we find the leaf node containing the value `$60,000`, which is the value we were searching for. At this point, the search is complete and we have found the value we were looking for.

In this example, the B-tree index allowed us to quickly and efficiently find the value we were searching for without having to search through the entire table. This is the key advantage of using a B-tree index - it allows for fast searches, even on large tables with millions of rows.

In addition to improving the speed of data retrieval operations, database indexes can also help to reduce the amount of disk space required to store a database table. This is because the index itself is typically stored in a separate area of disk storage, rather than being stored with the data in the table.

For example, let's say we have a database table with 100 million rows, and we want to create an index on the `Salary` column. If we were to store the index along with the data in the table, we would need to allocate additional disk space to store the index. This would increase the overall size of the table, and could potentially lead to disk space being wasted if the index and the data are not perfectly aligned.

On the other hand, if we store the index in a separate area of disk storage, we can avoid this problem. The index can be stored in a compact, efficient data structure that uses less disk space than the original data, and we can still access it quickly and efficiently. This allows us to reduce the amount of disk space required to store the table, while still benefiting from the performance improvements provided by the index.

In summary, database indexes are an important tool for improving the performance and efficiency of data retrieval operations on a database table. By creating an index on one or more columns in a table, we can speed up searches and reduce the amount of disk space required to store the table. Whether you're working with a small database or a large one, using indexes can help to make your database more efficient and effective.
