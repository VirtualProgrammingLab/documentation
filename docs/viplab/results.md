# /numlab/results

Message queue.

[Result JSON Format](#result-json-format) uses [Solution JSON Format](/viplab/solutions#solution-json-format).

## Interface

POST, GET ([see ''English section name unknown so far.''](/ecs2#message-resource))

### Generic

* __POST resourceURL (push)__ creates Result.
* __POST resourceURL/fifo (pull)__ gets first Result and removes it from the queue.
* __GET resourceURL/fifo (look)__ gets first Result (fifo) without removing it from the queue.

For getting Results in sensible time, clients have to poll the queue continuously.

### ViPLab Specific

POST resourceURL (push) will be used by CCs, POST resourceURL/fifo (pull) by SCs. GET may be used by SCs (but it shouldn't) or for debugging purposes.

Computation clients (CCs) push computation Results into this queue; a student client (SC) pulls them from there (logically each SC has its own queue separated from others). For getting next Result an SC has to pull the former.

## Message
### Result JSON Definition by Example (informal) 
```
{
    "Result" : {
        "ID" : "one chunk 2012-07-25 17:59:50",
        "status" : "final",
        "computation" : {
            "startTime" : "2012-07-25 17:59:49",
            "duration" : "555ms",
            "finishTime" : "2012-07-25 17:59:50",
            "CC_versionLong" : "[ViP CCv2.16 (C chain)v1.9 (Matlab chain)v2.6.1 (Octave chain)v1.4.1 (DuMuX chain)v1.2 (C++ chain)v1.3 (Tue Jun 26 19:10:42 CEST 2012)]",
            "CC_version" : "ViP CCv2.16",
            "chain_version" : "(C chain)v1.9",
            "technicalInfo" : {
                "host" : "numlab",
                "PID" : "6405",
                "ID" : "#1"
            },
            "userInfo" : {
                "summary" : "(C chain)v1.9 failed.",
                "elements" : [
                    {
                        "severity" : "info",
                        "type" : "compiler",
                        "message" : "source_0.c: In function \u2018trapez\u2019:",
                        "output" : {
                            "elementID" : "compile_stderr_0.txt",
                            "extract" : "source_0.c: In function \u2018trapez\u2019:",
                            "begin" : 0,
                            "end" : 37
                        }
                    },
                    {
                        "severity" : "warning",
                        "type" : "compiler",
                        "message" : "source_0.c:21:10: warning: unused variable \u2018x\u2019 [-Wunused-variable]",
                        "source" : {
                            "elementID" : "codeFromStudent",
                            "extract" : "  double x = 0;",
                            "begin" : 406,
                            "end" : 421,
                            "line" : 15,
                            "col" : 10
                        },
                        "output" : {
                            "elementID" : "compile_stderr_0.txt",
                            "extract" : "source_0.c:21:10: warning: unused variable \u2018x\u2019 [-Wunused-variable]",
                            "begin" : 38,
                            "end" : 108
                        }
                    },
                    {
                        "severity" : "error",
                        "type" : "callcheck",
                        "message" : "[C function filtering] Function call not allowed:\n\"system\"; original source: codeFromStudent, line (corrected): 20, col: 3\nForbidden calls:\nsystem.\n",
                        "source" : {
                            "elementID" : "codeFromStudent",
                            "extract" : "  system(\"/bin/rm /tmp/foo.txt\"); // illegal call (should be catched by checker)",
                            "begin" : 325,
                            "end" : 405,
                            "line" : 14,
                            "col" : 3
                        }
                    }
                ]
            }
        },
        "Solution" : {
            "postTime" : "2009-09-20T04:15:01.22Z",
            "ID" : "[with illegal call] 127.32.15.3#127",
            "comment" : "Die war aber schwer!",
            "exercise" : "https://nfldevvipecs.rus.uni-stuttgart.de/numlab/exercises/442641",
            "exerciseModifications" : {
                "elements" : [
                    {
                        "identifier" : "codeFromStudent",
                        "value" : "typedef double (funcPtr)(double); // type of function to be integrated\n\n/* Definition der zu integrierenden Funktion */\ndouble f(double x)\n{\n  return x * x; // Beispiel\n}\n\ndouble trapez(double a, double b, int n, funcPtr f)\n{\n  double sum = 0; // init\n  /* Ab hier kommt die Funktionalitaet der integrierenden Funktion.. */\n\n  system(\"/bin/rm /tmp/foo.txt\"); // illegal call (should be catched by checker)\n  double x = 0;\n  int k;\n  if (n < 1) return 0; // avoid endless recursion\n\n  // untere Grenze\n  sum+=f(a)/2;\n  // Schleife ueber die inneren Punkte\n  for(k=1; k < n; k++)\n  {\n    sum+=f(a+(b-a)*k/n);\n  }\n  // obere Grenze\n  sum+=f(b)/2;\n  // Skalieren\n  sum*=(b-a)/n;\n\n  return sum; // result\n}\n"
                    }
                ]
            }
        },
        "elements" : [
        ]
    }
}
```

### Result JSON Format

wrapper "Result" around the following

|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
|ID                   |string     |must | unique Result ID created by CC (automatically) | For tracking and debugging purposes. |
|comment              |string     |opt              | comment from CC | |
|status | One of {"final", "intermediate"} |must | "final": last Result of computation<BR>"intermediate": more Results expected | If there is only one Result, it status should be "final". If there are multiple Results, status of all before "final" should be "intermediate".|
|index |uint |- must: if there are multiple Results<BR>- opt: if there is only one "final" Result | counts starting from 0 | Note: "final" Result gets last index. |
|computation --startTime  | datetimeString | must | May be local or universal time. | Just after getting a Solution.|
|computation --duration   | durationString | must | e.g. "128ms" | Computation duration time since startTime (increases with each intermediate Result). |
|computation --finishTime | datetimeString | must, iff status == "final" | May be local or universal time. | Just before posting final Result.|
|computation --CC_versionLong | string | must | | |
|computation --CC_version  | string | must | | |
|computation --chain_version  | string | must | | |
|computation --technicalInfo | struct | must | | |
|computation --technicalInfo --host | string | must | CC host | |
|computation --technicalInfo --PID | unsigned | must | CC PID | |
|computation --technicalInfo --ID | string | must | | Info for finding log files (may be number of computations). |
|computation --technicalInfo --&lt;key&gt; | &lt;val_type&gt; | opt | &lt;key&gt;: locally unique, &lt;val_type&gt;: arbitrary type | For allowing further key/val pairs if there should be a need.|
|computation --userInfo  | struct | must | see [Result userInfo object spec](#json-object-in-result-userinfo) | |
|Solution             |struct     |must             | exact copy of Solution ([Solution JSON Format](/viplab/solutions#solution-json-format)) which this Result has been computed for| Having Solution--ID is important for SC: think of posting multiple different Solutions with waiting for different Results. And having all *dynamic* data together is good!|
|elements             |[ {...}, {...}, ... ] |must |array of objects containing elements described in the following: there has to be at least one of them | |

### JSON objects in Result--elements[]
An object in (array) elements[] has the following properties:

|**Key** |**Value Type** (an enum default is marked by *italics*) |**Opt / Must** |**Description** |**Comment** |
|---|----|---|---|---|
|identifier |string |must |for later referencing, has to be unique | Technical: e.g. it could be created by using a filename (of a file being source of value property).|
|name |string |opt |name, need **not** be unique | User info: may be shown in SC.|
|MIMEtype | One of {*"text/plain"*, "text/html", "text/uri-list", "image/png", "application/x-vgf", "application/x-vgf3", "application/x-vgfc"} |opt |info for interpreting value property| - "text/uri-list": links to websides to be presented in SC;<BR>- "image/png": encoding unclear so far;<BR>- "application/x-vgf": generated graphics output in 'ViP graphics format';<BR>- "application/x-vgf3": generated graphics output in 'ViP graphics format 3D';<BR>- "application/x-vgfc": generated graphics output in 'ViP graphics format contour plot'.|
|emphasis | One of {"low", *"medium"*, "high"} |opt |info for rendering | |
|value |string |must |text or other content | |

Notes:
* There is a - mandatory - unique ID for objects in elements[] for referencing them later: this is especially useful, if the number of elements is variable.
* If an - optional - key with default value is missing, it should be interpreted the same as being there and set to its default.
* Advanced: a numOfFinalResults/finalResultNo mechanism could be of interest later: needed for sequencing computed Results (think of multiple Results for one Solution, which are computed distributed).

### JSON object in Result--userInfo
An userInfo object (struct) has the following properties:

|**Key** |**Value Type** | **Opt/Must** | **Description** | **Comment** | **AS** |
|---|---|---|---|---|---|
|summary | string | opt | summary of all info_element's: e.g. one warning, one error may have a summary "An error occured." | E.g.: "[Error] Backend has detected an error: no result!" or "Success!" |
|elements | [{...}, {...}, ...] | opt | array of *info_element*s | |
|info_element | struct | opt | JSON object inside *elements*| |
|info_element --severity | One of {"error", "warning", "info"} | must | | |
|info_element --type | One of {"system", "chain", "output", "callcheck", "interpreter", "compiler", "linker", "executable"} | must | For all chains: {"system, "chain", "output"}; for Octave/Matlab: {"callcheck", "interpreter"}; for C: {"callcheck", "compiler", "linker", "executable"}; for DuMuX: {"executable"} | If type equals "system", *whole* Result if of interest for a bug report. | Java?? |
|info_element --message | string | must | summary of one message to the user; it sould not contain wrong error locations (file, line, col). Together with "soruce"/"line" or "col" (containing corrected locations) it should give the most interesting info. | may be empty string (',' in optional location_part) |
|info_element --source | struct | opt | contains location information | only allowed if "type" in {"compiler", "interpreter", "callcheck"} |
|info_element --source --elementID | ELEMENT_ID (string) | must | identifier of element from Solution/Exercise elements with source *input*, which has triggered (error/warning/...) message |
|info_element --source --line |1..uint_max | must | Location of error/warning ... |
|info_element --source --col |1..uint_max | opt | ... referring to position in triggering input element. | It is optional, since there are messages containing line info only |
|info_element --output | struct | opt | for output of tool detecting something |
|info_element --output --elementID | ELEMENT_ID (string) | must | identifier of element from Result elements with interpreter/compiler *output*, which contains error/warning message |
|info_element --source/output --extract | string | must | Part of referenced element. Extracted from message triggering input. |
|info_element --source/output --begin | unsigned int | must | Offset of extract into ... |
|info_element --source/output --end | unsigned int | must | ... referenced element: C-like (zero-based). |

Notes:
* There are error messages containing multiple error triggering locations. This leads to multiple info_elements containing same "message" and "output", but different "source" objects. Note: another way would be to specify "sources":[{},...] as alternative to "source":{}.
* "type": "fatal" may be introduced later
* Compiler output may contain multiple error positions; only the first one should be extracted.
