# Independent entities for order state

The order state is currently saved in `sales_order` database but a state should be have and event history also it must be not cupeled to an order. 

#### Common Problems

This section describes common problems which affect the current flow of order entity manipulations.

1. No possibility to update only order state or status without updating the whole entity.
2. An order's life circle depends on its state which can be updated in an inconsistent way.
3. As the state|status can be updated with the whole order and all related data it has more performance impact.
4. Payment integrations use asynchronous operations, they can't update order state|status.
5. Overloaded tables: *sales_order* - 206 columns (EE), *sales_invoice* - 66 (EE), *sales_creditmemo* - 73 (EE).
6. No possibility to create own states for order.



### Database Tables for Order State

There will no forrend key between `sales_order_state` and `sales_order_state_link`

### DB Design  for `sales_order_state`

| id   | state   | created_at          |
| ---- | ------- | ------------------- |
| 1    | new     | 2019-06-01 19:55:56 |
| 2    | new     | 2019-06-01 19:57:56 |
| 3    | pending | 2019-06-01 20:00:00 |



### DB Design `sales_order_state_link`

| id   | link_id | order_id |
| ---- | ------- | -------- |
| 1    | 1       | 1        |
| 2    | 2       | 2        |
| 3    | 3       | 1        |



### DB Design  for `sales_order_state_event_log`

| id   | link_id | Log                                      |
| ---- | ------- | ---------------------------------------- |
| 1    | 3       | State Changed by API from new to pending |

The event log tables to understand wehere changed the state and why this will help for complex event sourced payments flows it should be an`json` with information this information should be showed at the order. This table helps also to reduce the order comments for payment and service provider currently it very often used by them. 



### API Interfaces Database Tables for Order State

The GetOrderStateById will return only the last **order state**

 

#### GetOrderStateById

```php
<?php
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */
declare(strict_types=1);

namespace Magento\Sales\Model;

interface GetOrderStateByIdInterface
{
    /**
     *
     * @param int $orderId
     * @return OrderStateInterface
     */
    public function execute(int $orderId);
}
```

### OrderStateInterface

```php
<?php
/**
 * Copyright © Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */
declare(strict_types=1);

namespace Magento\Sales\Model;

class OrderStateInterface extends \Magento\Framework\Api\ExtensibleDataInterface
{ 
    public function getState() :string;

    public function getCreatedAt(): string;
  
    public function getEventInfomation(): string;
}
```
