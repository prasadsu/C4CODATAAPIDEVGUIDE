# SAP Cloud for Customer OData API Developer's Guide
The SAP Cloud for Customer OData API Developer’s Guide complements the [SAP Cloud for Customer OData API Reference](http://help.sap.com/saphelp_sapcloudforcustomer/en/ODATA_APIs/index.html#e4d5b5e4f6d847f7ad2025f5f343e03f.html) with usage details and samples for SAP Cloud for Customer OData API in a format that is most convenient to developers. Furthermore, it also covers known restrictions and limitations of the SAP Cloud for Customer OData API.

For a brief introduction to SAP Cloud for Customer OData API, please refer to [SAP Cloud for Customer OData API Getting Started Guide](http://help.sap.com/saphelp_sapcloudforcustomer/en/ODATA_APIs/index.html#17bf468d1b69474794a1ea4cac125be4.html).

## What is OData protocol?
[Open Data (OData) Protocol](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=odata) is an OASIS standard that defines best practices for building and consuming RESTful APIs. It is based on HTTP protocol and provides metadata for the entities it exposes and their relationships. In some ways, it is similar to SQL for a relational database system (RDBMS) as it provides querying options such as filtering, ordering results, support for pagination, number of records and more. It supports both XML (Atom) and JSON formats for querying and modifying data.

For more information on OData please refer to http://www.odata.org where you can find detailed documentation and tutorials. 

## OData versions
SAP Cloud for Customer, specifically, supports the V2.0 of the OData protocol (with some additional enhancements and a few limitations), you can read the details of OData V2 [here](http://www.odata.org/documentation/odata-version-2-0/).

## SAP Cloud for Customer (C4C) OData Services
You can try the examples shown in this document by accessing the OData API of your SAP Cloud for Customer (C4C) tenant using the following URL pattern:

```
https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/v1/odataservicecatalog/ODataServiceCollection
```

where myNNNNNN is the name of your tenant.

Here is an example URL for a test tenant:

```
https://odatac4ctrial.hana.ondemand.com/proxy/sap/c4c/odata/v1/c4codata/
```

## OData Service Catalog
OData Service Catalog contains the list of available OData Services in the corresponding C4C tenant. In order to get the list of available OData services in your C4C tenant use the following URL:

```
https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/v1/odataservicecatalog/ODataServiceCollection
```

The catalog service returns both standard OData services delivered by SAP as well as the custom services that you may have modeled in your tenant using the [OData Service Explorer](http://help.sap.com/saphelp_sapcloudforcustomer/en/ODATA_APIs/index.html#8e4220fa6dc943ef891fb3d0e91515d3.html).

## Authentication
SAP Cloud for Customer OData API supports two different authentication mechanisms:

* Basic Authentication (username and password pair)
* OAuth SAML Bearer flow (you can find sample Java implementation of OAuth SAML bearer client [here](OAuthSAMLClient).)

Please note that the C4C system used in the example URLs throughout this document, doesn't require authentication.

## SAP Standard vs. Custom OData Services

SAP Cloud for Customer provides a standard OData API. In addition, SAP Cloud for Customer also allows customers to build their own (custom) OData services based on the predefined business objects in the solution.

[OData Service Explorer](http://help.sap.com/saphelp_sapcloudforcustomer/en/ODATA_APIs/index.html#8e4220fa6dc943ef891fb3d0e91515d3.html) is a key user tool that allows exploring and testing SAP provided OData API as well as building custom services.

Standard and custom OData services offer the same capabilities and, are subjected to the same restrictions. 

The following URL pattern differetiates the Standard and Custom OData services.


* Standard services - `https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/v1/....`
* Custom services - `https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/cust/v1/...`

## OData Service Document
OData service document contains the list of OData entities (a.k.a. collections) contained within that OData service. In order to retrieve the complete list of entities included in C4C OData service, you can open the following URL in your browser.

```
https://myNNNNNN.crm.ondemand/sap/c4c/odata/v1/c4codata/ 
```

where myNNNNNN is the name of your C4C tenant. (Please note that ‘/’ character at the end of the URI is required!)

## OData Service Metadata
OData service metadata is retrieved via the following URL.

```
https://myNNNNNN.crm.ondemand/sap/c4c/odata/v1/c4codata/$metadata
```

Example:

```
https://odatac4ctrial.hana.ondemand.com/proxy/sap/c4c/odata/v1/c4codata/$metadata
```

####Supported HTTP operations

C4C OData API supports the following OData/HTTP operations:

Operation | Description
----------|------------
GET | Used to retrieve a single entity instance or multiple entity instances
POST | Used to create entity instances
PUT | Used to **completely** replace/overwrite and existing entity instance
PATCH | Used to replace/overwrite existing entity instance. The key difference between PUT and PATCH is that PUT overwrites the complete entity whereas PATCH only updates **only** attributes of the entity that are part of the payload
DELETE | Used to delete an entity record
$batch | Used to perform multiple query, create, update and delete operations with explicit transaction boundaries specified via Changesets as a part of the payload
Deep Insert | Used with **POST**. Allows the creation of complete entity (header entry, child entries etc) with a single POST request


## Making HTTP Requests
### Supported Formats
SAP Cloud for Customer OData API supports HTTP request and response payloads in both Atom (XML) and JSON formats. The default payload format is Atom (XML). In order to use JSON format please follow the instructions below:
* For HTTP GET requests, use the system query parameter `$format=json`. 

Example:
```
https://odatac4ctrial.hana.ondemand.com/proxy/sap/c4c/odata/v1/odataservicecatalog/?$format=json
```

will return 

```JSON
{"d":{"EntitySets":["ODataServiceCollection"]}}
```

* For HTTP POST/PATCH/PUT requests with JSON payload, set the HTTP `Content-Type` header as below:

```
Content-Type: application/json
```

### Authentication
All HTTP requests should have an `Authorization` header. 


Authentication Method |HTTP Header
-------|---------
Basic authentication | `Authorization: Basic _base64_encoded_value_of_username:password_`
OAuth SAML bearer flow  | `Authorization: Bearer _OAuth_token_` 

In the formats shown above, please note the space between `Basic`, `Bearer` and the values following them respectively.

Please note that the C4C system used in the example URLs throughout this document, doesn't require authentication.

### CSRF Token
In order to prevent possible [Cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) attacks, SAP Cloud for Customer OData API requires all modifying HTTP requests (POST/PUT/PATCH) to specify a CSRF token, in addition to the Authorization header.

Please follow the steps below to receive a CSRF token:

First, perform an HTTP GET request to the service end-point (e.g. retrieve the service document end-point `https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/v1/c4codata/`) with the HTTP request header:

```
	x-csrf-token: fetch
```

After a successful call, the C4C server will respond with the expected response payload and a CSRF token in the respective response `X-CSRF-Token`. 

Here is an example CSRF Token returned as part of the response header 

```
	x-csrf-token: Xi6wOfG-O55Wt8ZkhYW0eA==
```

The token value retrieved above needs to be used for subsequent modifying HTTP requests (i.e. POST/PUT/PATCH).

###Server side paging
For HTTP GET requests, if no query options are specified, the server enforces paging in order to provide better performance. Currently the page size is fixed at 1000 entries. 

If there are more than 1000 entries, the server includes a `<link re"next" href="...` element that can be used to retrieve the next 1000 entries. 

Here is an excerpt with the **next** link:

```XML
...
				<d:StartDate>2012-08-25T00:00:00</d:StartDate>
				<d:StatusCode>6</d:StatusCode>
				<d:StatusCodeText>Converted</d:StatusCodeText>
				<d:UUID>00163E03-A070-1EE2-8BE6-D1A72CF7B7D6</d:UUID>
			</m:properties>
		</content>
	</entry>
	<link rel="next" href="https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/v1/c4codata/LeadCollection/?$skiptoken=1001"/>
</feed>
```

(in this specific case the LeadCollection entity set is being queried).

###Sample Java Client
A sample Java client demonstrating how to make OData calls to C4C is available [here](ODataConsumerSample). The sample uses Apache Olingo library to construct and read OData payloads.

###Supported System Query Options
As stated above, SAP Cloud for Customer supports version 2 of the OData protocol. Here we list the set of system query options that are supported by the C4C OData implementation. For brevity, initial part of the URL https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/v1/c4codata is skipped in the following examples:

Query Option | Example | Description
-------|---------|------------
**$batch** | /$batch | Perform several OData query, create, update or delete operations with a single HTTP POST call.
**$count** | /OpportunityCollection/$count | Returns the total number of Opportunities
**$format**  | /OpportunityCollection?$format=json | Returns Opportunity entries in JSON format with server side paging
**$inlinecount** | /OpportunityCollection?$top=10&$inlinecount=allpages | Returns the top 10 opportunities and also returns the total number of opportunities.
**$orderby** | /OpportunityCollection?$orderby=CloseDate desc&$top=10 | First performs an orderby on the Opportunities and then selects the top 10 from that ordered list. Here **desc** means descending order.
**$search** | /CustomerCollection?$search='Porter' | Returns Customer entries with at least one of the search enabled fields contain the word 'Porter'
**$select** | /OpportunityCollection?$select=OpportunityID,AccountID | Returns Opportunity entries but only 2 attributes OpportunityID and AccountID
**$skip** | /OpportunityCollection?$skip=10 | Skips the first 10 entries and then returns the rest
**$top** |  /OpportunityCollection?$top=10 | Returns top 2 Opportunities. 'Top 2' is defined by server logic here

Below you will find additional details for some of the system query options.

####$batch
Used to query, create/update multiple entities with explicit transaction boundaries specified via Changesets as a part of the payload

The following example;
- retrieves top 3 entries from the ServiceRequestCollection (of the requestservice OData service)
- updates a few properties of a ServiceRequest entry
- creates a new ServiceRequestItem

Note: Please make sure to leave two blank lines after HTTP GET operations (as seen in the example below)

```XML
POST /sap/c4c/odata/v1/servicerequest/$batch HTTP/1.1
Host: myNNNNNN.crm.ondemand.com
Content-Type: multipart/mixed; boundary=batch_guid_01
Content-Length: 1000
Authorization: Basic <base64encoded(user:pass)>
x-csrf-token: <a_valid_csrf_token>

--batch_guid_01
Content-Type: application/http
Content-Transfer-Encoding:binary

GET ServiceRequestCollection/?$top=3 HTTP/1.1


--batch_guid_01
Content-Type: multipart/mixed; boundary=changeset_guid_01

--changeset_guid_01
Content-Type: application/http
Content-Transfer-Encoding: binary

PATCH ServiceRequestCollection('00163E0DBD9E1ED596EBDFDA564728AC') HTTP/1.1
Content-Type: application/json
Content-ID: 1
Content-Length: 10000

{
	"ServiceRequestUserLifeCycleStatusCode" : "YJ",
	"ScheduledStartDate" : "2015-10-22T00:00:00",
	"ScheduledEndDate" : "2015-10-22T00:00:00",
	"ScheduledStartTime" : "PT13H00M00S",
	"ScheduledEndTime" : "PT15H00M00S"
}

--changeset_guid_01
Content-Type: application/http 
Content-Transfer-Encoding: binary 

POST ServiceRequestItemCollection HTTP/1.1
Content-Type: application/json
Content-ID: 2
Content-Length: 10000

{
    "Description": "1m water hose",
    "ParentObjectID": "00163E0DBD9E1ED596EBDFDA564728AC",
    "ProductID": "10000760"
}

--changeset_guid_01-- 
--batch_guid_01--
```

Here is the response received for the above $batch call:

```XML
--ejjeeffe0
Content-Type: application/http
Content-Length: 18616
Content-Transfer-Encoding: binary

HTTP/1.1 200 OK
Content-Type: application/atom+xml;type=feed
Content-Length: 18503
dataserviceversion: 2.0


<?xml version="1.0" encoding="utf-8"?>
<feed xml:base="https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/servicerequest/" xmlns="http://www.w3.org/2005/Atom" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices">
    <id>https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/servicerequest/ServiceRequestCollection</id>
    <title type="text">ServiceRequestCollection</title>
    <updated>2015-10-02T23:43:16Z</updated>
    <author>
        <name/>
    </author>
    <link href="ServiceRequestCollection" rel="self" title="ServiceRequestCollection"/>
    <entry>
        <id>https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/servicerequest/ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')</id>
        <title type="text">ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')</title>
        <updated>2015-10-02T23:43:16Z</updated>
        <category term="servicerequest.ServiceRequest" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')" rel="edit" title="ServiceRequest"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')/AddressSnapshotPostalAddress" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/AddressSnapshotPostalAddress" type="application/atom+xml;type=feed" title="AddressSnapshotPostalAddress"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')/ServiceRequestAttachmentFolder" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestAttachmentFolder" type="application/atom+xml;type=feed" title="ServiceRequestAttachmentFolder"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')/ServiceRequestBusinessTransactionDocumentReference" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestBusinessTransactionDocumentReference" type="application/atom+xml;type=feed" title="ServiceRequestBusinessTransactionDocumentReference"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')/ServiceRequestDescription" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestDescription" type="application/atom+xml;type=feed" title="ServiceRequestDescription"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')/ServiceRequestItem" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestItem" type="application/atom+xml;type=feed" title="ServiceRequestItem"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28A8644CF8D83AE76')/ServiceRequestTextCollection" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestTextCollection" type="application/atom+xml;type=feed" title="ServiceRequestTextCollection"/>
        <content type="application/xml">
            <m:properties>
                <d:ActivityCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:ActivityCategoryName>
                <d:ActivityServiceIssueCategoryID/>
                <d:ApprovalStatusCode/>
                <d:ApprovalStatusCodeText/>
                <d:AssignedTechnician/>
                <d:AssignedTo>8000000207</d:AssignedTo>
                <d:CauseCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:CauseCategoryName>
                <d:CauseServiceIssueCategoryID/>
                <d:ChangedBy>Eddie Wright</d:ChangedBy>
                <d:CompletedOnDate m:null="true"/>
                <d:CompletionDueDate>2012-11-08T00:00:00</d:CompletionDueDate>
                <d:Contact/>
                <d:ContractID/>
                <d:CreatedBy>Eddie Wright</d:CreatedBy>
                <d:CreationDate>2012-11-06T00:00:00</d:CreationDate>
                <d:Customer>AgileX</d:Customer>
                <d:CustomerID>1000077</d:CustomerID>
                <d:DataOriginTypeCode>1</d:DataOriginTypeCode>
                <d:DataOriginTypeCodeText>Manual data entry</d:DataOriginTypeCodeText>
                <d:DiagnosticTest>false</d:DiagnosticTest>
                <d:ID>2</d:ID>
                <d:IncidentCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:IncidentCategoryName>
                <d:IncidentServiceIssueCategoryID/>
                <d:InitialResponseDate m:null="true"/>
                <d:InitialReviewDueDate>2012-11-06T00:00:00</d:InitialReviewDueDate>
                <d:InstallationPointID/>
                <d:InstalledBaseID/>
                <d:ItemListServiceRequestExecutionLifeCycleStatusCode>1</d:ItemListServiceRequestExecutionLifeCycleStatusCode>
                <d:ItemListServiceRequestExecutionLifeCycleStatusCodeText>Open</d:ItemListServiceRequestExecutionLifeCycleStatusCodeText>
                <d:LastChangeDate>2012-11-06T00:00:00</d:LastChangeDate>
                <d:LastResponseOnDate m:null="true"/>
                <d:Name>Monarch bike enquiry</d:Name>
                <d:NextResponseDueDate m:null="true"/>
                <d:ObjectCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:ObjectCategoryName>
                <d:ObjectID>00163E03A0701ED28A8644CF8D83AE76</d:ObjectID>
                <d:ObjectServiceIssueCategoryID/>
                <d:Partner/>
                <d:PartnerID/>
                <d:ProcessingTypeCode>SRRQ</d:ProcessingTypeCode>
                <d:ProcessingTypeCodeText>Service Request</d:ProcessingTypeCodeText>
                <d:ProductCategoryDescription/>
                <d:ProductID/>
                <d:ReferenceDate m:null="true"/>
                <d:ReportedForEmail/>
                <d:ReportedForPartyID/>
                <d:ReporterEmail/>
                <d:ReporterPartyID/>
                <d:RequestAssignmentStatusCode>1</d:RequestAssignmentStatusCode>
                <d:RequestAssignmentStatusCodeText>Processor Action</d:RequestAssignmentStatusCodeText>
                <d:RequestedEnd/>
                <d:RequestedEndTimeZoneCode/>
                <d:RequestedStart/>
                <d:RequestedStartTimeZoneCode/>
                <d:SalesTerritoryID/>
                <d:ScheduledEndDate m:null="true"/>
                <d:ScheduledEndTime>PT00H00M00S</d:ScheduledEndTime>
                <d:ScheduledStartDate m:null="true"/>
                <d:ScheduledStartTime>PT00H00M00S</d:ScheduledStartTime>
                <d:SerialID/>
                <d:ServiceAndSupportTeam>S3110</d:ServiceAndSupportTeam>
                <d:ServiceCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:ServiceCategoryName>
                <d:ServiceIssueCategoryID/>
                <d:ServiceLevelAgreement/>
                <d:ServicePriorityCode>3</d:ServicePriorityCode>
                <d:ServicePriorityCodeText>Normal</d:ServicePriorityCodeText>
                <d:ServiceRequestClassificationCode/>
                <d:ServiceRequestClassificationCodeText/>
                <d:ServiceRequestLifeCycleStatusCode>1</d:ServiceRequestLifeCycleStatusCode>
                <d:ServiceRequestLifeCycleStatusCodeText>Open</d:ServiceRequestLifeCycleStatusCodeText>
                <d:ServiceRequestUserLifeCycleStatusCode>1</d:ServiceRequestUserLifeCycleStatusCode>
                <d:ServiceRequestUserLifeCycleStatusCodeText>Open</d:ServiceRequestUserLifeCycleStatusCodeText>
                <d:ServiceTechnician/>
                <d:ServiceTechnicianTeam/>
                <d:Technician/>
                <d:WarrantyDescription/>
                <d:WarrantyFrom m:null="true"/>
                <d:WarrantyTo m:null="true"/>
            </m:properties>
        </content>
    </entry>
    <entry>
        <id>https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/servicerequest/ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')</id>
        <title type="text">ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')</title>
        <updated>2015-10-02T23:43:16Z</updated>
        <category term="servicerequest.ServiceRequest" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')" rel="edit" title="ServiceRequest"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')/AddressSnapshotPostalAddress" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/AddressSnapshotPostalAddress" type="application/atom+xml;type=feed" title="AddressSnapshotPostalAddress"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')/ServiceRequestAttachmentFolder" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestAttachmentFolder" type="application/atom+xml;type=feed" title="ServiceRequestAttachmentFolder"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')/ServiceRequestBusinessTransactionDocumentReference" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestBusinessTransactionDocumentReference" type="application/atom+xml;type=feed" title="ServiceRequestBusinessTransactionDocumentReference"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')/ServiceRequestDescription" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestDescription" type="application/atom+xml;type=feed" title="ServiceRequestDescription"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')/ServiceRequestItem" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestItem" type="application/atom+xml;type=feed" title="ServiceRequestItem"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC5A0FD')/ServiceRequestTextCollection" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestTextCollection" type="application/atom+xml;type=feed" title="ServiceRequestTextCollection"/>
        <content type="application/xml">
            <m:properties>
                <d:ActivityCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:ActivityCategoryName>
                <d:ActivityServiceIssueCategoryID/>
                <d:ApprovalStatusCode/>
                <d:ApprovalStatusCodeText/>
                <d:AssignedTechnician/>
                <d:AssignedTo>8000000207</d:AssignedTo>
                <d:CauseCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:CauseCategoryName>
                <d:CauseServiceIssueCategoryID/>
                <d:ChangedBy>Eddie Wright</d:ChangedBy>
                <d:CompletedOnDate>2012-07-20T00:00:00</d:CompletedOnDate>
                <d:CompletionDueDate>2012-07-25T00:00:00</d:CompletionDueDate>
                <d:Contact>Catherine Lesjak</d:Contact>
                <d:ContractID/>
                <d:CreatedBy>Eddie Wright</d:CreatedBy>
                <d:CreationDate>2012-11-23T00:00:00</d:CreationDate>
                <d:Customer>HP</d:Customer>
                <d:CustomerID>10332</d:CustomerID>
                <d:DataOriginTypeCode>1</d:DataOriginTypeCode>
                <d:DataOriginTypeCodeText>Manual data entry</d:DataOriginTypeCodeText>
                <d:DiagnosticTest>false</d:DiagnosticTest>
                <d:ID>1423</d:ID>
                <d:IncidentCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:IncidentCategoryName>
                <d:IncidentServiceIssueCategoryID/>
                <d:InitialResponseDate>2012-07-20T00:00:00</d:InitialResponseDate>
                <d:InitialReviewDueDate>2012-07-20T00:00:00</d:InitialReviewDueDate>
                <d:InstallationPointID/>
                <d:InstalledBaseID/>
                <d:ItemListServiceRequestExecutionLifeCycleStatusCode>1</d:ItemListServiceRequestExecutionLifeCycleStatusCode>
                <d:ItemListServiceRequestExecutionLifeCycleStatusCodeText>Open</d:ItemListServiceRequestExecutionLifeCycleStatusCodeText>
                <d:LastChangeDate>2014-08-15T00:00:00</d:LastChangeDate>
                <d:LastResponseOnDate m:null="true"/>
                <d:Name>System issue at HP</d:Name>
                <d:NextResponseDueDate m:null="true"/>
                <d:ObjectCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:ObjectCategoryName>
                <d:ObjectID>00163E03A0701ED28DABB39DFBC5A0FD</d:ObjectID>
                <d:ObjectServiceIssueCategoryID/>
                <d:Partner/>
                <d:PartnerID/>
                <d:ProcessingTypeCode>SRRQ</d:ProcessingTypeCode>
                <d:ProcessingTypeCodeText>Service Request</d:ProcessingTypeCodeText>
                <d:ProductCategoryDescription>B2C - Travel</d:ProductCategoryDescription>
                <d:ProductID>P110100</d:ProductID>
                <d:ReferenceDate m:null="true"/>
                <d:ReportedForEmail/>
                <d:ReportedForPartyID/>
                <d:ReporterEmail>c.lesjak@HP.COM</d:ReporterEmail>
                <d:ReporterPartyID>1000394</d:ReporterPartyID>
                <d:RequestAssignmentStatusCode>2</d:RequestAssignmentStatusCode>
                <d:RequestAssignmentStatusCodeText>Requestor Action</d:RequestAssignmentStatusCodeText>
                <d:RequestedEnd/>
                <d:RequestedEndTimeZoneCode/>
                <d:RequestedStart/>
                <d:RequestedStartTimeZoneCode/>
                <d:SalesTerritoryID/>
                <d:ScheduledEndDate m:null="true"/>
                <d:ScheduledEndTime>PT00H00M00S</d:ScheduledEndTime>
                <d:ScheduledStartDate m:null="true"/>
                <d:ScheduledStartTime>PT00H00M00S</d:ScheduledStartTime>
                <d:SerialID/>
                <d:ServiceAndSupportTeam>S3110</d:ServiceAndSupportTeam>
                <d:ServiceCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode>E</d:languageCode>
                    <d:content>Product Support</d:content>
                </d:ServiceCategoryName>
                <d:ServiceIssueCategoryID>PS</d:ServiceIssueCategoryID>
                <d:ServiceLevelAgreement>CLOUDFORSERVICE_STANDARD</d:ServiceLevelAgreement>
                <d:ServicePriorityCode>3</d:ServicePriorityCode>
                <d:ServicePriorityCodeText>Normal</d:ServicePriorityCodeText>
                <d:ServiceRequestClassificationCode/>
                <d:ServiceRequestClassificationCodeText/>
                <d:ServiceRequestLifeCycleStatusCode>3</d:ServiceRequestLifeCycleStatusCode>
                <d:ServiceRequestLifeCycleStatusCodeText>Completed</d:ServiceRequestLifeCycleStatusCodeText>
                <d:ServiceRequestUserLifeCycleStatusCode>5</d:ServiceRequestUserLifeCycleStatusCode>
                <d:ServiceRequestUserLifeCycleStatusCodeText>Completed</d:ServiceRequestUserLifeCycleStatusCodeText>
                <d:ServiceTechnician>John Smith</d:ServiceTechnician>
                <d:ServiceTechnicianTeam/>
                <d:Technician/>
                <d:WarrantyDescription/>
                <d:WarrantyFrom m:null="true"/>
                <d:WarrantyTo m:null="true"/>
            </m:properties>
        </content>
    </entry>
    <entry>
        <id>https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/servicerequest/ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')</id>
        <title type="text">ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')</title>
        <updated>2015-10-02T23:43:16Z</updated>
        <category term="servicerequest.ServiceRequest" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')" rel="edit" title="ServiceRequest"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')/AddressSnapshotPostalAddress" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/AddressSnapshotPostalAddress" type="application/atom+xml;type=feed" title="AddressSnapshotPostalAddress"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')/ServiceRequestAttachmentFolder" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestAttachmentFolder" type="application/atom+xml;type=feed" title="ServiceRequestAttachmentFolder"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')/ServiceRequestBusinessTransactionDocumentReference" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestBusinessTransactionDocumentReference" type="application/atom+xml;type=feed" title="ServiceRequestBusinessTransactionDocumentReference"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')/ServiceRequestDescription" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestDescription" type="application/atom+xml;type=feed" title="ServiceRequestDescription"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')/ServiceRequestItem" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestItem" type="application/atom+xml;type=feed" title="ServiceRequestItem"/>
        <link href="ServiceRequestCollection('00163E03A0701ED28DABB39DFBC660FD')/ServiceRequestTextCollection" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestTextCollection" type="application/atom+xml;type=feed" title="ServiceRequestTextCollection"/>
        <content type="application/xml">
            <m:properties>
                <d:ActivityCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:ActivityCategoryName>
                <d:ActivityServiceIssueCategoryID/>
                <d:ApprovalStatusCode/>
                <d:ApprovalStatusCodeText/>
                <d:AssignedTechnician/>
                <d:AssignedTo>8000000197</d:AssignedTo>
                <d:CauseCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:CauseCategoryName>
                <d:CauseServiceIssueCategoryID/>
                <d:ChangedBy>Juliane Beyer</d:ChangedBy>
                <d:CompletedOnDate>2012-06-09T00:00:00</d:CompletedOnDate>
                <d:CompletionDueDate>2012-06-13T00:00:00</d:CompletionDueDate>
                <d:Contact>Frank Sander</d:Contact>
                <d:ContractID/>
                <d:CreatedBy>Eddie Wright</d:CreatedBy>
                <d:CreationDate>2012-11-23T00:00:00</d:CreationDate>
                <d:Customer>Digital Plus AG</d:Customer>
                <d:CustomerID>10025</d:CustomerID>
                <d:DataOriginTypeCode>1</d:DataOriginTypeCode>
                <d:DataOriginTypeCodeText>Manual data entry</d:DataOriginTypeCodeText>
                <d:DiagnosticTest>false</d:DiagnosticTest>
                <d:ID>1422</d:ID>
                <d:IncidentCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:IncidentCategoryName>
                <d:IncidentServiceIssueCategoryID/>
                <d:InitialResponseDate>2012-06-09T00:00:00</d:InitialResponseDate>
                <d:InitialReviewDueDate>2012-06-11T00:00:00</d:InitialReviewDueDate>
                <d:InstallationPointID/>
                <d:InstalledBaseID/>
                <d:ItemListServiceRequestExecutionLifeCycleStatusCode>1</d:ItemListServiceRequestExecutionLifeCycleStatusCode>
                <d:ItemListServiceRequestExecutionLifeCycleStatusCodeText>Open</d:ItemListServiceRequestExecutionLifeCycleStatusCodeText>
                <d:LastChangeDate>2014-05-20T00:00:00</d:LastChangeDate>
                <d:LastResponseOnDate m:null="true"/>
                <d:Name>System issue at Digital Plus AG</d:Name>
                <d:NextResponseDueDate>2012-08-05T00:00:00</d:NextResponseDueDate>
                <d:ObjectCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode/>
                    <d:content/>
                </d:ObjectCategoryName>
                <d:ObjectID>00163E03A0701ED28DABB39DFBC660FD</d:ObjectID>
                <d:ObjectServiceIssueCategoryID/>
                <d:Partner/>
                <d:PartnerID/>
                <d:ProcessingTypeCode>SRRQ</d:ProcessingTypeCode>
                <d:ProcessingTypeCodeText>Service Request</d:ProcessingTypeCodeText>
                <d:ProductCategoryDescription>B2C - HT - Imaging</d:ProductCategoryDescription>
                <d:ProductID>P600103</d:ProductID>
                <d:ReferenceDate m:null="true"/>
                <d:ReportedForEmail/>
                <d:ReportedForPartyID/>
                <d:ReporterEmail>Frank.Sander@ONDEMAND.COM</d:ReporterEmail>
                <d:ReporterPartyID>1000118</d:ReporterPartyID>
                <d:RequestAssignmentStatusCode>2</d:RequestAssignmentStatusCode>
                <d:RequestAssignmentStatusCodeText>Requestor Action</d:RequestAssignmentStatusCodeText>
                <d:RequestedEnd/>
                <d:RequestedEndTimeZoneCode/>
                <d:RequestedStart/>
                <d:RequestedStartTimeZoneCode/>
                <d:SalesTerritoryID/>
                <d:ScheduledEndDate m:null="true"/>
                <d:ScheduledEndTime>PT00H00M00S</d:ScheduledEndTime>
                <d:ScheduledStartDate m:null="true"/>
                <d:ScheduledStartTime>PT00H00M00S</d:ScheduledStartTime>
                <d:SerialID/>
                <d:ServiceAndSupportTeam>S3111</d:ServiceAndSupportTeam>
                <d:ServiceCategoryName m:type="servicerequest.MEDIUM_Name">
                    <d:languageCode>E</d:languageCode>
                    <d:content>General Usage Issue</d:content>
                </d:ServiceCategoryName>
                <d:ServiceIssueCategoryID>PS-GEN</d:ServiceIssueCategoryID>
                <d:ServiceLevelAgreement>CLOUDFORSERVICE_SLA_VIP</d:ServiceLevelAgreement>
                <d:ServicePriorityCode>2</d:ServicePriorityCode>
                <d:ServicePriorityCodeText>Urgent</d:ServicePriorityCodeText>
                <d:ServiceRequestClassificationCode/>
                <d:ServiceRequestClassificationCodeText/>
                <d:ServiceRequestLifeCycleStatusCode>2</d:ServiceRequestLifeCycleStatusCode>
                <d:ServiceRequestLifeCycleStatusCodeText>In Process</d:ServiceRequestLifeCycleStatusCodeText>
                <d:ServiceRequestUserLifeCycleStatusCode>4</d:ServiceRequestUserLifeCycleStatusCode>
                <d:ServiceRequestUserLifeCycleStatusCodeText>In Process - Waiting for Customer</d:ServiceRequestUserLifeCycleStatusCodeText>
                <d:ServiceTechnician/>
                <d:ServiceTechnicianTeam/>
                <d:Technician/>
                <d:WarrantyDescription/>
                <d:WarrantyFrom m:null="true"/>
                <d:WarrantyTo m:null="true"/>
            </m:properties>
        </content>
    </entry>
</feed>
--ejjeeffe0
Content-Type: multipart/mixed; boundary=ejjeeffe1
Content-Length:      2329

--ejjeeffe1
Content-Type: application/http
Content-Length: 96
Content-Transfer-Encoding: binary

HTTP/1.1 204 No Content
Content-Type: text/html
Content-Length: 0
dataserviceversion: 2.0


--ejjeeffe1
Content-Type: application/http
Content-Length: 2008
Content-Transfer-Encoding: binary

HTTP/1.1 201 Created
Content-Type: application/atom+xml;type=entry
Content-Length: 1716
location: https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/servicerequest/ServiceRequestItemCollection('00163E0DBD9E1ED59AABEB57FF4A3F4B')
dataserviceversion: 2.0
cache-control: no-cache, no-store


<?xml version="1.0" encoding="utf-8"?>
<entry xml:base="https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/servicerequest/" xmlns="http://www.w3.org/2005/Atom" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices">
    <id>https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/servicerequest/ServiceRequestItemCollection('00163E0DBD9E1ED59AABEB57FF4A3F4B')</id>
    <title type="text">ServiceRequestItemCollection('00163E0DBD9E1ED59AABEB57FF4A3F4B')</title>
    <updated>2015-10-02T23:43:20Z</updated>
    <category term="servicerequest.ServiceRequestItem" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
    <link href="ServiceRequestItemCollection('00163E0DBD9E1ED59AABEB57FF4A3F4B')" rel="edit" title="ServiceRequestItem"/>
    <link href="ServiceRequestItemCollection('00163E0DBD9E1ED59AABEB57FF4A3F4B')/ServiceRequest" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequest" type="application/atom+xml;type=entry" title="ServiceRequest"/>
    <link href="ServiceRequestItemCollection('00163E0DBD9E1ED59AABEB57FF4A3F4B')/ServiceRequestFirstRequestedItemScheduleLine" rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/ServiceRequestFirstRequestedItemScheduleLine" type="application/atom+xml;type=entry" title="ServiceRequestFirstRequestedItemScheduleLine"/>
    <content type="application/xml">
        <m:properties>
            <d:Description>1m water hose</d:Description>
            <d:ObjectID>00163E0DBD9E1ED59AABEB57FF4A3F4B</d:ObjectID>
            <d:ParentObjectID>00163E0DBD9E1ED596EBDFDA564728AC</d:ParentObjectID>
            <d:ProductCategoryDescription>Parts</d:ProductCategoryDescription>
            <d:ProductID>10000760</d:ProductID>
        </m:properties>
    </content>
</entry>
--ejjeeffe1--

--ejjeeffe0--
```


####$expand

C4C OData API's support for $expand system query option is via Navigaton Properties. 

E.g. `/AccountCollection?$top=10&$format=json&$expand=AccountMainAddress` Where AccountMainAddress is a Navigation Property defined in the Entity Data Model (EDM) for the Account Entity (see the Entity defintion below).

```XML
			<EntityType Name="Account">
				<Key>
					<PropertyRef Name="ObjectID"/>
				</Key>
				<Property Name="ABCClassificationCode" Type="Edm.String" Nullable="true" MaxLength="1" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="ABCClassificationCodeText" Type="Edm.String" Nullable="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="AccountFormattedName" Type="Edm.String" Nullable="false" MaxLength="40" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="AccountID" Type="Edm.String" Nullable="true" MaxLength="10" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="AccountName" Type="Edm.String" Nullable="true" MaxLength="240" FixedLength="true" sap:creatable="false" sap:updatable="true" sap:filterable="false"/>
....
				<NavigationProperty Name="AccountMainAddress" Relationship="http://sap.com/xi/AP/CRM/Global.Account_AccountMainAddress" FromRole="Account" ToRole="AccountMainAddress"/>
				<NavigationProperty Name="AccountNotes" Relationship="http://sap.com/xi/AP/CRM/Global.Account_AccountNotes" FromRole="Account" ToRole="AccountNotes"/>
				<NavigationProperty Name="AccountRole" Relationship="http://sap.com/xi/AP/CRM/Global.Account_AccountRole" FromRole="Account" ToRole="AccountRole"/>
				<NavigationProperty Name="AccountSalesData" Relationship="http://sap.com/xi/AP/CRM/Global.Account_AccountSalesData" FromRole="Account" ToRole="AccountSalesData"/>
				<NavigationProperty Name="AccountTeam" Relationship="http://sap.com/xi/AP/CRM/Global.Account_AccountTeam" FromRole="Account" ToRole="AccountTeam"/>
				<NavigationProperty Name="ExternalIDMapping" Relationship="http://sap.com/xi/AP/CRM/Global.Account_ExternalIDMapping" FromRole="Account" ToRole="ExternalIDMapping"/>
			</EntityType>
```			

For every entity (that is not a Header entity e.g. Account or Opportunity etc.) the C4C OData framework implicitly generates a Navigation Property to the parent entity as well as a Navigation Property to the Header entity (if the parent entity is not a Header itself). 

If you consider the metadata below, the following can be seen:

  * Opportunity is the Header entity
  * OpportunityItem has the Opportunity as the Parent (as well as the Header) and 
  * OpportunityItemRevenuePlanReporting has OpportunityItem as the Parent and Opportunity as the Header entity

For each of the entities below, a Navigation Property is available that allows a given Entity type to navigate to the immediate parent e.g. OpportunityItemRevenuePlanReporting has a 

  * Navigation Property Opportunity that allows navigation to the Header and 
  * Navigation Property OpportunityItem that allows navigation to the Parent.

This pattern holds true for all OData services generated by C4C (both standard as well as custom services).

```XML
			<EntityType Name="Opportunity">
				<Key>
					<PropertyRef Name="ObjectID"/>
				</Key>
				<Property Name="ApprovalStatusCode" Type="Edm.String" Nullable="true" MaxLength="2" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="ConsistencyStatusCode" Type="Edm.String" Nullable="true" MaxLength="2" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="ID" Type="Edm.String" Nullable="true" MaxLength="35" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="LifeCycleStatusCode" Type="Edm.String" Nullable="true" MaxLength="2" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="Name" Type="Edm.String" Nullable="true" MaxLength="765" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="ObjectID" Type="Edm.String" Nullable="true" MaxLength="70" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="OriginTypeCode" Type="Edm.String" Nullable="true" MaxLength="3" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="PhaseProgressEvaluationStatusCode" Type="Edm.String" Nullable="true" MaxLength="2" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="PriorityCode" Type="Edm.String" Nullable="true" MaxLength="1" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="ProcessingTypeCode" Type="Edm.String" Nullable="true" MaxLength="4" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<NavigationProperty Name="OpportunityItem" Relationship="cust.Opportunity_OpportunityItem" FromRole="Opportunity" ToRole="OpportunityItem"/>
			</EntityType>
			<EntityType Name="OpportunityItem">
				<Key>
					<PropertyRef Name="ObjectID"/>
				</Key>
				<Property Name="ID" Type="Edm.String" Nullable="true" MaxLength="10" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="NetAmount" Type="cust.Amount" Nullable="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="ObjectID" Type="Edm.String" Nullable="true" MaxLength="70" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="ParentObjectID" Type="Edm.String" Nullable="true" MaxLength="70" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="ProductID" Type="Edm.String" Nullable="true" MaxLength="60" FixedLength="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="Quantity" Type="cust.Quantity" Nullable="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<NavigationProperty Name="Opportunity" Relationship="cust.Opportunity_OpportunityItem" FromRole="OpportunityItem" ToRole="Opportunity"/>
				<NavigationProperty Name="OpportunityItemRevenuePlanReporting" Relationship="cust.OpportunityItem_OpportunityItemRevenuePlanReporting" FromRole="OpportunityItem" ToRole="OpportunityItemRevenuePlanReporting"/>
			</EntityType>
			<EntityType Name="OpportunityItemRevenuePlanReporting">
				<Key>
					<PropertyRef Name="ObjectID"/>
				</Key>
				<Property Name="DistributionAmount" Type="cust.Amount" Nullable="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="DistributionDate" Type="Edm.DateTime" Nullable="true" Precision="8" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="ObjectID" Type="Edm.String" Nullable="true" MaxLength="70" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="ParentObjectID" Type="Edm.String" Nullable="true" MaxLength="70" FixedLength="true" sap:creatable="false" sap:updatable="false" sap:filterable="false"/>
				<Property Name="ReportingCurrencyDistributionAmount" Type="cust.Amount" Nullable="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<Property Name="RevenuePartnerUUID" Type="Edm.Guid" Nullable="true" sap:creatable="true" sap:updatable="true" sap:filterable="false"/>
				<NavigationProperty Name="Opportunity" Relationship="cust.Opportunity_OpportunityItemRevenuePlanReporting" FromRole="OpportunityItemRevenuePlanReporting" ToRole="Opportunity"/>
				<NavigationProperty Name="OpportunityItem" Relationship="cust.OpportunityItem_OpportunityItemRevenuePlanReporting" FromRole="OpportunityItemRevenuePlanReporting" ToRole="OpportunityItem"/>
			</EntityType>
```			
Please note that C4C OData API currently **DOES NOT** support the usage of properties from expanded navigations as part of $filter conditions. However, this can be achieved by applying the $filter condition at the child entity collection and then use an $expand condition to navigate to either the Parent collection or to the Header collection. 
E.g. if the requirement is to get all Opportunities that have a certain Product, as a part of a single GET request, this could be achieved with the following query call:

```
/OpportunityItemCollection?$format=json&$filter=ProductID eq 'P300104'&$expand=Opportunity
```


####$filter

Option | Example | Description
-------|---------|------------
eq | /OpportunityCollection?$filter=AccountID eq '1001910' | Gets all Opportunity entries that matches the specified AccountID
endswith | /AccountCollection?$filter=endswith(AccountName,'LLC') | All accounts whose AccountName ends with 'LLC'. **_Note that the Property Name has to be specified first_**.
startswith | /AccountCollection?$filter=startswith(AccountName,'Porter') | All accounts whose AccountName starts with 'Porter'. **_Similar to endswith note that the Property Name has to be specified first_**.


Please note that currently C4C OData API **DOES NOT** support usage of String, Date and Math Functions in $filter System Query Option.


####$inlinecount

XML response with inlinecount. The Element <m:count> contains the response to the $inlinecount.

```XML
<feed xmlns="http://www.w3.org/2005/Atom" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xml:base="https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/c4codata/">
	<id>https://myNNNNNN.crm.ondemand.com/sap/c4c/odata/v1/c4codata/OpportunityCollection</id>
	<title type="text">OpportunityCollection</title>
	<updated>2015-08-23T17:30:32Z</updated>
	<author>
		<name/>
	</author>
	<link href="OpportunityCollection" rel="self" title="OpportunityCollection"/>
	<m:count>39080</m:count>
	<entry>
		<id>https://myNNNNN.crm.ondemand.com/sap/c4c/odata/v1/c4codata/OpportunityCollection('00163E03A0701ED28BCEC7F4AA474109')</id>
		<title type="text">OpportunityCollection('00163E03A0701ED28BCEC7F4AA474109')</title>
		<updated>2015-08-23T17:30:32Z</updated>
		....
```

JSON response with inlinecount. The attribute __count contains the response to the $inlinecount.

```JSON
{
  "d": {
    "__count": "39080", 
    "results": [
      {
        "AccountID": "10009", 
        "AccountName": {
          "__metadata": {
            "type": "http://sap.com/xi/AP/CRM/Global.ENCRYPTED_LONG_Name"
          }, 
          "content": "Primo Sustainable products", 
          "languageCode": "E"
        }, 
        ...
```

####$search
Although $search is not part of OData V2 specification, $search is supported by C4C OData API. Once $search is performed, C4C OData API compares the term provided to $search against the properties marked as $search relevant in OData Service Expolorer. Standard C4C OData services are delievered with $search relevant properties marked. 

For custom OData services, it is possible to mark $search releveant entity properties individually as well as at the entity collection level. 

The following example shows the usage of the $search:

```
https://myNNNNNN.crm.ondemand.com/sap/byd/odata/cust/v1/c4codata/CustomerCollection?$search='Porter'
```

###Sample Payloads

  * [Service Ticket](sections/serviceticket.md)
  * [Mass query pattern](sections/massquery.md)




