---
layout: post
title: "You're up and running!"
published: true
---






Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.

```
public class CapexFormController {

   public Id dummy_contactId {get;set;}
   public String returnUrl {get;set;}

   public CapexFormController() {
        User tmp_u = [select id, Contact_Id__c from User where id = :UserInfo.getUserId() limit 1];
        system.debug('ms -> contact Id ' + tmp_u);
        dummy_contactId = tmp_u.Contact_Id__c;

        System.debug('ms previous page ->' + ApexPages.currentPage().getParameters().get('retURL'));

        returnUrl = 'https://ap4.salesforce.com/home/home.jsp';

        if(CapexFormController.isSF1() == false)
        {
            String retUrl = ApexPages.currentPage().getParameters().get('retURL');

            if(retUrl != null)
            {
                returnUrl = retUrl;
            }
        }

        System.debug('ms returnUrl ->' + returnUrl);
   }


   @RemoteAction
   public static void submitForApproval(String capexRequestJSON, String requestItemsJSON, String sumJSON)
   {
       CAPEX_Request__c capexRequest = (CAPEX_Request__c) JSON.deserialize(capexRequestJSON, CAPEX_Request__c.class);
       List<CapexRequestItemsWrapper> requestItems = (List<CapexRequestItemsWrapper>) JSON.deserialize(requestItemsJSON, List<CapexRequestItemsWrapper>.class);
       Decimal requestSum = (Decimal) JSON.deserialize(sumJSON, Decimal.class);

       System.debug('ms-> ' +requestItems.size());

       capexRequest.CurrencyIsoCode = capexRequest.Currency__c;

       // Create a database savepoint before inserts
       Savepoint sp = Database.setSavepoint();

       try{
           insert capexRequest;

           System.debug('ms-> calling insert request items');
           //insert the request items
           CapexFormController.insertRequestItems(capexRequest.Id, capexRequest.Currency__c, requestItems);

           System.debug('ms-> returning from insert request items');

           System.debug('ms-> createCase');
           CapexFormController.createCase(capexRequest, requestSum);
           System.debug('ms-> returning from createCase');

       }catch(DmlException e){
           //an exception has occurred, roll back the database
           Database.rollback(sp);


           System.debug('ms-> '+e.getMessage());
           System.debug(e);
       }
   }
 }
```
