# Sequence Sorter Documentation

## Nodes

|     | 1   | 2   | 3   | 4   |
| --- | --- | --- | --- | --- |
| 1 | Disabled | Stack 1 Feeder | Stack 1 | Unused |
| 2 | Unused | Control Flow | Stack 1 Drainer | Lowest Value State Machine |
| 3 | Unused | Stack 2 | Stack Recycle / Output | Comparer |

## Control Signals


### Stack 1 Feeder

| Port | Node |
| ---- | ---- |
| DOWN | Control Flow |
| RIGHT | Stack 1 |

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |
| SEND      | -1     | DOWN | Stack ready to drain |
| RECV      | -1     | DOWN | Outer loop done, start over |
| RECV      | 0      | UP   | Input stream has reached end of sequence |
| RECV      | 0      | DOWN | Stack drain sequence (inner loop) is done |
| RECV      | -1     | DOWN | Outer loop done, start over |

*note: we currently duplicate listening for -1 from DOWN in 2 spots. room to optimize, maybe.*

### Control Flow

| Port | Node |
| ---- | ---- |
| UP | Stack 1 Feeder |
| DOWN | Stack 2 |
| LEFT | unused |
| RIGHT | Stack 1 Drainer |

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |
| SEND      | -1     | RIGHT | |
| SEND      | 0      | RIGHT | Write 0 to output |
| RECV      | 0      | DOWN  | Outer loop done, start over |
| RECV      | ?      | RIGHT | Unblock and proceed |
| SEND      | 0      | UP | Inner loop done |
| SEND      | -1     | UP | Outer loop done |

### Stack 1 Drainer

| Port | Node |
| ---- | ---- |
| UP | Stack 1 |
| DOWN | Stack Recycle / Output |
| LEFT | Control Flow |
| RIGHT | Lowest Value State Machine |

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |
| RECV      | -1     | LEFT | No special meaning, just unblocks |
| RECV      | 0      | LEFT | Send 0 (sequence separator) to output |
| RECV      | -1     | UP | Stack is empty |
| SEND      | -1     | DOWN | Stack is empty |
| SEND      | 0      | DOWN | Write 0 to output |
| SEND      | 0      | RIGHT | |
| SEND      | 0      | LEFT | Unblock Control Flow |

### Lowest Value State Machine

| Port | Node |
| ---- | ---- |
| UP | unused |
| DOWN | Comparer |
| LEFT | Stack 1 Drainer |

| Direction | Signal | Port | Meaning |
| --------- | ------ | ---- | ------- |

### Comparer

| Port | Node |
| ---- | ---- |
| UP | Lowest Value State Machine |
| LEFT | Stack Recycle / Output (no comms) |
