## Configuration 

See https://docs.magento.com/user-guide/configuration/customers/reward-points.html

The following settings should be accessible via `storeConfig` query:
- Reward points functionality status on the storefront: enabled/disabled
- Enable reward points history for the customer
- Reward points redemption minimum threshold
- Whether to show a message in shopping cart about the rewards points earned for the purchase, as well as the customer’s current reward point balance
- Number of points customer gets for registration
- Number of points for newsletter subscription 
- Number of points for referral, when invitee registers on the site 
- Number of points for referral, when invitee places an initial order on the site
- Number of points for writing a review
- Reward points default subscription status 

Scenarios which may need these settings include:
- Reward program promotions and details
- Customer registration
- Rendering of the reward points section in the customer account

## Use cases

### View reward points information in customer account

The following information should be available to customer in his account when reward points functionality is enabled on the site:
 - Balance in points and currency
 - Exchange rate from points to currency (redemption rate)
 - Exchange rate from currency to points (earning rate)
 - Balance history, should include the following fields:
   - Balance in points
   - Amount in currency
   - Balance change in points
   - Reason for balance change
   - Date
 - "Balance Updates" email subscription status
 - "Points Expiration Notification" email subscription status

 
