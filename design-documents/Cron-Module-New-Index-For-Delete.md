# Overview

The `cron_schedule` table sometimes has a large amount of rows in it.  This can be for several reasons.  Sometimes there is bug outside of our control that cause the data aging policies not to delete the old data like it should.  Sometimes it is bugs in our infrastructure code that causes cron to run more or less often than it should.  Other times, it is just because the customer wants a very large data aging policy so that they can look at the data in the table as a historical log.

## Long Deletes (even when no old data)

Having a lot of records in this table causes problems with some of our SQL queries in the cron-module.  This propososal's goal is to help fix the problem when the DELETE query has to search several rows to see what needs to be deleted, but no rows, or only a small amount of rows need to be deleted.

Adding a multi column index (job_code, status, scheduled_at) to this table will increase the performance in this use-case a lot.  For example, when I had about 800,000 rows, and 0 needed to be deleted, It took about 13 seconds to delete, but with the addition of this index, it was less than a second.

### Benefits
1. A lot faster delete queries.

### Disadvantages
1. Having another index will make inserts slightly slower.  I don't think it will be impactful enough, but we may want to do performance tests just to make sure.

# Summary
When there is lots of rows in the database, but not that many that are old enough to be deleted, the delete queries spend most of their time searching.  This new index fixes that, so it makes the query faster, but also reduces load on the SQL server.

# prototype
To see an example of this and my other proposed changes, you can look at the prototype I made.
https://github.com/magento/magento2/compare/2.2-develop...JacobBrownAustin:MAGECLOUD-2503-2.2-develop?expand=1
