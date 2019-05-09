# Storefront query API Technical Vision

## Purpose

We wan to 

### Desired state


#### Storefront application and admin application are using different databases.

Actually, Magento already has this principle partially implemented.
Admin commands manipulate with raw data sources,
when queries work mostly with indexes which were pre-populated
to return data in the most efficient way.

#### Storefront queries are eventually consistent 

This is a direct consequence of the previous statement.
At the moment Magento follows this principle.
Storefront queries use pre baked data. 
So exists some delay between the moment
when data was submitted to the system
and the moment when data become available for a storefront.

#### Storefront queries support deferred execution

At the moment all storefront implementation are trees.
Blocks generates by instruction from assembled layout.
GraphQL resolving data by traversing query tree.
We can assume that all possible future storefront implementation
will stick to this approach.
This leads us to N+1 problem.
Deferring data retrieving operation can resolve it.
Technically, we do not want to have 
a few similar repeating queries during the single page load.

#### Storefront queries are granular



#### Storefront queries are replaceable
