# JSON Object Versioning Rules

* Status: proposed
* Deciders: Leon Kiefer, Per Pascal Seeland
* Date: 2020-05-14

Technical Story: https://github.com/VirtualProgrammingLab/documentation/issues/1

## Context and Problem Statement

The ViPLab Projects defines a set of JSON-objects, which are used in the different components of the project. Since ViPLab itself does not 
store any data, these objects have to be stored by third party systems. These will be, depending on the context they are used in either LMS (like ILIAS)
or data repositories (like dataverse). There needs to be a way to identify the version of the different objects stored by the third party system to 
allow e.g. upgrading/migrating the different components. 

## Decision Drivers

* Provide a way to identify the version of the objects and also provide a path for e.g. upgrading the API endpoints used by those systems.

## Considered Options

* Version all endpoints, but not the objects
* Version all objects, but not the endpoints
* Version parts of the objects and all of the endpoints.

## Decision Outcome


### Positive Consequences

### Negative Consequences

## Pros and Cons of the Options

## Links
