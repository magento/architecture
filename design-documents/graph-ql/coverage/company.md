# Tricky things

`Sales representative` is an admin user(merchant side) who is assigned to work with the company. Thus we can't use exsting `Customer` type for sales representative 

# Behavioral Notes
When a feature is disabled (e.g. Company) queries/mutations related to that feature should return a null response, along with an error indicating that the feature is not available.

Update/Create mutations should return the entity they acted upon.
Delete mutations return a field indicating success or failure. In the case of failure, an error should be returned indicating the error that occurred (suitable for storefront).

# Queries

```graphql
type Query {
    company: Company  @doc(description: "Company assigned to the currently authenticated user")
    checkCompanyEmail(email: String!): CompanyEmailCheckResponse  @doc(description: "Check if an email is valid for company registration")
    checkCompanyAdminEmail(email: String!): CompanyAdminEmailCheckResponse  @doc(description: "Check if an email is valid for company admin registration")
    checkCompanyUserEmail(email: String!): CompanyUserEmailCheckResponse  @doc(description: "Check if an email is valid for company user registration")
    checkCompanyRoleName(name: String!): CompanyRoleNameCheckResponse  @doc(description: "Check if a role name is valid for company")
}

type Company @doc(description: "Company entity output data schema.") {
    id: ID! @doc(description: "Company id.")
    name: String @doc(description: "Company name.")
    email: String @doc(description: "Company email address.")
    legal_name: String @doc(description: "Company legal name.")
    vat_id: String @doc(description: "Company VAT/TAX id.")
    reseller_id: String @doc(description: "Company re-seller id.")
    legal_address: CompanyLegalAddress @doc(description: "Company legal address.")
    company_admin: Customer @doc(description: "An object containing information about Company Administrator.")
    sales_representative: CompanySalesRepresentative @doc(description: "Company sales representative.")
    payment_methods: [String] @doc(description: "List of payment methods available for a Company.")
    users(
        filter: CompanyUsersFilterInput @doc(description: "Identifies which company users to search for and return."),
        pageSize: Int = 20 @doc(description: "Specifies the maximum number of results to return at once. Defaults to 20."),
        currentPage: Int = 1 @doc(description: "Specifies which page of results to return. The default value is 1."),
    ): CompanyUsers @doc(description: "Information about the company users.")
    user(id: ID!): Customer @doc(description: "Returns company user by id.")
    roles(
        pageSize: Int = 20 @doc(description: "Specifies the maximum number of results to return at once. Optional. Defaults to 20."),
        currentPage: Int = 1 @doc(description: "Specifies which page of results to return. The default value is 1."),
    ): CompanyRoles!  @doc(description: "Returns the list of defined roles at Company.")
    role(id: ID!): CompanyRole  @doc(description: "Returns company role by id.")
    acl_resources: [CompanyAclResource]  @doc(description: "Returns the list of all permission resources.")
    structure(
        root_id: ID = 0 @doc(description: "Tree depth to begin query")
        depth: Int = 10 @doc(description: "Specifies how deeply results are fetched")
    ): CompanyStructure @doc(description: "Company structure of teams and customers in depth-first order")
    team(id: ID!): CompanyTeam  @doc(description: "Returns company team data by id.")
}

type CompanyLegalAddress @doc(description: "Company legal address output data schema.") {
    street: [String] @doc(description: "An array of strings that defines the Company's street address.")
    city: String @doc(description: "City name.")
    region: CustomerAddressRegion @doc(description: "An object containing region data for the Company.")
    country_code: CountryCodeEnum @doc(description: "Country code.")
    postcode: String @doc(description: "ZIP/postal code.")
    telephone: String @doc(description: "Company's phone number.")
}

type CompanyAdmin @doc(description: "Company Administrator (Customer with corresponding privileges) output data schema.") {
    id: ID! @doc(description: "Company Administrator's id.")
    email: String @doc(description: "Company Administrator email address.")
    firstname: String @doc(description: "Company Administrator first name.")
    lastname: String @doc(description: "Company Administrator last name.")
    job_title: String @doc(description: "Company Administrator job title.")
    gender: Int @doc(description: "Company Administrator gender.")
}

type CompanySalesRepresentative @doc(description: "Company sales representative information output data schema.") {
    email: String @doc(description: "Sales representative email address.")
    firstname: String @doc(description: "Sales representative first name.")
    lastname: String @doc(description: "Sales representative last name.")
}

type CompanyUsers @doc(description: "Output data schema for an object returned by a Company users search query.") {
    items: [Customer] @doc(description: "An array of 'CompanyUser' objects that match the specified search criteria.")
    total_count: Int @doc(description: "The number of objects returned.")
    page_info: SearchResultPageInfo @doc(description: "Pagination meta data.")
}

type CompanyRoles @doc(description: "Output data schema for an object returned by a Company roles search query.") {
    items: [CompanyRole] @doc(description: "A list of company roles that match the specified search criteria.")
    total_count: Int @doc(description: "The total number of objects matching the specified filter.")
    page_info: SearchResultPageInfo @doc(description: "Pagination meta data.")
}

type CompanyRole @doc(description: "Company role output data schema returned in response to a query by Role id.") {
    id: ID! @doc(description: "Role id.")
    name: String @doc(description: "Role name.")
    users_count: Int @doc(description: "Total number of Users with such Role within Company Structure.")
    permissions: [CompanyAclResource] @doc(description: "A list of permission resources defined for a Role.")
}

type CompanyAclResource @doc(description: "Output data schema for an object with Role permission resource information.") {
    id: ID! @doc(description: "ACL resource id.")
    text: String @doc(description: "ACL resource label.")
    sortOrder: Int @doc(description: "ACL resource sort order.")
    children: [CompanyAclResource!] @doc(description: "An array of sub-resources.")
}

type CompanyRoleNameCheckResponse @doc(description: "Response object schema for a role name validation query.") {
    isNameValid: Boolean @doc(description: "Role name validation result")
}

type CompanyUserEmailCheckResponse @doc(description: "Response object schema for a Company User email validation query.") {
    isEmailValid: Boolean @doc(description: "Email validation result")
}

type CompanyAdminEmailCheckResponse @doc(description: "Response object schema for a Company Admin email validation query.") {
    isEmailValid: Boolean @doc(description: "Email validation result")
}

type CompanyEmailCheckResponse @doc(description: "Response object schema for a Company email validation query.") {
    isEmailValid: Boolean @doc(description: "Email validation result")
}

union CompanyStructureEntity = CompanyTeam | Customer

type CompanyStructureItem @doc(description: "Company Team and Customer structure") {
    id: ID! @doc(description: "ID of the item in the hierarchy")
    parent_id: ID @doc(description: "ID of the parent item in the hierarchy")
    entity: CompanyStructureEntity
}

type CompanyStructure {
    items: [CompanyStructureItem]
}

type CompanyTeam @doc(description: "Company Team entity output data schema.") {
    id: ID! @doc(description: "Team id.")
    name: String @doc(description: "Team name.")
    description: String @doc(description: "Team description.")
}

input CompanyUsersFilterInput @doc(description: "Defines the input filters for a Company Users query") {
    status: CompanyUserStatusEnum @doc(description: "Filter Customers by their status within a Company structure. Defaults to 'active'. Required.")
}

enum CompanyUserStatusEnum @doc(description: "List of available Company user statuses.") {
    ACTIVE @doc(description: "Only active users")
    INACTIVE @doc(description: "Only inactive users")
}
```

# Mutations

```graphql
type Mutation {
    createCompany(input: CompanyCreateInput!): CreateCompanyOutput  @doc(description:"Create new Company.")
    updateCompany(input: CompanyUpdateInput!): UpdateCompanyOutput  @doc(description:"Update Company information.")
    createCompanyUser(input: CompanyUserCreateInput!): CreateCompanyUserOutput  @doc(description:"Create new Company User (Customer assigned to Company).")
    updateCompanyUser(input: CompanyUserUpdateInput!): UpdateCompanyUserOutput  @doc(description:"Update Company User information.")
    deleteCompanyUser(id: ID!): DeleteCompanyUserOutput  @doc(description:"Delete Company User by ID.")
    createCompanyRole(input: CompanyRoleCreateInput!): CreateCompanyRoleOutput  @doc(description:"Create new Company role.")
    updateCompanyRole(input: CompanyRoleUpdateInput!): UpdateCompanyRoleOutput  @doc(description:"Update Company role data.")
    deleteCompanyRole(id: ID!): DeleteCompanyRoleOutput  @doc(description:"Delete Company Role by ID.")
    updateCompanyStructure(input: CompanyStructureUpdateInput!): UpdateCompanyStructureOutput  @doc(description:"Update Company Structure element's parent node assignment.")
    createCompanyTeam(input: CompanyTeamCreateInput!): CreateCompanyTeamOutput  @doc(description:"Create Company Team.")
    updateCompanyTeam(input: CompanyTeamUpdateInput!): UpdateCompanyTeamOutput  @doc(description:"Update Company Team data.")
    deleteCompanyTeam(id: ID!): DeleteCompanyTeamOutput  @doc(description:"Delete Company Team entity by ID.")
}

type CreateCompanyTeamOutput @doc(description: "Create company team output data schema.") {
    team: CompanyTeam! @doc(description: "New company team instance.")
}

type UpdateCompanyTeamOutput @doc(description: "Update company team output data schema.") {
    team: CompanyTeam! @doc(description: "Updated company team instance.")
}

type DeleteCompanyTeamOutput @doc(description: "Delete company team output data schema.") {
    success: Boolean! @doc(description: "Indicates whether or not the delete operation succeeded.")
}

type CreateCompanyOutput @doc(description: "Create company output data schema.") {
    company: CompanyOutput! @doc(description: "Create company output instance.")
}

type CompanyOutput {
    id: ID! @doc(description: "Company id.")
    name: String @doc(description: "Company name.")
    email: String @doc(description: "Company email address.")
    legal_name: String @doc(description: "Company legal name.")
    vat_id: String @doc(description: "Company VAT/TAX id.")
    reseller_id: String @doc(description: "Company re-seller id.")
    legal_address: CompanyLegalAddress! @doc(description: "An object containing Company legal address data.")
    company_admin: Customer! @doc(description: "An object containing Company Administrator information.")
}

type UpdateCompanyOutput @doc(description: "Update company output data schema.")  {
    company: Company! @doc(description: "Updated company instance.")
}

type CreateCompanyUserOutput @doc(description: "Create company user output data schema.") {
    user: Customer! @doc(description: "New company user instance.")
}

type UpdateCompanyUserOutput @doc(description: "Update company user output data schema.") {
    user: Customer! @doc(description: "Updated company user instance.")
}

type DeleteCompanyUserOutput @doc(description: "Delete company user output data schema.") {
    success: Boolean! @doc(description: "Indicates whether or not the delete operation succeeded.")
}

type CreateCompanyRoleOutput @doc(description: "Create company role output data schema.") {
    role: CompanyRole! @doc(description: "New company role instance.")
}

type UpdateCompanyRoleOutput @doc(description: "Update company role output data schema.") {
    role: CompanyRole! @doc(description: "Updated company role instance.")
}

type DeleteCompanyRoleOutput @doc(description: "Delete company role output data schema.") {
    success: Boolean! @doc(description: "Indicates whether or not the delete operation succeeded.")
}

type UpdateCompanyStructureOutput @doc(description: "Update company structure output data schema.") {
    company: Company! @doc(description: "Updated company instance.")
}

input CompanyCreateInput @doc(description: "Defines the Company input data schema for creating a new entity."){
    company_name: String! @doc(description: "Company name. Required.")
    company_email: String! @doc(description: "Company email address. Required.")
    legal_name: String @doc(description: "Company legal name.")
    vat_tax_id: String @doc(description: "Company VAT/TAX ID.")
    reseller_id: String @doc(description: "Company re-seller ID.")
    legal_address: CompanyLegalAddressCreateInput! @doc(description: "An object containing Company legal address data. Required.")
    company_admin: CompanyAdminInput! @doc(description: "An object containing Company Administrator information. Required.")
}

input CompanyAdminInput @doc(description: "Defines the Company's Administrator input data schema.") {
    email: String! @doc(description: "Company Administrator's email address. Required.")
    firstname: String! @doc(description: "Company Administrator's first name. Required.")
    lastname: String! @doc(description: "Company Administrator's last name. Required.")
    job_title: String @doc(description: "Company Administrator's custom job title.")
    gender: Int @doc(description: "Company Administrator's gender (Male - 1, Female - 2, Not Specified - 3).")
}

input CompanyLegalAddressCreateInput @doc(description: "Defines the Company legal address input data schema for creating a new entity.") {
    street: [String!]! @doc(description: "An array of strings that define the Company street address. Required array value for a field with strings as values of array.")
    city: String! @doc(description: "Company's city name. Required.")
    country_id: CountryCodeEnum! @doc(description: "Company's country ID. Required. See 'countries' query. Required.")
    region: CustomerAddressRegionInput! @doc(description: "An object containing the region name and/or region ID. Required.")
    postcode: String! @doc(description: "Company's ZIP/postal code. Required.")
    telephone: String! @doc(description: "Company's phone number. Required.")
}

input CompanyUpdateInput @doc(description: "Defines the Company input data schema for updating an existing entity. Allows only needed fields to be passed for update.") {
    company_name: String @doc(description: "Company name.")
    company_email: String @doc(description: "Company email address.")
    legal_name: String @doc(description: "Company legal name.")
    vat_tax_id: String @doc(description: "Company VAT/TAX ID.")
    reseller_id: String @doc(description: "Company re-seller ID.")
    legal_address: CompanyLegalAddressUpdateInput @doc(description: "An object containing Company legal address data.")
}

input CompanyLegalAddressUpdateInput @doc(description: "Defines the Company legal address input data schema for updating an existing entity. Allows only needed fields to be passed for update.") {
    street: [String!] @doc(description: "An array of strings that define the Company street address.")
    city: String @doc(description: "Company's city name.")
    country_id: CountryCodeEnum @doc(description: "Company's country ID. See 'countries' query.")
    region: CustomerAddressRegionInput @doc(description: "An object containing the region name and/or region ID. Required.")
    postcode: String @doc(description: "Company's ZIP/postal code.")
    telephone: String @doc(description: "Company's phone number.")
}

input CompanyUserCreateInput @doc(description: "Defines the input data schema for creating a new Customer - Company user.") {
    job_title: String! @doc(description: "Company user's job title. Required.")
    role_id: ID! @doc(description: "Company user's role ID. Required.")
    firstname: String! @doc(description: "Company user's first name. Required.")
    lastname: String! @doc(description: "Company user's last name. Required.")
    email: String! @doc(description: "Company user's email address. Required.")
    telephone: String! @doc(description: "Company user's phone number. Required.")
    status: Int! @doc(description: "Company user's status ID. Required.")
    target_id: ID @doc(description: "A target structure element ID within a Company's Structure for a user to be assigned to.")
}

input CompanyUserUpdateInput @doc(description: "Defines the input data schema for updating an existing Customer - Company user.") {
    id: ID! @doc(description: "Company user's ID (Customer ID). Required.")
    role_id: ID @doc(description: "Company user's role ID.")
    status: Int @doc(description: "Company user's status ID.")
    job_title: String @doc(description: "Company user's job title.")
    firstname: String @doc(description: "Company user's first name.")
    lastname: String @doc(description: "Company user's last name.")
    email: String @doc(description: "Company user's email address.")
    telephone: String @doc(description: "Company user's phone number.")
}

input CompanyRoleCreateInput @doc(description: "Defines the input data schema for creating a new Company role.") {
    name: String! @doc(description: "Role name. Required.")
    permissions: [String!]! @doc(description: "A list of Role permission resources. Required array value for a field with strings as values of array.")
}

input CompanyRoleUpdateInput @doc(description: "Defines the input data schema for updating an existing Company role.") {
    id: ID! @doc(description: "Role ID. Required.")
    name: String @doc(description: "Role name.")
    permissions: [String!] @doc(description: "A list of Role permission resources. Array value for a field, if provided, should consist only of string values.")
}

input CompanyStructureUpdateInput @doc(description: "Defines the input data schema for updating the Company Structure.") {
    tree_id: ID! @doc(description: "Company Structure element's hierarchical ID that is being moved to another parent. Required.")
    parent_tree_id: ID! @doc(description: "A target parent element tree ID within a Company's Structure. Required.")
}

input CompanyTeamCreateInput @doc(description: "Defines the input data schema for creating a new Company team.") {
    name: String! @doc(description: "Team name. Required.")
    description: String @doc(description: "Team description.")
    target_id: ID @doc(description: "A target structure element ID within a Company's Structure for a team to be assigned to.")
}

input CompanyTeamUpdateInput @doc(description: "Defines the input data schema for updating an existing Company team.") {
    id: ID! @doc(description: "Team ID. Required.")
    name: String @doc(description: "Team name.")
    description: String @doc(description: "Team description.")
}
```

# Existing type modifications

```graphql
type Customer {
    job_title: String @doc(description: "Company User job title.")
    role: CompanyRole @doc(description: "Company User role data (includes permissions).")
    team: CompanyTeam @doc(description: "Company User team data.")
    telephone: String @doc(description: "Company User phone number.")
    status: CompanyUserStatusEnum @doc(description: "Company User status.")
}
```
