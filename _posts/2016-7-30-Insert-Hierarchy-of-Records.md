---
layout: post
title: "Insert a Hierarchy of Records"
published: true
tags: "apex"
---

I was working on a feature where I needed to connect my company's Salesforce org with two other partners using the SOAP API. When I sent request to partners' services, I would receive back an XML which I would parse into a hierarchy of my wrapper classes, and then I needed to persist all of those objects into our database.

I was reading somewhere how to persist Parent-Child records at once when the parents' IDs are not available, and one of the solutions was by using the External Id field. I didn't felt like going this way, so I created a new class called `PersistHierarchy`. Basically, what this class will do is if you have a hierarchy that is N levels deep, it will insert this hierarchy with N insert statements by grouping nodes (records) at each hierarchy level and inserting them together. Since you can perform 150 DML statements during one transaction, that means that you can insert a hierarchy that is 150 levels deep using one call.

Let's see how to implement and use the `PersistHierarchy` class.

## PersistHierarchy implementation

{% highlight java tabsize=2 %}
public class PersistHierarchy {
	public static void persistAll(IPersistable parent, Id parentId) {
		parent.preparePersistSelf(parentId);
		insert parent.getDbObject();
		persistChildren(parent.getChildrenToPersist());
	}

	private static void persistChildren(List<IPersistable> children){
		//first, persist all children so their IDs get populated
		List<sObject> objectsToPersist = new List<sObject>();
		for(IPersistable child : children){
			sObject dbObject = child.getDbObject();
			if(dbObject != null){
				objectsToPersist.add(child.getDbObject());
			}
		}
		insert objectsToPersist;

		//now that childrens have their IDs, start preparing the grandchildrens
		List<IPersistable> grandChildren = new List<IPersistable>();
		for(IPersistable child : children){
			grandChildren.addAll(child.getChildrenToPersist());
		}

		if(grandChildren.size() != 0){
			persistChildren(grandChildren);
		}
	}
}
{% endhighlight %}

## IPersistable

As you saw above, `PersistHierarchy` is accepting objects that are implementing the `IPersistable` interface. You can see the definition of this interface below.

{% highlight java tabsize=2 %}
public interface IPersistable {
	void preparePersistSelf(Id parentId);
	sObject getDbObject();
	List<IPersistable> getChildrenToPersist();
}
{% endhighlight %}

Let's see an example of a few classes that are implementing this interface, and then we'll explain each method.

## Sample Implementation

{% highlight java tabsize=2 %}
public class ObjectA implements IPersistable {
   //define some object properties here
   public String prop1 = '';

   //Childrens
   List<ObjectB> someChildren;
   ObjectC anotherChild;

   //The Custom Salesforce Object for this wrapper
   DB_Obj_A dbObject;

   public void preparePersistSelf(Id parentId){
       this.dbObject = new DB_Obj_A();
       //this object is a root of our hierarchy, if it doesn't attaches to any
       //existing record in the system, it can ignore the 'parentId' that is passed
       //in this method
       this.dbObject.Parent_Id__c = parentId;

       //set some object properties here
       this.dbObject.Prop_1__c = this.prop1;
   }

   public List<IPersistable> getChildrenToPersist(){
       List<IPersistable> children = new List<IPersistable>();

       for(ObjectB o : someChildren){
           o.preparePersistSelf(this.dbObject.Id);
           children.add(o);
       }

       if(this.anotherChild != null) {
				  this.anotherChild.preparePersistSelf(this.dbObject.Id);
				  children.add(this.anotherChild);
			 }

       return children;
    }

    public sObject getDbObject(){
       return this.dbObject;
    }
}
{% endhighlight %}

{% highlight java tabsize=2 %}
public class ObjectB implements IPersistable {
   //define some object properties here
   public String propX = '';

   //Childrens
   List<ObjectC> someChildren;

   //The Custom Salesforce Object for this wrapper
   DB_Obj_B dbObject;

   public void preparePersistSelf(Id parentId){
       this.dbObject = new DB_Obj_B();
       this.dbObject.Parent_Id__c = parentId;

       //set some object properties here
       this.dbObject.Prop_X__c = this.propX;
   }

   public List<IPersistable> getChildrenToPersist(){
       List<IPersistable> children = new List<IPersistable>();

       for(ObjectC o : someChildren){
           o.preparePersistSelf(this.dbObject.Id);
           children.add(o);
       }

       return children;
    }

     public sObject getDbObject(){
         return this.dbObject;
     }
}
{% endhighlight %}

{% highlight java tabsize=2 %}
public class ObjectC implements IPersistable {
   //define some object properties here
   String propN = '';

   //The Custom Salesforce Object for this wrapper
   DB_Obj_C dbObject;

   public void preparePersistSelf(Id parentId){
       this.dbObject = new DB_Obj_C();
       this.dbObject.Parent_Id__c = parentId;

       //set some object properties here
       this.dbObject.Prop_N__c = this.propN;
   }

   public List<IPersistable> getChildrenToPersist(){
       //NOTE: this is a last node in a hierarchy, it doesn't have any children
       //so it should return an empty list of children
       List<IPersistable> children = new List<IPersistable>();
       return children;
    }

    public sObject getDbObject(){
      return this.dbObject;
    }
}
{% endhighlight %}

OK, so above we have a hierarchy that is 3 levels deep. The top of the hierarchy is the `ObjectA`, and it has 0 or more direct children of type `ObjectB` , and 0 or 1 child of type `ObjectC`. The `ObjectB` has 0 or more direct children of type `ObjectC`, and `ObjectC` doesn't have any more children under it.

Potentially, we can have many objects inside this hierarchy, but since this hierarchy is 3 levels deep, we know that our `PersistHierarchy` object will insert all of this records with just 3 INSERT statements. Here is how you persist this hierarchy of objects with a single call:

{% highlight java tabsize=2 %}
PersistHierarchy.persistAll('SOME_ID', objA);
{% endhighlight %}

Since the `ObjectA` is the root of our hierarchy, it can potentially have a parent that is some existing record within the Salesforce, in which case `SOME_ID` should be replaced with that record's ID. Otherwise, we can simply pass `null` instead, and handle it properly in `preparePersistSelf` method in `ObjectA` class.

Method explanations:
* `preparePersistSelf` - takes in the `parentId`, which it uses to assign a parent to the current record, thus creating the hierarchy, or Parent-Child connection.
* `getChildrenToPersist` - prepares all the children for persisting by calling the `preparePersistSelf` on each of them, and passing the `parentId`
* `getDbObject` - returns the `sObject` represented by this wrapper classes

## Conclusion

So that's how I handled the insertion of a dynamic hierarchy of records into my Salesforce org by making sure that each parent handles the preparation of its children.

If you have any questions or suggestions, feel free to let me know in the comments below.
