# Computation Template JSON Message

A *computation template*  can be 

 * the definition of an *Exercise* in the context of a learning environment, or
 * a pre-configured execution of a research software (stored in a docker image), used to show reproducabiltiy of a research work or to reduce complex software environments to specific functionality


## Example (informal)
Note: `//` with text following until EOL is a comment,

 * which is not covered by the JSON spec, and
 * should not be contained in message sends in real;
 * but nevertheless it would help, if JSON parsers could just ignore them.

```
{ "Exercise" :
{ "postTime"    : "1985-04-12T23:20:50.52Z", // must, given by TC
  "TTL"         : 360000, // opt, given by TC (here 10*10*3600s = 100h)
  "identifier"  : "Numerik II, SS10, Aufgabe 1, v1.0", // name of exercise for teachers
  "department"  : "RUS", // opt, for filtering exercises
  "comment"     : "Um die Studenten mal richtig zu fordern.", // optional
  "name"        : "Aufgabe 1",                       // name of exercise to be shown in SCs
  "description" : "Schreiben Sie eine C-Funktion...",// optional: short description (could be used as 
                                                     // subtitle, longer descriptions in "elements").
  "elements" : // must: at least one array element
  [
    { "identifier": "preamble",                       // for referencing
      "visible"   : true,                             // if it has to be shown in SC
      "modifiable": false,                            // optional
      "name"      : "Info: source before your code.", // name of element for student
      "MIMEtype"  : "text/plain",                     // optional (default: "text/plain")
      "syntaxHighlighting": "C",                      // optional (default: "none")
      "emphasis"  : "low",                            // optional (for rendering)
      "value"     : "#include <stdio.h>\n"            // source (newline should be in effect after decoding)
    },
    { "identifier": "codeFromStudent",
      "visible"   : true,
      "modifiable": true,                             // bool; has to be true here to make it editable in SC
      "name"      : "Fill in your code!",
      "MIMEtype"  : "text/plain",                     // optional (default: "text/plain")
      "syntaxHighlighting": "C",                      // optional (default: "none")
      "emphasis"  : "medium",
      "value"     : "void bar() { /* Schreiben Sie hier Code, der \"bar\" ausgibt. */\n\n}\n" 
                                                      // source (template) from teacher
    },
    { "identifier": "postscript",
      "visible"   : true,
      "modifiable": false,
      "name"      : "Info: source after your code calling bar() in it.",
      "MIMEtype"  : "text/plain",                     // optional (default: "text/plain")
      "syntaxHighlighting": "C",                      // optional (default: "none")
      "emphasis"  : "low",
      "value"     : "int main() { bar(); return 0; }" // source
    }
  ], // elements[]

  "environment" : "anIdentifier", // optional

  "config" :
  { "C" : // C language specific stuff
    {
      // phases
      "merging"  : // optional / must (for compiling)
      {
        "sources" : ["preamble", "codeFromStudent", "postscript"] // identifiers of to be merged sources
                                                                  // (elements with text content) from above
      },
      "compiling" : // optional / must (for checking/linking)
      {
        "compiler" : "gcc",      // string
        "flags"    : "-O2 -Wall" // string
      },
      "checking" : // optional
      {
        "sources": ["codeFromStudent"],   // sources to be checked
        "forbiddenCalls": "system execve" // forbidden call names separated by WS
      },
      "linking" : // optional / must (for running)
      {
        "flags" : "-lm" // string
      },
      "running" : // optional
      {
        "commandLineArguments" : "--stepwidth 0.001" // string
      },
      "stopAfterPhase" : "running" // optional: "running" here is the same as omitting "stopAfterPhase"
    } // "C"
  } // "config"
} // "Exercise"
} // "Exercise" wrapper
```

### Exercise JSON Format

Note: wrapper "Exercise" omitted.

_wrapper "Exercise" around the following_

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | AS |
|---------------|----------------------------------------|-----------|------------|--------|----|
|postTime |string UTC |must |timestamp from TC|Timestamp from TC: in addition a CC side timestamp could be made. A change of an exercise should be performed by a POST of the changed one, combined with DELETEs of older versions (PUT has problems, see [NumLab](/viplab)). | Still needed? Couldn't the websocket API or anybody else generate this? |
|TTL |uint seconds |optional |time to live from teacher|Limits validity of Exercise. If (currentTime > postTime + TTL) Exercise can be deleted automatically. In all other cases (includes no TTL) it **must not** be deleted automatically (it can always be deleted manually, of course). For extending the lifetime of an exercise it has to be rePOSTed (leading to a new one with updated postTime).| Is this actually done? Ensured by the backend? Could also be parameter under config/resources? |
|identifier |string |must       |name for teachers |Technically there is no need for uniqueness of the identifier at this protocol level: e.g. if modifying an exercise, there may be different versions with the same ID. | If there is no technically need, why do we have it? |
|department |string |opt        |institution shortcut| For filtering exercises at frontend (e.g. for showing only a subset).| Frontend feature |
|comment             |string     |opt               | comment from teacher |Generic element: same as comment from others: wrapper (here teacher) says who is originator.| |
|name |string |opt |name of exercise to be shown in SCs. | If it is missing, "identifier" could be used.| Frontend feature |
|description |string (plain text) |opt |short description |Mostly a few lines. A longer description in different formats can be put in "elements" (see below). | |
|elements             |[ {...}, {...}, ... ] |must |array containing [JSON objects](#json-objects-in-exercise-elements): there has to be at least one element | | |
|environment |string |opt |identifier for referring to static environment meta data |Static environment meta data stores complex exercise configurations containing data, which is changing very slowly: e.g. which external post processing tools will be called how for a set of exercises. Optional arguments here may also be placed into the static environment: e.g. linking--flags. | Do we need this? Didn't find it in tests. What can it be? |
|routing      |struct |opt | | | Needed without ECS? |
|routing --solutionQueue |string |must |Queue to be used for posting Solutions.|If there is *no* "routing" struct at all, implicit default value for "solutionQueue" is 'solutions', otherwise this key has to be set.|
|elementMap |{ SOURCE_ID: URI, ... } |opt |mapping of sources to URIs, SOURCE_ID has to be an identifier (string) of an existing element (ELEMENT_ID), or an identifier of a yet to be generated element (MERGE_ID, availability depends on chain) |Currently only file:// protocol mapping to local file without directory path supported (Note: this is 'file:///filename.txt' for 'filename.txt'). If and how mapped elements will be used, depends on semantics of computation chain defined by "config".<BR> **C/C++/Java**: created files with suffix '.c' resp. '.cpp' resp. '.java' can (explicitly) or will (implicitly) be taken as a source for compilation (see config --C/C++/Java --compiling --sources); all others will just be created (they may be used as input for compilation (e.g. '.h', '.hpp') or executable).| Move to "config--merging"? |
|elementProperties |{ SOURCE_ID: { string: VALUE, string: VALUE, ... } |opt |mapping of sources to {} containing properties ('string: VALUE' pairs); SOURCE_ID as defined for elementMap; unique property names, VALUE types and their allowed values have to be defined. |What kind of properties are allowed and their semantics may depend on chain (config --).| From test examples it seems to be that SOURCE_ID can also be the "group" name of an element. Unclear if still needed... |
|elementProperties --MERGE_ID --tabType | one of {"source", "header", "data"} |opt | [C/C++][frontend] kind of merged sources | Ignored by backend.| |
|config      |struct |opt | configuration | | make "merging" equal for all languages? |
|config --Octave |struct |opt | configuration for Octave programming|Same as 'config --Matlab', just replace "Matlab" by "Octave" (all sub attributes are the same).| |
|config --Matlab |struct |opt | configuration for Matlab programming|Chain: merging -> checking -> interpreting. Merging may be a trivial merge consisting of just one element. Checking needs a positive list of callable functions.| |
|config --Matlab --merging |struct |must (may be trivial merge) | | See [Notes to merging](#notes-to-merging). | |
|config --Matlab --merging --sources |[ELEMENT_ID, ELEMENT_ID, ...] |must |array of one or more identifiers of to be merged sources; defined by elements[]{ELEMENT_ID} above | | |
|config --Matlab --checking |struct |opt | checking for legal function calls in source code |Performed after merging. If omitted, no checking will be done.| |
|config --Matlab --checking --sources |[ELEMENT_ID, ELEMENT_ID, ...] |must |array of identifiers of to be checked sources; given by elements[]{ELEMENT_ID} above | | |
|config --Matlab --checking --allowedCalls |string |must |*allowed* call names separated by WS; only idents (no braces, no func args) | | |
|config --Matlab --interpreting |struct |opt | |If necessary, there may be sub keys later.|
|config --Matlab --interpreting --timelimitInSeconds |int |opt |CPU time limit |For semantics see [Notes to "timelimitInSeconds"](#notes-to-timelimitinseconds). | |
|config --Matlab --stopAfterPhase | one of {"merging", "checking", *"interpreting"*} |opt |chain: merging -> checking -> interpreting |If omitted, CC tries to execute all given phases (same as *"interpreting"*). | |
|config --C |struct |opt | configuration for C programming|Chain: merging -> compiling -> [checking] -> linking -> running. Currently checking is the only phase which can be omitted, if there is a program to be run. | |
|config --C++ |struct |opt | configuration for C++ programming|Chain: merging -> compiling -> linking -> running. Checking is not supported for C++. | |
|config --Java |struct |opt | configuration for Java programming|Chain: merging -> compiling -> running. Checking is not supported for Java. | Checking is supported?! See below various attributes...|
|config --C/C++/Java --merging |struct or [] |must / opt (if stopped before) | | See [Notes to merging](#notes-to-merging). | Can it be stopped before? |
|config --C/C++/Java --merging --sources |[ELEMENT_ID, ELEMENT_ID, ...] | xor must |array of one or more identifiers of to be merged sources; defined by elements[]{ELEMENT_ID} above | Xor must: this xor *config --C/C++/Java --merging []*. This alternative is for compiling a single source file, generated by a single merge. | Omit and make it always explicit? | 
|config --C/C++/Java --merging --mergeID |MERGE_ID |opt |identifier of merged sources; has to be unique in union set of: elements[]{ELEMENT_ID} above, together with all mergeIDs (in case of multiple merges) | If given, merge results can be explicitly defined as sources to be compiled by config-- C/C++/Java --compiling --sources; and/or mapped to a file by "elementMap" above. A merge result not being compiled explicitly or implicitly has to be elementMap'ped to become visible as a file to started commands: e.g. compiler (including it as header file) or executable (loading it as input file). | |
|config --C/C++/Java --merging [] |{ "sources": [ELEMENT_ID, ELEMENT_ID, ...], "mergeID": MERGE_ID }, ... |xor must |array of one or more structs as defined by previous **two** items | Xor must: this xor *config --C/C++ --merging --sources* and (opt) *config --C/C++ --merging --mergeID*. This alternative is for compiling multiple source files, generated by multiple merges.| |
|config --C/C++/Java --compiling |struct |must / opt (if stopped before) | | | |
|config --C/C++/Java --compiling --sources|[MERGE_ID, MERGE_ID, ...] |opt | Explicit compilation: MERGE_IDs have to be mergeIDs from merging phase, no mergeID should occur multiple times here. Overrides (implicit) default behavior.| If given, only referenced sources will be compiled (explicitly). If omitted, compilation may be done implicitly by not elementMap'ing some merge result (gets automatically generated filename then), elementMap'ing to a file with suited suffix ('.c', '.cpp', '.java') and name/path (Java). Also see config --C/C++/Java --merging --mergeID and elementMap.| Allow only explicit way? |
|config --C/C++ --compiling --compiler |string |must |compiler to be used, e.g. "gcc" | | |
|config --C/C++ --compiling --flags |string |must |CFLAGS |e.g. "-O2" or "" | |
|config --Java --compiling --flags |string |opt |compile flags |e.g. "-v" | |
|config --C/Java --checking |struct |opt | checking for illegal function calls in source code |Performed after compiling, because compiler gives better syntax error messages. Can be omitted, if no checking should be done.| How can it be omitted?  |
|config --C/Java --checking --sources |[ELEMENT_ID, ELEMENT_ID, ...] |must |array of identifiers of to be checked sources; given by elements[]{ELEMENT_ID} above | |
|config --C --checking --behavior | one of {*mergeAndInclude*, "element"} | **Ignored!** Hardwired semantics is "mergeAndInclude".|"mergeAndInclude": merged elements will be checked alltogether, types are given by header #include's; *"element"*: each element will be checked for itself, some predefined standard types| For more details see [C checking semantics](#c-checking-semantics).| |
|config --C --checking --forbiddenCalls |string |must |*forbidden* call names separated by WS; only idents (no braces, no func args) | | |
|config --Java --checking --forbiddenCalls |string |opt |*forbidden* call name expressions separated by WS | For semantics see [Java checking semantics](#java-checking-semantics).| |
|config --Java --checking --allowedCalls |string |opt |*allowed* call name expressions separated by WS | For semantics see [Java checking semantics](#java-checking-semantics).| |
|config --C/C++ --linking |struct |must / opt (if stopped before) | | | |
|config --C/C++ --linking --flags |string |must |LFLAGS |e.g. "" | |
|config --C/C++/Java --running |struct |must / opt (if stopped before) | | | |
|config --C/C++/Java --running --commandLineArguments |string |opt |arguments given to main() function |Outlook: mechanism for transforming input values (e.g. by sliders) into CLI arguments. | To be addressed now? |
|config --C/C++/Java --running --timelimitInSeconds |int |opt |CPU time limit |For semantics see [Notes to "timelimitInSeconds"](#notes-to-timelimitinseconds). | |
|config --Java --running --flags |string |opt |flags given to JVM | | |
|config --Java --running --mainClass |string |opt (if unique) / must (if not unique) |class containing "public static void main(String[] args) {" |"args" may be another name. If main function is unique, its correct class should be detected automatically.| |
|config --C --stopAfterPhase | one of {"merging", "compiling", "checking", "linking", *"running"*} |opt |chain: merging -> compiling -> checking -> linking -> running |If omitted, CC tries to execute all given phases (same as *"running"*). | |
|config --C++ --stopAfterPhase |one of {"merging", "compiling", "linking", *"running"*} |opt |chain: merging -> compiling -> linking -> running |If omitted, CC tries to execute all given phases (same as *"running"*). |  |
|config --Java --stopAfterPhase |one of {"merging", "compiling", *"running"*} |opt |chain: merging -> compiling -> running |If omitted, CC tries to execute all given phases (same as *"running"*). | checking? |
|config --DuMuX |struct |opt | configuration for DuMuX|Chain: running. There is a first simple chain for just running preinstalled executables. Later this will be extended. | |
|config --DuMuX --running |struct |must | | | |
|config --DuMuX --running --executable |string |must |name of executable to run (as in backend file system)| | |
|config --DuMuX --running --commandLineArguments |string |opt |additional command line args| | |
|config --DuMuX --running --timelimitInSeconds |int |opt |CPU time limit |For semantics see [Notes to "timelimitInSeconds"](#notes-to-timelimitinseconds). | |
|config --DuMuX --running --observe_stderr |bool (*false*)|opt |if true, transfer intermediate stderr Results| | Is this implemented?|
 
