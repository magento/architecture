## Glossary

poison pill
: message that added after list of events in Data Storage that signals that consumer should die.

### Overview

Right now our consumer depends on config/objects in-memory state. After application was bootstrapped it initialize a lot of object that are always kept in memory after first load - config, websites, etc.., since our consumers are long-living and their live time limited only by message count it could happens that our consumer keep outdated im-memory config/objects states. Considering that, we need to introduce the way to reinit them.

### Design

Basic implementation of how the pill will be added should be implemented inside Magento Framework. We will share the api that let each module call whenever it needed.

Api Interface - *PoisonPillInterface* will be inside Magento Framework, implementation in MessageQueue module.

PoisonPillInterface
```php
/**
 * Put poison pill inside DB.
 *
 * @throws \Exception
 * @return int
 */
public function put();
```

Before process new message, consumer will compare version of PoisonPill in-memory state with latest in DB. If version is different consumer dies and next cron:run will start new instance of consumer.

#### Acceptance Criteria Fulfillment

1. If consumer takes message to process after poison pill was placed, consumer must be reinited.

#### Extension Points and Scenarios

We will put poison pill into Data Storage based on common Magento extension points. 

