---
layout: post
title: "Run Time-Dependent Workflow on every record edit"
published: true
tags: "workflow"
---

If you ever wanted to create a Time-Dependent workflow rule that will be triggered _every_ time a record is edited,
your first instinct was probably to create a rule that is evaluated when a record is `created, and every time it's edited`.
However, as the image below shows, `you cannot add time-dependent workflow actions with this option`.

[![evaulate_rule_message]({{ site.baseurl }}/images/3-2016/evaluate_rule_message.png)]({{ site.baseurl }}/images/3-2016/evaluate_rule_message.png){:target="_blank"}

### How to overcome this limitation

Let's say you want to create a `Workflow Rule` that will notify an agent if an Opportunity hasn't been updated
for 7 days.

To do this, start by creating a rule on Opportunity object. Specify the `Rule Name` and select the `Evaluation Criteria` to
evaluate the rule when a record is `created, and any time it's edited to subsequently meet criteria`.

Next, for `Rule Criteria` select `Opportunity: Created Date` `not equal to` `BLANK`, as in the image below. By setting this criteria you are making
sure that the workflow rule will be triggered every time since `Created Date` field will never be `blank`.

[![workflow_definition]({{ site.baseurl }}/images/3-2016/workflow.png)]({{ site.baseurl }}/images/3-2016/workflow.png){:target="_blank"}

Finally, specify a `Time-Dependent Workflow Action` that is triggered 7 days after `Opportunity: Last Modified Date` and create a `Task` for the created action that will notify a specified user. Once you active your rule its `Workflow Actions` section should look similar to the image below.

[![workflow_actions]({{ site.baseurl }}/images/3-2016/workflow-actions.png)]({{ site.baseurl }}/images/3-2016/workflow-actions.png){:target="_blank"}

### Conclusion

Now you have a `Time-Dependent Workflow` that will create a `Time-Dependent Action` every time you create or edit an Opportunity record.

That's it for my first ever blog post. If you have any suggestions or questions feel free to comment below.
