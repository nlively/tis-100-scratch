# Sequence Sorter Documentation

## Nodes

|     | 1   | 2   | 3   | 4   |
| --- | --- | --- | --- | --- |
| 1 | Disabled | Stack 1 Feeder | Stack 1 | Unused |
| 2 | Unused | Control Flow | Stack 1 Drainer | Lowest Value State Machine |
| 3 | Unused | Stack 2 | Stack Recycle / Output | Comparer |

## Control Signals and Value Passing

### Stack 1 Feeder

#### Memory Values

* ACC is used to detect control signals and sequence terminations
* BAK is unused

#### Adjacent Ports

| Port | Node |
| ---- | ---- |
| DOWN | Control Flow |
| RIGHT | Stack 1 |

#### Control Signals

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |
| SEND      | -1     | RIGHT | Push termination signal as first value onto Stack 1 |
| SEND      | -1     | DOWN | Stack ready to drain |
| RECV      | -1     | DOWN | Outer loop done, start over |
| RECV      | 0      | UP   | Input stream has reached end of sequence |
| RECV      | 0      | DOWN | Stack drain sequence (inner loop) is done |
| RECV      | -1     | DOWN | Outer loop done, start over |

*note: we currently duplicate listening for -1 from DOWN in 2 spots. room to optimize, maybe.*

#### Values

| Direction | Port | Purpose |
| --------- | ---- | ------- |
| RECV      | UP   | Take value from input stream |
| RECV      | DOWN | Take value from Stack 2 (via Control Flow) |
| SEND      | RIGHT | Push input onto Stack 1 |

### Control Flow

#### Memory Values

* ACC is used to hold count of remaining items on Stack 2
* BAK is unused

#### Adjacent Ports

| Port | Node |
| ---- | ---- |
| UP | Stack 1 Feeder |
| DOWN | Stack 2 |
| LEFT | unused |
| RIGHT | Stack 1 Drainer |
 
#### Control Signals

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |
| SEND      | -1     | RIGHT | |
| SEND      | 0      | RIGHT | Write 0 to output |
| RECV      | 0      | DOWN  | Outer loop done, start over |
| RECV      | ?      | RIGHT | Unblock and proceed |
| SEND      | 0      | UP | Inner loop done |
| SEND      | -1     | UP | Outer loop done |

#### Values

| Direction | Port | Purpose |
| --------- | ---- | ------- |
| RECV      | DOWN | Pop value off of Stack 2 |
| SEND      | UP   | Pass Stack 2 value to Stack 1 Feeder |

### Stack 1 Drainer

#### Memory Values

* ACC is used to hold popped Stack 1 value and detect control signal
* BAK is unused

#### Adjacent Ports

| Port | Node |
| ---- | ---- |
| UP | Stack 1 |
| DOWN | Stack Recycle / Output |
| LEFT | Control Flow |
| RIGHT | Lowest Value State Machine |

#### Control Signals

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |
| RECV      | -1     | LEFT | No special meaning, just unblocks |
| RECV      | 0      | LEFT | Send 0 (sequence separator) to output |
| RECV      | -1     | UP | Stack is empty |
| SEND      | -1     | DOWN | Stack is empty |
| SEND      | 0      | DOWN | Write 0 to output |
| SEND      | 0      | RIGHT | Unblock reset and proceed |
| SEND      | 0      | LEFT | Unblock Control Flow |

#### Values

| Direction | Port | Purpose |
| --------- | ---- | ------- |
| RECV      | UP   | Pop value off of Stack 1 |
| SEND      | RIGHT | Send value to Lowest Value State Machine for comparison |
| RECV      | RIGHT | Receive higher of the 2 values from Lowest Value State Machine |
| RECV      | RIGHT | [at end of sequence] Receive lowest value from Lowest Value State Machine |


### Lowest Value State Machine

#### Memory Values

* BAK holds current low for each loop
* ACC stages input and is used to detect control signals

#### Adjacent Ports

| Port | Node |
| ---- | ---- |
| UP | unused |
| DOWN | Comparer |
| LEFT | Stack 1 Drainer |

#### Control Signals

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |
| RECV      | -1     | LEFT | Reset   |

#### Values

| Direction | Port | Purpose |
| --------- | ---- | ------- |
| RECV      | LEFT | Input to compare |
| SEND      | LEFT | Higher of two inputs |
| SEND      | LEFT | [at end of sequence] Retained lowest value |

### Comparer

#### Memory Values

* BAK holds saved input 1
* ACC holds result of subtraction
* After subtraction, ACC stages input 2
* Memory overwritten every loop

#### Adjacent Ports

| Port | Node |
| ---- | ---- |
| UP | Lowest Value State Machine |
| LEFT | Stack Recycle / Output (no comms) |

#### Control Signals

_**does not receive any control signals._

#### Values

| Direction | Port | Purpose |
| --------- | ---- | ------- |
| RECV      | UP   | First input to compare |
| RECV      | UP   | Second input to compare |
| RECV      | UP   | Second input to compare (again, because initial was used in SUB equation) |
| SEND      | UP   | Lower value |
| SEND      | UP   | Higher value |


### Stack Recycle / Output

#### Memory Values

* BAK holds running count of values recylced onto Stack 2
* ACC stages incoming values and detects control signals

#### Adjacent Ports

| Port | Node |
| ---- | ---- |
| UP | Stack 1 Drainer |
| DOWN | Output Stream |
| LEFT | Stack 2 |
| RIGHT | Comparer (no comms) |

#### Control Signals

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |
| RECV      | -1     | UP   | Inner loop done (so grab lowest value and pass to output) |
| RECV      | 0      | UP   | Outer loop done (send 0 to output)

#### Values

| Direction | Port | Purpose |
| --------- | ---- | ------- |
| SEND      | LEFT | Recycle value onto Stack 2 |
| SEND      | LEFT | [at end of each loop] Count of values sent to Stack 2 |
| SEND      | DOWN | Write a value to output |
