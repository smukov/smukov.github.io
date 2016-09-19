---
layout: post
title: "Accessing Salesforce Record Fields with JavaScript"
published: true
tags: "javascript visualforce utility"
---

I wanted to share with you a utility function that I've created and stored in my `Static Resources`. I'm using this function to dynamically access fields from JavaScript objects that represent Salesforce records obtained by `sforce.connection.query`. Here's the function below:

{% highlight javascript %}
/**
 * Get the sub records field value if the API contains dots
 * @param  {[Object]} record [Record containing the field]
 * @param  {[String]} api    [Property path (e.g. - Order_Line__r.Order_Line_Premise__r.Id)]
 * @return {[Field_Value]}   [Null - if part of the path is not defined, otherwise it will return the value of the field]
 */
var getRecordField = function(record, api){
   var subRecords = api.split('.');
   var topLevelObj = record;
   for(var i = 0; i < subRecords.length - 1; i++){
       if(topLevelObj == null){
           return null;
       }
       topLevelObj = topLevelObj[subRecords[i]];
   }

   if(topLevelObj == null || topLevelObj[subRecords[subRecords.length-1]] == null){
       return null;
   }else{
       return topLevelObj[subRecords[subRecords.length-1]];
   }
};
{% endhighlight %}

Sample use:

{% highlight javascript %}

var test = {
  one_property: 'something',
  another_property : {
    nested_property : 'nested',
    nested_array : [
      {x : 'y'}, {x : 'z'}
    ]
  }
};

//get first level property
console.log(getRecordField(test, 'one_property'));

//get property of a nested object
console.log(getRecordField(test, 'another_property.nested_property'));

//you can even access an array objects
console.log(getRecordField(test, 'another_property.nested_array.1.x'));

{% endhighlight %}

I've found this method to be incredibly useful when I have to work with hierarchy of objects returned from Salesforce (e.g. - `record.Order_Line__r.Primary_Site__r.Street_Addess__c`). I hope that you'll find this useful as well.
