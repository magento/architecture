# Magento Service Isolation Implementation

An iterative approach must be used for service isolation: one service at a time is extracted from a monolithic application.

## PWA & Service isolation

The decision to move all UI to the browser as a part of PWA effort significantly reduces the number of undesired dependencies in the codebase. Most dependencies reside in the UI and will not be present in PWA implementation.
This significantly reduces the scope of required modifications and makes implementations and customizations easier. 

## Platform modifications

Work on the platform is required to allow independent service deployment (module separation, application framework, configurable service invokers, backends for frontends, support for split extensions in marketplace). Additional work will  make independently deployed services (cloud native, development environments) manageable.

## Application framework

The following library components are currently implemented as Magento modules. Many modules depend on these library components, which clutters the module dependency graph. Effectively, they will be required for most application services. These library components (or application-agnostic parts of them) should be moved to `Magento\Framework` to reduce clutter in module dependencies:

* Amqp
* Config
* Cron
* Deploy
* EAV
* GraphQL
* Indexer
* MessageQueue
* MysqlMq
* WebAPI

## Configurable service contract invocations

As of 2.3-develop, Magento service contracts can be called in a process, or through REST, SOAP, and AMQP APIs by remote clients. Remote calls require manual implementation of the client. Magento should be able to invoke service contracts either locally or remotely (if a service whose contract is called is deployed remotely). Two options to implement this are:

* Introduce a service contract invoker that should be used by every client to invoke a service contract. For local calls, this invoker would just call a service contract. For remote services, it would make a remote call. This option gives more control over service contract invocation, but requires modification of all current service contract clients.

* Auto-generated remote call proxies for service contracts that represent remote services. This option does not require code modifications, but it will produce code that is harder to debug.

A remote invoker must contain improved logging, timeouts, retry logic, and circuit breakers to avoid failure state propagation between services.

The first iteration of a remote implementation should be based on existing synchronous APIs.

Long-term goal: To avoid the leaky abstraction of remote invocations, a new generation of service contracts must expose the asynchronous nature of APIs to avoid runtime coupling between services.
  
## Split modules

The described separation of UI from APIs, and admin from storefront requires extracting all UI and Admin application logic to separate modules. For storefront applications, UI code will be moved to separate modules in PWA.

To support real and proxied implementations of modules, the APIs should be extracted to separate modules. Webapi should be extracted to separate modules in order to glue APIs, implementations, and Webapi declarations together.

The following diagram demonstrates the split:

![Split module](split-modules.png)

* *CatalogPWA* - js module deployed on PWA host
* *CatalogStorefront* - deployed on GraphQL endpoint
* *CatalogGraphQL* - exposes GraphQL API, deployed on Storefront GraphQL endpoint
* *CatalogAdmin* - deployed on Admin BFF
* *CatalogWebapi* - deployed on Catalog service instance and depends on CatalogApi and Catalog modules
* *CatalogAPI* - contains module service contracts
* *Catalog* - contains an implementation of Catalog; deployed on Catalog instance
* *CatalogProxy* - contains proxy implementations of Catalog services. Deployed on client services (Checkout, OMS)

### Split database

Today Magento 2 supports ability to deploy Checkout and Order Management databases separately from main database. We should go further and isolate all service databases.

A service must not talk to other service database directly, only through service contracts.

This will require codebase modifications similar to those that were already done for Checkout and Order Management:
- Review application scenarios and minimize data relations in them
- Replace the required inter-service db-level data relations, joins, and transactions with application-level relations, joins, and transactions

Unlike current implementation (OS & Commerce split), the data relation decoupling should be performed in Open Source edition of Magento to simplify extension development.

### Support for split extensions on Marketplace

Marketplace should support an ability to create an extensions as a set of packages.

### Cloud-native

The following changes are proposed:

#### Application configuration

Magento services must support centralized application configuration management.

>NOTE: supported since 2.2 with configuration stored in `env.php`.

#### Tracing
Correlation IDs must be added to internal calls to make tracing easier.

#### Tooling

##### Magento CLI
The current Magento CLI tool depends on the application codebase being present on same machine. This makes it hard to manage a distributed instance of Magento. To solve these issues:

* Introduce an endpoint that can be exposed on an secure network adapter and that would listen to commands from remote magento tool and execute them.

* Create a new standalone magento tool. The tool should have following commands:
    * ```magento instance:add [name] [url]``` - register a new managed remote instance
    * ```magento instance:remove [name]``` - unregister a managed instance from the tool instance list
    * ```magento instance:list``` - list registered remote instances
    * ```magento instance:update``` - load a list of commands supported by the instance
    * ```magento context:set [name]``` - select the default instance to be used in commands

##### Instance builder

With distributed deployment and split modules, `composer.json` management for different services will be complicated. A tool to build instance `composer.json` files for specific services based on enabled features or installed extensions should be built.

Input: 
* Project metadata files describing services to deploy (Catalog, Checkout, etc) and extensions to be installed

Output: 
* A list of `composer.json` files that can be deployed on different instances

The tool should be able to:
* Generate a list of `composer.json` files based on following input:
    * A list of extensions that need to be installed
    * A list of services deployed separately (`composer.json` files to generate)
* Check the compatibility of API versions between modules of different services
* Analyze the distributed instance of Magento:
    * Check API version compatibility
    * Display the list of distributed extensions
     
#### Schema migrations

Expansion-cleanup stages should be introduced into the schema deployment tool to reduce downtime during service deployment.

NOTE: The declarative schema feature in 2.3 makes automated distinction possible.

#### Development environment

To make development of distributed instances easier, a new developer environment must be created. The environment should allow to easily manage multiple services of magento.

Two developer environments are evaluated:
* Environment based on [Minikube](https://kubernetes.io/docs/setup/minikube/) VM with [Kubernetes](https://kubernetes.io/) cluster
* Environment based on [Docker Compose](https://docs.docker.com/compose/)

>NOTE: Distributed service deployment will make the current Commerce/B2B linking approach inconvenient, so a new approach of internal development environment installation must be used: installation of Magento modules from “path” type composer repositories.

### Design documents

* [Service decomposition guidelines](../services-decomposition-guidelines.md)
* [Checkout service](../checkout-service.md)
* [Caching](caching-layer.md)
* [Distributed deployment configuration](../distributed-deployment-configuration.md)
* [Module separation naming](../module-separation-naming.md)
* [Distributed deployment tools](../distributed-deployment-tools.md)
