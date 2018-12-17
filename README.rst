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

    pip install blinker ipython fulfil-client

This removes unnecessary headers that will allow you to write clean and neat 

.. code-block:: python

    import urllib
    from copy import copy
    from fulfil_client import Client
    from fulfil_client.signals import response_received

    HEADERS_TO_REMOVE = [
        'Connection',
        'Accept-Encoding',
        'Accept',
        'User-Agent',
        'Content-Length',
    ]

    def to_curl(request, compressed=False):
        """
        Returns string with curl command by provided request object
        Parameters
        ----------
        compressed : bool
            If `True` then `--compressed` argument will be added to result
        """
        parts = [
            ('curl', urllib.unquote_plus(request.url).decode('utf8')),
            ('-X', request.method),
        ]

        for k, v in sorted(request.headers.items()):
            if k in HEADERS_TO_REMOVE:
                continue
            parts += [('-H', '{0}: {1}'.format(k.upper(), v))]
        parts += [('-H', 'Content-Type: application/json')]

        if request.body:
            body = request.body
            if isinstance(body, bytes):
                body = body.decode('utf-8')
            parts += [('-d', body)]

        if compressed:
            parts += [('--compressed', None)]

        flat_parts = []
        for k, v in parts:
            arguments = []
            if k:
                arguments.append(k)
            if v:
                arguments.append("'{0}'".format(v))
            flat_parts.append(' '.join(arguments))

        return ' \\\n '.join(flat_parts)


    @response_received.connect
    def curlify_response(response):
        request = copy(response.request)
        for header in HEADERS_TO_REMOVE:
            request.headers.pop(header, None)
        request.headers['x-api-key'] = "{YOUR_API_KEY}"
        request.url = request.url.replace('trunk', '{merchant_id}')
        request.url = request.url.replace('/v1/', '/{version}/')
        print('=' * 80)
        print(to_curl(request))
        print('=' * 80)
        print(response.content)
        print('=' * 80)

    # And then run code examples
    fulfil = Client('trunk', 'api-key')
    Product = fulfil.model('product.product')
    products = Product.search([])
