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

### Parameters: What is input into the Handlebars.js-Template?

<table>
  <tr>
    <th>Parameter guitype</th>
    <th>Returns</th>
  </tr>
  <tr>
    <td>checkbox</td>
    <td>
      ``` json title="Checkbox Example"
      {
        "mode" : "fixed",
        "identifier" : "__checkbox__", 
        "metadata" : {
          "guiType": "checkbox",
          "name": "Things I like",
          "decription" : "Select things you like"
        },
        "options": [
          {
            "value" : "programming",
            "selected" : true
          },
          {
            "value" : "music"
            "selected" : true
          },
          {
            "value" : "books"
          }
        ],
        "validation": "anyof"
      } 
      ```
      ``` title="Checkbox Returns"
      ["programming", "music"] 
      ```
    </td>
    <td>
      Array of Strings
    </td>
  </tr>
  <tr>
    <td>radio</td>
    <td>
      ``` json title="Radio Example"
      {
        "mode" : "fixed",
        "identifier" : "__radioButton__", 
        "metadata" : {
          "guiType": "radio",
          "name": "Favorite PL",
          "decription" : "Select your favorite programming language",
        },
        "options": [
          {
            "value" : "C"
          },
          {
            "value" : "Java",
            "selected" : true
          },
          {
            "value" : "Haskell",
            "disabled" : true
          },
          {
            "value" : "Python"
          }
        ],
        "validation": "oneof"
      }
      ```
      ``` title="Radio Returns"
      "Java"
      ```
    </td>
    <td>
      String
    </td>
  </tr>
  <tr>
    <td>dropdown (single)</td>
    <td>
      ``` json title="Dropdown (single) Example"
      {
        "mode" : "fixed",
        "identifier" : "__dropdownSingle__", 
        "metadata" : {
          "guiType": "dropdown",
          "name": "Fridge",
          "decription" : "How often do look into the fridge a day?"
        },
        "options": [
          {
            "value" : "Please choose one",
            "disabled" : true
          },
          {
            "value" : "never",
            "selected" : true
          },
          {
            "value" : "Once a day"
          },
          {
            "value" : "Twice a day"
          },
          {
            "value" : "Three times a day"
          },
          {
            "value" : "More than three times a day"
          }
        ],
        "validation": "oneof"
      }
      ```
      ``` title=" Dropdown (single) Returns"
      "never"
      ```
    </td>
    <td>
      String
    </td>
  </tr>
  <tr>
    <td>dropdown (multiple)</td>
    <td>
      ``` json title="Dropdown (multiple) Example"
      {
        "mode" : "fixed",
        "identifier" : "__dropdownMultiple__", 
        "metadata" : {
          "guiType": "dropdown",
          "name": "Dance Time",
          "decription" : "To which songs would you dance in the kitchen?"
        },
        "options": [
          {
            "value" : "Please choose multiple",
            "disabled" : true
          },
          {
            "value" : "Last Christmas",
            "selected" : true
          },
          {
            "value" : "White Christmas"
          },
          {
            "value" : "Winter Woderland"
          },
          {
            "value" : "Thats Christmas To Me", 
            "selected" : true
          },
          {
            "value" : "O Come All Ye Faithful", 
            "disabled" : true
          }
        ], 
        "validation": "anyof"
      }
      ```
      ``` title="Dropdown (multiple) Result"
      ["Last Christmas", "Thats Christmas To Me"]
      ```
    </td>
    <td>
      Array of Strings
    </td>
  </tr>
  <tr>
    <td>toggle</td>
    <td>
      ``` json title="Toggle Example"
      {
        "mode" : "fixed",
        "identifier" : "__toggle__", 
        "metadata" : {
          "guiType": "toggle",
          "name": "NO!",
          "decription" : "What do you dislike?"
        },
        "options": [
          {
            "value" : "Spiders",
            "selected" : true
          },
          {
            "value" : "All kinds of Bugs (also the ones living in your Computer)"
          },
          {
            "value" : "I never dislike anything!"
          }
        ], 
        "validation": "anyof"
      }
      ```
      ``` title="Toggle Result"
      ["Spiders"]
      ```
    </td>
    <td>
      Array of Strings
    </td>
  </tr>
  <tr>
    <td>slider (single)</td>
    <td>
      ``` json title="Slider (single) Example"
      {
        "mode" : "any",
        "identifier" : "__sliderSingle__", 
        "metadata" : {
          "guiType" : "slider",
          "name": "Temperature",
          "vertical": false,
          "decription" : "How hot do you like your coffee? (in degrees Celsius)"
        },
        "default": [
          75
        ],
        "min": 0,
        "max": 90,
        "step": 10,
        "validation": "range"
      }
      ```
      ``` title="Slider (single) Result"
      75
      ```
    </td>
    <td>
      Number
    </td>
  </tr>
  <tr>
    <td>slider (multiple)</td>
    <td>
      ``` json title="Slider (multiple) Example"
      {
        "mode" : "any",
        "identifier" : "__sliderMultiple__", 
        "metadata" : {
          "guiType" : "slider",
          "name": "random numbers",
          "vertical": true,
          "decription" : "Choose three random numbers to be output by the container"
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
      }
      ```
      ``` title="Slider (multiple) Result"
      [25, 50, 75]
      ```
    </td>
    <td>
      Array of Numbers
    </td>
  </tr>
  <tr>
    <td>input_field</td>
    <td>
      ``` json title="Input_field Example"
      {
        "mode" : "any",
        "identifier" : "__inputText__", 
        "metadata" : {
          "guiType" : "input_field",
          "type": "text",
          "name": "name",
          "decription" : "Enter your name"
        },
        "default" : [""],
        "validation": "none"
      }
      ```
      ``` title="Input_field Result"
      "base64:S2F0aHJ5bg"
      ```
    </td>
    <td>
      If the input_field is of type number: Number
      Else: Prefixed ("base64:") and Base64URL-encoded String - is decoded by Websocket API, before it is sent to Backend
    </td>
  </tr>
  <tr>
    <td>editor</td>
    <td>
      ``` json title="Editor Example"
      {
        "mode" : "any",
        "identifier" : "__default__", 
        "metadata" : {
          "guiType" : "editor", 
          "name": "code",
          "decription" : "Enter some code"
        },
        "default": ["aW50IG1haW4oaW50IGFyZ2MsIGNoYXIgKiphcmd2KSB7IC8vIFByaW50ICdIZWxsbyBXb3JsZCcgfQ"],
        "validation": "none"
      }
      ```
      ``` title="Editor Result"
      "base64:aW50IG1haW4oaW50IGFyZ2MsIGNoYXIgKiphcmd2KSB7IC8vIFByaW50ICdIZWxsbyBXb3JsZCcgfQ"
      ```
    </td>
    <td>
      Prefixed ("base64:") and Base64URL-encoded String - is decoded by Websocket API, before it is sent to Backend
    </td>
  </tr>
</table>

## Evaluation Task

The feature of ViPLab to automatically evaluate student code (correction service)
is still possible in ViPLab 3.0. The code needed for evaluation is send to the 
correction server as a JSON message with the same structure as a *Computation Task*.
The only difference is, that here all *parts* can be replaced, regardless of there
*access*-settings.
