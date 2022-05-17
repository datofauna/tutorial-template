Usage
=====

.. _installation:

Installation
------------

To test and debug the API, first intall VSCode and Postman using pip:

.. code-block:: console

   (.venv) $ pip install ...

Also .NET Core (for Windows: https://dotnet.microsoft.com/en-us/download/dotnet/6.0)


Using PostMan
----------------

#. Import API Collection
#. Setting environments (NB: the developmnet env in vscode is the local environment in postman)
#. Authentication
#. Send a request
#. Create a new request


Debug mode in VSCode
--------------------

#. Install necessary extensions
#. Import appsetting.Development.json

#. Run in terminal:

.. code-block:: console

   (.venv) $ dotnet run

Swagger
-------

https://api-fauna.azurewebsites.net/swagger/index.html