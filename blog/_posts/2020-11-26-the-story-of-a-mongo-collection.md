---
layout: post
title: The Story of a Mongo Collection
comments: true
---
In my last company, we were having more than 70 small APIs and we were using MongoDB heavily. Apart from having ELK stack, we also had database level logging for all the incoming requests to some of the endpoints to track the requests and responses.

A few months ago, we were struck with a disaster, because of something foolish that was missed in the code reviews. One night we did a deployment and after doing the sanity checks on production, considering everything is fine, we all went home thinking that all is good.

<figure align="center"> 
    <img src="{{ site.baseurl }}/img/20201126/allgood.jpeg" style="max-width: 490px; height: auto; margin-left: auto; margin-right: auto; display: block;" alt=""/>
</figure>

After a few hours, I got a call from our DevOps team the website is down. After some digging, we found out that MongoDB is causing the issue; there is some newly created collection having huge amount of data.

I checked our codebase and it turned out that someone enabled the database level request logging to MongoDB for the "autocomplete" API call in search. So imagine 1500 users sitting on the website, searching for flights and each keystroke in the search is getting logged to the database.

The size of the collection had grown to around 600 GB and there was no space left on the server to store more data and the website went down. The quickest hack was to put the capping of 1 GB on the collection until the issue is resolved; we did that and the website was live again.

To avoid this issue in the future, and also ensuring indexes on the collection, apart from making our code review process stricter, we decided to build a cli tool to help us manage the MongoDB indexes, capping and other constraints. The tool was responsible for the below listed items:

* Notify against any newly created collections that are not registered against the API in the cli tool.
* Notify if the size of a collection increases the specified size.
* Notify if the collection doesn't have any index or cap on it.
* Show the list of indexes currently applied to the collection.

We built and deployed this to production and the issue never happened again. We were getting notified of any new collections which were not registered, keeping an eye on the indexes and sizes of the collections.

I hope that you liked the article. Feel free to leave your questions and comments below.
