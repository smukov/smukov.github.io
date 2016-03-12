---
layout: post
title: "Run Time-Dependent Workflow on every record edit"
published: true
tags: "workflow"
---

If you ever wanted to create a Time-Dependent workflow rule that will be triggered every time a record is edited,
your first instinct was probably to create a rule that is evaluated when a record is `created, and every time it's edited`.
However, as the image below shows, `you cannot add time-dependent workflow actions with this option`.

![evaulate_rule_message]({{ site.baseurl }}/images/3-2016/evaluate_rule_message.png)

This post will show you how you can overcome this limitation.
