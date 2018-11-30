**Jquery**

.. code-block:: javascript

    var settings = {
        "url": "https://{merchant}.fulfil.io/api/v1/model/product.product/search",
        "method": "PUT",
        "headers": {
            "Accept": "*/*",
            "x-api-key": "YOUR_API_KEY",
            "Content-Type": "application/json"
        },
        "data": "[[]]",
        "async": true,
        "crossDomain": true,
        "processData": false,
    }

    $.ajax(settings).done(function (response) {
        console.log(response);
    });