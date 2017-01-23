---
layout: post
title: MySQL Order By Value
comments: true
---

Today I ran into a problem. I need to sort results of a given query by column values. I have a column named `priority` with values `'Low', 'Normal', 'High'`.

When I used normal sort method like
`Order By priority ASC` it returned `'High', 'Low', 'Normal'` and by using `Order By status DESC` it returned `'Normal', 'Low', 'High'`

But i want result in this order `'High', 'Normal', 'Low'`. So after searching i came up with a solution.

```sql
ORDER BY
   CASE WHEN priority = 'High' THEN 0
        WHEN priority = 'Normal' THEN 1
        WHEN priority = 'Low' THEN 2
   END
```

Simply assign each value a number lower number will be shown first and the high number will be shown later. :)