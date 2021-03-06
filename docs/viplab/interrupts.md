!!! note
    It's unclear, when Interrupts will be implemented: for short computations only this is not necessary (but for e.g. long running simulations).

    @Heiko: what's the status of the broadcasting mechanism?

    --sr

# /numlab/interrupts

Broadcast message queue.

## Interface

POST, GET (see [''English section name unknown so far.''](/ecs2#queue-resource))

### Generic

* **POST resourceURL (push)**: creates Interrupt.
* **POST resourceURL/fifo (pull)**: gets first Interrupt and removes it from the queue.
* **GET resourceURL/fifo (look)**: gets first Interrupt (fifo) without removing it from the queue.

### VipLab Specific

POST resourceURL (push) will be used by SCs, POST resourceURL/fifo (pull) by CCs.

Student clients (SCs) push Interrupts into this queue; computation clients (CCs) pulls them from there (each CC gets all Interrupts (broadcast)). For getting next Interrupt the former one has to be removed (by POST resourceURL/fifo (pull)).

* **POST resourceURL (push)**: SC creates Interrupt for interrupting a computation of a formerly posted Solution.
* **POST resourceURL/fifo (pull)**: each CC pulls next available Interrupt. If this Interrupt matches a currently computed Solution (affects only ''one'' CC), its computation will be finished, otherwise (all other CCs) it will be ignored.
* **GET resourceURL/fifo (look)**: for debugging purposes only.

Notes:

* For getting Interrupts in sensible time, CCs have to poll the queue often enough.
* CCs have to remove all Interrupt messages not affecting them (from CC point of view it has its own queue getting all broadcasted messages). This should be done regularily ''outside'' computations, too.

## Message

### Interrupt JSON Definition by Example (informal)
```
{ "Interrupt": // wrapper
{
  "postTime" : "1985-04-13T18:00:00.12Z", // must
  "ID"       : "#11", // must, for tracking and debugging
  "comment"  : "don't wanna wait anymore", // opt

  "Solution" :
  {
    "ID" : "#37" // ID of Solution formerly generated by SC
  } // Solution
}
} // "Interrupt"
```

### Interrupt JSON Format

 wrapper "Interrupt" around the following

|Key [--Subkey]            |Type |Opt / Must |Description |Comment |
|--------------------------|-----|-----------|------------|--------|
|postTime |string UTC |must |timestamp from SC|Timestamp from SC: in addition a CC side timestamp could be made.|
|ID                   |string     |must | unique ID created by SC (automatically) | For tracking and debugging purposes. |
|comment              |string     |opt              | | |
|Solution             |struct     |must             | | |
|Solution --ID        |String     |must             |refers to unique Solution ID | |


###Notes
* The computation the Interrupt applies to is defined by
    * Solution--ID above, and
    * sender identity: this is given outside this JSON-Message.
* An alternative could be to use UUIDs for Solution--IDs: but we already need/have some sender identity for routing back Results to Solution senders (and can use it here, too).
