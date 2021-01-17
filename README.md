# Magento 2 Ruby library

Ruby library to consume the magento 2 api

> Testado na versão 2.3 do magento

- [Getting started](#getting-started)
  - [Install](#install)
  - [Setup](#setup)
- [Model common methods](#model-common-methods)
  - [find](#find)
  - [find_by](#find_by)
  - [first](#first)
  - [count](#count)
  - [all](#count)
  - [create](#create)
  - [update](#update)
  - [delete](#delete)
- [Search criteria](#search-criteria)
  - [Select fields](#select-fields)
  - [Filters](#filters)
  - [Sort order](#sort-order)
  - [Pagination](#pagination)
- [Record Collection](#record-collection)
- [Product](#product)
  - [Shurtcuts](#shurtcut)
  - [Update stock](#update-stock)
  - [Create](#create-a-product)
  - [Update](#update-a-product)
  - [Delete](#delete-a-product)
- [Category](#category)
- [Order](#order)
  - [Invoice](#invoice-an-order)
  - [Offline Refund](#create-offline-refund-for-order)
  - [Creates new Shipment](#creates-new-shipment-for-given-order)
  - [Cancel](#cancel-an-order)
- [Invoice](#invoice)
  - [Refund](#create-refund-for-invoice)
  - [Capture](#capture-invoice)
  - [Void](#void-invoice)
  - [Send email](#send-email-invoice)
  - [Get comments](#get-invoice-comments)
- [Sales Rule](#sales-rule)
  - [Generate Sales Rules and Coupons](#generate-sales-rules-and-coupons)
- [Customer](#customer)
  - [Find by token](#get-customer-by-token)
- [Guest cart](#guest-cart)
  - [Payment information](#payment-information)
  - [Add Coupon](#add-coupon-to-guest-cart)
  - [Remove Coupon](#remove-coupon-from-guest-cart)
- [Inventory](#invoice)
  - [Check whether a product is salable](#check-whether-a-product-is-salable)
  - [Check whether a product is salable for a specified quantity](#check-whether-a-product-is-salable-for-a-specified-quantity)
- [Country](#country)
## Getting started

### Install
Add in your Gemfile

```rb
gem 'magento', '~> 0.26.1'
```

or run

```sh
gem install magento
```

### Setup

```rb
Magento.configure do |config|
  config.url   = 'https://yourstore.com'
  config.token = 'MAGENTO_API_KEY'
  config.store = :default # optional, Default is :all
end

Magento.with_config(store: :other_store) do # accepts store, url and token parameters
  Magento::Product.find('sku')
end
```

## Model common methods

All classes that inherit from `Magento::Model` have the methods described below

### `find`

Get resource details with the `find` method

Example:
```rb
Magento::Product.find('sku-test')
Magento::Order.find(25)
Magento::Country.find('BR')
```

### `find_by`

Returns the first resource found based on the argument passed

Example:
```rb
Magento::Product.find_by(name: 'Some product name')
Magento::Customer.find_by(email: 'customer@email.com')
```

### `first`

Returns the first resource found for the [search criteria](#search-criteria)

Example:
```rb
Magento::Order.where(grand_total_gt: 100).first
```

### `count`

Returns the total amount of the resource, also being able to use it based on the [search criteria](#search-criteria)

Example:
```rb

Magento::Order.count
>> 1500

Magento::Order.where(status: :pending).count
>> 48
```

### `all`

Used to get a list of a specific resource based on the [search criteria](#search-criteria).

Returns a [Record Collection](#record-collection)

Example:
```rb
# Default search criteria:
#  page: 1
#  page_size: 50
Magento::Product.all

Magento::Product
  .order(created_at: :desc)
  .page_size(10)
  .all
```

### `create`

Creates a new resource based on past attributes.

Consult the magento documentation for available attributes for each resource:

Documentation links:
- [Product](#)
- [Category](#)
- [Order](#)
- [Customer](#)

Example:
```rb
Magento::Order.create(
  customer_firstname: '',
  customer_lastname: '',
  customer_email: '',
  # others attrbutes ...,
  items: [
    {
      sku: '',
      price: '',
      qty_ordered: 1,
      # others attrbutes ...,
    }
  ],
  billing_address: {
    # attrbutes...
  },
  payment: {
    # attrbutes...
  },
  extension_attributes: {
    # attrbutes...
  }
)
```

#### `update`

Update a resource based on past attributes.

Consult the magento documentation for available attributes for each resource:

Documentation links:
- [Product](#)
- [Category](#)
- [Order](#)
- [Customer](#)

Example:

```rb
Magento::Product.update('sku-teste', name: 'Updated name')

# or by instance method

product = Magento::Product.find('sku-teste')

product.update(name: 'Updated name')

# or save after changing the object

product.name = 'Updated name'
product.save
```

### `delete`

Delete a especific resource.

```rb
Magento::Product.delete('sku-teste')

# or
product = Magento::Product.find('sku-teste')
product.delete
```

## Search Criteria

They are methods used to assemble the search parameters

All methods return an instance of the `Magento::Query` class. The request is only executed after calling method `all`.

Example:

```rb
customers = Magento::Customer
  .where(dob_gt: '1995-01-01')
  .order(:dob)
  .all

# or

query = Magento::Customer.where(dob_gt: '1995-01-01')

query = query.order(:dob) if ordered_by_date_of_birth

customers = query.all
```

### Select fields:

Example:
```rb
Magento::Product.select(:id, :sku, :name).all

Magento::Product
  .select(
    :id,
    :sku,
    :name,
    extension_attributes: :category_links
  )
  .all

Magento::Product
  .select(
    :id,
    :sku,
    :name,
    extension_attributes: [
      :category_links,
      :website_ids
    ]
  )
  .all
  
Magento::Product
  .select(
    :id,
    :sku,
    :name,
    extension_attributes: [
      { category_links: :category_id },
      :website_ids
    ]
  )
  .all
```

### Filters

Example:
```rb
Magento::Product.where(visibility: 4).all
Magento::Product.where(name_like: 'IPhone%').all
Magento::Product.where(price_gt: 100).all

# price > 10 AND price < 20
Magento::Product.where(price_gt: 10)
                .where(price_lt: 20).all

# price < 1 OR price > 100
Magento::Product.where(price_lt: 1, price_gt: 100).all

Magento::Order.where(status_in: [:canceled, :complete]).all

```

| Condition | Notes |
| --------- | ----- |
|eq         | Equals. |
|finset     | A value within a set of values |
|from       | The beginning of a range. Must be used with to |
|gt         | Greater than |
|gteq       | Greater than or equal |
|in         | In. The value is an array |
|like       | Like. The value can contain the SQL wildcard characters when like is specified. |
|lt         | Less than |
|lteq       | Less than or equal |
|moreq      | More or equal |
|neq        | Not equal |
|nfinset    | A value that is not within a set of values |
|nin        | Not in. The value is an array |
|notnull    | Not null |
|null       | Null |
|to         | The end of a range. Must be used with from |


### Sort Order

Example:
```rb
Magento::Product.order(:sku).all
Magento::Product.order(sku: :desc).all
Magento::Product.order(status: :desc, name: :asc).all
```

### Pagination:

Example:
```rb
# Set page and quantity per page
Magento::Product
  .page(1)       # Current page, Default is 1
  .page_size(25) # Default is 50
  .all

# per is an alias to page_size
Magento::Product.per(25).all
```

## Record Collection

The `all` method retorns a `Magento::RecordCollection` instance

Example:
```rb
products = Magento::Product.all

products.first
>> <Magento::Product @sku="2100", @name="Biscoito Piraque Salgadinho 100G">

products[0]
>> <Magento::Product @sku="2100", @name="Biscoito Piraque Salgadinho 100G">

products.last
>> <Magento::Product @sku="964", @name="Biscoito Negresco 140 G Original">

products.map(&:sku)
>> ["2100", "792", "836", "913", "964"]

products.size
>> 5

products.current_page
>> 1

products.next_page
>> 2

products.last_page?
>> false

products.page_size
>> 5

products.total_count
>> 307

products.filter_groups
>> [<Magento::FilterGroup @filters=[<Magento::Filter @field="name", @value="biscoito%", @condition_type="like">]>]
```

All Methods:

```rb
# Information about search criteria
:current_page
:next_page
:last_page?
:page_size
:total_count
:filter_groups

# Iterating with the list of items
:count
:length
:size

:first
:last
:[]
:find

:each
:each_with_index
:sample

:map
:select
:filter
:reject
:collect
:take
:take_while

:sort
:sort_by
:reverse_each
:reverse

:all?
:any?
:none?
:one?
:empty?
```

**Outside pattern**

Get customer by token

```rb
Magento::Customer.find_by_token('user_token')
```

## Shurtcut to get custom attribute value by custom attribute code in product

Exemple:

```rb
product.attr :description 
# it is the same as
product.custom_attributes.find { |a| a.attribute_code == 'description' }&.value

# or
product.description
```

when the custom attribute does not exists:

```rb
product.attr :special_price
> nil

product.special_price
> NoMethodError: undefined method `special_price' for #<Magento::Product:...>
```

```rb
product.respond_to? :special_price
> false

product.respond_to? :description
> true
```

## add tier price

Add `price` on product `sku` for specified `customer_group_id`

Param `quantity` is the minimun amount to apply the price

```rb
product = Magento::Product.find(1)
product.add_tier_price(3.99, quantity: 1, customer_group_id: :all)
> true

# OR

Magento::Product.add_tier_price(1, 3.99, quantity: 1, customer_group_id: :all)
> true
```

\* _same pattern to all models_

## GuestCart

Set payment information to finish the order
```rb
cart = Magento::GuestCart.find('gXsepZcgJbY8RCJXgGioKOO9iBCR20r7')

# or use "build" to not request information from the magento API
cart = Magento::GuestCart.build(
  cart_id: 'aj8oUtY1Qi44Fror6UWVN7ftX1idbBKN'
)

cart.payment_information(
  email: 'customer@gmail.com',
  payment: { method: 'cashondelivery' }
)

>> "234575" # return the order id
```

Add coupon to cart
```rb
cart = Magento::GuestCart.find('gXsepZcgJbY8RCJXgGioKOO9iBCR20r7')

cart.add_coupon('COAU4HXE0I')
# You can also use the class method
Magento::GuestCart.add_coupon('gXsepZcgJbY8RCJXgGioKOO9iBCR20r7', 'COAU4HXE0I')

>> true # return true on success
```

Delete coupon from cart
```rb
cart = Magento::GuestCart.find('gXsepZcgJbY8RCJXgGioKOO9iBCR20r7')

cart.delete_coupon()
# You can also use the class method
Magento::GuestCart.delete_coupon('gXsepZcgJbY8RCJXgGioKOO9iBCR20r7')

>> true # return true on success
```

## Invoice an Order

```rb
Magento::Order.invoice(order_id)
>> 25 # return incoice id

# or from instance

order = Magento::Order.find(order_id)

invoice_id = order.invoice

# you can pass parameters too

invoice_id = order.invoice(
  capture: false,
  appendComment: true,
  items: [{ order_item_id: 123, qty: 1 }], # pass items to partial invoice
  comment: {
    extension_attributes: { },
    comment: "string",
    is_visible_on_front: 0
  },
  notify: true
)
```

[Complete Invoice Documentation](https://magento.redoc.ly/2.4-admin/tag/orderorderIdinvoice#operation/salesInvoiceOrderV1ExecutePost)

## Create refund for invoice

```rb
Magento::Invoice.invoice(invoice_id)
>> 12 # return refund id

# or from instance

invoice = Magento::Invoice.find(invoice_id)

refund_id = invoice.refund

# you can pass parameters too

invoice.refund(
  items: [
    {
      extension_attributes: {},
      order_item_id: 0,
      qty: 0
    }
  ],
  isOnline: true,
  notify: true,
  appendComment: true,
  comment: {
    extension_attributes: {},
    comment: string,
    is_visible_on_front: 0
  },
  arguments: {
    shipping_amount: 0,
    adjustment_positive: 0,
    adjustment_negative: 0,
    extension_attributes: {
      return_to_stock_items: [0]
    }
  }
)
```

[Complete Refund Documentation](https://magento.redoc.ly/2.4-admin/tag/invoicescomments#operation/salesRefundInvoiceV1ExecutePost)


## Create offline refund for order

```rb
Magento::Order.refund(order_id)
>> 12 # return refund id

# or from instance

order = Magento::Order.find(order_id)

order.refund

# you can pass parameters too

order.refund(
  items: [
    {
      extension_attributes: {},
      order_item_id: 0,
      qty: 0
    }
  ],
  notify: true,
  appendComment: true,
  comment: {
    extension_attributes: {},
    comment: string,
    is_visible_on_front: 0
  },
  arguments: {
    shipping_amount: 0,
    adjustment_positive: 0,
    adjustment_negative: 0,
    extension_attributes: {
      return_to_stock_items: [0]
    }
  }
)
```

[Complete Refund Documentation](https://magento.redoc.ly/2.4-admin/tag/invoicescomments#operation/salesRefundOrderV1ExecutePost)

## Other Invoice methods

```rb
invoice = Magento::Invoice.find(invoice_id)

invoice.capture # or
Magento::Invoice.capture(invoice_id)

invoice.void # or
Magento::Invoice.void(invoice_id)

invoice.send_email # or
Magento::Invoice.send_email(invoice_id)

Magento::Invoice.comments(invoice_id).all
Magento::Invoice.comments(invoice_id).where(created_at_gt: Date.today.prev_day).all
```

## Creates new Shipment for given Order.

```rb
Magento::Order.ship(order_id)
>> 25 # return shipment id

# or from instance

order = Magento::Order.find(order_id)

order.ship

# you can pass parameters too

order.ship(
  capture: false,
  appendComment: true,
  items: [{ order_item_id: 123, qty: 1 }], # pass items to partial shipment
  tracks: [
    {
      extension_attributes: { },
      track_number: "string",
      title: "string",
      carrier_code: "string"
    }
  ]
  notify: true
)
```

[Complete Shipment Documentation](https://magento.redoc.ly/2.4-admin/tag/orderorderIdship#operation/salesShipOrderV1ExecutePost)


## Cancel an Order

```rb
order = Magento::Order.find(order_id)

order.cancel # or

Magento::Order.cancel(order_id)
```

## Generate Sales Rules and Coupons

```rb
rule = Magento::SalesRule.create(
  name: 'Discount name',
  website_ids: [1],
  customer_group_ids: [0,1,2,3],
  uses_per_customer: 1,
  is_active: true,
  stop_rules_processing: true,
  is_advanced: false,
  sort_order: 0,
  discount_amount: 100,
  discount_step: 1,
  apply_to_shipping: true,
  times_used: 0,
  is_rss: true,
  coupon_type: 'specific',
  use_auto_generation: true,
  uses_per_coupon: 1
)

rule.generate_coupon(quantity: 1, length: 10)
```

Generate by class method
```rb
Magento::SalesRule.generate_coupon(
  couponSpec: {
    rule_id: 7,
    quantity: 1,
    length: 10
  }
)
```
see all params in:
- [Magento docs Coupon](https://magento.redoc.ly/2.3.5-admin/tag/couponsgenerate#operation/salesRuleCouponManagementV1GeneratePost)
- [Magento docs SalesRules](https://magento.redoc.ly/2.3.5-admin/tag/salesRules#operation/salesRuleRuleRepositoryV1SavePost)
	
See [how to add coupons to cart](#guestcart)

### First result
```rb
Magento::Product.first
>> <Magento::Product @sku="some-sku" ...>

Magento::Product.where(name_like: 'some name%').first
>> <Magento::Product @sku="some-sku" ...>
```

### Count result
```rb
Magento::Product.count
>> 7855
Magento::Product.where(name_like: 'some name%').count
>> 15
```

## Inventory

### Check whether a product is salable

```rb
Inventory.get_product_salable_quantity(sku: '4321', stock_id: 1)
>> 1
```

### Check whether a product is salable for a specified quantity

```rb
Inventory.is_product_salable_for_requested_qty(
  sku: '4321',
  stock_id: 1,
  requested_qty: 2
)
>> OpenStruct {
  :salable => false,
  :errors => [
    [0] {
      "code" => "back_order-disabled",
      "message" => "Backorders are disabled"
    },
    ...
  ]
}
```

## Update product stock

```rb
product = Magento::Product.find('sku')
product.update_stock(qty: 12, is_in_stock: true)
```

or by class method

```rb
Magento::Product.update_stock(sku, id, {
  qty: 12,
  is_in_stock: true 
})
```

see all available attributes in: https://magento.redoc.ly/2.4.1-admin/tag/productsproductSkustockItemsitemId

___

##TODO:

### Search products
```rb
Magento::Product.search('tshort')
```

### Last result
```rb
Magento::Product.last
>> <Magento::Product @sku="some-sku" ...>

Magento::Product.where(name_like: 'some name%').last
>> <Magento::Product @sku="some-sku" ...>
```

### Tests


## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/WallasFaria/nfce_crawler.
