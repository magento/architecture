## Configuration Provisioning

### Overview

When you start a new Magento project there are similar initial tasks to set up configuration including websites, stores, tax, store configuration, attributes and more. This leads to the following known problems:

* Boilerplate Code in data patches (copied from Google, DevDocs, StackOverflow, other projects, maybe poorly adjusted)
* Update and Delete operations are tedious and complex as you have to manage relations and identification inside data patches (i.e. store code vs. autoincrement id)
* Installation from scratch with scope level configuration inside `config.php` is not possible if you rely on it in data patches

There should be a declarative schema approach to provision a new instance with all necessary configuration to run. It is related to the existing delcarative database schema but manages configuration data like:

* Websites
* Stores
* Store Groups
* System configuration
* Attribtues
* Attribute Sets
* Attribute Groups
* Tax

This is at least the most basic configuration. The design approach aims to leverage the extensibility so that more configuration entities can be included in the list above.

**Additional feature:** There should also be an export functionality which generates declarative schema files for an existing Magento instance which includes all configuration. This feature would be priority 2 as it is more important to import them first via declarative schema.

### Design

#### Module structure

The main dataflow and import functionality is provided by a core module. It controls the workflow and provides generic functionality for all configuration modules. On top of that there is a module for every configuration entity (website, store, tax...) which uses the core module to import and export the data.

![module-structure](module-structure.jpg)

The core module provides:

* CLI Commands: Imports and exports single configuration entities
* Backend GUI: Offers functionality to export configuration via admin interface
* Import / Export workflow: Abstract definition of the workflow
* Serialize / Input / Output: Read- and write adapter, offers merge logic for declarative files and offers `maintained` flag logic

#### Provisioning sequence

The usual behavior for installing from scratch or upgrading is the following:

`bin/magento setup:install`

* Declarative Schema (DB)
* Install Schema
* Upgrade Schema
* Recurring Schema
* Install Data
* Upgrade Data
* Recurring Data
* app:config:import

`bin/magento setup:upgrade`

* Declarative Schema (DB)
* Install Schema
* Upgrade Schema
* Recurring Schema
* Install Data
* Upgrade Data
* Recurring Data
* app:config:import

The configuration provisioning core logic should be executed as one of the last recurring data scripts. As alternative the execution sequence mentioned above can be changed so that it runs directly after recurring data and before config import.

The reason for this is that also registered themes (inside a recurring data script) can still be used by the provisioning logic. This would be obsolete if theme registration is part of configuration provisioning.

#### Data structure

As already mentioned the data structure is related to the existing database declarative schema. Every module can define such a XML file for a configuration type. All configuration files for the same type would be merged to one big logical structure. Therefore every configuration type module should at least support merge logic. This is done individually as for example websites are merged differently than attributes.

Therefore a virtual type of `Magento\Framework\Config\Reader\Filesystem` will be defined which has a ` Magento\Framework\Config\ConverterInterface` which is responsible for merging the XML files.

On top of that every configuration type module defines a `maintained` flag which basically defines if the configuration can be overriden by a user or not. If it is set to `yes` the configuration provisioning will always override the values, independently if it was changed or not.

#### Change or delete configuration

Changing or deleting configuration is a challenging task, especially for all the relations and true identifier of an entity (i.e. store code vs. autoincrement id). Every configuration type module should therefore define the identifier on its own, dependent on which configuration type it supports.

Changes can then be done via the identifier and the `maintained` flag.

Deletions must be done explicit similar how you explicitly deactivate a plugin with `disabled = true`. This allows for proper merging of the XML files.

#### Extension Points and Scenarios

Once the core provisioning logic and module is set up including the main workflow, we can adapt legacy code to the new functionality. For example registering themes is done via a recurring patch which we can easily adapt to the new configuration provisioning logic.

### Prototype or Proof of Concept

tbd.
