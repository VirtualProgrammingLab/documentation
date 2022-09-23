# Common Validation Errors

## Required Element is Missing

=== "Error"
    !!! error
        ```
        [
            {
                "keyword": "required",
                "dataPath": ".configuration",
                "schemaPath": "#/properties/configuration/required",
                "params": {
                "missingProperty": "resources.image"
                },
                "message": "should have required property 'resources.image'"
            }
        ]
        ```

=== "Solution"
    !!! success
        1. Click on the outer part (if empty - anywhere in the middle section; if there is files oder commandline arguements - click right next to them) in the middle section to open the Configuration of the Template
        2. Enter your Docker-Image

## Given Docker-Image has no URI-Schema

=== "Error"
    !!! error
        ```
        [
            {
                "keyword": "format",
                "dataPath": ".configuration['resources.image']",
                "schemaPath": "#/properties/configuration/properties/resources.image/format",
                "params": {
                "format": "uri"
                },
                "message": "should match format \"uri\""
            }
        ]
        ```

=== "Solution"
    !!! success
        Click on the outer part (if empty - anywhere in the middle section; if there is files oder commandline arguements - click right next to them) in the middle section to open the Configuration of the Template
        2. Enter your Docker-Image and make sure that is a URI

## File Path is not correct

=== "Error"
    !!! error
        ```
        [
            {
                "keyword": "pattern",
                "dataPath": ".files[0].path",
                "schemaPath": "#/properties/files/items/properties/path/pattern",
                "params": {
                "pattern": "^([A-z0-9-_+]+/?)*([A-z0-9]+([.]([A-z]+))?)$"
                },
                "message": "should match pattern \"^([A-z0-9-_+]+/?)*([A-z0-9]+([.]([A-z]+))?)$\""
            }
        ]
        ```

=== "Solution"
    !!! success
        1. Click on file in the middle section to open file configuration on the left
            - To select the correct file right away, take a look at the "dataPath" in the Error-Message
            - The number given behind "files" in the brackets + 1 is the file you have to click
        2. Modify the value of "Path to File" to match a relative path

## File has no Part(s)

=== "Error"
    !!! error
        ```
        [
            {
                "keyword": "minItems",
                "dataPath": ".files[0].parts",
                "schemaPath": "#/properties/files/items/properties/parts/minItems",
                "params": {
                "limit": 1
                },
                "message": "should NOT have fewer than 1 items"
            }
        ]
        ```

=== "Solution"
    !!! success
        1. Drag-and-Drop one or more part-items from the left to the middle-section
        2. Drop the part onto the file, that contains no part yet

## Missing Commandline Arguments
=== "Error"
    !!! error
        ```
        [
            {
                "keyword": "minItems",
                "dataPath": ".parameters",
                "schemaPath": "#/properties/parameters/minItems",
                "params": {
                "limit": 1
                },
                "message": "should NOT have fewer than 1 items"
            }
        ]
        ```

=== "Solution"
    !!! success
        1. Drag-and-Drop one or more of the parameter-items from the left to the middle-section
            - Choose one of:
                - checkbox
                - radio
                - dropdown
                - toggle
        2. Drop the items onto the empty commandline argument-item

## Missing Parameters
=== "Error"
    !!! error
        ```
        [
            {
                "keyword": "minItems",
                "dataPath": ".files[0].parts[0].parameters",
                "schemaPath": "#/properties/files/items/properties/parts/items/then/properties/parameters/minItems",
                "params": {
                "limit": 1
                },
                "message": "should NOT have fewer than 1 items"
            }
        ]
        ```

=== "Solution"
    !!! success
        1. Click on the part in the middle-section specified by the Error-Message
            - Take a look at the "dataPath" in the message
            - Choose the file + 1 and inside the file, part + 1
        2. Drag-and-Drop one of the parameter-items from the left to the specified part in the middle-section
            - Choose one of:
                - checkbox
                - radio
                - dropdown
                - toggle
                - input_field
                - editor
                - slider
        3. Drop the item onto the empty part-item       
            - After selecting the item to drop, the area(s) where it can be dropped will light up in green

## Missing Values to specify input_field

=== "Error"
    !!! error
        ```
        [
            {
                "keyword": "additionalProperties",
                "dataPath": "[2].metadata",
                "schemaPath": "#/items/allOf/1/then/oneOf/0/properties/metadata/additionalProperties",
                "params": {
                "additionalProperty": "type"
                },
                "message": "should NOT have additional properties"
            },
            {
                "keyword": "enum",
                "dataPath": "[2].metadata.type",
                "schemaPath": "#/items/allOf/1/then/oneOf/1/properties/metadata/properties/type/enum",
                "params": {
                "allowedValues": [
                    "text",
                    "number"
                ]
                },
                "message": "should be equal to one of the allowed values"
            },
            {
                "keyword": "additionalProperties",
                "dataPath": "[2].metadata",
                "schemaPath": "#/items/allOf/1/then/oneOf/2/properties/metadata/additionalProperties",
                "params": {
                "additionalProperty": "type"
                },
                "message": "should NOT have additional properties"
            },
            {
                "keyword": "oneOf",
                "dataPath": "[2]",
                "schemaPath": "#/items/allOf/1/then/oneOf",
                "params": {
                "passingSchemas": null
                },
                "message": "should match exactly one schema in oneOf"
            }
        ]
        ```

=== "Solution"
    !!! success
        In this case, an input_field was added to a part and it was not configured correctly. That results in the JSON-Schema not matching. 

        1. Click on the newly added input_field in the middle section
        2. On the left in the Configuration-Tab, select a "Field Type" to either be "number" or  "text"
            - If you choose "number", this will likely result in more Validation-Errors until you fill in all necessary values:
                - "Default Value"
                - "Minimum Value"
                - "Maximum Value"
                - "Step Size"