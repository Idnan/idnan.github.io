---
layout: post
title: Quick Tip - Deleting Remote tags
comments: true
---

Just like branches, deleting the tags locally does not delete it from the remote. If you ever want to do that, here is how to do that:

```
git tag -d v1.2
git push origin :refs/tags/v1.2
```

First command is deleting the tag `v1.2` from our local repository and the second one is pushing the refs to the remote. 
