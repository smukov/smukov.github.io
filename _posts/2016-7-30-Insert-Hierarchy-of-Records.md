---
layout: post
title: "Insert a Hierarchy of Records"
published: true
tags: "apex"
---

I was working on a feature where I had to connect my company's Salesforce org with two other partners using the SOAP API. When I send a request to partners' services, I receive back an XML which I parse into a hierarchy of my wrapper classes, and then I needed to persist all of those objects into our database.

I was reading [here](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/langCon_apex_dml_foreign_keys.htm) how to persist Parent-Child records at once when the parents' IDs are not available. The solution is to use the External Id field, however, I didn't felt like going this way, especially since with that approach you can only insert parent-child records that are of different sObject type. 

In order to solve this problem I created a new class called `PersistHierarchy`. Basically, if you have a hierarchy that is N levels deep, this class will insert the hierarchy with N insert statements by grouping nodes (records), at each hierarchy level, and will then insert them together. Since you can perform 150 DML statements during one transaction, that means that you can insert a hierarchy that is 150 levels deep using one call. Also, you are not limited to sObject types, you can have any number of different or same sObjects types at any hierarchy level.

Let's see how to implement and use the `PersistHierarchy` class.

## PersistHierarchy implementation

{% highlight java tabsize=4 %}
public class PersistHierarchy {
 public static void persistAll(IPersistable parent, Id parentId) {
  parent.preparePersistSelf(parentId);
  insert parent.getDbObject();
  persistChildren(parent.getChildrenToPersist());
 }

 private static void persistChildren(List < IPersistable > children) {
  //first, persist all children so their IDs get populated
  List < sObject > objectsToPersist = new List < sObject > ();
  for (IPersistable child: children) {
   sObject dbObject = child.getDbObject();
   if (dbObject != null) {
    objectsToPersist.add(child.getDbObject());
   }
  }
  insert objectsToPersist;

  //now that childrens have their IDs, start preparing the grandchildrens
  List < IPersistable > grandChildren = new List < IPersistable > ();
  for (IPersistable child: children) {
   grandChildren.addAll(child.getChildrenToPersist());
  }

  if (grandChildren.size() != 0) {
   persistChildren(grandChildren);
  }
 }
}
{% endhighlight %}

## IPersistable

As you saw above, `PersistHierarchy` is accepting objects that are implementing the `IPersistable` interface. You can see the definition of this interface below.

{% highlight java tabsize=4 %}
public interface IPersistable {
 void preparePersistSelf(Id parentId);
 List<IPersistable> getChildrenToPersist();
 sObject getDbObject();
}
{% endhighlight %}

Method explanations:

* `preparePersistSelf` - takes in the `parentId`, which it uses to assign a parent to the current record, thus creating the hierarchy, or Parent-Child connection.
* `getChildrenToPersist` - prepares all the children for persisting by calling the `preparePersistSelf` on each of them, and passing the `parentId`
* `getDbObject` - returns the `sObject` represented by this wrapper classes

Let's see an example of a few classes that are implementing this interface.

## Sample Implementation

{% highlight java tabsize=4 %}
public class ObjectAWrapper implements IPersistable {

 //A Custom or Standard Salesforce Object
 DB_Obj_A dbObject {get; set;}
 
 //Children
 List < ObjectAWrapper > childrenA;
 ObjectBWrapper childB;

 public void preparePersistSelf(Id parentId) {
  //all that you need to do in this method is to set the parentId to correct field
  
  //if the object is a root of the hierarchy, it doesn't need to  attach to any
  //existing record in the system, so it can potentially ignore the 
  //'parentId' that is passed in this method
  this.dbObject.Parent_Id__c = parentId;
 }

 public List < IPersistable > getChildrenToPersist() {
  //this method prepars the children for insert, by calling the
  //preparePersistSelf method on each one and providing the parentId
  
  List < IPersistable > children = new List < IPersistable > ();

  for (ObjectAWrapper o: childrenA) {
   o.preparePersistSelf(this.dbObject.Id);
   children.add(o);
  }

  if (this.childB != null) {
   this.childB.preparePersistSelf(this.dbObject.Id);
   children.add(this.childB);
  }

  return children;
 }

 public sObject getDbObject() {
  return this.dbObject;
 }
}
{% endhighlight %}

The 3 interface methods are similar for `ObjectBWrapper`.

{% highlight java tabsize=4 %}
public class ObjectBWrapper implements IPersistable {
 
 //A Custom or Standard Salesforce Object
 DB_Obj_B dbObject {get; set;}
 
 //Children
 List < ObjectBWrapper > childrenB;

 public void preparePersistSelf(Id parentId) {
  this.dbObject.Parent_Id__c = parentId;
 }

 public List < IPersistable > getChildrenToPersist() {
  List < IPersistable > children = new List < IPersistable > ();

  for (ObjectBWrapper o: childrenB) {
   o.preparePersistSelf(this.dbObject.Id);
   children.add(o);
  }

  return children;
 }

 public sObject getDbObject() {
  return this.dbObject;
 }
}
{% endhighlight %}


The above 2 wrapper classes can form a hierarchy that is N levels deep. The top of the hierarchy is the `ObjectAWrapper`, and it has 0 or more direct children of the same `ObjectAWrapper` type (property: `childrenA`), and 0 or 1 child of type `ObjectBWrapper` (property: `childB`). The `ObjectB` has 0 or more direct children of the same `ObjectBWrapper` type (property: `childrenB`).

Potentially, we can have many records inside this hierarchy and it can be N levels deep, however, we know that our `PersistHierarchy` class will insert all of these records with N INSERT statements (same as the hierarchy depth). Here is how you persist this hierarchy of objects with a single call:

{% highlight java tabsize=4 %}
PersistHierarchy.persistAll(objA, 'SOME_ID');
{% endhighlight %}

Since the `DB_Obj_A` is the root of our hierarchy, it can potentially have a parent that is some existing record within the Salesforce, in which case `SOME_ID` should be replaced with that record's ID. Otherwise, we can simply pass `null` instead, and handle it properly in `preparePersistSelf` method in `ObjectAWrapper` class.

## Conclusion

So that's how I handled the insertion of a dynamic hierarchy of records into my Salesforce org by making sure that each parent handles the preparation of its children.

If you have any questions or suggestions, feel free to let me know in the comments below.
