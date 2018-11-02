## Add GiftCardRedeemerInterface interface as a service contract
#### Why?
Redeeming a gift card is an existing part of the gift card module functionality,
API should reflect that and allow for client code to execute it and/or use it as Web API.
#### Situation
Right now "redeem" method only exists in the Giftcardaccount model and
actually does not contain all the logic required to redeem a gift card - only Customer\Index controller has all the logic -
it would be much better if a service had a method for this functionality with all the required logic in it
and to remove the logic from the controller by leaving only the GiftCardRedeemerInterface::redeem() call.
Also there's additional logic regarding gift card usages limits which is included in other methods of GiftCardAccountManagementInterface,
but right now has to be duplicated in the controller - the new redeem method would include it.
#### Backward compatibility
Adding new API to a minor version does not break bc
