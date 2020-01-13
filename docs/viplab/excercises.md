<<TableOfContents>>

= /numlab/exercises =

Persistent resource.

== Interface ==

POST, GET, DELETE ([[../../core#persistent-messages|see ''English section name unknown so far.'']])

=== Generic ===

  POST:: creates a new persistent Exercise: it gets an automatically gererated index for accessing the POSTed Exercise.
  GET:: returns an URL list of all stored Exercises there. Each list entry can be used for accessing a formerly stored Exercise.
  GET resourceURL/ix:: returns stored Exercise with index ix.
  DELETE resourceURL/ix:: deletes stored Exercise with index ix.

=== ViPLab Specific ===

POST/DELETE will be used by TCs, GET by all clients.

A teacher client (TC) POSTs and DELETEs Exercises; student clients (SCs) and computation clients (CCs) only GET Exercises.

There are some constraints:
 * No PUT of Exercises:
   for changing an Exercise PUT ''won't'' be used. POST of a changed one, followed (sometime) from DELETE of outdated version/s, is the way to go.
   <<BR>>Rationale: content of an Exercise associated to a decent URL (automatically created, but fixed index) must not change.

 * No reuse of Exercise indices:
   automatically created Exercise indices are monotonically increasing (practically infinite). So the following is valid for an old Solution after deleting some Exercise:
   * it references the correct Exercise, which has existed at time of Solution creation and continues to exist, or
   * it references nothing.
 
Fullfilling these constraints avoids the big problem of referencing a ''wrong'' Exercise!

== Message ==

=== Excercise Message JSON Definition by Example (informal) ===
Note: `//` with text following until EOL is a comment,

 * which is not covered by the JSON spec, and
 * should not be contained in message sends in real;
 * but nevertheless it would help, if JSON parsers could just ignore them.

{{{#!cplusplus numbers=off
{ "Exercise" :
{ "postTime"    : "1985-04-12T23:20:50.52Z", // must, given by TC
  "TTL"         : 360000, // opt, given by TC (here 10*10*3600s = 100h)
  "identifier"  : "Numerik II, SS10, Aufgabe 1, v1.0", // name of exercise for teachers
  "department"  : "RUS", // opt, for filtering exercises
  "comment"     : "Um die Studenten mal richtig zu fordern.", // optional
  "name"        : "Aufgabe 1",                       // name of exercise to be shown in SCs
  "description" : "Schreiben Sie eine C-Funktion...",// optional: short description (could be used as subtitle,
                                                     //   longer descriptions should be in "elements").
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
      "value"     : "void bar() { /* Schreiben Sie hier Code, der \"bar\" ausgibt. */\n\n}\n" // source (template) from teacher
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
      //
      "merging"  : // optional / must (for compiling)
      {
        "sources" : ["preamble", "codeFromStudent", "postscript"] // identifiers of to be merged sources (elements with text content) from above
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
}}}

=== Exercise JSON Format ===

Note: wrapper "Exercise" omitted.

||<-5:> wrapper "Exercise" around the following ||
||'''Key [--Subkey]''' ||'''Type''' (a default is marked by ''italics'')||'''Opt / Must''' ||'''Description''' ||'''Comment''' ||
||postTime ||string UTC ||must ||timestamp from TC||Timestamp from TC: in addition a CC side timestamp could be made. A change of an exercise should be performed by a POST of the changed one, combined with DELETEs of older versions (PUT has problems, see [[../|NumLab]]). ||
||TTL ||uint seconds ||optional ||time to live from teacher||Limits validity of Exercise. If (currentTime > postTime + TTL) Exercise can be deleted automatically. In all other cases (includes no TTL) it ''' must not ''' be deleted automatically (it can always be deleted manually, of course). For extending the lifetime of an exercise it has to be rePOSTed (leading to a new one with updated postTime).||
||identifier ||string ||must       ||name for teachers ||Technically there is no need for uniqueness of the identifier at this protocol level: e.g. if modifying an exercise, there may be different versions with the same ID. ||
||department ||string ||opt        ||institution shortcut|| For filtering exercises at frontend (e.g. for showing only a subset).||
||comment             ||string     ||opt               || comment from teacher ||Generic element: same as comment from others: wrapper says who is originator.||
||name ||string ||opt ||name of exercise to be shown in SCs. || If it is missing, "identifier" could be used.||
||description ||string (plain text) ||opt ||short description ||Mostly a few lines. A longer description in different formats can be put in "elements" (see below). ||
||elements             ||[ {...}, {...}, ... ] ||must ||array containing [[#JSON_objects_in_Exercise--elements.5B.5D|JSON objects in Exercise--elements[] ]]: there has to be at least one element || ||
||environment ||string ||opt ||identifier for referring to static environment meta data ||Static environment meta data stores complex exercise configurations containing data, which is changing very slowly: e.g. which external post processing tools will be called how for a set of exercises. Optional arguments here may also be placed into the static environment: e.g. linking--flags. ||
||routing      ||struct ||opt || || ||
||routing --solutionQueue ||string ||must ||Queue to be used for posting Solutions.||If there is *no* "routing" struct at all, implicit default value for "solutionQueue" is 'solutions', otherwise this key has to be set.||
||elementMap ||{ SOURCE_ID: URI, ... } ||opt ||mapping of sources to URIs, SOURCE_ID has to be<<BR>>- an identifier (string) of an existing element (ELEMENT_ID), or<<BR>>- an identifier of a yet to be generated element (MERGE_ID, availability depends on chain) ||Currently only file:// protocol mapping to local file without directory path supported (Note: this is 'file:///filename.txt' for 'filename.txt'). If and how mapped elements will be used, depends on semantics of computation chain defined by "config".<<BR>>- '''C/C++/Java''': created files with suffix '.c' resp. '.cpp' resp. '.java' can (explicitly) or will (implicitly) be taken as a source for compilation (see config --C|C++|Java --compiling --sources); all others will just be created (they may be used as input for compilation (e.g. '.h', '.hpp') or executable).||
||elementProperties ||{ SOURCE_ID: { string: VALUE, string: VALUE, ... } ||opt ||mapping of sources to {} containing properties ('string: VALUE' pairs); SOURCE_ID as defined for elementMap; unique property names, VALUE types and their allowed values have to be defined. ||What kind of properties are allowed and their semantics may depend on chain (config --).||
||elementProperties --MERGE_ID --tabType ||"source"|"header"|"data" ||opt ||[C/C++][frontend] kind of merged sources || Ignored by backend.||
||config      ||struct ||opt || configuration || ||
||config --Octave ||struct ||opt || configuration for Octave programming||Same as 'config --Matlab', just replace "Matlab" by "Octave" (all sub attributes are the same).||
||config --Matlab ||struct ||opt || configuration for Matlab programming||Chain: merging -> checking -> interpreting. Merging may be a trivial merge consisting of just one element. Checking needs a positive list of callable functions.||
||config --Matlab --merging ||struct ||must (may be trivial merge) || || See [[#Notes_to_merging|Notes to merging]]. ||
||config --Matlab --merging --sources ||[ELEMENT_ID, ELEMENT_ID, ...] ||must ||array of one or more identifiers of to be merged sources; defined by elements[]{ELEMENT_ID above || ||
||config --Matlab --checking ||struct ||opt || checking for legal function calls in source code ||Performed after merging. If omitted, no checking will be done.||
||config --Matlab --checking --sources ||[ELEMENT_ID, ELEMENT_ID, ...] ||must ||array of identifiers of to be checked sources; given by elements[]{ELEMENT_ID above || ||
||config --Matlab --checking --allowedCalls ||string ||must ||''allowed'' call names separated by WS; only idents (no braces, no func args) || ||
||config --Matlab --interpreting ||struct ||opt || ||If necessary, there may be sub keys later.||
||config --Matlab --interpreting --timelimitInSeconds ||int ||opt ||CPU time limit ||For semantics see [[#Notes_to_.22timelimitInSeconds.22|Notes to "timelimitInSeconds"]]. ||
||config --Matlab --stopAfterPhase || "merging" | "checking" | ''"interpreting"'' ||opt ||chain: merging -> checking -> interpreting ||If omitted, CC tries to execute all given phases (same as ''"interpreting"''). ||
||config --C ||struct ||opt || configuration for C programming||Chain: merging -> compiling -> [checking] -> linking -> running. Currently checking is the only phase which can be omitted, if there is a program to be run. ||
||config --C++ ||struct ||opt || configuration for C++ programming||Chain: merging -> compiling -> linking -> running. Checking is not supported for C++. ||
||config --Java ||struct ||opt || configuration for Java programming||Chain: merging -> compiling -> running. Checking is not supported for Java. ||
||config --C|C++|Java --merging ||struct | [] ||must / opt (if stopped before) || || See [[#Notes_to_merging|Notes to merging]]. ||
||config --C|C++|Java --merging --sources ||[ELEMENT_ID, ELEMENT_ID, ...] ||xor must ||array of one or more identifiers of to be merged sources; defined by elements[]{ELEMENT_ID above || Xor must: this xor ''config --C|C++|Java --merging []''.<<BR>>This alternative is for compiling a single source file, generated by a single merge. ||
||config --C|C++|Java --merging --mergeID ||MERGE_ID ||opt ||identifier of merged sources; has to be unique in union set of: <<BR>>- elements[]{ELEMENT_ID above, together with<<BR>>- all mergeIDs (in case of multiple merges) || If given, merge results can be<<BR>>- explicitly defined as sources to be compiled by config-- C|C++|Java --compiling --sources;<<BR>>- and/or mapped to a file by "elementMap" above.<<BR>>A merge result not being compiled explicitly or implicitly has to be elementMap'ped to become visible as a file to started commands: e.g. compiler (including it as header file) or executable (loading it as input file). ||
||config --C|C++|Java --merging [] ||{ "sources": [ELEMENT_ID, ELEMENT_ID, ...], "mergeID": MERGE_ID }, ... ||xor must ||array of one or more structs as defined by previous '''two''' items || Xor must: this xor ''config --C|C++ --merging --sources'' and (opt) ''config --C|C++ --merging --mergeID''.<<BR>>This alternative is for compiling multiple source files, generated by multiple merges.||
||config --C|C++|Java --compiling ||struct ||must / opt (if stopped before) || || ||
||config --C|C++|Java --compiling --sources||[MERGE_ID, MERGE_ID, ...] ||opt || Explicit compilation: MERGE_IDs have to be mergeIDs from merging phase, no mergeID should occur multiple times here. Overrides (implicit) default behavior.|| If given, only referenced sources will be compiled (explicitly).<<BR>>If omitted, compilation may be done implicitly by<<BR>>- not elementMap'ing some merge result (gets automatically generated filename then),<<BR>>- elementMap'ing to a file with suited suffix ('.c', '.cpp', '.java') and name/path (Java).<<BR>>Also see config --C|C++|Java --merging --mergeID and elementMap.||
||config --C|C++ --compiling --compiler ||string ||must ||compiler to be used, e.g. "gcc" || ||
||config --C|C++ --compiling --flags ||string ||must ||CFLAGS ||e.g. "-O2" or "" ||
||config --Java --compiling --flags ||string ||opt ||compile flags ||e.g. "-v" ||
||config --C|Java --checking ||struct ||opt || checking for illegal function calls in source code ||Performed after compiling, because compiler gives better syntax error messages. Can be omitted, if no checking should be done.||
||config --C|Java --checking --sources ||[ELEMENT_ID, ELEMENT_ID, ...] ||must ||array of identifiers of to be checked sources; given by elements[]{ELEMENT_ID above || ||
||config --C --checking --behavior ||''"mergeAndInclude"'' | "element"||'''Ignored!''' Hardwired semantics is "mergeAndInclude".||"mergeAndInclude": merged elements will be checked alltogether, types are given by header #include's;<<BR>>''"element"'': each element will be checked for itself, some predefined standard types|| For more details see [[#C_checking_semantics|C checking semantics]].||
||config --C --checking --forbiddenCalls ||string ||must ||''forbidden'' call names separated by WS; only idents (no braces, no func args) || ||
||config --Java --checking --forbiddenCalls ||string ||opt ||''forbidden'' call name expressions separated by WS || For semantics see [[#Java_checking_semantics|Java checking semantics]].||
||config --Java --checking --allowedCalls ||string ||opt ||''allowed'' call name expressions separated by WS || For semantics see [[#Java_checking_semantics|Java checking semantics]].||
||config --C|C++ --linking ||struct ||must / opt (if stopped before) || || ||
||config --C|C++ --linking --flags ||string ||must ||LFLAGS ||e.g. "" ||
||config --C|C++|Java --running ||struct ||must / opt (if stopped before) || || ||
||config --C|C++|Java --running --commandLineArguments ||string ||opt ||arguments given to main() function ||Outlook: mechanism for transforming input values (e.g. by sliders) into CLI arguments. ||
||config --C|C++|Java --running --timelimitInSeconds ||int ||opt ||CPU time limit ||For semantics see [[#Notes_to_.22timelimitInSeconds.22|Notes to "timelimitInSeconds"]]. ||
||config --Java --running --flags ||string ||opt ||flags given to JVM || ||
||config --Java --running --mainClass ||string ||opt (if unique) / must (if not unique) ||class containing "public static void main(String[] args) {" ||"args" may be another name. If main function is unique, its correct class should be detected automatically.||
||config --C --stopAfterPhase ||"merging" | "compiling" | "checking" | "linking" | ''"running"'' ||opt ||chain: merging -> compiling -> checking -> linking -> running ||If omitted, CC tries to execute all given phases (same as ''"running"''). ||
||config --C++ --stopAfterPhase ||"merging" | "compiling" | "linking" | ''"running"'' ||opt ||chain: merging -> compiling -> linking -> running ||If omitted, CC tries to execute all given phases (same as ''"running"''). ||
||config --Java --stopAfterPhase ||"merging" | "compiling" | ''"running"'' ||opt ||chain: merging -> compiling -> running ||If omitted, CC tries to execute all given phases (same as ''"running"''). ||
||config --DuMuX ||struct ||opt || configuration for DuMuX||Chain: running. There is a first simple chain for just running preinstalled executables. Later this will be extended. ||
||config --DuMuX --running ||struct ||must || || ||
||config --DuMuX --running --executable ||string ||must ||name of executable to run (as in backend file system)|| ||
||config --DuMuX --running --commandLineArguments ||string ||opt ||additional command line args|| ||
||config --DuMuX --running --timelimitInSeconds ||int ||opt ||CPU time limit ||For semantics see [[#Notes_to_.22timelimitInSeconds.22|Notes to "timelimitInSeconds"]]. ||
||config --DuMuX --running --observe_stderr ||bool (''false'')||opt ||if true, transfer intermediate stderr Results|| ||

==== Notes to "timelimitInSeconds" ====
There is a default CPU time limit (see RLIMIT_CPU of 'man setrlimit') for running or interpreting, which depends on backend configuration. This is good for terminating non-terminating programs, e.g. endless loops.

Optional attribute "timelimitInSeconds" may lower this default CPU time limit; if it is higher than default, it will be ignored.

Setting it to a value as much as possible below default is good for backend response time under high load; especially, if default is configured for performing expensive computations.


==== Notes to TTL mechanism ====
 * TTL mechanism as specified is suited for automatically deleting outdated exercises at development system. However it is not suited for limiting lifetime at production system, because
   * extending exercise' lifetime is not possible without changing its resource index, and
   * referenced exercises may diminish too early (this problems would stay with a PUT new TTL approach!). 


==== Notes to C config ====

===== C checking semantics =====
08.05.2013: Semantics is "mergeAndInclude" for both prod systems and devel system ("element" unused).

The semantics of config--C--checking--behavior values "mergeAndInclude" (default) and ''"element"'' (unused) are totally different.
 * '''"mergeAndInclude"''' (default): Checked will be the merge result of elements after being preprocessed by the C preprocessor, so #include's are allowed and honored. Types have to be defined by the corresponding system headers (e.g. by {{{#include <stdio.h>}}}) and no predefined standard types should be expected. It's possible to have illegal function calls in source elements (being part of some merge) '''not''' being checked (typically teacher code). This means, that during checking it has to and will be looked, in which source element an illegal function call happens, for knowing, if this actually is an error case.
 * '''''"element"''''' (unused): Each element has to contain correct C code for itself. In addition to basic types some standard types defined in system headers are predefined (e.g. FILE, size_t). Using types from the outside -- e.g. other elements defining them or #include's in teacher code elements -- does '''not''' work.

Common for both: no preprocessor commands are allowed in to be checked elements, with only one exception: #include's are allowed in case of "mergeAndInclude". The reason for the latter is to give students access to source elements serving as headers - editable for them or not. Functions from system headers could be #include'd, too.

The teacher has the responsibility to put unwished system calls like system() and others from libc into "forbiddenCalls".
Another point of control is to avoid linking with libs, whose functions shouldn't be used (this does not work with libc (automatically linked)).


==== Notes to Java config ====
 * Mapping of merged sources to correct files in package directory hierarchy works automatically by package detection. So there is no need to manually map merges to corresponding files.
 * Main function detection looks for "public static void main(String[] args) {" in all merges ("args" may be another name) ''not explicitly'' mapped by "elementMap". If it is unique - only one class containing a main() -, it will be used automatically; otherwise which one to use has to be given by config--Java--running--mainClass.



===== Java checking semantics =====
{{{
Matching expressions for use in "allowedCalls" and "forbiddenCalls"

'**' is for pure prefix matching; '*' for more fine-granular matching.

Examples of matching expressions:
- java.io.*   matches all calls to methods in all classes in package, but does not match calls to methods in classes in subpackages;
- java.io.**  matches all calls to methods in all classes in package, _and_ in all classes in subpackages (if they exist);

- java.io.Foo    matches all calls to methods in class Foo, _and_ to methods in inner classes;
- java.io.Foo.*  "       "   "     "  "       "  "     "  , but _not_ to methods in inner classes (eg. to java.io.Foo$Bar.callMe);

- java.io.Foo.callMe     matches (only) the call to method callMe in Foo;
- java.io.Foo$Bar.callMe matches (only) the call to method callMe in inner class Foo$Bar.

If used in "forbiddenCalls" only (property "allowedCalls" missing):
- java.io.*   forbids all calls to methods in all classes in package, but allows calls to methods in classes in subpackages;
- java.io.**  forbids all calls to methods in all classes in package, _and_ in all classes in subpackages (if they exist);

- java.io.Foo    forbids all calls to methods in class Foo, _and_ to methods in inner classes;
- java.io.Foo.*  "       "   "     "  "       "  "     "  , but _not_ to methods in inner classes (eg. to java.io.Foo$Bar.callMe);

- java.io.Foo.callMe     forbids (only) the call to method callMe in Foo;
- java.io.Foo$Bar.callMe forbids (only) the call to method callMe in inner class Foo$Bar.

One or more of these expressions may be given by "allowedCalls" and/or "forbiddenCalls", separated by WS; e.g.
  "forbiddenCalls": "java.io.** java.lang.Class"
.


An allowed() or forbidden() predicate gives true,
- if one of its corresponding matching expressions - an entry in property "allowedCalls" resp. "forbiddenCalls" - matches, or
- if there is no corresponding property given at all.
Otherwise it gives false.

All calls will be filtered by following composed predicate for getting allowed ones:
  allowed(call) && ! forbidden(call)   <=>  ! forbidden(call) && allowed(call)
.

If only one part is given, this predicate reduces to:
  allowed(call)
resp.
  ! forbidden(call)
.

Default for not given part of predicate is
- 'no forbidden': ! forbidden(call) == true, and
- 'all allowed' : allowed(call)     == true.
(opposite defaults would render the given part useless).

If sets 'allowed' and 'forbidden'
- are disjunct         -> 'allowed' are allowed and all other forbidden.
- have an intersection -> ('allowed' minus intersection) is allowed (all other forbidden).

With this logic it is possible to define
- a negative list by giving 'forbiddenCalls' only, or
- a positive one by giving "allowedCalls", or
- a mixture of both.
}}}

==== Notes to DuMuX config ====
 * This "config" is a second step to a more extended one later: it supports choosing between different executables with - optional - one or more input files each.
 * CC allows one or more objects in Exercise--elements[], which will be used as file input for the given executable. Mappings of element to input file have to be given by "elementMap"; how to become input to executable by "commandLineArguments" (see above).<<BR>> How resulting input file(s) (in CWD of started executable)  will be given by "commandLineArgs" to executable, has to be defined outside this document.

==== Notes to merging ====
Merging concatenates given elements for further processing. If non-empty elements do not end with a '\n', it will be added (to have a mapping from each line number of merge result to just ''one'' originating element).

==== JSON objects in Exercise--elements[] ====
An object in array elements[] has the following members:
||'''Key''' ||'''Type''' (an enum default is marked by ''italics'') ||'''Opt / Must''' ||'''Description''' ||'''Comment''' ||
||identifier ||ELEMENT_ID (string) ||must ||for later referencing, has to be unique || ||
||group ||string ||opt ||for grouping elements in frontend || Ignored by backend.||
||visible ||bool (''false'')||opt ||if element should be shown in SC || There may be elements not to be shown: e.g. invisible source code.||
||modifiable ||bool (''false'') ||opt ||editability/changeability of "value" (see below) in SC || ||
||name ||string ||opt ||name to be shown in SC|| If it is missing, "identifier" could be used.||
||MIMEtype ||''"text/plain"'' | "text/html" | "text/xml" | "text/uri-list" ||opt ||info for interpreting content ||"text/uri-list": weblinks to be followed and presented in SC. ||
||syntaxHighlighting || ''"none"'' | "C" | "C++" | "Matlab" | "XML" | "Java" || opt || ''"none"'': no syntax highlighting; "C": syntax highlighting for C programming language; etc. || ||
||emphasis ||"low" | ''"medium"'' | "high" ||opt ||info for rendering || ||
||value ||string ||must ||text or other content || This is the member, which may be modified, if "modifiable" (see above) is true.||

Note: objects in array elements[] are identified via their "identifier" member.
