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