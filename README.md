# SalesForce Developer

## Notes
- change set
    - it contains information about the org but not contain data or record
    - inbound change set : Sandbox code deployment to Prod
    - outbound change set : Prod code deployment to Sandbox
    - Do's
        - deploy customization from one salesforce org to another
        - new object creation
        - changes can be made using setup menu
    - Don't
        - upload list of contact records

- Salesforce CLI 
    -  extract metadata from a development environment to integrate with a version control system (VCS).
    -  script routine tasks and boost the productivity of everyone contributing to the release
- Apex sObject => refers to a row in Standard or Custom Object
  - 
- Apex generic sObject variable
  
 - 

## Application Lifecycle Management

### Flow
> Plan > Develop > Test > Build Releases > Test Releases > Release  

### Release Management Process
- Falls into 3 categories
  1. Patch 
  2. Bug fixes
    
  3. Simple changes
    - included reports, dashboards, list views, and email templates.

- Minor Changes with limited impact, such as a new trigger impacting a single business process.
-  development
  - is a set of metadata changes, like a diff or delta



## APEX
- Salesforce records and their fields directly from Apex

### sObject 
- Every record in Salesforce is natively represented as an sObject in Apex
- 'Acme' record in Account corresponds to an Account sObject in Apex
- Ex 
```
Account acct = new Account(Name='Acme', Phone='(415)555-1212', NumberOfEmployees=100);
// or
Account acct = new Account();
acct.Name = 'Acme';
acct.Phone = '(415)555-1212';
acct.NumberOfEmployees = 100;
```
- To access this field in Apex, you’ll need to use the API name for the field
    - A custom object with a label of Merchandise has an API name of Merchandise__c.
    - A custom field with a label of Description has an API name of Description__c.
    - A custom relationship field with a label of Items has an API name of Items__r.
-  spaces in labels are replaced with underscores in API names. 

### Generic Object 
- Ex # 01 

```

sObject genericObject = new Account();
Account account = (Account) genericObject;
``` 
- Ex # 02

```
sObject sobj1 = new Account(Name='Trailhead');
sObject sobj2 = new Book__c(Name='Workbook 1');
``` 
    
### Manipulate Records with DML
- Create and modify records in Salesforce by using the Data Manipulation Language,
- using DML, you can manage records by providing simple statements to insert, update, merge, delete, and restore records.
-  following DML statements are available
    - insert
    - update
    - upsert
    - delete
    - undelete
    - merge
- Ex 

```
// Create the account sObject 
Account acct = new Account(Name='Acme', Phone='(415)555-1212', NumberOfEmployees=100);
// Insert the account by using DML
insert acct;
// Get the new ID on the inserted sObject argument
ID acctID = acct.Id;
```

#### Bulk DML
- Each DML statement accepts either a single sObject or a list (or array) of sObjects
- bulk DML operations is the recommended way because it helps avoid hitting governor limits
    - DML limit of 150 statements per Apex transaction
```
// Create a list of contacts
List<Contact> conList = new List<Contact> {
    new Contact(FirstName='Joe',LastName='Smith',Department='Finance'),
    new Contact(FirstName='Kathy',LastName='Smith',Department='Technology'),
    new Contact(FirstName='Caroline',LastName='Roth',Department='Finance'),
    new Contact(FirstName='Kim',LastName='Shain',Department='Education')};

// Bulk insert all contacts with one DML call
insert conList;

// List to hold the new contacts to update
List<Contact> listToUpdate = new List<Contact>();
for(Contact con : conList) {
    if (con.Department == 'Finance') {
        con.Title = 'Financial analyst';
        // Add updated contact sObject to the list.
        listToUpdate.add(con);
    }
}
// Bulk update all contacts with one DML call
update listToUpdate;
```

#### Upert
- upsert statement matches the sObjects with existing records by comparing values of one field
  - upsert jane2 Contact.fields.Email;
- upsert
    - If the key is not matched, a new object record is created.
    - If the key is matched once, the existing object record is updated.
    - If the key is matched multiple times, an error is generated and the object record is neither inserted or updated.
- ex

```
// Insert the Josh contact
Contact josh = new Contact(FirstName='Josh',LastName='Kaplan',Department='Finance');       
insert josh;
josh.Description = 'Josh\'s record has been updated by the upsert operation.';

// Create the Kathy contact, but don't persist it in the database
Contact kathy = new Contact(FirstName='Kathy',LastName='Brown',Department='Technology');

// List to hold the new contacts to upsert
List<Contact> contacts = new List<Contact> { josh, kathy };
// Call upsert
upsert contacts;
// Result: Josh is updated and Kathy is created.
```

#### Delete
- Deleted records aren’t deleted permanently from Lightning Platform
- deleted records are placed in the Recycle Bin for 15 days from where they can be restored
- ex

```
Contact[] contactsDel = [SELECT Id FROM Contact WHERE LastName='Smith']; 
delete contactsDel;
```

#### Exceptions
- If a DML operation fails, it returns an exception of type DmlException
- ex

```
try {
    // This causes an exception because 
    //   the required Name field is not provided.
    Account acct = new Account();
    // Insert the account 
    insert acct;
} catch (DmlException e) {
    System.debug('A DML exception has occurred: ' +e.getMessage());
}
```

#### Database Methods
- Apex contains the built-in Database class, which provides methods that perform DML operations
- Database methods are static and are called on the class name.
    - Database.insert()
    - Database.update()
    - Database.upsert()
    - Database.delete()
    - Database.undelete()
    - Database.merge()
- above methods have an optional 'allOrNone' parameter which allows you to specify whether the operation should partially succeed. 
    - if false
        - if errors occur on a partial set of records, the successful records will be committed and errors will be returned for the failed records. Also, no exceptions are thrown with the partial success option. 
    - default = true
        - will throw an exception if a failure is encountered
- Ex :
```
// result objects containing success or failure information for each record
Database.SaveResult[] results = Database.insert(recordList, false);
for (Database.SaveResult sr : results) {
    if (sr.isSuccess()) {
    } else {
        for(Database.Error err : sr.getErrors()) {
            System.debug('The following error has occurred.');
            System.debug(err.getStatusCode() + ': ' + err.getMessage());
            System.debug('Contact fields that affected this error: ' + err.getFields());
	 }
    }
}
```

## Write SOQL Queries
- SOQL => Salesforce Object Query Language
- Writing SOQL in Apex
    - wrap the SOQL statement within square brackets
    - Ex : Account[] accts = [SELECT Name,Phone FROM Account];
- Note
    - never use * for get data for all fields as it is not supported
    - You must specify every field you want
    - need to specify the Id field in the query, id will be returned by default
        - ex : SELECT Id,Phone FROM Account

### Child Records in Parent
- Master-Details Relationship & lookup records
- Ex : SELECT Name, (SELECT LastName FROM Contacts) FROM Account WHERE Name = 'SFDC Computing'
    - Account is the parent and contact is the child
```
Account[] acctsWithContacts = [SELECT Name, (SELECT FirstName,LastName FROM Contacts)
                               FROM Account 
                               WHERE Name = 'SFDC Computing'];
// Get child records
Contact[] cts = acctsWithContacts[0].Contacts;
System.debug('Name of first associated contact: ' 
             + cts[0].FirstName + ', ' + cts[0].LastName);
``` 

### Parent record info in Child 
- You can traverse a relationship from a child object (contact) to a field on its parent (Account.Name) by using dot notation
```
Contact[] cts = [SELECT Account.Name FROM Contact WHERE FirstName = 'Carol' AND LastName='Ruiz'];
Contact carol = cts[0];
String acctName = carol.Account.Name;
System.debug('Carol\'s account name is ' + acctName);
```

### Assignment

```
public class ContactSearch {
    public static Contact[] searchForContacts(String lastName, String mailingPostalCode) {
        try {
            Contact[] contacts = [SELECT FirstName, LastName FROM Contact where LastName=:lastName and MailingPostalCode=:mailingPostalCode];
            return contacts;
        } catch(Exception e) {
            return null;
        }
    }
}
```


## Write SOSL Queries
- SOSL => Salesforce Object Search Language
- used to perform text searches in records 
- ex

```
List<List<SObject>> searchList = [FIND 'SFDC' IN ALL FIELDS 
                                  RETURNING Account(Name), Contact(FirstName,LastName)];
```

### SOQL vs SOSL

| # | SOQL | SOSL | 
| :---: | :--- | :--- |
|1|Used to retrieve records for a single object|single SOSL query can search fields across multiple objects


### Assignment

```
public class ContactAndLeadSearch {
    public static List<List<sObject>> searchContactsAndLeads(String searchText) {
        try {
            List<List<sObject>> searchResultsList = [FIND :searchText IN ALL FIELDS
                    RETURNING Contact(FirstName,LastName),Lead(FirstName,LastName)];
            return searchResultsList;
        } catch(Exception e) {
            return null;
        }
    }
}
```

## Insert Data into Lead Object

```
Lead lead = new Lead(FirstName='S1', LastName='Smith', Company='Smith Inc');
insert lead;
```

## Insert Data into Contact & Account Objects

```
Account acct = new Account(
    Name='Smith Computing',
    Phone='(415)555-1212',
    NumberOfEmployees=50,
    BillingCity='San Francisco');
insert acct;

ID acctID = acct.ID;
Contact contact = new Contact(
    FirstName='Carol',
    LastName='Smith',
    Phone='(415)555-1212',
    Department='Wingo',
    AccountId=acctID);
insert contact;
```


## Apex Triggers
- same as oracle triggers
- perform custom actions before or after events to records in Salesforce, such as insertions, updates, or deletions
- you can use triggers in Apex, including executing SOQL and DML or calling custom Apex methods

### trigger syntax
 
- Syntax
```
trigger TriggerName on ObjectName (trigger_events) {
   code_block
}
```

- Example

```
trigger HelloWorldTrigger on Account (before insert) {
	System.debug('Hello World!');
}
```

- Trigger events - below are supported
    - before insert
    - before update
    - before delete
    - after insert
    - after update
    - after delete
    - after undelete

- Trigger Types

| # | Type | Notes | 
| :---: | :--- | :--- |
|1|Before triggers|used to update or validate record values before they’re saved to the database
|2|After triggers|





## Apex Integration Services
- Apex callout enables you to with an external service using Apex Code
- callout sends an HTTP request from Apex code and received the response
- 2 types of callout
  - SOAP Services (XML based request/response, require WSDL for code generation)
  - Http Service(JSON Based request/response using http methos [], require Swaggers for code generation)
 - Note
   - Every request needs to be authorized when you call external services using the Apex Callouts
- Steps
  - From Setup, enter Remote Site Settings in the Quick Find box, then click Remote Site Settings.
  - Click New Remote Site
  - For the remote site name, enter 'xxx'.
  - For the remote site URL, enter https://xyz.com.
  - Click Save

### Rest Callouts
- 
- Http Methods

| # | Http Method | Notes | DB Operation|
| :---: | :--- | :--- |:---|
|1|Get|Retrieve data identified by a URL| Select 
|2|Post|Create a resource or post data to the server|Create
|3|Delete|Delete a resource identified by a URL|Delete
|4|Put|Create or replace the resource sent in the request body|Update

- Code for Get Call
```
Http http = new Http();
HttpRequest request = new HttpRequest();
request.setEndpoint('https://th-apex-http-callout.herokuapp.com/animals');
request.setMethod('GET');
HttpResponse response = http.send(request);
// If the request is successful, parse the JSON response.
if(response.getStatusCode() == 200) {
  // Deserializes the JSON string into collections of primitive data types.
  Map<String, Object> results = (Map<String, Object>) JSON.deserializeUntyped(response.getBody());
  // Cast the values in the 'animals' key as a list
  List<Object> animals = (List<Object>) results.get('animals');
  System.debug('Received the following animals:');
  for(Object animal: animals) {
    System.debug(animal);
  }
}
```

- Code for Post Call

```
Http http = new Http();
HttpRequest request = new HttpRequest();
request.setEndpoint('https://th-apex-http-callout.herokuapp.com/animals');
request.setMethod('POST');
request.setHeader('Content-Type', 'application/json;charset=UTF-8');
request.setBody('{"name":"mighty moose"}');
HttpResponse response = http.send(request);
// Parse the JSON response
if(response.getStatusCode() != 201) {
   System.debug('The status code returned was not expected: ' +
   response.getStatusCode() + ' ' + response.getStatus());
} else {
   System.debug(response.getBody());
}
```

 

  
