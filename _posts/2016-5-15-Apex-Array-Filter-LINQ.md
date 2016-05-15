---
layout: post
title: "Array Filter"
published: true
tags: "apex"
---

Recently I needed a way to quickly filter out an array (i.e. list) of sObjects in Apex. Since I was a C# developer for 5 years, I immediately remembered how I used LINQ in C# for this purpose and I hoped there's something similar in Apex as well. However, in Apex, there is no easy way to filter through a list without using the `for` loop.

I'm usually trying to limit the number of DML statements in Apex, so I'd often query more records at once (if I know I'm not going to reach 50.000 records limit) and then filter through it via `for` loop in case I need to get multiple sets of records from that query. Because of this need, I decided to create a helper class that will help me achieve this without writing the `for` loop every time I need to filter something. I'm sharing the class with you below, and you can share your thoughts with me in the comments.

{% highlight java %}
public class ArrayFilter {

  public static List<sObject> filter(List<sObject> arrayToFilter, List<ArrayFilter.Filter> filters){
    List<sObject> retVal = new List<sObject>();

    for(Integer i = 0; i < arrayToFilter.size(); i++){
      for(Integer j = 0; j < filters.size(); j++){
        Filter f = filters[j];
        if(ArrayFilter.Assert(arrayToFilter[i], f)){
          retVal.add(arrayToFilter[i]);
          continue;
        }
      }
    }
    return retVal;
  }

  public static Boolean Assert(sObject o, Filter f){
    if(f.valueType == 'Boolean'){
      if(f.operator == '!='){
        return ((Boolean)o.get(f.fieldName)) != f.valBoolean;
      }else{
        return ((Boolean)o.get(f.fieldName)) == f.valBoolean;
      }
    }else if(f.valueType == 'Date'){
      if(f.operator == '!='){
        return ((Date)o.get(f.fieldName)) != f.valDate;
      }else if(f.operator == '>'){
        return ((Date)o.get(f.fieldName)) > f.valDate;
      }else if(f.operator == '<'){
        return ((Date)o.get(f.fieldName)) < f.valDate;
      }else{
        return ((Date)o.get(f.fieldName)) == f.valDate;
      }
    }else if(f.valueType == 'DateTime'){
      if(f.operator == '!='){
        return ((DateTime)o.get(f.fieldName)) != f.valDateTime;
      }else if(f.operator == '>'){
        return ((DateTime)o.get(f.fieldName)) > f.valDateTime;
      }else if(f.operator == '<'){
        return ((DateTime)o.get(f.fieldName)) < f.valDateTime;
      }else{
        return ((DateTime)o.get(f.fieldName)) == f.valDateTime;
      }
    }else if(f.valueType == 'Time'){
      if(f.operator == '!='){
        return ((Time)o.get(f.fieldName)) != f.valTime;
      }else if(f.operator == '>'){
        return ((Time)o.get(f.fieldName)) > f.valTime;
      }else if(f.operator == '<'){
        return ((Time)o.get(f.fieldName)) < f.valTime;
      }else{
        return ((Time)o.get(f.fieldName)) == f.valTime;
      }
    }else if(f.valueType == 'Integer'){
      if(f.operator == '!='){
        return ((Integer)o.get(f.fieldName)) != f.valInteger;
      }else if(f.operator == '>'){
        return ((Integer)o.get(f.fieldName)) > f.valInteger;
      }else if(f.operator == '<'){
        return ((Integer)o.get(f.fieldName)) < f.valInteger;
      }else{
        return ((Integer)o.get(f.fieldName)) == f.valInteger;
      }
    }else if(f.valueType == 'Decimal'){
      if(f.operator == '!='){
        return ((Decimal)o.get(f.fieldName)) != f.valDecimal;
      }else if(f.operator == '>'){
        return ((Decimal)o.get(f.fieldName)) > f.valDecimal;
      }else if(f.operator == '<'){
        return ((Decimal)o.get(f.fieldName)) < f.valDecimal;
      }else{
        return ((Decimal)o.get(f.fieldName)) == f.valDecimal;
      }
    }else if(f.valueType == 'Double'){
      if(f.operator == '!='){
        return ((Double)o.get(f.fieldName)) != f.valDouble;
      }else if(f.operator == '>'){
        return ((Double)o.get(f.fieldName)) > f.valDouble;
      }else if(f.operator == '<'){
        return ((Double)o.get(f.fieldName)) < f.valDouble;
      }else{
        return ((Double)o.get(f.fieldName)) == f.valDouble;
      }
    }else if(f.valueType == 'Long'){
      if(f.operator == '!='){
        return ((Long)o.get(f.fieldName)) != f.valLong;
      }else if(f.operator == '>'){
        return ((Long)o.get(f.fieldName)) > f.valLong;
      }else if(f.operator == '<'){
        return ((Long)o.get(f.fieldName)) < f.valLong;
      }else{
        return ((Long)o.get(f.fieldName)) == f.valLong;
      }
    }else if(f.valueType == 'Id'){
      if(f.operator == '!='){
        return ((Id)o.get(f.fieldName)) != f.valId;
      }else{
        return ((Id)o.get(f.fieldName)) == f.valId;
      }
    }else if(f.valueType == 'String'){
      if(f.operator == '!='){
        return !((String)o.get(f.fieldName)).equals(f.valString);
      }else if(f.operator == 'CONTAINS'){
        return ((String)o.get(f.fieldName)).contains(f.valString);
      }else if(f.operator == 'STARTS'){
        return ((String)o.get(f.fieldName)).startsWith(f.valString);
      }else if(f.operator == 'ENDS'){
        return ((String)o.get(f.fieldName)).endsWith(f.valString);
      }else{
        return ((String)o.get(f.fieldName)).equals(f.valString);
      }
    }

    return false;
  }

  public class Filter {

    public String fieldName {get; private set;}
    public String valueType {get; private set;}
    public String operator {get; private set;}//>,<,=,!=, CONTAINS, STARTS, ENDS

    public Boolean valBoolean {get; private set;}
    public Date valDate {get; private set;}
    public DateTime valDateTime {get; private set;}
    public Time valTime {get; private set;}
    public Integer valInteger {get; private set;}
    public Decimal valDecimal {get; private set;}
    public Double valDouble {get; private set;}
    public Long valLong {get; private set;}
    public Id valId {get; private set;}
    public String valString {get; private set;}

    public Filter(String fieldName, String operator, Boolean value){
      this.fieldName = fieldName;
      this.valueType = 'Boolean';
      this.valBoolean = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, Date value){
      this.fieldName = fieldName;
      this.valueType = 'Date';
      this.valDate = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, DateTime value){
      this.fieldName = fieldName;
      this.valueType = 'DateTime';
      this.valDateTime = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, Time value){
      this.fieldName = fieldName;
      this.valueType = 'Time';
      this.valTime = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, Integer value){
      this.fieldName = fieldName;
      this.valueType = 'Integer';
      this.valInteger = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, Decimal value){
      this.fieldName = fieldName;
      this.valueType = 'Decimal';
      this.valDecimal = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, Double value){
      this.fieldName = fieldName;
      this.valueType = 'Double';
      this.valDouble = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, Long value){
      this.fieldName = fieldName;
      this.valueType = 'Long';
      this.valLong = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, Id value){
      this.fieldName = fieldName;
      this.valueType = 'Id';
      this.valId = value;
      this.operator = operator;
    }

    public Filter(String fieldName, String operator, String value){
      this.fieldName = fieldName;
      this.valueType = 'String';
      this.valString = value;
      this.operator = operator;
    }

  }
}
{% endhighlight %}

## Sample usage

Below you can see some of the ways you could use this `ArrayFilter` class to filter a list of objects.

Example #1:
{% highlight java %}
//filter by String
List<sObject> filteredArray = ArrayFilter.filter(someArrayToFilter, new ArrayFilter.Filter[]{new ArrayFilter.Filter('Name', '=', 'John Doe')});
{% endhighlight %}

Example #2:
{% highlight java %}
List<ArrayFilter.Filter> filters = new List<ArrayFilter.Filter>();
filters.add(new ArrayFilter.Filter('Migration_Start_Date__c', '=', Date.newInstance(2000,1,2)));
filters.add(new ArrayFilter.Filter('Migration_Start_Date__c', '>', Date.newInstance(2010,3,4)));

List<Account> filtered = ArrayFilter.filter(someArrayToFilter,filters);
{% endhighlight %}

Example #3:
{% highlight java %}
//text supports operators: =, !=, CONTAINS, ENDS, STARTS
List<ArrayFilter.Filter> filters = new List<ArrayFilter.Filter>();
filters.add(new ArrayFilter.Filter('Phone', 'STARTS',  '0355'));

List<Account> filtered = ArrayFilter.filter(someArrayToFilter,filters);
{% endhighlight %}

## Ideas for improvement

I didn't had the time yet, nor the need, but this class can be easily improved to accept groups of filter conditions to which you could apply the `OR` or `AND` logical operators. The code currently applies `OR` logic to multiple filter conditions.
