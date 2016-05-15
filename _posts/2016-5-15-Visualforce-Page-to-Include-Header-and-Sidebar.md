---
layout: post
title: "Make a Managed Visualforce Page show Header and Sidebar"
published: true
tags: "managed visualforce"
---

Let's say you've installed some managed package to your org and there's a visualforce page in this package that doesn't include *header* and *sidebar* . You need to redirect your users to this visualforce page, but the page breaks the look and feel by hiding the Salesforce header and sidebar. You can't access or change the managed package's code, so what else can you do?

Try creating a new visualforce page that will host the managed page within an `iframe`. Your new page will make sure that header and sidebar are displayed.

{% highlight html %}
<apex:page >
  <apex:iframe src="URL_TO_YOUR_MANAGED_PAGE"/>
</apex:page>
{% endhighlight %}

If you need to send any parameters to the managed page, simply change the `src` to include the required parameters. That's it, I hope this helps someone.
