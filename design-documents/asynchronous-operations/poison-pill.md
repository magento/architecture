## Glossary

poison pill
: message in Data Storage that signals that consumer should die

### Overview

Right now our consumer depends on config/objects in-memory state. After application was bootstrapped it initialize a lot of object that are always kept in memory after first load - config, websites, etc.., since our consumers are long-living and their live time limited only by message count it leads to wrong im-memory config/objects states and we need to introduce the way to reinit them.

### Design

Before process new message consumer should ask Data Storage if there are poison pill for him. If yes, consumer dies and next cron:run will start new instance of consumer.

#### Acceptance Criteria Fulfillment

1. If consumer takes message to process after poison pill was placed, consumer must be reinited

#### Extension Points and Scenarios

We will put poison pill into Data Storage based on common Magento extension points like events and plugins
