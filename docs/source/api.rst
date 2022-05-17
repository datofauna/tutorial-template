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
The Main() method is the entry point of the application. The host is created calling the ``Build().Run()`` extension method in a CreateDefaultBuilder method. The ConfigureWebHostDefaults() method can be used to configure a web application host .In our case it refers to Startup


Startup.cs
----------
The Startup class contains the ConfigureService (which is used to configure the required services) and Configure (used to configure the request processing pipeline) methods. When the application starts, the first method is executed and the second too, immediately after. In our API i.e., in ConfigureService we add some services defined in the built-in interface IServiceCollection for the monitoring and memory caching, plus the swagger documentation, the tools to access data (i.e. databases and repos) and our user-defined services (related to insect activity i.e.).
The Configure method is quite default. The UseRouting and UseEndPoints methods are used to add and configure middleware to the request processing pipeline

Http request
------------

The HTTP requests are handled in Controller. Generally for every feature you define a talking Controller class that inherits from BaseController (defined in /Shared/BaseController.cs). The ControllerBase class provides the basic integration with routing and HttpContext). In here you set the authorisation protocols (i.e. check if a session is valid for a certain user) and a mediator (that makes the various part of the code communicate to each other in a synced way). Practically, you can set the various responses to request, such as 404NotFound if no results are available (i.e. no authorised sessions) or 200Ok if results can be displayed. If allowed, the request is handles by a class in the form Get[feature]Request, defined in the namesake class and script. 
The results are generally stored in the variable result and return through the method Ok(). 
In here you can also set up documentation for swagger using ///

The Get[feature]Request is a class child of IRequest, an interface for Http requests classes. In here we just set up the constructor. For example the sessionids of our request (or user, or subscriptionids), which is then also a property. 

The Get[feature]Request is a class child of IRequest, an interface for Http requests classes. In here we just set up the constructor. For example the sessionids of our request (or user, or subscriptionids), which is then also a property. 

Get[Feature]Handler.cs
----------------------

The Handlers are the scripts that effectively connects to data source (i..e the TableStorage), queries data (i.e. filtering by date), processes the data (i.e. retrieve information from PartitionKey and RowKey) and collect the final result into objects instance of a certain class. For example, the final result of a biodiversity request by sessions is a list of object instances of the class SessionBiodiversityIndicator, each of which comprised a numerical attribute (SessionId) and an object instance of class BiodiversityIndicator, which in turn has four numerical attributes (Period, IndicatorId, InsectCount, Value) plus useful methods (i.e. checking that InsectCount>100). 

Data Sources
------------

The API gets data from the following storages: faunaphotonicsapi, fpbobprod, faunadb, iothub.
As defined in /Shared/TableCoonectionfactory or SqlConnectionfactory.


Recurrent objects
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
      overrides the base class method with the same name

   ?? operator
      null-coalescing operator
      aa??bb??cc?? will give the result of a if it's not null, otherwise try b, otherwise c

   ? 
      nullable type, i.e. bool? can be [True, False, Null]

   properties
      by default all members of a class are private. Private variables can be accessed through the concept of "proprieties", an hybrid between variable and method. A property has two methods: get and set, used for having encapsulatation and making fields read-only (get) or write-only(set)  

   virtual 


   