Linked List

| Operations | Time Complexity |
|------------|-----------------|
| Insert End | O(1)            |
| Remove End | O(1)            |
| Insert Mid | O(1)            |
| Remove Mid | O(1)            |

```mermaid
flowchart LR
    id0(head) -.-> id1
    subgraph linked list
      id1(3) --> id2
      id2(6) --> id3
      id3(8) --> id4
      id4(9) --> id5
      id5(null)
    end
    id6(mid) -.-> id2
    id7(end) -.-> id4
```
