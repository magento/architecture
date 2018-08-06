# Overview

The `cron_schedule` table sometimes has a large amount of rows in it.  This can be for several reasons.  Sometimes there is bug outside of our control that cause the data aging policies not to delete the old data like it should.  Sometimes it is bugs in our infrastructure code that causes cron to run more or less often than it should.  Other times, it is just because the customer wants a very large data aging policy so that they can look at the data in the table as a historical log.

## trySetJobUniqueStatusAtomic() slow and lock issues

Having a lot of records in this table causes problems with some of our SQL queries in the cron-module.  This propososal's goal is to help fix the problem related to the UPDATE query used in trySetJobUniqueStatusAtomic().  In this proposal, we solve this problem by creating a new table where we peform this functionality, thus avoiding performance and lock issues.

When the UPDATE query runs, it does a left outter join against the same table, with a complicated WHERE clause.  The query works (though there is one race condition bug I found with it), but it has a huge performance impact on large tables.  That is because as the query is running, it has to readlock all of the rows it has searched, until it can find that the query works, or fails.  This can cause both query lock timeout issues as well as deadlock on itself.

In order to solve this problem, I propose that we use a small table that keeps track of which job_code are locked.  This is much, much quicker, and doesn't have any locking issues against the other table at all because it is completely separate.

The table name:
`cron_schedule_jobcode_lock`
  
The tables has 3 columns:
A) `job_code` which is the primary key, and thus unique.
B) `schedule_id` so that we can quickly,
C) `execute_at` so that we know how long it has been running, because after a certain amount of time, we assume that it is no longer running.

There's two indexes:
A) The primary key, `job_code` which is implicitly unique.
B) I also created an index on `schedule_id` which normally has no affect, but if someone wants to go crazy and create millions of distinct `job_code`s, it will still scale and perform well for them.

### Benefits
1. This is a lot faster than the UPDATE with LEFT OUTTER JOIN on a massive table that is in use by other commands as cron:run runs in parallel.
2. Less CPU/Memory resources consumed each time trySetJobUniqueStatusAtomic() is called.
3. The query is easier to read because we no longer do any JOINs or subqueries or unions.


### Disadvantages
1. In use cases where there aren't many rows in cron_schedule, this may seem like overkill.  But, it shouldn't cause any noticeable negative impact.

Also, if anyone is wondering why I didn't create the schedule_id as a foreign key, that is because I believe that if it were a foreign key that had a trigger set to automatically delete on foreign key being deleted, I felt that would trigger the same deadlock issues that I was trying to fix.  (Please feel free to prove me wrong!)

So, instead of a foreign key, I added a method called `setStatus` in Cron\Model\Schedule that does the deleting using the `schedule_id` index to keep it fast.

# Summary
The UPDATE queries in trySetJobUniqueStatusAtomic() can have a huge performance impact on large cron_schedule table.  Creating a new table for this method saves us a lot of time, and other resources.

# prototype
To see an example of this and my other proposed changes, you can look at the prototype I made.
https://github.com/magento/magento2/compare/2.2-develop...JacobBrownAustin:MAGECLOUD-2503-2.2-develop?expand=1
