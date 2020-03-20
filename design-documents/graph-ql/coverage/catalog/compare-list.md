# Use cases
* Buyer can add a product to the compare list.
* If Buyer logging in or creating an account, an active compare list should be transferred to the revealed customer account.
* Loginned buyer can access the latest compare list if the list has not been cleared with the last session. 
* The buyer can request a compare list which has to return as a bare minimum price, link to a product page, name, and SKU.


* A Compare list ID is a client-side generated ID.
* A new compare list could be created by calling `addItemsToCompareList`.
* If a shopper calls  `addItemsToCompareList` with the new id the previous cart will be retired and replaced with a new one.


![compare-list.graphqls](compare-list/compare-list.png)

# Non functional requirements:
* Compare list should return enough data to perform the following command with no additional calls: 
  * adding product to cart.
  * adding product to wishlist.

