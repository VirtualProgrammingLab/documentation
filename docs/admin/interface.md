# Interface to Websocket API

The Repository containing the code of the Websocket API can be found on [Git](https://github.com/VirtualProgrammingLab/viplab-websocket-api).

The Websocket API is based on JSON messages. 
Those messages are sent between the browser and the server. 
Each of the messages has a type and a content, whereby the type determines how the content of the message is processed. 

``` json title="JSON Message"
{
    "$schema": "http://json-schema.org/draft-07/schema",
    "$id": "message-schema.json",
    "title": "Message",
    "description": "Message format of the ViPLab Websocket API",
    "type": "object",
    "properties": {
        "type": {
            "type": "string",
            "description": "The type of the message"
        },
        "content": {
            "type": "object",
            "description": "The content of the message"
        }
    },
    "required": [
        "type",
        "content"
    ]
}
```

## Message Types

### Messages from Browser to Server

#### authenticate

After establishing a WebSocket connection, authentication takes place. 
Therefore the first message sent ist the authenticate message. 
This message contains authentication tokens in terms of JWTs, that are provided to the browser by an external service. 

``` json title="authenticate Message Example"
{
    "type": "authenticate",
    "content": {
        "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
    }
}
```

#### create-computation

Message to create a new [Computation](../developer/computation.md) from a [Computation Template](../developer/computation_template.md) and a [Computation Task](../developer/computation_task.md). 
This Computation is sent back to the browser using a [computation message](#computation)

``` json title="create-computation Message Example"
{
    "type": "create-computation",
    "content": {
        "template" : "eyJpZGVudGlmaWVyIjoiMTE0ODNmMjMtOTViZi00MjRhLTk4YTUtZWU1ODY4Yzg1YzNmIiw...",
        "task" : {
            "template" : "11483f23-95bf-424a-98a5-ee5868c85c3f",
            "identifier" : "b547d5f0-9346-05c6-6bdd-8e32e50abc96",
            "files" : [ 
                {
                    "identifier" : "22483f42-95bf-984a-98a5-ee9485c85c3f",
                    "parts" : [
                        {
                            "identifier": "f3fc4404-3529-4962-b252-47bc4ddd02a1",
                            "content": "eyJfX3NsaWRlclNpbmdsZV9fIjoiNzUifQ"
                        },
                        {
                            "identifier": "ceb051d8-b50c-4814-983a-b9d703cae0c6",
                            "content": "eyJfX2NoZWNrYm94X18iOiJwcm9ncmFtbWluZyIsIl9fcmFkaW9CdXR0b25fXyI6IkphdmEiLCJfX2Ryb3Bkb3duU2luZ2xlX18iOiJuZXZlciIsIl9fZHJvcGRvd25NdWx0aXBsZV9fIjoiTGFzdCBDaHJpc3RtYXMiLCJfX3RvZ2dsZV9fIjoiU3BpZGVycyIsIl9fc2xpZGVyTXVsdGlwbGVfXyI6IjI1LDUwLDc1IiwiX19pbnB1dFRleHRXT01heGxlbmd0aF9fIjoiYmFzZTY0OmF3IiwiX19pbnB1dFRleHRXTWF4bGVuZ3RoX18iOiJiYXNlNjQ6ZHciLCJfX2lucHV0TnVtYmVyX18iOiIyNSJ9"
                        }
                    ]
                },
                {
                    "identifier" : "22483f42-95bf-984a-98a5-ee9485c85c31",
                    "parts" : [
                        {
                            "identifier": "f3fc4404-3529-4962-b252-47bc4ddd02a2",
                            "content": "eyJfX2RlZmF1bHRfXyI6ImJhc2U2NDphVzUwSUcxaGFXNG9hVzUwSUdGeVoyTXNJR05vWVhJZ0tpcGhjbWQyS1NCN0lBMEtMeThnVUhKcGJuUWdKMGhsYkd4dklGZHZjbXhrSnlBTkNuMCJ9"
                        },
                    ]
                }
            ]
        }
    }
}
```

#### prepare-computation

Message to inform the backend about the computation that is currently worked on.  
The backend can then prepare for running the computation. 
For example, if the value for environment in the [computation template](../developer/computation_template.md) is `docker`, the backend can prepare by already pulling the respective docker image.

This message is identical to the [create-computation message](#create-computation) above, apart from the message type. 
After preparation, an information is sent back to the browser using a [prepared-computation message](#prepared-computation).

The goal of this message for the future is, that the template ultimately only has to be sent once. 
For this, the 'prepare-computation-id' is used, that is returned by the [prepared-computation message](#prepared-computation).

``` json title="prepare-computation Message Example"
{
    "type": "prepare-computation",
    "content": {
        "template" : "eyJpZGVudGlmaWVyIjoiMTE0ODNmMjMtOTViZi00MjRhLTk4YTUtZWU1ODY4Yzg1YzNmIiw...",
        "task" : {
            "template" : "11483f23-95bf-424a-98a5-ee5868c85c3f",
            "identifier" : "b547d5f0-9346-05c6-6bdd-8e32e50abc96",
            "files" : [ ... ]
        }
    }
}
```

#### subscribe

Using this message, one can subscribe to different events. 
For subscribing to an event, the respective event needs to be passed in the message content as string in the topic property. 
The string is separated by a colon. 
The message is idempotent, thus you can only subscribe to an event once per WebSocket session. 
No response is sent back from the server to confirm the subscription.

| Event | Description |
| ----- | ----------- |
| `computation:{id}` | All events about one computation |

``` json title="subscribe Message Example"
{
    "type": "subscribe",
    "content": {
        "topic": "computation:b30fca08-7267-4aa9-adc7-8c89f5efd928"
    }
}
```

### Messages from Server to Browser

#### computation

This message is used to send a Computation back to the browser.

``` json title="computation Message Example"
{
    "type": "computation",
    "content": {
        "id": "20f32dc9-e43f-4a65-af8c-42412ed9d977",
        "created": "2019-08-19T13:31:30+0200",
        "expires": "2019-08-19T16:31:30+0200",
        "status": "created"
    }
}
```

#### prepared-computation

This message is used to signal, that preparation of the backend was successful. 
The id that is returned is a prepare-computation-id.

``` json title="prepared-computation Message Example"
{
    "type": "prepared-computation",
    "content": {
        "id": "20f32dc9-e43f-4a65-af8c-42412ed9d977",
        "created": "2019-08-19T13:31:30+0200",
        "expires": "2019-08-19T16:31:30+0200",
        "status": "prepared"
    }
}
```