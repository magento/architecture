# Mutations

```graphql
type Mutation {
    requestPasswordResetEmail(email: String!): Boolean @doc(description: "Request an email with reset password token for the registered customer identified by the provided email")
    resetPassword(email: String!, resetPasswordToken: String!, newPassword: String!): Boolean @doc(description: "Reset customer password using reset password token received in the email after requesting it using requestPasswordResetEmail")
}
```

The client app will initiate password reset using `requestPasswordResetEmail` mutation. The email sent to a customer will include a link with reset password token. When the customer follows this link, he will be taken to the "Password reset" page served by the client app. This page will have to extract the reset token from the URL, request customer to enter his email and a new password. Then customer password reset will be completed using `resetPassword` mutation.

# Configuration

The settings defined in the following config files must be honored by the password reset mutations:

- `Magento/Security/etc/adminhtml/system.xml`
- `Magento/Customer/etc/adminhtml/system.xml`


The following config settings need to be whitelisted in the `storeConfig` query. See [example](https://github.com/magento/magento2/blob/52b66acf17e049dc2c5c7d9e12bd6d29d6a1a16d/app/code/Magento/CatalogGraphQl/etc/graphql/di.xml#L96).

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <tab id="customer" translate="label" sortOrder="300">
            <label>Customers</label>
        </tab>
        <section id="customer" translate="label" sortOrder="130" showInDefault="1" showInWebsite="1" showInStore="1">
            <class>separator-top</class>
            <label>Customer Configuration</label>
            <tab>customer</tab>
            <resource>Magento_Customer::config_customer</resource>
            <group id="password" translate="label" type="text" sortOrder="30" showInDefault="1" showInWebsite="1" showInStore="1">
                <field id="required_character_classes_number" translate="label comment" type="text" sortOrder="70" showInDefault="1" canRestore="1">
                    <label>Number of Required Character Classes</label>
                    <comment>Number of different character classes required in password: Lowercase, Uppercase, Digits, Special Characters.</comment>
                    <validate>required-entry validate-digits validate-digits-range digits-range-1-4</validate>
                </field>
                <field id="minimum_password_length" translate="label comment" type="text" sortOrder="80" showInDefault="1" canRestore="1">
                    <label>Minimum Password Length</label>
                    <comment>Please enter a number 1 or greater in this field.</comment>
                    <validate>required-entry validate-digits validate-digits-range digits-range-1-</validate>
                </field>
                <field id="autocomplete_on_storefront" type="select" translate="label" sortOrder="65" showInDefault="1" showInWebsite="1" canRestore="1">
                    <label>Enable Autocomplete on login/forgot password forms</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
            </group>
        </section>
    </system>
</config>
```

