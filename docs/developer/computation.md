# Computation JSON Message

A *computation* represents the combination of a *computation template* and a
corresponding *computation task*. It is the input of any backend.

The JSON message of a *computation* is very similar to a *computation template*,
so the interested reader is also referred to [its documentation](computation_template.md).

## Example (informal)
Note: `//` with text following until EOL is a comment,

 * which is not covered by the JSON spec, and
 * should not be contained in message sends in real;
 * but nevertheless it would help, if JSON parsers could just ignore them.

``` json title="Computation Message Example"
{ 
  "identifier"  : "4598393-95bf-409a-98a5-ee375982c3e", // uuid, created by websocket api
  "environment" : "C", // important for interpreting configuration 
  "files" : // must: at least one array element
  [
    { 
      "identifier": "22483f42-95bf-984a-98a5-ee9485c85c3e", // uuid from template 
      "path"      : "code.c"                                // filename on backend 
      "parts" : // must: at least one array element
      [ 
        { "identifier": "preamble",  // identifier from template
          "access"    : "visible",   // access from template
          "content"   : "I2luY2x1ZGUgPHN0ZGlvLmg-Cg"      // source (base64url encoded) 
                                                          // decoded: #include <stdio.h>\n
        },
        { 
          "identifier": "codeFromStudent", // identifier from template
          "access"    : "modifiable",             
          "content" : "dm9pZCBiYXIoKSB7IHByaW50ZigiYmFyIQoiKTsKfQo" // content from task
                                    // decoded: void bar() { printf(\"bar!\\n\");\n}\n 
        },
        { 
          "identifier": "postscript", // identifier from template
          "access"    : "visible",
          "content" : "aW50IG1haW4oKSB7IGJhcigpOyByZXR1cm4gMDsgfQ" // source
                                                // decoded: int main() { bar(); return 0; }
        }
      ] // parts[]
    }
  ], // files[]
  "configuration" : { 
    "compiling.compiler" : "gcc",                  
    "compiling.flags"    : "-O2 -Wall"            
    "checking.sources"   : ["codeFromStudent"],    // identifier to parts
    "checking.forbiddenCalls": "system execve"     // forbidden call names separated by WS
    "linking.flags"      : "-lm"                 
    "running.commandLineArguments" : "--stepwidth 0.5"
                                            // mustache template with injected arguments
  }
}
```

## Explaining the JSON Format

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment 
|---------------|----------------------------------------|-----------|------------|-------
|identifier | string (UUID) | must | the identifier of this computation | is generated by the websocket api |
|environment |one of {"C", "C++", "Java", "Matlab", "Octave", "Container", "DuMuX"} | must | Specifies the environment used for the Computation. It defines language, runtime, libraries and tools |
|files | [ {...}, {...}, ... ] |must |array containing [File objects](#json-objects-in-files): there has to be at least one element |
|configuration | struct |opt/must (depends on environment) | Environment specific configurations | Different phases can be configured like compiling, checking (for legal function calls in source code), ... | 
|configuration --compiling.sources | [FILE_ID, FILE_ID, ...] | must | Array of identifiers of [JSON File objects](#json-objects-in-files). Explicit compilation (only referenced sources will be compiled). | for **C, C++, Java** | 
|configuration --compiling.compiler |string |must |compiler to be used, e.g. "gcc" | for **C, C++** |
|configuration --compiling.flags |string |must for **C, C++**; optional for **Java** |CFLAGS for **C/C++**; compile flags for **Java** |e.g. "gcc" for **C/C++**; "-O2" or "" for **Java** | |
|configuration --checking.sources |[PART_ID, PART_ID, ...] |must if checking should be performed |array of identifiers of to be checked sources; given by parts[]{PART_ID} (see [below](#json-objects-in-parts)) | for **Matlab, Octave, C, Java** |
|configuration --checking.allowedCalls |string |must if checking should be performed |for **Matlab/Octave**: *allowed* call names separated by WS; only idents (no braces, no func args)<br>for **Java**: *allowed* call name expressions separated by WS | for **Java** semantics see [Java checking semantics](computation_template.md#java-checking-semantics). | 
|configuration --checking.forbiddenCalls |string |must if checking should be performed |for **C**: *forbidden* call names separated by WS; only idents (no braces, no func args)<br>for **Java**: *forbidden* call name expressions separated by WS | for **C** semantics see [C checking semantics](computation_template.md#c-checking-semantics)); for **Java** semantics see [Java checking semantics](computation_template.md/#java-checking-semantics).  | 
|configuration --linking.flags |string |must |LFLAGS | for **C, C++**; e.g. "" | 
|configuration --running.stdinFilename | FILE_ID | must | the file identifier that is passed to **Matlab/Octave** via standard-in | |
|configuration --running.timelimitInSeconds |int |opt |CPU time limit | for all **environments**; for semantics see [Notes to "timelimitInSeconds"](computation_template.md/#notes-to-timelimitinseconds). | |
|configuration --running.commandLineArguments |string |opt | for **C, C++, Java**: arguments given to main() function; for **DuMuX, Container**: additional command line args | 
|configuration --running.flags |string |opt |flags given to JVM | for **Java** | |
|configuration --running.mainClass |string |opt (if unique) / must (if not unique) |class containing "public static void main(String[] args) {" | for **Java**: "args" may be another name. If main function is unique, its correct class should be detected automatically.| |
|configuration --running.executable |string |must |name of executable to run (as in backend file system)| for **DuMuX** | |
|configuration --running.entrypoint | string | must | executable to run inside the container | for **Container** |
|configuration --running.observe_stderr |bool (*false*)|opt |if true, transfer intermediate stderr Results| for **DuMuX, Container**|
|configuration --resources.image | url | must | location of the image to be executed | for **Container**; has to be a tar |
|configuration --resources.volume | string | must | path in the container where data is placed | for **Container** | 
|configuration --resources.memory | string | opt (*64mb*)| memory limit for the container | for **Container** | 
|configuration --resources.numCPUs | int | opt | number of CPUs for the container | for **Container** | 

### JSON objects in files

An object in array files[] has the following members:

|Key |Type (an enum default is marked by _italics_) |Opt / Must |Description|Comment
|----|----------------------------------------------|-----------|-----------|-------
|identifier | string (UUID) | must | reference to a computation template file | 
|path | string | must | absolute path to file | It is *not* allowed to start with '/' |
|parts | [{...}, {...}, ...] | must | array containing [part objects](#json-objects-in-parts). There has to be at least one. |


### JSON objects in parts

An object in array parts[] has the following members:

|Key |Type (an enum default is marked by _italics_) |Opt / Must |Description|Comment
|----|----------------------------------------------|-----------|-----------|-------
|identifier | string | must | reference to a computation template part | 
|access | one of {"invisible", "visible", "modifiable", "template"} | must | the access level of this part | equals to the access level of this part in the computation template |
|content |string |must | base64url-encoded source code | 
