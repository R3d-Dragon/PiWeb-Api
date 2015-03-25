---
category: dataservice
subCategory: inspection-plan
title: Data Service
subTitle: Inspection Plan
isSubPage: true
permalink: /dataservice/inspection-plan/
sections:
  general: General Information
  endpoint: REST API Endpoints
  sdk: .NET SDK Methods
---

## {{ page.sections['general'] }}

Both parts and characteristics are PiWeb inspection plan entities. They are either ```InspectionPlanPart``` or ```InspectionPlanCharacteristic```objects. Both are derived from ```InspectionPlanBase```.

### InspectionPlanBase

Name | Type | Description
-----|------|--------------
uuid | ```Guid``` | Identifies this inspection plan entity uniquely.
path | ```PathInformation``` | The path of this entity.
attributes | ```Attribute``` | A set of attributes which specifies this entity.
comment | ```string``` | A comment which describes the last inspection plan change.
version | ```int``` | Contains the revision number of the entity. The revision number starts with zero and is incremented by one each time when changes are applied to the inspection plan. The version is only returned in case versioning is enabled in the server settings.
current | ```bool``` | Indicates whether the entity is the current version.
timeStamp | ```dateTime``` | Contains the date and time of the last update applied to this entity.

### InspectionPlanPart : InspectionPlanBase

charChangeDate |  ```dateTime``` | The timestamp for the most recent characteristic change on any characteristic that belongs to this part

### InspectionPlanCharacteristic : InspectionPlanBase

{% comment %}----------------------------------------------------------------------------------------------- {% endcomment %}

## {{ page.sections['endpoint'] }}

Parts and characteristics can be fetched, created, updated and deleted via the following endpoints. Filters which restrict GET or DELETE requests can be set as described in the [URL-Parameter section]({{site.baseurl }}/general/restapi/#{{ page.subCategory }}).

###Parts

URL Endpoint | GET | POST | PUT | DELETE
-------------|-----|-----|------|-------
/parts | Returns all parts | Creates the committed part(s) which is/are transfered in the body of the request | Updates the committed parts | Deletes all parts
/parts/:partPath | Returns the part specified by *:partPath* as well as the parts beneath this part | *--* | *--* | Deletes the part specified by *:partPath* as well as the parts and characteristics beneath this part
parts/(:uuidList) | Returns all parts of which the uuid is within the *:uuidList* | *--* | *--* |  Deletes all parts of which the uuid is within the *:uuidList* as well as the parts and characteristics beneath the particular part

### Characteristics

URL Endpoint | GET | POST | PUT | DELETE
-------------|-----|-----|------|-------
parts/characteristics | Returns all characteristics | Creates the committed characteristic(s) which is/are transfered in the body of the request | Updates the committed characteristics | *--*
parts/:partsPath/characteristics | Returns all characteristics beneath the part specified by *:partPath* | *--* | *--* | *--*
parts/characteristics/:characteristicPath | Returns the characteristic specified by *:characteristicPath*. | *--* | *--* | Deletes the characteristic specified by *:characteristicPath* as well as all children beneath this characteristic
parts/characteristics/(:uuidList) | Returns all characteristics of which the uuid is within the *:uuidList* | *--* | *--* |  Deletes all characteristics of which the uuid is within the *:uuidList*

{% comment %}----------------------------------------------------------------------------------------------- {% endcomment %}

## Get Entities

Fetching inspection plan entites returns the respective parts or characteristics depending on the specified entity constraint and/or filter. 
The most important filter parameter is the *depth* parameter which controls down to which level of the inspection plan the entities should be fetched. Setting *depth:0* means that only the entity itself should be fetched, *depth:1* means the entity and its direct children should be fetched and so on.

Parts can be fetched in several ways:

* fetch a certain part by its path (the filter parameter *depth* must be 0)
* fetch a certain part and its children by its path (the filter parameter *depth* must be ≥ 1)
* fetch one or more certain parts by their UUIDs
* fetch all parts (can be restricted with filter parameters)

There are also several possibilities to fetch characteristics:

* fetch a certain characteristic by its path (the filter parameter *depth* must be 0)
* fetch one or more certain characteristics by their UUIDs
* fetch characteristics beneath a certain part path
* fetch all characteristics (can be restricted with filter parameters)

{% assign exampleCaption="Fetch the direct characteristics beneath the 'metal part'. Restrict to attribute keys 2110 and 2111" %}
{% assign comment="As the filter parameter *depth* has the default value 1, it can be omitted in this example." %}

{% capture jsonrequest %}
{% highlight http %}
GET /dataServiceRest/parts/metal%20part/characteristics?filter=characteristicAttributes:{2110,2111} HTTP/1.1
{% endhighlight %}
{% endcapture %}

{% capture jsonresponse %}
{% highlight json %}
{
   ...
   "data":
   [
       {
           "path": "PC:/metal part/diameter_circle3/",
           "attributes":
           {
               "2110": "-0.2",
               "2111": "0.3",
           },
           "uuid": "1429c5e2-599c-4d3e-b724-4e00ecb0caa7",
           "version": 0,
           "timestamp": "2012-11-19T10:48:32.887Z",
           "current": true
       },
       ...
   ]
}
{% endhighlight %}
{% endcapture %}

{% include exampleFieldset.html %}
{% assign comment="" %}


## Add Entities

To create an inspection plan entity it is necessary to transfer the entity object within the request's body. A unique identifier and the path are mandatory, attributes and a comment are optional. The attribute keys which are used for the attributes must come from the parts/characteristics attribute range (specified in the {{ site.links['configuration'] }})

{{ site.images['info'] }} The comment is only added if versioning is enabled in the server settings.

{% assign exampleCaption="Adding the 'metal part' part with the uuid 05040c4c-f0af-46b8-810e-30c0c00a379e" %}

{% capture jsonrequest %}
{% highlight http %}
POST /dataServiceRest/parts HTTP/1.1
{% endhighlight %}

{% highlight json %}
[
  {
    "uuid": "05040c4c-f0af-46b8-810e-30c0c00a379e",
    "path": "P:/metal part",
    "attributes": 
    {
      "1001": "4466",
      "1003": "mp"
    }       
  }
]
{% endhighlight %}
{% endcapture %}

{% capture jsonresponse %}
{% highlight http %}
HTTP/1.1 201 Created
{% endhighlight %}

{% highlight json %}
{
   "status":
   {
       "statusCode": 201,
       "statusDescription": "Created"
   },
   "category": "Success"
}
{% endhighlight %}
{% endcapture %}

{% include exampleFieldset.html %}

## Update entities

Updating inspection plan entities might regard the following aspects: 

* Rename/move inspection plan entities
* Change inspection plan entity's attributes

{{site.images['info']}} If versioning is activated on server side, every update of one or more inspection plan entities creates a new version entry.

{% assign exampleCaption="Rename the characteristic "metal part/diameter_circle3" to "metal part/diameterCircle3" %}
{% capture jsonrequest %}
{% highlight http %}
PUT /dataServiceRest/parts/characteristics HTTP/1.1
{% endhighlight %}

{% highlight json %}
[
  {
     "path": "PC:/metal part/diameterCircle3/",
     "attributes": { "2110": "-0.2", "2111": "0.3", },
     "uuid": "1429c5e2-599c-4d3e-b724-4e00ecb0caa7",
     "version": 0,
     "timestamp": "2012-11-19T10:48:32.887Z",
     "current": true
  }
]
{% endhighlight %}
{% endcapture %}

{% capture jsonresponse %}
{% highlight http %}
HTTP/1.1 200 Ok
{% endhighlight %}

{% highlight json %}
{
   "status":
   {
       "statusCode": 200,
       "statusDescription": "Ok"
   },
   "category": "Success"
}
{% endhighlight %}
{% endcapture %}

{% include exampleFieldset.html %}

## Delete Entities

There are two possibilities to delete inspection plan entities, either by their path or by their uuid. In both cases the entity itself as well as all children are deleted.

{% assign exampleCaption="Delete the part 'metal part'  and all entities beneath it" %}
{% capture jsonrequest %}
{% highlight http %}
DELETE /dataServiceRest/parts/metal%20part HTTP/1.1
{% endhighlight %}
{% endcapture %}

{% capture jsonresponse %}
{% highlight http %}
HTTP/1.1 200 Ok
{% endhighlight %}

{% highlight json %}
{
   "status":
   {
       "statusCode": 200,
       "statusDescription": "Ok"
   },
   "category": "Success"
}
{% endhighlight %}
{% endcapture %}

{% include exampleFieldset.html %}

{% comment %}----------------------------------------------------------------------------------------------- {% endcomment %}

## {{ page.sections['sdk'] }}

### Get Entities

#### Parts

{% assign caption="GetPartByPath" %}
{% assign icon=site.images['function-get'] %}
{% assign description="Fetches the part identified by the ```partPath```. " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
partPath       | ```PathInformation```                 | The path of the part which should be returned.
filter         | ```InspectionPlanFilterAttributes ``` | Parameter is optional and is used to restrict the query.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign returnParameter="Task<InspectionPlanPart>" %}

{% assign exampleCaption="Get the 'metal part' and restrict the result to the attributes 'part number' and 'comment'" %}

{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
var partPath = PathHelper.String2PartPathInformation("/metal part");
var filter = new InspectionPlanFilterAttributes()
  { RequestedPartAttributes = new AttributeSelector(
    new[]{ WellKnownKeys.Parts.Number, WellKnownKeys.Parts.Comment } )
  };
var part = client.GetCharacteristicByPath( partPath, filter );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

{% assign caption="GetPartsByUuids" %}
{% assign icon=site.images['function-get'] %}
{% assign description="Fetches parts by its uuids. " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
uuids          | ```Guid[]```                          | The uuids of the parts which should be returned.
filter         | ```InspectionPlanFilterAttributes ``` | Parameter is optional and is used to restrict the query.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Get the 'metal part' by its uuid 05040c4c-f0af-46b8-810e-30c0c00a379e" %}

{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
var part = client.GetCharacteristicsByUuids( new[]{ new Guid("05040c4c-f0af-46b8-810e-30c0c00a379e") } );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

{% assign caption="GetChildPartsForPart" %}
{% assign icon=site.images['function-get'] %}
{% assign description="Fetches the children of the given ```parentPart``` as well as the part itself. " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
parentPart     | ```PathInformation```                 | The parent part the child part should be returned for.
filter         | ```InspectionPlanFilterAttributes ``` | Parameter is optional and is used to restrict the query.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Get the 'metal part' and its child parts" %}

{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
var parentPartPath = PathHelper.String2PartPathInformation("/metal part");
var part = client.GetPartsForPart( parentPartPath );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

#### Characteristics

{% assign caption="GetCharacteristicByPath" %}
{% assign icon=site.images['function-get'] %}
{% assign description="Fetches the characteristic identified by the ```partPath```. " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
partPath       | ```PathInformation```                 | The path of the characteristic which should be returned.
filter         | ```InspectionPlanFilterAttributes ``` | Parameter is optional and is used to restrict the query.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Get the characteristic 'metal part/diameterCircle3'" %}

{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
var charPath = PathHelper.String2PathInformation( "/metal part/diameterCircle3", "PC" );
var characteristic = client.GetCharacteristicByPath( charPath );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

{% assign caption="GetCharacteristicsByUuids" %}
{% assign icon=site.images['function-get'] %}
{% assign description="Fetches characteristics by its uuids. " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
uuids          | ```Guid[]```                          | The uuids of the characteristics which should be returned.
filter         | ```InspectionPlanFilterAttributes ``` | Parameter is optional and is used to restrict the query.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Get the 'metal part/diameterCircle3' by its uuid 1429c5e2-599c-4d3e-b724-4e00ecb0caa7" %}

{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
var characteristic = client.GetCharacteristicsByUuids( new[]{ new Guid("1429c5e2-599c-4d3e-b724-4e00ecb0caa7") } );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

{% assign caption="GetCharacteristicsForPart" %}
{% assign icon=site.images['function-get'] %}
{% assign description="Fetches the characteristics below the given ```parentPart```. If the ```depth``` property of the ```filter``` parameter is not set, only the direct children are returned by default. " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
parentPart     | ```PathInformation```                 | The parent part the charateristics should be returned for.
filter         | ```InspectionPlanFilterAttributes ``` | Parameter is optional and is used to restrict the query.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Get the characteristics below 'metal part'. Restrict the result to lower and upper tolerance attributes." %}

{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
var parentPartPath = PathHelper.String2PartPathInformation("/metal part");
var filter = new InspectionPlanFilterAttributes()
  { RequestedCharacteristicAttributes = new AttributeSelector(
    new[]{ WellKnownKeys.Characteristic.LowerSpecificationLimit, 
    WellKnownKeys.Characteristic.UpperSpecificationLimit} )
  };
var characteristics = client.GetCharacteristicsForPart( parentPartPath, filter );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

### Add Entities

#### Parts

{% assign caption="CreateParts" %}
{% assign icon=site.images['function-create'] %}
{% assign description="Creates the parts which are included in ```parts``` " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
parts          | ```InspectionPlanPart[]```            | The parts that should be created.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Add the part 'metal part'." %}
{% capture example %}
{% highlight csharp %}
var part = new InspectionPlanPart{ 
  Uuid = new Guid( "05550c4c-f0af-46b8-810e-30c0c00a379e" ),
  Path = PathHelper.String2PartPathInformation( "metal part"),
  Attributes = new[]{ 
    new Attribute( WellKnownKeys.Parts.Number, "4466" ), 
    new Attribute( WellKnownKeys.Parts.Abbreviation, "mp" ) }
};
var client = new DataServiceRestClient( "http://piwebserver:8080" );
client.CreateParts( new[]{ part } );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

#### Characteristics

{% assign caption="CreateCharacteristics" %}
{% assign icon=site.images['function-create'] %}
{% assign description="Creates the characteristics which are included in ```characteristics``` " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
characteristics| ```InspectionPlanCharacteristic[]```  | The characteristics that should be created.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Add the characteristic '/metal part/diameterCircle3'." %}

{% capture example %}
{% highlight csharp %}
var characteristic = new InspectionPlanPart{ 
  Uuid = new Guid( "1429c5e2-599c-4d3e-b724-4e00ecb0caa7" ),
  Path =  PathHelper.String2PathInformation( "/metal part/diameterCircle3", "PC" ),
  Attributes = new[]{ 
    new Attribute( WellKnownKeys.Characteristic.LowerSpecificationLimit, "-0.2" ), 
    new Attribute( WellKnownKeys.Characteristic.UpperSpecificationLimit, "0.3" ) }
};
var client = new DataServiceRestClient( "http://piwebserver:8080" );
client.CreateCharacteristics( new[]{ characteristic } );
{% endhighlight %}
{% endcapture %}

### Update Entities

#### Parts

{% assign caption="UpdateParts" %}
{% assign icon=site.images['function-update'] %}
{% assign description="Update the parts which are included in ```parts```. Updating might regard the following: rename/move inspection plan entities or change inspection plan entity’s attributes. If versioning is enabled on server side, every update of one or more inspection plan entities creates a new version entry." %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
parts          | ```InspectionPlanPart[]```            | The parts that should be updated.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Update the 'metal part' part's attribute 'part number'." %}
{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );

//Get the part
var charPath = PathHelper.String2PathInformation( "/metal part/diameterCircle3", "PC" );
var part = client.GetPartByPath( charPath );

part.Attributes[ 0 ] = new global::DataService.Attribute( WellKnownKeys.Part.Number, "446" );
client.UpdateParts( new[]{ part } );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

#### Characteristics

{% assign caption="UpdateCharacteristics" %}
{% assign icon=site.images['function-update'] %}
{% assign description="Update the characteristics which are included in ```characteristics``` Updating might regard the following: rename/move inspection plan entities or change inspection plan entity’s attributes. If versioning is enabled on server side, every update of one or more inspection plan entities creates a new version entry. " %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
characteristics| ```InspectionPlanCharacteristic[]```  | The characteristics that should be updated.
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Rename/move the characteristic '/metal part/diameterCircle3' to '/metal part/diameterCircle_3'." %}

{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );

//Get the characteristic
var charPath = PathHelper.String2PathInformation( "/metal part/diameterCircle3", "PC" );
var characteristic = client.GetCharacteristicByPath( charPath );

var newPath = PathHelper.String2PathInformation( "/metal part/diameterCircle_3", "PC" );
characteristic.Path = newPath;
client.UpdateCharacteristics( new InspectionPlanCharacteristic[]{characteristic} );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

### Delete Entities

#### Parts

{% assign caption="DeleteAllParts" %}
{% assign icon=site.images['function-delete'] %}
{% assign description="Deletes all parts within the database and therefore the whole inspection plan." %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Delete the whole inspection plan." %}
{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
client.DeleteAllParts();
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

{% assign caption="DeleteParts" %}
{% assign icon=site.images['function-delete'] %}
{% assign description="Deletes all parts (and characteristics) below the part ```startPart``` including the part itself." %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
startPart      | ```PathInformation```                 | The path of the part which should be deleted. 
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Delete everything below 'metal part' including the part itself." %}
{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
client.DeleteParts( PathHelper.String2PartPathInformation( "metal part" ) );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

{% assign caption="DeleteParts" %}
{% assign icon=site.images['function-delete'] %}
{% assign description="Deletes all parts (and characteristics) of which the uuid is within ```guids```, including the parts themselves." %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
guids          | ```PathInformation```                 | The guids of the parts which shall be deleted. 
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Delete the part with the uuid 05550c4c-f0af-46b8-810e-30c0c00a379e." %}
{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
client.DeleteParts( new []{ new Guid ( "05550c4c-f0af-46b8-810e-30c0c00a379e" ) } );
{% endhighlight %}
{% endcapture %}

#### Characteristics

{% assign caption="DeleteCharacteristics" %}
{% assign icon=site.images['function-delete'] %}
{% assign description="Deletes all characteristics below the characteristic ```startPart```, including the characteristic itself." %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
startPart      | ```PathInformation```                 | The path of the characteristic which should be deleted. 
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Delete everything below 'diameterCircle_3' including the characteristic itself." %}
{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
client.DeleteCharacteristics( PathHelper.String2PathInformation( "/metal part/diameterCircle_3", "PC" ) );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}

{% assign caption="DeleteCharacteristics" %}
{% assign icon=site.images['function-delete'] %}
{% assign description="Deletes all characteristics of which the uuid is within ```guids```, including the characteristics themselves." %}
{% capture parameterTable %}
Name           | Type                                  | Description
---------------|---------------------------------------|--------------------------------------------------
guids          | ```PathInformation```                 | The guids of the characteristics which shall be deleted. 
token          | ```CancellationToken```               | Parameter is optional and allows to cancel the asyncronous call.
{% endcapture %}

{% assign exampleCaption="Delete the characteristic with the uuid 1429c5e2-599c-4d3e-b724-4e00ecb0caa7." %}
{% capture example %}
{% highlight csharp %}
var client = new DataServiceRestClient( "http://piwebserver:8080" );
client.DeleteCharacteristics( new []{ new Guid ( "1429c5e2-599c-4d3e-b724-4e00ecb0caa7" ) } );
{% endhighlight %}
{% endcapture %}

{% include sdkFunctionFieldset.html %}