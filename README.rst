Developer Documentation
=======================

This repository contains documentation built for Fulfil
developers. The documentation is built using Sphinx and
rst.

Writing Documentation
---------------------

You can either fork the repo and send an edit (Github
offers a great preview) for small changes (like typos)
or clone, build and send a pull-request.

To locally build the documentation, clone the repo and
then run the following commands in the cloned folder.

.. code-block:: shell

   pip install -r requirements.txt

   # Building the documentation
   make html

   # Or watch for changes and continously build
   sphinx-autobuild -B source build/html


The last command constantly watches for changes, rebuilds
and reloads the documentation on the browser.


Deploying
---------

The built documentation is auto-deployed to S3 by circleci.

Creating code examples
----------------------

Install curlify

.. code-block:: sh

    pip install curlify blinker ipython fulfil-client

.. code-block:: python

    import curlify
    from fulfil_client import Client
    from fulfil_client.signals import response_received

    fulfil = Client('trunk', 'api-key')
    Product = fulfil.model('product.product')

    @response_received.connect
    def curlify_response(response):
        print('=' * 80)
        print(curlify.to_curl(response.request))
        print('=' * 80)
        print(response.content)
        print('=' * 80)

    products = Product.search([])
