Usage
=====

.. _installation:

Installation
------------

To test and debug the API, it is advised to intall VSCode and Postman using pip:

.. code-block:: console

   (.venv) $ pip install 

Also .NET compiler


Using PostMan
----------------

Import API Collection
Setting environments
Authentication
Send a request
Create a new request

To retrieve a list of random ingredients,
you can use the ``lumache.get_random_ingredients()`` function:

.. autofunction:: lumache.get_random_ingredients

The ``kind`` parameter should be either ``"meat"``, ``"fish"``,
or ``"veggies"``. Otherwise, :py:func:`lumache.get_random_ingredients`
will raise an exception.

.. autoexception:: lumache.InvalidKindError

For example:

>>> import lumache
>>> lumache.get_random_ingredients()
['shells', 'gorgonzola', 'parsley']


Debug mode in VSCode
--------------------

Install extension
IMport appsetting.Development.json

Run in terminal:

.. code-block:: console

   (.venv) $ dotnet run

Swagger
-------