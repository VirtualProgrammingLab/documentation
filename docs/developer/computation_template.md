# Computation Template JSON Message

A *computation template*  can be 

 * the definition of an *Exercise* in the context of a learning environment, or
 * a pre-configured research software (stored in a docker image), used to show reproducability of a research work or to reduce complex software environments to specific functionality


## Examples (informal)
Note: `//` with text following until EOL is a comment,

 * which is not covered by the JSON spec, and
 * should not be contained in message sends in real;
 * but nevertheless it would help, if JSON parsers could just ignore them.

### C Student Example
``` json title="Computation Template Example using C"
{ 
  "identifier"  : "11483f23-95bf-424a-98a5-ee5868c85c3e", // uuid, created by a frontend launcher
  "version" : "3.0.0" // version of this JSON-spec definition
  "metadata": // information for frontend
    { 
      "displayName" : "Aufgabe 1",  // name of computation template shown in frontend
      "description" : "Schreiben Sie eine C-Funktion...", // short description (could be used  
                                                     // as subtitle, further descriptions in "parts").
    },
  "environment" : "C", // important for interpreting configuration 
  "files" : // must: at least one array element
  [
    { 
      "identifier": "22483f42-95bf-984a-98a5-ee9485c85c3e", // uuid, for referencing
      "path"      : "code.c",                                // filename on backend 
      "metadata"  : // information for frontend
        { 
          "decription" : "Programming exercise to test your knowledge of C",
          "syntaxHighlighting": "C",                    // optional (default: "none")
        },
      "parts" : // must: at least one array element
      [ 
        { 
          "identifier": "preamble",
          "access"    : "visible",   // it is rendered, but can not be changed
          "metadata"  : // what has to be moved to files ?
          { 
            "name"    : "Info: source before your code." // name of element in frontend     
          },
          "content"   : "I2luY2x1ZGUgPHN0ZGlvLmg-Cg"      // source (base64url encoded) 
                                                          // decoded: #include <stdio.h>\n
        },
        { 
          "identifier": "codeFromStudent",
          "access"    : "modifiable",             // it can be edited in the frontend
          "metadata"  :
          { 
            "name"    : "Fill in your code!"
          },
          "content" : "dm9pZCBsZWZ0cGFkX2JhcigpIHsgLyogV3JpdGUgY29kZSB0aGF0IHByaW50cyAiYmFyIiBhbmQgYWRkcyB0aGUgcG9zc2liaWxpdHkgdG8gc3BlY2lmeSBhIGZpZWxkLXdpZHRoICovDQoNCn0"
            // source (template)
            // decoded: void leftpad_bar() { /* Write code that prints "bar" and adds the possibility to specify a field-width */\n\n}\n 
        },
        { 
          "identifier": "postscript",
          "access"    : "visible",
          "metadata"  :
          { 
            "name"      : "Info: source after your code calling bar() in it."
          },
          "content" : "aW50IG1haW4oKSB7IGJhcigpOyByZXR1cm4gMDsgfQ" // source
                                                // decoded: int main() { bar(); return 0; }
        }
      ] // parts[]
    }
  ], // files[]
  "parameters" : // parameters can be used to supply values at runtime to the configuration
  [
    {
      "mode": "fixed", // in this section, the parameters can only be of mode fixed
      "identifier": "__STEPWIDTH__",
      "metadata": {
        "guiType": "dropdown",
        "name": "fieldwidth",
        "decription" : "Fieldwidth that is used to specify the width of the field of the output. To give an example, if the fieldwidth is 6, the output of bar(6) is padded like this: '   bar'",
      },
      "options": [
        {
          "value": "0.05",
          "selected": true
        },
        {
          "value": "0.5"
        },
        {
          "value": "1"
        }
      ],
      "validation": "onlyone"
    }
  ],
  "configuration" :
  { 
    "compiling.compiler" : "gcc",                  // string
    "compiling.flags"    : "-O2 -Wall"             // string
    "checking.sources"   : ["codeFromStudent"],    // identifier to parts
    "checking.forbiddenCalls": "system execve",    // forbidden call names separated by WS
    "linking.flags"      : "-lm",                  // string
    "running.commandLineArguments" : "--stepwidth {{ __STEPWIDTH__ }}"
                                               // mustache template if parameters are used
  }
}
```

### Parameters Example
``` json title="Computation Template Example using Container"
{ 
  "identifier"  : "11483f23-95bf-424a-98a5-ee5868c85c3f", 
  "version" : "3.0.0",
  "metadata": { 
    "displayName" : "Parameters Example",  
    "description" : "This is a 'Hello World' example showing the usage of parameters. Please introduce yourself so that the Hello World-Container can print your information...",
    "viewer" : ["CSV", "ParaView", "Image"]
  },
  "environment" : "Container", 
  "files" : 
  [
    { 
      "identifier": "22483f42-95bf-984a-98a5-ee9485c85c3f", 
      "path"      : "params.ini",                              
      "metadata"  : 
        {  
          "syntaxHighlighting": "ini"                   
        },
      "parts" : 
      [
        {
          "identifier": "f3fc4404-3529-4962-b252-47bc4ddd02a1",
          "access": "template",
          "metadata": {
            "name": "Parameter in part"
          },
          "parameters" : 
          [
            {
              "mode" : "any",
              "identifier" : "__sliderSingle__", 
              "metadata" : {
                "guiType" : "slider",
                "name": "temperature",
                "vertical": false,
                "description" : "How hot do you like your coffee? (in degrees Celsius) - Tip: Typical Serving Temperature lies between 65 and 70 Degrees"
              },
              "default": [
                10
              ],
              "min": 0,
              "max": 500,
              "step": 10,
              "validation": "range"
            }
          ],
          "content": "W2NvZmZlZSBwcmVmZXJlbmNlXQpjb2ZmZWVUZW1wZXJhdHVyZT17e19fc2xpZGVyU2luZ2xlX199fQ"
          // decoded: 
          // [coffee preference]
          // coffeeTemperature={{__sliderSingle__}}
        },
        {
          "identifier": "ceb051d8-b50c-4814-983a-b9d703cae0c6",
          "access"    : "template",
          "metadata"  :
              { 
                "name"      : "params.ini file part"
              },
          "parameters":
          [
            {
              "mode" : "fixed",
              "identifier" : "__checkbox__", 
              "metadata" : {
                "guiType": "checkbox",
                "name": "Things I like",
                "description" : "Select things you like"
              },
              "options": [
                {
                  "text" : "Programming",
                  "value" : "programming",
                  "selected" : true
                },
                {
                  "value" : "debug"
                },
                {
                  "value" : "make_plot"
                }
              ],
              "validation": "anyof"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__radioButton__", 
              "metadata" : {
                "guiType": "radio",
                "name": "Favorite PL",
                "description" : "Select your favorite programming language"
              },
              "options": [
                {
                  "value" : "debug"
                },
                {
                  "value" : "serial",
                  "selected" : true
                },
                {
                  "value" : "hpc",
                  "disabled" : true
                },
                {
                  "text" : "Sssss... Python ...ssssS",
                  "value" : "Python"
                }
              ],
              "validation": "oneof"
            },
            {
              "mode" : "fixed",
              "identifier" : "__dropdownSingle__", 
              "metadata" : {
                "guiType": "dropdown",
                "name": "Fridge",
                "description" : "How often do look into the fridge a day?"
              },
              "options": [
                {
                  "value" : "Please choose one",
                  "disabled" : true
                },
                {
                  "value" : "1p",
                  "selected" : true
                },
                {
                  "text" : "1 a day",
                  "value" : "Once a day"
                },
                {
                  "value" : "1pncmin"
                },
                {
                  "value" : "2p"
                },
                {
                  "value" : "2p1c"
                }
              ],
              "validation": "oneof"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__dropdownMultiple__", 
              "metadata" : {
                "guiType": "dropdown",
                "name": "Dance Time",
                "description" : "To which songs would you dance in the kitchen?"
              },
              "options": [
                {
                  "value" : "Please choose multiple",
                  "disabled" : true
                },
                {
                  "text" : "Last Christmas (aka the one that drives everybody else crazy)",
                  "value" : "Last Christmas",
                  "selected" : true
                },
                {
                  "value" : "1pnc"
                },
                {
                  "value" : "1pncmin"
                },
                {
                  "value" : "2p", 
                  "selected" : true
                },
                {
                  "value" : "2p1c", 
                  "disabled" : true
                }
              ], 
              "validation": "anyof"
            }, 
            {
              "mode" : "fixed",
              "identifier" : "__toggle__", 
              "metadata" : {
                "guiType": "toggle",
                "name": "NO!",
                "description" : "What do you dislike?"
              },
              "options": [
                {
                  "value" : "verbose",
                  "selected" : true
                },
                {
                  "text" : "All kinds of Bugs (also the ones living in your Computer)",
                  "value" : "All kinds of Bugs"
                },
                {
                  "value" : "make_plot"
                }
              ], 
              "validation": "anyof"
            }, 
            {
              "mode" : "any",
              "identifier" : "__sliderMultiple__", 
              "metadata" : {
                "guiType" : "slider",
                "name": "temperature",
                "vertical": true,
                "description" : "Choose three random numbers to be output by the container"
              },
              "default": [
                25,
                50,
                75
              ],
              "min": 0,
              "max": 100,
              "step": 5,
              "validation": "range"
            },
            {
              "mode" : "any",
              "identifier" : "__inputTextWOMaxlength__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "text",
                "name": "name",
                "description" : "Enter your name"
              },
              "default" : [""],
              "validation": "pattern"
            },
            {
              "mode" : "any",
              "identifier" : "__inputTextWMaxlength__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "text",
                "name": "Christmas Wish",
                "description" : "Enter what you wish for at christmas"
              },
              "maxlength": 200,
              "default" : [""],
              "validation": "pattern"
            },
            {
              "mode" : "any",
              "identifier" : "__inputNumber__", 
              "metadata" : {
                "guiType" : "input_field",
                "type": "number",
                "name": "Age",
                "description" : "Enter your current age"
              },
              "default": [10],
              "min": 0,
              "max": 500,
              "step": 0.1,
              "validation": "range"
            }
          ],
          "content"   : "W2Fib3V0IHlvdV0NCmxpa2VkVGhpbmdzPXt7X19jaGVja2JveF9ffX0NCmZhdm9yaXRlUEw9e3tfX3JhZGlvQnV0dG9uX199fQ0KZnJpZGdlPXt7X19kcm9wZG93blNpbmdsZV9ffX0NCmRhbmNpbmc9e3tfX2Ryb3Bkb3duTXVsdGlwbGVfX319DQpkaXNsaWtlZFRoaW5ncz17e19fdG9nZ2xlX199fQ0KcmFuZG9tTnVtYmVycz17e19fc2xpZGVyTXVsdGlwbGVfX319DQpuYW1lPXt7X19pbnB1dFRleHRXT01heGxlbmd0aF9ffX0NCmNocmlzdG1hc1dpc2g9e3tfX2lucHV0VGV4dFdNYXhsZW5ndGhfX319DQphZ2U9e3tfX2lucHV0TnVtYmVyX199fQ"
          // decoded:
          // [about you]
          // likedThings={{__checkbox__}}
          // favoritePL={{__radioButton__}}
          // fridge={{__dropdownSingle__}}
          // dancing={{__dropdownMultiple__}}
          // dislikedThings={{__toggle__}}
          // randomNumbers={{__sliderMultiple__}}
          // name={{__inputTextWOMaxlength__}}
          // christmasWish={{__inputTextWMaxlength__}}
          // age={{__inputNumber__}}
        }
      ]
    },
    { 
      "identifier": "22483f42-95bf-984a-98a5-ee9485c85c31", 
      "path"      : "code.json",                              
      "metadata"  : 
        {  
          "syntaxHighlighting": "json"                   
        },
      "parts" : 
      [
        {
          "identifier": "f3fc4404-3529-4962-b252-47bc4ddd02a2",
          "access": "template",
          "metadata": {
            "name": "Parameter in part"
          },
          "parameters" : 
          [
            {
              "mode" : "any",
              "identifier" : "__default__", 
              "metadata" : {
                "guiType" : "editor", 
                "name": "code",
                "description" : "Enter some code"
              },
              "default": ["aW50IG1haW4oaW50IGFyZ2MsIGNoYXIgKiphcmd2KSB7IA0KLy8gUHJpbnQgJ0hlbGxvIFdvcmxkJyANCn0"],
              // decoded: 
              // {
              // "codeSnippet" : "{{__default__}}"
              // }
              "validation": "none"
            }
          ],
          "content": "e3t7X19kZWZhdWx0X199fX0"
          // decoded: 
          // {{{__default__}}}
        }
      ]
    }
  ], 
  "configuration" :
    { "resources.image"  : "name://viplab/viplab-example-image",
      "resources.volume" : "/data/shared",
      "resources.memory" : "1g",
      "resources.numCPUs" : 1
    }
}
```

## Explaining the JSON Format

*version 3.0.0*

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | AS |
|---------------|----------------------------------------|-----------|------------|--------|----|
|identifier | string (UUID) | must | the identifier of this computation template | can be generated by the frontend launcher | |
|version | string | opt | version of the json specification | should be given for backwards compatibility |
|metadata | struct | opt | contains information for frontend | See [metadata json object](#metadata-json-object) for details about the object | |
|environment |one of \{ "C", "C++", "Java", "Matlab", "Octave", "Container", "DuMuX" \} | must | Specifies the environment used for the Computation. It defines language, runtime, libraries and tools | | |
|files | [ {...}, {...}, ... ] |must |array containing [File objects](#json-objects-in-files): there has to be at least one element | | |
|parameters | [{...}, {...}, ...] | opt | Parameters can be used to supply values at runtime to the configuration. Each parameter has a unique identifier (as string) and is a [Parameter object](#json-object-parameter). | For security reasons free text *guiType*, i.e., text input field or editor, are not allowed here. **The only allowed parameters are those, that have mode set to fixed** |
|configuration | struct |opt/must (depends on environment) | Environment specific configurations | Different phases can be configured like compiling, checking (for legal function calls in source code), ... | 
|configuration --compiling.sources | [FILE_ID, FILE_ID, ...] | must | Array of identifiers of [JSON File objects](#json-objects-in-files). Explicit compilation (only referenced sources will be compiled). | for **C, C++, Java**; The frontend should suggest defaults here, e.g. by suited file suffix ('.c', '.cpp', '.java'). |  name/path (Java) for implicit compiling? check backend?|
|configuration --compiling.compiler |string |must |compiler to be used, e.g. "gcc" | for **C, C++** | |
|configuration --compiling.flags |string |must for **C, C++**; optional for **Java** |CFLAGS for **C/C++**; compile flags for **Java** |e.g. "gcc" for **C/C++**; "-O2" or "" for **Java** | |
|configuration --checking.sources |[PART_ID, PART_ID, ...] |must if checking should be performed |array of identifiers of to be checked sources; given by parts[]{PART_ID} (see [below](#json-objects-in-parts)) | for **Matlab, Octave, C, Java**; frontend should suggest all "modifiable" and "template" parts here |  |
|configuration --checking.allowedCalls |string |must if checking should be performed |for **Matlab/Octave**: *allowed* call names separated by WS; only idents (no braces, no func args)<br>for **Java**: *allowed* call name expressions separated by WS | for **Java** semantics see [Java checking semantics](#java-checking-semantics). | |
|configuration --checking.forbiddenCalls |string |must if checking should be performed |for **C**: *forbidden* call names separated by WS; only idents (no braces, no func args)<br>for **Java**: *forbidden* call name expressions separated by WS | for **C** semantics see [C checking semantics](#c-checking-semantics)); for **Java** semantics see [Java checking semantics](#java-checking-semantics).  | |
|configuration --linking.flags |string |must |LFLAGS | for **C, C++**; e.g. "" | |
|configuration --running.stdinFilename | FILE_ID | must | the file identifier that is passed to **Matlab/Octave** via standard-in | |
|configuration --running.timelimitInSeconds |int |opt |CPU time limit | for all **environments**; for semantics see [Notes to "timelimitInSeconds"](#notes-to-timelimitinseconds). | |
|configuration --running.commandLineArguments |string |opt | for **C, C++, Java**: arguments given to main() function; for **DuMuX, Container**: additional command line args | mustache template syntax can be used to transform input values (e.g. of sliders) into CLI arguments (see *parameters*-attribute) | |
|configuration --running.flags |string |opt |flags given to JVM | for **Java** | |
|configuration --running.mainClass |string |opt (if unique) / must (if not unique) |class containing "public static void main(String[] args) {}" | for **Java**: "args" may be another name. If main function is unique, its correct class should be detected automatically.| |
|configuration --running.executable |string |must |name of executable to run (as in backend file system)| for **DuMuX** | |
|configuration --running.entrypoint | string | opt | executable to run inside the container | for **Container**; can contain mustache template syntax for injecting PARAM_IDs (see *parameters*-attribute) |
|configuration --running.intermediateFilesPattern | [ REGEX, REGEX, ... ] | opt | regex-expression in stdout which file is ready to be transferred as intermediate result. The Regex has to return the complete filename as matching group 1 | for **Container**|
|configuration --running.userId | int | opt | user id of the user that writes files inside the container | for **Container**; needed to set correct permissions |
|configuration --resources.image | url | must | location of the image to be executed | for **Container**. The content can have the following prefixes: <ul><li>file:// stands for a local tar-file on the host viplab is running on (good for local testing)</li><li>name:// has to be used for an image that is in a publically reachable docker-registry, e.g. dockerhub</li><li>id:// is for an image-id (sha) of the image (mostly image is already in local image-list and you do not want docker to pull it again - good for local testing)</li><li>http:// is a "normal" url to a tar-file, e.g. a doi or a s3-link</li></ul> |
|configuration --resources.volume | string | opt | path in the container where data is placed | for **Container** | we need workaround for kata containers?! |
|configuration --resources.memory | string | opt (*64mb*)| memory limit for the container | for **Container** | |
|configuration --resources.numCPUs | int | opt | number of CPUs for the container | for **Container** | default?; kubernetes map to softlimit cpu-shares...|
|configuration --resources.diskSpace | int | opt | Disk space limit for the container | for **Container** | |

### metadata JSON object
|Key | Type |Opt / Must |Description |Comment | AS |
|---------------|----------------------------------------|-----------|------------|--------|----|
|displayName | string | opt | name of computation template shown in frontend | | Makes 'opt' sense for frontend? |
|description |string (plain text) |opt |short description |Mostly a few lines. A longer description in different formats can be put in "elements" (see below). | There is nothing 'below'. Where are descriptions actually used in the frontend? |
|output | struct | opt | Gives information about the output to the Frontend | See [output json object](#output-json-object) for details on contained objects | |

#### output JSON object
|Key | Type |Opt / Must |Description |Comment |
|---------------|----------------------------------------|-----------|------------|--------|
|viewer | list containing one or more of \{ "Image", "ParaView", "ViPLabGraphics", "CSV" \} | opt | When given, specific file extension, like ".vtu" are interpreted by the frontend for displaying results. Otherwise files are only downloadable. | |
|csv | [ {...}, {...}, ... ] | opt | Details for displaying csv-files | See [csv json object](#csv-json-object) for details on contained objects |
|vtk | [ {...}, {...}, ... ] | opt | Details for displaying vtk-files | See [vtk json object](#vtk-json-object) for details on contained objects |

##### csv JSON object
|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | AS |
|---------------|----------------------------------------|-----------|------------|--------|----|
|basename | string | opt | Basename defines connected files: Path of file begins with basename | | |
|xlabel | struct | opt | Define labels for csv-files | | |
|xlabel --key | string | opt | key for x-axis given in csv header | | |
|xlabel --label | string | opt | x-axis label for diagram | | |
|xlabel --factor | string | opt | Multiply x-values with this factor | | |
|xlabel --format | string | opt | Format the x-axis values according to format string | For information on the avaliable formating take a look at the info given by [Plotly](https://github.com/d3/d3-format/blob/main/README.md#locale_format); Example: To format a number to have two decimals, use "0.2f" | |
|plots | struct | opt | Define datasets: Provide y-axis labels for y-key given in csv | | For each dataset one diagram is rendered in the result |
|plots --key | string | opt | key for y-axis given in csv header | | |
|plots --label | string | opt | y-axis label for diagram | | |
|xlabel --factor | string | opt | Multiply y-values with this factor | | |
|plots --format | string | opt | Format the y-axis values according to format string | For information on the avaliable formating take a look at the info given by [Plotly](https://github.com/d3/d3-format/blob/main/README.md#locale_format); Example: To format a number to have two decimals, use "0.2f" | |

##### vtk JSON object
|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | AS |
|---------------|----------------------------------------|-----------|------------|--------|----|
|basename | string | opt | Basename defines connected files: Path of file begins with basename | | |

### C checking semantics
08.05.2013: Semantics is "mergeAndInclude" for both prod systems and devel system ("element" unused).

 * **"mergeAndInclude"** (default): Checked will be the merge result of elements after being preprocessed by the C preprocessor, so #include's are allowed and honored. Types have to be defined by the corresponding system headers (e.g. by `#include <stdio.h>`) and no predefined standard types should be expected. It's possible to have illegal function calls in source elements (being part of some merge) **not** being checked (typically teacher code). This means, that during checking it has to and will be looked, in which source element an illegal function call happens, for knowing, if this actually is an error case.
 * *__"element"__* (unused): Each element has to contain correct C code for itself. In addition to basic types some standard types defined in system headers are predefined (e.g. FILE, size_t). Using types from the outside -- e.g. other elements defining them or #include's in teacher code elements -- does **not** work.

Common for both: no preprocessor commands are allowed in to be checked elements, with only one exception: #include's are allowed in case of "mergeAndInclude". The reason for the latter is to give students access to source elements serving as headers - editable for them or not. Functions from system headers could be #include'd, too.

The teacher has the responsibility to put unwished system calls like system() and others from libc into "forbiddenCalls".
Another point of control is to avoid linking with libs, whose functions shouldn't be used (this does not work with libc (automatically linked)).


### Java checking semantics
```
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
```


### Notes to "timelimitInSeconds"
There is a default CPU time limit (see RLIMIT_CPU of 'man setrlimit') for running or interpreting, which depends on backend configuration. This is good for terminating non-terminating programs, e.g. endless loops.

Optional attribute "timelimitInSeconds" may lower this default CPU time limit; if it is higher than default, it will be ignored.

Setting it to a value as much as possible below default is good for backend response time under high load; especially, if default is configured for performing expensive computations.


 
### JSON objects in files

An object in array files[] has the following members:

|Key |Type (an enum default is marked by _italics_) |Opt / Must |Description|Comment
|----|----------------------------------------------|-----------|-----------|-------
|identifier | string (UUID) | must | for later referencing, has to be unique | can be autogenerated by frontend |
|path | string | must | path to file | Can be absolute (no "/") or relative. If relative, the path has to proceed from the value of "resources.volume" given in the configuration. Example: absolute - `/data/shared/input.txt`; relative - `input.txt` |
|metadata | struct | opt | contains information for frontend | See definition of [file metadata](#file-metadata-json-object) | |
|parts | [{...}, {...}, ...] | must | array containing [part objects](#json-objects-in-parts). There has to be at least one. |

### file metadata JSON object
|Key |Type (an enum default is marked by _italics_) |Opt / Must |Description|Comment
|----|----------------------------------------------|-----------|-----------|-------
|syntaxHighlighting | string (*text*) | opt | Mode of the ace editor. List can be found in on [github](https://github.com/ajaxorg/ace/tree/master/lib/ace/mode) | Examples: "ini", "c_cpp", "matlab", "java". See also [Ace demo](http://ajaxorg.github.io/ace-builds/kitchen-sink.html) | 
|description | string | opt | description of the file | |

### JSON objects in parts

An object in array parts[] has the following members:

|Key |Type (an enum default is marked by _italics_) |Opt / Must |Description|Comment|AS
|----|----------------------------------------------|-----------|-----------|-------|--
|identifier | string | must |for later referencing, has to be unique | can be autogenerated by frontend |
|access | one of \{ "invisible", "visible", "modifiable", "template" \} | must | defines the access level of this part for the user | see [Notes on access levels](#notes-on-access-levels-in-parts) for more details |
|metadata | struct | opt | contains information mainly for the frontend |
|metadata --name |string |opt | additional description of this part | To be shown in the frontend | Where? Is it used?
|parameters | array of parameter-objects | opt | definition of [parameters](#json-object-parameter) that are injected to *content* at runtime | Any number of parameters can be specified, but the PARAM_ID (identifier) has to be unique.
|content |string |must |base64url-encoded source code | Can contain [Handlebars.js](https://handlebarsjs.com/guide/) expressions with PARAM_IDs (identifiers) if the access type of this part is "template".

#### Notes on access levels in parts

Four access levels can be specified inside a part:

* **invisible**: The *content* is not shown to a user, i.e., student or re-user of a software. This can be used to hide irrelevant source code from the user, to focus on the important parts, etc...
* **visible**: The *content* is shown to a user, but can not be changed by him/her.
* **modifiable**: The *content* is shown to a user and can be changed. This comprise functions students should implement or input files of a research software that can be changed by a re-user.
* **template**: The *content* is shown to a user, but can not be changed. Additionally, GUI elements like input fields, sliders, buttons, etc. that are specified within the *metadata* can be set by the user and the template is then filled with these parameter values. This access-level can be used to simplify complex research software configurations for the re-user. 

### JSON object Parameter

A parameter-object, has the following members:

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | 
|---------------|----------------------------------------|-----------|------------|--------|
|mode | one of \{ "any", "fixed" \} | must | specifies type of the parameter | used to define the type of validation that is performed |
|identifier | string | must | unique id for this parameter | This id must be valid mustach template variable. Example: "*\_\_BINARY\_\_*" | 

#### fixed-type parameter JSON object

A fixed-type PARAM-object, like *\_\_checkbox\_\_*, has the following members:

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | 
|---------------|----------------------------------------|-----------|------------|--------|
|metadata | Object | must | JSON object containing information how to render this parameter | See definition of [fixed-type JSON object Parameter-Metadata](#fixed-type-json-object-parameter-metadata) |
|options | array of objects | must for *gui_type* "checkbox", "radio", "dropdown", "toggle" | specifies the allowed values | See [fixed-type options JSON object](#fixed-type-options-json-object) for details on contained objects|
|validation | one of \{ "oneof", "minone", "anyof" \} | must | See [Parameter validation semantics](#parameter-validation-semantics) for details | |

#### fixed-type JSON object Parameter-Metadata

A metadata-object, has the following members:

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | 
|---------------|----------------------------------------|-----------|------------|--------|
| guiType | one of \{ "checkbox", "radio", "dropdown", "toggle" \} | must | specifies how the frontend renders the parameter | | 
| name | string | must | Label for the parameter | frontend feature |
| description | string | must | description of the parameter | frontend feature |

#### fixed-type options JSON object

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | 
|---------------|----------------------------------------|-----------|------------|--------|
|value | string | must | specifies one avaliable value | Example: \{ "value" : "verbose" \} |
|text | string | opt | Text shown besides or as dropdown of the value | |
  |disabled | boolean | opt | Shows disabled options in frontend | Example: \{ "value" : "Please choose multiple", "disabled" : true \} |
|selected | boolean | opt | specifies defaults value/values for frontend | the strings have to be part of *values*; for "toogle" given values mean *true* |
| description | string | must | description of the parameter | frontend feature |


#### any-type parameter JSON object

A any-type PARAM-object, like *\_\_sliderMultiple\_\_*, has the following members:

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | 
|---------------|----------------------------------------|-----------|------------|--------|
|metadata | Object | must | JSON object containing information how to render this parameter | See definition of [any-type JSON object Parameter-Metadata](#any-type-json-object-parameter-metadata) |
|default | array of number(s) or base64url-encoded string(s) | opt | the default value(s) shown in frontend | |
|min | number | opt | minimal allowed value | for slider, or input_field with type number |
|max | number | opt | maximal allowed value | for slider, or input_field with type number |
|step | number | opt | defines together with *min* and *max* attributes a finite set of allowed values | for slider, or input_field with type number |
|maxlength | number | opt | Specifies for *gui_type* "input_field" the length of the input ||
|validation | one of \{ "range", "pattern", "none" \} | must | See [Parameter validation semantics](#parameter-validation-semantics) for details | |
|pattern | string | opt | A regex pattern for validation | |

#### any-type JSON object Parameter-Metadata

A metadata-object, has the following members:

|Key [--Subkey] | Type (a default is marked by _italics_)|Opt / Must |Description |Comment | 
|---------------|----------------------------------------|-----------|------------|--------|
| guiType | one of {*"editor"*, "input_field", "slider"} | must | specifies how the frontend renders the parameter | |
| type | one of \{ "number", "text" \} | opt | Type of the input field |  | 
| name | string | must | Label for the parameter | frontend feature |
| vertical | boolean | opt (*false*) | Specifies for *gui_type* "slider" whether it is rendered horizontal or vertical |

#### Parameter Validation Semantics

Four types of validation are implemented at the moment:

* **oneof**: Only one value can be chosen. The value has to be included in *options* and *disabled* for the *value* has to be set to false (is false by default, so *disabled* can also be missing).
* **minone**: One or more values can be chosen. The values have to be included in *options* and *disabled* for the *value*s has to be set to false (is false by default, so *disabled* can also be missing).
* **anyof**: All of the chosen values have to be included in *options* and *disabled* for the *value*s has to be set to false (is false by default, so *disabled* can also be missing).
* **range**: A numerical value is checked whether is is between *min* and *max*. If *step* is given a finite number of possible values is computed and the value has to be within this set.
* **pattern**: A regex pattern that the text value has to fulfill.
* **none**: If no validation is necessary, because there are no restraints on the value.