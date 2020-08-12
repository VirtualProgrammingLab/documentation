# Computation Task JSON Message

A *computation task*  is the result of a possibly manipulated *computation template*.
It contains the changes made on the template.
Together with the template, a *computation task* forms a *computation* that is
sent to the backend.

In the context of a learning environment, a *computation task* can be seen as a
*Solution*.

## Example (informal)

```
{ "template" : "11483f23-95bf-424a-98a5-ee5868c85c3e", // uuid of corresponding computation template
  "arguments" : {
     "__STEPWIDTH__" : "0.5" // parameter values from template, filled out by frontend
  },
  "metadata" : {
      "comment" : "Die war aber schwer!",
  },
  "parts" : 
    [
      { "identifier": "codeFromStudent", // must: identifier of template part that has changed
        "content"   : "dm9pZCBiYXIoKSB7IHByaW50ZigiYmFyIQoiKTsKfQo" // changed source from user 
                                               // decoded: void bar() { printf(\"bar!\\n\");\n}\n
        // other fields defining this (text) element must *not* be repeated here.
      }
      // unchanged elements needn't be repeated here
    ]
}
```

## Explaining the JSON Format

|Key [--Subkey] |Type |Opt / Must |Description |Comment | AS |
|------|-----|-----|-----|----|----|
| template | string (UUID) | must | computation template identifier | | 
| arguments | {PARAM_ID: value, ..., PARAM_ID: value} | opt | arguments of, e.g., *configuration.commandLineArguments* extracted by the frontend | see [computation template](computation_template.md#explaining-the-json-format) |
| metadata | struct | opt | contains information for frontend | | |
| metadata --comment | string | opt | comment from user | | implemented in frontend?
| parts | [{...}, {...}, ...] | opt | array containing *modified* [part objects](computation_template.md#json-objects-in-parts). Only *identifier*- and *content*-attributes are allowed. Only "modifiable" or "template" parts can be referenced here (see *access*-attribute). | The frontend creates the content for "template" parts. See [notes](#notes-for-template-parts). |  


### Notes for template parts

The frontend will encode parts with *access*-value "template" as JSON messages with
the following structure:
```
{ "PARAM_ID1" : "value",
  "PARAM_ID2" : "value_2",
  ...
  "PARAM_IDn" : "value_n"
}
```
*PARAM_ID* is the reference (key) to a parameter name inside the mustache template.
"value" is the selected value of the user in the frontend.

## Evaluation Task

The feature of ViPLab to automatically evaluate student code (correction service)
is still possible in ViPLab 3.0. The code needed for evaluation is send to the 
correction server as a JSON message with the same structure as a *Computation Task*.
The only difference is, that here all *parts* can be replaced, regardless of there
*access*-settings.
