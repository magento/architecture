# Overview
The cron is one of the most business critical components of Magento, offloading resource 
intensive tasks from the user request into the backend and allowing scheduling long running
processes.

The business critical element of the cron needs better supporting in the foundation of the module and
supporting documentation giving better control to developers, technology providers and eCommerce managers.

In addition to the cron, certain processes should be moved away from the cron architecture to a service
based architecture leveraging AMQP.

# Problem
The cron implementation in it's current existence doesn't provide the business critical foundation 
or visibility required for a fully functioning eCommerce system.

# Solution - Business Critical Foundation
The overall approach of how magento handled backend/scheduled tasks needs to be reviewed with a 
*business critical* foundation in mind. This involves a re-architecture of the cron itself
and a move toward a service based AMQP  

1. Separation
2. Visibility
3. Education

## 1. Separation
Separation of business critical and none business critical backend processes as part of the core
architecture of Magento. This should be multi threaded "out the box" running business critical processes
in their own threads (Emails, Indexes).

At the foundation no rogue process should ever prevent a business critical operation.

- Multi thread core processes out the box (e.g. workers)
- Process timeouts (Default X minutes and configurable by job) 
- Retry limits
- Exponential back off

The architectural goal for separation is service isolation using the Cron to schedule and message queues
for all processes with scalable workers for execution.

### Cron
The current cronjobs that run within the core of Magento are:
- sales_send_order_emails
- sales_send_order_invoice_emails
- sales_send_order_shipment_emails
- sales_send_order_creditmemo_emails
- sales_clean_quotes
- sales_clean_orders
- aggregate_sales_report_order_data
- aggregate_sales_report_invoiced_data
- aggregate_sales_report_refunded_data
- aggregate_sales_report_bestsellers_data
- sales_grid_order_async_insert
- sales_grid_order_invoice_async_insert
- sales_grid_order_shipment_async_insert
- sales_grid_order_creditmemo_async_insert
- catalog_index_refresh_price
- catalog_product_flat_indexer_store_cleanup
- catalog_product_outdated_price_values_cleanup
- catalog_product_frontend_actions_flush
- catalog_product_attribute_value_synchronize
- indexer_reindex_all_invalid
- indexer_update_all_views
- indexer_clean_all_changelogs
- persistent_clear_expired
- security_clean_admin_expired_sessions
- security_clean_password_reset_request_event_records
- magento_newrelicreporting_cron
- mysqlmq_clean_messages
- outdated_authentication_failures_cleanup
- expired_tokens_cleanup
- catalog_product_alert
- sitemap_generate
- backend_clean_cache
- aggregate_sales_report_shipment_data
- currency_rates_update
- captcha_delete_old_attempts
- captcha_delete_expired_images
- paypal_fetch_settlement_reports
- messagequeue_clean_outdated_locks
- consumers_runner
- consumerConsumerName
- bulk_cleanup
- catalogrule_apply_all
- newsletter_send_all
- system_backup
- visitor_clean
- analytics_subscribe
- analytics_update
- analytics_collect_data
- aggregate_sales_report_coupons_data

This excludes 3rd party bundled extensions such as DotDigital.

## 2. Visibility
On Magento 1 we had the AOE Scheduler which provided us a considerably enhanced user experience. This
module isn't available for Magento 2. Allowing webmasters and developers to clearly see what processes
are running, scheduled and complete gives them greater control and reduces support requests and increases
confidence in the platform.

- Admin visible timeline of scheduled tasks (Scheduled, in progress, complete, errored)
- Ability to schedule, delete, remove, pause specific tasks.
- Cleanup of redundant processes to reduce noise
- Logging and Alerting

## 3. Education
- Technical documentation for effectively implementing schedules into Magento modules
- Technical documentation for when to use cron vs AMQP
- Setup documentation for multi threaded implementation  

# Examples

## Example 1 - Rogue process
A process that connects to an API such as a payment provider could end up getting stuck in a loop
and blocking any other process from progressing currently there is no mechanic in place to
a: Allow other business critical processes to continue aside from defining multiple crontab processes in groups
b: Report or stop the rogue process from continuing until manually spotted
c: No visibility of the processes aside from checking the database
d: No reporting or alerting of the above condition

# Staged Approach
The overhaul of the background processing within Magento is an large project with high impact to to the
current public API's and implementation of the current Magento Cron.

## Stage 1 - Framework
Create/expand the framework for AMQP and Worker management.

1. Review queue interfaces
2. Standardise data structure
3. Error handling, logging and alerting
4. Review worker interfaces
5. Review worker management

###Problems to solve:
1. Worker management
How to manage workers at from simple to expert levels of complexity. Currently the cron spawns workers
with (from the looks of it) no visibility over the threads that are spawned. We also need to factor
in completely multi-node infrastructure.
The aim should be to provide a highly scalable implementation from a highly scaled environment to a
single server infrastructure with simple platform implementation.

## Stage 1 - Dashboard
1. Visibility of each queue and number of items in each queue
2. Visibility of any errors

## Stage 2 - Transactional Emails
Migrate the following cron tasks into the new queue based implementation
- sales_send_order_emails
- sales_send_order_invoice_emails
- sales_send_order_shipment_emails
- sales_send_order_creditmemo_emails

## Stage 3 - Indexers
- sales_grid_order_async_insert
- sales_grid_order_invoice_async_insert
- sales_grid_order_shipment_async_insert
- sales_grid_order_creditmemo_async_insert
- catalog_index_refresh_price
- catalog_product_flat_indexer_store_cleanup
- catalog_product_outdated_price_values_cleanup
- catalog_product_frontend_actions_flush
- catalog_product_attribute_value_synchronize
- indexer_reindex_all_invalid
- indexer_update_all_views
- indexer_clean_all_changelogs

## Additional

- Content Staging
    - Content staging product team required at this point 
