# Coverage for currency symbol

Magento allows to use custom currency symbol.The current graphql implementation does not fully support custom currency symbol feature. The currency query only returns base currency symbol and current store default currency symbol.
 `available_currency_codes` should be deprecated and instead `available_currency` field should be added, or both should be kept
 
 ## Schema
```graphql
type CurrencyData {
    currency_code: String
    currency_symbol: String
}

type Currency {
    available_currency_codes: [String] @deprecated
    base_currency_code: String
    base_currency_symbol: String
    default_display_currency_code: String
    default_display_currency_symbol: String
    exchange_rates: [ExchangeRate]
    available_currency: [CurrencyData!]!
}
```

All the currency symbol fields should be moved to CurrencySymbolGraphQl module and schema stitched to currency type.