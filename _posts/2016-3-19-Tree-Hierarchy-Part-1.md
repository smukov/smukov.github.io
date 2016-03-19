---
layout: post
title: "Tree Hierarchy - Part 1"
published: true
tags: "apex visualforce component"
---

I'm creating a series that will show you how I created a custom tree hierarchy with generic components that are easily
reusable and allow you to create a hierarchy of various objects using different criteria and custom groupings.

### Technologies used

For this mini-project I used a set of `Apex` classes to build my hierarchy structure from Salesforce Records. I tried to make these classes
as generic as I could so that I can easily reuse them to create a different hierarchy tree, as I know that this is often a requirement in my company.

I used a single `Visualforce Component` that is recursively building a tree using `AngularJS`. A good thing about `Visualforce Component` is that you can easily re-use it anywhere by only passing different arguments or collections to be rendered. You can even use the same component multiple times within a single `Visualforce` page in order to show multiple trees, which was a requirement that I had.

### Why AngularJS?

`Visualforce Components` cannot call themselves recursively, and since I'm building a tree I needed a way to recursively go through code and `AngularJS` gave me this ability. I defined an `ng-template` that serves as a `tree node`. I'm then using the `ngRepeat` and `ngInclude` directives to iterate over the `ng-template` recursively and construct my tree hierarchy in the `DOM`.

### Class Hierarchy

[![Class Hierarchy]({{ site.baseurl }}/images/3-2016/19/TreeHierarchy.png)]({{ site.baseurl }}/images/3-2016/19/TreeHierarchy.png){:target="_blank"}

A quick explanation of the class hierarchy above:

* **ObjectHierarchyWrapper** - a wrapper class to store record details that I'm showing as a `tree node` later on. I'm also storing all
the child nodes as `Map<Id, ObjectHierarchyWrapper> ChildRecords` within this class.
* **ObjectHierarchyBuilder** - an abstract class that contains the main logic required to create a hierarchy out of a set of `Salesforce Records`.
    * **wrapRecord** - an abstract method that provides a custom wrapping mechanism for records. You will see later on that my code could have done without this method, but, in my opinion, leaving it in code will provide great flexibility for future extensions. I could be wrong, so I'll be glad to hear your opinion in comments.
    * **evaluateParent** - this is an abstract method that must be implemented in a child class in order to provide a correct logic that evaluates what makes one record a `parent` of another. This is required as different types of records are using different fields for this purpose.
* **OrderLineObjectHierarchyBuilder** and **OrderLineByPremiseHierarchyBuilder** - these are two concrete implementations of the `ObjectHierarchyBuilder` class that contain custom logic that extends the initial class. Both of these classes are creating specific hierarchies that are grouped in a different way.

### Visualforce Hierarchy

[![Visualforce Hierarchy]({{ site.baseurl }}/images/3-2016/19/VF_TreeHierarchy.png)]({{ site.baseurl }}/images/3-2016/19/VF_TreeHierarchy.png){:target="_blank"}

Above you can see that our final `Visualforce Page` will contain 1 or more `GenericHierarchyComponents`, and each component will represent a different tree. In my case I will display 2 hierarchies side by side. I've also listed the parameters that the component will take. I'll explain those parameters later on when I get to explaining the component in detail.

### End product

[![Final Hierarchy]({{ site.baseurl }}/images/3-2016/19/final_hierarchy.png)]({{ site.baseurl }}/images/3-2016/19/final_hierarchy.png){:target="_blank"}

In the image above you can see what will be the final end product of this post series (you can click on an image to see it larger).

### Conclusion

This was just a quick overview of the whole setup that I'll cover in great details over the coming weeks. Stay tuned for more.
