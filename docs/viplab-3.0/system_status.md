# System Status JSON Message

To inform frontend about computations in-preparation.

## Example (informal)

``` json title="System Status Message of Type 'info'"
{
  "type": "system-status",
  "content": {
    "identifier" : "86165eea-14df-4a76-805a-09b21441cbf7",
    "version" : "3.0.0",
    "computation" : "4598393-95bf-409a-98a5-ee375982c3e",  // uuid of corresponding computation
    "status" : "info", // warning and error, 
    "timestamp" : "2022-04-21T18:21Z", // creation time of this message in ISO-8601
    "message" : "Docker container is starting"
  }
}
```

### System Status JSON Object

|Key            |Value Type |Opt / Must |Description |Comment |
|---------------|-----------|-----------|------------|--------|
| identifier    |string (uuid) |must | Unique System Status ID  | |
| version | string | must | Version of the json specification used. The major version must match the version of the ComputationTask | |
| computation | string (uuid) | must | Identifier of the computation that is currently processed | |
| status | One of {"info", "warning", "error"} | must | "info": normal status update<BR>"warning": An unexpected condition was detected during programming<BR>"error": An error occured during computation | If the status is "error", the progress bar in the GUI is canceled and the computation can be run again using the run-button |
| timestamp |ISO-8601 datetimeString | must | Timestamp when this status message is generated. Need for ordering of status messages | |
| message | string | must | Describes current state of computation | For displaying messages in the progress bar in the GUI to keep the user updated |