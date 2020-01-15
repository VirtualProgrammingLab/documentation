
ViPLab (formerly NumLab)
========================
Virtual programming laboratory for education and teaching at universities. Thick/Rich clients (frontends) use the ECS middleware to communicate to several backends (computation clients) working in a cluster. Please look the requirements shown at [[../core|Core ECS]].

A _computation client_ instance supports one or more _computation chain_s (e.g. 'C computation chain', 'Matlab computation chain').


Descriptions are done from the ECS point of view. The term "server" is used as a synonym for the ECS. All other participants are known as "clients", e.g. "computation client", "student client".

Terms (ViPLab specific)
-----
* **ViPLab** virtual programming laboratory (for programming this term is more generic than ''!NumLab'' (outdated))
* **NumLab** legacy term for ViPLab (don't use it for new documentation)
* **ViP** 
    * ViPLab shortcut
    * "Virtuelles Programmierlabor" (German project shortcut)
* **CC** computation client
* **SC** student client
* **TC** teacher client
* **client** a system component requesting ECS, one of CC, SC, TC
* **computation** a CC computes a ''Result'' of a ''Solution'' to an ''Exercise'' (''Solution'' and - referenced - ''Exercise'' being input, ''Result'' output of computation).
* **computation chain**
    * computation chain specified in an ''Exercise'', e.g. 'C computation chain', 'Matlab computation chain',
    * chain of commands being performed during a computation.

ECS ressources
--------------

[/numlab/exercises](/excercises) <BR>
[/numlab/solutions](/solutions) <BR>
[/numlab/results](/results) <BR>
[/numlab/interrupts](/interrupts)


### Overview Resource Properties

#### General Properties
Property|[/numlab/exercises](/excerises)|[/numlab/solutions](/solutions)|[/numlab/results](/results)
--------|-------------------------------|-------------------------------|---------------------------
POST    | POST from teacher client creates ''new'' resource - with automatically created index. | POST from student client ''pushes'' Solution into public queue.<<BR>> POST from computation client ''pulls'' Solution from public queue. | POST from computation client ''pushes'' Result into private queue of student client.<<BR>> POST from student client ''pulls'' Result from its private queue.
GET     | GET: returns URL list of available Exercises; thereafter GETs may be used for accessing indexed Excercises (e.g. .../exercises/4711). | GET: not used (use POST for pulling/pushing from/into queue). | GET from queue: not used (use POST for pulling/pushing from/into queue).
DELETE  |DELETE of indexed Exercise ''only'': gets/deletes Exercise.<<BR>> This works only ''once''. | DELETE: not used (use POST for pulling/pushing from/into queue). | DELETE: not used (use POST for pulling/pushing from/into queue). 
access | POST/DELETE: <<BR>>- all teacher clients;<<BR>> GET: <<BR>>- all teacher clients, <<BR>>- all student clients, <<BR>>- all computation clients.| POST: <<BR>>- student clients for pushing Solutions into queue;<<BR>>- computation clients for pulling Solutions from queue.<<BR>> GET/DELETE: unused.| POST: <<BR>>- computation clients for pushing Results into queue;<<BR>>- student clients for pulling Results from queue (each SC has its ''own'' private queue for pulling Results).<<BR>> GET/DELETE: <<BR>>not used. 

Notes:

- Pulling from a queue returns an element or the information, that there is none.
- Pushing into a queue should always succeed.

#### Properties of Queue Resources
Property| [/numlab/solutions](/solutions)|[/numlab/results](/results)
-|-|-
sender/receiver relation |m-to-n (logical, ECS POV: m-to-1):<<BR>> multiple student clients -> multiple computation clients| m-to-n (logical, ECS POV: 1-to-n):<<BR>> multiple computation clients -> multiple student clients
single queue or multiple ones? | single queue for all  | multiple queues: one for ''each'' student client
 access push/pull | shared push (SCs), shared pull (CCs) | shared push (CCs), single pull (SC) 

Notes:

 * 'multiple queues' is meant in a behavioral sense: how this is realized technically is out of scope here<<FootNote(Routing computation Result back from CC to some SC being sender of a Solution is realized via a mechanism using special HTTP headers.)>>;
 * from ECS POV (point of view) technically there is only one CC (virtual participant), but in reality ('logical') there are multiple ones (clustering).

Version History of ViPLab JSON Message Formats
----------------------------------------------
Version|Date|Author|Comment
-------|----|------|-------
1.0    |17.02.14|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<BR>- [fix][new] config --Octave : already in use for some time.<BR>- Good time for declaring v1.0.
0.21   |13.02.14|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<BR>- [new] config --* --interpreting|running --timelimitInSeconds : CPU time limit.
0.20   |08.05.13|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<BR>- [changed] config --C --checking --behavior : entry ignored: "mergeAndInclude" semantics for all systems.
0.19   |11.03.13|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<BR>- [changed] config --C --checking --behavior : entry ignored: devel system "mergeAndInclude", prod systems "element" semantics.
0.18   |06.02.13|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<BR>- [new] config --C --checking --behavior <BR>- [fix] config --Java --checking --sources : only elementIDs are allowed
0.17.1 |20.11.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): elementProperties --MERGE_ID --tabType : fix of unsuited spec
0.17   |16.11.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<BR>- elementProperties<<BR>>- config --Java --checking; [[/exercises#Java_checking_semantics|Java checking semantics]]
0.16.1 |14.11.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): config --C|C++|Java --compiling --sources : implicit and explicit compilation
0.16   |13.11.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): config --C|C++|Java --compiling --sources
0.15.1 |12.11.12|Stephan Rudlof|[Notes to merging](/exercises#Notes_to_merging): LF at end of merged elements
0.15   |12.09.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): state that merging expects '''non-empty''' array(s)
0.14   |24.08.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<BR>- [new] Java chain<BR> - [fix] some corrections/updates regarding other chains
0.13   |25.07.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<BR>- [new] department
0.12   |25.07.12|Stephan Rudlof|[Result JSON Format](/results#Result_JSON_Format):<BR>- [new] computation<BR>- [done]: no more [toDo] and [outdated] sections
0.11   |14.06.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): support for mapping of generated elements (mergeID)<BR>- [new] config --C|C++: --merging --mergeID<BR>- [mod] elementMap
0.10   |13.06.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format):<<BR>>- added config --DuMuX --running --observe_stderr<BR>[Result JSON Format](/results#Result_JSON_Format):<BR>- added status, index<<BR>>- marked [toDo] and [outdated] sections
0.9    |13.04.12|Stephan Rudlof|[JSON Objects in Reuslt--element[] ](/exercises#JSON_objects_in_Exercise--elements.5B.5D): added ''group''<<BR>>[[/exercises#Exercise_JSON_Format|Exercise JSON Format]]:<<BR>>- added C++ chain<<BR>>- added config --C|C++ --merging [] 
0.8    |11.04.12|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): added ''TTL''.
0.7.1  |30.05.11|Stephan Rudlof|[JSON Objects in Reuslt--element[] ](/exercises#JSON_objects_in_Exercise--elements.5B.5D):<BR>- added: ''MIMEtype'' value "application/x-vgf3", ''MIMEtype'' value "application/x-vgfc".
0.7    |30.05.11|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): added ''routing'', ''elementMap''<BR>[JSON Objects in Reuslt--element[] ](/exercises#JSON_objects_in_Exercise--elements.5B.5D): removed ''filename'' (functionality given by ''elementMap'' now)
0.6.1  |09.05.11|Stephan Rudlof|[JSON Objects in Reuslt--element[] ](/exercises#JSON_objects_in_Exercise--elements.5B.5D): added ''syntaxHighlighting'' values "Matlab", "XML"
0.6    |04.05.11|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): added specification config--DuMuX<<BR>>[JSON Objects in Reuslt--element[] ](/exercises#JSON_objects_in_Exercise--elements.5B.5D): added ''MIMEtype'' value "text/xml"
0.5    |27.10.10|Stephan Rudlof|[JSON Objects in Reuslt--element[] ](/exercises#JSON_objects_in_Exercise--elements.5B.5D):<<BR>>- added: ''MIMEtype'' value "application/x-vgf",<<BR>>- new: ''ID''.
0.4    |16.12.09|Stephan Rudlof|[Exercise JSON Format](/exercises#Exercise_JSON_Format): added specification config--Matlab
0.3    |30.11.09|Stephan Rudlof|[JSON Objects in Reuslt--element[] ](/exercises#JSON_objects_in_Exercise--elements.5B.5D): added ''MIMEtype'' value "image/png"
0.2    |21.09.09|Stephan Rudlof|new: [/numlab/interrupts](/interrupts)<<BR>renaming: postTimeClient, resultPostTimeClient -> postTime; solutionGetTimeClient -> solutionGetTime 
0.1    |08.09.09|Stephan Rudlof|initial version||

!!! note

    This history is just for changes in syntax and/or semantics of ViPLab related JSON messages:

     * It must ''not'' be updated for mere improvements of the documentation.
     * Fixes in JSON examples are ''not'' reflected here: Exercise|Solution|Result|Interrupt JSON Format sections (the tables) count.

ViPLab JSON Message Formats are found in:

 * [Excercise Message](exercises#Message)
 * [Solution Message](/solutions#Message)
 * [Result message](/results#Message)
 * [Interrupt Mesage](/interrupts#Message)

Because they depend on each other, there is a centralized history here.

ECS Filter-Plugins
------------------
With a filter plugin you are able to transform the messages received or send by
the ECS, i.e.  this makes it possible to transfer some application logic to the
ECS (for general description of filter plugins see
[[../core#filter_plugins|Filter-Plugins]]). 

### Exercises Filter-Plugin
This filter masquerades the response of an request to {{{/numlab/exercises/<id>}}}. With a query string the request only contain toplevel properties with simple string type. For now these toplevel properties are:

 * postTime
 * identifier
 * comment
 * name
 * description
 * environment

The querystring parameter is known as: {{{properties}}}. You're allowed to note
multiple toplevel properties, spaced by comma. Please be patient not to include
any spaces in the list.

Working ressource notations are:

    http://ecs.uni-stuttgart.de/numlab/exercises/27?properties=name
    http://ecs.uni-stuttgart.de/numlab/exercises/27?properties=name,description
    http://ecs.uni-stuttgart.de/numlab/exercises/27?properties=name,description,comment,identifier

Problematic/forbidden notations are:

    http://ecs.uni-stuttgart.de/numlab/exercises/27?properties= name
    http://ecs.uni-stuttgart.de/numlab/exercises/27?properties=name ,description
    http://ecs.uni-stuttgart.de/numlab/exercises/27?properties =name,description,comment , identifier


Also have a look at the [examples](../core/examples#exercises_show_filter).
