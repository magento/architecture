# Asynchronous mutations

## Purpose

As a platform, Magento is going to evolve to support higher concurrency of operations.
The best we can do to achieve this is the introduction of asynchronous commands for state modifying operations.
As a result, the storefront application will be capable to process flow without waiting for a response.
Such defer will allow server side to aggregate multiple requests and process them in the most efficient way we can achieve with the current application.
The decision to use asynchronous operations requires changes in mutation interfaces design.

## Schema for asynchronous mutation

Let's consider an example of an asynchronous workflow by using with shopping cart mutations.

### Setting a command 
Actually, because we do not want to process command immediately mutations can not return results.
Instead of the result, such mutation will return a code which will identify a command in execution queue.

```graphql schema
type Mutation {
    createEmptyCart(cartMask: String!): OperationStateInterface!
}
interface OperationStateInterface {
    identifier: String!
}
type OperationAccepted implements OperationStateInterface {
     identifier: String!
}
```

#### Request
```graphql
mutation {
    createEmptyCart($cartMask: String!) {
        identifier
        __typename__
    }
}
```

#### Response
```json
{
   "data":{
      "operation":{
         "identifier":"21e0e65f29bcaf513d2685815916ab7e5ac9d473",
         "__typename__":"OperationAccepted"
      }
   }
}
```

### Tracking execution

Operation status can be checked by using the corresponding query.
Query `queue` receives list of operation identifiers and retrieves statuses for requested operations.
Command that has been invoked and did not complete yet will be marked as *in progress*.

```graphql schema
type Query {
    queue(operationIdentifiers: [String!]!) : [OperationStateInterface]
}

type OperationCompleted implements OperationStateInterface {
     identifier: String!
}

type OperationInProgress implements OperationStateInterface {
     identifier: String!
}
```

#### Request
```graphql
query {
    queue($operationIdentifier: [String!]!) {
        identifier
        __typename__
    }
}
```
#### Response
```json
{  
   "queue":[  
      {  
         "identifier":"21e0e65f29bcaf513d2685815916ab7e5ac9d473",
         "__typename__":"OperationAccepted"
      },
      {  
         "identifier":"95f7cf4623b9a53171e77caffd05b0e54cdc11b3",
         "__typename__":"OperationInProgress"
      },
      {  
         "identifier":"e211fb19a5ce81969aa5ec3ab216d2401c8766ed",
         "__typename__":"OperationCompleted"
      }
   ]
}
```

### Handling failures

In case of failure operation status will be changed in a way to provide client information about the cause of the issue.

```graphql schema
interface FailureReportInterface {
    message: String!
}

type LogicalErrorReport implements FailureReportInterface {
    message: String!
}

type OperationFailed implements OperationStateInterface {
     identifier: String!
     failures: [FailureReportInterface!]!
}

```
#### Request
```graphql 
query {
    queue($operataionIdentifier: [String!]) {
        identifier
        __typename__
        ... on OperationFailed {
            failures {
                message
                __typename__
            }
        }
    }
}
```
#### Response
```json
{
   "queue":[
      {
         "identifier":"25d7d02fa2b73238b098b0254d9ec98a5cd23696",
         "__typename__":"OperationFailed",
         "failures":[
            {
               "__typename__":"LogicalErrorReport",
               "message":"No such entity with email example@example.com"
            }
         ]
      }
   ]
}
```

### Retrieving results

Not all of commands require execution results.
For some operations this is enough to return just an execution status.
In case the application requires to work with the mutation execution results exists two ways for this. Generic and custom.

#### Generic

Generic way provides a possibility to return with success list of types that were changes during mutation execution.
Basically, this is a list of updated entities which should be reloaded for application work. The application can use this result as tags for invalidation local cached entities etc.
Filling of this list is a developer responsibility and does not guarantee by the framework.
To return information about such entities operation state must implement 

```graphql schema
type ChangedEntity {
    type: String!
    identifiers: [String!]
}

type PersistOperationInterface {
    changedEntities: [ChangedEntity!]
}

type PersistOperationCompleted implements PersistOperationInterface, OperationStateInterface {
     identifier: String!
     changedEntities: [ChangedEntity!]
}
```

#### Request
```graphql 
query {
    queue($operataionIdentifier: [String!]) {
        identifier
        __typename__
        ... on PersistOperationCompleted {
            changedEntities {
                type
                identifier
            }
        }
    }
}
```
#### Response
```json
{
   "queue":[
      {
         "identifier":"3d8d487165a5653733e3a3e8bd46783e46b9a58d",
         "__typename__":"PersistOperationCompleted",
         "changedEntities":[
            {
               "Cart":[
                  "29f77a662a7f4d019d5160a9dbe4b83067a75f68"
               ]
            }
         ]
      }
   ]
}
```
#### Custom

A developer can specify custom operation status, which may aggregate success result with output information.
For instance, `CartSaveOperationCompleted` state may, return a corresponding cart as with query.
```graphql schema
type CartSaveOperationCompleted implements OperationStateInterface {
     identifier: String!
     cart: Cart!
}
```
For such cases, a client should be aware of the operation and result that it expects to receive.
This is kind of trade-off between generalized approach and numbers of request to the server.
Such an approach may help to achieve better performance but definitely will cost more in development and testing.
#### Request
```graphql 
query {
    queue($operataionIdentifier: [String!]) {
        identifier
        __typename__
        ... on CartSaveOperationCompleted {
            cart {
                increment_id
                items {
                    sku
                    qty
                }
            }
        }
    }
}
```
#### Response
```json
{
   "queue":[
      {
         "identifier":"3d8d487165a5653733e3a3e8bd46783e46b9a58d",
         "__typename__":"PersistOperationCompleted",
         "cart":{
            "increment_id":"AA000000001",
            "items":[
               {
                  "sku":"simple-product-1",
                  "qty":1
               },
               {
                  "sku":"simple-product-2",
                  "qty":2
               }
            ]
         }
      }
   ]
}
```

### Mimicking async mutation

Actually, with this proposal, we will have to change application workflows to support asynchronous communication. With the great benefit for the future system will face some challenges during the transition period.
Async operations cause the domino effect. There are not enough to change the single mutation, all operation should be rewritten at some moment.
Taking to account backward compatibility policies the migration logic should be defined in advance.
So, the system should be proactive and provides a possibility to write a client application which will be resilient to mutations mode switch.

To achieve this goal we can wrap the existing synchronous operations with `OperationCompleted` state.
As a result, server side mutations will be protected from interface change.
A client application can be written with taking into account of the possibility that operation will not be executed immediately.
