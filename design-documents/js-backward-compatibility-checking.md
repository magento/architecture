# [HLD]: Javascript backward compatibility check for AMD modules marked with @api

## Overview

The goal is to create an automated script that checks for backward incompatibility in the same manner for PHP Classes and Interfaces.

### Javascript in Magento core

* The implementation of Javascript in Magento core codebases does not utilize interfaces nor classes, instead we have AMD modules via require.js.
* Javascript does not have Classes nor interfaces. There are classes in ES6 but they are still just syntactic sugar on top of prototypal inheritance. There are no interfaces because JS does not have strict typing.
* The only place that we have interface and class interface is in the Pagebuilder module in which uses Typescript (A superset of Javascript) that adds Interface and Class features (but not truly enforced, only with parser).

#### So what can we check

* Add/remove classes (modules)
* Add/remove methods
* Add/remove arguments
* (Add/remove events and event properties may be difficult as some events are bound in phtml files)

### Design

JS tools to be used:

* Node: Runtime for javascript.
* Acorn: Abstract syntax tree analyzer (just means we can statically analyze javascript code)
* Acorn-walk: Allows traversal of the AST
* Semantic version checker standalone module.
* Symfony Process module: to execute javascript SVC in scanner class - allows asynchronous processing of the JS SVC

Acorn will handle the static analyzing of the file for add/removal of individual methods and arguments via acorn

SVC module will handle the I/O for the report generation by calling the node script and reading the output from it.

SVC PHP needs a new Finder class for JS since SemVerChecker ignores everything not .php in BasicFinder implementation. Will need a new scanner class in SVC PHP for JS and give the results of scan to new JS analyzer class then add the results from the JS analyzer to the reports.

#### Acceptance Criteria Fulfillment

1. Statically analyze JS files.
2. Append JS statically analysis results to current SVC result report.

#### Implementation questions

1. Should we only check for @api to determine if a file needs to be analyzed or whitelisting files to be checked?
2. What should the SVC expect as the input to the node script and what should the expected output be from the node script?
