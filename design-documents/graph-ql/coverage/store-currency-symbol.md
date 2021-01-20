# Coverage for currency symbol

Magento allows to use custom currency symbol.The current graphql implementation does not fully support custom currency symbol feature. The currency query only returns base currency symbol and current store default currency symbol.
 `available_currency_codes` should be deprecated and instead `available_currency` field should be added, or both should be kept
 
 ## Schema
```graphql
type CurrencyObject {
    currency_code: String
    currency_symbol: String
}

type CurrencySettings {
    base_currency_code: String
    base_currency_symbol: String
    default_display_currency_code: String
    default_display_currency_symbol: String
    exchange_rates: [ExchangeRate]
    available_currency: [CurrencyObject!]!
}

type Currency @deprecated {
 available_currency_codes: [String]
 base_currency_code: String
 base_currency_symbol: String
 default_display_currency_code: String
 default_display_currency_symbol: String
 exchange_rates: [ExchangeRate]
}

type Query {
 currency: Currency @deprecated(description: "use `currency_settings` instead")
 currency_settings: CurrencySettings
}
```

All the currency symbol fields should be moved to CurrencySymbolGraphQl module and schema stitched to currency type.