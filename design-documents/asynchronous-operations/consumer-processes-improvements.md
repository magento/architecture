# Improvements to message queue consumer processes

## Remark

The naming suggested in this proposal for the new configuration options and for the new CLI command aren't final yet.

## The current situation

Currently all defined message queue consumers are getting spawned by a cronjob called `consumers_runner`. You can optionally choose to disable this entirely or only allow specific consumers to run based on some config settings in the `app/etc/env.php` file (see [docs](https://devdocs.magento.com/guides/v2.3/config-guide/mq/manage-message-queues.html#configuration)).  
This `consumers_runner` job first looks around on the server(s) to see if there are already running consumer processes for the same queue. If that's the case it won't spawn a new one, but if no consumer process is found for a certain queue, it will spawn one. By default it will listen for maximum 10.000 messages and if those 10.000 messages have been handled, it will kill itself. The next time the cronjob `consumers_runner` triggers, it will spawn a new process.

There is also a poison pill feature in Magento, which is basically just a random hash stored in the database in the table `queue_poison_pill`. If some configuration value gets changed using the Magento backend, the poison pill version is changed. 
The running consumer processes check on this version, but only when a new message is in the queueu. If a consumer discovers a new message in the queue, it first checks if the poison pill version has changed. If the version wasn't changed, the message will be handled. If the version did get changed, the consumer will kill itself and the cronjob will spawn a new one on its next run and only then will handle that message.

Magento 2.3.0 Open Source shipped with the message queue system and made use of it when using the bulk api import feature.  
And with Magento version 2.3.2 some already existing features were converted to make use of the message queue system. These are tasks which are triggered using the backend of Magento which can potentially take a while to execute. In order to prevent the webserver or php-fpm to run against a timeout, it was chosen to send these tasks to the message queue system and let these tasks get executed asynchronously. Some examples are:

- Generating coupon codes
- Mass editing products
- Exporting data
- ...

Currently in Magento 2.3.2, 4 consumer processes get spawned by the cron system. Each of these processes take memory and cpu and regularly queries the database (or RabbitMQ if that broker is being used) to see if new messages are available and then process them.


## Problem 1: not enough options to keep consumers under control

### The problem

There is currently too little control over these consumer processes.  
What if people only very irregularly use one of these features mentioned above, let's say they only once a year export all their products to check their inventory levels.  
Then you have some consumer process sitting there, doing nothing for 364 days in the year, wasting precious cpu cycles and taking up precious memory, until finally once a year the shopowner decides to execute a certain task it can execute.

### The suggested solution

The suggestion is to give more control per consumer to the shopowner or developers managing the shop.  
We could add some additional options to the consumer processes to keep them more under control.  
Currently there is one limit available: `max-messages`. If that number of messages get processed, the consumer will kill itself.  
I'd like to suggest another limit which we can set:

- `max-idle-time`: if no message was being handled in xx seconds, then kill yourself

Next to these limits, a configurable sleep time might be nice:

- `sleep`: xx milliseconds to sleep before checking if a new message is available (currently this is [hardcoded to 1 second](https://github.com/magento/magento2/blob/2.3.2/lib/internal/Magento/Framework/MessageQueue/CallbackInvoker.php#L59))

I'd also like to see an option defined on the consumer, but being used by the `consumers_runner` cronjob:

- `only-spawn-when-message-available`: the idea is that the `consumers_runner` job checks the queue before spawning a consumer, to see if there is actually a message pending in the queue. If there is one, then go ahead and spawn a consumer (only if one isn't already running). If there isn't a message in the queue, then don't spawn a consumer.

### Some options combined

The problem outlined above, where a specific consumer only needs to run very infrequently could be solved by combining the options:

- `only-spawn-when-message-available`
- `max-idle-time`

The consumer will only spawn when it is needed, and it will kill itself when it wasn't active for a certain period.  
That should save some precious server resources.

### Making these options configurable and have some defaults

These options should be configurable per consumer type.
Some sensible defaults could be set in the [`queue_consumer.xml`](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/message-queues/config-mq.html#queueconsumerxml) file for some of these options.  
Next to that, developers or shopowners should be able to override these values per consumer type. At least being able to override them in the `app/etc/env.php` file would be nice, but a backend interface for making these things configurable would also be very nice.

### Other solution currently being worked on by Magento

[MC-19250](https://github.com/magento/magento2/commit/269b47af3e37fbbe76e9f38d45fdb0cf969d45e3) was recently introduced in the code base. ([Docs](https://github.com/magento/devdocs/pull/5289))  
Which introduces an option `consumers_wait_for_messages`. When that option is set to false, a consumer will stop the moment it doesn't find any new messages.

The problem with this option, is that every time the cronjob `consumers_runner` runs, it will spawn a new consumer process, the consumer checks if messages are available and if none found it will kill itself. So this means it will spawn unneeded processes which take up memory, live for a very short period and then disappear again.  

In this proposal, the flag `only-spawn-when-message-available` would run logic to check *before* the consumer process gets spawned and not *after*.

## Problem 2: deployment problems

### The problem

When deploying a new version of Magento/third party/custom code to a server, we can potentially run against a problem where consumers are still using old code loaded in memory. This is not only the code the consumer is running itself, but might also be Magento core code itself for the consumer processes themselves (which might change when upgrading Magento to a newer version).

Also when using a capistrano-style deployment where you have a symlinked a `current` directory pointing to a certain release directory, after a new deploy the running consumers will still supposedly run from the old release directory referencing a `.pid` file containing its process id in a directory which is in the old release directory. The cronjob `consumers_runner` will go searching for that file in the new release directory and won't find it. Causing a second consumer to start up, even though the old one is still running. (This is probably already fixed by this unreleased commit: [MC-18477](https://github.com/magento/magento2/commit/1d9e07b218c7c8ad1f05706828cb2dd47d2d2d58))

### The suggested solution

The suggestion here would be to update the poison pill version using a command.  
That way, consumers which have messages in the queue, will see an updated poison pill version, stop and get spawned again by the cronjob.  
And for consumers not having messages in the queue, they will either stop after a single message was put in the queue eventually, or if they make use of this new `max-idle-time` flag, they will stop when that time is reached.

I'm seeing this as some new command being added to `bin/magento` (suggestion: `queue:poison-pill:update-version` ?).
