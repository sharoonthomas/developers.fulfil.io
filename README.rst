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