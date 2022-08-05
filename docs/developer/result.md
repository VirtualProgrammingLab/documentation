# Result JSON Message

To inform frontend about computations in-preparation.

## Example (informal)

``` json title="Result Message Example"
{
    "type": "result",
    "content":  {
        "result": {
            "artifacts": [
                {
                    "identifier": "506d791e-18e0-4eb6-b68a-0850e250f63d",
                    "type":"file",
                    "MIMEtype": "application/octet-stream",
                    "content": "W2ludHJvZHVjdGlvbl0NCm5hbWU9S2F0Cg",
                    "path": "input.ini"
                }
            ],
            "computation":"0def42c2-8bde-478c-94c3-a1bd278861ed",
            "identifier":"7620aba7-a345-4858-9a14-50c1ad46689c",
            "output": {
                "stderr":"",
                "stdout":"SGVsbG8sIBtbMDszNW0gS2F0IBtbMG0KCg"
            },
            "status":"final",
            "timestamp":"2022-08-05T09:13:25Z"
        }
    }
}
```

### Result JSON Object

|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| type |string |must | Type of the message | Set to 'result' |
| content | object | must | Contains a [Computation Result JSON Message](computation_result.md) | |