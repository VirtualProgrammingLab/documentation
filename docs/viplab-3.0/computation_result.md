# Computation Result JSON Message

A *computation result*  is the result of a *computation*. It always contains
the stdout and stderr of of the computation. In addition, it may containIt contains 
arbitrary information contained in one or more artifacts. Artifacts
are information about the success or failure (showing details for students)
and result objects, like files, images, links, etc.

## Example (informal)

```
{
  "identifier" : "86165eea-14df-4a76-805a-09b21441cbf7",
  "version" : "3.0.0"
  "computation" : "4598393-95bf-409a-98a5-ee375982c3e",  // uuid of corresponding computation
  "status" : "final", // final and intermediate, 
  "timestamp" :   // creation time of this message in ISO-8601
  "output" :
    {
      "stdout" : "", // base64url encoded content, mandatory, even if empty
      "stderr" : "", // base64url encoded content, mandatory, even if empty
    },
  "artifacts" : [ // additional artifacts (files, notifications, etc)
    {
      "type": "notifications", // each artifact requires a type and
      "identifier" : "1dd479d9-c9bd-4711-aed8-5bfb3ec5fcfa", // and an identifier
      "summary" : "(C chain)v1.9 failed.",
      "elements" : [
        {
          "severity" : "info",
          "type" : "compiler",
          "message" : "source_0.c: In function \u2018trapez\u2019:",
          "output" : {
            "source" : "stderr",
            "extract" : "source_0.c: In function \u2018trapez\u2019:",
            "begin" : 0,
            "end" : 37
          }
        },
        {
          "severity" : "warning",
          "type" : "compiler",
          "message" : "source_0.c:21:10: warning: unused variable \u2018x\u2019 [-Wunused-variable]",
          "origin" : {
              "source" : "parts://7a1808d0-c997-4e28-acdf-bfda3ce70960", //code the student can edit
              "extract" : "  double x = 0;",
              "begin" : 406,
              "end" : 421,
              "line" : 15,
              "col" : 10
          },
          "output" : {
              "source" : "stderr",
              "extract" : "source_0.c:21:10: warning: unused variable \u2018x\u2019 [-Wunused-variable]",
              "begin" : 38,
              "end" : 108
          }
        },
        {
          "severity" : "error",
          "type" : "callcheck",
          "message" : "[C function filtering] Function call not allowed:\n\"system\"; original source: codeFromStudent, line (corrected): 20, col: 3\nForbidden calls:\nsystem.\n",
          "origin" : {
              "elementID" : "parts://7a1808d0-c997-4e28-acdf-bfda3ce70960",
              "extract" : "  system(\"/bin/rm /tmp/foo.txt\"); // illegal call (should be catched by checker)",
              "begin" : 325,
              "end" : 405,
              "line" : 14,
              "col" : 3
          }
        }
      ]
    },
    {
      "type" : "file",
      "identifier" : "de762095-6cd2-439f-80eb-313e85d33869",
      "MIMEtype": "image/png",
      "path" : "/images/img.png",
      "content": "" // base64url encoded content
    },
    {
      "type" : "file",
      "identifier" : "10516761-d937-4ba4-a82f-dc2847d45032",
      "MIMEtype": "image/png",
      "path" : "/images/img2.png",
      "content": "" // base64url encoded content
    },
    {
      "type" : "s3file",
      "identifier" : "cc3c1cf9-c02d-4694-902c-93c298d68c51",
      "MIMEtype": "application/gzip",
      "path" : "/largefile/result.tar.gz",
      "url": "https://s3.temporary.file.url/result.tar.gz",
      "size" : 123456789,
      "hash" : "sha512:hashcode_of_file"
    }
  ]
}
```
## Result Artifacts
Each computation ends with a ComputationResult containing at least the output generated on stdout and stderr. 

All additional information about an executed computation like created files is contained inside an arbitrary number of artifacts.
Each artifact containes at least an identifier and a type describing the content. The main purposes of this concept is to allow a 
standard way how to extend what information can be created and returned during a computation.

### artifact types
In addition to the two mandatory fields for each artifact, each artifact has a specific set of additional fields. Currently, the following list of 
artifact types are defined with a given set fields:

* file
* s3file
* notifications

## Specification of the JSON formats

### ComputationResult JSON object

|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| identifier    |string (uuid) |must       | unique ComputationResult ID  | |
| version | string | must | version of the json specification used. The major version must match the version of the ComputationTask | |
| computation |    string (uuid) |must              | identifier of the computation that is responsible for this result | |
| status | One of {"final", "intermediate"} |must | "final": last Result of computation<BR>"intermediate": more Results expected | If there is only one Result, it status should be "final". If there are multiple Results, status of all before "final" should be "intermediate".|
| timestamp |ISO-8601 datetimeString | must | Timestamp when this result is generated. Need for ordering of intermediate result |  |
| output          | output json object     |must             | an object containing stdout and stderr | |
| artifacts       | array of artifact objects |opt |array of objects containing artifacts described at [Artifacts](#artifact-json-object-format) | |

### output JSON object format

|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| stdout        | string    | must      | the stdout of the exection, base64url encode | must be provided even if empty |
| stderr        | string    | must      | the stderr of the exection, base64url encode | must be provided even if empty | 


### artifact JSON objects

The base artifact object that needs to be extended all other artifacts defined.

#### artifact JSON object
|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| identifier    | string (uuid) |must   | unique id for this artifact | |
| type          |  string   | must      | the type of this artifact | The type may be used by the client to determine how to display the artifact| 


#### file artifact JSON object
|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| path          | string    | must      | the path in which the content had been created during the computation | |
| MIMEtype      | string    | must      | MIMEtype of the content | |
| content       | string    | must      | base64url encoded content of the file | | 

In Viplab 2.0, the following MIMEtypes where allowed. They contain some custom defined types, which need to be keep supported:

* text/plain
* text/html
* text/uri-list": links to websides to be presented in SC
* image/png"
* application/x-vgf: generated graphics output in 'ViP graphics format'
* application/x-vgf3: generated graphics output in 'ViP graphics format 3D'
* application/x-vgfc: generated graphics output in 'ViP graphics format contour plot'.|


#### s3file artifact JSON object
|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| path          | string    | must      | the path in which the content had been created during the computation | |
| MIMEtype      | string    | must      | MIMEtype of the content | |
| url           | string    | must      | url on which the content of the file can be retrieved | | 
| size          | int       | must      | size in bytes of the file stored on the external system | |
| hash          | string    | must      | the hash of the remote file in the format usedhash:hashcode_of_file, e.g. sha512:a12355.... ||


#### notifications artifact JSON object
Each notifications artifact contains at least one summary and unlimited number of notification elemets providing more details

|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| summary | string | must | summary of all info_element's: e.g. one warning, one error may have a summary "An error occured." | E.g.: "[Error] Backend has detected an error: no result!" or "Success!" |
| notifications |  array of notification objects | opt | the more detailed notification objects for this notifications object| |


#### notification JSON object
|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| severity | string | must | One of {"error", "warning", "info"} | | |
| type | string | must | One of {"system", "chain", "output", "callcheck", "interpreter", "compiler", "linker", "executable"} | For all chains: {"system, "chain", "output"}; for Octave/Matlab: {"callcheck", "interpreter"}; for C: {"callcheck", "compiler", "linker", "executable"}; for Java: {"callcheck", "compiler", "executable"}; for DuMuX: {"executable"}  If type equals "system", *whole* Result if of interest for a bug report. |
| message | string | must | summary of one message to the user; it should not contain wrong error locations (file, line, col). Together with "source"/"line" or "col" (containing corrected locations) it should give the most interesting info. | may be empty string (',' in optional location_part) |
| origin  | notification origin json object | opt | If the message can be linked to a part from the ComputationTask, the original position can be found inside | only allowed if "type" in {"compiler", "interpreter", "callcheck"} |
| output | notification output json object | opt | The position and text from either stdout or stderr (see [Output](#output-json-object-format)) which caused notification.||


Notes:

* There are error messages containing multiple error triggering locations. This leads to multiple info_elements containing same "message" and "output", but different "source" objects. Note: another way would be to specify "sources":[{},...] as alternative to "source":{}.
* "type": "fatal" may be introduced later
* Compiler output may contain multiple error positions; only the first one should be extracted.


##### notification origin JSON object
|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
|source | string | must | identifier of part from [ComputationTask](computation_task.md) which has triggered (error/warning/...) message |
|extract | string | must | Part of referenced part. Extracted from message triggering input. |
|begin | unsigned int | must | Offset of extract into ... |
|end | unsigned int | must | ... referenced part: C-like (zero-based). |
|line |1..uint_max | must | Location of error/warning ... |
|col |1..uint_max | opt | ... referring to position in triggering part. | It is optional, since there are messages containing line info only |

##### notification output JSON object
|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
|source | string | must | part of [Output](#output-json-object-format), which contained the message |
|extract | string | must | Part of referenced output. Extracted from message triggering input. |
|begin | unsigned int | must | Offset of extract into ... |
|end | unsigned int | must | ... referenced element: C-like (zero-based). |


## Leftovers from Viplab 2.0

might be converted later, just keep around as inspration

|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
|startTime  | datetimeString | must | May be local or universal time. | Just after getting a Solution.|
|duration   | durationString | must | e.g. "128ms" | Computation duration time since startTime (increases with each intermediate Result). |
|finishTime | datetimeString | must, iff status == "final" | May be local or universal time. | Just before posting final Result.|
|CC_versionLong | string | must | | |
|CC_version  | string | must | | |
|chain_version  | string | must | | |
|technicalInfo | struct | must | | |
|technicalInfo/host | string | must | CC host | |
|technicalInfo/PID | unsigned | must | CC PID | |
|technicalInfo/&lt;key&gt; | &lt;val_type&gt; | opt | &lt;key&gt;: locally unique, &lt;val_type&gt;: arbitrary type | For allowing further key/val pairs if there should be a need.|

