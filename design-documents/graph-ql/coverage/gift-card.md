# Problem

1. For a gift card, 6 entered options are possible. Sender name/email, Recipient name/email, Message, Custom gift card amount.
   Not all fields are needed/required for the clients to fill in. This options metadata should be made available to the clients when adding a giftcard product to cart.
2. When clients send this data they need a uid to distinguish each option and uid is not supposed to be generated on the client side. So when we pass in the options metadata a server generated uid should be made available to the clients.

## Expecations

This is in reference to this PR [Solution Architecture](https://github.com/magento/architecture/blob/master/design-documents/graph-ql/coverage/add-items-to-cart-single-mutation.md)
on single mutation for adding items to cart. So any changes proposed should respect the input interfaces described in this PR.

## Proposal

Add a new field gift_card_options under the GiftCardProduct type. The values can leverage the exisiting CustomizableOptionInterface.

```
type GiftCardProduct {
    gift_card_options: [CustomizableOptionInterface!]!
}
```

On querying the gift_card_options the sample response should look like this.

```
{
    gift_card_options: [
        {
            title: "Sender Name",
            required: true
            "__typename": "CustomizableFieldOption"
            value: {
                uid: "Y3VzdG9tLW9wdGlvbi8xNzE"  #base64_encode(giftcard/Magento\GiftCard\Model\Giftcard\Option::KEY_SENDER_NAME)
            }
        },
        {
            title: "Sender Email",
            required: true
            "__typename": "CustomizableFieldOption"
            value: {
                uid: "Y3VzdG9tLW9wdGlvbi8xNzE"  #base64_encode(giftcard/Magento\GiftCard\Model\Giftcard\Option::KEY_SENDER_EMAIL)
            }
        },
        # Recipient name and email should not be returned if the product type is Physical.
        {
            title: "Recipient Name",
            required: true
            "__typename": "CustomizableFieldOption"
            value: {
                uid: "Y3VzdG9tLW9wdGlvbi8xNzE"  #base64_encode(giftcard/Magento\GiftCard\Model\Giftcard\Option::KEY_RECIPIENT_NAME)
            }
        },
        {
            title: "Recipient Email",
            required: true
            "__typename": "CustomizableFieldOption"
            value: {
                uid: "Y3VzdG9tLW9wdGlvbi8xNzE"  #base64_encode(giftcard/Magento\GiftCard\Model\Giftcard\Option::KEY_RECIPIENT_EMAIL)
            }
        },
        # Message can be optional/required
        {
            title: "Message",
            required: false/true
            "__typename": "CustomizableFieldOption"
            value: {
                uid: "Y3VzdG9tLW9wdGlvbi8xNzE"  #base64_encode(giftcard/Magento\GiftCard\Model\Giftcard\Option::KEY_MESSAGE)
            }
        },
        # Custom giftcard amount can be optional/required
        {
            title: "Custom Giftcard Amount",
            required: false/true
            "__typename": "CustomizableFieldOption"
            value: {
                uid: "Y3VzdG9tLW9wdGlvbi8xNzE"  #base64_encode(giftcard/Magento\GiftCard\Model\Giftcard\Option::KEY_CUSTOM_GIFTCARD_AMOUNT)
            }
        },
    ]
}
```
This enables the clients to know what options to fill in along with the uids. 
When the gift card is added to the cart the entered options can be passed in the same way as [Solution Architecture](https://github.com/magento/architecture/blob/master/design-documents/graph-ql/coverage/add-items-to-cart-single-mutation.md).
