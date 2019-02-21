## Exception handling in service communication

### Problem

We need to handle network exceptions that may happen in communication between services.

This proposal aims to answer the following questions:
* Should we use PHP exception class name in API responses or exception codes
* What should we do with network exceptions (throw or suppress)
* How do we limit amount of services we need to redeploy when one of the services start to return more exceptions

As we are leaning towards using REST for communication between services, so this proposal will focus on REST.

### Handling exceptions in API best practices

For REST APIs the common practice is to incorporate response code in HTTP header. However, it's not enough for a client to do optimal exception handling as it's not granular enough. This is why most APIs include error message and exception code transferred as a part of message.

#### Examples

Twitter

```
{
    "errors": [
        {
            "code": 215,
            "message": "Bad Authentication data."
        }
    ]
}
```

Facebook

```
{
    "error": {
        "message": "An active access token must be used to query information about the current user.",
        "type": "OAuthException",
        "code": 2500,
        "fbtrace_id": "DzkTMkgIA7V"
    }
}
```

Twilio

```
<?xml version='1.0' encoding='UTF-8'?>
<TwilioResponse>
    <RestException>
        <Code>20404</Code>
        <Message>The requested resource /2010-04-01/Accounts/1234/IncomingPhoneNumbers/1234 was not found</Message>
        <MoreInfo>https://www.twilio.com/docs/errors/20404</MoreInfo>
        <Status>404</Status>
    </RestException>
</TwilioResponse>
```

### Exception class name or class name

Existing REST API returns only error message, so we need to add exception class name or error code in addition to HTTP response code. This would allow different consumers of the service handle errors or understand what exception need to be handled or which exception type need to be thrown.

Return exception class names
Pros
    * No mapping needed, exceptions from API can easily be caught or thrown
Cons
    * If consumer or producer service has non PHP or Magento implementation PHP exception class name doesn't make a lot of sense
    
Return error code
Pros
    * Error codes make sense regardless of technology
Cons
    * We would have to maintain some sort of mapping to make it work.
    
Replaceability of services is one of the goals of the project, so the second option makes more sense.

There are could be third option, which is combination of both options (what Facebook does), it also makes sense but we have some data duplication in this case.

### Throw or suppress exceptions in consumers

All exceptions except network exceptions we should map to exceptions declared on the service contracts that communicate through the network.

When we consume services in BFF we could benefit from knowing that network exception happen, because we could make UI more friendly. For instance, we were not able to calculate totals on shopping cart page. We can still show shopping art and display the message that we retrying total calculation.

In order to make this improvements in UI, we would have to declare new type of exception on service contracts `DataRetrievalFailedException`. This is would be backwards incompatible change for monolith.

Possible options:
    1. Don't declare and don't throw new exception and loose ability to do UI improvements.
    2. Declare exception, but don't do major version bump. As on monolith this exception will not be thrown the issue is small, it's just hacky. 
    3. Don't declare but throw exception. Backwards compatible on monolith, also hacky.
    4. Declare exception and do major version bump. This will bring a lot of issues.
    5. Introduce new service contracts that will have declared exception, these may lead to naming problems.
    
For now, I suggest to go with option #1. If we would want to handle network communication exceptions in UI, we could do #5 or #2.

### Limiting amount of services to deploy after adding new exception in our of the producers

Apply API versioning. New exception should be thrown in the new version API. After producer service redeployed, clients can be switched to a new API one by one.

## Resources

* [Top REST API Best Practices](https://code-maze.com/top-rest-api-best-practices/)
