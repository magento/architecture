# Overview
There is a need to create a tool for generating magento2 project composer.json file

## Why?
Magento comes with a lot of dependencies out of the box. Often totally no needed from the particular shop requirements. 
For example some small shops come with mysql search, and they do not such dependency as elasticsearch. 
Such tool as suggested in Overview could help with limiting such unneeded dependencies and lowering app costs.

## How it should work?

That tool ought to be cli-app or web-app asking you several questions, and based on answers that tool will return composer.json file only with required dependencies for the application. Example questions might be like:

`1. Do you prefer using mysql or elasticsearch for searching? - possible answers: Mysql, Elasticsearch, None`
And if Elasticsearch will not be choosen, we can remove from our dependencies: magento/module-elasticsearch
`2. What shipping method will you use in your application? - possible multiselect answers: DHL, Fedex, UPS, UsPS, Other/Custom`
According to that answers we can get remove dependencies such as: magento/module-dhl, magento/module-fedex, magento/module-ups, magento/module-usps
`3. Are you going to use "Email to a Friend" functionallity? - possible answers: Yes, No`
If No we can remove magento/module-send-friend

## What should to be done before? 

We need to clean up current dependencies of magento and move them to the module which actually need them. For example `braintree/braintree_php`, `elasticsearch/elasticsearch`.

# What can be achieved by that tool
We will be able to build smaller, more flexible e-commerce shops. Hosting of them might be cheaper, making magento more achievable for small merchants. 