# Overview
The cron is one of the most business critical components of Magento, offloading resource 
intensive tasks from the user request into the backend and allowing scheduling long running
processes.

The business critical element of the cron needs better supporting in the foundation of the module and
supporting documentation giving better control to developers, technology providers and eCommerce managers.

# Problem
The cron implementation in it's current existence doesn't provide the business critical foundation 
or visibility required for a fully functioning eCommerce system.

# Solution - Business Critical Foundation
The cron should be built with a  *business critical* foundation. It's responsible for some of the
most critical processes on a Magento implementation; from transactional emails to integrations.

1. Separation
2. Visibility
3. Education

## 1. Separation
Separation of business critical and none business critical processes as part of the core architecture
of the cron. This should be multi threaded "out the box" running business critical processes
in their own threads (Emails, Indexes).

At the foundation no rogue process should ever prevent a business critical operation.

- Multi thread core cron processes out the box
- Process timeouts (Default X minutes and configurable by job) 
- Retry limits
- Updated documentation to encourage multi thread setup as standard


## 2. Visibility
On Magento 1 we had the AOE Scheduler which provided us a considerably enhanced user experience. This
module isn't available for Magento 2. Allowing webmasters and developers to clearly see what processes
are running, scheduled and complete gives them greater control and reduces support requests and increases
confidence in the platform.

- Admin visible timeline of scheduled cron tasks (Scheduled, in progress, complete, errored)
- Ability to schedule, delete, remove, pause specific cron tasks.
- Cleanup of redundant processes to reduce noise
- Logging and Alerting

Alternative approach would be a cron dashboard displaying processes in progress, queued and error conditions.

## 3. Education
- Technical documentation for effectively implementing crons into Magento modules
- Setup documentation for correctly implementing multi threaded cron (Standard setup)

# Additional discussion
1. Should RabbitMQ be standard/recommended for certain tasks such as transactional emails and indexes?
2. New Relic currently logging as *unknown* we have an implementation but the Cron module requires work to allow it

# Examples

## Example 1 - Rogue process
A process that connects to an API such as a payment provider could end up getting stuck in a loop
and blocking any other process from progressing currently there is no mechanic in place to
a: Allow other business critical processes to continue aside from defining multiple crontab processes in groups
b: Report or stop the rogue process from continuing until manually spotted
c: No visibility of the processes aside from checking the database
d: No reporting or alerting of the above condition 