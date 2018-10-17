## Add GiftCardAccountManagementInterface::redeem() method
#### Why?
Redeeming a gift card is an existing part of the gift card module functionality,
API should reflect that and allow for client code to execute it and/or use it as Web API.
#### Situation
Right now "redeem" method only exists in the Giftcardaccount model and
actually does not contain all the logic required to redeem a gift card - only Customer\Index constroller has all the logic -
it would be much better if a service had a method for this functionality with all the required logic in it
and to remove the logic from the controller by leaving only the GiftCardAccountManagementInterface::redeem() call.
Also there's additional logic regarding gift card usages limits which is included in other methods of GiftCardAccountManagementInterface,
but right now has to be duplicated in the controller - the new redeem method would include it.
#### Backward compatibility
We strive for 3rd party devs to be able to customize Magento logic only by implementing interfaces, and in this case adding a new method
to an API would hurt backward compatibility BUT right now it's virtually impossible to do that and 3rd party devs
are ussually just extending our classes and in this case - adding this method doesn't hurt anybody.
