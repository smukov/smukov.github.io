---
layout: post
title: "Time-Dependent Workflow that triggers in less than 1 hour"
published: true
tags: "workflow"
---

As you probably know, Salesforce `Time-Based Triggers` can trigger before (or after) certain number of days (or hours) from a specified `Date/Time` field on the record that triggered the `workflow`. The valid range is 0 to 999 days or hours, and you can't use decimal values, only integers. This means that out of the box you can't, for example, make a workflow be triggered `15 minutes` or `half an hour` after the rule was evaulated. However, in case you really want to do something like this, there is a workaround.

### Step 1: Create a new custom Formula field

Let's say that, for some reason, you want to create a task 15 minutes after an Opportunity record was updated. First, you'll need to create a new `custom formula field` that evaluates to a `Date/Time value`. We are going to call this new field `Record Edited Time Minus 45`, and set its formula to `LastModifiedDate - 45 * 0.00069`.

> In a `Date/Time` formula field `0.00069` represents 1 minute. So, this part of our formula `45 * 0.00069` actually represents 45 minutes.

As for security, make this field accessible to all users, but don't place it on any layout.

After we save our new formula field, every time we edit an Opportunity record, this field will hold a `Date/Time` value that is 45 minutes in the past from the time the record was edited.

### Step 2: Make your Time-Based Workflow use the new formula field

Now that you have created the required formula field you can go ahead and start setting up a `workflow` with a `Time-Dependent Action` that should fire 15 minutes after an Opportunity record was edited.

I'm not going to cover here how to create a `Workflow`, you should already know this, just keep in mind when creating a new `Time-Dependent Action` to set it to fire 1 hour after the `Opportunity: Record Edited Time Minus 45`, as shown in the image below.

[![workflow_formula]({{ site.baseurl }}/images/3-2016/13/Formula_2.png)]({{ site.baseurl }}/images/3-2016/13/Formula_2.png){:target="_blank"}

The last step should be to acivate your `workflow` and test it out.

### Conclusion

Now you know how to create a `Time-Dependent Workflow Rule` that will trigger in less than 1 hour after the record that triggered the workflow is edited and the workflow is evaluated.

Don't forget to add your comments below.

> I should point out that `Time-Based Worfklow Actions` are batched together and will be triggered within 1 hour of their designated time. This means that even if we set that an action should be executed 15 minutes after the record was edited, this still doesn't guarantee that it will be executed **exactly** after 15 minutes.
