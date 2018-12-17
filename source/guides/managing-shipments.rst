Managing Shipments
==================

This guide explains how to use the shipments API to manage outbound shipments.
You may use this to complement the Fulfil shipping app functionality or
to build a shipping interface that works on top of the Fulfil API.

.. contents:: In this guide

.. tip::
    
    **Creating Shipments**

    Though possible over the API, we discourage creating shipments directly
    on the Fulfil API. We recommend creating orders instead and shipments to be
    created from orders.

    This helps our merchants take benefit from features like merging shipments,
    cycle time statistics, cost reports and a variety of features that work only
    on shipments created from orders.

States of a shipment
--------------------

Before you begin working with products, it’s helpful to understand the 
different states of a shipment:

    * **Waiting**: Shipments waiting for inventory allocation. Some items may have
      inventory reserved, but the shipment stays in the *waiting* stage until all
      items have inventory allocated.
    * **Assigned**: All item have been allocated inventory.
    * **Packed**: Items have been picked and packed.
    * **Done**: Items have been shipped.

These are the primary states of a shipment and works whether the shipment is fulfilled
from your warehouse (with a pick, pack, ship process) or through a 3PL.

Pick, Pack, Ship states
-----------------------

When the shipment is fulfilled at a merchant controlled warehouse, some of the above
states in combination with other information helps track the current status of the
shipment in the pick, pack and ship process.

  * **Ready to pick**: Shipment `state` is `assigned` and `picking_status` is `null`.
  * **Picking in progress**: Shipment `state` is `assigned` and `picking_status` is `in-progress`.
  * **Ready to pack**: Shipment `state` is `assigned` and `picking_status` is `done`.
  * **Ready to label**: Shipment `state` is `packed` and `tracking_number` is `null`.
  * **Ready to ship**: Shipment `state` is `packed` and `tracking_number` is not `null`.

Using the API to building fulfillment apps
-------------------------------------------

The following API calls should give you a picture on how to build your own shipping
interfaces on top of Fulfil. These are the same API endpoints and methods used by
the Fulfil shipping app for fulfillment.

Based on the scope of your app, you can select the API calls you will need and it is
unlikely you will need all of them.

You will notice that there are many smaller steps in the process of shipping from
being ready to pick, picking in-progress, picking done (ready to pack), packed and so on.
These stages exist so Fulfil can adapt to the unique needs of merchants as they scale
from hundreds of shipments per day to thousands of shipments per day. If you're building
a custom app for a specific merchant, we recommend only building for the unique
workflow of this merchant.

Filtering by warehouse
``````````````````````

When building apps for the warehouse, if there are more than 1 warehouses, it is likely
that you want to filter the results for a specific warehouse.

To filter for a warehouse, you will need the `id` of the warehouse or the `code` of
the warehouse.

If you have an id, add this to the filter:

.. code-block:: JSON

   ["warehouse", "=", 10]

or if you have the code of the warehouse

.. code-block:: JSON

   ["warehouse.code", "=", "code"]


Filtering Pick Up 
`````````````````

If you have brick and mortar stores or you provide customers with the ability to
pick up items, then you may not want to see thos shipments for your app. 

.. code-block:: JSON

   ["delivery_mode", "!=", "pick_up"]

Perhaps you are building an app for the stores and you *only* want to see the
pickup shipments

.. code-block:: JSON

   ["delivery_mode", "=", "pick_up"]



Waiting for allocation
``````````````````````

These are shipments waiting for inventory allocation. Some items may have
inventory reserved, but the shipment stays in the *waiting* stage until all
items have inventory allocated.

.. code-block:: python

   Shipment = fulfil.model('stock.shipment.out')
   Shipment.search([
      ('state', '=', 'waiting'),
   ])


Ready to pick
``````````````

* Shipments have been allocated inventory.
* Pickers have not started picking them yet.

.. code-block:: JSON 

   [
      ["state", "=", "assigned"],
      ["picking_status", "=", null],
   ]

If you'd like to display this like the Fulfil shipping app does
with different priorities, then you can either read shipment
data and then do client side segmenting or add another criteria
to the search filters.

.. code-block:: JSON

   [
      ["priority", "=", 0],
      ["state", "=", "assigned"],
      ["picking_status", "=", null],
   ]

The priority levels are:

* `0` - Highest
* `1` - High
* `2` - Normal
* `3` - Low
* `4` - Lowest

Start picking
``````````````

When a picker starts picking, it is important to remove the shipment
from the ready to pick queue and add it to the picking in progress
queue. This avoids two pickers picking the same shipment.

.. code-block:: shell

   curl 'https://{merchant}.fulfil.io/api/{version}/model/stock.shipment.out/start_picking' 
      -H 'X-API-KEY: {your-api-key}' 
      -X PUT
      -d '[SHIPMENT_ID1,SHIPMENT_ID2]'

.. code-block:: python

   Shipment.start_picking([id1, id2])

The picker is automatically set as the current user.

Picking in progress
```````````````````

You can fetch a list of shipment currently being picked by fitlering
shipments with the picking_status `in-progress`

.. code-block:: JSON

   ["picking_status", "=", "in-progress"]


Finish Picking
``````````````

When a picker is done picking, you can mark the shipment as picked. This
is only used by Fulfil merchants with over 15,000 shipments per day and
a workflow that involves completed picked items being staged for a separate
person to pack.

.. code-block:: shell

   curl 'https://{merchant}.fulfil.io/api/{version}/model/stock.shipment.out/done_picking' 
      -X PUT
      -H 'X-API-KEY: {your-api-key}'
      -d '[SHIPMENT_ID1,SHIPMENT_ID2]'

.. code-block:: python

   Shipment.done_picking([id1, id2])

.. note::

   If the user marking the shipment as picked is different from the
   user who started picking, the data is overwritten. This is to support
   workflows where shipments to pick are allocated on paper to multiple
   pickers by a warehouse manager and picked by warehouse employees who
   then mark shipments as picked with a barcode scan.

Ready to pack
``````````````

The search filter for fetching ready to pack shipments:

.. code-block:: JSON 

   [
      ["state", "=", "assigned"],
      ["picking_status", "=", "done"],
   ]


Mark as packed
``````````````

When a shipment is ready, you can mark the shipment as packed. Usually
this corresponds to a packer putting items into shipping boxes, wrapping
items and finishing any necessary prep to ship the item.

This is a point of no-return. Once a shipment has been marked as packed,
it is not possible to unpack it. This point is required so customer service
teams can stop accepting changes to a shipment.

.. code-block:: shell

   curl 'https://{merchant}.fulfil.io/api/{version}/model/stock.shipment.out/pack' 
      -H 'X-API-KEY: {your-api-key}' 
      -X PUT
      -d '[SHIPMENT_ID1,SHIPMENT_ID2]'

.. code-block:: python

   Shipment.pack([id1, id2])

This method could raise exceptions when being marked as packed if the order
has exceptions. This could be for a variety of reasons from failed payments to
address changes a customer makes on a storefront like Shopify.

The exception will be returned with a 4XX response and a JSON encoded error
message that should be displayed to the user for further action.

Packed (ready to label)
```````````````````````

These are packed shipments that don't have a tracking number yet.

.. code-block:: JSON 

   [
      ["state", "=", "packed"],
      ["tracking_number", "=", null],
   ]

At this point, you are also likely to know if this is a multi-package
shipment. To create more packages, fulfil offers a convenient method
to auto-create packages for you.

.. code-block:: shell

   curl 'https://{merchant}.fulfil.io/api/{version}/model/stock.shipment.out/shipping_app_add_remove_packages' 
      -H 'X-API-KEY: {your-api-key}' 
      -X PUT
      -d '[SHIPMENT_ID, PACKAGES]'

.. code-block:: python

   Shipment.shipping_app_add_remove_packages(shipment_id, number_of_packages)

This step creates new packages and splits the weight across the packages. If your app
wants to also track which items go into each package (usually not a required detail),
you can do so by associating the package ids with the items in the shipment.

The usual next step is to generate a shipping label for the shipment.

Get shipping rates
``````````````````

In most cases, especially at scale you would not want to have your shippers
do any kind of rate shopping. However, it might still be needed where the
rules have not automatically set a carrier (a new international destination) or
a super heavy package that cannot be sent by regular methods.

Fulfil's rate shopping API is also available on the API. To fetch available
rates for a shipment

.. code-block:: python

   Shipment.shipping_app_compare_rates(shipment_id)

The response is an object with multiple rates

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 2,18,20,28

   [{
        "service_name": "Xpresspost USA",
        "cost": {
            "decimal": "34.41",
            "__class__": "Decimal"
        },
        "currency": "CAD",
        "delivery_days": 4,
        "rate": {
            "decimal": "34.41",
            "__class__": "Decimal"
        },
        "list_rate_currency": "CAD",
        "list_rate": {
            "decimal": "34.41",
            "__class__": "Decimal"
        },
        "display_name": "Xpresspost USA - 34.41 CAD",
        "service_code": "USA.XP",
        "carrier_service": 178,
        "terms": null,
        "account_id": "0000000",
        "trackable": null,
        "insurance": null,
        "guaranteed": false,
        "delivery_time": null,
        "messages": null,
        "carrier": 15,
        "cost_currency": 134,
        "booking_url": null,
        "delivery_date": {
            "iso_string": "2018-12-11",
            "month": 12,
            "__class__": "date",
            "day": 11,
            "year": 2018
        }
    }, ...]


The list of rates can be displayed in a format you prefer. This
works across carriers including Amazon Seller Fulfilled.

You can then select a rate by setting `carrier` (line 28) and the
`carrier_service` (line 20).

.. code-block:: python

   Shipment.write(
      [shipment_id],
      {'carrier': carrier_id, 'carrier_service': service_id}
   )

Generating a label
``````````````````

If the carrier and service are set on the shipment, you can
generate a label.

.. code-block:: python

   Shipment.generate_shipping_labels(shipment_id)

Fetching and printing labels
````````````````````````````

To print the shipping label, you can call the Fulfil
reporting engine to print a "label" for you.

.. code-block:: python

   Shipment.get_shipping_labels([shipment_id1, shipment_id2])

This always returns a pdf and the url to download the PDF from.

The shipping labels are usually stored as a JPEG attachment
on the shipment. The format is different between different
shipping carriers, so this may not be the best approach to
handling labels if you have multiple carriers.

You can also send the file directly to a printer in
a known workstation.

.. code-block:: python

   Workstation = fulfil.model('fulfil.devicehub.packing_station')
   Workstation.print_(
        url, 'label',
        context={'workstation': workstation_id}
    )

The second argument to the `print_` function determines whether
the document will be printed on an attached `label` printer or a
`document` printer.

Printing documents
------------------

TODO


