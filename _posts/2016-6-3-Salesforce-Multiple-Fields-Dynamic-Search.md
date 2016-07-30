---
layout: post
title: Search records by multiple fields
published: false
tags: javascript visualforce
---

{% highlight javascript %}

var QueryOptions = { primary_object : 'Custom_Object**c ', select_fields : undefined, where : [`filter_group_0`], filter_groups : [[ { inner_logic: 'OR',//only matters if there is more than 1 condition conditions: [ { html_id : 'Service_ID**c', type : 'TEXT', api : 'Service_ID__c', operator : 'LIKE', text_search : 'CONTAINS', //CONTAINS, STARTS, END, EQUALS } ] } ]] };

{% endhighlight %}

{% highlight javascript %}

/**
 * Query Builder
 * @param  {[type]} queryOptions [description]
 * @param  {[type]} queryOrder   [description]
 * @return {String}              Returns a query ready
 */
var buildQuery = function (queryOptions, queryOrder) {
  var baseQuery = "SELECT select_fields FROM primary_object WHERE ";

  baseQuery += queryOptions.where[queryOrder];
  baseQuery = baseQuery.replace('primary_object', queryOptions.primary_object);

  var selectFields = '';
  for (var i = 0; i < queryOptions.select_fields.length; i++) {
      if (selectFields === '') {
          selectFields += queryOptions.select_fields[i];
      } else {
          selectFields += ',' + queryOptions.select_fields[i];
      }
  }

  baseQuery = baseQuery.replace('select_fields', selectFields);

  for (var i = 0; i < queryOptions.filter_groups[queryOrder].length; i++)
  {
    var group = queryOptions.filter_groups[queryOrder][i];
    var conditions = queryOptions.filter_groups[queryOrder][i].conditions;
    var groupQuery = '';

    for (var j = 0; j < conditions.length; j++) {
        var con = conditions[j];
        var input = $('#' + selectedSearchType + ' [id$=' + con.html_id + ']');

        //don't add the condition if it's not inputted
        if (con.type != 'BOOLEAN' && input.val() === '') {
            continue;
        }

        //append LIKE|OR
        if (groupQuery !== '') {
            groupQuery += ' ' + group.inner_logic + ' ';
        }

        if (con.supports_multiple_values === true) {
            var queryValues = input.val().split(',');
            var multiQuery = '';
            for (var z = 0; z < queryValues.length; z++) {
                if (multiQuery === '') {
                    multiQuery = ' ( ' + con.api + ' ' + con.operator + ' ' + buildCondition(con, queryValues[z].trim());
                } else {
                    multiQuery += ' OR ' + con.api + ' ' + con.operator + ' ' + buildCondition(con, queryValues[z].trim());
                }
            }
            groupQuery += multiQuery + ' ) ';
        } else {
            groupQuery += ' ' + con.api + ' ' + con.operator + ' ' + buildCondition(con, input);
        }
    }

    if (groupQuery !== '') {
        groupQuery = " ( " + groupQuery + " )";
    } else {
        groupQuery = " ( Id != '' ) ";
    }

    baseQuery = baseQuery.replace('filter_group_' + i, groupQuery);

    groupQuery['filter_group_' + i] = groupQuery;
  }

  return baseQuery;
}

{% endhighlight %}
