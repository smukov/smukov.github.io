---
layout: post
title: "apex:commandButton with Return Value"
published: true
tags: "apex visualforce"
---

Below you can see a simple code that can be used to return a value from an Apex controller to a Visualforce page when calling a controller button from an `apex:commandButton`. This approach doesn't require you to use `@RemoteAction` (i.e. static methods), so you doesn't lose the stateful nature of your page.

Visualforce Page:
{% highlight html %}
<apex:page controller="ReturnController">

    <apex:form >
        <apex:commandButton action="{!callMe}"
                    oncomplete="processReturnedValue({!valueReturned}); "
                    value="Click Me!"/>
    </apex:form>

    <script>
        var processReturnedValue = function(valueReturned){
            alert('Returned Value: ' + valueReturned);
        };
    </script>

</apex:page>
{% endhighlight %}

Controller:
{% highlight java %}
public class ReturnController {

    public Integer ValueReturned {get; set;}

    public ReturnController(){
        this.ValueReturned = 0;
    }

    public void callMe(){
        this.valueReturned++;
    }

}
{% endhighlight %}

That's it. Try it, it works. Let me know in the comments if there are any drawbacks to this approach. I'm always open to constructive critique.
