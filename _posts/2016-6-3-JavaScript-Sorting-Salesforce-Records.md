---
layout: post
title: "Sorting Salesforce Records with JavaScript"
published: true
tags: "javascript visualforce utility"
---

In a [previous post](http://smukov.github.io/blog/2016/06/03/JavaScript-Accessing-Salesforce-Record-Fields/) I've shared with you a JavaScript function that I'm using in order to easily access object's properties that go from 1 to n levels of depth. Now, I've decided to extend on that post, and show you how I'm using the same method to achieve the JavaScript array sorting using dynamic properties.

First of all, go to my [previous post](http://smukov.github.io/blog/2016/06/03/JavaScript-Accessing-Salesforce-Record-Fields/) to see how the `getRecordField` function is defined and used.

Next, let's see how the `advancedDynamicSort` function, that we'll use for array sorting, looks like:

{% highlight javascript %}
/**
 * Usage: array.sort(advancedDynamicSort('Property_Name.Sub_Property.Sub_Sub_Property'));
 * for Descending: array.sort(advancedDynamicSort('-Property_Name.Sub_Property.Sub_Sub_Property'));
 */
 var advancedDynamicSort = function(property) {
         var sortOrder = 1;
         if(property[0] === "-") {
             sortOrder = -1;
             property = property.substr(1);
         }
         return function (a,b) {
             var result = (getRecordField(a,property) < getRecordField(b,property)) ? -1 : (getRecordField(a,property) > getRecordField(b,property)) ? 1 : 0;
             return result * sortOrder;
         }
     };
{% endhighlight %}

Sample use:

{% highlight javascript %}

var test = [{
  one_property: 'something',
  another_property : {
    nested_property : 1,
    nested_array : [
      {x : 'y'}, {x : 'z'}
    ]
  }
},{
  one_property: 'something else',
  another_property : {
    nested_property : 3,
    nested_array : [
      {x : 'y'}, {x : 'a'}
    ]
  }
},{
  one_property: 'random',
  another_property : {
    nested_property : 2,
    nested_array : [
      {x : 'y'}, {x : 's'}
    ]
  }
}];

//sort by first level property
test.sort(advancedDynamicSort('one_property'));

//sort by property of a nested object
test.sort(advancedDynamicSort('another_property.nested_property'));

//you can even access an array objects, and sort descending by
//prepending minus (-) in front of the property string
test.sort(advancedDynamicSort('-another_property.nested_array.1.x'));

{% endhighlight %}

I have to give credit to the original creator of the `dynamicSort` function, Ege Özcan, and his [StackOverflow answer](http://stackoverflow.com/a/4760279/634951). All I did in the end was adjusting his method to work with my function that dynamically reads object's properties in order to make the sorter even more flexible.

That's it, I hope you'll find this useful, and if you have any suggestions or critique, feel free to drop me a line in the comments below.
