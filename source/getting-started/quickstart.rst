Development Quickstart
======================

Get up and running with our API libraries and start developing your Fulfil integration.

Integrating Fulfil into your app can begin as soon as you create a Fulfil account,
reqiring only three steps.

.. contents::

Step 1: Obtain your API keys
----------------------------

Fulfil authenticates your API requests using your personal API keys. If you do
not include your key when making an API request, or use one that is incorrect or outdated, 
Fulfil returns an error.


.. warning:: 

   Treat the API key and password like you would any other password,
   since whoever has access to these credentials has full API access
   to the platform.


Step 2: Install an API library
------------------------------

We provide an official library for Python. Unofficial libraries are available for
other languages and platforms.

.. example-code::

   .. code-block:: python

      pip install fulfil-client


Step 3: Make a test API request
-------------------------------

To check that your integration is working correctly, make a test API request 
using your API key to get your preferences.

.. example-code::

   .. code-block:: python

      from fulfil_client import Client
      fulfil = Client('demo', 'YOUR_API_KEY')
      Product = fulfil.model('product.product')

      # Search for products
      product_ids = Product.search([], None, 1)

      # Read the products
      products = Product.read(product_ids, ['code'])

Fulfil returns an array of objects with the code and id of the products

.. code-block:: json

   [{"code": "SHIP", "id": 9503}]
