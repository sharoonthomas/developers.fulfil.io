Inventory Management
====================

This guide explains how to retrieve and update product inventory. Fulfil
is a multi-location, multi-warehouse system. This guide also explains how
to handle inventory by locations.


.. contents:: In this guide
   :local:
   :depth: 1

Inventory Resources
-------------------

It's helpful to understand the different objects and their relationships before
building applications that read or update inventory.

Variant
```````

Each color and size combination of a product is called a **product (variant)**. 
Each variation is an item the customer can buy, while the template itself is a way to 
organize them under one common set of attributes.

::doc:`Learn More <../product-management>`


Location
````````

Represents a geographic location (like a warehouse) or a sub location (like a bin) 
within a geographic location. 

Locations can have many levels depending on the customer implementation

Locations
---------

Locations in fulfil serve multiple purposes. They could be geographic locations
like a warehouse, a virtual location to move inventory in and out of like a
`lost and found` location or a sublocation of a warehouse like a bin.

Default Implementation
````````````````````````

The default implementation usually includes the following strucutre for warehouses.

.. image:: ../_static/images/article-images/inventory/default-warehouse-zones.png

In addition to this setup for each warehouse of the company, Fulfil also creates some
virtual locations by default.

Customer 
    Virtual location to track inventory shipped to customers.

Suppliers
    Virtual location to track inbound inventory from suppliers.

Lost and Found
    Virtual location to which inventory is moved when lost or received from when
    inventory is found.

Initial Inventory
    This location is used when initial inventory is imported into Fulfil when a customer
    migrates into Fulfil.

Transit
    The transit location is a storage zone that houses inventory in transit between 
    warehouses.

Drop
    Similar to the transit location, but used as the interim location for drop shipments. 
    Inventory resides in the location after the supplier has shipped, but before the inventory
    has been received by the customer.

How locations work
```````````````````

All stock movements happen between two locations, very much like a double entry accounting
principle but as a single entry. So for example

Supplier Inbound Shipment
    Inventory moves from `supplier` virtual location to `input zone` of warehouse

Customer Shipment
    Inventory moves from `output` zone of the warehouse to `customer` 

Customer return shipment
    Inventory moves from `customer` virtual location to `input zone` of warehouse

Drop Shipment  
    Inventory moves from `supplier` location to `drop` once the supplier ships and then
    from `drop` to the `customer` location when the customer receives it.

Inventory is lost 
    Inventory moves from the storage location to `lost and found`.

Inventory is found
    Inventory moves from `lost and found` to the storage zone.