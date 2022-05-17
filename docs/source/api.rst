API 
===

.. autosummary::
   :toctree: generated

   lumache

Intro
-----
The API is built in ASP .NET Core (.NET 6) with C#. ASP stands for Active Server Pages and is a development framework for welding webpages and executing scripts on a web server. This code framework is anyway more similar to .NET 5 as far as it seems to me (i.e. there is still a Startup.cs file which is not necessary anymore in .NET 6).
Startup.cs is about registering services and injection of modules in HTTP Pipeline. It defines class Startup that is triggered when application launches and in each HTTP request/response.


Language: C#
------------

The code is written in C#, an object oriented language. A basic introduction is given at https://www.w3schools.com/cs/index.php . In terms of syntax, is similar to C++, but more flexible, meaning it allows (and probably incentives) the programer to define everything in a class, which can have different attributes 
(public, private, …). 
The “vocabulary” is defined within namespaces, which can be both built-in or defined by user in other script of the same folder, that can be imported within a script with 

.. code-block:: csharp

   using System;
   using FaunaPhotonics.Data.Api.Activity;


Program.cs
----------

A .NET Core application runs inside a host that handles the application startup, web server configuration etc., as well as resources like logging, dependency injection and any IHostedService implementations. A host is created, configured, and executed using the code in Program.cs. 
The  ``Main()`` method is the entry point of the application. The host is created calling the ``Build().Run()`` extension method in a  ``CreateDefaultBuilder `` method. The  ``ConfigureWebHostDefaults() `` method can be used to configure a web application host .In our case it refers to Startup


Startup.cs
----------
The Startup class contains the ConfigureService (which is used to configure the required services) and Configure (used to configure the request processing pipeline) methods. When the application starts, the first method is executed and the second too, immediately after. In our API i.e., in ConfigureService we add some services defined in the built-in interface  ``IServiceCollection`` for the monitoring and memory caching, plus the swagger documentation, the tools to access data (i.e. databases and repos) and our user-defined services (related to insect activity i.e.).
The Configure method is quite default. The UseRouting and UseEndPoints methods are used to add and configure middleware to the request processing pipeline

Http request
------------

The HTTP requests are handled in Controller. Generally for every feature you define a talking Controller class that inherits from BaseController (defined in /Shared/BaseController.cs). The ControllerBase class provides the basic integration with routing and HttpContext. In here you set the authorisation protocols (i.e. check if a session is valid for a certain user) and a mediator (that makes the various part of the code communicate to each other in a synced way). Practically, you can set the various responses to request, such as 404NotFound if no results are available (i.e. no authorised sessions) or 200Ok if results can be displayed. If allowed, the request is handles by a class in the form Get[feature]Request, defined in the namesake class and script. 
The results are generally stored in the variable result and return through the method  ``Ok() ``. 
In here you can also set up documentation for swagger using 

.. code-block:: csharp

        /// <summary>
        /// Retrieve biodiversity indicators for one or more sessions.
        /// </summary>
        /// <param name="sessionIds">A list of sessionIds to retrieve biodiversity indicators for</param>
        /// <returns>A collection of indicators, grouped by session</returns>
 

The Get[feature]Request is a class child of IRequest, an interface for Http requests classes. In here we just set up the constructor. For example the sessionids of our request (or user, or subscriptionids), which is then also a property. 

Get[Feature]Handler.cs
----------------------

The Handlers are the scripts that effectively connects to data source (i..e the TableStorage), queries data (i.e. filtering by date), processes the data (i.e. retrieve information from PartitionKey and RowKey) and collect the final result into objects instance of a certain class. For example, the final result of a biodiversity request by sessions is a list of object instances of the class SessionBiodiversityIndicator, each of which comprised a numerical attribute (SessionId) and an object instance of class BiodiversityIndicator, which in turn has four numerical attributes (Period, IndicatorId, InsectCount, Value) plus useful methods (i.e. checking that InsectCount>100). 

Data Sources
------------

The API gets data from the following storages: faunaphotonicsapi, fpbobprod, faunadb, iothub.
As defined in /Shared/TableCoonectionfactory or SqlConnectionfactory.


Biodiversity
------------

#. Biodiversityindicator.cs: a class that describes the indicator of the biodiversity (namely Period (i.e. “2022-52”), IndicatorId (i.e. 2), InsectCount (i.e. 500) , Value (i.e. 50)), there is also a method that certifies when insectCount is valid (>100) 
#. BiodiversityIndicatorEntity.cs: a class that represents a TableEntity with an IndicatorId, InsectCount, Values
#. GetBiodiversityIndicatorHandler.cs: It defines a class that has a StorageConnectionFactory variable (used also in the constructor), a method Handle(request, cancellationToken) that from request get the sessionids and transform them in partitionKeys (by zero-padding on the left) then get the table (via connectionFactory) from the Tables.BioIndicators, and Accounts.Api (?), then entities which is a list obtained the following way: you create a query from BiodiversityIndicatorEntity and where the partitionkey (aka sessionids) are the ones requested. Finally a response, an object of SessionBiodiversityIndicator obtained as: for each partitionkey in partitionKeys you defined indicators as the entities with that partitionkey and select the column Rowkey (“year-week”), IndicatorId, InsectCount and Value and you put them in an object BiodiversityIndicator. Finally a sessionBiodiversityIndicator is defined as a SessionBiodiversityIndicator object initialized with indicators. You add this to the response and return at the end of the method

   NB the tables are in faunaphotonicsapi/Tables/Bioindicators
   NB a BiodiversityIndicatorEntity is a class from Table with entries: IndicatorId (either 1 or 2), InsectCount and Value (of richness)
   NB a BiodiversityIndicator is just a class with only the values of one “row” of the table but also has a Period (year and week, i.e.”2022-18”) and a method isValid (when InsectCount > 100 , otherwise Value is -1). It’s the stuff that is eventually shown in the portal

#. GetBiodiversityIndicatorsRequest.cs: defines the request (inheriting framework from a more general IRequest class) with a constructor initialized by a list of sessionids. The variable SessionIds is then declared ar proprietary with get
#. SessionBiodiversityIndicator.cs: define the class SessionBiodiversityIndicator initialize with long SessionId and IEnumerable<BiodiversityIndicator> Indicators


InsectAggregate
---------------

#. DailyInsectAggregateEntity.cs: as it is often the case when the word Entity is involved, it is a class inheriting from TableEntity with two properties: Count and Date
#. DailyInsectData.cs: a class with 5 properties ClassifierName, ClassifierId, SessionId, Count, Date
#. InsectAggreagteEntity.cs: again, a class from TableEntity with 6 properties: SessionId, ClassifierId, Count. Interval, Fraction, IntervalStart
#. InsectAggregateRepository.cs: see later
#. InsectData.cs: a class with 9 properties (ClassifierId, CLassifierName, SessionId, Count, TimeZone, Fraction, StartTimeUtc, StartTimeLocal and a boolean value IsProbablyInsect (when Fraction > 0.7)

InsectAggregateRepository:
The class has a connection string to the Storage _tableFactory and 7 main methods
#. GetInsectAggregates(sessionId, intervalStart, interlEnd, classifiers, timezone): the variables are partitionKey (generated by zero-padding the sessionId), aggregateIntervals (obtained from GenerateIntervals), rowKeys (obtained from GenerateRowKeys), chunked (RowKeys chunked into list of 50 elements max), an insectAggregate object, namely a (now empty) list (CuncurrentBag) of  instances of InsectAggregateEntity. Then for each chunk in chunked we add to it all the entity in GetInsectAggregatesFromTable(partitionKey). Finally we return for each row a Insectdata object with ClassifierId, Count, fraction, SesionId, StraTimeUtc, TimeZone, ClassifierName (when IsProbablyInsects() == True)
#. GetDailyInsectAggregates(sessionId, intervalStart, intervalEnd, classifiers, timezone): the variables are again partitionKeys, rowkeys (this time generated by GenerateDailyRowkeys, so in the format “date-classifier”, ane the entities from GetDailyINsectAggregatesFromTable with those partitionkeys and rowkeys. For each one of them we return the classifierId and date (splitting rowkey) and DailyInsectData object (with the associated SessionId, ClassfierId, ClasifierNaem, date, Count)
#. GenerateDailyRowKeys(classifiersIds, intervalStart, intervalEnd): generates some rowkeys in the form “{dateId}-{classifier}”
#. GetInsectAggregatesFromTable(partitionKey, rowKey): connect to table faunaphotnicsapi/Tables/InsectAggregates and return the entries with that partitionkey and rowkey (classifier-interval)
#. GetDailyInsectAggregatesFromTable(partitionKey, rowKey): connect to table faunaphotnicsapi/Tables/DailyInsectAggregates and return the entries with that partitionkey and rowkey (classifier-interval). 
#. GenerateRowKeys(classifiers, aggregateIntervals): returns the array of strings “{classifier}-{interval}” for every classifier and interval
#. GenerateIntervals(sessionStart, sessionEnd): cosmos = 19900101, firstInterval = rounded number of half-hours from cosmos to sessionStart, lastInterval = number of half-hours from cosmos to sessionEnd, return Enumerable.Range(start = firstInterval, count = lastInterval -firstInterval). Basically is a collection of half-hours in string format from sessionStart to SessionEnd 


Morphology
----------

#. GetMorphologyGroupsHandler.cs: initializes and handles the various requests and task (i.e. it makes sure functions are synchronized) 
#. GetMorphologyGroupRequests.cs: class with a property ValidSessions and constructor
#. MorphologyActivity.cs: class with 3 properties (Name, SessionName, (a list of) Groups), one method MaxActivity ( returns the max activity of a group if any, otherwise 0)
#. MorphologyController.cs: a controller with authorisationservice and mediator, checks when a session is valid for a certain user
#. MorphologyGroupActivity.cs: class with 3 properties: Name, SessionName, Activity ( a dictionary of string,long)
#. MorphologyService.cs: 


Class with 4 attributes (connectionstring to sql database, to session repo, to insectaggregate repo, and a translationService) and 6 methods:
#. GetMorphologicalActivity: 5 variables: session from sessionrepo, timezone from GetTimezoneforSession, classifier from GetMorphClassifiers, dailydata from aggregaterepo with GetDailyInsectAggregates. If there is dailydata, return GetWeekMorphologicalActivity with that, otherwise data is not from daily but from simply GetInsectAggregates. Pass that to GetWeekMorphologicalActivity instead
#. GetWeekMorphologicalActivity: (overloaded with DailyInsectdata or InsectData): in both cases you have inputData, startdate, endDate, weekYears, groups (now empty list of MorphpologicalGroupActvity objects, classifier and sessionname. For each classifier and for each week in weekYears you add to groups the object activity, initialized by Name and SessionName, and the Activity dictionary (week-count). NB: translate names when required. NB2: If instead of InsectData we have the DailyInsectData object, is the same, just the names of attributes are slightly different 
#. TranslatedNameFromClassifierId: translated names
#. GetMorphClassifiers: connects to sql database and get the names of classifiers required for that session (in particular that subjectcomposition, namely crop)
#. GetTimezoneForSession: connects to sql database and get the timezone of that session 


Glossary
-----------------

.. glossary::

   private, public
      access modifiers for class members

   Abstract class
      it cannot be created to initiate object, only to be inherited

   Abstract method
      Only in abstract classes, it has no body (only in derived classes)

   Interface
      A fully abstract class with only abstract methods. Conventionally its name starts with "I". When a class implements (inherits from) an interface, you must override all of its methods. An interface can contain properties and methods (without specifying "abstract" keyword, since they are like that by default) but no fields.. NB: While a class can only inherit from one parent class, it can implement from multiple interfaces.

   Request
      
   override
      overrides the base class method with the same name within child class

   ?? operator
      null-coalescing operator
      aa??bb??cc?? will give the result of a if it's not null, otherwise try b, otherwise c

   ? 
      nullable type, i.e. bool? can be [True, False, Null]

   properties
      by default all members of a class are private. Private variables can be accessed through the concept of "proprieties", an hybrid between variable and method. A property has two methods: get and set, used for having encapsulatation and making fields read-only (get) or write-only(set)  

   virtual 


   