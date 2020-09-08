## Configuration 

See https://docs.magento.com/user-guide/configuration/customers/reward-points.html

The following settings should be accessible via `storeConfig` query:
- Reward points functionality status: enabled/disabled
- Reward points functionality status on the storefront: enabled/disabled
- Enable reward points history for the customer
- Reward points redemption minimum threshold
- Whether customer earns points for shopping according to the reward point exchange rate. In Luma this also controls whether to show a message in shopping cart about the rewards points earned for the purchase, as well as the customer’s current reward point balance
- Number of points customer gets for registration
- Number of points for newsletter subscription 
- Number of points for referral, when invitee registers on the site 
- Maximum number of registration referrals that will qualify for rewards
- Number of points for referral, when invitee places an initial order on the site
- Maximum number of order placements by invitees that will qualify for rewards
- Number of points for writing a review
- Maximum number of reviews that will qualify for the rewards

Scenarios which may need these settings include:
- Reward program promotions and details
- Customer registration
- Rendering of the reward points section in the customer account

```graphql
{
  storeConfig {
    magento_reward_general_is_enabled
    magento_reward_general_is_enabled_on_front
    magento_reward_general_publish_history
    magento_reward_general_min_points_balance
    magento_reward_points_order
    magento_reward_points_register
    magento_reward_points_newsletter
    magento_reward_points_invitation_customer
    magento_reward_points_invitation_customer_limit
    magento_reward_points_invitation_order
    magento_reward_points_invitation_order_limit
    magento_reward_points_review
    magento_reward_points_review_limit
  }
}
```

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
 
 ```graphql
{
  customer {
    reward_points {
      balance {
        points
        money {
          value
          currency
        }
      }
      exchange_rates {
        earning
        redemption
      }
      subscription_status {
        balance_updates
        points_expiration_notifications
      }
      balance_history {
        balance {
          points
          money {
            value
            currency
          }
        }
        points_change
        change_reason
        date
      }
    }
  }
}
```
 
### Apply/remove reward points to/from cart and view applied reward points summary

Apply reward points to the cart and view applied reward points balance:

```graphql
mutation {
  applyRewardPointsToCart(cartId: "existing-cart-id") {
    cart {
      applied_reward_points {
        money {
          currency
          value
        }
        points
      }
    }
  }
}
```

Remove applied reward points from the cart and view applied balance:

```graphql
mutation {
  removeRewardPointsFromCart(cartId: "existing-cart-id") {
    cart {
      applied_reward_points {
        money {
          currency
          value
        }
        points
      }
    }
  }
}
```
