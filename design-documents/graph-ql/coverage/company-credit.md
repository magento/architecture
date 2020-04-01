# Queries

```graphql

###### Begin: Extending existing types ######
type Company {
    credit: CompanyCredit! @doc(description: "Company credit balance")
    credit_history: CompanyCreditHistory! @doc(description: "Company credit operations history")
}
###### End: Extending existing types ######


###### Begin: Defining new types ######
type CompanyCreditHistory {
    items: [CompanyCreditOperation]! @doc(description: "An array of company credit operations")
    page_info: SearchResultPageInfo! @doc(description: "Metadata for pagination rendering")
}

type CompanyCreditOperation {
    date: String! @doc(description: "The date of the company credit operation")
    # Operation may be of enum type
    operation: String! @doc(description: "The type of the company credit operation")
    amount: Money @doc(description: "The amount fo the company credit operation")
    balance: CompanyCredit! @doc(description: "Credit balance after the company credit operation")
    purchase_order: String @doc(description: "Purchase order number associated with the company credit operation")
    updated_by: String! @doc(description: "The name of the person submitting the company credit operation")
}

type CompanyCredit {
    outstanding_balance: Money! @doc(description: "Outstanding company credit")
    available_credit: Money! @doc(description: "Available company credit")
    credit_limit: Money! @doc(description: "Company credit limit")
}
###### End: Defining new types ######

```



