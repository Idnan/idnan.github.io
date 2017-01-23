---
layout: post
title: Submit Form to New Window/Tab
comments: true
---

## Submit Form to New Window/Tab

Most of you guys have often used `targe="_blank"` on anchor links (like given in the example) to open that link in new tab.
 
```html
<a href="#" target="_blank">link</a>
```

But what if you want to submit a form data to a link and open it in new tab/window??? So today i got that situation. So after searching on the internet i came up with a solution. Here's an example that explains it:

```html
<form action="#" method="post" target="_blank">
    ...
</form>
```

No need for Javascript, you just have to add a `target="_blank"` attribute in your form tag.