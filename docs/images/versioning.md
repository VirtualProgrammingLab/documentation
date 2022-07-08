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
* Version all objects and all of the endpoints.

## Decision Outcome

It has been decided to version all long living objects, and all endpoints. The reason for requiring all objects to have version tags is for consistency
reasons. Endpoints will only accept objects versions of one major version. Breaking changes will require to define a new version. Incremental 
updates, which only extend the object are allowed. All systems are expected to handle these incremental updates gracefully. This means that a
version 3.1.x message returned by the api must be accept by a 3.0.x client. The additional information contained by the api may be discarded. The same
rules apply when a 3.1.x client talks to a 3.0.x api. The api will process the message according to the 3.0.x rules and extra information might be discarded.

The client has to determine the correct endpoint versions to use. When the client will use a ComputationTemplate with version 3.x.x the corresponding
websocket api will be ws://example.com/v3/computation. All other messages used in any further communication between the websocket-api and the client
are also expected to be version 3.x.x. There is no possiblity to mix different versions. If a client wants to use a v4 api, he has to reconnect to the correct
endpoint at ws://example.com/v4/computation

The objects have been devided into three groups:
- Long living object: ComputationTemplate, ComputationTask and ComputationResult
- Wrappers: Computation
- Message Types: CreateComputation and Result

All long living objects are expected to be stored by clients and have version attribute on their top level of the json object, making it easy to get the information. Wrappers only exist during the communication between the client and the api and thus will not contain any version information, as it can be derived from the version of the api version they are being used with. Their only functionality is to group long living objects during a request. Messages Types are the messages used for the communication of the client with the websocket api. Like the wrappers they belong to a specific endpoint version and are also not versioned.


### Positive Consequences

Providing version information allows api and the client to be updated independently. It also allows the objects being held by the client to be easily identified without the need to either store this information somewhere external, where is might get lost or require some method to identify it based on the attributes available.

### Negative Consequences

The version needs to stored on all objects, which requires additional space and bandwidth. Also this information needs to be processed in the endpoints. But this in neglactable considered to the need of upgrading different systems of different third party users at once.

## Pros and Cons of the Options

Versioning either only the endpoints would still cause the issue, that a client needs to keep track of which implicit version an object has, to make sure he picks
the right endpoint. This might also lead to the fact, that one might try to send the data to the wrong endpoint, since it is not known where it should belong.
Only versioning the endpoints would require to add a lot of version identification cold on the api side, which should be avoided.

Between the last to options, the only difference is, that the actions wrappers are not being versioned. This tradeoff has been made due to the fact, that they should never be stored somewhere and that they are considered stable for one api version. Adding actions or changing the wrappers itself will cause a upgrade of the api version.

## Links
