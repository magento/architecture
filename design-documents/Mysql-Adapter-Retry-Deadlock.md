# Overview

Sometimes, database transactions results in deadlocks that cannot be avoided.  When working on fixing some locking issues in cron-module, I found that even though I can reduce the chance of them occurring, they are still unavoidable.  When a deadlock happens, MySQL/MariadDB, and probably most database systems, have deadlock detection algorithms that detect the deadlock situation as it happens, and then kills one of the two transactions in contention - allowing the other to continue on.  When this happens in a SQL query that is a single-query transaction, it is safe to retry that transaction until it succeeds.  The Mysql adapter in Magento already tries retrying the query in other situations (gone away, and lost connect). 

## Retry after deadlock.

Because there is already code in Magento's Mysql adapter that retries under other situations, this is an easy thing to add.  Of course we only want to retry the query if there were no previous queries in the transaction.  (It looks the current code will retry even if it is part of a transaction, which looks like a bug to me, so I may try to fix that as part of this.)

### Benefits
1. Commands that sometimes fail because of deadlocks will now fail much less often.  It will keep retrying until MAX_CONNECTION_RETRIES which is currently set to 10.

### Disadvantages
1. None that I know of.

# Summary
It is safe to retry single query transactions that fail because of deadlocks.  This will help our program be more robust when these situations are unavoidable.

# prototype
To see an example of this and my other proposed changes, you can look at the prototype I made.
https://github.com/magento/magento2/compare/2.2-develop...JacobBrownAustin:MAGECLOUD-2503-2.2-develop?expand=1
