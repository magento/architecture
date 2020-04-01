# Queries

```graphql
type Query {
    company: Company  @doc(description: "Returns all information about the Company.") 
    companyUsers (
        filter: CompanyUsersFilterInput! @doc(description: "Identifies which Company users to search for and return. Required."),
        pageSize: Int = 20 @doc(description: "Specifies the maximum number of results to return at once. Optional. Defaults to 20."),
        currentPage: Int = 1 @doc(description: "Specifies which page of results to return. The default value is 1."),
    ): CompanyUsers
     @doc(description: "Returns all information about the Company Users.") 
    companyUser(id: Int): CompanyUser  @doc(description: "Returns Company User object for current authenticated Customer or, if ID provided, for specific one.") 
    companyRoles(
        pageSize: Int = 20 @doc(description: "Specifies the maximum number of results to return at once. Optional. Defaults to 20."),
        currentPage: Int = 1 @doc(description: "Specifies which page of results to return. The default value is 1."),
    ): CompanyRoles  @doc(description: "Returns the list of defined roles at Company.") 
    companyRole(id: Int!): CompanyRole  @doc(description: "Returns Company role data by ID.") 
    companyAclResources: [CompanyAclResource]  @doc(description: "Returns the list of all permission resources available for a role.") 
    checkCompanyEmail(email: String!): Boolean  @doc(description: "Returns boolean value of validation whether provided email address is valid for a new Company registration or not.") 
    checkCompanyAdminEmail(email: String!): Boolean  @doc(description: "Returns boolean value of validation whether provided email address is valid for a Company Administrator registration or not.") 
    checkCompanyUserEmail(email: String!): CompanyUserEmailCheckResponse  @doc(description: "Returns an object with result of validation whether provided email address is valid for a new Customer - Company User - registration or not.") 
    checkCompanyRoleName(name: String!): Boolean  @doc(description: "Returns boolean value of validation whether provided Role name is available.") 
    companyHierarchy: CompanyHierarchyOutput  @doc(description: "Returns the complete data about Company structure.") 
    companyTeam(id: Int!): CompanyTeam  @doc(description: "Returns Company Team data by ID.") 
}

type Company @doc(description: "Company entity output data schema.") {
    id: Int @doc(description: "Company ID.")
    company_name: String @doc(description: "Company name.")
    company_email: String @doc(description: "Company email address.")
    legal_name: String @doc(description: "Company legal name.")
    vat_tax_id: String @doc(description: "Company VAT/TAX ID.")
    reseller_id: String @doc(description: "Company re-seller ID.")
    legal_address: CompanyLegalAddress @doc(description: "An object containing Company legal address data.")
    company_admin: CompanyAdmin @doc(description: "An object containing information about Company Administrator.")
    sales_representative: CompanySalesRepresentative @doc(description: "An object containing information about Company representative.")
	payment_methods: [String] @doc(description: "List of payment methods awailable for a Company.")
}

type CompanyLegalAddress @doc(description: "Company legal address output data schema.") {
    street: [String] @doc(description: "An array of strings that defines the Company's street address.")
    city: String @doc(description: "City name.")
    region: CompanyAddressRegion @doc(description: "An object containing region data for the Company.")
    country_id: String @doc(description: "Country ID.")
    country_name: String @doc(description: "Country name.")
    postcode: String @doc(description: "ZIP/postal code.")
    telephone: String @doc(description: "Company's phone number.")
}

type CompanyAdmin @doc(description: "Company Administrator (Customer with corresponding privileges) output data schema.") {
    id: Int @doc(description: "Company Administrator's ID.")
    email: String @doc(description: "Company Administrator email address.")
    firstname: String @doc(description: "Company Administrator first name.")
    lastname: String @doc(description: "Company Administrator last name.")
    job_title: String @doc(description: "Company Administrator job title.")
    gender: Int @doc(description: "Company Administrator gender.")
}

type CompanyAddressRegion @doc(description: "Output data schema for a region data in Company address.") {
    region_id: Int @doc(description: "Unique region identifier. Required for certain countries.")
    region: String @doc(description: "State or province name.")
    region_code: String @doc(description: "Region code.")
}

type CompanySalesRepresentative @doc(description: "Company sales representative information output data schema.") {
    email: String @doc(description: "Sales representative email address.")
    firstname: String @doc(description: "Sales representative first name.")
    lastname: String @doc(description: "Sales representative last name.")
}

type CompanyUsers @doc(description: "Output data schema for an object returned by a Company users search query.") {
    items: [CompanyUser] @doc(description: "An array of 'CompanyUser' objects that match the specified search criteria.")
    total_count: Int @doc(description: "The number of objects returned.")
    page_info: SearchResultPageInfo @doc(description: "An object that includes the pagination meta data of the search query.")
}

type CompanyUser @doc(description: "Company User (Customer assigned to Company) entity output data schema.") {
    id: Int @doc(description: "Company User/Customer ID.")
    email: String @doc(description: "Company User email address.")
    firstname: String @doc(description: "Company User first name.")
    lastname: String @doc(description: "Company User last name.")
    status: Int @doc(description: "Company User status ID.")
    status_name: String @doc(description: "Company User status name.")
    job_title: String @doc(description: "Company User job title.")
    telephone: String @doc(description: "Company User phone number.")
    role: CompanyRole @doc(description: "Company User role data (includes permissions).")
    team: CompanyTeam @doc(description: "Company User team data.")
}

type CompanyRoles @doc(description: "Output data schema for an object returned by a Company roles search query.") {
    items: [CompanyRole] @doc(description: "An array of 'CompanyUser' objects that match the specified search criteria.")
    total_count: Int @doc(description: "The number of objects returned.")
    page_info: SearchResultPageInfo @doc(description: "An object that includes the pagination meta data of the search query.")
}

type CompanyRole @doc(description: "Company role output data schema returned in response to a query by Role ID.") {
    id: Int @doc(description: "Role ID.")
    name: String @doc(description: "Role name.")
    users_count: Int @doc(description: "Total number of Users with such Role within Company Hierarchy.")
    permissions: [String] @doc(description: "A list of permission resources defined for a Role.")
}

type CompanyAclResource @doc(description: "Output data schema for an object with Role permission resource information.") {
    id: String @doc(description: "ACL resource ID.")
    text: String @doc(description: "ACL resource label.")
    sortOrder: Int @doc(description: "ACL resource sort order.")
    children: [CompanyAclResource!] @doc(description: "An array of sub-resources.")
}

type CompanyUserEmailCheckResponse @doc(description: "Response object schema for a Company User email validation query.") {
    status: Int @doc(description: "1 - 'OK', 0 - 'Error'")
    message: String @doc(description: "Error or notice message for a User.")
}

type CompanyHierarchyOutput @doc(description: "Response object schema for a Company Hierarchy query.") {
    structure: CompanyHierarchyElement @doc(description: "An array of Company structure elements.")
    draggable: Boolean @doc(description: "Flag that defines whether Company Hierarchy can be changed by current User or not.")
    max_nesting: Int @doc(description: "Indicator of maximun nesting of elements within a whole Company Hierarchy.")
}

type CompanyHierarchyElement @doc(description: "Company Hierarchy element output data schema.") {
    id: Int @doc(description: "Hierarchy element ID.")
    tree_id: Int @doc(description: "The hierarchical ID of the element within a structure. Used for changing element's position in hierarchy.")
    type: String @doc(description: "Hierarchy element type: 'customer' or a 'team'.")
    text: String @doc(description: "Hierarchy element name.")
    description: String @doc(description: "Hierarchy element description.")
    opened: Boolean @doc(description: "Flag that defines whether Hierarchy element should be expanded or not.")
    children: [CompanyHierarchyElement!] @doc(description: "An array of child elements.")
}

type CompanyTeam @doc(description: "Company Team entity output data schema.") {
    id: Int @doc(description: "Team ID.")
    name: String @doc(description: "Team name.")
    description: String @doc(description: "Team description.")
}

type SearchResultPageInfo @doc(description: "Output data schema for a meta data object of the search query response.") {
    page_size: Int @doc(description: "Maximum number of items returned by a search query.")
    current_page: Int @doc(description: "Current result page number.")
    total_pages: Int @doc(description: "Total number of pages.")
}

input CompanyUsersFilterInput @doc(description: "Defines the input filters for a Company Users query") {
    status: CompanyUserStatusFilterValuesEnum! @doc(description: "Filter Customers by their status within a Company structure. Defaults to 'active'. Required.")
}

enum CompanyUserStatusFilterValuesEnum @doc(description: "List of available Company user statuses.") {
    active @doc(description: "Only active users")
    inactive @doc(description: "Only inactive users")
    all @doc(description: "All users")
}
```

# Mutations

```graphql
type Mutation {
    createCompany(input: CompanyCreateInput!): Company  @doc(description:"Create new Company.")
    updateCompany(input: CompanyUpdateInput!): Company  @doc(description:"Update Company information.")
    createCompanyUser(input: CompanyUserCreateInput!): CompanyUser  @doc(description:"Create new Company User (Customer assigned to Company).")
    updateCompanyUser(input: CompanyUserUpdateInput!): CompanyUser  @doc(description:"Update Company User information.")
    deleteCompanyUser(id: Int!): Boolean  @doc(description:"Delete Company User by ID.")
    createCompanyRole(input: CompanyRoleCreateInput!): CompanyRole  @doc(description:"Create new Company role.")
    updateCompanyRole(input: CompanyRoleUpdateInput!): CompanyRole  @doc(description:"Update Company role data.")
    deleteCompanyRole(id: Int!): Boolean  @doc(description:"Delete Company Role by ID.")
    updateCompanyHierarchy(input: CompanyHierarchyUpdateInput!): Boolean  @doc(description:"Update Company Hierarchy element's parent node assignment.")
    createCompanyTeam(input: CompanyTeamCreateInput!): CompanyTeam  @doc(description:"Create Company Team.")
    updateCompanyTeam(input: CompanyTeamUpdateInput!): CompanyTeam  @doc(description:"Update Company Team data.")
    deleteCompanyTeam(id: Int!): Boolean  @doc(description:"Delete Company Team entity by ID.")
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
    region: CompanyAddressRegionInput! @doc(description: "An object containing the region name and/or region ID. Required.")
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
    region: CompanyAddressRegionInput @doc(description: "An object containing the region name and/or region ID. Required.")
    postcode: String @doc(description: "Company's ZIP/postal code.")
    telephone: String @doc(description: "Company's phone number.")
}

input CompanyAddressRegionInput @doc(description: "Defines the Company's region input data schema.") {
    region_id: Int @doc(description: "Unique region identifier. Required for certain countries. See 'country(id: ID)' query.")
    region: String @doc(description: "State or province name.")
}

input CompanyUserCreateInput @doc(description: "Defines the input data schema for creating a new Customer - Company user.") {
    job_title: String! @doc(description: "Company user's job title. Required.")
    role_id: Int! @doc(description: "Company user's role ID. Required.")
    firstname: String! @doc(description: "Company user's first name. Required.")
    lastname: String! @doc(description: "Company user's last name. Required.")
    email: String! @doc(description: "Company user's email address. Required.")
    telephone: String! @doc(description: "Company user's phone number. Required.")
    status: Int! @doc(description: "Company user's status ID. Required.")
    target_id: Int @doc(description: "A target structure element ID within a Company's Hierarchy for a user to be assigned to.")
}

input CompanyUserUpdateInput @doc(description: "Defines the input data schema for updating an existing Customer - Company user.") {
    id: Int! @doc(description: "Company user's ID (Customer ID). Required.")
    role_id: Int @doc(description: "Company user's role ID.")
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
    id: Int! @doc(description: "Role ID. Required.")
    name: String @doc(description: "Role name.")
    permissions: [String!] @doc(description: "A list of Role permission resources. Array value for a field, if provided, should consist only of string values.")
}

input CompanyHierarchyUpdateInput @doc(description: "Defines the input data schema for updating the Company Hierarchy.") {
    tree_id: Int! @doc(description: "Company Hierarchy element's hierarchical ID that is being moved to another parent. Required.")
    parent_tree_id: Int! @doc(description: "A target parent element tree ID within a Company's Hierarchy. Required.")
}

input CompanyTeamCreateInput @doc(description: "Defines the input data schema for creating a new Company team.") {
    name: String! @doc(description: "Team name. Required.")
    description: String @doc(description: "Team description.")
    target_id: Int @doc(description: "A target structure element ID within a Company's Hierarchy for a team to be assigned to.")
}

input CompanyTeamUpdateInput @doc(description: "Defines the input data schema for updating an existing Company team.") {
    id: Int! @doc(description: "Team ID. Required.")
    name: String @doc(description: "Team name.")
    description: String @doc(description: "Team description.")
}

```

# Directory graphql changes

```graphql

###### Begin: New types ######
enum CountryCodeEnum @doc(description: "The list of countries codes.") {
    AF @doc(description: "Afghanistan")
    AX @doc(description: "Åland Islands")
    AL @doc(description: "Albania")
    DZ @doc(description: "Algeria")
    AS @doc(description: "American Samoa")
    AD @doc(description: "Andorra")
    AO @doc(description: "Angola")
    AI @doc(description: "Anguilla")
    AQ @doc(description: "Antarctica")
    AG @doc(description: "Antigua & Barbuda")
    AR @doc(description: "Argentina")
    AM @doc(description: "Armenia")
    AW @doc(description: "Aruba")
    AU @doc(description: "Australia")
    AT @doc(description: "Austria")
    AZ @doc(description: "Azerbaijan")
    BS @doc(description: "Bahamas")
    BH @doc(description: "Bahrain")
    BD @doc(description: "Bangladesh")
    BB @doc(description: "Barbados")
    BY @doc(description: "Belarus")
    BE @doc(description: "Belgium")
    BZ @doc(description: "Belize")
    BJ @doc(description: "Benin")
    BM @doc(description: "Bermuda")
    BT @doc(description: "Bhutan")
    BO @doc(description: "Bolivia")
    BA @doc(description: "Bosnia & Herzegovina")
    BW @doc(description: "Botswana")
    BV @doc(description: "Bouvet Island")
    BR @doc(description: "Brazil")
    IO @doc(description: "British Indian Ocean Territory")
    VG @doc(description: "British Virgin Islands")
    BN @doc(description: "Brunei")
    BG @doc(description: "Bulgaria")
    BF @doc(description: "Burkina Faso")
    BI @doc(description: "Burundi")
    KH @doc(description: "Cambodia")
    CM @doc(description: "Cameroon")
    CA @doc(description: "Canada")
    CV @doc(description: "Cape Verde")
    KY @doc(description: "Cayman Islands")
    CF @doc(description: "Central African Republic")
    TD @doc(description: "Chad")
    CL @doc(description: "Chile")
    CN @doc(description: "China")
    CX @doc(description: "Christmas Island")
    CC @doc(description: "Cocos (Keeling) Islands")
    CO @doc(description: "Colombia")
    KM @doc(description: "Comoros")
    CG @doc(description: "Congo-Brazzaville")
    CD @doc(description: "Congo-Kinshasa")
    CK @doc(description: "Cook Islands")
    CR @doc(description: "Costa Rica")
    CI @doc(description: "Côte d’Ivoire")
    HR @doc(description: "Croatia")
    CU @doc(description: "Cuba")
    CY @doc(description: "Cyprus")
    CZ @doc(description: "Czech Republic")
    DK @doc(description: "Denmark")
    DJ @doc(description: "Djibouti")
    DM @doc(description: "Dominica")
    DO @doc(description: "Dominican Republic")
    EC @doc(description: "Ecuador")
    EG @doc(description: "Egypt")
    SV @doc(description: "El Salvador")
    GQ @doc(description: "Equatorial Guinea")
    ER @doc(description: "Eritrea")
    EE @doc(description: "Estonia")
    ET @doc(description: "Ethiopia")
    FK @doc(description: "Falkland Islands")
    FO @doc(description: "Faroe Islands")
    FJ @doc(description: "Fiji")
    FI @doc(description: "Finland")
    FR @doc(description: "France")
    GF @doc(description: "French Guiana")
    PF @doc(description: "French Polynesia")
    TF @doc(description: "French Southern Territories")
    GA @doc(description: "Gabon")
    GM @doc(description: "Gambia")
    GE @doc(description: "Georgia")
    DE @doc(description: "Germany")
    GH @doc(description: "Ghana")
    GI @doc(description: "Gibraltar")
    GR @doc(description: "Greece")
    GL @doc(description: "Greenland")
    GD @doc(description: "Grenada")
    GP @doc(description: "Guadeloupe")
    GU @doc(description: "Guam")
    GT @doc(description: "Guatemala")
    GG @doc(description: "Guernsey")
    GN @doc(description: "Guinea")
    GW @doc(description: "Guinea-Bissau")
    GY @doc(description: "Guyana")
    HT @doc(description: "Haiti")
    HM @doc(description: "Heard &amp; McDonald Islands")
    HN @doc(description: "Honduras")
    HK @doc(description: "Hong Kong SAR China")
    HU @doc(description: "Hungary")
    IS @doc(description: "Iceland")
    IN @doc(description: "India")
    ID @doc(description: "Indonesia")
    IR @doc(description: "Iran")
    IQ @doc(description: "Iraq")
    IE @doc(description: "Ireland")
    IM @doc(description: "Isle of Man")
    IL @doc(description: "Israel")
    IT @doc(description: "Italy")
    JM @doc(description: "Jamaica")
    JP @doc(description: "Japan")
    JE @doc(description: "Jersey")
    JO @doc(description: "Jordan")
    KZ @doc(description: "Kazakhstan")
    KE @doc(description: "Kenya")
    KI @doc(description: "Kiribati")
    KW @doc(description: "Kuwait")
    KG @doc(description: "Kyrgyzstan")
    LA @doc(description: "Laos")
    LV @doc(description: "Latvia")
    LB @doc(description: "Lebanon")
    LS @doc(description: "Lesotho")
    LR @doc(description: "Liberia")
    LY @doc(description: "Libya")
    LI @doc(description: "Liechtenstein")
    LT @doc(description: "Lithuania")
    LU @doc(description: "Luxembourg")
    MO @doc(description: "Macau SAR China")
    MK @doc(description: "Macedonia")
    MG @doc(description: "Madagascar")
    MW @doc(description: "Malawi")
    MY @doc(description: "Malaysia")
    MV @doc(description: "Maldives")
    ML @doc(description: "Mali")
    MT @doc(description: "Malta")
    MH @doc(description: "Marshall Islands")
    MQ @doc(description: "Martinique")
    MR @doc(description: "Mauritania")
    MU @doc(description: "Mauritius")
    YT @doc(description: "Mayotte")
    MX @doc(description: "Mexico")
    FM @doc(description: "Micronesia")
    MD @doc(description: "Moldova")
    MC @doc(description: "Monaco")
    MN @doc(description: "Mongolia")
    ME @doc(description: "Montenegro")
    MS @doc(description: "Montserrat")
    MA @doc(description: "Morocco")
    MZ @doc(description: "Mozambique")
    MM @doc(description: "Myanmar (Burma)")
    NA @doc(description: "Namibia")
    NR @doc(description: "Nauru")
    NP @doc(description: "Nepal")
    NL @doc(description: "Netherlands")
    AN @doc(description: "Netherlands Antilles")
    NC @doc(description: "New Caledonia")
    NZ @doc(description: "New Zealand")
    NI @doc(description: "Nicaragua")
    NE @doc(description: "Niger")
    NG @doc(description: "Nigeria")
    NU @doc(description: "Niue")
    NF @doc(description: "Norfolk Island")
    MP @doc(description: "Northern Mariana Islands")
    KP @doc(description: "North Korea")
    NO @doc(description: "Norway")
    OM @doc(description: "Oman")
    PK @doc(description: "Pakistan")
    PW @doc(description: "Palau")
    PS @doc(description: "Palestinian Territories")
    PA @doc(description: "Panama")
    PG @doc(description: "Papua New Guinea")
    PY @doc(description: "Paraguay")
    PE @doc(description: "Peru")
    PH @doc(description: "Philippines")
    PN @doc(description: "Pitcairn Islands")
    PL @doc(description: "Poland")
    PT @doc(description: "Portugal")
    QA @doc(description: "Qatar")
    RE @doc(description: "Réunion")
    RO @doc(description: "Romania")
    RU @doc(description: "Russia")
    RW @doc(description: "Rwanda")
    WS @doc(description: "Samoa")
    SM @doc(description: "San Marino")
    ST @doc(description: "São Tomé & Príncipe")
    SA @doc(description: "Saudi Arabia")
    SN @doc(description: "Senegal")
    RS @doc(description: "Serbia")
    SC @doc(description: "Seychelles")
    SL @doc(description: "Sierra Leone")
    SG @doc(description: "Singapore")
    SK @doc(description: "Slovakia")
    SI @doc(description: "Slovenia")
    SB @doc(description: "Solomon Islands")
    SO @doc(description: "Somalia")
    ZA @doc(description: "South Africa")
    GS @doc(description: "South Georgia & South Sandwich Islands")
    KR @doc(description: "South Korea")
    ES @doc(description: "Spain")
    LK @doc(description: "Sri Lanka")
    BL @doc(description: "St. Barthélemy")
    SH @doc(description: "St. Helena")
    KN @doc(description: "St. Kitts & Nevis")
    LC @doc(description: "St. Lucia")
    MF @doc(description: "St. Martin")
    PM @doc(description: "St. Pierre & Miquelon")
    VC @doc(description: "St. Vincent & Grenadines")
    SD @doc(description: "Sudan")
    SR @doc(description: "Suriname")
    SJ @doc(description: "Svalbard & Jan Mayen")
    SZ @doc(description: "Swaziland")
    SE @doc(description: "Sweden")
    CH @doc(description: "Switzerland")
    SY @doc(description: "Syria")
    TW @doc(description: "Taiwan")
    TJ @doc(description: "Tajikistan")
    TZ @doc(description: "Tanzania")
    TH @doc(description: "Thailand")
    TL @doc(description: "Timor-Leste")
    TG @doc(description: "Togo")
    TK @doc(description: "Tokelau")
    TO @doc(description: "Tonga")
    TT @doc(description: "Trinidad & Tobago")
    TN @doc(description: "Tunisia")
    TR @doc(description: "Turkey")
    TM @doc(description: "Turkmenistan")
    TC @doc(description: "Turks & Caicos Islands")
    TV @doc(description: "Tuvalu")
    UG @doc(description: "Uganda")
    UA @doc(description: "Ukraine")
    AE @doc(description: "United Arab Emirates")
    GB @doc(description: "United Kingdom")
    US @doc(description: "United States")
    UY @doc(description: "Uruguay")
    UM @doc(description: "U.S. Outlying Islands")
    VI @doc(description: "U.S. Virgin Islands")
    UZ @doc(description: "Uzbekistan")
    VU @doc(description: "Vanuatu")
    VA @doc(description: "Vatican City")
    VE @doc(description: "Venezuela")
    VN @doc(description: "Vietnam")
    WF @doc(description: "Wallis & Futuna")
    EH @doc(description: "Western Sahara")
    YE @doc(description: "Yemen")
    ZM @doc(description: "Zambia")
    ZW @doc(description: "Zimbabwe")
}
###### End: New types ######
```