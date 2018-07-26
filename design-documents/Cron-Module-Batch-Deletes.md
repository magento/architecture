# Overview

The `cron_schedule` table sometimes has a large amount of rows in it.  This can be for several reasons.  Sometimes there is bug outside of our control that cause the data aging policies not to delete the old data like it should.  Sometimes it is bugs in our infrastructure code that causes cron to run more or less often than it should.  Other times, it is just because the customer wants a very large data aging policy so that they can look at the data in the table as a historical log.

## Long Deletes (when many rows are to be deleted)

Having a lot of records in this table causes problems with some of our SQL queries in the cron-module.  This propososal's goal is to help fix the problem when the DELETE query tries to delete massive ammounts of rows in on single SQL query.  When the DELETE query is deleting hundreds of thousands or more rows at once, it can take a minute, or several minutes to run.  This causes other commands to get lock wait timeout errors.  It also increases the chance of deadlocks occurring. 

In order to remove the lock time, there are two main solutions to this problem.

A) Batching.  Instead of trying to delete as many matching rows at once, we will delete at most, a certain number.  In my testing, I've been using 1024 as the limit, but we might choose a more optimal number after benchmarking our different use-cases.  I think this solution is pretty typical in other enterprise database application to solve this same problem.
B) Read Committed.  We could choose to change the Transaction Isolation Level to "READ COMMITTED" for the delete queries.  I haven't tested this one because I think it is a bad idea.

## Batching
In this example, I will use 1024 as the LIMIT, but this number should be changed to a better number based on performance testing and measurement.
A normal SQL command my look like:
DELETE FROM `cron_schedule` WHERE (status = 'success') AND (job_code in ('indexer_reindex_all_invalid', 'indexer_update_all_views', 'indexer_clean_all_changelogs', 'magento_targetrule_index_reindex')) AND (created_at < '2018-07-11 20:42:38') LIMIT 1024
After this proposal, it would look like DELETE FROM `cron_schedule` WHERE (status = 'success') AND (job_code in ('indexer_reindex_all_invalid', 'indexer_update_all_views', 'indexer_clean_all_changelogs', 'magento_targetrule_index_reindex')) AND (created_at < '2018-07-11 20:42:38') LIMIT 1024

Ending the delete query after 1024 are found, and then starting a new delete query allows other queries that are also using the same rows in the table to be able to conintue in parallel.  It also helps reduce the chance of deadlock because less rows are locked at any one moment in time.

### Benefits
1. Prevents lock timeouts because we allow other queries to execute in between the 1024 batches.
2. Reduces, but does not eliminate deadlocks.

### Disadvantages
1. In cases where there are several batches to be deleted, it does take more time over all for all of the batches to be deleted.  Though, in such cases, other SQL queries running at the same time will be timing out.  I think the benefit of not getting errors and better parallelization outweighs this.

## Read Commited
It should be possible to also solve this problem in another way by setting the Transaction Isolation Level to READ COMMITTED.  There are several disadvantages to this solution, but I felt like I needed to put it here for the sake of completeness.

### Benefits
1. Other queries can run in parallel at the same time despite locks.  (Note: I haven't actually tried this, so it may not actually be this nice.)

### Disadvantages
1. This might by database specific.  I know it works for MySQL/MariaDB, but I don't know how/if it works with others, including GaleraDB that we use on cloud.
2. It seems like it would go against best practice.
3. We would have to keep track of the isolation levels and change back once it is done.

# Summary
In my opinion, batching the deletes seems like an easy solution to this problem.  I will also be making another proposal for another solution using a new index that also helps solve this problem in a different way for a different use case.
